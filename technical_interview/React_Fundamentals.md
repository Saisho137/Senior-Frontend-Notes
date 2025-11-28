# React

## SSR vs CSR (Rendering Strategies)

### Core Concept

**Server-Side Rendering (SSR)**: The server executes a renderer (templates / `renderToString`) and **generates complete HTML** before sending it to the client. The browser receives that HTML and executes its **Critical Rendering Path** (parse → DOM/CSSOM → layout → paint).

**Client-Side Rendering (CSR)**: The server sends minimal HTML; the browser downloads and executes JavaScript that **generates the markup** (e.g., `data.map(...)`) and inserts nodes into the DOM.

> **Important**: The server does **not** perform layout/paint or execute the browser engine; it only produces HTML text ready for the browser to parse.

### What the Server "Pre-Renders"

The server produces:

- **HTML markup** (strings with tags)
- **Initial state** serialized (e.g., `window.__INITIAL_DATA__`) so the client doesn't need to re-fetch data

The server does **not** produce DOM or CSS/layout calculations — that's done by the browser.

### Flow Comparison

#### CSR Flow

1. Browser receives minimal HTML (`<div id="app"></div>`)
2. Downloads and executes JavaScript bundle
3. JS executes operations like `data.map(...)` → creates nodes (or virtual DOM) → updates DOM → paint
4. Visible content depends on JS bundle execution

#### SSR Flow

1. Server executes renderer and returns HTML with content (`<ul><li>...</li></ul>`)
2. Browser parses HTML → DOM/CSSOM → layout → paint: **content appears quickly**
3. Client JS arrives and **hydrates** (attaches handlers, synchronizes state) to add interactivity

### Initial State — What and Why

**Definition**: A serialized data object embedded in HTML that allows the client to reconstruct exactly the UI that the server already generated, without re-requesting data.

**Can be**:

- Complete store snapshot (Redux: `store.getState()`)
- Props or `pageProps` (Next.js)
- Initial Apollo/GraphQL cache
- Values received in `useState`

**Typical format**:

```html
<script>
  window.__INITIAL_DATA__ = { items: [...], user: {...} };
</script>
```

**Usage**: On the client, create store/state with that data to hydrate without additional fetch:

```javascript
const store = createStore(rootReducer, window.__INITIAL_DATA__);
hydrateRoot(root, <App store={store} />);
```

### Hydration and Duplicate Work

**Hydration**: The client bundle executes code that attaches event handlers and can reconcile the virtual DOM with the existing DOM.

- If server-side HTML matches what the client would render, the library avoids rewriting nodes and only attaches handlers
- If they don't match, it may replace nodes (diff + patch)
- Hydration can be CPU-intensive and block interactivity if the entire app hydrates at once

**Optimization techniques**:

- **Partial/Progressive Hydration**: Hydrate components incrementally
- **Islands Architecture**: Only hydrate interactive components
- **Streaming SSR**: Send HTML in chunks for incremental rendering

### Important Variants

**SSG (Static Site Generation)**: Render at build-time → static HTML

**SSR per-request**: Render on each request → dynamic, higher server load

**Streaming SSR**: Send HTML in chunks for incremental client rendering

**Islands / Partial Hydration**: Only hydrate interactive components

### Best Practices

1. **Include critical CSS** in `<head>` or inline (avoid FOUC - Flash of Unstyled Content)
2. **Load scripts with `defer`** when possible
3. **Serialize only necessary data** (don't expose secrets); compress/sanitize JSON
4. **Use caching** (CDN / edge) to reduce server load in per-request SSR
5. **Consider hydration cost reduction** strategies (hydration on interaction, partial hydration)

### Minimal Examples

#### Server (React + Node) — Produces HTML + Initial State

```javascript
// server.js
const html = renderToString(<App initialData={data} />);
const initialState = { items: data };

res.send(`
  <!doctype html>
  <html>
    <body>
      <div id="root">${html}</div>
      <script>
        window.__INITIAL_DATA__ = ${JSON.stringify(initialState)};
      </script>
      <script src="/client.js" defer></script>
    </body>
  </html>
`);
```

#### Client (Hydration)

```javascript
// client.js
const initial = window.__INITIAL_DATA__;
const store = createStore(rootReducer, initial);

hydrateRoot(
  document.getElementById('root'), 
  <App store={store} />
);
```

#### Pure CSR (Client Generates Items)

```html
<!-- index.html -->
<div id="app"></div>
<script src="bundle.js" defer></script>
```

```javascript
// bundle.js
fetch('/api/items')
  .then(r => r.json())
  .then(data => {
    document.getElementById('app').innerHTML = `
      <ul>
        ${data.map(item => `<li>${item.name}</li>`).join('')}
      </ul>
    `;
  });
```

### Key Points Summary

| Aspect | SSR | CSR |
|--------|-----|-----|
| **HTML Generation** | Server generates complete HTML | Client JS generates markup at runtime |
| **Initial Load** | Content visible immediately | Blank until JS executes |
| **SEO** | Better (crawlers see content) | Requires pre-rendering or dynamic rendering |
| **FCP/LCP** | Faster | Slower |
| **TTI** | Slower (needs hydration) | Can be faster (no hydration) |
| **Server Load** | Higher | Lower |
| **Use Cases** | Public sites, blogs, e-commerce | Dashboards, SPAs, authenticated apps |

**Critical Understanding**:

- SSR moves **markup generation** to the server (improving FCP/SEO)
- The browser is still responsible for **visual rendering** (parse → layout → paint)
- **Initial state** is the piece that synchronizes server and client to avoid refetches and visual flashes
- Hydration is necessary for interactivity but can be optimized to avoid unnecessary work
