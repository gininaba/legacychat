# API Integration & Streaming Guide

This document describes how LegacyChat connects to public **OpenRouter** API endpoints, formats text and multimodal vision payloads, and processes streaming responses using ES5 `XMLHttpRequest` techniques compatible with early browser engines (specifically iOS 9 Safari).

---

## 1. Public Endpoints Reference

LegacyChat connects directly from the browser to the following public endpoints:

### A. List Available Models
- **Method**: `GET`
- **URL**: `https://openrouter.ai/api/v1/models`
- **Authentication**: None required (public endpoint)
- **Role**: Refreshes local model dropdown options dynamically, sorting models alphabetically.

### B. Chat Completions (with SSE Streaming)
- **Method**: `POST`
- **URL**: `https://openrouter.ai/api/v1/chat/completions`
- **Authentication**: Bearer Token (`Authorization: Bearer <API_KEY>`)
- **Role**: Transmits conversation history and parameters to stream AI completions token by token.

---

## 2. Request HTTP Headers

The following headers are attached to completion write operations to satisfy security, CORS requirements, and OpenRouter rankings:

```http
Content-Type: application/json
Authorization: Bearer sk-or-v1-xxxxxxxxxxxxxxxxxxxxxxx
HTTP-Referer: http://localhost:3000/ (or active deployed URL)
X-Title: LegacyChat iPad
```

*Note: `HTTP-Referer` and `X-Title` enable proper app branding and analytics breakdown on the OpenRouter dashboard.*

---

## 3. Request Payload Specifications

### Standard Text Payload
```json
{
  "model": "anthropic/claude-3.5-sonnet",
  "messages": [
    { "role": "system", "content": "You are a helpful assistant." },
    { "role": "user", "content": "Hello! What is 2+2?" }
  ],
  "max_tokens": 2048,
  "temperature": 0.7,
  "stream": true
}
```

### Vision Multimodal Payload (Image Uploads)
When a camera photo is attached, the user message converts to a multimodal array containing optimized JPEG Base64 data:

```json
{
  "role": "user",
  "content": [
    {
      "type": "text",
      "text": "Describe the contents of this image in detail."
    },
    {
      "type": "image_url",
      "image_url": {
        "url": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEA..."
      }
    }
  ]
}
```

---

## 4. ES5 Streaming & 60ms Render Throttling Architecture

Modern web apps use `fetch()` and `ReadableStream`. Because iOS 9 Safari lacks both `fetch` and `ReadableStream`, LegacyChat implements streaming using `XMLHttpRequest` state change events (`readyState === 3` for LOADING and `readyState === 4` for DONE):

```mermaid
sequenceDiagram
    participant UI as Browser UI
    participant Buffer as Sliding Window Buffer
    participant Render as 60ms Throttled Renderer
    participant XHR as XMLHttpRequest (readyState 3)
    participant OR as OpenRouter API

    UI->>OR: POST /chat/completions (stream: true)
    loop SSE Chunks Transmission
        OR-->>XHR: Send raw SSE chunks ("data: {...}\n\n")
        XHR->>Buffer: Calculate responseText.substring(seenLength)
        Buffer->>Buffer: Extract JSON deltas & append to currentStreamText
        loop Every 60ms Timer (~16fps)
            Render->>UI: Execute formatMarkdown() & update bubble innerHTML
            Render->>UI: scrollToBottom()
        end
    end
    OR-->>XHR: data: [DONE]
    XHR->>Render: Finalize formatMarkdown() & clear timer
    Render->>UI: Store finalized thread in localStorage
```

### Buffer Extraction Algorithm
1. `seenLength` stores character count processed so far.
2. On `readyState 3` or `4`, newly arrived text is sliced:
   ```javascript
   var newData = xhr.responseText.substring(seenLength);
   seenLength = xhr.responseText.length;
   buffer += newData;
   ```
3. Segments split on double newlines (`\n\n`), leaving trailing partial lines in the buffer.
4. Segments stripped of `data: ` prefix are parsed as JSON objects to update aggregate `sessionTokens` and append delta content strings to `currentStreamText`.
5. **60ms Render Timer**: A 60ms `setInterval` periodically updates the active DOM bubble innerHTML with `sanitizeHTML(formatMarkdown(currentStreamText))`, delivering smooth visual generation without CPU thrashing on Apple A5 dual-core processors.

---

## 5. Comprehensive Error & Quota Handling Matrix

| Error Condition | HTTP Code / Trigger | User Interface Alert / Message | Remediation Action |
|---|---|---|---|
| **Unauthorized** | `401` | *"Unauthorized. Please check your API key."* | Prompt user to verify OpenRouter API key in Settings. |
| **Rate Limit Exceeded** | `429` | *"Rate limit exceeded. Please wait a moment and try again."* | Wait briefly before retrying request. |
| **Credit / Quota Limit** | `402` or `400` | Extracts provider error (e.g. *"This request requires more credits, or fewer max_tokens..."*) | Lower `Max Tokens` in Settings (e.g. `80`), switch to a Free model preset, or add OpenRouter credits. |
| **Request Timeout** | `xhr.ontimeout` (60s) | *"Request timed out. Please check your connection and try again."* | Check device WiFi network or select a faster model (e.g. Claude 3 Haiku). |
| **Network Error** | `xhr.onerror` | *"Network error. Check your connection."* | Verify URL connectivity or tunnel server. |
| **Local Storage Full** | `QuotaExceededError` | *"Storage Limit Exceeded: LegacyChat cannot save history..."* | Prompt user to run **Clear All** or **Export** history in History Drawer. |
