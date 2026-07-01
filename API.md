# API Integration & Streaming Guide

This document describes how LegacyChat interacts with external API endpoints to fetch model indexes, complete chat payloads, and stream tokens progressively using standard ES5 `XMLHttpRequest` techniques compatible with early browser environments.

---

## 1. Endpoints Reference

LegacyChat connects directly to the following public **OpenRouter** API endpoints:

### A. List Available Models
- **Protocol / Method**: `GET`
- **URL**: `https://openrouter.ai/api/v1/models`
- **Authentication**: None required (public endpoint)
- **Role**: Refreshes the local dropdown options with all available models on OpenRouter, sorted alphabetically.

### B. Chat Completion (with Streaming)
- **Protocol / Method**: `POST`
- **URL**: `https://openrouter.ai/api/v1/chat/completions`
- **Authentication**: Bearer Token (`Authorization: Bearer <API_KEY>`)
- **Role**: Submits conversation history and parameters to return AI answers.

---

## 2. HTTP Headers

To satisfy security, CORS requirements, and OpenRouter rankings, the following headers are appended to all write operations:

```http
Content-Type: application/json
Authorization: Bearer sk-or-v1-xxxxxxxxxxxxxxxxxxxxxxx
HTTP-Referer: http://localhost:3000/ (or your deployed URL)
X-Title: LegacyChat iPad
```

*Note: The `HTTP-Referer` and `X-Title` headers allow your app to be recognized properly on the OpenRouter dashboard and analytics pages.*

---

## 3. Request Payload Specifications

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

### Vision Support (Multimodal Inputs)
If the user attaches an image, the message payload transforms to an array structure:

```json
{
  "role": "user",
  "content": [
    {
      "type": "text",
      "text": "Describe this image."
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

## 4. ES5 Streaming Implementation Details

Modern apps use the `fetch` API and read stream blocks via `ReadableStream` chunks. Because iOS 9 Safari lacks `fetch` and `ReadableStream`, LegacyChat implements streaming purely using `XMLHttpRequest` status events (`readyState === 3` for loading):

```mermaid
sequenceDiagram
    participant UI as Browser UI
    participant XHR as XMLHttpRequest (readyState 3)
    participant OR as OpenRouter API

    UI->>OR: POST /chat/completions (stream: true)
    loop While receiving chunks
        OR-->>XHR: Send raw SSE stream text
        XHR->>UI: Trigger onreadystatechange
        Note over UI: Calculate newData = responseText.substring(seenLength)
        Note over UI: Split by "\n\n" and parse JSON line "data: {...}"
        UI->>UI: Append content delta to bubble HTML
    end
    OR-->>XHR: data: [DONE]
    XHR->>UI: state change (readyState 4)
    Note over UI: Store final chat thread in localStorage
```

### Chunk Processing Logic
The algorithm handles fragmented payloads by keeping a sliding buffer:
1. `seenLength` stores the index of text processed so far.
2. In `readyState 3` (LOADING) or `readyState 4` (DONE), it retrieves the newly arrived characters:
   ```javascript
   var newData = xhr.responseText.substring(seenLength);
   seenLength = xhr.responseText.length;
   buffer += newData;
   ```
3. The buffer splits on double newlines (`\n\n`), leaving any trailing, incomplete block in the buffer.
4. Each segment is stripped of the `data: ` prefix, parsed as JSON, and the content updates the bubble:
   ```javascript
   if (line.indexOf('data: ') === 0) {
     var jsonStr = line.substring(6);
     if (jsonStr !== '[DONE]') {
       var data = JSON.parse(jsonStr);
       var chunk = data.choices[0].delta.content;
       onChunk(chunk);
     }
   }
   ```

---

## 5. Error & Limits Strategy

| Error Condition | HTTP Status Code | Resolution Flow |
|---|---|---|
| **Unauthorized** | `401` | Displays inline warning: *"Unauthorized. Please check your API key."* |
| **Rate Limits** | `429` | Displays: *"Rate limit exceeded. Please wait a moment and try again."* |
| **Quota Limits** | `500+` or `402` | Renders the exact error message string returned by the provider. |
| **Timeout (60s)** | Timeout trigger | Triggers `xhr.ontimeout` to prompt network verification. |
| **Local Full** | `QuotaExceededError` | Catches browser storage bounds, alerting the user to run history maintenance. |
