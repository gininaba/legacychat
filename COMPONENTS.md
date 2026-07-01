# Code Architecture & Component Breakdown

This document provides a comprehensive analysis of the components, global states, and functions defined within the standalone [index.html](file:///Volumes/1TB%20Graphics%20SSD/Sansan_DO_NOT_TOUCH/Projects/legacychat/index.html) file.

---

## 1. UI & DOM Architecture

LegacyChat's layout is split into two major screens toggleable via DOM style manipulation, alongside drawer overlays:

```
┌────────────────────────────────────────────────────────┐
│                        index.html                      │
├───────────────────────────┬────────────────────────────┤
│   #setup-screen           │   #chat-screen             │
│   (Configuration Form)    │   (Active Conversation)    │
│   ├── API Key Input       │   ├── Navigation Bar       │
│   ├── Model Filter        │   ├── Messages Viewport    │
│   ├── Model Dropdown      │   │   ├── Bubbles          │
│   ├── Persona Select      │   │   └── Actions          │
│   ├── Theme Selector      │   └── Input & Control Bar  │
│   └── Advanced settings   │                            │
│                           │   #history-drawer (Overlay)│
│                           │   └── Search & Chat List   │
└───────────────────────────┴────────────────────────────┘
```

---

## 2. Global State Variables

The application manages local state using the following ES5 variables:

| Variable Name | Type | Purpose / Description |
|---|---|---|
| `API_KEY` | `String` | Stores the user's OpenRouter API key. |
| `MODEL` | `String` | The model slug selected (e.g., `anthropic/claude-3.5-sonnet`). |
| `SYSTEM_PROMPT` | `String` | Instructions injected as the `system` role roleplay context. |
| `conversationHistory` | `Array` | List of JSON message objects: `[{role: "user"|"assistant", content: String|Array}]`. |
| `isLoading` | `Boolean` | Flag indicating an active API request is in progress. |
| `chats` | `Array` | History list of all stored conversations loaded from `localStorage`. |
| `currentImageBase64` | `String` | Holds the Base64 representation of the currently attached image. |
| `ADVANCED` | `Object` | Configuration object: `{ temp: Number, tokens: Number }`. |
| `currentChatId` | `String` | Timestamp string matching the active conversation ID. |
| `currentXHR` | `XMLHttpRequest` | References the active streaming XHR instance to allow aborting. |
| `sessionTokens` | `Number` | Track aggregated token usage in the active session. |
| `customPersonas` | `Array` | Array of saved user-created system prompt structures. |
| `allModels` | `Array` | Cached model metadata containing `{ id: String, name: String }`. |

---

## 3. Core JavaScript Functions Reference

Below is a breakdown of key functional blocks inside the `<script>` section of `index.html`.

### API Communications & Streaming

#### A. `callAPI(callback, onChunk)`
- **Purpose**: Establishes a POST request to OpenRouter's `/chat/completions` API using standard `XMLHttpRequest` with SSE (Server-Sent Events) streaming chunk processing.
- **Inputs**:
  - `callback`: `function(reply, error)` - Invoked upon complete execution or error.
  - `onChunk`: `function(chunk)` - Invoked progressively as character sequences stream in.
- **Outputs**: None (Asynchronous side-effects).
- **Dependencies**: `API_KEY`, `MODEL`, `SYSTEM_PROMPT`, `conversationHistory`, `ADVANCED`, `xhr.onreadystatechange`.
- **Usage Example**:
  ```javascript
  callAPI(function(reply, err) {
    if (err) console.error("Error: ", err);
    else console.log("Complete!");
  }, function(chunk) {
    console.log("Chunk received: ", chunk);
  });
  ```

---

### Markdown & HTML Parsing

#### B. `formatMarkdown(text)`
- **Purpose**: Parse raw Markdown text into safe ES5-compatible HTML (supporting lists, headers, bold, italic, code blocks with syntax highlighting, and GFM tables).
- **Inputs**: `text` (`String`) - Raw markdown.
- **Outputs**: `String` - Formatted HTML.
- **Dependencies**: `escapeHTML()`.
- **Usage Example**:
  ```javascript
  var rawMarkdown = "### Hello\nThis is **bold** and `code`.";
  var renderedHtml = formatMarkdown(rawMarkdown);
  // Outputs: "<h3>Hello</h3><p>This is <strong>bold</strong> and <code>code</code>.</p>"
  ```

#### C. `sanitizeHTML(htmlString)`
- **Purpose**: Sanitizes input HTML string, removing dangerous elements to guard against Cross-Site Scripting (XSS).
- **Inputs**: `htmlString` (`String`) - Unsanitized markup.
- **Outputs**: `String` - Clean HTML.
- **Dependencies**: None.
- **Usage Example**:
  ```javascript
  var dangerousHtml = '<script>alert("XSS")</script><p onload="run()">Hello</p>';
  var safeHtml = sanitizeHTML(dangerousHtml);
  // Outputs: '<p>Hello</p>'
  ```

---

### Image Upload & Compression

#### D. `handleImageUpload(event)`
- **Purpose**: Reads user-uploaded image files, resizes them using a hidden HTML5 `<canvas>`, and converts them to a lightweight JPEG Base64 URI.
- **Inputs**: `event` (`Event`) - File selection input change event.
- **Outputs**: None (Sets `currentImageBase64`).
- **Dependencies**: `FileReader`, `Image`, `HTMLCanvasElement`.
- **Usage Example**:
  ```html
  <input type="file" onchange="handleImageUpload(event)">
  ```

---

### Storage & History Management

#### E. `saveChatsToStorage()`
- **Purpose**: Backs up all chat histories to `localStorage`. Stored images from previous chats are dynamically cleared (replaced by lightweight SVG templates) to fit within browser storage quotas.
- **Inputs**: None.
- **Outputs**: None.
- **Dependencies**: `localStorage`, `chats`, `currentChatId`, `safeLocalStorageSetItem()`.
- **Usage Example**:
  ```javascript
  saveChatsToStorage();
  ```

#### F. `safeLocalStorageSetItem(key, value)`
- **Purpose**: Safely commits a key-value pair to `localStorage`, catching `QuotaExceededError` and alerting the user of resolution steps if full.
- **Inputs**:
  - `key` (`String`)
  - `value` (`String`)
- **Outputs**: `Boolean` - Returns `true` if successful, `false` otherwise.
- **Dependencies**: `localStorage`.
- **Usage Example**:
  ```javascript
  var success = safeLocalStorageSetItem('theme', 'dark');
  ```
