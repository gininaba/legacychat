# Changelog

All notable changes to the LegacyChat project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.3.0] - 2026-06-30
### Added
- **Real-time Model Search**: Users can now filter models by typing in the search bar under settings.
- **HTML Sanitization**: Introduced a lightweight HTML purifier that strips dangerous elements (`<script>`, `<iframe>`, link/style tags) and DOM event handlers to protect against XSS injection.
- **Storage Quota Management**: Gracefully handles `QuotaExceededError` limitations by prompting the user and cleaning up older images from the message history to free up `localStorage`.
- **Predefined Personas**: Select pre-configured personas directly from settings (Helpful Assistant, Coding Assistant, Translator) or save your own custom personas persistently.

### Changed
- Refined UI styling with improved CSS compatibility for older WebKit engines.
- Updated project screenshots and README references to match the newer interface layouts.

---

## [1.2.0] - 2026-06-15
### Added
- **Image Upload Compression**: Large photos uploaded from a camera roll are now dynamically compressed using a hidden `<canvas>` element (scaled to max 800x800, JPEG quality 0.6) before conversion to Base64.
- **Streaming Completeness**: Stable character-by-character text streaming using pure `XMLHttpRequest` chunk processing on state changes.

### Changed
- Improved error messages for network timeouts (60 seconds threshold) and OpenRouter API errors.

---

## [1.1.0] - 2026-06-01
### Added
- **Per-Message Actions**:
  - **Edit**: Allows users to rewrite their previous messages, which truncates subsequent thread history to spawn a clean rewrite.
  - **Regenerate**: Truncates from that point in history and queries the model again.
  - **Delete**: Remove single messages from both user interface bubbles and history array.
  - **Copy**: Copy AI text to browser clipboard with a dynamic "Copied!" button feedback state.
- **History Search Bar**: Instantly filter and search past chat histories by title.
- **Stop Generation**: Dedicated abort button that utilizes `xhr.abort()` to immediately halt incoming AI response chunks.
- **Session Token Tracking**: Keep track of aggregate token usage inside the navigation bar dynamically updated on every response.
- **Dynamic Model Selection**: Support for pulling available models list from OpenRouter dynamically and updating select options.

---

## [1.0.0] - 2026-05-10
### Added
- **Initial Release**: Complete, standalone single-file chat app targeting older iPad and iPhone devices (specifically running Safari on iOS 9.3.6).
- **ES5 Compatibility**: Strict ES5 JavaScript syntax with no ES6 features (such as `let`, `const`, arrow functions, `fetch()`, or `async`/`await`).
- **PWA Capabilities**: Out-of-the-box support for installation on home screen with custom web app configuration, icon, and full-screen standalone window.
- **MIT License**: Included terms of use and permission structure.
