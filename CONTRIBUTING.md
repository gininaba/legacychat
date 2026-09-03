# Contributing to LegacyChat

Thank you for helping improve LegacyChat! Contributions from the community ensure this application remains performant, feature-rich, and compatible with legacy WebKit browsers.

Please review these guidelines before submitting issue reports or opening Pull Requests.

---

## 1. Core Principles & Architecture Rules

1. **Strict Single-File Constraint**: LegacyChat is, and must remain, a single self-contained [`index.html`](file:///Volumes/1TB%20Graphics%20SSD/Sansan_DO_NOT_TOUCH/Projects/legacychat/index.html) file. Do not introduce package managers (npm dependencies), bundlers (Webpack, Vite), preprocessors (Sass, LESS), or transpilers (Babel).
2. **Strict ES5 Compatibility**: Target environment is Safari 9.3.6 on iPad Mini 1st Gen. Do not use ES6+ features (`const`, `let`, `=>`, `async`/`await`, `fetch`).
3. **Low CPU Overhead**: Optimizations must respect legacy dual-core hardware (Apple A5 processor). Always buffer streaming DOM updates (e.g. 60ms throttle timers).

---

## 2. Issue Reporting & Feature Requests

### Reporting Bugs
Before opening an issue:
1. Search active and closed issues to avoid duplicate reports.
2. If new, submit a detailed bug report specifying:
   - Target device and OS version (e.g., iPad Mini 1st Gen on iOS 9.3.6, iPhone 5c on iOS 10.3.3).
   - Exact steps to reproduce the failure.
   - Remote Web Inspector console output or error tracebacks (if available via USB debugging).

### Suggesting Features
When proposing new capabilities:
1. Explain the practical value of the feature for legacy tablet users.
2. Verify that the feature can be implemented using pure ES5 JavaScript and CSS flexbox.
3. Assess the impact on browser `localStorage` bounds (5MB quota limit).

---

## 3. Code Style & Quality Standards

- **Variable Declarations**: Always use `var`. Never use `let` or `const`.
- **Function Syntax**: Declare standard functions (`function name() {}`). Do not use arrow functions.
- **DOM Traversal**: Use safe element tree traversal functions like `findParentRow(element)` instead of hardcoded `parentNode` chains.
- **Vendor Prefixes**: Include `-webkit-` prefixes for Flexbox and visual filters (`-webkit-backdrop-filter`).
- **Sanitization**: Ensure all dynamic text content rendered into bubbles is processed via `escapeHTML()` and `sanitizeHTML()`.

---

## 4. Pull Request Checklist

When submitting a Pull Request:

- [ ] PR targets the `main` branch.
- [ ] Code is verified strictly inside `index.html`.
- [ ] Confirmed zero ES6 keywords (`const`, `let`, `=>`, `async`, `await`, `fetch`) in `<script>`.
- [ ] Verified visual layout responsiveness across Light and Dark themes.
- [ ] Updated corresponding documentation in [`COMPONENTS.md`](file:///Volumes/1TB%20Graphics%20SSD/Sansan_DO_NOT_TOUCH/Projects/legacychat/COMPONENTS.md) and [`CHANGELOG.md`](file:///Volumes/1TB%20Graphics%20SSD/Sansan_DO_NOT_TOUCH/Projects/legacychat/CHANGELOG.md).
- [ ] Included descriptive commit messages (e.g., `feat: add markdown strikethrough support` or `fix: resolve canvas upload ratio bug`).
