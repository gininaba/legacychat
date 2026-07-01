# Contributing to LegacyChat

Thank you for your interest in improving LegacyChat! Contributions from the community help make this project more robust, feature-rich, and compatible.

This document outlines the guidelines and policies for contributing to the repository. Please review them before submitting changes.

---

## 1. Ground Rules

- **Strict Client-Side / Single-File Constraint**: LegacyChat is, and must remain, a single self-contained `index.html` file. Do not introduce preprocessors, compilers (Babel, Sass), package managers (npm dependencies), or modular JavaScript builders.
- **Backwards Compatibility First**: The core objective of this project is compatibility with iOS 9.3.6 Safari on iPad Mini 1st Gen. Any new styling or logic must not break execution in older browser versions.

---

## 2. Reporting Issues & Requesting Features

### Reporting Bugs
If you find compatibility errors or runtime crashes:
1. Search active and closed issues first to see if it has been discussed.
2. If it is new, open a bug report issue.
3. Be sure to specify:
   - Target device and OS version (e.g., iPhone 5c on iOS 10.3.3, iPad 2 on iOS 9.3.5).
   - Steps to reproduce the error.
   - Any inspector logs or browser console output (if available via USB debugging).

### Requesting Features
If you want to suggest enhancements:
1. Explain the value of the feature and how it can be adapted into the ES5 environment.
2. Outline how it impacts the browser's storage and performance limits (such as `localStorage` 5MB quota).

---

## 3. Local Development Guidelines

- **Style Compliance**: Check your code structure. Avoid all ES6+ constructs. Double-check variable declarations (`var` only).
- **CSS Formatting**: Match existing aesthetic conventions:
  - Clean flexbox positioning.
  - Proper vendor prefixes (`-webkit-`).
  - Native iOS-like look and feel.
- **Testing**:
  - Load the modified `index.html` locally using a server.
  - Verify that the layout remains responsive across desktop, iPad, and iPhone resolutions.
  - Test transitions, inputs, custom markdown rendering, image compression canvas, and storage limits.

---

## 4. Pull Request Requirements

When submitting a Pull Request:
1. Ensure your PR branch is updated against the `main` branch.
2. Clearly describe what changes you've made and what issues they solve.
3. If visual edits are made, update the matching media inside the `Screenshots/` folder and include preview links in your PR description.
4. Keep commit messages concise and descriptive (e.g. `feat: implement dark mode adjustments` or `fix: resolve canvas upload sizing bug`).
