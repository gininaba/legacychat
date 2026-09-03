# Code Architecture & Component Breakdown

This document provides a technical analysis of the visual DOM hierarchy, CSS styling systems, global state variables, and functional blocks defined within [`index.html`](file:///Volumes/1TB%20Graphics%20SSD/Sansan_DO_NOT_TOUCH/Projects/legacychat/index.html).

---

## 1. UI & DOM Architecture Hierarchy

LegacyChat renders two main view states (`#setup-screen` and `#chat-screen`), alongside slide-over drawer panels and modal dialogs:

```text
index.html
├── <head> (Meta tags, Apple PWA WebApp icons, CSS stylesheet)
└── <body>
    ├── #setup-screen (Configuration Screen)
    │   └── .setup-card
    │       ├── h1 / p (Header & Description)
    │       ├── #group-api-key (API Key Input)
    │       ├── #group-model (Model Filter & Dropdown Selector)
    │       ├── #group-persona (Persona Selector & Custom System Prompt Input)
    │       ├── #group-theme (Light / Dark Theme Selector)
    │       ├── #group-temp & #group-tokens (Advanced Temperature & Token Cap Sliders)
    │       └── #start-btn ("Start Chatting" Action Button)
    │
    ├── #chat-screen (Active Chat Environment)
    │   └── .chat-container
    │       ├── .nav-bar (Glassmorphic Header)
    │       │   ├── .nav-btn (Open History Drawer Button)
    │       │   ├── .nav-title (App Title, Model Badge, Token Counter)
    │       │   └── .nav-btn ("New Chat" Action Button)
    │       ├── #scroll-bottom-btn (Floating SVG Chevron Scroll Button)
    │       ├── .messages-wrap#messages (Scrollable Message Viewport)
    │       │   ├── .empty-state#empty-state (Landing Prompt View)
    │       │   └── .msg-row (Dynamic Message Rows with slideInUp Animation)
    │       │       └── .bubble (User gradient bubble / AI white-dark bubble)
    │       │           ├── Markdown HTML Content / Syntactical Code Blocks
    │       │           └── .bubble-actions (Copy, Edit, Regenerate, Delete Buttons)
    │       └── .input-bar (Glassmorphic Footer Bar)
    │           ├── .attach-btn & #image-upload (File Selector Input)
    │           ├── .input-wrap
    │           │   └── .input-col
    │           │       ├── #image-preview-wrap (Base64 Image Thumbnail Preview)
    │           │       ├── #user-input (Auto-resizing Textarea)
    │           │       └── #char-counter (Surfaces when prompt > 500 chars)
    │           ├── #stop-btn (Abort Generation XHR Button)
    │           └── #send-btn (Submit Prompt Button)
    │
    ├── #confirm-overlay (Single Thread Clear Confirmation Modal)
    ├── #confirm-all-overlay (Clear All History Confirmation Modal)
    ├── #history-overlay (Drawer Backdrop Overlay)
    └── #history-drawer (Slide-Over Navigation Drawer)
        ├── .drawer-header (Title & Close Button)
        ├── #history-search (Debounced Chat Search Input)
        ├── #history-list (Dynamic Saved Threads List)
        └── .drawer-footer (Export, Import, Clear All, and Settings Buttons)
```

---

## 2. Global State Variables Reference

All application state is maintained in standard ES5 global variables:

| Variable Name | Type | Purpose & Lifecycle Description |
|---|---|---|
| `API_KEY` | `String` | Stores OpenRouter credentials loaded from or committed to `localStorage.api_key`. |
| `MODEL` | `String` | Selected model slug (e.g. `anthropic/claude-3.5-sonnet`). |
| `SYSTEM_PROMPT` | `String` | System role instruction injected at index 0 of completion request payloads. |
| `conversationHistory` | `Array` | Active thread message objects array: `[{role: "user"|"assistant", content: String|Array}]`. |
| `isLoading` | `Boolean` | True when an API streaming request is in progress; disables send buttons. |
| `chats` | `Array` | Complete array of historical conversation objects loaded from `localStorage.chats`. |
| `currentImageBase64` | `String` | Holds Base64 JPEG data URL of currently attached image before sending. |
| `ADVANCED` | `Object` | Configuration object: `{ temp: Number, tokens: Number }`. |
| `currentChatId` | `String` | Unique timestamp ID matching active thread in `chats`. |
| `currentXHR` | `XMLHttpRequest` | Active streaming `XMLHttpRequest` instance; used by `stopGeneration()`. |
| `sessionTokens` | `Number` | Aggregated count of total tokens consumed in the current active session. |
| `customPersonas` | `Array` | Saved user custom system prompts array loaded from `localStorage.custom_personas`. |
| `allModels` | `Array` | Models catalog array containing `{ id: String, name: String }`. |
| `saveChatsDebounceTimer` | `Timeout` | 500ms debounce timer for writing history to browser storage. |
| `filterHistoryDebounceTimer` | `Timeout` | 150ms debounce timer for history list search filtering. |

---

## 3. Categorized JavaScript Functions Reference

### A. Lifecycle & Screen Navigation

#### `startChat()`
- **Purpose**: Validates settings form inputs, updates global variables (`API_KEY`, `MODEL`, `SYSTEM_PROMPT`), commits values to `localStorage`, hides `#setup-screen`, displays `#chat-screen`, and initializes the chat viewport.
- **Dependencies**: `safeLocalStorageSetItem()`, `loadChat()`, `startNewChat()`.

#### `showSettings()`
- **Purpose**: Switches view from `#chat-screen` back to `#setup-screen`, populating form controls with current global variable values.

#### `toggleTheme(theme)`
- **Purpose**: Applies or removes `.dark-mode` CSS class on `document.body` and stores selection (`light` / `dark`) in `localStorage`.

---

### B. Generation Pipeline & API Communications

#### `sendMessage()`
- **Purpose**: Reads `#user-input`, validates text or image attachments, appends user bubble to DOM, pushes message to `conversationHistory`, resets input controls, and triggers `runGeneration()`.

#### `runGeneration(onComplete)`
- **Purpose**: Handles complete completion lifecycle. Spawns typing indicator, establishes API request, and initializes a **60ms interval render timer** (`~16fps`) to buffer incoming SSE chunks without thrashing legacy CPU cores.
- **Dependencies**: `callAPI()`, `showTyping()`, `removeTyping()`, `formatMarkdown()`, `cleanAIResponse()`, `debouncedSaveChatsToStorage()`.

#### `regenerateMessage(btn)`
- **Purpose**: Truncates `conversationHistory` up to selected AI message index, removes trailing DOM bubbles, saves thread, and calls `runGeneration()`.

#### `stopGeneration()`
- **Purpose**: Invokes `currentXHR.abort()`, removes active typing indicator, resets `isLoading = false`, and restores send button state.

#### `callAPI(callback, onChunk)`
- **Purpose**: Sends POST payload to OpenRouter `/chat/completions` endpoint via standard `XMLHttpRequest` with `readyState === 3` SSE buffer parsing.
- **Error Handling**: Extracts HTTP 400, 401, 402, 429, and 60s timeout error message strings for user feedback.

#### `generateTitleIfNeeded()`
- **Purpose**: Sends a lightweight completion request (`max_tokens: 10`) after initial message exchange to generate a concise 3-word title for the thread.

---

### C. DOM Traversal & UI Helpers

#### `findParentRow(element)`
- **Purpose**: Safely walks up parent nodes from `element` until it finds the enclosing `.msg-row` container, providing robust traversal independent of DOM structure changes.
```javascript
function findParentRow(element) {
  var cur = element;
  while (cur && cur !== document.body) {
    if (cur.className && cur.className.indexOf('msg-row') > -1) {
      return cur;
    }
    cur = cur.parentNode;
  }
  return null;
}
```

#### `appendBubble(role, text, isError, noScroll, isRawHtml, rawTextContent)`
- **Purpose**: Dynamically constructs message row markup (`.msg-row`), applies sender styles, formats content, appends bubble action buttons (Copy, Edit, Regenerate, Delete), and inserts into `#messages`.

#### `copyCodeBlock(btn)`
- **Purpose**: Extracts raw code string from code block container and writes it to clipboard using `document.execCommand('copy')`, displaying temporary "Copied!" button feedback.

#### `copyMessage(btn)`
- **Purpose**: Writes raw message text stored in `data-raw` attribute to clipboard.

#### `scrollToBottom(force)`
- **Purpose**: Scrolls `#messages` viewport to maximum scroll height. Shows/hides `#scroll-bottom-btn` based on viewport scroll position.

---

### D. Markdown Engine & Sanitization

#### `formatMarkdown(text)`
- **Purpose**: Parses raw Markdown string into HTML output:
  - Code Blocks (` ```lang ... ``` `) → Extracted into syntax-highlighted containers with header copy buttons.
  - Headers (`#`, `##`, `###`) → `<h1>`, `<h2>`, `<h3>`.
  - Inline Formatting → `**bold**`, `__bold__`, `*italic*`, `_italic_`, `~~strikethrough~~`, `` `inline code` ``.
  - Lists & Tables → Unordered (`<ul>`), Ordered (`<ol>`), and Markdown GFM tables (`<table>`).

#### `sanitizeHTML(htmlString)`
- **Purpose**: Purifies HTML output, stripping dangerous elements (`<script>`, `<iframe>`, `<object>`, `<style>`, `<link>`), event handlers (`onload`, `onerror`, `onclick`), and `javascript:` URIs to prevent XSS attacks.

---

### E. Image Canvas Compression

#### `handleImageUpload(event)`
- **Purpose**: Reads selected image file via `FileReader`, resizes image on hidden `<canvas>` to maximum 800x800 resolution, converts canvas to 60% JPEG Base64 URI (`currentImageBase64`), and renders thumbnail preview.

#### `removeImage()`
- **Purpose**: Clears `currentImageBase64` data and hides thumbnail preview container.

---

### F. Storage, History & Maintenance

#### `loadChatsFromStorage()`
- **Purpose**: Parses `localStorage.chats` JSON string into global `chats` array.

#### `saveChatsToStorage()`
- **Purpose**: Serializes `chats` array to `localStorage`. Replaces image Base64 payloads from historical threads with tiny SVG placeholder strings to fit within 5MB browser limits.

#### `debouncedSaveChatsToStorage()`
- **Purpose**: Wraps `saveChatsToStorage()` in a 500ms timer to prevent rapid storage writes during fast message streaming.

#### `pruneStaleDrafts()`
- **Purpose**: Enumerates `localStorage` keys on startup and removes any `draft_*` key whose chat ID no longer exists in `chats`.

#### `clearAllChats()`
- **Purpose**: Clears all thread objects in `chats`, updates `localStorage`, re-renders history list, and resets UI to a new conversation.

#### `exportHistory()` & `importHistory(event)`
- **Purpose**: Downloads `chats` history array as a `legacychat-history.json` file or parses an uploaded JSON file to merge historical threads.
