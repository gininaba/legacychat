# Developer Onboarding Guide

Welcome to LegacyChat! This guide is designed to help software engineers set up the repository locally, master legacy WebKit compatibility requirements, and debug applications targeting iOS 9.3.6 Safari on iPad Mini 1st Gen.

---

## 1. Quick-Start Local Launch

Because LegacyChat is a standalone client with zero build steps or npm compilers, launching it takes seconds:

### Prerequisites
- A modern web browser (for code editing) or a target device (e.g., iPad Mini 1st Gen running iOS 9.3.6) for testing.
- A local static web server runner (to prevent local file CORS restrictions during API calls).

### Launch Steps
1. **Clone the repository**:
   ```bash
   git clone https://github.com/gininaba/legacychat.git
   cd legacychat
   ```

2. **Start a local static web server**:
   - **Node.js**:
     ```bash
     npx http-server . -p 3000
     ```
   - **Python 3**:
     ```bash
     python3 -m http.server 3000
     ```

3. **Open browser**:
   Navigate to `http://localhost:3000`.

---

## 2. Mandatory ES5 Syntax & API Matrix

Developing for 2016-era WebKit engines requires strict adherence to legacy standards. Never use ES6+ features in `index.html`:

| Category | ❌ Forbidden (ES6+) | ✅ Mandatory (ES5 Standard) |
|---|---|---|
| **Variables** | `let x = 10;`<br>`const y = 20;` | `var x = 10;`<br>`var y = 20;` |
| **Functions** | `const add = (a, b) => a + b;` | `function add(a, b) { return a + b; }` |
| **Strings** | ``var text = `Hello ${name}`;`` | `var text = "Hello " + name;` |
| **HTTP Requests** | `fetch(url).then(res => res.json())` | `var xhr = new XMLHttpRequest(); ...` |
| **Async Logic** | `async function()` / `await` | Standard asynchronous callback functions |
| **Objects** | `{ a, b }` (Short-hand) | `{ a: a, b: b }` (Explicit key-value) |
| **Arrays / Rest** | `[...items, newItem]` | `items.concat([newItem])` or `items.push(newItem)` |
| **DOM Slicing** | `Array.from(nodes)` | `Array.prototype.slice.call(nodes)` |

---

## 3. Styling & Glassmorphism Guidelines

1. **Vendor Prefixes**: Always include `-webkit-` prefixes for CSS flexbox and visual filters:
   ```css
   .my-container {
     display: -webkit-box;
     display: -webkit-flex;
     display: flex;
     -webkit-box-orient: vertical;
     -webkit-box-direction: normal;
     -webkit-flex-direction: column;
     flex-direction: column;
   }
   ```
2. **Progressive Glassmorphism**: Use backdrop blurs with clear translucent fallback backgrounds so engines without backdrop support render gracefully:
   ```css
   .nav-bar {
     background: rgba(249, 249, 249, 0.88);
     -webkit-backdrop-filter: blur(20px);
     backdrop-filter: blur(20px);
   }
   ```
3. **No CSS Custom Properties**: Do not use CSS variables (`var(--primary-color)`). Hardcode values or use theme classes on `document.body` (`body.dark-mode`).

---

## 4. USB Remote Debugging on iOS 9 Safari

To inspect DOM states or debug JavaScript execution on a physical iPad Mini:

1. **Enable Web Inspector on iPad**:
   - Open **Settings** → **Safari** → **Advanced**.
   - Toggle **Web Inspector** to **ON**.

2. **Enable Developer Menu on macOS Safari**:
   - Open Safari on macOS.
   - Go to **Settings** → **Advanced** → Check **"Show features for web developers"**.

3. **Connect iPad via USB**:
   - Connect iPad Mini to Mac using a Lightning-to-USB cable.
   - Open LegacyChat in Safari on the iPad.
   - On macOS Safari, open the **Develop** menu, select your iPad device, and click `index.html` to open the Web Inspector console.

4. **Expose Localhost Server via Tunnel**:
   - To access your local `localhost:3000` server on the physical iPad Mini, run a tunnel:
     ```bash
     npx ngrok http 3000
     ```
   - Open the generated `https://xxxx.ngrok-free.app` URL in iPad Safari.

---

## 5. Development & PR Checklist

When making contributions:

1. **Verify Syntax**: Run an ES5 keyword scan to confirm no `const`, `let`, or `=>` syntax exists in `<script>`:
   ```bash
   node -e "const code = require('fs').readFileSync('index.html','utf8'); ['const ','let ','=>','async ','await '].forEach(k => console.log(k.trim(), code.split(k).length-1));"
   ```
2. **Test Responsive Layouts**: Test in both Light and Dark modes.
3. **Update Documentation**: Update [`COMPONENTS.md`](file:///Volumes/1TB%20Graphics%20SSD/Sansan_DO_NOT_TOUCH/Projects/legacychat/COMPONENTS.md) or [`API.md`](file:///Volumes/1TB%20Graphics%20SSD/Sansan_DO_NOT_TOUCH/Projects/legacychat/API.md) if adding new global state variables or functions.
