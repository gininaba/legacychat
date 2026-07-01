# Developer Onboarding Guide

Welcome to LegacyChat! This onboarding guide is designed to help you set up the codebase locally, understand our development workflows, and master the technical constraints of building for legacy environments like iOS 9 Safari.

---

## 1. Quick-Start Local Setup

Since LegacyChat is a standalone client with zero bundlers or compiler steps, launching it is extremely straightforward.

### Prerequisites
- A modern web browser (for development) or a target device (e.g., iPad Mini 1st Gen running iOS 9.3.6) for testing.
- A local server runner (highly recommended to avoid `file://` CORS blockages when testing APIs).

### Step-by-Step Launch
1. **Clone the repository**:
   ```bash
   git clone https://github.com/gininaba/legacychat.git
   cd legacychat
   ```

2. **Start a local static server**:
   You can use python, node, or ruby to host `index.html` locally:
   - **Node.js (npx)**:
     ```bash
     npx http-server . -p 3000
     ```
   - **Python 3**:
     ```bash
     python3 -m http.server 3000
     ```

3. **Navigate to the address**:
   Open browser to `http://localhost:3000`.

---

## 2. Key Tech Stack & Compatibility Constraints

Developing for 2016-era WebKit browsers means you **cannot** use modern Javascript or CSS specs. Every modification must be validated against these pillars:

### Pure ES5 Syntax
- **No `let` or `const`**: Use `var` for all declarations.
- **No arrow functions**: Use `function() {}` syntax.
- **No template literals**: Use string concatenation (`"Hello " + name`).
- **No destructurings or rest/spread operators**: Use index-based selection or object property assignments.

### Legacy DOM APIs
- **No `fetch()`**: Use `XMLHttpRequest` for all HTTP requests (see [API.md](file:///Volumes/1TB%20Graphics%20SSD/Sansan_DO_NOT_TOUCH/Projects/legacychat/API.md)).
- **No native Promises, async/await**: Rely on standard async callbacks or custom callback triggers.
- **Canvas-based compression**: Large file payloads will crash the 5MB browser `localStorage` boundary. Any camera files must be rendered to Canvas first and converted to optimized standard JPEGs.

### Legacy Style/Layouts
- **No CSS Grid**: Use CSS Flexbox layouts.
- **Webkit Prefixes**: Include `-webkit-` prefixed values for Flexbox configurations (e.g., `-webkit-flex`, `-webkit-box-orient`, etc.) to support iOS 9 Safari.
- **No CSS Custom Variables**: Hardcode color and padding values or use theme classes applied directly to the `<body>` element.

---

## 3. Development & PR Workflow

1. **Create a feature branch**:
   Follow the naming format: `feat/feature-name` or `fix/bug-name`.
   ```bash
   git checkout -b feat/my-new-feature
   ```

2. **Local Testing**:
   Ensure you verify layout responsiveness in both light and dark themes. Test compatibility using iOS Simulator or directly on an older target device by running a tunnel program like `ngrok` to expose your localhost server:
   ```bash
   ngrok http 3000
   ```

3. **Submit a Pull Request**:
   - Keep pull requests focused on a single feature or bug fix.
   - Attach screenshot updates under the `Screenshots/` directory if your PR alters visual elements.

---

## 4. Beginner FAQ

### Why does the API key input keep resetting?
The API key is cached locally using `localStorage`. If your browser is in Private/Incognito mode or your storage permissions are blocked, settings will revert on refresh.

### Can I build or bundle this into multiple files?
No. The core value of this project is that it exists entirely as a single `index.html` file that users can load from any environment (e.g., download to local desktop, save on home screen) with zero installation steps. Do not add Webpack, Vite, or npm bundle dependencies.

### How do I troubleshoot Safari on iOS 9?
Connect your iPad to a macOS computer via USB. Open Safari on macOS, enable the developer menu (`Settings` -> `Advanced` -> `Show features for web developers`), and select your iPad Mini target under the `Develop` menu to open a remote web inspector panel.
