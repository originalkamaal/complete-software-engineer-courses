---
title: "React and Next.js from Zero to Advanced"
subtitle: "How React really renders — from JSX and hooks to concurrent rendering, the React Compiler, server components, and a tested full-stack Next.js application"
lang: en
---

# Preface

React is easy to start and easy to misuse. Most React bugs come from a mental model that is almost right: that a component is a template rather than a function React calls whenever it likes; that `setState` changes a variable rather than scheduling a new render; that `useEffect` is a lifecycle method rather than a way to synchronise with something outside React; that `useMemo` is a performance switch rather than a cache with rules; that a server component is a component that "runs on the server" in the same way a client component runs in the browser. Almost-right models produce code that works in the demo and fails under real data, real users, and real networks.

This book builds the accurate model. It starts with what JSX compiles to and what an element is — and has you write a renderer for React elements in Chapter 0 — then covers components, props, and state; every hook in depth; effects and data fetching; forms; context, reducers, and external stores; performance, reconciliation, and the React Compiler; routing; server state with TanStack Query; TypeScript; styling; and testing. The last third is Next.js: the App Router, server components, server actions, caching, and streaming; authentication, databases, and deployment; accessibility and internationalisation; and a capstone full-stack application.

The book assumes the knowledge in *HTML, CSS, and JavaScript from Zero to Advanced* — the DOM, the event loop, modules, TypeScript, HTTP — and refers to it where React builds on it.

Three promises shape every page:

1. **Precision.** Each *Deep & Tricky* fact is labelled with its source of truth: the React documentation (react.dev), the React source code, the Next.js documentation, or a specification.
2. **Evidence.** Every example shown with output was run — with React 19.3, Next.js 16.4, Vite 8, and Node.js 22, pages in a real headless Chromium — and the output is real.
3. **Order.** Nothing is used before it is taught; later chapters refer back by unit number.

# How to use this book

## The shape of a unit

| Part | What it gives you |
| --- | --- |
| **One-line summary** | The whole idea in one sentence. |
| **Lesson** | Small complete programs and their real output. |
| **Deep & Tricky (DT-n)** | Precise facts that surprise experienced developers, labelled with their source of truth. |
| **Patterns (P-n)** | Engineering practices with their trade-offs. |
| **Mastery check** | *Predict* questions; answers at the end of each chapter. |

Each chapter ends with a **Chapter Project** with complete code, real output, and a review checklist.

## Running the code

All chapters share one `package.json` and `node_modules` at the top of the code folder; Chapter 0 creates them. Each chapter's folder has a `vite.config.js`, and every subfolder with an `index.html` is a page served by Vite at `/subfolder/`. Programs that only render to strings run directly in Node.js with `tsx`, which compiles JSX on the fly. Next.js projects (Chapters 12–15) have their own folders.

To show what a page does in print, pages are run in headless Chromium by a small driver, `tools/page.py` — the same one as in the HTML, CSS, and JavaScript book, which prints the page's console output, the results of expressions (`=> …`), and accessibility trees. It hides messages from development tooling (Vite's connection notices and React's DevTools hint). From Chapter 14 it also reports keyboard focus and runs the axe-core accessibility checker, and can set the browser's time zone and language. When you read, open the same pages in your browser with the developer tools open.

```python
# file: ../tools/page.py
"""Drive a real browser (headless Chromium, through Playwright) from the command line.

    python3 page.py TARGET [STEP ...] [--serve CMD [--server-log]] [--width W] [--height H]
                    [--dpr N] [--media FEATURE] [--slow] [--tz ZONE] [--locale TAG]

TARGET is a file (opened as file://) or a URL. Console messages and uncaught errors
are printed as they happen. STEPs run in order:
    eval:EXPR       evaluate a JavaScript expression and print "=> result" (JSON)
    click:SEL       click the element matching a CSS selector
    hover:SEL       move the mouse over it
    mouse:X,Y       click at a position (no waiting for the page, unlike click:)
    fill:SEL|TEXT   type TEXT into an input
    upload:SEL|FILE choose FILE in a file input
    select:SEL|VAL  choose an option of a <select>
    press:KEY       press a key (e.g. Enter, Tab)
    wait:MS         wait MS milliseconds
    viewport:WxH    resize the viewport
    media:dark      emulate prefers-color-scheme: dark (or light, reduce for reduced motion)
    offline:on|off  cut or restore the browser's network connection
    reload          reload the page
    shot:FILE       save a screenshot
    aria:SEL        print the accessibility tree of an element, as the browser exposes it
    focus           print the element that has keyboard focus (tag, id, text)
    axe             run the axe-core accessibility checker and print each rule violated
--dpr N sets the device pixel ratio (default 1); --media dark|light|reduce emulates
a media feature from the start; --slow emulates a slow network (400 ms latency, 400 kB/s)
and a 4x slower CPU. --tz and --locale set the browser's time zone (e.g. Asia/Tokyo) and
language (e.g. de-DE; also sent as Accept-Language).
--serve CMD starts a server first (e.g. "node server.js"), waits until TARGET answers,
and stops it at the end; the server's own output is shown only with --server-log.
Messages from development tooling (Vite's connection notices, React's DevTools hint) are hidden.
"""
import json
import os
import re
import shlex
import subprocess
import sys
import time
import urllib.request
from playwright.sync_api import sync_playwright

# Development-server chatter that says nothing about the page itself
NOISE = re.compile(r"^\[vite\] (connecting|connected)|Download the React DevTools")
# The axe-core accessibility checker, installed with the book's packages
AXE = os.path.join(os.path.dirname(os.path.abspath(__file__)), "..", "node_modules", "axe-core", "axe.min.js")

def main():
    args, steps, serve, width, height, dpr, server_log = sys.argv[1:], [], None, 800, 600, 1, False
    media, slow, tz, locale = [], False, None, None
    target = args.pop(0)
    while args:
        a = args.pop(0)
        if a == "--serve": serve = args.pop(0)
        elif a == "--width": width = int(args.pop(0))
        elif a == "--height": height = int(args.pop(0))
        elif a == "--dpr": dpr = float(args.pop(0))
        elif a == "--server-log": server_log = True
        elif a == "--media": media.append(args.pop(0))
        elif a == "--slow": slow = True
        elif a == "--tz": tz = args.pop(0)
        elif a == "--locale": locale = args.pop(0)
        else: steps.append(a)
    server = None
    if serve:
        server = subprocess.Popen(shlex.split(serve), start_new_session=True,
                                  stdout=None if server_log else subprocess.DEVNULL,
                                  stderr=None if server_log else subprocess.DEVNULL)
        for _ in range(100):                                  # wait up to 10 s for the server
            try:
                urllib.request.urlopen(urllib.request.Request(target, method="HEAD"), timeout=1); break
            except urllib.error.HTTPError:
                break                                         # it answered, even if with an error
            except Exception:
                time.sleep(0.1)
    if "://" not in target:
        target = "file://" + os.path.abspath(target)
    try:
        with sync_playwright() as p:
            browser = p.chromium.launch()
            page = browser.new_page(viewport={"width": width, "height": height}, device_scale_factor=dpr,
                                    timezone_id=tz, locale=locale)
            if slow:
                cdp = page.context.new_cdp_session(page)
                cdp.send("Network.emulateNetworkConditions", {"offline": False, "latency": 400,
                         "downloadThroughput": 400_000, "uploadThroughput": 400_000})
                cdp.send("Emulation.setCPUThrottlingRate", {"rate": 4})
            for m in media:
                page.emulate_media(**({"reduced_motion": "reduce"} if m == "reduce" else {"color_scheme": m}))
            page.on("console", lambda m: NOISE.search(m.text) or print(m.text, flush=True))
            page.on("pageerror", lambda e: print(f"Uncaught {e.name}: {e.message}", flush=True))
            page.goto(target)
            for step in steps:
                kind, _, arg = step.partition(":")
                if kind == "eval":
                    print("=>", json.dumps(page.evaluate(arg), ensure_ascii=False), flush=True)
                elif kind == "click": page.click(arg)
                elif kind == "hover": page.hover(arg)
                elif kind == "mouse":
                    x, y = map(float, arg.split(","))
                    page.mouse.click(x, y)
                elif kind == "fill":
                    sel, _, text = arg.partition("|")
                    page.fill(sel, text)
                elif kind == "select":
                    sel, _, val = arg.partition("|")
                    page.select_option(sel, val)
                elif kind == "upload":
                    sel, _, file = arg.partition("|")
                    page.set_input_files(sel, file)
                elif kind == "press": page.keyboard.press(arg)
                elif kind == "wait": page.wait_for_timeout(int(arg))
                elif kind == "viewport":
                    w, h = map(int, arg.split("x"))
                    page.set_viewport_size({"width": w, "height": h})
                elif kind == "media":
                    if arg == "reduce": page.emulate_media(reduced_motion="reduce")
                    else: page.emulate_media(color_scheme=arg)
                elif kind == "offline": page.context.set_offline(arg == "on")
                elif kind == "reload": page.reload()
                elif kind == "shot": page.screenshot(path=arg)
                elif kind == "aria": print(page.locator(arg).first.aria_snapshot(), flush=True)
                elif kind == "focus":
                    print("focus:", page.evaluate("""() => { const e = document.activeElement;
                        return e ? `${e.tagName.toLowerCase()}${e.id ? '#' + e.id : ''} "${(e.textContent || e.value || '').trim().slice(0, 40)}"` : 'none' }"""), flush=True)
                elif kind == "axe":
                    page.add_script_tag(path=AXE)
                    found = page.evaluate("""async () => (await axe.run()).violations.map((v) =>
                        `${v.id} (${v.impact}): ${v.help} — ${v.nodes.map((n) => n.target.join(' ')).join(', ')}`)""")
                    print(f"axe: {len(found)} violation(s)", *found, sep="\n  ", flush=True)
                else: sys.exit(f"unknown step {step!r}")
            page.wait_for_timeout(50)                         # let late console messages arrive
            browser.close()
    finally:
        if server:
            os.killpg(server.pid, 15)

if __name__ == "__main__":
    main()
```

# Chapter 0 — Why React: JSX and the Rendering Model

*Goal: understand what React is before using it. React is a library for describing user interfaces as a function of state: you write components — functions that return descriptions of what to show — and React works out how to make the screen match. This chapter sets up the tools, shows what JSX compiles to, shows that React elements are plain objects and that components are called by React rather than by you, renders to HTML on the server, starts a first interactive app, and ends with you writing a renderer for React elements whose output is identical to React's own.*

**What you will be able to do after this chapter**

- Set up a React project with Vite, and run JSX in Node.js.
- Explain what JSX compiles to, and its rules: expressions, attributes, conditional rendering, lists, keys.
- Explain the difference between an element, a component, and a rendered DOM node.
- Render React on the server to a string, and on the client into the DOM.
- Explain why React calls components twice in development.

## Topic 0A — The idea

## Unit 0.1 — UI as a function of state

**One-line summary:** In the DOM APIs of the previous book, the program changes the page step by step (*imperative*): find the element, set its text, add a class; React lets you write what the page *should look like* for a given state (*declarative*) and does the step-by-step updates itself — so the code that describes the UI is a function of the data, and the bugs where the page and the data disagree largely disappear.

| Imperative (DOM) | Declarative (React) |
| --- | --- |
| `count++; span.textContent = count; button.disabled = count >= 10;` | `<span>{count}</span><button disabled={count >= 10}>` |
| Every change must update every affected place | Describe the result once; React updates what changed |
| Order of updates matters | Only the current state matters |

The price is a model you must understand: React decides *when* your component functions run, and expects them to be pure — same props and state, same output, no side effects. This chapter and the next three are about that model.

## Unit 0.2 — Setting up

**One-line summary:** A React project needs `react` (the core: elements, components, hooks), `react-dom` (rendering to the browser DOM and to HTML on the server), and a tool that compiles JSX — Vite for the browser, `tsx` for scripts in Node.js; all chapters of this book share one installation.

```console
$ cd .. && npm init -y > /dev/null && npm pkg set type=module && npm pkg set private=true --json && npm install react@19 react-dom@19 2>&1 | tail -1
found 0 vulnerabilities
$ cd .. && npm install --save-dev vite@8 @vitejs/plugin-react@6 vitest@5 jsdom @testing-library/react @testing-library/user-event @testing-library/dom tsx typescript@7 @types/react @types/react-dom @playwright/test@1.63 2>&1 | tail -1
found 0 vulnerabilities
$ cd .. && echo "{ \"compilerOptions\": { \"jsx\": \"react-jsx\", \"allowJs\": true, \"strict\": true, \"noEmit\": true, \"target\": \"es2024\", \"module\": \"nodenext\", \"skipLibCheck\": true } }" > tsconfig.json && npm ls --depth=0 | grep -E "react|vite|tsx"
code@1.0.0 /home/claude/books/react/code
├── @testing-library/react@16.3.3
├── @types/react-dom@19.3.0
├── @types/react@19.3.0
├── @vitejs/plugin-react@6.3.0
├── react-dom@19.3.1
├── react@19.3.1
├── tsx@4.23.15
├── vite@8.3.8
└── vitest@5.1.0
```

```js
// file: vite.config.js
// One Vite configuration per chapter: every folder with an index.html is a page at /folder/.
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  logLevel: "warn",
  // Pre-bundle React when the server starts. Otherwise Vite discovers it on the first page load,
  // bundles it, and reloads the page, so everything in it runs twice.
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client"] },
});
```

## Topic 0B — JSX and elements

## Unit 0.3 — What JSX compiles to

**One-line summary:** JSX is syntax for function calls: `<h2>{title}</h2>` becomes `jsx("h2", { children: title })`, a lowercase tag becomes a string type and a capitalised one a reference to a component, attributes become a props object (`class` is `className`, `for` is `htmlFor`, events are camel-cased functions), and anything inside braces is an ordinary JavaScript expression.

```jsx
// file: jsx/card.jsx
export function Card({ title, tags, onOpen }) {
  return (
    <article className="card">
      <h2>{title}</h2>
      {tags.length > 0 && <ul>{tags.map((t) => <li key={t}>{t}</li>)}</ul>}
      <button onClick={onOpen}>Open</button>
    </article>
  );
}
```

```console
$ npx esbuild jsx/card.jsx --jsx=automatic --format=esm
import { jsx, jsxs } from "react/jsx-runtime";
function Card({ title, tags, onOpen }) {
  return /* @__PURE__ */ jsxs("article", { className: "card", children: [
    /* @__PURE__ */ jsx("h2", { children: title }),
    tags.length > 0 && /* @__PURE__ */ jsx("ul", { children: tags.map((t) => /* @__PURE__ */ jsx("li", { children: t }, t)) }),
    /* @__PURE__ */ jsx("button", { onClick: onOpen, children: "Open" })
  ] });
}
export {
  Card
};
```

The "automatic" JSX runtime imports `jsx` and `jsxs` (for static lists of children) from `react/jsx-runtime`. `{tags.length > 0 && …}` is plain JavaScript: when the condition is false the expression is `false`, which renders nothing. The `key` is passed separately from the props — React uses it to match list items between renders (Chapter 1).

## Unit 0.4 — Elements are data; components are called by React

**One-line summary:** Evaluating JSX produces a React *element* — a small immutable object `{ type, key, props }` that describes what to show — and nothing more: a component in the tree is not called when the element is created, but later, by React, whenever React decides to render.

```jsx
// file: jsx/elements.jsx
// A React element is a plain, immutable object that DESCRIBES what to show. Nothing is rendered yet.
import { renderToString } from "react-dom/server";
import { Card } from "./card.jsx";

const el = <Card title="Hello" tags={["new"]} onOpen={() => {}} />;
console.log("type is the component function itself:", el.type === Card);
console.log("props:", Object.keys(el.props), "| key:", el.key, "| frozen:", Object.isFrozen(el));

let calls = 0;
function Counted() { calls++; return <p>counted</p>; }
const tree = <div><Counted /><Counted /></div>;
console.log("after creating the tree, Counted has been called", calls, "times");   // React calls it, later

const user = { name: "Ana" };
console.log(<p>{user.name}</p>.props.children, "|", <p>{"<b>not bold</b>"}</p>.props.children);
try { renderToString(<p>{user}</p>); } catch (e) { console.log(e.message.split(". ")[0]); }
```

```console
$ npx tsx jsx/elements.jsx
type is the component function itself: true
props: [ 'title', 'tags', 'onOpen' ] | key: null | frozen: true
after creating the tree, Counted has been called 0 times
Ana | <b>not bold</b>
Objects are not valid as a React child (found: object with keys {name})
```

The element for `<Card …>` holds a reference to the `Card` function and the props; `Counted` was not called at all. Strings are children too, and they stay strings — `"<b>not bold</b>"` will be escaped, never parsed. A plain object is not a valid child; React says so when it renders.

## Unit 0.5 — Rendering

**One-line summary:** Rendering is React calling components until only host elements (HTML tags) and text are left, and then producing output: `react-dom/server` produces an HTML string, and `react-dom/client` creates or updates DOM nodes; all text and attribute values are escaped, so JSX is safe from the HTML-injection bugs of `innerHTML`.

```jsx
// file: jsx/render.jsx
// Rendering turns elements into output. On the server: a string of HTML.
import { renderToString } from "react-dom/server";
import { Card } from "./card.jsx";

const xss = '<img src=x onerror="alert(1)">';
console.log(renderToString(<Card title={xss} tags={["new", "sale"]} />));
console.log(renderToString(<Card title="No tags" tags={[]} />));
console.log(renderToString(<ul>{[0, "", null, undefined, false, true, NaN].map((v, i) => <li key={i}>{v}</li>)}</ul>));
```

```console
$ npx tsx jsx/render.jsx
<article class="card"><h2>&lt;img src=x onerror=&quot;alert(1)&quot;&gt;</h2><ul><li>new</li><li>sale</li></ul><button>Open</button></article>
<article class="card"><h2>No tags</h2><button>Open</button></article>
Received NaN for the `children` attribute. If this is expected, cast the value to a string.
<ul><li>0</li><li></li><li></li><li></li><li></li><li></li><li>NaN</li></ul>
```

The would-be XSS title is escaped. The empty `tags` list rendered no `<ul>`. The last line shows which values render: `0` and `NaN` render as text, while `""`, `null`, `undefined`, `false`, and `true` render nothing (and React warns about `NaN`).

### Deep & Tricky

**DT-1. `{count && <Badge />}` renders a literal `0` when `count` is 0, because `&&` returns its left operand when it is falsy and React renders numbers — a classic source of stray zeros in lists and layouts (and a crash in React Native, where text outside `<Text>` is not allowed). Use `count > 0 && …` or a ternary.** *(React documentation, "Conditional Rendering — Pitfall: Don't put numbers on the left side of &&")*

## Unit 0.6 — A first interactive app

**One-line summary:** In the browser, `createRoot(container).render(element)` mounts a tree; when a component's state changes (`useState`'s setter), React calls the component again, compares the new elements with the previous ones, and updates only the DOM that changed — and in development, `<StrictMode>` calls every component twice to expose components that are not pure.

```html
<!-- file: first/index.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>A first React app</title></head>
<body>
  <div id="root"></div>
  <script type="module" src="./main.jsx"></script>
</body>
</html>
```

```jsx
// file: first/main.jsx
import { StrictMode, useState } from "react";
import { createRoot } from "react-dom/client";

function Counter() {
  const [count, setCount] = useState(0);              // state: React remembers it between renders
  console.log(`Counter renders with count = ${count}`);
  return (
    <p>
      Clicked {count} {count === 1 ? "time" : "times"}{" "}
      <button onClick={() => setCount(count + 1)}>Click</button>
    </p>
  );
}

const root = createRoot(document.getElementById("root"));
root.render(<StrictMode><Counter /></StrictMode>);   // StrictMode: extra checks in development
```

```console
$ python3 ../tools/page.py http://localhost:5173/first/ click:button click:button "eval:document.querySelector('p').textContent" --serve "npx vite --port 5173 --strictPort"
Counter renders with count = 0
Counter renders with count = 0
Counter renders with count = 1
Counter renders with count = 1
Counter renders with count = 2
Counter renders with count = 2
=> "Clicked 2 times Click"
```

Each render is logged twice: StrictMode renders each component twice in development and throws one result away, so that a component whose output depends on something other than its props and state — a global counter, `Math.random()`, a mutation — shows its problem immediately. In production builds, components render once.

### Deep & Tricky

**DT-2. In development, `<StrictMode>` intentionally calls component functions, initialisers of `useState`/`useMemo`/`useReducer`, and the updater functions passed to setters twice, and mounts every component twice (mount, simulated unmount, mount) to check that effects clean up after themselves. Logs, counters, or network requests that appear twice in development and once in production are this check working, not a bug in React — and code that breaks under it would also break when React re-renders, reuses, or restores components in ways StrictMode simulates.** *(React documentation, "<StrictMode>": "Your components will re-render an extra time to find bugs caused by impure rendering… re-run Effects an extra time to find bugs caused by missing Effect cleanup")*

The first version of this book's Vite configuration also logged everything *four* times on the first load: Vite discovered React on the first request, pre-bundled it, and reloaded the page. The `optimizeDeps.include` setting in `vite.config.js` pre-bundles it at start-up instead.

### Mastery check

1. **Predict.** What does `<p>{["a", "b"]}</p>` render? And `<p>{{}}</p>`?
2. **Explain.** Why does React need components to be pure?

## Chapter 0 Project — A renderer for React elements

### Requirements

Write two renderers for React elements, without using `react-dom`: `renderToHtml(element)`, which produces the same HTML as React's `renderToString` for a tree with components, fragments, lists, conditionals, `className`, `htmlFor`, `style` objects, boolean attributes, void elements, and event handlers (ignored on the server); and `mount(element, container)`, which creates real DOM nodes with event listeners. Check the first against React's output, and the second in a browser.

### Reference solution

```js
// file: project/tiny-render.js
// A renderer for React elements, in 60 lines. React elements are plain objects; components are
// functions from props to elements. Rendering means walking that tree until only strings are left.
import { Fragment } from "react";

const escapeText = (s) => String(s).replace(/[&<>]/g, (c) => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;" })[c]);
const escapeAttr = (s) => escapeText(s).replace(/"/g, "&quot;");
const VOID = new Set(["br", "hr", "img", "input", "meta", "link"]);
const ATTR = { className: "class", htmlFor: "for" };

/** Render an element tree to an HTML string (like react-dom/server's renderToString, minus its markers). */
export function renderToHtml(node) {
  if (node === null || node === undefined || typeof node === "boolean") return "";   // renders nothing
  if (typeof node === "string" || typeof node === "number") return escapeText(node);
  if (Array.isArray(node)) return node.map(renderToHtml).join("");
  const { type, props } = node;
  if (type === Fragment) return renderToHtml(props.children);
  if (typeof type === "function") return renderToHtml(type(props));              // a component: call it
  let attrs = "";
  for (const [name, value] of Object.entries(props)) {
    if (name === "children" || name.startsWith("on") || value == null || value === false) continue;
    if (name === "style") {
      const css = Object.entries(value).map(([k, v]) => `${k.replace(/[A-Z]/g, (c) => "-" + c.toLowerCase())}:${v}`).join(";");
      attrs += ` style="${escapeAttr(css)}"`;
    } else attrs += ` ${ATTR[name] ?? name}${value === true ? '=""' : `="${escapeAttr(value)}"`}`;
  }
  if (VOID.has(type)) return `<${type}${attrs}/>`;
  return `<${type}${attrs}>${renderToHtml(props.children)}</${type}>`;
}

/** Create real DOM nodes from an element tree (once: no updates, no state). */
export function mount(node, parent) {
  if (node === null || node === undefined || typeof node === "boolean") return;
  if (typeof node === "string" || typeof node === "number") return parent.append(String(node));
  if (Array.isArray(node)) return node.forEach((n) => mount(n, parent));
  const { type, props } = node;
  if (type === Fragment) return mount(props.children, parent);
  if (typeof type === "function") return mount(type(props), parent);
  const el = document.createElement(type);
  for (const [name, value] of Object.entries(props)) {
    if (name === "children" || value == null) continue;
    if (name.startsWith("on")) el.addEventListener(name.slice(2).toLowerCase(), value);
    else if (name === "style") Object.assign(el.style, value);
    else if (name in el) el[name] = value;                                         // className, htmlFor, value...
    else el.setAttribute(name, value);
  }
  mount(props.children, el);
  parent.append(el);
}
```

```jsx
// file: project/app.jsx
export function Badge({ children, tone = "info" }) {
  return <span className={`badge badge-${tone}`} title={tone}>{children}</span>;
}
export function ProductList({ products, onBuy }) {
  return (
    <>
      <h1>Products ({products.length})</h1>
      <ul>
        {products.map((p) => (
          <li key={p.id} style={{ fontWeight: p.featured ? "bold" : "normal" }}>
            {p.name} — {p.price.toFixed(2)} {p.stock === 0 && <Badge tone="warn">sold out</Badge>}
            <button disabled={p.stock === 0} onClick={() => onBuy(p)}>Buy</button>
          </li>
        ))}
      </ul>
      <label htmlFor="q">Search</label><input id="q" placeholder={'"quoted" & <angled>'} />
    </>
  );
}
export const products = [
  { id: 1, name: "Tea <green>", price: 4.5, stock: 10, featured: true },
  { id: 2, name: "Coffee & cream", price: 6, stock: 0, featured: false },
];
```

```jsx
// file: project/compare.jsx
// Does our renderer agree with React's? Compare the HTML both produce for the same element tree.
import { renderToString } from "react-dom/server";
import { renderToHtml } from "./tiny-render.js";
import { ProductList, products } from "./app.jsx";

const tree = <ProductList products={products} onBuy={() => {}} />;
const ours = renderToHtml(tree);
const reacts = renderToString(tree).replaceAll("<!-- -->", "");   // React separates adjacent text nodes
console.log(ours);
console.log(ours === reacts ? "identical to React's output" : `DIFFERENT from React's:\n${reacts}`);
```

```console
$ npx tsx project/compare.jsx
<h1>Products (2)</h1><ul><li style="font-weight:bold">Tea &lt;green&gt; — 4.50 <button>Buy</button></li><li style="font-weight:normal">Coffee &amp; cream — 6.00 <span class="badge badge-warn" title="warn">sold out</span><button disabled="">Buy</button></li></ul><label for="q">Search</label><input id="q" placeholder="&quot;quoted&quot; &amp; &lt;angled&gt;"/>
identical to React's output
```

Character for character the same as React's output (once React's `<!-- -->` markers between adjacent text nodes are removed — they let the browser-side code find text-node boundaries when it takes over the server's HTML, Chapter 12). In the browser:

```html
<!-- file: project/index.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Tiny renderer in the browser</title></head>
<body>
  <div id="root"></div>
  <script type="module" src="./main.jsx"></script>
</body>
</html>
```

```jsx
// file: project/main.jsx
import { mount } from "./tiny-render.js";
import { ProductList, products } from "./app.jsx";

mount(<ProductList products={products} onBuy={(p) => console.log(`buy ${p.name}`)} />, document.getElementById("root"));
console.log(`mounted: ${document.querySelectorAll("li").length} items, ` +
            `second button disabled: ${document.querySelectorAll("button")[1].disabled}`);
```

```console
$ python3 ../tools/page.py http://localhost:5173/project/ click:button --serve "npx vite --port 5173 --strictPort"
mounted: 2 items, second button disabled: true
buy Tea <green>
```

What the tiny renderer does *not* do is everything that makes React React: it cannot update. When state changes, it would have to throw the DOM away and mount again — losing focus, scroll positions, and input contents. Comparing the new element tree with the old one and changing only the differences (reconciliation, Chapter 6), keeping state between renders (hooks, Chapters 1–2), and scheduling work so that typing stays responsive (concurrent rendering, Chapter 6) are the subjects of the rest of the book.

### Review checklist

- Add support for `dangerouslySetInnerHTML`. Why does React give it such an alarming name? (Book 1, Chapter 14.)
- React renders `<input value="x">` on the server as an attribute but on the client sets the `value` *property*. Why? (Book 1, Unit 8.2.)
- Make `renderToHtml` stream: return an async iterator of chunks. Where could a slow component then be skipped and filled in later? (Chapter 12.)

## Answers to the Chapter 0 prediction questions

**Unit 0.6, Q1.** `<p>ab</p>` — arrays of children are rendered one after another (React warns about missing keys only for arrays of elements, not strings). `<p>{{}}</p>` throws: objects are not valid children. **Q2.** Because React calls them whenever it chooses — possibly several times, possibly discarding the result (StrictMode, concurrent rendering, Suspense); a component with side effects or hidden inputs would behave differently depending on how often and when it was called.

# Chapter 1 — Components, Props, and State

*Goal: build interfaces from components that receive data through props and remember things through state. You will compose components, render lists with keys (and see exactly what goes wrong without the right ones), discover that state is a snapshot for each render, that setters schedule renders rather than change variables, that mutating state does nothing, and that React keeps state by position in the tree — which explains both a common bug and the trick of resetting a component with a key. The project is a shopping cart whose only state is the cart itself; everything else is derived.*

**What you will be able to do after this chapter**

- Split a UI into components and pass data and children through props.
- Render lists with stable keys.
- Use `useState` correctly: snapshots, updater functions, batching, immutable updates.
- Decide where state should live, and derive what can be derived.
- Predict when React keeps, and when it resets, a component's state.

```js
// file: vite.config.js
// One Vite configuration per chapter: every folder with an index.html is a page at /folder/.
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  logLevel: "warn",
  // Pre-bundle React when the server starts. Otherwise Vite discovers it on the first page load,
  // bundles it, and reloads the page, so everything in it runs twice.
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client"] },
});
```

## Topic 1A — Props

## Unit 1.1 — Components and props

**One-line summary:** A component is a function whose name starts with a capital letter, takes one argument — the props object — and returns elements; props flow from parent to child, `children` holds whatever was nested between the tags, defaults are written with destructuring defaults, and props are read-only for the component that receives them.

```jsx
// file: props/props.jsx
// Components compose: props flow down, children are just another prop, and props are read-only.
import { renderToString } from "react-dom/server";

function Panel({ title, tone = "info", children }) {          // a default value for a missing prop
  return <section className={`panel ${tone}`}><h2>{title}</h2>{children}</section>;
}
function Avatar({ user, size = 32 }) {
  return <img src={user.avatarUrl} alt={user.name} width={size} height={size} />;
}
function Profile({ user }) {          // everything between <Panel> and </Panel> becomes its children
  return (
    <Panel title={user.name}>
      <Avatar user={user} size={48} />
      <p>{user.bio}</p>
    </Panel>
  );
}
const user = { name: "Ana", bio: "Writes compilers.", avatarUrl: "/ana.png" };
console.log(renderToString(<Profile user={user} />));

function Greedy(props) {
  try { props.title = "changed"; } catch (e) { console.log(`props are frozen: ${e.constructor.name}`); }
  return null;
}
renderToString(<Greedy title="original" />);
```

```console
$ npx tsx props/props.jsx
<link rel="preload" as="image" href="/ana.png"/><section class="panel info"><h2>Ana</h2><img src="/ana.png" alt="Ana" width="48" height="48"/><p>Writes compilers.</p></section>
props are frozen: TypeError
```

`Profile` passes data down to `Panel` and `Avatar`; `Panel` does not know or care what its children are. The `<link rel="preload">` in front was added by React itself: since React 19, rendering an `<img>` on the server also emits a preload hint, so that the browser can start downloading the image before it reaches the tag. The attempt to change a prop fails: in development React freezes props, and assigning to a frozen object's property throws in a module (strict mode).

## Unit 1.2 — Lists and keys

**One-line summary:** When rendering arrays, each element needs a `key` that is unique among its siblings and *stable* — the same item gets the same key on every render — because React uses keys, not positions, to decide which item a component and its state belong to; array indexes are wrong whenever items can be inserted, removed, or reordered.

```html
<!-- file: keys/index.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Keys</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```jsx
// file: keys/main.jsx
// Each row has its own state (a note being typed). Reversing the list shows what keys are for.
import { useState } from "react";
import { createRoot } from "react-dom/client";

function Row({ name }) {
  const [note, setNote] = useState("");
  return <li>{name}: <input aria-label={`note for ${name}`} value={note} onChange={(e) => setNote(e.target.value)} /></li>;
}

function List({ useIndexKeys }) {
  const [people, setPeople] = useState(["Ana", "Bo", "Cy"]);
  return (
    <section id={useIndexKeys ? "index" : "id"}>
      <button onClick={() => setPeople([...people].reverse())}>Reverse</button>
      <ul>{people.map((name, i) => <Row key={useIndexKeys ? i : name} name={name} />)}</ul>
    </section>
  );
}

createRoot(document.getElementById("root")).render(<><List useIndexKeys /><List /></>);
```

The driver types a note for Ana in both lists and then reverses both. (The other pages of this chapter use the same `index.html`, copied by the first command.)

```console
$ for d in state identity project; do mkdir -p $d && cp keys/index.html $d/; done
$ python3 ../tools/page.py http://localhost:5173/keys/ "fill:#index input >> nth=0|Ana's note" "fill:#id input >> nth=0|Ana's note" "click:#index button" "click:#id button" "eval:[...document.querySelectorAll('section')].map((s) => s.id + ': ' + [...s.querySelectorAll('li')].map((li) => li.firstChild.textContent + '=' + JSON.stringify(li.querySelector('input').value)).join(', '))" --serve "npx vite --port 5173 --strictPort"
=> ["index: Cy=\"Ana's note\", Bo=\"\", Ana=\"\"", "id: Cy=\"\", Bo=\"\", Ana=\"Ana's note\""]
```

With index keys, Ana's note is now next to Cy: the component at key 0 kept its state, and key 0 now means Cy. With the names as keys, the note moved with Ana.

### Deep & Tricky

**DT-3. Keys identify components among their siblings, so a key that changes on every render — `key={Math.random()}`, or `key={crypto.randomUUID()}` computed during render — is worse than no key: every render destroys every item and creates new ones, losing their state, focus, and DOM, and making long lists slow. Keys must come from the data (database IDs), or be generated once when the item is created.** *(React documentation, "Rendering Lists — Rules of keys": "Keys must not change or that defeats their purpose! Don't generate them while rendering")*

## Topic 1B — State

## Unit 1.3 — State is a snapshot

**One-line summary:** `useState` returns the state value *for this render* and a setter; calling the setter does not change the variable — it asks React to render again with a new value — so three calls of `setCount(count + 1)` in one handler all compute `0 + 1`, while updater functions (`setCount(c => c + 1)`) are queued and each receives the latest pending value; and all updates in one event are batched into a single render.

```jsx
// file: state/main.jsx
// Three facts about state, each with a button.
import { useState } from "react";
import { createRoot } from "react-dom/client";

function Snapshot() {
  const [count, setCount] = useState(0);
  function addThreeWrong() {
    setCount(count + 1); setCount(count + 1); setCount(count + 1);   // count is 0 in all three
    console.log(`right after the setters, count is still ${count}`);
  }
  function addThreeRight() {
    setCount((c) => c + 1); setCount((c) => c + 1); setCount((c) => c + 1);   // updater functions queue
  }
  return <p id="snap">count {count} <button onClick={addThreeWrong}>+3 (wrong)</button> <button onClick={addThreeRight}>+3 (right)</button></p>;
}

function Mutation() {
  const [items, setItems] = useState(["a"]);
  console.log(`Mutation renders: ${items.join(",")}`);
  return (
    <p id="mut">
      {items.join(", ")}{" "}
      <button onClick={() => { items.push("b"); setItems(items); }}>push (mutate)</button>{" "}
      <button onClick={() => setItems([...items, "c"])}>append (copy)</button>
    </p>
  );
}

function Batching() {
  const [a, setA] = useState(0), [b, setB] = useState(0);
  console.log(`Batching renders: a=${a} b=${b}`);
  return <p id="batch"><button onClick={() => { setA(a + 1); setB(b + 1); }}>both</button>
    <button onClick={() => setTimeout(() => { setA((x) => x + 1); setB((x) => x + 1); }, 0)}>both, later</button></p>;
}

createRoot(document.getElementById("root")).render(<><Snapshot /><Mutation /><Batching /></>);
```

```console
$ python3 ../tools/page.py http://localhost:5173/state/ "click:text=+3 (wrong)" "click:text=+3 (right)" "eval:document.getElementById('snap').firstChild.textContent + document.getElementById('snap').childNodes[1].textContent" "click:text=push (mutate)" "click:text=append (copy)" "click:text=both" "click:text=both, later" wait:100 --serve "npx vite --port 5173 --strictPort"
Mutation renders: a
Batching renders: a=0 b=0
right after the setters, count is still 0
=> "count 4"
Mutation renders: a,b,c
Batching renders: a=1 b=1
Batching renders: a=2 b=2
```

Reading the output line by line:

- "+3 (wrong)" added 1, and `count` was still 0 right after the three setter calls; "+3 (right)" added 3: total 4.
- "push (mutate)" changed the array in place and passed the *same* array to the setter: React compared old and new state with `Object.is`, saw no change, and did not render at all. The pushed `"b"` only appeared when the next real update ("append (copy)") rendered.
- Two setters in a click handler caused one render, and — since React 18 — so did two setters in a timer.

### Deep & Tricky

**DT-4. React decides whether state changed with `Object.is(old, new)`, so mutating an object or array and setting it again is invisible: no render happens, and the mutation shows up later, at an unrelated render, as if by magic. State must be treated as immutable — replace it with a new object, array, or copy (`[...a, x]`, `{ ...o, k: v }`, `structuredClone`, or a library such as Immer).** *(React documentation, "Updating Objects in State" and "Updating Arrays in State"; `useState` reference: "If the new value you provide is identical to the current state, as determined by an Object.is comparison, React will skip re-rendering")*

### Mastery check

1. **Predict.** `setCount(count + 5); setCount((c) => c * 2);` starting from 1 — what is the next `count`?

## Unit 1.4 — Where state lives: position, identity, and keys

**One-line summary:** React stores state for a component at its *position* in the rendered tree; as long as the same component type renders at the same position, its state survives re-renders (even with different props), and it is destroyed when the type or position changes — so defining a component inside another component resets it on every render, and giving a component a new `key` is the deliberate way to reset it.

```jsx
// file: identity/main.jsx
// React keeps state per position in the tree, per component type. Two ways that surprises people.
import { useState } from "react";
import { createRoot } from "react-dom/client";

function Field({ label }) {
  const [text, setText] = useState("");
  return <label>{label} <input value={text} onChange={(e) => setText(e.target.value)} /></label>;
}

function Form() {
  const [user, setUser] = useState("ana");
  const [ticks, setTicks] = useState(0);
  // BUG: a component defined inside another is a NEW function on every render, so React sees a
  // different component type each time, unmounts the old one, and its state is lost.
  function InnerField() { return <Field label="inner" />; }
  return (
    <div>
      <p>
        <button onClick={() => setTicks(ticks + 1)}>re-render ({ticks})</button>{" "}
        <button onClick={() => setUser(user === "ana" ? "bo" : "ana")}>switch user ({user})</button>
      </p>
      <p id="outer"><Field label="outer" /></p>
      <p id="inner"><InnerField /></p>
      <p id="same"><Field label={`message to ${user}`} /></p>
      <p id="keyed"><Field key={user} label={`message to ${user}, keyed`} /></p>
    </div>
  );
}
createRoot(document.getElementById("root")).render(<Form />);
```

The driver types "x" into all four fields, re-renders the form, and then switches user:

```console
$ python3 ../tools/page.py http://localhost:5173/identity/ "fill:#outer input|x" "fill:#inner input|x" "fill:#same input|x" "fill:#keyed input|x" "eval:['outer','inner','same','keyed'].map((id) => id + '=' + JSON.stringify(document.querySelector('#' + id + ' input').value)).join(' ')" "click:text=re-render" "eval:['outer','inner','same','keyed'].map((id) => id + '=' + JSON.stringify(document.querySelector('#' + id + ' input').value)).join(' ')" "click:text=switch user" "eval:['outer','inner','same','keyed'].map((id) => id + '=' + JSON.stringify(document.querySelector('#' + id + ' input').value)).join(' ')" --serve "npx vite --port 5173 --strictPort"
=> "outer=\"x\" inner=\"x\" same=\"x\" keyed=\"x\""
=> "outer=\"x\" inner=\"\" same=\"x\" keyed=\"x\""
=> "outer=\"x\" inner=\"\" same=\"x\" keyed=\"\""
```

After a mere re-render, the field inside `InnerField` is empty: `InnerField` is a new function each time `Form` renders, so React sees a different component type and replaces it. After switching user, the unkeyed "message to bo" still contains the text typed to Ana — same type, same position — while the keyed field was reset because its key changed. That second behaviour is usually what you want for a form tied to a record: `<EditForm key={record.id} …/>`.

### Pattern

**P-1. Keep the minimum state: store only what cannot be computed from props or other state, and derive everything else during rendering; lift state to the closest common parent of the components that need it; and do not copy props into state (it stops following the prop).** *(React documentation, "Choosing the State Structure" and "Sharing State Between Components")* *Trade-off:* deriving values on every render costs computation; when that becomes measurable, cache it with `useMemo` (Chapter 2) rather than storing it.

### Mastery check

1. **Predict.** In a component, `{isAdmin ? <Panel title="Admin" /> : <Panel title="User" />}`. Does toggling `isAdmin` reset `Panel`'s state?

## Chapter 1 Project — A shopping cart

### Requirements

Build a shop page with a product list and a cart. The only state is the cart: a map from product ID to quantity, held in the common parent. Derive everything else while rendering: item count, line totals, the total, which "Add" and "+" buttons are disabled (stock limits), and the empty-cart message. Update state immutably, use product IDs as keys, label every button for assistive technology, and keep each cart line in its place when its quantity changes.

### Reference solution

```jsx
// file: project/shop.jsx
// A product list and a cart. One piece of state — the cart lines — lives in the common parent;
// everything else (counts, totals, which buttons are disabled) is derived from it while rendering.
import { useState } from "react";

const money = new Intl.NumberFormat("en-US", { style: "currency", currency: "USD" });
const fmt = (cents) => money.format(cents / 100);

export const PRODUCTS = [
  { id: "tea", name: "Green tea", price: 450, stock: 3 },
  { id: "cof", name: "Coffee beans", price: 1299, stock: 5 },
  { id: "mug", name: "Mug", price: 899, stock: 0 },
];

function ProductList({ products, inCart, onAdd }) {
  return (
    <section aria-labelledby="products-h">
      <h2 id="products-h">Products</h2>
      <ul>
        {products.map((p) => {
          const left = p.stock - (inCart[p.id] ?? 0);
          return (
            <li key={p.id}>
              {p.name} — {fmt(p.price)}{" "}
              <button onClick={() => onAdd(p.id)} disabled={left === 0} aria-label={p.stock === 0 ? `${p.name}: sold out` : `Add ${p.name} to cart`}>
                {p.stock === 0 ? "Sold out" : "Add"}
              </button>
            </li>
          );
        })}
      </ul>
    </section>
  );
}

function CartLine({ product, qty, onChange }) {
  return (
    <li>
      {product.name} × {qty} = {fmt(product.price * qty)}{" "}
      <button onClick={() => onChange(qty - 1)} aria-label={`One less ${product.name}`}>−</button>
      <button onClick={() => onChange(qty + 1)} disabled={qty >= product.stock} aria-label={`One more ${product.name}`}>+</button>
      <button onClick={() => onChange(0)} aria-label={`Remove ${product.name}`}>Remove</button>
    </li>
  );
}

export function Shop({ products = PRODUCTS }) {
  const [cart, setCart] = useState({});               // { productId: quantity } — the only state
  const byId = Object.fromEntries(products.map((p) => [p.id, p]));
  const setQty = (id, qty) => setCart((c) => {
    if (qty > 0) return { ...c, [id]: qty };          // a new object every time: never mutate state
    const { [id]: _removed, ...rest } = c;            // (and spreading keeps each line in its place)
    return rest;
  });
  const lines = Object.entries(cart);
  const count = lines.reduce((n, [, q]) => n + q, 0);                       // derived, not stored
  const total = lines.reduce((s, [id, q]) => s + byId[id].price * q, 0);
  return (
    <main>
      <ProductList products={products} inCart={cart} onAdd={(id) => setQty(id, (cart[id] ?? 0) + 1)} />
      <section aria-labelledby="cart-h">
        <h2 id="cart-h">Cart ({count} {count === 1 ? "item" : "items"})</h2>
        {lines.length === 0 ? <p>Your cart is empty.</p> : (
          <ul>{lines.map(([id, qty]) => <CartLine key={id} product={byId[id]} qty={qty} onChange={(q) => setQty(id, q)} />)}</ul>
        )}
        <p>Total: <strong>{fmt(total)}</strong></p>
      </section>
    </main>
  );
}
```

```jsx
// file: project/main.jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { Shop } from "./shop.jsx";

createRoot(document.getElementById("root")).render(<StrictMode><Shop /></StrictMode>);
```

```console
$ python3 ../tools/page.py http://localhost:5173/project/ "aria:main >> nth=0" "click:text=Add >> nth=0" "click:[aria-label='Add Coffee beans to cart']" "click:[aria-label='One more Green tea']" "click:[aria-label='One more Green tea']" "eval:document.querySelector('[aria-labelledby=cart-h]').innerText.replaceAll('\n', ' | ')" "eval:document.querySelector('[aria-label=\"One more Green tea\"]').disabled" "click:[aria-label='Remove Coffee beans']" "click:[aria-label='One less Green tea']" "eval:document.querySelector('[aria-labelledby=cart-h]').innerText.replaceAll('\n', ' | ')" --serve "npx vite --port 5173 --strictPort"
- main:
  - region "Products":
    - heading "Products" [level=2]
    - list:
      - listitem:
        - text: Green tea — $4.50
        - button "Add Green tea to cart": Add
      - listitem:
        - text: Coffee beans — $12.99
        - button "Add Coffee beans to cart": Add
      - listitem:
        - text: Mug — $8.99
        - 'button "Mug: sold out" [disabled]': Sold out
  - region "Cart (0 items)":
    - heading "Cart (0 items)" [level=2]
    - paragraph: Your cart is empty.
    - paragraph:
      - text: "Total:"
      - strong: $0.00
=> "Cart (4 items) | Green tea × 3 = $13.50 −+Remove | Coffee beans × 1 = $12.99 −+Remove |  | Total: $26.49"
=> true
=> "Cart (2 items) | Green tea × 2 = $9.00 −+Remove |  | Total: $9.00"
```

Three green teas reach the stock limit, so "One more Green tea" is disabled — without any code that disables it, because it is derived from the state on each render. Removing a line and reducing a quantity update the count and total consistently, since there is no second copy of them to forget.

The first version of `setQty` removed the key and added it again (`{ ...rest, [id]: qty }`), which moved a product to the end of the cart every time its quantity changed: object keys keep insertion order. Spreading the old object and overwriting the key keeps the line in place.

### Review checklist

- Prices are integers of cents (Book 1, Unit 5.2). What would go wrong with `price: 4.5`?
- Store the cart in `localStorage` so it survives a reload. Where should that code go? (Chapter 3.)
- The product list re-renders whenever the cart changes. Is that a problem? (Chapter 6.)

## Answers to the Chapter 1 prediction questions

**Unit 1.3, Q1.** 12: the first call queues "replace with 6" (`count` is 1 in this render), the updater then doubles the pending value.

**Unit 1.4, Q1.** No: both branches render a `Panel` at the same position, so React sees the same component with new props and keeps its state. To reset it, give the two branches different keys.

# Chapter 2 — Hooks in Depth

*Goal: understand hooks well enough to predict them. Hooks are functions that let a component keep things between renders — state, references, cached values, identities — and React identifies each hook only by the order in which it is called. You will see a broken order silently give one variable another's value, use refs for values that must not cause renders and for DOM access, see `useMemo` cache correctly and uselessly, write custom hooks that share logic but never state, and finally implement `useState`, `useRef`, and `useMemo` yourself in a hundred-line component runtime.*

**What you will be able to do after this chapter**

- State the rules of hooks and explain why they exist.
- Choose between state and refs, and use refs for DOM nodes.
- Use `useMemo` and `useCallback` with correct dependencies, and know when they do nothing.
- Write custom hooks, including accessible form-field hooks with `useId`.
- Explain how React stores hook state per component instance and call order.

```js
// file: vite.config.js
// One Vite configuration per chapter: every folder with an index.html is a page at /folder/.
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  logLevel: "warn",
  // Pre-bundle React when the server starts. Otherwise Vite discovers it on the first page load,
  // bundles it, and reloads the page, so everything in it runs twice.
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client"] },
});
```

```html
<!-- file: page.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Chapter 2</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```console
$ for d in rules refs memo custom; do mkdir -p $d && cp page.html $d/index.html; done
```

## Topic 2A — The rules

## Unit 2.1 — Hooks are identified by call order

**One-line summary:** Hooks must be called at the top level of a component or custom hook — never inside conditions, loops, or nested functions — because React stores each component's hook data as a list and matches the n-th hook call of this render to the n-th slot of the previous one; it has no other way to know which `useState` is which.

```jsx
// file: rules/main.jsx
// Hooks are identified by the ORDER in which they are called. A conditional hook breaks the order.
import { useState } from "react";
import { createRoot } from "react-dom/client";

function Profile({ showNickname }) {
  const [name, setName] = useState("Ana");
  if (showNickname) {
    const [nickname] = useState("Annie");          // BREAKS THE RULES: only called on some renders
    return <p>{name} ({nickname})</p>;
  }
  const [age] = useState(36);
  return <p>{name}, {age}</p>;
}

function App() {
  const [show, setShow] = useState(false);
  return <><button onClick={() => setShow(!show)}>toggle</button><Profile showNickname={show} /></>;
}
createRoot(document.getElementById("root")).render(<App />);
```

```console
$ python3 ../tools/page.py http://localhost:5173/rules/ "eval:document.querySelector('p').textContent" click:button "eval:document.querySelector('p').textContent" click:button "eval:document.querySelector('p').textContent" --serve "npx vite --port 5173 --strictPort"
=> "Ana, 36"
=> "Ana (36)"
=> "Ana, 36"
```

After the toggle, the nickname shows 36: the second `useState` call of this render was the nickname's, and React gave it the second slot — which holds the age. Because both hooks are `useState`, React cannot even detect the mistake (it warns only when the *kind* of hook at a position changes). The ESLint plugin `eslint-plugin-react-hooks` (rule `rules-of-hooks`) reports such code before it runs.

### Mastery check

1. **Fix.** Rewrite `Profile` so that it follows the rules and still shows the nickname only when asked.

## Topic 2B — The basic hooks

## Unit 2.2 — `useRef`: values that do not render

**One-line summary:** `useRef(initial)` returns the same mutable object `{ current }` on every render; changing `current` does not cause a render, which makes refs right for values the UI does not display — timers, previous values, instance counters — and for holding DOM nodes, which React assigns to a ref passed as the `ref` prop.

```jsx
// file: refs/main.jsx
// useRef: a box that survives renders and does NOT cause one when it changes.
import { useRef, useState } from "react";
import { createRoot } from "react-dom/client";

function Search() {
  const input = useRef(null);                     // React puts the DOM node here after it renders
  const renders = useRef(0);
  const [query, setQuery] = useState("");
  renders.current++;                              // fine: not used to decide what to render
  return (
    <p>
      <input ref={input} value={query} onChange={(e) => setQuery(e.target.value)} aria-label="Search" />
      <button onClick={() => input.current.focus()}>Focus the search box</button>
      <button onClick={() => { input.current.value = "typed by the DOM"; }}>Write to the DOM directly</button>
      <span id="info"> renders: {renders.current}, state: "{query}"</span>
    </p>
  );
}

function Stopwatch() {
  const [, force] = useState(0);
  const clicks = useRef(0);
  return <p><button onClick={() => { clicks.current++; console.log(`ref is ${clicks.current}`); }}>count in a ref</button>
    <button onClick={() => force((n) => n + 1)}>re-render</button> shown: {clicks.current}</p>;
}
createRoot(document.getElementById("root")).render(<><Search /><Stopwatch /></>);
```

```console
$ python3 ../tools/page.py http://localhost:5173/refs/ "fill:input|abc" "click:text=Focus the search box" "eval:document.activeElement.tagName" "click:text=Write to the DOM directly" "eval:document.querySelector('input').value + ' |' + document.getElementById('info').textContent" "click:text=count in a ref" "click:text=count in a ref" "eval:document.querySelectorAll('p')[1].textContent" "click:text=re-render" "eval:document.querySelectorAll('p')[1].textContent" --serve "npx vite --port 5173 --strictPort"
=> "INPUT"
=> "typed by the DOM | renders: 2, state: \"abc\""
ref is 1
ref is 2
=> "count in a refre-render shown: 0"
=> "count in a refre-render shown: 2"
```

The ref gives access to the real input for `focus()`. Writing to the DOM through the ref *worked* — the input shows "typed by the DOM" — but React's state still says "abc", and the next render will overwrite the input: the DOM and React now disagree. The click counter in a ref went to 2 without the screen changing; only an unrelated re-render revealed it. Refs are for things React does not render.

### Deep & Tricky

**DT-5. Reading or writing `ref.current` during rendering makes a component impure — its output depends on something React does not track — so concurrent rendering, StrictMode's double render, or a discarded render can show stale or doubled values; the render counter above counts *attempts*, and under StrictMode it would count two per update. Read and write refs in event handlers and effects; the one sanctioned exception is initialising a ref lazily (`if (ref.current === null) ref.current = new Thing()`).** *(React documentation, `useRef` reference, "Pitfall: Do not write or read ref.current during rendering, except for initialization")*

## Unit 2.3 — `useMemo` and `useCallback`

**One-line summary:** `useMemo(compute, deps)` returns the cached result of `compute` until one of the dependencies changes according to `Object.is`, and `useCallback(fn, deps)` is the same for a function; they are performance tools for expensive computations and for keeping object identities stable, and they do nothing useful when a dependency is a new object on every render.

```jsx
// file: memo/main.jsx
// useMemo caches a computed value until one of its dependencies changes (compared with Object.is).
import { useMemo, useState } from "react";
import { createRoot } from "react-dom/client";

const PRODUCTS = Array.from({ length: 20_000 }, (_, i) => ({ id: i, name: `item ${i}`, price: (i * 7919) % 1000 }));
let filterRuns = 0;
function filterProducts(products, { maxPrice }) {
  filterRuns++;
  return products.filter((p) => p.price <= maxPrice);
}

function List() {
  const [maxPrice, setMaxPrice] = useState(100);
  const [dark, setDark] = useState(false);
  const options = { maxPrice };                                           // a NEW object on every render
  const cached = useMemo(() => filterProducts(PRODUCTS, { maxPrice }), [maxPrice]);
  const useless = useMemo(() => filterProducts(PRODUCTS, options), [options]);   // deps change every time
  return (
    <p className={dark ? "dark" : ""}>
      {cached.length} / {useless.length} products{" "}
      <button onClick={() => setDark(!dark)}>toggle theme</button>
      <button onClick={() => setMaxPrice(maxPrice + 100)}>raise price limit</button>
    </p>
  );
}
createRoot(document.getElementById("root")).render(<List />);
window.filterRuns = () => filterRuns;
```

```console
$ python3 ../tools/page.py http://localhost:5173/memo/ "eval:filterRuns()" "click:text=toggle theme" "click:text=toggle theme" "eval:filterRuns()" "click:text=raise price limit" "eval:[filterRuns(), document.querySelector('p').firstChild.textContent]" --serve "npx vite --port 5173 --strictPort"
=> 2
=> 4
=> [6, "4020"]
```

After the first render both memos had computed once (2). Toggling the theme twice re-rendered twice: the correct memo returned its cached array, while the one depending on `options` — a fresh object each render — recomputed every time (4). Raising the price limit changed `maxPrice`, so both recomputed (6). The React Compiler (Chapter 6) inserts this kind of memoisation automatically, and correctly.

### Mastery check

1. **Predict.** `useMemo(() => compute(a), [])` — what happens when `a` changes?

## Unit 2.4 — Custom hooks

**One-line summary:** Any function whose name starts with `use` and that calls hooks is a custom hook; it lets components share *logic* — not state: every component that calls it gets its own copy of the state — and is the standard way to package behaviour such as toggles, form fields, subscriptions, and data loading; `useId` gives stable unique IDs for accessibility attributes.

```jsx
// file: custom/main.jsx
// A custom hook is a function whose name starts with "use" and that calls other hooks.
// It shares LOGIC between components; each component that calls it gets its OWN state.
import { useState, useId } from "react";
import { createRoot } from "react-dom/client";

function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  const toggle = () => setOn((v) => !v);
  return [on, toggle];
}

function useField(label, validate) {
  const id = useId();                                 // unique, stable, and the same on server and client
  const [value, setValue] = useState("");
  const [touched, setTouched] = useState(false);
  const error = touched ? validate(value) : null;
  return {
    value, error,
    inputProps: { id, value, onChange: (e) => setValue(e.target.value), onBlur: () => setTouched(true),
                  "aria-invalid": Boolean(error), "aria-describedby": error ? `${id}-err` : undefined },
    labelProps: { htmlFor: id, children: label },
    errorProps: { id: `${id}-err`, children: error },
  };
}

function Field({ label, validate }) {
  const f = useField(label, validate);
  return <p><label {...f.labelProps} /> <input {...f.inputProps} /> {f.error && <span {...f.errorProps} />}</p>;
}

function Panel({ title }) {
  const [open, toggle] = useToggle();
  return <section><button aria-expanded={open} onClick={toggle}>{title}</button>{open && <p>{title} content</p>}</section>;
}

createRoot(document.getElementById("root")).render(<>
  <Panel title="First" /><Panel title="Second" />
  <Field label="Email" validate={(v) => (v.includes("@") ? null : "needs an @")} />
  <Field label="Name" validate={(v) => (v ? null : "required")} />
</>);
```

```console
$ python3 ../tools/page.py http://localhost:5173/custom/ "click:text=First" "fill:#root input >> nth=0|ana" press:Tab "aria:body" --serve "npx vite --port 5173 --strictPort"
- button "First" [expanded]
- paragraph: First content
- button "Second"
- paragraph:
  - text: Email
  - textbox "Email" [invalid]: ana
  - text: needs an @
- paragraph:
  - text: Name
  - textbox "Name"
```

Only the first panel opened: each `Panel`'s `useToggle` has its own state. The email field, after losing focus with an invalid value, is marked invalid with its message; the name field, never touched, shows nothing. The IDs from `useId` connect each label, input, and error message without any naming scheme of our own — and are the same on the server and the client, which matters when HTML is rendered on the server (Chapter 12).

### Pattern

**P-2. Extract a custom hook when the same combination of hooks and logic appears in two components, or when a component's logic deserves a name (`useCart`, `useOnlineStatus`); return the smallest useful interface (often a tuple or an object of named values and functions), and keep hooks free of JSX.** *(React documentation, "Reusing Logic with Custom Hooks")* *Trade-off:* hooks that hide effects and subscriptions make components shorter but their behaviour less visible — a component calling five custom hooks may subscribe to five things; name hooks after what they do.

## Chapter 2 Project — Implement hooks

### Requirements

Write a tiny component runtime that renders components to strings and supports `useState` (with updater functions, `Object.is` bail-out, and batching of updates in the same tick), `useRef`, and `useMemo`. Identify component instances by their position in the tree, key, and type, so that state survives re-renders, disappears when a component unmounts, and starts fresh when it mounts again. Detect a change in the number of hooks between renders. Demonstrate each behaviour.

### Reference solution

```js
// file: project/hooks.js
// A tiny component runtime with hooks: enough to show HOW React remembers state between renders.
// State lives outside the component, in a slot list per component instance; an instance is
// identified by its position in the tree (and its key and type), and its hooks by call order.
import { Fragment } from "react";

const instances = new Map();          // instance id -> { hooks: [], count }
let current = null;                   // the instance currently rendering
let rerender = null, scheduled = false;
export const handlers = new Map();    // element id -> onClick, so that tests can "click"

function schedule() {
  if (scheduled) return;
  scheduled = true;
  queueMicrotask(() => {                                     // batch all updates of this tick
    scheduled = false;
    try { rerender(); } catch (e) { console.log(`render failed: ${e.message}`); }
  });
}

export function useState(initial) {
  const inst = current, i = inst.cursor++;
  if (!(i in inst.hooks)) inst.hooks[i] = typeof initial === "function" ? initial() : initial;
  const set = (v) => {
    const next = typeof v === "function" ? v(inst.hooks[i]) : v;
    if (Object.is(next, inst.hooks[i])) return;               // same value: no re-render
    inst.hooks[i] = next;
    schedule();
  };
  return [inst.hooks[i], set];
}

export function useRef(initial) {
  const inst = current, i = inst.cursor++;
  if (!(i in inst.hooks)) inst.hooks[i] = { current: initial };
  return inst.hooks[i];
}

export function useMemo(compute, deps) {
  const inst = current, i = inst.cursor++;
  const old = inst.hooks[i];
  if (old && deps.length === old.deps.length && deps.every((d, k) => Object.is(d, old.deps[k]))) return old.value;
  inst.hooks[i] = { value: compute(), deps };
  return inst.hooks[i].value;
}

function renderNode(node, path, seen) {
  if (node == null || typeof node === "boolean") return "";
  if (typeof node === "string" || typeof node === "number") return String(node);
  if (Array.isArray(node)) return node.map((n, i) => renderNode(n, `${path}/${n?.key ?? i}`, seen)).join("");
  const { type, props } = node;
  if (type === Fragment) return renderNode(props.children, path, seen);
  if (typeof type === "function") {
    const id = `${path}<${type.name}>`;                       // position + type = identity
    seen.add(id);
    const inst = instances.get(id) ?? { hooks: [], count: null };
    instances.set(id, inst);
    const prev = current;
    current = inst; inst.cursor = 0;
    try {
      const out = type(props);
      if (inst.count !== null && inst.cursor !== inst.count)
        throw new Error(`${type.name} called ${inst.cursor} hooks, but ${inst.count} during the previous render`);
      inst.count = inst.cursor;
      return renderNode(out, `${id}`, seen);
    } finally { current = prev; }
  }
  if (props.id && props.onClick) handlers.set(props.id, props.onClick);
  const attrs = props.id ? ` id="${props.id}"` : "";
  return `<${type}${attrs}>${renderNode(props.children, path + "/" + type, seen)}</${type}>`;
}

export function createRoot(onCommit) {
  let element;
  rerender = () => {
    const seen = new Set();
    handlers.clear();
    const html = renderNode(element, "root", seen);
    for (const id of instances.keys()) if (!seen.has(id)) instances.delete(id);   // unmounted: state is gone
    onCommit(html);
  };
  return { render(el) { element = el; rerender(); } };
}
export const click = (id) => handlers.get(id)();
export const settle = () => new Promise((r) => setTimeout(r, 0));
```

```jsx
// file: project/demo.jsx
import { createRoot, useState, useMemo, useRef, click, settle } from "./hooks.js";

let expensive = 0;
function Counter({ label }) {
  const [count, setCount] = useState(0);
  const renders = useRef(0);
  renders.current++;
  const squared = useMemo(() => (expensive++, count * count), [count]);
  return <li>{label}: {count} (squared {squared}, rendered {renders.current}×)
    <button id={`inc-${label}`} onClick={() => { setCount((c) => c + 1); setCount((c) => c + 1); }}>+2</button></li>;
}

function App() {
  const [showB, setShowB] = useState(true);
  const [tick, setTick] = useState(0);
  return <div>
    <button id="toggle" onClick={() => setShowB(!showB)}>toggle B</button>
    <button id="tick" onClick={() => setTick(tick + 1)}>tick</button>
    <ul><Counter label="A" />{showB && <Counter label="B" />}</ul>
  </div>;
}

const root = createRoot((html) => console.log(html.replace(/<\/?(div|ul|button)[^>]*>|toggle B|tick|\+2/g, "").replace(/<\/li>/g, " | ")));
root.render(<App />);
click("inc-A"); await settle();                 // two updates, one render
click("tick"); await settle();                  // App re-renders: counters keep their state
click("toggle"); await settle();                // B unmounts: its state is deleted
click("toggle"); await settle();                // B mounts again: fresh state
console.log(`useMemo computed ${expensive} times`);

function Broken({ flag }) {
  const [a] = useState(1);
  if (flag) useState(2);                          // breaks the order
  return <p>{a}</p>;
}
function Parent() { const [f, setF] = useState(false); return <div><button id="f" onClick={() => setF(true)}>x</button><Broken flag={f} /></div>; }
const root2 = createRoot(() => {});
root2.render(<Parent />);
click("f");
await settle();
```

```console
$ npx tsx project/demo.jsx
<li>A: 0 (squared 0, rendered 1×) | <li>B: 0 (squared 0, rendered 1×) | 
<li>A: 2 (squared 4, rendered 2×) | <li>B: 0 (squared 0, rendered 2×) | 
<li>A: 2 (squared 4, rendered 3×) | <li>B: 0 (squared 0, rendered 3×) | 
<li>A: 2 (squared 4, rendered 4×) | 
<li>A: 2 (squared 4, rendered 5×) | <li>B: 0 (squared 0, rendered 1×) | 
useMemo computed 4 times
render failed: Broken called 2 hooks, but 1 during the previous render
```

Line by line: the first render; one click that queued two updater functions produced *one* render with A at 2 (batching) — and B rendered too, because this runtime re-renders the whole tree on every update, where React re-renders only the component whose state changed and its descendants; a parent re-render left both counters' state alone while the render counts went up; hiding B deleted its instance, and showing it again created a fresh one (count 0, rendered once). `useMemo` computed 4 times in 5 renders of A and 3 of B — only when `count` changed or an instance was new. Finally, the component that called a hook conditionally was caught.

The real React differs in scale, not in idea: instances are *fibers* in a tree rather than strings in a map, hooks are a linked list on each fiber, updates are queued per hook and processed with priorities (Chapter 6), and the tree is reconciled rather than re-rendered to a string. But "state belongs to a position in the tree, hooks belong to a call order" is exactly how it works.

### Review checklist

- Add `useEffect` to the runtime: run the effect after "commit", re-run when dependencies change, and call the cleanup first. (Chapter 3 explains the semantics.)
- Our instance IDs use the function's `name`. What breaks with two different anonymous components, or after minification?
- Why does React need the `count` check at all if every hook is a `useState`? (Look back at Unit 2.1.)

## Answers to the Chapter 2 prediction questions

**Unit 2.1, Q1.** Call every hook unconditionally, then decide what to render: `const [name] = useState("Ana"); const [nickname] = useState("Annie"); const [age] = useState(36); return showNickname ? <p>{name} ({nickname})</p> : <p>{name}, {age}</p>;`.

**Unit 2.3, Q1.** Nothing: with an empty dependency list the value is computed once and never updated, so the component keeps showing the result for the first `a` — a stale-value bug that the `exhaustive-deps` lint rule reports.

# Chapter 3 — Effects and Data Fetching

*Goal: use effects for what they are for — keeping a component synchronised with something outside React — and not for what they are not. You will watch StrictMode expose an effect without cleanup by leaving connections open, see an interval stuck on a stale value and three ways to fix it (including `useEffectEvent`), measure the extra render and the inconsistent frame that an unnecessary effect causes, and reproduce the classic data-fetching race in which an old response overwrites a new one. The project is search-as-you-type with debouncing, cancellation, and loading states.*

**What you will be able to do after this chapter**

- Write effects with correct dependencies and cleanups.
- Recognise and fix stale closures in effects.
- Remove effects that only compute derived data or respond to events.
- Fetch data in effects without race conditions, and know when to use a library instead.
- Choose between `useEffect` and `useLayoutEffect`.

```js
// file: vite.config.js
// This chapter's Vite configuration also serves a small fake API, with deliberate delays.
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

const USERS = { 1: "Ana", 2: "Bo", 3: "Cy" };
const WORDS = ["react", "reason", "reduce", "redux", "ref", "render", "rust", "ruby", "remix", "relay"];

function fakeApi() {
  return {
    name: "fake-api",
    configureServer(server) {
      server.middlewares.use("/api", (req, res) => {
        const url = new URL(req.url, "http://x");
        const reply = (data, ms) => setTimeout(() => {
          res.setHeader("Content-Type", "application/json");
          res.end(JSON.stringify(data));
        }, ms);
        const user = /^\/users\/(\d+)$/.exec(url.pathname);
        if (user) return reply({ id: Number(user[1]), name: USERS[user[1]] }, user[1] === "1" ? 600 : 100);
        if (url.pathname === "/search") {             // shorter queries are slower: replies arrive out of order
          const q = url.searchParams.get("q") ?? "";
          console.log(`  api: search "${q}"`);
          return reply({ q, results: WORDS.filter((w) => w.startsWith(q)) }, Math.max(50, 400 - q.length * 100));
        }
        res.statusCode = 404; res.end("{}");
      });
    },
  };
}

export default defineConfig({
  plugins: [react(), fakeApi()],
  logLevel: "warn",
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client"] },
});
```

```html
<!-- file: page.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Keys</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```console
$ for d in effect stale needless race project; do mkdir -p $d && cp page.html $d/index.html; done
```

## Topic 3A — Effects

## Unit 3.1 — Synchronising with the outside world

**One-line summary:** `useEffect(setup, deps)` runs `setup` after React has updated the DOM, and again whenever a dependency has changed; the function that `setup` returns is the cleanup, which React runs before the next setup and when the component unmounts — so an effect is a description of how to start *and stop* synchronising with an external system for the current props and state.

```jsx
// file: effect/main.jsx
// An effect synchronises a component with something outside React — here, a "chat connection".
import { StrictMode, useEffect, useState } from "react";
import { createRoot } from "react-dom/client";

const connections = new Set();
function connect(room) {                          // a stand-in for a WebSocket or a subscription
  const c = { room };
  connections.add(c);
  console.log(`  connect to ${room}: ${connections.size} open`);
  return () => { connections.delete(c); console.log(`  disconnect from ${room}: ${connections.size} open`); };
}

function ChatRoom({ room, withCleanup }) {
  useEffect(() => {
    const disconnect = connect(room);
    if (withCleanup) return disconnect;           // the cleanup runs before the next effect, and on unmount
  }, [room, withCleanup]);                         // re-synchronise when the room changes
  return <p>Room: {room}</p>;
}

function App() {
  const [room, setRoom] = useState("general");
  const [cleanup, setCleanup] = useState(new URLSearchParams(location.search).has("cleanup"));
  const [shown, setShown] = useState(true);
  return <>
    <button onClick={() => setRoom(room === "general" ? "travel" : "general")}>switch room</button>
    <button onClick={() => setShown(!shown)}>show/hide</button>
    {shown && <ChatRoom room={room} withCleanup={cleanup} />}
  </>;
}
console.log("mount (StrictMode mounts, unmounts, and mounts again in development):");
createRoot(document.getElementById("root")).render(<StrictMode><App /></StrictMode>);
window.openConnections = () => connections.size;
```

```console
$ python3 ../tools/page.py "http://localhost:5173/effect/" "click:text=switch room" "click:text=show/hide" "eval:openConnections()" --serve "npx vite --port 5173 --strictPort"
mount (StrictMode mounts, unmounts, and mounts again in development):
  connect to general: 1 open
  connect to general: 2 open
  connect to travel: 3 open
=> 3
$ python3 ../tools/page.py "http://localhost:5173/effect/?cleanup" "click:text=switch room" "click:text=show/hide" "eval:openConnections()" --serve "npx vite --port 5173 --strictPort"
mount (StrictMode mounts, unmounts, and mounts again in development):
  connect to general: 1 open
  disconnect from general: 0 open
  connect to general: 1 open
  disconnect from general: 0 open
  connect to travel: 1 open
  disconnect from travel: 0 open
=> 0
```

Without a cleanup, StrictMode's extra mount opened a second connection at once, switching rooms opened a third, and unmounting closed nothing: three connections leak. With the cleanup, every connect is paired with a disconnect, and nothing is open at the end. StrictMode's mount–unmount–mount in development exists precisely to make the missing cleanup visible on the first load instead of after a week in production.

### Deep & Tricky

**DT-6. An effect's dependency list is not a choice of "when to run" — it must contain every reactive value (prop, state, or value derived from them) that the effect reads, and React compares each with `Object.is`; omitting one does not make the effect run less, it makes it use stale values. To run an effect less often, change the code so it does not need the value (updater functions, moving objects inside the effect, `useEffectEvent`), never lie in the list.** *(React documentation, "Removing Effect Dependencies": "you can't 'choose' your dependencies… they're determined by the code"; the `react-hooks/exhaustive-deps` lint rule)*

## Unit 3.2 — Stale closures

**One-line summary:** Every render creates new functions that capture that render's props and state; an effect set up once (`[]`) keeps calling the functions of the first render forever — a stale closure — which is fixed by depending on the value, by using an updater function, or, for a function that must read the latest values without restarting the effect, with `useEffectEvent`.

```jsx
// file: stale/main.jsx
// An effect sees the props and state of the render that created it. With the wrong dependencies,
// it keeps seeing old ones forever: a "stale closure".
import { useEffect, useEffectEvent, useState } from "react";
import { createRoot } from "react-dom/client";

function Stale() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => setCount(count + 1), 100);   // count is 0 in this closure, forever
    return () => clearInterval(id);
  }, []);                                                       // the linter would flag the missing count
  return <p id="stale">{count}</p>;
}

function Updater() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => setCount((c) => c + 1), 100); // needs no outside value at all
    return () => clearInterval(id);
  }, []);
  return <p id="updater">{count}</p>;
}

function WithEvent({ step }) {
  const [count, setCount] = useState(0);
  const onTick = useEffectEvent(() => setCount(count + step)); // always sees the latest count and step,
  useEffect(() => {                                            // without restarting the interval
    const id = setInterval(onTick, 100);
    return () => clearInterval(id);
  }, []);
  return <p id="event">{count}</p>;
}

createRoot(document.getElementById("root")).render(<><Stale /><Updater /><WithEvent step={10} /></>);
```

```console
$ python3 ../tools/page.py http://localhost:5173/stale/ wait:1050 "eval:['stale','updater','event'].map((id) => id + '=' + document.getElementById(id).textContent).join(' ')" --serve "npx vite --port 5173 --strictPort"
=> "stale=1 updater=10 event=100"
```

After about a second, the stale counter is stuck at 1 — every tick sets `0 + 1` — while the updater version counted to 10 and the effect-event version to 100 (ten ticks of step 10). `useEffectEvent` (stable since React 19.2) declares a function that is *not* reactive: it always sees the latest props and state, and the effect does not depend on it.

## Unit 3.3 — You might not need an effect

**One-line summary:** Effects are for synchronising with external systems; values that can be computed from props and state should be computed while rendering, responses to user actions belong in event handlers, and resetting state when a prop changes is done with a `key` — an effect used for any of these costs an extra render and briefly shows inconsistent UI.

```jsx
// file: needless/main.jsx
// Two ways to show a filtered list. The effect version renders twice per change and, for one
// render, shows a list that does not match the query.
import { useEffect, useLayoutEffect, useState } from "react";
import { createRoot } from "react-dom/client";

const ITEMS = ["apple", "apricot", "banana", "blueberry", "cherry"];

function WithEffect({ query }) {
  const [visible, setVisible] = useState(ITEMS);
  useEffect(() => { setVisible(ITEMS.filter((i) => i.startsWith(query))); }, [query]);   // copies derived data into state
  console.log(`WithEffect render: query "${query}", showing ${visible.length}`);
  return <p>{visible.join(", ")}</p>;
}

function Derived({ query }) {
  const visible = ITEMS.filter((i) => i.startsWith(query));                              // just compute it
  console.log(`Derived render:    query "${query}", showing ${visible.length}`);
  return <p>{visible.join(", ")}</p>;
}

function Timing() {
  console.log("Timing: render");
  useLayoutEffect(() => { console.log("Timing: layout effect (DOM updated, not yet painted)"); });
  useEffect(() => {
    console.log("Timing: effect");
    requestAnimationFrame(() => console.log("Timing: next frame"));
  });
  return null;
}

function App() {
  const [query, setQuery] = useState("");
  return <><button onClick={() => setQuery("b")}>filter "b"</button><WithEffect query={query} /><Derived query={query} /><Timing /></>;
}
createRoot(document.getElementById("root")).render(<App />);
```

```console
$ python3 ../tools/page.py http://localhost:5173/needless/ wait:100 "eval:console.log('--- click ---')" 'click:text=filter "b"' wait:100 --serve "npx vite --port 5173 --strictPort"
WithEffect render: query "", showing 5
Derived render:    query "", showing 5
Timing: render
Timing: layout effect (DOM updated, not yet painted)
Timing: effect
WithEffect render: query "", showing 5
Timing: next frame
--- click ---
=> null
WithEffect render: query "b", showing 5
Derived render:    query "b", showing 2
Timing: render
Timing: layout effect (DOM updated, not yet painted)
Timing: effect
Timing: next frame
WithEffect render: query "b", showing 2
```

After the click, `WithEffect` first rendered with query "b" but still five items — a state the user can see, since the frame was painted ("next frame") before the effect's state update produced the second, correct render. `Derived` was right the first time, with one render. The `Timing` lines show the order of a commit: render, then layout effects (DOM updated, before paint), then effects.

### Pattern

**P-3. Before writing an effect, ask what it synchronises with; if the answer is "other state", compute during render (with `useMemo` if expensive); if it is "the user did something", put it in the event handler; if it is "a prop changed, reset everything", use a `key`. Keep effects for subscriptions, timers, DOM integrations, network synchronisation, and analytics.** *(React documentation, "You Might Not Need an Effect")* *Trade-off:* some "derived" values are expensive or asynchronous; those are what `useMemo`, data-fetching libraries, and Suspense are for, not effects that copy data into state.

## Unit 3.4 — `useLayoutEffect`

**One-line summary:** `useLayoutEffect` runs after React has changed the DOM but *before* the browser paints, so it can measure layout (an element's size, a tooltip's position) and update state without the user seeing a flicker — at the cost of delaying the paint; everything else belongs in `useEffect`, which runs after paint.

The order is visible in the `Timing` output of Unit 3.3: layout effects run in the same task as the DOM update, effects afterwards. Use it sparingly: it blocks rendering, and it does not run on the server.

## Topic 3B — Fetching data

## Unit 3.5 — Races and cancellation

**One-line summary:** Fetching in an effect means the response can arrive after the props have changed again — and an older request can finish after a newer one — so every fetching effect must either ignore or abort the outdated request in its cleanup; even then, caching, deduplication, retries, and loading states make a library (TanStack Query, Chapter 8) or a framework's data loading (Chapter 12) the better choice for real applications.

This chapter's Vite configuration includes a fake API whose user 1 takes 600 ms and user 2 takes 100 ms.

```jsx
// file: race/main.jsx
// Fetching in an effect. User 1 takes 600 ms, user 2 takes 100 ms: switch from 1 to 2 quickly,
// and the slow, outdated answer arrives LAST.
import { useEffect, useState } from "react";
import { createRoot } from "react-dom/client";

function Profile({ id, safe }) {
  const [user, setUser] = useState(null);
  const [error, setError] = useState(null);
  useEffect(() => {
    const controller = new AbortController();
    setUser(null);
    fetch(`/api/users/${id}`, { signal: safe ? controller.signal : undefined })
      .then((r) => r.json())
      .then(setUser)
      .catch((e) => { if (e.name !== "AbortError") setError(e); });
    if (safe) return () => controller.abort();        // the cleanup cancels the request we no longer want
  }, [id, safe]);
  if (error) return <p>Error: {error.message}</p>;
  return <p id={safe ? "safe" : "unsafe"}>{safe ? "safe" : "unsafe"}: user {id} is {user ? user.name : "loading…"}</p>;
}

function App() {
  const [id, setId] = useState(1);
  return <>
    <button onClick={() => setId(2)}>show user 2</button>
    <Profile id={id} safe={false} />
    <Profile id={id} safe />
  </>;
}
createRoot(document.getElementById("root")).render(<App />);
```

```console
$ python3 ../tools/page.py http://localhost:5173/race/ wait:50 "click:text=show user 2" wait:300 "eval:[...document.querySelectorAll('p')].map((p) => p.textContent)" wait:500 "eval:[...document.querySelectorAll('p')].map((p) => p.textContent)" --serve "npx vite --port 5173 --strictPort"
=> ["unsafe: user 2 is Bo", "safe: user 2 is Bo"]
=> ["unsafe: user 2 is Ana", "safe: user 2 is Bo"]
```

At first both components show Bo. Half a second later, the unsafe component shows "user 2 is Ana": the slow response for user 1 arrived last and overwrote the right answer. The safe component aborted that request when its effect was cleaned up, so the late response never arrived.

### Deep & Tricky

**DT-7. Effects do not run on the server, and they run only after the component has rendered and committed — so fetching in an effect creates "waterfalls": the parent renders, then fetches, then renders a child, which then starts its own fetch. Every level adds a full round trip, and nothing starts until the JavaScript has loaded. Server components and route loaders (Chapter 12) start all fetches as early as possible, in parallel.** *(React documentation, "Synchronizing with Effects — What are good alternatives to data fetching in Effects?")*

### Mastery check

1. **Predict.** In the unsafe component, what would the user see if the response for user 1 failed instead of succeeding late?

## Chapter 3 Project — Search as you type

### Requirements

Build a search box that queries `/api/search` as the user types: wait until typing pauses for 150 ms (debounce), never let an outdated response replace a newer one (abort it), keep showing the previous results while a new search runs (marked `aria-busy`), announce the result count in a live region, and show errors. Package the logic as two custom hooks. The fake API answers shorter queries more slowly, so that out-of-order responses really happen.

### Reference solution

```js
// file: project/hooks.js
import { useEffect, useState } from "react";

/** The value, but only after it has stopped changing for `delay` ms. */
export function useDebouncedValue(value, delay) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const t = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(t);                  // a newer value cancels the pending one
  }, [value, delay]);
  return debounced;
}

/** Fetch JSON for a URL (or nothing for null). Outdated requests are aborted, so a slow old
 *  response can never overwrite a newer one. Returns { data, error, loading }. */
export function useJson(url) {
  const [state, setState] = useState({ data: null, error: null, loading: false });
  useEffect(() => {
    if (url === null) { setState({ data: null, error: null, loading: false }); return; }
    const controller = new AbortController();
    setState((s) => ({ ...s, loading: true, error: null }));      // keep showing old data while loading
    fetch(url, { signal: controller.signal })
      .then((r) => { if (!r.ok) throw new Error(`HTTP ${r.status}`); return r.json(); })
      .then((data) => setState({ data, error: null, loading: false }))
      .catch((error) => {
        if (error.name === "AbortError") console.log(`aborted ${url}`);
        else setState({ data: null, error, loading: false });
      });
    return () => controller.abort();
  }, [url]);
  return state;
}
```

```jsx
// file: project/main.jsx
import { StrictMode, useState } from "react";
import { createRoot } from "react-dom/client";
import { useDebouncedValue, useJson } from "./hooks.js";

function Search() {
  const [text, setText] = useState("");
  const query = useDebouncedValue(text.trim(), 150);
  const { data, error, loading } = useJson(query ? `/api/search?q=${encodeURIComponent(query)}` : null);
  return (
    <search>
      <label>Search packages <input value={text} onChange={(e) => setText(e.target.value)} /></label>
      <p aria-live="polite">{loading ? "Searching…" : error ? `Failed: ${error.message}` : data ? `${data.results.length} results for "${data.q}"` : ""}</p>
      {data && <ul aria-busy={loading}>{data.results.map((r) => <li key={r}>{r}</li>)}</ul>}
    </search>
  );
}
createRoot(document.getElementById("root")).render(<StrictMode><Search /></StrictMode>);
```

The driver types "r", pauses long enough for that search to start, then quickly types "eac":

```console
$ python3 ../tools/page.py http://localhost:5173/project/ "click:input" press:r wait:170 press:e press:a press:c wait:600 "eval:[document.querySelector('[aria-live]').textContent, [...document.querySelectorAll('li')].map((li) => li.textContent).join(' ')]" --serve "npx vite --port 5173 --strictPort" --server-log
  api: search "r"
  api: search "reac"
aborted /api/search?q=r
=> ["1 results for \"reac\"", "react"]
```

Only two requests were made for four keystrokes: "r", and "reac" (the debounce absorbed "re" and "rea"). The slow "r" search was still running when "reac" started, and it was aborted, so its late answer could not replace the results for "reac".

### Review checklist

- Results for "re" are fetched again if the user deletes back to "re". Add a small cache keyed by URL. What else would you need — expiry, a size limit, invalidation? (Chapter 8 does it properly.)
- Show a "searching…" indicator only if the search takes longer than 300 ms, to avoid flashing.
- What should happen when the user presses Enter before the debounce has fired?

## Answers to the Chapter 3 prediction questions

**Unit 3.5, Q1.** The `.catch` would call `setError`, so the unsafe component would replace Bo's correct profile with an error message for a request the user no longer cares about — the same race, with a different symptom.

# Chapter 4 — Forms, Controlled Inputs, and Actions

*Goal: handle user input the React way, and the React 19 way. You will compare controlled inputs (React state is the truth) with uncontrolled ones (the DOM is the truth), meet the warnings that mixing them produces, and then use form actions — `<form action={fn}>`, `useActionState`, `useFormStatus`, and `useOptimistic` — which handle pending states and optimistic updates for you, but also reset the form after every submission, including failed ones. The project is a comment form with server-side validation, accessible errors, preserved input, and an optimistic comment.*

**What you will be able to do after this chapter**

- Build controlled and uncontrolled forms, and choose between them.
- Avoid the read-only-field and controlled/uncontrolled-switch mistakes.
- Submit forms with actions and track pending state with `useActionState` and `useFormStatus`.
- Show optimistic updates with `useOptimistic`, and roll them back on failure.
- Display server validation errors accessibly without losing the user's input.

```js
// file: vite.config.js
// This chapter's fake API: POST /api/comments validates and stores comments, slowly.
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

const comments = [{ id: 1, author: "Ana", text: "First!" }];
function fakeApi() {
  return {
    name: "fake-api",
    configureServer(server) {
      server.middlewares.use("/api/comments", async (req, res) => {
        res.setHeader("Content-Type", "application/json");
        if (req.method === "GET") return res.end(JSON.stringify(comments));
        let body = ""; for await (const c of req) body += c;
        const { author = "", text = "" } = JSON.parse(body || "{}");
        const errors = {};
        if (!author.trim()) errors.author = "Please give your name.";
        if (text.trim().length < 5) errors.text = "A comment needs at least 5 characters.";
        if (/spam/i.test(text)) errors.text = "That looks like spam.";
        await new Promise((r) => setTimeout(r, 400));                 // a slow server
        console.log(`  api: POST comment by "${author}" -> ${Object.keys(errors).length ? "422" : "201"}`);
        if (Object.keys(errors).length) { res.statusCode = 422; return res.end(JSON.stringify({ errors })); }
        const comment = { id: comments.length + 1, author: author.trim(), text: text.trim() };
        comments.push(comment);
        res.statusCode = 201; res.end(JSON.stringify(comment));
      });
    },
  };
}
export default defineConfig({
  plugins: [react(), fakeApi()],
  logLevel: "warn",
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client", "react-dom"] },
});
```

```html
<!-- file: page.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Keys</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```console
$ for d in controlled actions project; do mkdir -p $d && cp page.html $d/index.html; done
```

## Topic 4A — Inputs

## Unit 4.1 — Controlled and uncontrolled inputs

**One-line summary:** A *controlled* input gets its `value` (or `checked`) from React state and reports every change through `onChange`, so React can transform, validate, or reject each keystroke; an *uncontrolled* input keeps its own value in the DOM, with only a `defaultValue` from React, and is read when needed — typically through `FormData` on submit.

```jsx
// file: controlled/main.jsx
// Controlled inputs: React state is the single source of truth, and every keystroke goes through it.
import { useState } from "react";
import { createRoot } from "react-dom/client";

function Controlled() {
  const [name, setName] = useState("");
  const [qty, setQty] = useState(1);
  const [agree, setAgree] = useState(false);
  const [size, setSize] = useState("m");
  return (
    <form id="controlled" onSubmit={(e) => { e.preventDefault(); console.log(JSON.stringify({ name, qty, agree, size })); }}>
      <input aria-label="Name" value={name} onChange={(e) => setName(e.target.value.toUpperCase())} />
      <input aria-label="Quantity" type="number" value={qty} onChange={(e) => setQty(e.target.valueAsNumber || 0)} />
      <input aria-label="Agree" type="checkbox" checked={agree} onChange={(e) => setAgree(e.target.checked)} />
      <select aria-label="Size" value={size} onChange={(e) => setSize(e.target.value)}>
        <option value="s">S</option><option value="m">M</option><option value="l">L</option>
      </select>
      <button>Submit controlled</button>
    </form>
  );
}

function Uncontrolled() {
  // The DOM holds the values; React only sets the defaults. Read them once, on submit.
  return (
    <form id="uncontrolled" onSubmit={(e) => { e.preventDefault(); console.log(JSON.stringify(Object.fromEntries(new FormData(e.currentTarget)))); }}>
      <input name="name" aria-label="Name 2" defaultValue="Ana" />
      <input name="agree" type="checkbox" aria-label="Agree 2" defaultChecked />
      <button>Submit uncontrolled</button>
    </form>
  );
}

function Frozen() {
  return <input aria-label="Frozen" value="can't change me" />;     // value without onChange: read-only
}

function Switching() {
  const [v, setV] = useState();                                       // undefined at first: uncontrolled!
  return <input aria-label="Switching" value={v} onChange={(e) => setV(e.target.value)} />;
}

createRoot(document.getElementById("root")).render(<><Controlled /><Uncontrolled /><Frozen /><Switching /></>);
```

```console
$ python3 ../tools/page.py http://localhost:5173/controlled/ "fill:[aria-label=Name]|ana" "fill:[aria-label=Quantity]|3" "click:[aria-label=Agree]" "select:[aria-label=Size]|l" "click:text=Submit controlled" "fill:[aria-label='Name 2']|Bo" "click:text=Submit uncontrolled" "click:[aria-label=Frozen]" press:x "eval:document.querySelector('[aria-label=Frozen]').value" "fill:[aria-label=Switching]|hi" --serve "npx vite --port 5173 --strictPort"
You provided a `value` prop to a form field without an `onChange` handler. This will render a read-only field. If the field should be mutable use `defaultValue`. Otherwise, set either `onChange` or `readOnly`.
{"name":"ANA","qty":3,"agree":true,"size":"l"}
{"name":"Bo","agree":"on"}
=> "can't change me"
A component is changing an uncontrolled input to be controlled. This is likely caused by the value changing from undefined to a defined value, which should not happen. Decide between using a controlled or uncontrolled input element for the lifetime of the component. More info: https://react.dev/link/controlled-components
```

The controlled form turned "ana" into "ANA" as it was typed, and its state is already a proper object of a string, a number, a boolean, and a string. The uncontrolled form needed no state at all; `FormData` gives strings (`"on"` for the checkbox). React warned twice: a `value` without `onChange` makes a read-only field (the typed "x" was rejected), and a field whose value started as `undefined` and later became a string switched from uncontrolled to controlled.

### Deep & Tricky

**DT-8. `value={undefined}` and `value={null}` mean "uncontrolled", so `useState()` with no initial value, or data that arrives later from a fetch (`value={user?.name}`), makes an input uncontrolled at first and controlled afterwards — React warns, and the input may keep the old DOM value. Initialise controlled fields to `""` (and checkboxes to `false`).** *(React documentation, `<input>` reference — "A component is changing an uncontrolled input to be controlled": "make sure that value is never null or undefined")*

### Mastery check

1. **Explain.** The Quantity field uses `valueAsNumber || 0`. What does the user experience when they clear the field to type a new number, and how would you fix it?

## Topic 4B — Actions

## Unit 4.2 — Form actions, `useActionState`, and `useFormStatus`

**One-line summary:** In React 19, a function passed to `<form action>` receives the form's `FormData` when the form is submitted; React runs it inside a transition, so `useActionState(fn, initial)` gives the action's latest result and an `isPending` flag, any component inside the form can read the submission status with `useFormStatus`, and when the action completes React resets the form's uncontrolled fields.

```jsx
// file: actions/main.jsx
// React 19 form actions: pass a function as the form's action; React calls it with the FormData,
// tracks the pending state, and resets the (uncontrolled) form when the action finishes.
import { useActionState } from "react";
import { useFormStatus } from "react-dom";
import { createRoot } from "react-dom/client";

const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

function SubmitButton() {
  const { pending } = useFormStatus();            // the status of the nearest parent <form>
  return <button disabled={pending}>{pending ? "Saving…" : "Save"}</button>;
}

async function save(previous, formData) {         // (previous state, form data) -> next state
  await sleep(300);
  const title = formData.get("title");
  if (title.length < 3) return { ...previous, error: "Title needs at least 3 characters" };
  return { saved: [...previous.saved, title], error: null };
}

function Notes() {
  const [state, action, isPending] = useActionState(save, { saved: [], error: null });
  return (
    <form action={action}>
      <input name="title" aria-label="Title" />
      <SubmitButton />
      <p id="status">{isPending ? "pending" : "idle"}; saved: {state.saved.join(", ") || "none"}; error: {state.error ?? "none"}</p>
    </form>
  );
}
createRoot(document.getElementById("root")).render(<Notes />);
```

```console
$ python3 ../tools/page.py http://localhost:5173/actions/ "fill:input|Groceries" "click:button" wait:100 "eval:[document.getElementById('status').textContent, document.querySelector('button').textContent, document.querySelector('input').value]" wait:400 "eval:[document.getElementById('status').textContent, document.querySelector('button').textContent, document.querySelector('input').value]" "fill:input|ab" "click:button" wait:400 "eval:[document.getElementById('status').textContent, document.querySelector('button').textContent, document.querySelector('input').value]" --serve "npx vite --port 5173 --strictPort"
=> ["pending; saved: none; error: none", "Saving…", "Groceries"]
=> ["idle; saved: Groceries; error: none", "Save", ""]
=> ["idle; saved: Groceries; error: Title needs at least 3 characters", "Save", ""]
```

While the action ran, the state said "pending" and the button — which knows nothing about the form except that it is inside one — said "Saving…". After success, the field was cleared automatically. After the *failed* submission of "ab", the field was cleared too: the user's input is gone, next to an error message about it.

### Deep & Tricky

**DT-9. React resets a form after its action finishes, whether the action "succeeded" or returned an error — an action that returns validation errors therefore erases what the user typed. Return the submitted values with the errors and use them as the fields' `defaultValue` (as the project does), call `event.preventDefault()` in an `onSubmit` handler and start the action yourself, or make the fields controlled.** *(React documentation, `<form>` reference: "When a function is passed to action… React will automatically reset the form for uncontrolled components"; `requestFormReset` in `react-dom`)*

## Unit 4.3 — Optimistic updates

**One-line summary:** `useOptimistic(state, update)` returns a version of the state that includes temporary changes made while an action is pending — such as a new comment shown before the server confirms it — and automatically falls back to the real state when the action finishes, so a failed request simply makes the optimistic item disappear.

The project shows both outcomes.

### Pattern

**P-4. Validate on the server always (it is the only validation that cannot be bypassed), and on the client for immediate feedback; return machine-readable, per-field errors (HTTP 422 with `{ errors: { field: message } }`), show them next to the fields with `aria-invalid` and `aria-describedby`, and never discard the user's input because of an error.** *(Book 1, Chapter 1 project and Unit 14.2; WCAG 2.2 Success Criterion 3.3.1 "Error Identification" and 3.3.3 "Error Suggestion")* *Trade-off:* optimistic updates make interfaces feel instant but must be undone convincingly on failure; use them where failure is rare and cheap to explain.

## Chapter 4 Project — A comment form

### Requirements

Build a comment section that lists comments and posts new ones to `/api/comments` (this chapter's fake API validates, answers slowly, and returns 422 with per-field errors). Use a form action with `useActionState`; show "Posting…" on the button with `useFormStatus`; show the new comment immediately with `useOptimistic`, marked as sending; show server errors next to their fields accessibly; keep the user's input when validation fails, and keep the name but clear the comment after success.

### Reference solution

```jsx
// file: project/comments.jsx
// A comment form with React 19 actions: server-side validation errors next to each field,
// the typed text kept when validation fails, a pending button, and an optimistic comment
// that appears at once and is replaced (or removed) when the server answers.
import { startTransition, useActionState, useOptimistic, useState } from "react";
import { useFormStatus } from "react-dom";

async function postComment(comment) {
  const res = await fetch("/api/comments", { method: "POST", body: JSON.stringify(comment) });
  const data = await res.json();
  if (res.status === 422) return { errors: data.errors };
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return { comment: data };
}

function Submit() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? "Posting…" : "Post comment"}</button>;
}

export function Comments({ initial }) {
  const [comments, setComments] = useState(initial);
  const [optimistic, addOptimistic] = useOptimistic(comments, (list, c) => [...list, { ...c, pending: true }]);

  const [form, formAction] = useActionState(async (_previous, data) => {
    const values = { author: data.get("author"), text: data.get("text") };
    addOptimistic({ id: "pending", ...values });                         // show it now ...
    const result = await postComment(values);
    if (result.errors) return { values, errors: result.errors };         // ... it vanishes: keep the input
    startTransition(() => setComments((list) => [...list, result.comment]));   // ... replaced by the real one
    return { values: { author: values.author, text: "" }, errors: {} };  // keep the name, clear the text
  }, { values: { author: "", text: "" }, errors: {} });

  const field = (name, label, Tag = "input") => (
    <p>
      <label htmlFor={name}>{label}</label>{" "}
      <Tag id={name} name={name} defaultValue={form.values[name]} aria-invalid={Boolean(form.errors[name])}
           aria-describedby={form.errors[name] ? `${name}-error` : undefined} />
      {form.errors[name] && <span id={`${name}-error`} role="alert"> {form.errors[name]}</span>}
    </p>
  );
  return (
    <section aria-labelledby="c-h">
      <h2 id="c-h">Comments ({optimistic.length})</h2>
      <ul>{optimistic.map((c) => <li key={c.id} aria-busy={c.pending ? true : undefined}>
        <b>{c.author}</b>: {c.text}{c.pending && " (sending…)"}</li>)}</ul>
      <form action={formAction}>
        {field("author", "Name")}
        {field("text", "Comment", "textarea")}
        <Submit />
      </form>
    </section>
  );
}
```

```jsx
// file: project/main.jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { Comments } from "./comments.jsx";

const initial = await fetch("/api/comments").then((r) => r.json());
createRoot(document.getElementById("root")).render(<StrictMode><Comments initial={initial} /></StrictMode>);
```

A spam comment first, then a real one:

```console
$ python3 ../tools/page.py http://localhost:5173/project/ "fill:#author|Bo" "fill:#text|spam spam" "click:button" wait:100 "eval:[...document.querySelectorAll('li')].map((li) => li.textContent).concat(['author=' + document.getElementById('author').value, 'text=' + document.getElementById('text').value, 'button=' + document.querySelector('button').textContent])" wait:500 "eval:[...document.querySelectorAll('li')].map((li) => li.textContent).concat(['author=' + document.getElementById('author').value, 'text=' + document.getElementById('text').value, 'button=' + document.querySelector('button').textContent])" "aria:form" "fill:#text|Nice article, thanks." "click:button" wait:100 "eval:[...document.querySelectorAll('li')].map((li) => li.textContent).concat(['author=' + document.getElementById('author').value, 'text=' + document.getElementById('text').value, 'button=' + document.querySelector('button').textContent])" wait:500 "eval:[...document.querySelectorAll('li')].map((li) => li.textContent).concat(['author=' + document.getElementById('author').value, 'text=' + document.getElementById('text').value, 'button=' + document.querySelector('button').textContent])" --serve "npx vite --port 5173 --strictPort" --server-log
=> ["Ana: First!", "Bo: spam spam (sending…)", "author=Bo", "text=spam spam", "button=Posting…"]
  api: POST comment by "Bo" -> 422
Failed to load resource: the server responded with a status of 422 (Unprocessable Entity)
=> ["Ana: First!", "author=Bo", "text=spam spam", "button=Post comment"]
- paragraph:
  - text: Name
  - textbox "Name": Bo
- paragraph:
  - text: Comment
  - textbox "Comment" [invalid]: spam spam
  - alert: That looks like spam.
- button "Post comment"
=> ["Ana: First!", "Bo: Nice article, thanks. (sending…)", "author=Bo", "text=Nice article, thanks.", "button=Posting…"]
  api: POST comment by "Bo" -> 201
=> ["Ana: First!", "Bo: Nice article, thanks.", "author=Bo", "text=", "button=Post comment"]
```

During each request, the optimistic comment was in the list, marked "(sending…)", and the button said "Posting…". The spam comment was rejected with 422: the optimistic item vanished, the error appeared as an alert linked to the comment field, and both fields still held what the user typed. The real comment replaced its optimistic copy, the comment field was cleared, and the name was kept for the next comment.

### Review checklist

- What happens if the network fails (an exception rather than a 422)? Add an error boundary or an error state, and test it by stopping the server.
- Two comments posted quickly: are they both shown optimistically? In which order do they settle?
- Make the form work without JavaScript, so that it posts to a real endpoint and re-renders on the server. (Chapter 12's server actions do exactly this.)

## Answers to the Chapter 4 prediction questions

**Unit 4.1, Q1.** Clearing the field makes `valueAsNumber` `NaN`, so the state becomes 0 and the field immediately shows "0" again — the user cannot delete it to type a fresh number. Keep the raw string in state (`value={qtyText}`) and convert it when using or submitting it.

# Chapter 5 — Reducers, Context, and External Stores

*Goal: manage state that many components share. You will move update logic into reducers that can be tested without React, pass values through the tree with context — and count exactly which components re-render when it changes — then use external stores (Zustand and Redux Toolkit) whose selectors re-render only the components whose slice changed. The project builds such a store from scratch on `useSyncExternalStore`, including the selector mistake that sends React into an infinite loop.*

**What you will be able to do after this chapter**

- Write reducers and test them as plain functions.
- Use context for values that many components need, and know its re-rendering cost.
- Use Zustand and Redux Toolkit with selectors.
- Implement an external store with `useSyncExternalStore`, and write correct selectors.
- Choose between local state, lifted state, context, and an external store.

```js
// file: vite.config.js
// One Vite configuration per chapter: every folder with an index.html is a page at /folder/.
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  logLevel: "warn",
  // Pre-bundle React when the server starts. Otherwise Vite discovers it on the first page load,
  // bundles it, and reloads the page, so everything in it runs twice.
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client"] },
});
```

```html
<!-- file: page.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Keys</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```console
$ cd .. && npm install zustand@5 @reduxjs/toolkit@2 react-redux@9 2>&1 | tail -1
found 0 vulnerabilities
$ for d in context stores project; do mkdir -p $d && cp page.html $d/index.html; done
```

## Topic 5A — Reducers

## Unit 5.1 — `useReducer`

**One-line summary:** A reducer is a pure function `(state, action) → newState` that holds all the update logic of a piece of state in one place; `useReducer(reducer, initial)` gives the state and a `dispatch` function, and components describe *what happened* (`{ type: "added", text }`) instead of computing new state themselves — which also makes the logic testable without React.

```js
// file: reducer/todos.js
// A reducer: (state, action) -> new state. Pure, so it can be tested without React at all.
export const initialTodos = { items: [], filter: "all", nextId: 1 };

export function todosReducer(state, action) {
  switch (action.type) {
    case "added":
      if (!action.text.trim()) return state;                         // unchanged state: no re-render
      return { ...state, nextId: state.nextId + 1,
               items: [...state.items, { id: state.nextId, text: action.text.trim(), done: false }] };
    case "toggled":
      return { ...state, items: state.items.map((t) => (t.id === action.id ? { ...t, done: !t.done } : t)) };
    case "cleared":
      return { ...state, items: state.items.filter((t) => !t.done) };
    case "filtered":
      return { ...state, filter: action.filter };
    default:
      throw new Error(`unknown action ${action.type}`);              // a typo should fail loudly
  }
}

export const visibleTodos = ({ items, filter }) =>
  filter === "all" ? items : items.filter((t) => t.done === (filter === "done"));
```

```js
// file: reducer/test.mjs
import assert from "node:assert/strict";
import { todosReducer, initialTodos, visibleTodos } from "./todos.js";

const actions = [{ type: "added", text: "milk" }, { type: "added", text: "  " }, { type: "added", text: "eggs" },
                 { type: "toggled", id: 1 }, { type: "filtered", filter: "open" }];
const final = actions.reduce(todosReducer, initialTodos);         // replaying actions = Array.reduce
console.log(JSON.stringify(final));
console.log("visible:", visibleTodos(final).map((t) => t.text));
assert.equal(todosReducer(initialTodos, { type: "added", text: " " }), initialTodos);   // same object back
assert.throws(() => todosReducer(initialTodos, { type: "add" }), /unknown action add/);
console.log("reducer tests passed");
```

```console
$ node reducer/test.mjs
{"items":[{"id":1,"text":"milk","done":true},{"id":2,"text":"eggs","done":false}],"filter":"open","nextId":3}
visible: [ 'eggs' ]
reducer tests passed
```

Replaying a list of actions is just `Array.prototype.reduce` — where the name comes from. Returning the *same* state object for a no-op lets React skip the re-render. In a component: `const [state, dispatch] = useReducer(todosReducer, initialTodos)` and `onClick={() => dispatch({ type: "toggled", id })}`.

## Topic 5B — Context

## Unit 5.2 — Context and its re-renders

**One-line summary:** `createContext(default)` creates a channel; a provider (`<Ctx value={…}>` in React 19) supplies a value to every component below it, which reads it with `use(Ctx)` or `useContext(Ctx)`; whenever the provided value changes (by `Object.is`), every component that reads that context re-renders — even those that use a part that did not change, and even through `memo`.

```jsx
// file: context/main.jsx
// Context passes a value to any depth without props. Every consumer re-renders when the value
// changes — and an object created during render is a new value every time.
import { createContext, memo, use, useMemo, useState } from "react";
import { createRoot } from "react-dom/client";

const renders = {};
const count = (name) => { renders[name] = (renders[name] ?? 0) + 1; };
window.report = () => ({ ...renders });

const Settings = createContext({ theme: "light", user: null });    // the default: used with no provider

const ThemeLabel = memo(function ThemeLabel() {                     // memo: skip re-render if props are equal
  const { theme } = use(Settings);                                  // React 19: use(Context) = useContext
  count("ThemeLabel");
  return <p>theme: {theme}</p>;
});
const UserLabel = memo(function UserLabel() {
  const { user } = use(Settings);
  count("UserLabel");
  return <p>user: {user}</p>;
});
const Static = memo(function Static() { count("Static"); return <p>no context here</p>; });

function App({ stable }) {
  const [theme, setTheme] = useState("light");
  const [user, setUser] = useState("ana");
  const [clicks, setClicks] = useState(0);
  const fresh = { theme, user };                                    // a new object on every render
  const memoised = useMemo(() => ({ theme, user }), [theme, user]); // the same object until they change
  return (
    <Settings value={stable ? memoised : fresh}>
      <button onClick={() => setUser(user === "ana" ? "bo" : "ana")}>switch user</button>
      <button onClick={() => setClicks(clicks + 1)}>unrelated update</button>
      <ThemeLabel /><UserLabel /><Static />
    </Settings>
  );
}
const stable = new URLSearchParams(location.search).has("stable");
createRoot(document.getElementById("root")).render(<App stable={stable} />);
```

```console
$ python3 ../tools/page.py "http://localhost:5173/context/" "eval:report()" "click:text=unrelated update" "eval:report()" "click:text=switch user" "eval:report()" --serve "npx vite --port 5173 --strictPort"
=> {"ThemeLabel": 1, "UserLabel": 1, "Static": 1}
=> {"ThemeLabel": 2, "UserLabel": 2, "Static": 1}
=> {"ThemeLabel": 3, "UserLabel": 3, "Static": 1}
$ python3 ../tools/page.py "http://localhost:5173/context/?stable" "eval:report()" "click:text=unrelated update" "eval:report()" "click:text=switch user" "eval:report()" --serve "npx vite --port 5173 --strictPort"
=> {"ThemeLabel": 1, "UserLabel": 1, "Static": 1}
=> {"ThemeLabel": 1, "UserLabel": 1, "Static": 1}
=> {"ThemeLabel": 2, "UserLabel": 2, "Static": 1}
```

With a fresh object as the value, an unrelated update in `App` re-rendered both consumers, although neither theme nor user had changed — `memo` cannot help, because context bypasses props. With the memoised value, the unrelated update re-rendered nothing; switching the user re-rendered both consumers — including `ThemeLabel`, which only reads the theme. `Static`, which reads no context, never re-rendered.

### Mastery check

1. **Predict.** With the memoised value, a button that changed the theme from "light" to "dark" is added. Which components re-render when it is clicked?

### Deep & Tricky

**DT-10. Context has no selectors: a component that reads one field of a context value re-renders whenever *any* field changes. Splitting one context into several (theme, user, dispatch) or moving fast-changing state into an external store with selectors is the remedy; memoising the value object only removes the re-renders caused by the provider's own re-renders.** *(React documentation, `useContext` reference — "Optimizing re-renders when passing objects and functions"; `use(Context)` behaves the same way)*

## Topic 5C — External stores

## Unit 5.3 — Zustand and Redux Toolkit

**One-line summary:** External stores keep state outside the React tree and let each component subscribe to a *selected* slice — re-rendering only when that slice changes; Zustand makes the store itself a hook, and Redux Toolkit organises state into slices of reducers (written with Immer, so "mutating" code produces immutable updates) dispatched through a single store.

```jsx
// file: stores/main.jsx
// External stores keep state outside React; components subscribe to the slice they select,
// and re-render only when THAT slice changes. The same feature with Zustand and Redux Toolkit.
import { memo } from "react";
import { createRoot } from "react-dom/client";
import { create } from "zustand";
import { configureStore, createSlice } from "@reduxjs/toolkit";
import { Provider, useDispatch, useSelector } from "react-redux";

const renders = {};
const count = (name) => { renders[name] = (renders[name] ?? 0) + 1; };
window.report = () => ({ ...renders });

// --- Zustand: a hook that IS the store ---
const useSettings = create((set) => ({
  theme: "light",
  user: "ana",
  switchUser: () => set((s) => ({ user: s.user === "ana" ? "bo" : "ana" })),   // returns a partial update
}));
const ZTheme = memo(function ZTheme() { const theme = useSettings((s) => s.theme); count("zustand theme"); return <p>{theme}</p>; });
const ZUser = memo(function ZUser() { const user = useSettings((s) => s.user); count("zustand user"); return <p>{user}</p>; });
function ZButton() { const switchUser = useSettings((s) => s.switchUser); return <button onClick={switchUser}>zustand: switch user</button>; }

// --- Redux Toolkit: a slice of reducer logic, a store, and react-redux hooks ---
const settings = createSlice({
  name: "settings",
  initialState: { theme: "light", user: "ana" },
  reducers: { userSwitched: (s) => { s.user = s.user === "ana" ? "bo" : "ana"; } },   // Immer: "mutations" are safe here
});
const store = configureStore({ reducer: { settings: settings.reducer } });
const RTheme = memo(function RTheme() { const theme = useSelector((s) => s.settings.theme); count("redux theme"); return <p>{theme}</p>; });
const RUser = memo(function RUser() { const user = useSelector((s) => s.settings.user); count("redux user"); return <p>{user}</p>; });
function RButton() { const dispatch = useDispatch(); return <button onClick={() => dispatch(settings.actions.userSwitched())}>redux: switch user</button>; }

createRoot(document.getElementById("root")).render(<>
  <ZButton /><ZTheme /><ZUser />
  <Provider store={store}><RButton /><RTheme /><RUser /></Provider>
</>);
window.reduxState = () => store.getState();
```

```console
$ python3 ../tools/page.py "http://localhost:5173/stores/" "eval:report()" "click:text=zustand: switch user" "click:text=redux: switch user" "click:text=redux: switch user" "eval:report()" "eval:reduxState()" --serve "npx vite --port 5173 --strictPort"
=> {"zustand theme": 1, "zustand user": 1, "redux theme": 1, "redux user": 1}
=> {"zustand theme": 1, "zustand user": 2, "redux theme": 1, "redux user": 3}
=> {"settings": {"theme": "light", "user": "ana"}}
```

Only the user components re-rendered; the theme components, which select only the theme, stayed at one render. Redux's reducer "assigned" `s.user`, but Immer turned that into a new state object, and after two switches the state is back where it began.

### Pattern

**P-5. Keep state as local as possible: in a component, then lifted to the nearest common parent, then in context for rarely changing global values (theme, locale, the signed-in user), and in an external store only for frequently changing state shared across distant parts of the app; and keep server data out of all of these — it belongs in a server-state cache (Chapter 8).** *(React documentation, "Managing State"; TanStack Query documentation, "Does TanStack Query replace Redux?")* *Trade-off:* external stores add a dependency, a second way of holding state, and a debugging surface; they pay off with many subscribers and frequent updates, not before.

## Chapter 5 Project — An external store from scratch

### Requirements

Implement `createStore(init)` with `get`, `set` (merging partial updates, skipping no-ops, never mutating), and `subscribe`; and a hook `useStore(store, selector, isEqual)` built on `useSyncExternalStore`, which re-renders a component only when its selected value changes. Build a cart store with derived selectors (item count, total with a coupon), and components that select a number, a derived number, an object (with shallow equality), and functions. Count renders, and show what happens with an object selector compared by identity.

### Reference solution

```js
// file: project/store.js
// An external store in 40 lines, the core of libraries such as Zustand.
import { useRef, useSyncExternalStore } from "react";

export function createStore(init) {
  let state;
  const listeners = new Set();
  const get = () => state;
  const set = (partial) => {
    const changes = typeof partial === "function" ? partial(state) : partial;
    const next = { ...state, ...changes };
    if (Object.keys(changes).every((k) => Object.is(next[k], state[k]))) return;   // nothing changed
    state = next;                                                                  // a new object: immutable
    for (const listener of listeners) listener();
  };
  const subscribe = (listener) => { listeners.add(listener); return () => listeners.delete(listener); };
  state = init(set, get);
  return { get, set, subscribe };
}

export const shallow = (a, b) =>
  Object.is(a, b) || (typeof a === "object" && typeof b === "object" && a && b &&
    Object.keys(a).length === Object.keys(b).length && Object.keys(a).every((k) => Object.is(a[k], b[k])));

/** Subscribe to the part of the store that `selector` picks; re-render only when it changes. */
export function useStore(store, selector = (s) => s, isEqual = Object.is) {
  const last = useRef(null);
  const getSnapshot = () => {
    const next = selector(store.get());
    if (last.current && isEqual(last.current.value, next)) return last.current.value;   // keep the old identity
    last.current = { value: next };
    return next;
  };
  return useSyncExternalStore(store.subscribe, getSnapshot, getSnapshot);
}
```

```js
// file: project/cart.js
import { createStore } from "./store.js";

export const cart = createStore((set) => ({
  items: [],                                       // [{ id, name, price, qty }]
  coupon: null,
  add: (product) => set((s) => {
    const existing = s.items.find((i) => i.id === product.id);
    return { items: existing
      ? s.items.map((i) => (i.id === product.id ? { ...i, qty: i.qty + 1 } : i))
      : [...s.items, { ...product, qty: 1 }] };
  }),
  applyCoupon: (code) => set({ coupon: code }),
}));
export const selectCount = (s) => s.items.reduce((n, i) => n + i.qty, 0);
export const selectTotal = (s) => s.items.reduce((t, i) => t + i.price * i.qty, 0) * (s.coupon === "HALF" ? 0.5 : 1);
```

```jsx
// file: project/main.jsx
import { memo } from "react";
import { createRoot } from "react-dom/client";
import { useStore, shallow } from "./store.js";
import { cart, selectCount, selectTotal } from "./cart.js";

const renders = {};
const count = (name) => { renders[name] = (renders[name] ?? 0) + 1; };
window.report = () => ({ ...renders });

const Badge = memo(function Badge() {
  const n = useStore(cart, selectCount);                     // a number: Object.is is enough
  count("Badge");
  return <p>{n} items</p>;
});
const Total = memo(function Total() {
  const total = useStore(cart, selectTotal);
  count("Total");
  return <p>total {(total / 100).toFixed(2)}</p>;
});
const Summary = memo(function Summary() {
  const { n, coupon } = useStore(cart, (s) => ({ n: selectCount(s), coupon: s.coupon }), shallow);   // an object: compare shallowly
  count("Summary");
  return <p>{n} items, coupon {coupon ?? "none"}</p>;
});
function Buttons() {
  const add = useStore(cart, (s) => s.add);                  // functions never change: never re-renders
  count("Buttons");
  return <p>
    <button onClick={() => add({ id: "tea", name: "Tea", price: 450 })}>add tea</button>
    <button onClick={() => cart.get().applyCoupon("HALF")}>apply coupon</button>
    <button onClick={() => cart.get().applyCoupon("HALF")}>apply the same coupon again</button>
  </p>;
}
createRoot(document.getElementById("root")).render(<><Buttons /><Badge /><Total /><Summary /></>);

// The classic mistake: a selector that returns a NEW object every time, compared with Object.is.
function Broken() {
  const { n } = useStore(cart, (s) => ({ n: selectCount(s) }));
  return <p>{n}</p>;
}
window.mountBroken = () => {
  const el = document.body.appendChild(document.createElement("div"));
  createRoot(el, { onUncaughtError: (e) => console.log(`Broken crashed: ${e.message.split(".")[0]}`) }).render(<Broken />);
};
```

```console
$ python3 ../tools/page.py "http://localhost:5173/project/" "eval:report()" "click:text=add tea" "click:text=add tea" "eval:report()" "click:text=apply coupon" "click:text=apply the same coupon again" "eval:report()" "eval:[...document.querySelectorAll('p')].slice(1).map((p) => p.textContent)" "eval:mountBroken()" wait:300 --serve "npx vite --port 5173 --strictPort"
=> {"Buttons": 1, "Badge": 1, "Total": 1, "Summary": 1}
=> {"Buttons": 1, "Badge": 3, "Total": 3, "Summary": 3}
=> {"Buttons": 1, "Badge": 3, "Total": 4, "Summary": 4}
=> ["2 items", "total 4.50", "2 items, coupon HALF"]
=> null
The result of getSnapshot should be cached to avoid an infinite loop
Broken crashed: Maximum update depth exceeded
```

Two "add tea" clicks re-rendered the three components whose selection changed; `Buttons` never re-rendered, because the `add` function in the store never changes. Applying the coupon re-rendered `Total` and `Summary` but not `Badge` (the count did not change); applying the same coupon again changed nothing at all — `set` skipped it. The `Broken` component selected `{ n }`, a new object on every call: `useSyncExternalStore` saw a "change" on every snapshot, warned that the snapshot must be cached, re-rendered, saw another change, and gave up with "Maximum update depth exceeded".

### Deep & Tricky

**DT-11. `useSyncExternalStore` calls `getSnapshot` repeatedly — on every render and on every store change — and requires that it return the *same* value (by `Object.is`) as long as the store has not changed; a selector that builds a new object or array (`s => ({ a: s.a })`, `s => s.items.filter(...)`) therefore causes an infinite re-render loop. Return primitives or existing objects, memoise derived objects, or compare with shallow equality (Zustand's `useShallow`, Redux's `shallowEqual`).** *(React documentation, `useSyncExternalStore` reference — "The result of getSnapshot should be cached"; Zustand v5 migration guide, "selectors returning new references")*

### Review checklist

- Our `useStore` creates a new `getSnapshot` function on every render. Is that a problem? What does `useSyncExternalStore` do when `subscribe` changes on every render?
- Add a `persist` option that saves selected keys to `localStorage` and restores them. Where must the restoring happen to avoid a hydration mismatch with server rendering? (Chapter 12.)
- Write unit tests for `createStore` and the selectors with `node:test` (Book 1, Chapter 15), without React.

## Answers to the Chapter 5 prediction questions

**Unit 5.2, Q1.** `ThemeLabel` and `UserLabel` (the value object changes, and both read the context), plus `App` itself; `Static` does not.

# Chapter 6 — Performance: Reconciliation, Memoisation, the Compiler, and Concurrency

*Goal: make React applications fast by understanding what React actually does on each update. You will count the DOM operations that reconciliation performs with good and bad keys, measure a typing box that re-renders 300 slow components on every keystroke and fix it three ways — `memo`, the React Compiler, and nothing else — read the compiler's output, keep typing responsive while an expensive list re-renders with `useDeferredValue`, and finally take a 10,000-row list from a one-second first render and 300-ms keystrokes to under 20 ms and a few dozen milliseconds with windowing.*

**What you will be able to do after this chapter**

- Explain reconciliation: how React compares element trees and what keys change.
- Find unnecessary re-renders with the Profiler and remove them with `memo`, `useMemo`, and `useCallback` — or let the React Compiler do it.
- Use transitions and deferred values to keep interactions responsive.
- Render very long lists with windowing.
- Measure before and after, with numbers that matter to users.

```js
// file: vite.config.js
// One Vite configuration per chapter: every folder with an index.html is a page at /folder/.
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  logLevel: "warn",
  // Pre-bundle React when the server starts. Otherwise Vite discovers it on the first page load,
  // bundles it, and reloads the page, so everything in it runs twice.
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client"] },
});
```

```js
// file: vite.compiler.config.js
// The same app, compiled with the React Compiler (through Babel).
import { defineConfig } from "vite";
import react, { reactCompilerPreset } from "@vitejs/plugin-react";
import babel from "@rolldown/plugin-babel";

export default defineConfig({
  plugins: [react(), babel({ presets: [reactCompilerPreset()] })],
  logLevel: "warn",
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client", "react/compiler-runtime"] },
});
```

```html
<!-- file: page.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Keys</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```js
// file: measure.js
// Helpers shared by this chapter's pages: a deliberately slow function, a render counter fed by
// React's <Profiler>, and the slowest interaction from the Event Timing API (Book 1, Chapter 13).
export const busy = (ms) => { const end = performance.now() + ms; while (performance.now() < end) {} };
const stats = { commits: 0, renderMs: 0, itemRenders: 0, worstInputMs: 0 };
export const countItem = () => { stats.itemRenders++; };
export const onRender = (_id, _phase, actualDuration) => { stats.commits++; stats.renderMs += actualDuration; };
new PerformanceObserver((list) => {
  for (const e of list.getEntries()) stats.worstInputMs = Math.max(stats.worstInputMs, e.duration);
}).observe({ type: "event", durationThreshold: 16, buffered: true });
window.stats = () => ({ ...stats, renderMs: Math.round(stats.renderMs) });
window.resetStats = () => Object.assign(stats, { commits: 0, renderMs: 0, itemRenders: 0, worstInputMs: 0 });
```

```console
$ for d in keys rerender deferred project; do mkdir -p $d && cp page.html $d/index.html; done
```

## Topic 6A — What React does on an update

## Unit 6.1 — Reconciliation

**One-line summary:** On every update React calls the affected components, compares the new element tree with the previous one position by position, and changes only the DOM that differs: a different element type at a position replaces the whole subtree, the same type is updated in place, and among siblings in a list, keys decide which old element each new element corresponds to — and therefore whether DOM nodes are moved or rewritten.

```jsx
// file: keys/main.jsx
// Reconciliation in numbers: what does React do to the DOM when a list is reversed?
import { useState } from "react";
import { createRoot } from "react-dom/client";

const ITEMS = Array.from({ length: 100 }, (_, i) => ({ id: i, text: `row ${i}` }));

function List({ byIndex }) {
  const [items, setItems] = useState(ITEMS);
  return <section id={byIndex ? "index" : "id"}>
    <button onClick={() => setItems([...items].reverse())}>reverse</button>
    <ul>{items.map((it, i) => <li key={byIndex ? i : it.id}>{it.text}</li>)}</ul>
  </section>;
}

createRoot(document.getElementById("root")).render(<><List byIndex /><List /></>);
window.watch = (id) => {
  const counts = { nodesMoved: 0, textChanged: 0 };
  new MutationObserver((records) => {
    for (const r of records) {
      if (r.type === "childList") counts.nodesMoved += r.addedNodes.length;
      if (r.type === "characterData") counts.textChanged++;
    }
  }).observe(document.querySelector(`#${id} ul`), { childList: true, characterData: true, subtree: true });
  return () => counts;
};
```

```console
$ python3 ../tools/page.py "http://localhost:5173/keys/" "eval:void (window.a = watch('index'), window.b = watch('id'))" "click:#index button" "click:#id button" wait:100 "eval:({ indexKeys: a(), idKeys: b() })" --serve "npx vite --port 5173 --strictPort"
=> null
=> {"indexKeys": {"nodesMoved": 0, "textChanged": 100}, "idKeys": {"nodesMoved": 99, "textChanged": 0}}
```

Reversing 100 rows keyed by index moved no nodes and rewrote all 100 texts: React matched old row 0 with new row 0 and updated its content. Keyed by ID, it moved 99 nodes and rewrote no text. For plain text the difference is small; for rows with state, focus, images, or animations, moving the right nodes is the difference between correct and broken (Chapter 1, Unit 1.2).

## Unit 6.2 — Re-renders, `memo`, and the React Compiler

**One-line summary:** When a component's state changes, React re-renders it *and all its descendants* by default, whether or not their props changed; `memo(Component)` skips a component whose props are all `Object.is`-equal to last time (which requires stable object and function props — `useMemo`, `useCallback`), and the React Compiler inserts this memoisation automatically, at a finer grain than people do by hand.

```jsx
// file: rerender/main.jsx
// Typing in a box re-renders its parent, and by default every child of the parent as well —
// here 300 slow list items that do not depend on the text at all.
import { memo, Profiler, useState } from "react";
import { createRoot } from "react-dom/client";
import { busy, countItem, onRender } from "../measure.js";

const ITEMS = Array.from({ length: 300 }, (_, i) => ({ id: i, label: `item ${i}` }));
const useMemoList = new URLSearchParams(location.search).has("memo");

function Item({ item }) { countItem(); busy(0.3); return <li>{item.label}</li>; }
function List({ items }) { return <ul>{items.map((it) => <Item key={it.id} item={it} />)}</ul>; }
const MemoList = memo(List);                               // skip re-rendering when props are equal

function App() {
  const [text, setText] = useState("");
  const ListComponent = useMemoList ? MemoList : List;
  return (
    <Profiler id="app" onRender={onRender}>
      <input aria-label="Note" value={text} onChange={(e) => setText(e.target.value)} />
      <ListComponent items={ITEMS} />
    </Profiler>
  );
}
createRoot(document.getElementById("root")).render(<App />);
```

Typing "hello" — without `memo`, with `memo`, and without `memo` but compiled by the React Compiler:

```console
$ python3 ../tools/page.py "http://localhost:5173/rerender/" "eval:resetStats()" "click:input" press:h press:e press:l press:l press:o wait:300 "eval:stats()" --serve "npx vite --port 5173 --strictPort"
=> {"commits": 0, "renderMs": 0, "itemRenders": 0, "worstInputMs": 0}
=> {"commits": 5, "renderMs": 527, "itemRenders": 1500, "worstInputMs": 128}
$ python3 ../tools/page.py "http://localhost:5173/rerender/?memo" "eval:resetStats()" "click:input" press:h press:e press:l press:l press:o wait:300 "eval:stats()" --serve "npx vite --port 5173 --strictPort"
=> {"commits": 0, "renderMs": 0, "itemRenders": 0, "worstInputMs": 0}
=> {"commits": 5, "renderMs": 0, "itemRenders": 0, "worstInputMs": 16}
$ python3 ../tools/page.py "http://localhost:5173/rerender/" "eval:resetStats()" "click:input" press:h press:e press:l press:l press:o wait:300 "eval:stats()" --serve "npx vite --port 5173 --strictPort --config vite.compiler.config.js"
=> {"commits": 0, "renderMs": 0, "itemRenders": 0, "worstInputMs": 0}
=> {"commits": 5, "renderMs": 1, "itemRenders": 0, "worstInputMs": 16}
```

Five keystrokes re-rendered 1,500 items — 300 per keystroke — spending over half a second, with the slowest keystroke taking more than 100 ms from key to paint. With `memo` around the list, the items did not render at all. The compiled version reached the same result with *unchanged source code*. What the compiler did:

```jsx
// file: compiled/product.jsx
export function ProductRow({ product, onAdd }) {
  const price = (product.price / 100).toFixed(2);
  return <li>{product.name} {price} <button onClick={() => onAdd(product.id)}>Add</button></li>;
}
```

```js
// file: compile.mjs
// Show what the React Compiler does to a component.
import { transformFileSync } from "@babel/core";
const { code } = transformFileSync(process.argv[2], {
  babelrc: false, configFile: false,
  plugins: [["babel-plugin-react-compiler", {}]],
  presets: [],
  parserOpts: { plugins: ["jsx"] },
});
console.log(code);
```

```console
$ node compile.mjs compiled/product.jsx
import { c as _c } from "react/compiler-runtime";
// file: compiled/product.jsx
export function ProductRow(t0) {
  const $ = _c(9);
  const {
    product,
    onAdd
  } = t0;
  const t1 = product.price / 100;
  let t2;
  if ($[0] !== t1) {
    t2 = t1.toFixed(2);
    $[0] = t1;
    $[1] = t2;
  } else {
    t2 = $[1];
  }
  const price = t2;
  let t3;
  if ($[2] !== onAdd || $[3] !== product.id) {
    t3 = <button onClick={() => onAdd(product.id)}>Add</button>;
    $[2] = onAdd;
    $[3] = product.id;
    $[4] = t3;
  } else {
    t3 = $[4];
  }
  let t4;
  if ($[5] !== price || $[6] !== product.name || $[7] !== t3) {
    t4 = <li>{product.name} {price} {t3}</li>;
    $[5] = price;
    $[6] = product.name;
    $[7] = t3;
    $[8] = t4;
  } else {
    t4 = $[8];
  }
  return t4;
}
```

The component got a cache of nine slots (`_c(9)`); each computed value and element is recomputed only if the values it depends on changed, so an unchanged `ProductRow` returns the *same* element object as last time, and React skips that subtree — the effect of `memo`, `useMemo`, and `useCallback` combined, applied everywhere.

### Deep & Tricky

**DT-12. The React Compiler can only memoise code that follows the rules of React — pure rendering, immutable props and state, hooks at the top level — and it silently skips (bails out of) components it cannot prove safe, such as those that mutate values created during render or read refs while rendering; such components keep working, unoptimised. Code that "worked" while breaking the rules can also change behaviour when compiled, because a mutation that used to be repeated on every render now happens only when inputs change.** *(React documentation, "React Compiler — Introduction" and "Rules of React"; the `react-compiler` ESLint rules report bail-outs)*

### Mastery check

1. **Predict.** `memo(List)` is used, but `App` renders `<MemoList items={ITEMS.filter(() => true)} />`. Do the items re-render on each keystroke?

## Topic 6B — Concurrency

## Unit 6.3 — Transitions and deferred values

**One-line summary:** React can render in the background and abandon unfinished work: updates marked as non-urgent — with `startTransition`/`useTransition`, or by rendering with a `useDeferredValue` — are rendered after urgent ones (typing, clicking), and are interrupted and restarted when newer input arrives, so an expensive re-render no longer blocks the keystroke that caused it.

```jsx
// file: deferred/main.jsx
// The list DOES depend on the text (it is filtered by it), so memo cannot skip it. useDeferredValue
// lets React render the input first and the expensive list afterwards, interruptibly.
import { memo, Profiler, useDeferredValue, useState } from "react";
import { createRoot } from "react-dom/client";
import { busy, countItem, onRender } from "../measure.js";

const ITEMS = Array.from({ length: 600 }, (_, i) => `item ${i}`);
const deferred = new URLSearchParams(location.search).has("deferred");

function Item({ label }) { countItem(); busy(0.2); return <li>{label}</li>; }
const Results = memo(function Results({ query }) {
  const visible = ITEMS.filter((label) => label.includes(query));
  return <ul>{visible.map((label) => <Item key={label} label={label} />)}</ul>;
});

function App() {
  const [text, setText] = useState("");
  const query = useDeferredValue(text);                      // lags behind `text` while it is changing
  const shown = deferred ? query : text;
  return (
    <Profiler id="app" onRender={onRender}>
      <input aria-label="Filter" value={text} onChange={(e) => setText(e.target.value)} />
      <p style={{ opacity: shown !== text ? 0.5 : 1 }}>{shown !== text ? "updating…" : "up to date"}</p>
      <Results query={shown} />
    </Profiler>
  );
}
createRoot(document.getElementById("root")).render(<App />);
```

```console
$ python3 ../tools/page.py "http://localhost:5173/deferred/" "eval:resetStats()" "click:input" press:i press:t press:e press:m press:Space press:5 wait:1500 "eval:({ ...stats(), shown: document.querySelectorAll('li').length, status: document.querySelector('p').textContent })" --serve "npx vite --port 5173 --strictPort"
=> {"commits": 0, "renderMs": 0, "itemRenders": 0, "worstInputMs": 0}
=> {"commits": 12, "renderMs": 920, "itemRenders": 3111, "worstInputMs": 192, "shown": 111, "status": "up to date"}
$ python3 ../tools/page.py "http://localhost:5173/deferred/?deferred" "eval:resetStats()" "click:input" press:i press:t press:e press:m press:Space press:5 wait:1500 "eval:({ ...stats(), shown: document.querySelectorAll('li').length, status: document.querySelector('p').textContent })" --serve "npx vite --port 5173 --strictPort"
=> {"commits": 0, "renderMs": 0, "itemRenders": 0, "worstInputMs": 0}
=> {"commits": 7, "renderMs": 41, "itemRenders": 111, "worstInputMs": 24, "shown": 111, "status": "up to date"}
```

Here the list depends on the text, so `memo` cannot skip it. Without deferral, every keystroke rendered all matching items synchronously — over 3,000 item renders, a second of rendering, and keystrokes delayed by up to about 200 ms. With the deferred query, React rendered each keystroke's input immediately, started rendering the list in the background, threw that work away when the next key arrived, and committed the list only for the final query: about a hundred item renders in total, and keystrokes painted within a frame or two. Both end with the same 111 rows.

### Pattern

**P-6. Optimise in this order: measure (React DevTools Profiler, the Performance panel, Core Web Vitals); render less (keys, state placed lower in the tree, children passed as props, windowing); render cheaper (memoisation — preferably by the React Compiler); and render later (transitions and deferred values) — and measure again after each change.** *(React documentation, "Profiler" and `useDeferredValue`; web.dev, "Interaction to Next Paint")* *Trade-off:* every memoisation costs memory and comparison time, and deferral shows temporarily stale UI; both are worth it only where measurements show a real cost.

## Chapter 6 Project — 10,000 rows

### Requirements

An order list has 10,000 rows and a filter box. Measure the naive version — first render time, number of DOM nodes, row renders and the slowest keystroke while typing — then build an optimised version with windowing (render only the visible rows in a scroll container), a memoised list, and a deferred query, and measure again. Check that scrolling to the bottom shows the last rows.

### Reference solution

```js
// file: project/data.js
export const ORDERS = Array.from({ length: 10_000 }, (_, i) => ({
  id: i + 1,
  customer: ["Ana", "Bo", "Cy", "Dee", "Eli"][i % 5] + " " + String.fromCharCode(65 + (i * 7) % 26),
  total: ((i * 7919) % 50000) / 100,
  status: ["paid", "shipped", "refunded"][i % 3],
}));
```

```jsx
// file: project/Virtual.jsx
// Windowing: render only the rows that are visible (plus a few extra), inside a spacer as tall as
// all the rows would be. 10,000 rows cost the same as 30.
import { useState } from "react";

export function VirtualList({ items, rowHeight, height, renderRow, overscan = 5 }) {
  const [scrollTop, setScrollTop] = useState(0);
  const first = Math.max(0, Math.floor(scrollTop / rowHeight) - overscan);
  const last = Math.min(items.length, Math.ceil((scrollTop + height) / rowHeight) + overscan);
  return (
    <div role="list" aria-rowcount={items.length} style={{ height, overflowY: "auto" }}
         onScroll={(e) => setScrollTop(e.currentTarget.scrollTop)} data-testid="viewport">
      <div style={{ height: items.length * rowHeight, position: "relative" }}>
        {items.slice(first, last).map((item, i) => (
          <div role="listitem" key={item.id} style={{ position: "absolute", top: (first + i) * rowHeight, height: rowHeight }}>
            {renderRow(item)}
          </div>
        ))}
      </div>
    </div>
  );
}
```

```jsx
// file: project/main.jsx
// The same order list, naive and optimised. Measure: first render, DOM size, typing latency.
import { memo, Profiler, useDeferredValue, useState } from "react";
import { createRoot } from "react-dom/client";
import { ORDERS } from "./data.js";
import { VirtualList } from "./Virtual.jsx";
import { countItem, onRender } from "../measure.js";

const fast = new URLSearchParams(location.search).get("mode") === "fast";

function Row({ order }) {
  countItem();
  return <span>#{order.id} {order.customer} — {order.total.toFixed(2)} ({order.status})</span>;
}
const match = (q) => (o) => o.customer.toLowerCase().includes(q) || o.status.startsWith(q);

const Naive = ({ query }) => {
  const rows = ORDERS.filter(match(query));
  return <ul>{rows.map((o) => <li key={o.id}><Row order={o} /></li>)}</ul>;
};
const Fast = memo(function Fast({ query }) {
  const rows = ORDERS.filter(match(query));
  return <VirtualList items={rows} rowHeight={24} height={480} renderRow={(o) => <Row order={o} />} />;
});

function App() {
  const [text, setText] = useState("");
  const query = useDeferredValue(text.toLowerCase());
  return (
    <Profiler id="orders" onRender={onRender}>
      <input aria-label="Filter orders" value={text} onChange={(e) => setText(e.target.value)} />
      {fast ? <Fast query={query} /> : <Naive query={text.toLowerCase()} />}
    </Profiler>
  );
}
const t0 = performance.now();
createRoot(document.getElementById("root")).render(<App />);
requestAnimationFrame(() => setTimeout(() => { window.firstRenderMs = Math.round(performance.now() - t0); }));
```

```console
$ python3 ../tools/page.py "http://localhost:5173/project/?mode=naive" wait:300 "eval:({ firstRenderMs, domNodes: document.getElementsByTagName('*').length })" "eval:resetStats()" "click:input" press:a press:n press:a wait:800 "eval:({ ...stats(), rowsInDom: document.querySelectorAll('li, [role=listitem]').length })" --serve "npx vite --port 5173 --strictPort"
=> {"firstRenderMs": 896, "domNodes": 20011}
=> {"commits": 0, "renderMs": 0, "itemRenders": 0, "worstInputMs": 0}
=> {"commits": 4, "renderMs": 249, "itemRenders": 8308, "worstInputMs": 288, "rowsInDom": 2000}
$ python3 ../tools/page.py "http://localhost:5173/project/?mode=fast" wait:300 "eval:({ firstRenderMs, domNodes: document.getElementsByTagName('*').length })" "eval:resetStats()" "click:input" press:a press:n press:a wait:800 "eval:({ ...stats(), rowsInDom: document.querySelectorAll('li, [role=listitem]').length })" "eval:void (document.querySelector('[data-testid=viewport]').scrollTop = 1e9)" wait:200 "eval:[...document.querySelectorAll('[role=listitem]')].at(-1).textContent" --serve "npx vite --port 5173 --strictPort"
=> {"firstRenderMs": 17, "domNodes": 62}
=> {"commits": 0, "renderMs": 0, "itemRenders": 0, "worstInputMs": 0}
=> {"commits": 4, "renderMs": 7, "itemRenders": 25, "worstInputMs": 16, "rowsInDom": 25}
=> null
=> "#9996 Ana Z — 4.05 (refunded)"
```

| | Naive | Optimised |
| --- | --- | --- |
| First render | about 1 s | under 20 ms |
| DOM elements | about 20,000 | about 60 |
| Row renders while typing "ana" | over 8,000 | under 100 |
| Slowest keystroke | about 300 ms | under 35 ms |

After scrolling to the bottom, the last row is order 9,996 — the last order whose customer contains "ana" — so the window follows the scroll position correctly.

### Deep & Tricky

**DT-13. Windowing removes off-screen rows from the DOM, so the browser's find-in-page (Ctrl+F) cannot find them, screen readers cannot browse them, and their state (an open menu, a half-typed note) is destroyed when they scroll away. Expose the full count to assistive technology (`aria-rowcount`/`aria-setsize`), keep row state outside the rows, and prefer pagination or CSS `content-visibility: auto` — which skips rendering work while keeping the DOM — when the list is only hundreds of rows long.** *(WAI-ARIA 1.2, `aria-rowcount` and `aria-setsize`; CSS Containment Level 2, `content-visibility`; web.dev, "content-visibility: the new CSS property that boosts your rendering performance")*

### Review checklist

- Rows have a fixed height of 24 px. How would you window rows of variable height?
- Keyboard users tab through the list: what happens when focus moves to a row outside the window?
- Build the optimised version with the React Compiler instead of `memo`. Does anything change?

## Answers to the Chapter 6 prediction questions

**Unit 6.2, Q1.** Yes: `ITEMS.filter(…)` creates a new array on every render, so `memo` sees a changed `items` prop every time and re-renders. Memoise the array (`useMemo`), or let the compiler do it.

# Chapter 7 — Routing

*Goal: turn a React app into a multi-page application in which the URL is the source of truth. Using React Router's data APIs, you will compare the request waterfall of fetching in effects with route loaders that fetch in parallel before rendering, write actions for mutations with automatic revalidation, keep search terms in the URL, handle errors per route, and show pending states. The project is a contacts manager with search, viewing, editing with server-side validation, deletion, deep links, and a working Back button.*

**What you will be able to do after this chapter**

- Define nested routes with layouts (`Outlet`), links, and URL parameters.
- Load data with route loaders, and explain why they avoid waterfalls.
- Mutate data with `<Form>` and actions, and rely on revalidation.
- Keep UI state such as search terms in the URL.
- Handle errors and "not found" per route, and show pending navigation.

```js
// file: vite.config.js
// A fake contacts API (150 ms per request) and an SPA fallback, so that /project/contacts/3 loads the app.
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

let contacts = [
  { id: 1, first: "Ada", last: "Lovelace", notes: "First programmer." },
  { id: 2, first: "Grace", last: "Hopper", notes: "Compilers; COBOL." },
  { id: 3, first: "Alan", last: "Turing", notes: "Computability." },
];
const delay = (ms) => new Promise((r) => setTimeout(r, ms));

function fakeApi() {
  return {
    name: "fake-api",
    configureServer(server) {
      server.middlewares.use(async (req, res, next) => {
        const url = new URL(req.url, "http://x");
        if (!url.pathname.startsWith("/api/")) {
          // client-side routes of the project page: serve its index.html (HTML, CSS, and JavaScript book, §17.2)
          if (/^\/(project|waterfall)\/./.test(url.pathname) && !url.pathname.includes(".")) req.url = `/${url.pathname.split("/")[1]}/`;
          return next();
        }
        await delay(Number(url.searchParams.get("delay") ?? 150));
        const send = (status, data) => { res.statusCode = status; res.setHeader("Content-Type", "application/json"); res.end(JSON.stringify(data)); };
        console.log(`  api: ${req.method} ${url.pathname}${url.search}`);
        const m = /^\/api\/contacts\/(\d+)$/.exec(url.pathname);
        if (url.pathname === "/api/contacts" && req.method === "GET") {
          const q = (url.searchParams.get("q") ?? "").toLowerCase();
          return send(200, contacts.filter((c) => `${c.first} ${c.last}`.toLowerCase().includes(q)));
        }
        if (url.pathname === "/api/contacts" && req.method === "POST") {
          const c = { id: Math.max(0, ...contacts.map((x) => x.id)) + 1, first: "", last: "", notes: "" };
          contacts.push(c); return send(201, c);
        }
        if (m) {
          const c = contacts.find((x) => x.id === Number(m[1]));
          if (!c) return send(404, { error: "not found" });
          if (req.method === "GET") return send(200, c);
          if (req.method === "DELETE") { contacts = contacts.filter((x) => x !== c); return send(204, null); }
          if (req.method === "PUT") {
            let body = ""; for await (const chunk of req) body += chunk;
            const data = JSON.parse(body);
            if (!data.first?.trim()) return send(422, { errors: { first: "First name is required" } });
            Object.assign(c, data); return send(200, c);
          }
        }
        if (url.pathname === "/api/stats") return send(200, { total: contacts.length });
        send(404, { error: "no such endpoint" });
      });
    },
  };
}
export default defineConfig({
  plugins: [react(), fakeApi()],
  logLevel: "warn",
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client", "react-router"] },
});
```

```html
<!-- file: page.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Keys</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```console
$ cd .. && npm install react-router@8 2>&1 | tail -1
found 0 vulnerabilities
$ mkdir -p waterfall && cp page.html waterfall/index.html
```

## Topic 7A — Routes and data

## Unit 7.1 — Routes, layouts, and links

**One-line summary:** A router maps URL paths to components: `createBrowserRouter` takes a tree of route objects (`path`, `Component`, `children`), parent routes render their matched child through `<Outlet />` (so layouts nest), `<Link>` and `<NavLink>` navigate without reloading (with `NavLink` marking the active link), and `useParams` reads dynamic segments such as `:id`.

The building blocks appear together in the project at the end of the chapter; first, the reason to load data the router's way.

## Unit 7.2 — Loaders versus effects

**One-line summary:** A route's `loader` runs when a navigation matches the route — before its component renders — and the loaders of all matched routes (parent and children) run in parallel; fetching in effects instead creates a waterfall, because a child cannot start fetching until its parent has fetched, rendered, and mounted it.

```jsx
// file: waterfall/main.jsx
// Fetch-in-effect makes a waterfall: the parent fetches, renders, THEN the child fetches.
// Route loaders start every fetch of the matched routes at once, before rendering.
import { useEffect, useState } from "react";
import { createRoot } from "react-dom/client";
import { createBrowserRouter, RouterProvider, Outlet, useLoaderData } from "react-router";

const t0 = performance.now();
const log = (what) => console.log(`${what} at ${Math.round(performance.now() - t0)} ms`);
const getJson = (url) => fetch(url).then((r) => r.json());

function EffectParent() {
  const [stats, setStats] = useState(null);
  useEffect(() => { getJson("/api/stats?delay=300").then((s) => { log("effects: parent data"); setStats(s); }); }, []);
  return stats ? <><p>{stats.total} contacts</p><EffectChild /></> : <p>loading…</p>;
}
function EffectChild() {
  const [c, setC] = useState(null);
  useEffect(() => { getJson("/api/contacts/1?delay=300").then((d) => { log("effects: child data"); setC(d); }); }, []);
  return <p>{c ? c.first : "loading…"}</p>;
}

function LoaderParent() { const stats = useLoaderData(); return <><p>{stats.total} contacts</p><Outlet /></>; }
function LoaderChild() { const c = useLoaderData(); log("loaders: child rendered with data"); return <p>{c.first}</p>; }
const router = createBrowserRouter([{
  path: "/waterfall/loaders", Component: LoaderParent, HydrateFallback: () => <p>loading…</p>,
  loader: async () => { const s = await getJson("/api/stats?delay=300"); log("loaders: parent data"); return s; },
  children: [{ index: true, Component: LoaderChild,
    loader: async () => { const c = await getJson("/api/contacts/1?delay=300"); log("loaders: child data"); return c; } }],
}]);

const root = createRoot(document.getElementById("root"));
if (location.pathname.endsWith("/loaders")) root.render(<RouterProvider router={router} />);
else root.render(<EffectParent />);
```

```console
$ python3 ../tools/page.py "http://localhost:5173/waterfall/" wait:900 --serve "npx vite --port 5173 --strictPort"
effects: parent data at 323 ms
effects: child data at 635 ms
$ python3 ../tools/page.py "http://localhost:5173/waterfall/loaders" wait:900 --serve "npx vite --port 5173 --strictPort"
loaders: parent data at 311 ms
loaders: child data at 311 ms
loaders: child rendered with data at 318 ms
```

Two 300-ms requests take about 600 ms with effects — the child's request starts only after the parent's data arrived and the child mounted — and about 300 ms with loaders, which started both at once; the child rendered once, with its data, instead of first rendering "loading…".

### Mastery check

1. **Calculate.** Three nested routes each have a loader that takes 200 ms. Roughly how long until the page renders with loaders, and with fetching in effects?

## Unit 7.3 — Actions, forms, and revalidation

**One-line summary:** A route's `action` handles non-GET submissions of a React Router `<Form>` (which works like an HTML form, without a page reload); after every action the router re-runs the loaders of the matched routes, so data shown anywhere on the page is fresh without manual cache updates, and an action can `redirect` or return data (such as validation errors) to `useActionData`.

### Deep & Tricky

**DT-14. Revalidation after actions is the router's caching strategy: after *every* action — including one that returns validation errors — the loaders of all matched routes run again, so a failed save of one field re-fetches the page's data. That keeps the UI consistent with the server at the cost of extra requests; a route's `shouldRevalidate` function can opt out when the action cannot have changed its data.** *(React Router documentation, "Revalidation" and `shouldRevalidate`; the project's server log shows the two GET requests after the 422)*

## Topic 7B — The URL as state

## Unit 7.4 — Search parameters and pending UI

**One-line summary:** State that should survive a reload, be shareable, or work with Back — search terms, filters, page numbers, the selected tab — belongs in the URL: a GET `<Form>` writes its fields to the query string, the loader reads them from `request.url`, and `useNavigation` reports whether a navigation is loading or submitting, for pending indicators.

### Pattern

**P-7. Make the URL the source of truth for navigation state: every screen a user can reach should have a URL that recreates it, so the Back button, bookmarks, links, and reloads work; keep only ephemeral UI state (an open menu, a hover) in component state.** *(React Router documentation, "Search Params"; Book 1, Chapter 17)* *Trade-off:* URL state is serialised as strings and limited in size; complex filter objects need careful encoding, and replacing history entries (`replace: true`) on every keystroke is needed to avoid flooding the Back button.

## Chapter 7 Project — A contacts manager

### Requirements

Build a contacts app on this chapter's fake API: a sidebar list with a search box (the query in the URL, without flooding the history), a contact page, an edit page with server-side validation errors that keep the user's input, "New" and "Delete" via actions with redirects, a pending state while navigating, a "not found" page for unknown contacts that keeps the sidebar visible, and deep links that work on reload.

### Reference solution

```html
<!-- file: project/index.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Contacts</title></head>
<body>
  <div id="root"></div>
  <!-- an absolute path: this page is also served at /project/contacts/3/edit -->
  <script type="module" src="/project/main.jsx"></script>
</body>
</html>
```

```js
// file: project/api.js
async function request(path, options) {
  const res = await fetch(`/api${path}`, options);
  if (res.status === 404) throw new Response("Not found", { status: 404 });   // routes turn this into a 404 page
  const data = res.status === 204 ? null : await res.json();
  return { status: res.status, data };
}
export const listContacts = (q, signal) => request(`/contacts?q=${encodeURIComponent(q)}`, { signal }).then((r) => r.data);
export const getContact = (id, signal) => request(`/contacts/${id}`, { signal }).then((r) => r.data);
export const createContact = () => request("/contacts", { method: "POST" }).then((r) => r.data);
export const updateContact = (id, data) => request(`/contacts/${id}`, { method: "PUT", body: JSON.stringify(data) });
export const deleteContact = (id) => request(`/contacts/${id}`, { method: "DELETE" });
```

```jsx
// file: project/routes.jsx
// A contacts app with React Router's data APIs: loaders read, actions write, and after every
// action the router re-runs the loaders, so the screen always shows the server's data.
import { Form, Link, NavLink, Outlet, redirect, useLoaderData, useNavigation, useRouteError,
         useSubmit, isRouteErrorResponse, useActionData } from "react-router";
import * as api from "./api.js";

export function Root() {
  const { contacts, q } = useLoaderData();
  const navigation = useNavigation();                       // "idle" | "loading" | "submitting"
  const submit = useSubmit();
  return (
    <div className="app">
      <nav aria-label="Contacts">
        <Form role="search" id="search">
          <input aria-label="Search contacts" name="q" defaultValue={q} key={q}
                 onChange={(e) => submit(e.currentTarget.form, { replace: q !== "" })} />
        </Form>
        <Form method="post"><button>New</button></Form>
        <ul>{contacts.map((c) => <li key={c.id}>
          <NavLink to={`contacts/${c.id}`}>{c.first || c.last ? `${c.first} ${c.last}` : <i>No name</i>}</NavLink>
        </li>)}</ul>
      </nav>
      <main aria-busy={navigation.state !== "idle"}><Outlet /></main>
    </div>
  );
}
Root.loader = async ({ request }) => {
  const q = new URL(request.url).searchParams.get("q") ?? "";   // the search lives in the URL
  return { contacts: await api.listContacts(q, request.signal), q };
};
Root.action = async () => {
  const contact = await api.createContact();
  return redirect(`contacts/${contact.id}/edit`);
};

export function Contact() {
  const c = useLoaderData();
  return (
    <article>
      <h1>{c.first} {c.last}</h1>
      <p>{c.notes}</p>
      <Link to="edit">Edit</Link>
      <Form method="post" action="destroy" onSubmit={(e) => { if (!confirm("Delete this contact?")) e.preventDefault(); }}>
        <button>Delete</button>
      </Form>
    </article>
  );
}
Contact.loader = ({ params, request }) => api.getContact(params.id, request.signal);

export function EditContact() {
  const c = useLoaderData();
  const errors = useActionData()?.errors ?? {};
  const navigation = useNavigation();
  return (
    <Form method="post">
      <h1>Edit contact</h1>
      <p><label>First <input name="first" defaultValue={c.first} aria-invalid={!!errors.first}
           aria-describedby={errors.first ? "first-error" : undefined} /></label>
         {errors.first && <span id="first-error" role="alert"> {errors.first}</span>}</p>
      <p><label>Last <input name="last" defaultValue={c.last} /></label></p>
      <p><label>Notes <textarea name="notes" defaultValue={c.notes} /></label></p>
      <button disabled={navigation.state === "submitting"}>{navigation.state === "submitting" ? "Saving…" : "Save"}</button>
    </Form>
  );
}
EditContact.loader = Contact.loader;
EditContact.action = async ({ request, params }) => {
  const data = Object.fromEntries(await request.formData());
  const { status, data: body } = await api.updateContact(params.id, data);
  if (status === 422) return body;                           // shown by useActionData; the form keeps its input
  return redirect(`/project/contacts/${params.id}`);
};

export const destroy = { action: async ({ params }) => { await api.deleteContact(params.id); return redirect("/project/"); } };

export function ErrorPage() {
  const error = useRouteError();
  return <article role="alert"><h1>{isRouteErrorResponse(error) && error.status === 404 ? "Not found" : "Something went wrong"}</h1>
    <Link to="/project/">Back to all contacts</Link></article>;
}

export const Welcome = () => <p>Choose a contact, or create a new one.</p>;
```

```jsx
// file: project/main.jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router";
import { Root, Contact, EditContact, ErrorPage, Welcome, destroy } from "./routes.jsx";

const router = createBrowserRouter([{
  path: "/project/", Component: Root, loader: Root.loader, action: Root.action,
  ErrorBoundary: ErrorPage, HydrateFallback: () => <p>Loading…</p>,
  children: [{
    ErrorBoundary: ErrorPage,                                 // errors in a contact keep the list on screen
    children: [
      { index: true, Component: Welcome },
      { path: "contacts/:id", Component: Contact, loader: Contact.loader },
      { path: "contacts/:id/edit", Component: EditContact, loader: EditContact.loader, action: EditContact.action },
      { path: "contacts/:id/destroy", action: destroy.action },
    ],
  }],
}]);
router.subscribe((state) => { if (state.navigation.state === "idle") console.log(`[router] ${state.location.pathname}${state.location.search}`); });
createRoot(document.getElementById("root")).render(<StrictMode><RouterProvider router={router} /></StrictMode>);
```

A tour: open a contact, search, open the result, try to save an empty first name, fix it — then a deep link to an edit page, Back, and an unknown contact:

```console
$ python3 ../tools/page.py "http://localhost:5173/project/" wait:400 "click:text=Grace Hopper" wait:400 "eval:[document.querySelector('main').innerText.replaceAll('\n', ' | '), [...document.querySelectorAll('nav li')].map((li) => li.textContent).join(', '), location.pathname + location.search]" "fill:[aria-label='Search contacts']|tur" wait:500 "eval:[document.querySelector('main').innerText.replaceAll('\n', ' | '), [...document.querySelectorAll('nav li')].map((li) => li.textContent).join(', '), location.pathname + location.search]" "click:text=Alan Turing" wait:400 "click:text=Edit" wait:400 "fill:[name=first]|" "click:text=Save" wait:400 "eval:[document.querySelector('main').innerText.replaceAll('\n', ' | '), [...document.querySelectorAll('nav li')].map((li) => li.textContent).join(', '), location.pathname + location.search]" "fill:[name=first]|Alan M." "click:text=Save" wait:600 "eval:[document.querySelector('main').innerText.replaceAll('\n', ' | '), [...document.querySelectorAll('nav li')].map((li) => li.textContent).join(', '), location.pathname + location.search]" --serve "npx vite --port 5173 --strictPort" --server-log
  api: GET /api/contacts?q=
[router] /project/
  api: GET /api/contacts/2
[router] /project/contacts/2
=> ["Grace Hopper |  | Compilers; COBOL. |  | Edit | Delete", "Ada Lovelace, Grace Hopper, Alan Turing", "/project/contacts/2"]
  api: GET /api/contacts?q=tur
[router] /project?q=tur
=> ["Choose a contact, or create a new one.", "Alan Turing", "/project?q=tur"]
  api: GET /api/contacts?q=
  api: GET /api/contacts/3
[router] /project/contacts/3
  api: GET /api/contacts/3
[router] /project/contacts/3/edit
  api: PUT /api/contacts/3
Failed to load resource: the server responded with a status of 422 (Unprocessable Entity)
  api: GET /api/contacts?q=
  api: GET /api/contacts/3
[router] /project/contacts/3/edit
=> ["Edit contact |  | First  First name is required |  | Last  |  | Notes  |  | Save", "Ada Lovelace, Grace Hopper, Alan Turing", "/project/contacts/3/edit"]
  api: PUT /api/contacts/3
  api: GET /api/contacts/3
  api: GET /api/contacts?q=
[router] /project/contacts/3
=> ["Alan M. Turing |  | Computability. |  | Edit | Delete", "Ada Lovelace, Grace Hopper, Alan M. Turing", "/project/contacts/3"]
$ python3 ../tools/page.py "http://localhost:5173/project/contacts/3/edit" wait:500 "fill:[name=first]|" "fill:[name=notes]|Enigma" "click:text=Save" wait:500 "eval:document.querySelector('[name=notes]').value" "click:text=Ada Lovelace" wait:400 "eval:history.back()" wait:500 "eval:location.pathname" "eval:void (location.href = '/project/contacts/99')" wait:600 "eval:[document.querySelector('main').innerText.replaceAll('\n', ' | '), [...document.querySelectorAll('nav li')].map((li) => li.textContent).join(', '), location.pathname + location.search]" --serve "npx vite --port 5173 --strictPort" --server-log
  api: GET /api/contacts?q=
  api: GET /api/contacts/3
[router] /project/contacts/3/edit
  api: PUT /api/contacts/3
Failed to load resource: the server responded with a status of 422 (Unprocessable Entity)
  api: GET /api/contacts?q=
  api: GET /api/contacts/3
[router] /project/contacts/3/edit
=> "Enigma"
  api: GET /api/contacts/1
[router] /project/contacts/1
=> null
  api: GET /api/contacts/3
[router] /project/contacts/3/edit
=> "/project/contacts/3/edit"
=> null
  api: GET /api/contacts?q=
  api: GET /api/contacts/99
Failed to load resource: the server responded with a status of 404 (Not Found)
[router] /project/contacts/99
=> ["Not found | Back to all contacts", "Ada Lovelace, Grace Hopper, Alan Turing", "/project/contacts/99"]
```

Every screen has a URL: the search put `?q=tur` in the address bar and its loader asked the API for matching contacts; the failed save returned the error to `useActionData` (and, as DT-14 says, the loaders ran again); the successful save redirected to the contact page, whose fresh data — and the sidebar's — came from revalidation. The deep link to an edit page loaded the app at that page, the notes typed before a failed save were kept, Back returned to the edit page, and the unknown contact produced a "Not found" page inside the layout, with the sidebar still usable.

### Review checklist

- The edit page's form fields are uncontrolled with `defaultValue`. Why does the form keep its input after a 422 here, when Chapter 4's React form action reset it?
- Add `useFetcher` to toggle a "favourite" star on a contact without navigating.
- Split the edit page into its own chunk with `lazy` routes, and check the network requests.

## Answers to the Chapter 7 prediction questions

**Unit 7.2, Q1.** About 200 ms with loaders (all three run in parallel), about 600 ms with effects (each level waits for its parent to fetch and mount it).

# Chapter 8 — Server State with TanStack Query

*Goal: stop treating data from the server as ordinary state. Server data is a cache of something that lives elsewhere, changes without you, and can be stale; TanStack Query manages that cache — deduplicating requests, serving fresh data without a network round trip, refreshing stale data in the background, paginating, and coordinating mutations with optimistic updates and rollbacks. The project is an issue tracker whose "close" button updates every cached list at once and undoes itself when the server refuses.*

**What you will be able to do after this chapter**

- Explain the difference between client state and server state.
- Fetch with `useQuery`: query keys, caching, staleness, background refetching, cancellation.
- Paginate with `useInfiniteQuery`.
- Write mutations that invalidate, or optimistically update and roll back, cached data.
- Organise query keys and fetchers so that a codebase stays consistent.

```js
// file: vite.config.js
// A fake issue tracker API: 200 issues, paging, filtering, and a PATCH that always fails for issue 7.
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

const issues = Array.from({ length: 200 }, (_, i) => ({
  id: i + 1, title: `Issue number ${i + 1}`, state: i % 4 === 0 ? "closed" : "open", comments: (i * 7) % 13,
}));
let requests = 0;
function fakeApi() {
  return {
    name: "fake-api",
    configureServer(server) {
      server.middlewares.use("/api", async (req, res) => {
        const url = new URL(req.url, "http://x");
        requests++;
        console.log(`  api #${requests}: ${req.method} ${url.pathname}${url.search}`);
        await new Promise((r) => setTimeout(r, 200));
        const send = (status, data) => { res.statusCode = status; res.setHeader("Content-Type", "application/json"); res.end(JSON.stringify(data)); };
        if (url.pathname === "/issues") {
          const state = url.searchParams.get("state") ?? "all";
          const page = Number(url.searchParams.get("page") ?? 1), size = 20;
          const all = issues.filter((i) => state === "all" || i.state === state);
          return send(200, { items: all.slice((page - 1) * size, page * size), nextPage: page * size < all.length ? page + 1 : null, total: all.length });
        }
        const m = /^\/issues\/(\d+)$/.exec(url.pathname);
        if (m) {
          const issue = issues.find((i) => i.id === Number(m[1]));
          if (req.method === "PATCH") {
            let body = ""; for await (const c of req) body += c;
            if (issue.id === 7) return send(500, { error: "unlucky issue: the database is down" });
            Object.assign(issue, JSON.parse(body));
          }
          return send(200, issue);
        }
        send(404, {});
      });
    },
  };
}
export default defineConfig({
  plugins: [react(), fakeApi()],
  logLevel: "warn",
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client", "@tanstack/react-query"] },
});
```

```html
<!-- file: page.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Keys</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```console
$ cd .. && npm install @tanstack/react-query@5 2>&1 | tail -1
found 0 vulnerabilities
$ for d in dedupe project; do mkdir -p $d && cp page.html $d/index.html; done
```

## Topic 8A — Queries

## Unit 8.1 — A cache keyed by queries

**One-line summary:** `useQuery({ queryKey, queryFn })` looks up the key in a shared cache: components asking for the same key share one request and one entry; data younger than `staleTime` is returned without a request; stale data is returned at once *and* refetched in the background (on mount, window focus, or reconnect); and entries nobody uses are garbage-collected after `gcTime`.

```jsx
// file: dedupe/main.jsx
// Server state is a CACHE of data that lives elsewhere. TanStack Query keeps it per query key:
// identical queries share one request and one cache entry, fresh data is served without a request,
// and stale data is shown at once and refreshed in the background.
import { useState } from "react";
import { createRoot } from "react-dom/client";
import { QueryClient, QueryClientProvider, useQuery } from "@tanstack/react-query";

const fetchIssue = (id) => fetch(`/api/issues/${id}`).then((r) => r.json());

function IssueTitle({ id, label }) {
  const { data, isPending, isFetching, dataUpdatedAt } = useQuery({
    queryKey: ["issue", id],                     // the cache key: same key, same data
    queryFn: () => fetchIssue(id),
    staleTime: 1000,                             // considered fresh for 1 s: no refetch in that time
  });
  return <p>{label}: {isPending ? "loading…" : data.title}{isFetching && !isPending ? " (refreshing)" : ""}</p>;
}

function App() {
  const [shown, setShown] = useState(true);
  return <>
    <button onClick={() => setShown(!shown)}>toggle</button>
    {shown && <><IssueTitle id={1} label="header" /><IssueTitle id={1} label="sidebar" /><IssueTitle id={2} label="other" /></>}
  </>;
}
const client = new QueryClient();
createRoot(document.getElementById("root")).render(<QueryClientProvider client={client}><App /></QueryClientProvider>);
window.texts = () => [...document.querySelectorAll("p")].map((p) => p.textContent);
```

```console
$ python3 ../tools/page.py "http://localhost:5173/dedupe/" wait:400 "eval:texts()" "click:text=toggle" "click:text=toggle" "eval:texts()" wait:1200 "click:text=toggle" "click:text=toggle" "eval:texts()" wait:400 "eval:texts()" --serve "npx vite --port 5173 --strictPort" --server-log
  api #1: GET /issues/1
  api #2: GET /issues/2
=> ["header: Issue number 1", "sidebar: Issue number 1", "other: Issue number 2"]
=> ["header: Issue number 1", "sidebar: Issue number 1", "other: Issue number 2"]
  api #3: GET /issues/1
  api #4: GET /issues/2
=> ["header: Issue number 1 (refreshing)", "sidebar: Issue number 1 (refreshing)", "other: Issue number 2 (refreshing)"]
=> ["header: Issue number 1", "sidebar: Issue number 1", "other: Issue number 2"]
```

Three components, two keys: two requests. Unmounting and remounting them within the one-second `staleTime` made no requests at all — the data came from the cache. After the data went stale, remounting showed the cached titles immediately, marked "(refreshing)", while two background requests confirmed them. Compare Chapter 3's hand-written `useJson`: every component fetched for itself, every time.

### Mastery check

1. **Predict.** A fourth component asks for `["issue", "1"]` (a string). How many requests are made now?

### Deep & Tricky

**DT-15. With TanStack Query's default `staleTime` of 0, every cached query is stale as soon as it arrives, so it is refetched on every mount of a component that uses it and every time the window regains focus — correct but chatty. Set `staleTime` to how long data can reasonably be shown without checking (seconds for prices, minutes for a user profile), which is a product decision, not a technical one.** *(TanStack Query documentation, "Important Defaults": "Query instances via useQuery… by default consider cached data as stale"; `refetchOnWindowFocus`, `staleTime`, `gcTime`)*

## Unit 8.2 — Pagination with infinite queries

**One-line summary:** `useInfiniteQuery` stores a list of pages under one key, with `getNextPageParam` deciding how to ask for the next one, so "load more" and infinite scrolling keep all loaded pages in the cache — and when the query is refetched or invalidated, every loaded page is refetched in order, so the whole list stays consistent.

## Topic 8B — Mutations

## Unit 8.3 — Mutations, invalidation, and optimistic updates

**One-line summary:** `useMutation` runs a change on the server and reports its status; afterwards you either *invalidate* the queries it affects (mark them stale and refetch the active ones) or update the cache directly — and for an optimistic update, you cancel in-flight refetches, snapshot the cache, write the expected result in `onMutate`, restore the snapshot in `onError`, and invalidate in `onSettled` to converge on the server's truth.

### Pattern

**P-8. Keep server state in a server-state cache and nowhere else: no copying query results into `useState`, Redux, or context; derive what you need from the query data; and define query keys in one module (a "key factory") so that invalidations always match the queries they mean.** *(TanStack Query documentation, "Query Keys" and "Effective React Query Keys" by Dominik Dorfmeister)* *Trade-off:* the cache becomes a central dependency of every data-reading component; tests must provide a fresh `QueryClient`, and server rendering must hydrate the cache (Chapter 12).

## Chapter 8 Project — An issue tracker

### Requirements

On this chapter's fake API (200 issues, 20 per page; changing issue 7 always fails), build an issue list with "open/closed/all" filters (a different query key each), "Load more" pagination, and a Close/Reopen button per issue. Closing must update the list at once (optimistically, in every cached list), be confirmed by refetching, and — when the server fails — roll back and show an error. Switching back to a filter that was viewed recently must not refetch.

### Reference solution

```js
// file: project/queries.js
// All server-state access in one module: query keys, fetchers, and the mutation with an
// optimistic update that is rolled back if the server refuses.
import { useInfiniteQuery, useMutation, useQueryClient } from "@tanstack/react-query";

async function getJson(url, options) {
  const res = await fetch(url, options);
  const body = await res.json();
  if (!res.ok) throw new Error(body.error ?? `HTTP ${res.status}`);
  return body;
}

export const issueKeys = {                                  // one place that defines the key hierarchy
  all: ["issues"],
  list: (state) => ["issues", "list", state],
};

export function useIssues(state) {
  return useInfiniteQuery({
    queryKey: issueKeys.list(state),
    queryFn: ({ pageParam, signal }) => getJson(`/api/issues?state=${state}&page=${pageParam}`, { signal }),
    initialPageParam: 1,
    getNextPageParam: (last) => last.nextPage,              // null: no more pages
  });
}

export function useSetIssueState() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: ({ id, state }) => getJson(`/api/issues/${id}`, { method: "PATCH", body: JSON.stringify({ state }) }),
    onMutate: async ({ id, state }) => {                    // optimistic: change every cached list now
      await qc.cancelQueries({ queryKey: issueKeys.all });  // an in-flight refetch must not overwrite it
      const snapshot = qc.getQueriesData({ queryKey: issueKeys.all });
      qc.setQueriesData({ queryKey: issueKeys.all }, (data) => data && {
        ...data,
        pages: data.pages.map((p) => ({ ...p, items: p.items.map((i) => (i.id === id ? { ...i, state } : i)) })),
      });
      return { snapshot };
    },
    onError: (_err, _vars, context) => {                     // roll back to exactly what was cached before
      for (const [key, data] of context.snapshot) qc.setQueryData(key, data);
    },
    onSettled: () => qc.invalidateQueries({ queryKey: issueKeys.all }),   // then refetch the truth
  });
}
```

```jsx
// file: project/main.jsx
import { StrictMode, useState } from "react";
import { createRoot } from "react-dom/client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { useIssues, useSetIssueState } from "./queries.js";

function Issues() {
  const [state, setState] = useState("open");
  const { data, error, isPending, fetchNextPage, hasNextPage, isFetchingNextPage } = useIssues(state);
  const toggle = useSetIssueState();
  if (isPending) return <p>Loading issues…</p>;
  if (error) return <p role="alert">Could not load issues: {error.message}</p>;
  const items = data.pages.flatMap((p) => p.items);
  return (
    <main>
      <p>
        {["open", "closed", "all"].map((s) => (
          <button key={s} aria-pressed={s === state} onClick={() => setState(s)}>{s}</button>
        ))}{" "}
        {items.length} of {data.pages[0].total}
      </p>
      {toggle.isError && <p role="alert">Could not update issue {toggle.variables.id}: {toggle.error.message}</p>}
      <ul>
        {items.map((i) => (
          <li key={i.id} data-id={i.id}>
            #{i.id} {i.title} [{i.state}]{" "}
            <button onClick={() => toggle.mutate({ id: i.id, state: i.state === "open" ? "closed" : "open" })}>
              {i.state === "open" ? "Close" : "Reopen"}
            </button>
          </li>
        ))}
      </ul>
      {hasNextPage && <button onClick={() => fetchNextPage()} disabled={isFetchingNextPage}>
        {isFetchingNextPage ? "Loading…" : "Load more"}</button>}
    </main>
  );
}

const client = new QueryClient({ defaultOptions: { queries: { staleTime: 30_000, retry: 1 } } });
createRoot(document.getElementById("root")).render(
  <StrictMode><QueryClientProvider client={client}><Issues /></QueryClientProvider></StrictMode>);
window.issue = (id) => document.querySelector(`li[data-id="${id}"]`)?.textContent ?? "(not in the list)";
window.summary = () => document.querySelector("main p").textContent.replace(/^(open|closed|all)+\s*/, "");
```

```console
$ python3 ../tools/page.py "http://localhost:5173/project/" wait:500 "eval:summary()" "click:text=Load more" wait:400 "eval:summary()" "click:li[data-id='2'] button" wait:50 "eval:issue(2)" wait:700 "eval:[issue(2), summary()]" "click:text=closed" wait:500 "eval:[summary(), issue(2)]" "click:text=open" wait:100 "click:li[data-id='7'] button" wait:50 "eval:issue(7)" wait:900 "eval:[issue(7), document.querySelector('[role=alert]')?.textContent]" --serve "npx vite --port 5173 --strictPort" --server-log
  api #1: GET /issues?state=open&page=1
  api #2: GET /issues?state=open&page=1
=> "20 of 150"
  api #3: GET /issues?state=open&page=2
=> "40 of 150"
  api #4: PATCH /issues/2
=> "#2 Issue number 2 [closed] Reopen"
  api #5: GET /issues?state=open&page=1
  api #6: GET /issues?state=open&page=2
=> ["(not in the list)", "40 of 149"]
  api #7: GET /issues?state=closed&page=1
=> ["20 of 51", "#2 Issue number 2 [closed] Reopen"]
  api #8: PATCH /issues/7
=> "#7 Issue number 7 [closed] Reopen"
Failed to load resource: the server responded with a status of 500 (Internal Server Error)
  api #9: GET /issues?state=open&page=1
  api #10: GET /issues?state=open&page=2
=> ["#7 Issue number 7 [open] Close", "Could not update issue 7: unlucky issue: the database is down"]
```

The story the server log and the page tell together:

- The first page was requested twice: in development, StrictMode unmounted and remounted the component while the first request was in flight, and because the query function uses the `signal`, TanStack Query aborted the unobserved request and started a new one.
- "Load more" fetched page 2 into the same cache entry: 40 of 150 open issues.
- Closing issue 2 changed the list *before* the PATCH completed; the invalidation that followed refetched both loaded pages, and the issue left the "open" list (40 of 149).
- The "closed" filter was a new key, so it was fetched — and contains issue 2. Switching back to "open" made no request: that data was 30 seconds fresh.
- Closing issue 7 showed it closed at once; the server answered 500, the snapshot was restored, the error was shown, and the refetch confirmed the issue is still open.

### Deep & Tricky

**DT-16. If a query function uses the `AbortSignal` that TanStack Query passes in, the library cancels the request when the last component observing the query unmounts — which saves bandwidth, but also means a quick unmount–remount (a tab switch, StrictMode's double mount in development) aborts and restarts the request, as the log above shows. Without the signal, the first request is allowed to finish and is reused.** *(TanStack Query documentation, "Query Cancellation": "if you consume the AbortSignal… the Query will be cancelled when the component unmounts or becomes unused")*

### Review checklist

- The optimistic update edits every cached list, including "closed" lists that do not yet contain the issue. What should it do with them?
- Prefetch page 2 when the user hovers over "Load more" (`queryClient.prefetchInfiniteQuery`).
- Replace "Load more" with infinite scrolling using an `IntersectionObserver` (HTML, CSS, and JavaScript book, §8.6).

## Answers to the Chapter 8 prediction questions

**Unit 8.1, Q1.** Three: query keys are compared structurally, and the number `1` and the string `"1"` are different, so `["issue", "1"]` is a separate cache entry with its own request — a common source of duplicate fetches when IDs come from URLs (strings) in one place and from data (numbers) in another.

# Chapter 9 — TypeScript with React

*Goal: type React code precisely enough that the compiler catches the mistakes reviewers usually catch. Building on Book 1's TypeScript chapter, you will type props (including every native attribute of an element), children, events, state, and refs; type reducers with discriminated unions and context with a hook that cannot return `undefined`; write generic components whose props are checked against each other; and build a sortable table whose column definitions are checked against the row type — with the compile-time errors themselves as tests.*

**What you will be able to do after this chapter**

- Type component props, children, events, state, and refs.
- Extend native element props with `ComponentProps<"button">`.
- Type reducers and context so that impossible actions and missing providers are caught.
- Write generic components, and use mapped and conditional types for safer APIs.
- Test types with `@ts-expect-error`.

The chapter's `tsconfig.json` is created by a command (JSON cannot carry the file-name comment); a second configuration also includes the file of deliberate mistakes:

```console
$ echo "{ \"compilerOptions\": { \"jsx\": \"react-jsx\", \"strict\": true, \"noEmit\": true, \"target\": \"es2024\", \"module\": \"preserve\", \"moduleResolution\": \"bundler\", \"allowImportingTsExtensions\": true, \"skipLibCheck\": true }, \"include\": [\"basics/Button.tsx\", \"generic\", \"project\"] }" > tsconfig.json && echo "{ \"extends\": \"./tsconfig.json\", \"include\": [\"basics\"] }" > tsconfig.mistakes.json
```

`"module": "preserve"` and `"moduleResolution": "bundler"` describe code that a bundler (Vite) or a runner (`tsx`) will process; `"jsx": "react-jsx"` matches React's automatic runtime.

## Topic 9A — Typing components

## Unit 9.1 — Props, children, events, state, and refs

**One-line summary:** Props are typed with an interface or type alias (optional props with `?`); `ReactNode` is the type of anything renderable; extending `ComponentProps<"button">` gives a component every native button attribute; event handlers receive typed events (`React.ChangeEvent<HTMLInputElement>`); `useState` infers its type from the initial value unless it needs a union; DOM refs are `useRef<HTMLInputElement>(null)`.

```tsx
// file: basics/Button.tsx
// Typing components: props interfaces, children, events, state, refs, and native props.
import { useRef, useState, type ComponentProps, type ReactNode } from "react";

type Variant = "primary" | "danger";
interface ButtonProps extends ComponentProps<"button"> {   // every native <button> prop, plus ours
  variant?: Variant;
  icon?: ReactNode;                                          // anything renderable: elements, strings, null...
}
export function Button({ variant = "primary", icon, children, ...rest }: ButtonProps) {
  return <button className={`btn btn-${variant}`} {...rest}>{icon}{children}</button>;
}

export function NameField({ onSubmit }: { onSubmit: (name: string) => void }) {
  const [name, setName] = useState("");                      // inferred: string
  const [age, setAge] = useState<number | null>(null);       // a union needs an explicit type argument
  const input = useRef<HTMLInputElement>(null);              // a DOM ref: RefObject<HTMLInputElement | null>
  function handleChange(e: React.ChangeEvent<HTMLInputElement>) { setName(e.target.value); }
  return (
    <form onSubmit={(e) => { e.preventDefault(); onSubmit(name); input.current?.focus(); }}>
      <input ref={input} value={name} onChange={handleChange} />
      <input type="number" value={age ?? ""} onChange={(e) => setAge(e.target.value ? Number(e.target.value) : null)} />
      <Button type="submit" variant="primary" disabled={!name}>Save</Button>
    </form>
  );
}
```

```tsx
// file: basics/mistakes.tsx
// Mistakes the compiler catches.
import { useState } from "react";
import { Button, NameField } from "./Button.tsx";

export function Page() {
  const [count, setCount] = useState(0);
  return (
    <main>
      <Button variant="warning">Delete</Button>
      <Button onClick={(e) => e.target.value}>Click</Button>
      <NameField onSubmit={(name: number) => setCount(name)} />
      <NameField />
      {setCount("1")}
      <input onChange={(e) => setCount(e.target.value)} />
    </main>
  );
}
```

```console
$ npx tsc -p tsconfig.mistakes.json
basics/mistakes.tsx(10,15): error TS2322: Type '"warning"' is not assignable to type 'Variant | undefined'.
basics/mistakes.tsx(11,40): error TS2339: Property 'value' does not exist on type 'EventTarget'.
basics/mistakes.tsx(12,18): error TS2322: Type '(name: number) => void' is not assignable to type '(name: string) => void'.
  Types of parameters 'name' and 'name' are incompatible.
    Type 'string' is not assignable to type 'number'.
basics/mistakes.tsx(13,8): error TS2741: Property 'onSubmit' is missing in type '{}' but required in type '{ onSubmit: (name: string) => void; }'.
basics/mistakes.tsx(14,7): error TS2322: Type 'void' is not assignable to type 'ReactNode'.
basics/mistakes.tsx(14,17): error TS2345: Argument of type 'string' is not assignable to parameter of type 'SetStateAction<number>'.
basics/mistakes.tsx(15,40): error TS2345: Argument of type 'string' is not assignable to parameter of type 'SetStateAction<number>'.
```

Seven mistakes, all caught: an unknown variant; `e.target` on a click is just an `EventTarget` (use `e.currentTarget`, which is typed as the button); a callback with the wrong parameter type; a missing required prop; a function call (returning `void`) where JSX expects something renderable; and two attempts to put a string into numeric state.

### Deep & Tricky

**DT-17. `e.target` is typed as `EventTarget` because the event may have been dispatched by any descendant — a click on a button can come from the `<svg>` icon inside it — while `e.currentTarget` is the element whose handler is running, and React's types give it the element's type. The same distinction is the source of event-delegation bugs at run time (Book 1, DT on `currentTarget`): the types are telling the truth.** *(React type definitions, `React.SyntheticEvent<T>` — `currentTarget: EventTarget & T`, `target: EventTarget`; DOM Standard §2.2 "Interface Event")*

### Mastery check

1. **Predict.** Does `<Button aria-label="Close" onPointerDown={() => {}}>×</Button>` type-check?

## Unit 9.2 — Reducers, context, and generic components

**One-line summary:** A reducer's actions are a discriminated union, so `dispatch` accepts only valid actions and an exhaustive `switch` proves every case is handled; a context created with `null` as its default should be read through a custom hook that throws outside its provider, narrowing the type for every caller; and a component can be generic (`function List<T>(props: ListProps<T>)`), with `T` inferred from one prop and enforced on the others.

```tsx
// file: generic/context.tsx
// Typed reducers and context: a discriminated union of actions, and a context hook that fails
// loudly when used outside its provider, so components never have to handle "undefined".
import { createContext, use, useReducer, type Dispatch, type ReactNode } from "react";

interface Todo { id: number; text: string; done: boolean }
type Action =
  | { type: "added"; text: string }
  | { type: "toggled"; id: number }
  | { type: "removed"; id: number };

function reducer(todos: Todo[], action: Action): Todo[] {
  switch (action.type) {
    case "added": return [...todos, { id: todos.length + 1, text: action.text, done: false }];
    case "toggled": return todos.map((t) => (t.id === action.id ? { ...t, done: !t.done } : t));
    case "removed": return todos.filter((t) => t.id !== action.id);
    default: { const unreachable: never = action; throw new Error(`unknown action ${JSON.stringify(unreachable)}`); }
  }
}

const TodosContext = createContext<{ todos: Todo[]; dispatch: Dispatch<Action> } | null>(null);

export function TodosProvider({ children }: { children: ReactNode }) {
  const [todos, dispatch] = useReducer(reducer, []);
  return <TodosContext value={{ todos, dispatch }}>{children}</TodosContext>;
}
export function useTodos() {
  const value = use(TodosContext);
  if (!value) throw new Error("useTodos must be used inside <TodosProvider>");
  return value;                                     // typed without null from here on
}

export function AddButton() {
  const { dispatch } = useTodos();
  // @ts-expect-error: "add" is not an action type, and "text" is required
  dispatch({ type: "add" });
  return <button onClick={() => dispatch({ type: "added", text: "milk" })}>Add</button>;
}
```

```tsx
// file: generic/List.tsx
// A generic component: the type of `items` decides the type of `renderItem`'s argument and of `getKey`.
import type { Key, ReactNode } from "react";

interface ListProps<T> {
  items: readonly T[];
  getKey: (item: T) => Key;
  renderItem: (item: T) => ReactNode;
  empty?: ReactNode;
}
export function List<T>({ items, getKey, renderItem, empty = "Nothing here." }: ListProps<T>) {
  if (items.length === 0) return <p>{empty}</p>;
  return <ul>{items.map((item) => <li key={getKey(item)}>{renderItem(item)}</li>)}</ul>;
}

const users = [{ id: 1, name: "Ana" }, { id: 2, name: "Bo" }];
export const ok = <List items={users} getKey={(u) => u.id} renderItem={(u) => u.name.toUpperCase()} />;
// @ts-expect-error: users have no "email"; T was inferred from `items`
export const bad = <List items={users} getKey={(u) => u.id} renderItem={(u) => u.email} />;
```

```console
$ npx tsc -p . && echo "no type errors"
no type errors
```

Every `@ts-expect-error` line is an assertion that the code below it does *not* compile — if a future change made it compile (a weaker type, an accidental `any`), `tsc` would report the now-unused directive. Type-level tests are written this way in most React libraries.

### Pattern

**P-9. Let TypeScript infer what it can (state from initial values, generic parameters from props, return types of components), annotate boundaries explicitly (props, context values, reducer actions, API responses after validation), and never use `any` to silence an error — use `unknown` and narrow, or fix the type.** *(TypeScript Handbook, "Everyday Types"; React documentation, "Using TypeScript")* *Trade-off:* precise generic types are harder to read in error messages; keep the most complex types inside library code, and give components simple, documented props.

## Chapter 9 Project — A typed, sortable data table

### Requirements

Write a generic `DataTable<T>` that renders rows of any type with a column list, where each column either names a *sortable field* of `T` (a field whose type is a string, number, boolean, or date — not an array or object) or is a computed column that must provide a `render` function. Sorting by a column header toggles ascending and descending and sets `aria-sort`. An initial sort must name a sortable field. Prove with `@ts-expect-error` that invalid columns and sorts do not compile, and render a table on the server to check the output.

### Reference solution

```tsx
// file: project/DataTable.tsx
// A generic, sortable, accessible table. Columns are checked against the row type at compile time.
import { useState, type ReactNode } from "react";

type Primitive = string | number | boolean | Date | null | undefined;
type SortableKey<T> = { [K in keyof T]: T[K] extends Primitive ? K : never }[keyof T];   // keys of sortable fields

export type Column<T> =
  | { key: SortableKey<T>; header: string; render?: (row: T) => ReactNode; sortable?: boolean }
  | { key: string; header: string; render: (row: T) => ReactNode; sortable?: false };     // computed columns

interface Props<T> {
  rows: readonly T[];
  columns: readonly Column<T>[];
  rowKey: (row: T) => string | number;
  caption: string;
  initialSort?: { key: SortableKey<T>; dir: "asc" | "desc" };
}

const compare = (a: Primitive, b: Primitive) =>
  a == null ? 1 : b == null ? -1 : a instanceof Date && b instanceof Date ? a.getTime() - b.getTime()
  : typeof a === "string" && typeof b === "string" ? a.localeCompare(b) : Number(a) - Number(b);

export function DataTable<T>({ rows, columns, rowKey, caption, initialSort }: Props<T>) {
  const [sort, setSort] = useState(initialSort ?? null);
  const sorted = sort
    ? [...rows].sort((x, y) => compare(x[sort.key] as Primitive, y[sort.key] as Primitive) * (sort.dir === "asc" ? 1 : -1))
    : rows;
  return (
    <table>
      <caption>{caption}</caption>
      <thead><tr>{columns.map((c) => {
        const active = sort?.key === c.key;
        return (
          <th key={String(c.key)} scope="col" aria-sort={active ? (sort!.dir === "asc" ? "ascending" : "descending") : undefined}>
            {c.sortable ? (
              <button onClick={() => setSort({ key: c.key as SortableKey<T>, dir: active && sort!.dir === "asc" ? "desc" : "asc" })}>
                {c.header}
              </button>
            ) : c.header}
          </th>
        );
      })}</tr></thead>
      <tbody>{sorted.map((row) => (
        <tr key={rowKey(row)}>{columns.map((c) => (
          <td key={String(c.key)}>{c.render ? c.render(row) : String((row as Record<string, unknown>)[c.key as string] ?? "")}</td>
        ))}</tr>
      ))}</tbody>
    </table>
  );
}
```

```tsx
// file: project/orders.tsx
import { renderToString } from "react-dom/server";
import { DataTable, type Column } from "./DataTable.tsx";

interface Order { id: number; customer: string; total: number; placed: Date; tags: string[] }
const orders: Order[] = [
  { id: 1, customer: "Bo", total: 25.5, placed: new Date("2026-03-01"), tags: ["gift"] },
  { id: 2, customer: "Ana", total: 120, placed: new Date("2026-01-15"), tags: [] },
  { id: 3, customer: "Cy", total: 9.99, placed: new Date("2026-02-10"), tags: ["express", "gift"] },
];
const money = new Intl.NumberFormat("en-US", { style: "currency", currency: "USD" });
const columns: Column<Order>[] = [
  { key: "id", header: "#", sortable: true },
  { key: "customer", header: "Customer", sortable: true },
  { key: "total", header: "Total", sortable: true, render: (o) => money.format(o.total) },
  { key: "placed", header: "Placed", sortable: true, render: (o) => o.placed.toISOString().slice(0, 10) },
  { key: "tagList", header: "Tags", render: (o) => o.tags.join(", ") || "—" },   // computed: needs render
];

const html = renderToString(<DataTable rows={orders} columns={columns} rowKey={(o) => o.id} caption="Orders"
                                       initialSort={{ key: "total", dir: "desc" }} />);
console.log(html.replace(/<\/tr>/g, "</tr>\n"));

// Compile-time checks: each line must fail to type-check.
// @ts-expect-error: "tags" is an array, not a sortable field
const _a: Column<Order> = { key: "tags", header: "Tags", sortable: true };
// @ts-expect-error: a computed column must have a render function
const _b: Column<Order> = { key: "tagList", header: "Tags" };
// @ts-expect-error: "price" is not a field of Order
const _c = <DataTable rows={orders} columns={columns} rowKey={(o) => o.id} caption="x" initialSort={{ key: "price", dir: "asc" }} />;
```

```console
$ npx tsc -p . && echo "no type errors" && npx tsx project/orders.tsx
no type errors
<table><caption>Orders</caption><thead><tr><th scope="col"><button>#</button></th><th scope="col"><button>Customer</button></th><th scope="col" aria-sort="descending"><button>Total</button></th><th scope="col"><button>Placed</button></th><th scope="col">Tags</th></tr>
</thead><tbody><tr><td>2</td><td>Ana</td><td>$120.00</td><td>2026-01-15</td><td>—</td></tr>
<tr><td>1</td><td>Bo</td><td>$25.50</td><td>2026-03-01</td><td>gift</td></tr>
<tr><td>3</td><td>Cy</td><td>$9.99</td><td>2026-02-10</td><td>express, gift</td></tr>
</tbody></table>
```

The three invalid uses fail to compile exactly as asserted, and the rendered table is sorted by total, descending, with `aria-sort="descending"` on that header. `SortableKey<T>` is a mapped type that keeps only the keys whose value types are primitive: for `Order`, `"id" | "customer" | "total" | "placed"` — not `"tags"`.

### Review checklist

- `DataTable` uses two `as` casts. Can you remove them? What would that cost in the column type?
- Add a `filter` prop typed as `(row: T) => boolean`. Does `T` still infer correctly from `rows`?
- Make sorting stable when two rows are equal, and explain why `Array.prototype.sort` already is.

## Answers to the Chapter 9 prediction questions

**Unit 9.1, Q1.** Yes: `ButtonProps` extends `ComponentProps<"button">`, which includes every ARIA attribute and every event handler a button accepts.

# Chapter 10 — Styling React applications

*Goal: choose and use the main ways React applications are styled today. Plain CSS still does the work (Book 1 of this series covers it); the question for a component library is how styles are scoped to components, how variants are expressed, and how much CSS is shipped. You will use CSS Modules (local class names generated at build time), Tailwind CSS v4 (utility classes generated from the files that use them), and build a variant system for a button component — measuring the computed styles in a real browser each time.*

**What you will be able to do after this chapter**

- Scope styles to a component with CSS Modules, and explain how the class names are generated.
- Use Tailwind CSS utilities, responsive and state variants, theme tokens, and arbitrary values.
- Explain why the order of class names in `className` does not decide which rule wins.
- Express component variants as data rather than as conditionals scattered in JSX.

The chapter's pages share one Vite configuration and one HTML template:

```js
// file: vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],                 // Tailwind v4: a Vite plugin, configured in CSS
  logLevel: "warn",
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client"] },
  build: { rollupOptions: { input: { tw: "tw/index.html", project: "project/index.html" } } },
});
```

```html
<!-- file: page.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Chapter 10</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```console
$ for d in modules tw project; do mkdir -p $d && cp page.html $d/index.html; done
```

## Topic 10A — Scoping styles

## Unit 10.1 — CSS Modules

**One-line summary:** A file named `*.module.css` is a CSS Module: importing it gives an object that maps each class name you wrote to a unique generated name, so two components can both use `.title` without clashing; `composes` reuses another class; everything else is ordinary CSS.

```css
/* file: modules/Card.module.css */
/* A CSS Module: every class name is local to the file that imports it. */
.card { border: 1px solid #ccc; border-radius: 8px; padding: 1rem; }
.title { margin: 0; font-size: 1.25rem; }
.highlighted { composes: card; border-color: rebeccapurple; }   /* reuse another class */
```

```css
/* file: modules/Alert.module.css */
.title { color: darkred; font-weight: bold; }                  /* the same name as in Card: no clash */
```

```jsx
// file: modules/main.jsx
import { createRoot } from "react-dom/client";
import card from "./Card.module.css";
import alert from "./Alert.module.css";

console.log("Card classes:", JSON.stringify(card));
console.log("Alert classes:", JSON.stringify(alert));
function App() {
  return <>
    <article className={card.highlighted}><h2 className={card.title}>Card title</h2></article>
    <p className={alert.title}>Alert title</p>
  </>;
}
createRoot(document.getElementById("root")).render(<App />);
window.colors = () => [...document.querySelectorAll("h2, p")].map((e) => `${e.textContent}: ${getComputedStyle(e).color}, ${getComputedStyle(e).fontSize}`);
```

```console
$ python3 ../tools/page.py http://localhost:5173/modules/ "eval:colors()" "eval:document.querySelector('article').className" --serve "npx vite --port 5173 --strictPort"
Card classes: {"card":"_card_4fkhr_3","title":"_title_4fkhr_4","highlighted":"_highlighted_4fkhr_5 _card_4fkhr_3"}
Alert classes: {"title":"_title_yngc3_2"}
=> ["Card title: rgb(0, 0, 0), 20px", "Alert title: rgb(139, 0, 0), 16px"]
=> "_highlighted_4fkhr_5 _card_4fkhr_3"
```

Both files define `.title`, and each import receives a different generated name (the file's hash is part of it), so the card title is black at 20 px and the alert title is dark red at the default 16 px. `composes: card` makes `highlighted` expand to *two* class names. The CSS stays plain CSS — selectors, media queries, and cascade layers all work — but a class can only be reached through the import, which makes unused classes easy to find and deletions safe.

### Deep & Tricky

**DT-18. CSS Modules scope *class names*, not *styles*: a rule such as `.card p { … }` or `h2 { … }` in a module still applies to every matching element on the page, because only the class selector is renamed — element selectors, attribute selectors, and the cascade are untouched. A module that styles bare elements leaks exactly as a global stylesheet does.** *(CSS Modules specification, "Naming" and "Composition"; Vite documentation, "CSS Modules")*

### Mastery check

1. **Predict.** In `Card.module.css`, you add `h2 { color: green }`. Which headings turn green — the card's only, or every `h2` on the page?

## Topic 10B — Utility-first CSS

## Unit 10.2 — Tailwind CSS v4

**One-line summary:** Tailwind generates small single-purpose classes (`p-4`, `text-2xl`, `bg-brand`) for the class names it finds in your source files; variants are prefixes (`hover:`, `md:`, `dark:`, `disabled:`); design tokens declared in CSS with `@theme` become new utilities; square brackets give arbitrary values (`w-[37px]`); and in v4 it is configured in CSS and plugged into Vite with `@tailwindcss/vite`.

```css
/* file: tw/style.css */
@import "tailwindcss";                  /* Tailwind v4: generates only the utilities used in your files */
@theme {                                /* design tokens become utilities: bg-brand, text-brand ... */
  --color-brand: oklch(55% 0.2 260);
}
```

```jsx
// file: tw/main.jsx
import "./style.css";
import { createRoot } from "react-dom/client";

function App() {
  return <main className="mx-auto max-w-md p-4">
    <h1 className="text-2xl font-bold text-brand">Tailwind</h1>
    <button id="a" className="rounded bg-brand px-4 py-2 text-white hover:opacity-90 md:px-8">Responsive padding</button>
    <button id="b" className="p-4 p-2">"p-4 p-2"</button>
    <button id="c" className="p-2 p-4">"p-2 p-4"</button>
    <p id="d" className="w-[37px] dark:text-yellow-300">arbitrary value</p>
  </main>;
}
createRoot(document.getElementById("root")).render(<App />);
window.pads = () => ["a", "b", "c"].map((id) => `${id}: ${getComputedStyle(document.getElementById(id)).paddingLeft}`)
  .concat(`d width ${getComputedStyle(document.getElementById("d")).width}`);
```

```console
$ python3 ../tools/page.py http://localhost:5173/tw/ "eval:pads()" viewport:1000x600 "eval:pads()" --width 500 --serve "npx vite --port 5173 --strictPort"
=> ["a: 16px", "b: 16px", "c: 16px", "d width 37px"]
=> ["a: 32px", "b: 16px", "c: 16px", "d width 37px"]
```

At 500 px wide, button `a` has 16 px of horizontal padding (`px-4`); at 1000 px, `md:px-8` applies (the `md` breakpoint is 48rem, 768 px) and it is 32 px. Buttons `b` and `c` are the surprise: `"p-4 p-2"` and `"p-2 p-4"` both give 16 px.

```console
$ npx vite build --logLevel info 2>&1 | grep "dist/"
dist/tw/index.html                     0.35 kB │ gzip:  0.25 kB
dist/project/index.html                0.36 kB │ gzip:  0.25 kB
dist/assets/tw-DGaKNPvY.css            7.21 kB │ gzip:  2.32 kB
dist/assets/tw-Co--uKyz.js             0.84 kB │ gzip:  0.46 kB
dist/assets/project-B_puaOCW.js        1.59 kB │ gzip:  0.83 kB
dist/assets/jsx-runtime-DsRmeUu0.js  219.39 kB │ gzip: 68.53 kB
```

The built stylesheet holds only the utilities used in the source files, plus Tailwind's base styles — a few kilobytes, whatever the size of the framework.

### Deep & Tricky

**DT-19. The order of class names in a `class` attribute has no effect on the cascade: when two rules of equal specificity set the same property, the one *later in the stylesheet* wins. Tailwind emits `p-2` before `p-4`, so `p-4` wins in both buttons. This is why a component cannot let callers "override" a utility by appending another one (`className={"p-4 " + props.className}` with `"p-2"`), and why libraries such as `tailwind-merge` exist: they remove the conflicting class instead.** *(CSS Cascading and Inheritance Level 5 §6.1 "Cascade Sorting Order" — order of appearance; Tailwind CSS documentation, "Styling with utility classes — Managing style conflicts")*

**DT-20. Tailwind finds class names by scanning source files as plain text, so a class must appear whole in the source: `` `bg-${color}-600` `` generates nothing, and the element is unstyled. Map props to complete class strings (`{ danger: "bg-red-600" }`) instead.** *(Tailwind CSS documentation, "Detecting classes in source files — Dynamic class names")*

### Mastery check

1. **Predict.** At 1000 px, what is button `a`'s left padding if its classes are `md:px-8 px-4`?
2. **Explain.** Why is `w-[37px]` exactly 37 px, when `w-9` would be 36 px?

## Unit 10.3 — Choosing a styling approach

**One-line summary:** Plain global CSS, CSS Modules, utility classes, and CSS-in-JS solve the same scoping problem with different costs; for server-rendered React (Chapter 12), prefer approaches that produce static CSS at build time — runtime CSS-in-JS libraries that inject styles while rendering do not work in React Server Components.

| Approach | Scoping | Variants | Runtime cost | Works in Server Components |
|---|---|---|---|---|
| Global CSS (BEM naming) | by convention | modifier classes | none | yes |
| CSS Modules | generated class names | extra classes, `composes` | none | yes |
| Tailwind CSS | none needed (utilities) | prefixes, class maps | none | yes |
| Runtime CSS-in-JS (styled-components, Emotion) | generated at run time | props in JS | serialises styles while rendering | client components only |
| Zero-runtime CSS-in-JS (vanilla-extract, Panda) | generated at build time | typed variant APIs | none | yes |

### Pattern

**P-10. Keep a component's visual variants in one table (variant name → classes), choose from it by props, and let callers pass layout-only classes (margin, width, grid placement) through `className` — not colours or padding, which conflict with the variants (DT-19).** *(Tailwind CSS documentation, "Reusing styles — Using components"; the class-variance-authority pattern)* *Trade-off:* a closed variant table means a new look needs a change to the component, which is the point for a design system and a nuisance for a one-off page.

## Chapter 10 Project — A variant-driven button

### Requirements

Write a `variants()` helper that turns `{ base, variants, defaults, compound }` into a function from props to a class string, and use it to build a `Button` with `intent` (primary, secondary, danger) and `size` (sm, md, lg), where a large danger button is also upper-case. `undefined` props must fall back to the defaults. Check the computed styles of four buttons in a browser.

### Reference solution

```js
// file: project/variants.js
// A small variant helper (the idea behind libraries such as cva): turn a component's props into
// a class list, with defaults and combinations, so that styling rules live in one table.
export function variants({ base = "", variants = {}, defaults = {}, compound = [] }) {
  return (props = {}) => {
    const chosen = { ...defaults, ...Object.fromEntries(Object.entries(props).filter(([, v]) => v !== undefined)) };
    const classes = [base];
    for (const [name, options] of Object.entries(variants)) classes.push(options[chosen[name]] ?? "");
    for (const { when, className } of compound)
      if (Object.entries(when).every(([k, v]) => chosen[k] === v)) classes.push(className);
    return classes.filter(Boolean).join(" ");
  };
}
```

```jsx
// file: project/Button.jsx
import { variants } from "./variants.js";

const button = variants({
  base: "inline-flex items-center gap-2 rounded-md font-medium focus-visible:outline-2 focus-visible:outline-offset-2 disabled:opacity-50",
  variants: {
    intent: { primary: "bg-brand text-white", secondary: "bg-zinc-100 text-zinc-900", danger: "bg-red-600 text-white" },
    size: { sm: "px-2 py-1 text-sm", md: "px-4 py-2 text-base", lg: "px-6 py-3 text-lg" },
  },
  defaults: { intent: "primary", size: "md" },
  compound: [{ when: { intent: "danger", size: "lg" }, className: "uppercase tracking-wide" }],
});

export function Button({ intent, size, className = "", ...props }) {
  return <button className={`${button({ intent, size })} ${className}`} {...props} />;
}
```

```css
/* file: project/style.css */
@import "tailwindcss";
@theme { --color-brand: oklch(55% 0.2 260); }
```

```jsx
// file: project/main.jsx
import "./style.css";
import { createRoot } from "react-dom/client";
import { Button } from "./Button.jsx";

createRoot(document.getElementById("root")).render(<p className="flex gap-2 p-4">
  <Button id="b1">Save</Button>
  <Button id="b2" intent="secondary" size="sm">Cancel</Button>
  <Button id="b3" intent="danger" size="lg">Delete everything</Button>
  <Button id="b4" disabled>Disabled</Button>
</p>);
window.styles = () => ["b1", "b2", "b3", "b4"].map((id) => {
  const s = getComputedStyle(document.getElementById(id));
  return `${id}: bg ${s.backgroundColor}, padding ${s.paddingTop} ${s.paddingLeft}, font ${s.fontSize}, ${s.textTransform}, opacity ${s.opacity}`;
});
```

```console
$ python3 ../tools/page.py http://localhost:5173/project/ "eval:styles()" "eval:document.getElementById('b3').className" --serve "npx vite --port 5173 --strictPort"
=> ["b1: bg oklch(0.55 0.2 260), padding 8px 16px, font 16px, none, opacity 1", "b2: bg oklch(0.967 0.001 286.375), padding 4px 8px, font 14px, none, opacity 1", "b3: bg oklch(0.577 0.245 27.325), padding 12px 24px, font 18px, uppercase, opacity 1", "b4: bg oklch(0.55 0.2 260), padding 8px 16px, font 16px, none, opacity 0.5"]
=> "inline-flex items-center gap-2 rounded-md font-medium focus-visible:outline-2 focus-visible:outline-offset-2 disabled:opacity-50 bg-red-600 text-white px-6 py-3 text-lg uppercase tracking-wide "
```

The default button uses the brand colour and medium padding (8 × 16 px); `secondary sm` is grey with 4 × 8 px; `danger lg` adds the compound `uppercase`; and the disabled button is at half opacity from the `disabled:` variant. Filtering `undefined` before merging matters: `{ intent: undefined }` from a destructured prop would otherwise overwrite the default and select no colour at all.

### Review checklist

- `Button` appends `className` after the variant classes. Which caller classes would silently fail, and why (DT-19)?
- Add an `icon` slot and a `loading` state that disables the button and shows a spinner, without a new variant per combination.
- Write unit tests for `variants()` (defaults, compound rules, unknown values) — Chapter 11 shows how.

## Answers to the Chapter 10 prediction questions

**Unit 10.1, Q1.** Every `h2` on the page: only class selectors are renamed (DT-18).

**Unit 10.2, Q1.** 32 px — order in the attribute does not matter; the `md:` rule is inside a media query emitted after the base utilities.

**Unit 10.2, Q2.** Spacing utilities are multiples of 0.25rem (`w-9` = 2.25rem = 36 px); square brackets bypass the scale and use the value as written.

# Chapter 11 — Testing React applications

*Goal: write tests that give confidence a change has not broken the application, without breaking every time the implementation is refactored. You will use Vitest for unit tests, Testing Library to test components the way users use them — by role and label, not by class name or state — and Playwright to drive the real application in a real browser. Along the way you will meet the failures every React developer meets once: fake timers that hang, queries that run too early, and an input that has no role at all.*

**What you will be able to do after this chapter**

- Write unit tests with Vitest, including table-driven and property-style tests.
- Test components with Testing Library and `user-event`, using accessible queries.
- Test asynchronous UI with `findBy…`, fake timers, and a fake `fetch`.
- Write end-to-end tests with Playwright, including intercepted network requests.
- Read a coverage report and know what it does not measure.

## Topic 11A — Unit and component tests

## Unit 11.1 — Vitest

**One-line summary:** Vitest runs tests with Vite's transformer, so JSX, TypeScript, and ES modules work with no extra setup; tests are grouped with `describe`, written with `it`, and checked with `expect`; `it.each` turns a table into tests; and an environment option (`jsdom`) supplies a simulated DOM for component tests.

```js
// file: vitest.config.js
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",                  // a simulated DOM in Node.js
    include: ["unit/**/*.test.*", "components/**/*.test.*", "project/**/*.test.*"],
    exclude: process.env.PITFALLS ? [] : ["components/pitfalls.test.jsx"],   // failing on purpose (Unit 11.2)
    unstubGlobals: true,                   // undo vi.stubGlobal after each test
    coverage: { provider: "v8", include: ["project/**/*.jsx"], exclude: ["**/*.test.*"] },
    setupFiles: ["./setup.js"],
  },
});
```

```js
// file: setup.js
import { afterEach, expect, vi } from "vitest";
import { cleanup } from "@testing-library/react";
import * as matchers from "@testing-library/jest-dom/matchers";

expect.extend(matchers);                   // toBeDisabled, toHaveTextContent, toHaveAccessibleDescription ...
afterEach(cleanup);                        // unmount everything rendered by the previous test
// Testing Library advances fake timers only through a global named `jest` (DT-22).
globalThis.jest = vi;
```

```js
// file: unit/money.js
export function formatCents(cents, currency = "USD") {
  if (!Number.isInteger(cents)) throw new TypeError(`cents must be an integer, got ${cents}`);
  return new Intl.NumberFormat("en-US", { style: "currency", currency }).format(cents / 100);
}
export function splitBill(totalCents, people) {
  const base = Math.floor(totalCents / people);
  const extra = totalCents - base * people;          // the leftover cents go to the first people
  return Array.from({ length: people }, (_, i) => base + (i < extra ? 1 : 0));
}
```

```js
// file: unit/money.test.js
import { describe, it, expect } from "vitest";
import { formatCents, splitBill } from "./money.js";

describe("formatCents", () => {
  it.each([[0, "$0.00"], [5, "$0.05"], [123456, "$1,234.56"], [-250, "-$2.50"]])("%i → %s", (cents, text) => {
    expect(formatCents(cents)).toBe(text);
  });
  it("rejects fractions of a cent", () => {
    expect(() => formatCents(1.5)).toThrow(/integer/);
  });
});

describe("splitBill", () => {
  it("never loses a cent", () => {
    for (let total = 0; total < 500; total++)
      for (let people = 1; people < 8; people++)
        expect(splitBill(total, people).reduce((a, b) => a + b, 0)).toBe(total);
  });
  it("gives the leftover to the first people", () => {
    expect(splitBill(1000, 3)).toEqual([334, 333, 333]);
  });
});
```

```console
$ npx vitest run unit --reporter=verbose 2>&1 | grep -E "✓|×|Tests " | sed -E "s/ [0-9]+ms$//"
 ✓ unit/money.test.js > formatCents > 0 → $0.00
 ✓ unit/money.test.js > formatCents > 5 → $0.05
 ✓ unit/money.test.js > formatCents > 123456 → $1,234.56
 ✓ unit/money.test.js > formatCents > -250 → -$2.50
 ✓ unit/money.test.js > formatCents > rejects fractions of a cent
 ✓ unit/money.test.js > splitBill > never loses a cent
 ✓ unit/money.test.js > splitBill > gives the leftover to the first people
      Tests  7 passed (7)
```

(Timings are removed from the output with `sed`; they change on every run.) The "never loses a cent" test checks a *property* over 3,500 inputs rather than one example: whatever the total and the number of people, the parts add up to the total. A single example (`[334, 333, 333]`) documents the rule for the leftover cents.

### Mastery check

1. **Write.** Add a test that `splitBill` never gives two people amounts that differ by more than one cent.

## Unit 11.2 — Testing Library and user-event

**One-line summary:** Testing Library renders a component into the DOM and finds elements the way a user (or a screen reader) would — `getByRole("button", { name: "Sign up" })`, `getByLabelText("Email")` — so tests survive refactoring and fail when accessibility breaks; `user-event` simulates complete interactions (focus, key presses, clicks); `getBy…` throws if the element is not there *now*, `findBy…` waits for it, and `queryBy…` returns `null` for asserting absence.

```jsx
// file: components/SignupForm.jsx
import { useState } from "react";

export function SignupForm({ onSignup }) {
  const [errors, setErrors] = useState({});
  const [status, setStatus] = useState("idle");      // idle | saving | done | failed

  async function handleSubmit(event) {
    event.preventDefault();
    const data = Object.fromEntries(new FormData(event.currentTarget));
    const found = {};
    if (!/^\S+@\S+\.\S+$/.test(data.email)) found.email = "Enter a valid email address";
    if (data.password.length < 8) found.password = "Use at least 8 characters";
    setErrors(found);
    if (Object.keys(found).length) return;
    setStatus("saving");
    try { await onSignup(data); setStatus("done"); }
    catch { setStatus("failed"); }
  }

  if (status === "done") return <p role="status">Welcome aboard!</p>;
  return (
    <form onSubmit={handleSubmit} noValidate>
      <label htmlFor="email">Email</label>
      <input id="email" name="email" type="email" aria-invalid={!!errors.email} aria-describedby="email-error" />
      <p id="email-error">{errors.email}</p>
      <label htmlFor="password">Password</label>
      <input id="password" name="password" type="password" aria-invalid={!!errors.password} aria-describedby="password-error" />
      <p id="password-error">{errors.password}</p>
      {status === "failed" && <p role="alert">Sign-up failed. Try again.</p>}
      <button disabled={status === "saving"}>{status === "saving" ? "Signing up…" : "Sign up"}</button>
    </form>
  );
}
```

```jsx
// file: components/SignupForm.test.jsx
import { describe, it, expect, vi } from "vitest";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { SignupForm } from "./SignupForm.jsx";

function setup(onSignup = vi.fn().mockResolvedValue(undefined)) {
  const user = userEvent.setup();                    // simulates real key presses and clicks
  render(<SignupForm onSignup={onSignup} />);
  return { user, onSignup };
}

describe("SignupForm", () => {
  it("shows errors next to the fields and does not submit", async () => {
    const { user, onSignup } = setup();
    await user.type(screen.getByLabelText("Email"), "ana@");
    await user.type(screen.getByLabelText("Password"), "short");
    await user.click(screen.getByRole("button", { name: "Sign up" }));

    expect(screen.getByLabelText("Email")).toHaveAccessibleDescription("Enter a valid email address");
    expect(screen.getByLabelText("Password")).toHaveAttribute("aria-invalid", "true");
    expect(onSignup).not.toHaveBeenCalled();
  });

  it("submits valid data and shows a welcome message", async () => {
    const { user, onSignup } = setup();
    await user.type(screen.getByLabelText("Email"), "ana@example.org");
    await user.type(screen.getByLabelText("Password"), "correct horse");
    await user.click(screen.getByRole("button", { name: "Sign up" }));

    expect(onSignup).toHaveBeenCalledWith({ email: "ana@example.org", password: "correct horse" });
    expect(await screen.findByRole("status")).toHaveTextContent("Welcome aboard!");
  });

  it("disables the button while saving, then reports a failure", async () => {
    let reject;
    const { user } = setup(() => new Promise((_, r) => { reject = r; }));
    await user.type(screen.getByLabelText("Email"), "ana@example.org");
    await user.type(screen.getByLabelText("Password"), "correct horse");
    await user.click(screen.getByRole("button", { name: "Sign up" }));

    expect(screen.getByRole("button", { name: "Signing up…" })).toBeDisabled();
    reject(new Error("500"));
    expect(await screen.findByRole("alert")).toHaveTextContent("Sign-up failed");
    expect(screen.getByRole("button", { name: "Sign up" })).toBeEnabled();
  });
});
```

```jsx
// file: components/SearchBox.jsx
import { useEffect, useState } from "react";

export function SearchBox({ onSearch, delay = 300 }) {
  const [text, setText] = useState("");
  useEffect(() => {
    if (!text) return;
    const id = setTimeout(() => onSearch(text), delay);   // debounce: wait for a pause in typing
    return () => clearTimeout(id);
  }, [text, delay, onSearch]);
  return <input aria-label="Search" value={text} onChange={(e) => setText(e.target.value)} />;
}
```

```jsx
// file: components/SearchBox.test.jsx
import { describe, it, expect, vi, afterEach } from "vitest";
import { render, screen, act } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { SearchBox } from "./SearchBox.jsx";

afterEach(() => vi.useRealTimers());

describe("SearchBox", () => {
  it("searches once, after the user stops typing", async () => {
    vi.useFakeTimers();                                   // control setTimeout from the test
    const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });
    const onSearch = vi.fn();
    render(<SearchBox onSearch={onSearch} />);

    await user.type(screen.getByRole("textbox", { name: "Search" }), "react");
    expect(onSearch).not.toHaveBeenCalled();              // still inside the 300 ms pause
    act(() => vi.advanceTimersByTime(299));
    expect(onSearch).not.toHaveBeenCalled();
    act(() => vi.advanceTimersByTime(1));
    expect(onSearch).toHaveBeenCalledTimes(1);
    expect(onSearch).toHaveBeenCalledWith("react");
  });
});
```

```console
$ npx vitest run components --reporter=verbose 2>&1 | grep -E "✓|×|Tests " | sed -E "s/ [0-9]+ms$//"
 ✓ components/SignupForm.test.jsx > SignupForm > shows errors next to the fields and does not submit
 ✓ components/SignupForm.test.jsx > SignupForm > submits valid data and shows a welcome message
 ✓ components/SignupForm.test.jsx > SignupForm > disables the button while saving, then reports a failure
 ✓ components/SearchBox.test.jsx > SearchBox > searches once, after the user stops typing
      Tests  4 passed (4)
```

The tests never look at state, CSS classes, or component internals: a user cannot see them. `toHaveAccessibleDescription` checks the error through `aria-describedby`, which is what a screen reader announces — so the test fails if someone breaks that link. The third test controls the "server" with a promise it settles itself, so it can observe the in-between state (the disabled "Signing up…" button) deterministically.

Now the three classic failures:

```jsx
// file: components/pitfalls.test.jsx
// Three tests that fail for reasons worth recognising.
import { it, expect, vi, afterEach } from "vitest";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { SearchBox } from "./SearchBox.jsx";
import { SignupForm } from "./SignupForm.jsx";

afterEach(() => vi.useRealTimers());

it("fake timers without telling user-event", async () => {
  vi.useFakeTimers();
  const user = userEvent.setup();                         // user-event waits with setTimeout between keys...
  render(<SearchBox onSearch={() => {}} />);
  await user.type(screen.getByRole("textbox"), "hi");     // ...which never fires: the test times out
});

it("getBy for something that appears later", async () => {
  const user = userEvent.setup();
  render(<SignupForm onSignup={() => new Promise((r) => setTimeout(r, 20))} />);  // a 20 ms "server"
  await user.type(screen.getByLabelText("Email"), "ana@example.org");
  await user.type(screen.getByLabelText("Password"), "correct horse");
  await user.click(screen.getByRole("button", { name: "Sign up" }));
  expect(screen.getByRole("status")).toBeInTheDocument();  // not there yet: use findByRole
});

it("a query that matches the wrong thing", () => {
  render(<SignupForm onSignup={() => {}} />);
  expect(screen.getByRole("textbox", { name: "Password" })).toBeInTheDocument();
});
```

```console
$ PITFALLS=1 COLORS=false npx vitest run components/pitfalls --testTimeout=1000 2>&1 | grep -E "×|Error|Unable|Name \"" | sed -E "s/ [0-9]+ms$//"
   × fake timers without telling user-event
   × getBy for something that appears later
   × a query that matches the wrong thing
Error: Test timed out in 1000ms.
TestingLibraryElementError: Unable to find an accessible element with the role "status"
  Name "Email":
  Name "":
  Name "":
  Name "Signing up…":
 ❯ Object.getElementError ../node_modules/@testing-library/dom/dist/config.js:37:19
TestingLibraryElementError: Unable to find an accessible element with the role "textbox" and name "Password"
  Name "Email":
  Name "":
  Name "":
  Name "Sign up":
 ❯ Object.getElementError ../node_modules/@testing-library/dom/dist/config.js:37:19
```

(`PITFALLS=1` includes the file that the configuration normally excludes; `COLORS=false` turns off Testing Library's colours; `grep` keeps the interesting lines of a long report.) The first test hangs: `user-event` waits between key presses with `setTimeout`, which fake timers have frozen — pass `advanceTimers`, as `SearchBox.test.jsx` does. The second looks for the welcome message while the 20 ms request is still running; the error shows the button already says "Signing up…". The third is a wrong belief about roles.

### Deep & Tricky

**DT-21. `<input type="password">` has no ARIA role — deliberately, so that assistive technology does not treat its contents like ordinary text — so `getByRole("textbox", { name: "Password" })` cannot find it, and the error lists every role on the page without it. Query password fields with `getByLabelText`.** *(HTML Accessibility API Mappings, `input type=password` — "No corresponding role"; Testing Library documentation, "ByRole")*

**DT-22. Under Vitest's fake timers, Testing Library's asynchronous helpers can hang even when `user-event` is configured: React Testing Library's `asyncWrapper` drains pending work with `setTimeout(…, 0)` and advances fake timers only if a global named `jest` exists and `setTimeout` is a fake. Vitest does not define `jest`, so the zero-delay timeout never fires. `setup.js` assigns `globalThis.jest = vi`; the alternative is to avoid fake timers and wait for real ones with `findBy…`.** *(`@testing-library/react` source, `pure.js` — `asyncWrapper` and `jestFakeTimersAreEnabled`; Vitest documentation, "Fake timers")*

### Pattern

**P-11. Query priority: `getByRole` (with a name) first, then `getByLabelText`, `getByPlaceholderText`, `getByText`, and `getByTestId` only as a last resort; a test that can only find an element by test ID is often reporting an accessibility problem.** *(Testing Library documentation, "About Queries — Priority")* *Trade-off:* role queries are slower than ID lookups on very large DOMs; the cost matters only in suites with thousands of component tests, where scoping queries with `within()` helps.

### Mastery check

1. **Predict.** In `SearchBox.test.jsx`, the user types "react" and the timers are advanced 299 ms. How many times has `setTimeout` been scheduled, and how many cleared?
2. **Fix.** The second pitfall test: change one line so that it passes without fake timers.

## Topic 11B — End-to-end tests

## Unit 11.3 — Playwright

**One-line summary:** Playwright runs tests against the real application in real browsers; `webServer` starts the development server for the test run; locators use the same accessible queries as Testing Library (`getByRole`, `getByLabel`) and *wait automatically*; web-first assertions (`toHaveText`, `toBeDisabled`) retry until they pass or time out; and `page.route` intercepts network requests to simulate slow or failing servers.

```js
// file: vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

// A fake sign-up API: 409 for an address that is already taken, 201 otherwise.
const api = {
  name: "fake-api",
  configureServer(server) {
    server.middlewares.use("/api/signup", (req, res) => {
      let body = "";
      req.on("data", (c) => (body += c));
      req.on("end", () => {
        const { email } = JSON.parse(body);
        res.statusCode = email === "taken@example.org" ? 409 : 201;
        res.setHeader("content-type", "application/json");
        res.end(JSON.stringify({ email }));
      });
    });
  },
};

export default defineConfig({
  plugins: [react(), api],
  logLevel: "warn",
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client"] },
});
```

```html
<!-- file: app/index.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Sign up</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```jsx
// file: app/main.jsx
import { createRoot } from "react-dom/client";
import { SignupForm } from "../components/SignupForm.jsx";

async function signup(data) {
  const res = await fetch("/api/signup", { method: "POST", body: JSON.stringify(data) });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
}
createRoot(document.getElementById("root")).render(<main><h1>Create an account</h1><SignupForm onSignup={signup} /></main>);
```

```js
// file: playwright.config.js
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "e2e",
  reporter: "list",
  use: { baseURL: "http://localhost:5174" },
  webServer: { command: "npx vite --port 5174 --strictPort", url: "http://localhost:5174/app/", reuseExistingServer: false },
});
```

```js
// file: e2e/signup.spec.js
import { test, expect } from "@playwright/test";

test.beforeEach(async ({ page }) => { await page.goto("/app/"); });

test("a new user can sign up", async ({ page }) => {
  await page.getByLabel("Email").fill("ana@example.org");
  await page.getByLabel("Password").fill("correct horse");
  await page.getByRole("button", { name: "Sign up" }).click();
  await expect(page.getByRole("status")).toHaveText("Welcome aboard!");
});

test("a taken address shows an error and keeps the input", async ({ page }) => {
  await page.getByLabel("Email").fill("taken@example.org");
  await page.getByLabel("Password").fill("correct horse");
  await page.getByRole("button", { name: "Sign up" }).click();
  await expect(page.getByRole("alert")).toHaveText("Sign-up failed. Try again.");
  await expect(page.getByLabel("Email")).toHaveValue("taken@example.org");
});

test("the button is disabled while the request is in flight", async ({ page }) => {
  let release;
  await page.route("**/api/signup", async (route) => {        // hold the request until we say so
    await new Promise((r) => (release = r));
    await route.fulfill({ status: 201, body: "{}" });
  });
  await page.getByLabel("Email").fill("ana@example.org");
  await page.getByLabel("Password").fill("correct horse");
  await page.getByRole("button", { name: "Sign up" }).click();
  await expect(page.getByRole("button", { name: "Signing up…" })).toBeDisabled();
  release();
  await expect(page.getByRole("status")).toBeVisible();
});
```

```console
$ npx playwright test 2>&1 | grep -E "✓|✘|passed|failed" | sed -E "s/ \([0-9.]+m?s\)//"
  ✓  1 e2e/signup.spec.js:6:1 › a new user can sign up
  ✓  2 e2e/signup.spec.js:13:1 › a taken address shows an error and keeps the input
  ✓  3 e2e/signup.spec.js:21:1 › the button is disabled while the request is in flight
  3 passed
```

The same `SignupForm` that was tested in jsdom now runs in Chromium against the fake API in `vite.config.js`. The third test holds the request inside `page.route` until the test releases it — the end-to-end version of the controlled promise in Unit 11.2.

### Deep & Tricky

**DT-23. Playwright's `expect(locator).toHaveText(…)` retries until the assertion passes (5 seconds by default), but `expect(await locator.textContent()).toBe(…)` reads the text *once* and fails if the page has not caught up: awaiting the value before the assertion throws away the retry. The same mistake in Testing Library is `getBy…` where `findBy…` was needed.** *(Playwright documentation, "Assertions — Auto-retrying assertions" and "Non-retrying assertions")*

### Pattern

**P-12. The testing trophy: many fast component tests of behaviour, a smaller number of end-to-end tests for the journeys that make money or lose data (sign-up, checkout, saving work), unit tests for pure logic, and static checks (TypeScript, ESLint) underneath them all.** *(Kent C. Dodds, "The Testing Trophy and Testing Classifications"; Playwright documentation, "Best Practices")* *Trade-off:* end-to-end tests catch integration bugs nothing else does, but they are slower and fail for reasons outside the code (timing, environment); keep them few and stable.

## Chapter 11 Project — Testing a paginated list

### Requirements

`UserList` fetches pages of users, shows a loading state, an error with a Retry button, and Previous/Next buttons that are disabled at the ends. Write component tests that replace `fetch` with a fake server (pages, an `hasMore` flag, and a switch to fail the next request), cover loading, paging forward and back (checking the exact sequence of requests), and error recovery. Then measure coverage.

### Reference solution

```jsx
// file: project/UserList.jsx
import { useEffect, useState } from "react";

export function UserList({ pageSize = 2 }) {
  const [page, setPage] = useState(1);
  const [attempt, setAttempt] = useState(0);
  const [state, setState] = useState({ status: "loading" });

  useEffect(() => {
    const controller = new AbortController();
    setState({ status: "loading" });
    fetch(`/api/users?page=${page}&size=${pageSize}`, { signal: controller.signal })
      .then((res) => { if (!res.ok) throw new Error(`HTTP ${res.status}`); return res.json(); })
      .then((data) => setState({ status: "ok", ...data }))
      .catch((error) => { if (error.name !== "AbortError") setState({ status: "error", error }); });
    return () => controller.abort();
  }, [page, pageSize, attempt]);

  return (
    <section aria-labelledby="users-h">
      <h2 id="users-h">Users</h2>
      {state.status === "loading" && <p role="status">Loading…</p>}
      {state.status === "error" && <p role="alert">Could not load users ({state.error.message}).{" "}
        <button onClick={() => setAttempt((a) => a + 1)}>Retry</button></p>}
      {state.status === "ok" && <ul>{state.users.map((u) => <li key={u.id}>{u.name}</li>)}</ul>}
      <nav aria-label="Pages">
        <button onClick={() => setPage((p) => p - 1)} disabled={page === 1}>Previous</button>
        <span>Page {page}</span>
        <button onClick={() => setPage((p) => p + 1)} disabled={state.status !== "ok" || !state.hasMore}>Next</button>
      </nav>
    </section>
  );
}
```

```jsx
// file: project/UserList.test.jsx
import { describe, it, expect, vi, beforeEach } from "vitest";
import { render, screen, within } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { UserList } from "./UserList.jsx";

const USERS = ["Ana", "Bo", "Cy", "Di", "Ed"].map((name, i) => ({ id: i + 1, name }));

// A fake server: pages of users, with a switch to make the next request fail.
function fakeServer() {
  const server = { failNext: false, requests: [] };
  server.fetch = vi.fn(async (url) => {
    const { searchParams } = new URL(url, "http://test");
    const page = Number(searchParams.get("page")), size = Number(searchParams.get("size"));
    server.requests.push(`page ${page}`);
    if (server.failNext) { server.failNext = false; return new Response("", { status: 503 }); }
    const users = USERS.slice((page - 1) * size, page * size);
    return Response.json({ users, hasMore: page * size < USERS.length });
  });
  return server;
}

let server;
beforeEach(() => { server = fakeServer(); vi.stubGlobal("fetch", server.fetch); });  // restored by unstubGlobals

const names = () => within(screen.getByRole("list")).getAllByRole("listitem").map((li) => li.textContent);

describe("UserList", () => {
  it("shows a loading state, then the first page", async () => {
    render(<UserList />);
    expect(screen.getByRole("status")).toHaveTextContent("Loading…");
    await screen.findByRole("list");
    expect(names()).toEqual(["Ana", "Bo"]);
    expect(screen.getByRole("button", { name: "Previous" })).toBeDisabled();
  });

  it("pages forward to the end and back", async () => {
    const user = userEvent.setup();
    render(<UserList />);
    await screen.findByRole("list");
    await user.click(screen.getByRole("button", { name: "Next" }));
    await screen.findByText("Cy");
    await user.click(screen.getByRole("button", { name: "Next" }));
    await screen.findByText("Ed");
    expect(names()).toEqual(["Ed"]);
    expect(screen.getByRole("button", { name: "Next" })).toBeDisabled();
    await user.click(screen.getByRole("button", { name: "Previous" }));
    await screen.findByText("Di");
    expect(server.requests).toEqual(["page 1", "page 2", "page 3", "page 2"]);
  });

  it("shows an error with a retry button that recovers", async () => {
    const user = userEvent.setup();
    server.failNext = true;
    render(<UserList />);
    expect(await screen.findByRole("alert")).toHaveTextContent("Could not load users (HTTP 503).");
    await user.click(screen.getByRole("button", { name: "Retry" }));
    await screen.findByRole("list");
    expect(names()).toEqual(["Ana", "Bo"]);
    expect(screen.queryByRole("alert")).not.toBeInTheDocument();
  });
});
```

```console
$ npx vitest run project --coverage 2>&1 | grep -E "Tests |File|UserList|Branches"
 Test Files  1 passed (1)
      Tests  3 passed (3)
File          | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s 
 UserList.jsx |     100 |     92.3 |     100 |     100 | 15                
Branches     : 92.3% ( 12/13 )
```

All three tests pass with 100% of lines covered — and one branch uncovered, on line 15: the test suite never aborts a request, so the `AbortError` case (a page change while a request is in flight, or an unmount) has never run. Coverage shows which code *ran*, not which behaviour was *checked*; this uncovered branch is exactly where a race condition (Chapter 3, DT on stale responses) would hide.

### Review checklist

- Write the missing test: click Next twice quickly with a fake server that answers page 2 after page 3. Which page must be shown?
- `fakeServer` builds a `Response` with `Response.json`. Why is that better than returning a plain object with a `json()` method?
- The tests use `findByText("Cy")` to wait for page 2. What would go wrong with `findByRole("list")` instead?

## Answers to the Chapter 11 prediction questions

**Unit 11.2, Q1.** Five timeouts were scheduled (one per key press, as `text` changes) and four cleared by the effect cleanup; the fifth fires at 300 ms.

**Unit 11.2, Q2.** Replace `getByRole("status")` with `await screen.findByRole("status")`, which waits up to 1 second by default.

# Chapter 12 — Next.js and React Server Components

*Goal: understand the model that Next.js's App Router is built on — components that run only on the server, components that also run in the browser, and functions the browser can call on the server — well enough to predict what is sent to the browser, when a page is rendered, and what is cached. You will look inside the build output and the wire format, stream a page whose parts arrive at different times, cache data with `"use cache"`, and build a guestbook with validated Server Functions and cache invalidation. Everything here was run with Next.js 16.4.6.*

**What you will be able to do after this chapter**

- Explain the difference between Server Components, Client Components, and server-side rendering.
- Decide where the `"use client"` boundary goes, and what may cross it.
- Write Server Functions (`"use server"`) for forms, and treat them as public endpoints.
- Stream slow parts of a page with `<Suspense>`, and cache data with `"use cache"`, `cacheLife`, and `cacheTag`.
- Read the route table that `next build` prints.

```console
$ npx next telemetry disable | grep Status
Status: Disabled
```

## Topic 12A — Server and client

## Unit 12.1 — Server Components and the `"use client"` boundary

**One-line summary:** In the App Router, components are Server Components by default: they run only on the server (at build time or per request), may be `async`, may read files, databases, and secrets, and send *no JavaScript* to the browser; a module that starts with `"use client"` is a Client Component boundary — it and everything it imports are bundled for the browser, where they hydrate and can use state, effects, and event handlers; Client Components are *also* rendered to HTML on the server.

The `rsc` application has a "database" module that may only be imported by server code (the `server-only` package turns a mistake into a build error), a Server Component page, and one Client Component:

```js
// file: rsc/next.config.mjs
export default {};
```

```js
// file: rsc/lib/db.js
// A stand-in for a database, used only on the server.
import "server-only";
export const SECRET_API_KEY = "sk-live-do-not-ship-1234";
const notes = [{ id: 1, text: "Buy milk" }, { id: 2, text: "Read chapter 12" }];
export async function getNotes() {
  await new Promise((r) => setTimeout(r, 50));
  return structuredClone(notes);
}
export async function addNote(text) { notes.push({ id: notes.length + 1, text }); }
```

```jsx
// file: rsc/app/layout.jsx
export const metadata = { title: "Chapter 12" };
export default function RootLayout({ children }) {
  return <html lang="en"><body>{children}</body></html>;
}
```

```jsx
// file: rsc/app/page.jsx
import { getNotes, SECRET_API_KEY } from "../lib/db.js";
import { LikeButton } from "./LikeButton.jsx";

// A Server Component: it runs only on the server, may be async, and may read secrets.
export default async function Home() {
  const notes = await getNotes();
  console.log(`[server] rendering Home with key ending ${SECRET_API_KEY.slice(-4)}`);
  return <main>
    <h1>Notes</h1>
    <ul>{notes.map((n) => <li key={n.id}>{n.text} <LikeButton id={n.id} /></li>)}</ul>
  </main>;
}
```

```jsx
// file: rsc/app/LikeButton.jsx
"use client";                              // this module and its imports are sent to the browser
import { useState } from "react";

export function LikeButton({ id }) {
  const [likes, setLikes] = useState(0);
  return <button onClick={() => setLikes(likes + 1)} aria-label={`Like note ${id}`}>♥ {likes}</button>;
}
```

```console
$ cd rsc && npx next build 2>&1 | grep -E "server\]|Route|[○ƒ◐] "
[server] rendering Home with key ending 1234
Route (app)
┌ ○ /
├ ○ /_not-found
└ ○ /actions
○  (Static)  prerendered as static content
```

The page has no per-request data, so Next.js rendered it **at build time** — the server log line appeared during the build — and marked it `○` (static). What reached the browser?

```console
$ cd rsc && echo "files in .next/static containing the secret: $(grep -rl sk-live .next/static | wc -l)" && echo "containing LikeButton: $(grep -rl 'Like note' .next/static | wc -l)"
files in .next/static containing the secret: 0
containing LikeButton: 1
$ cd rsc && { ../../node_modules/.bin/next start -p 3100 > /dev/null & S=$!; sleep 1; curl -s localhost:3100/ | grep -o "<main>.*</main>"; kill $S; }
<main><h1>Notes</h1><ul><li>Buy milk<!-- --> <button aria-label="Like note 1">♥ <!-- -->0</button></li><li>Read chapter 12<!-- --> <button aria-label="Like note 2">♥ <!-- -->0</button></li></ul></main>
$ cd rsc && grep -E "LikeButton|Buy milk" .next/server/app/index.segments/__PAGE__.segment.rsc | cut -c1-230
9:I[78287,["$2","$8"],"LikeButton"]
7:[["children",{"s":"__PAGE__","h":160,"d":{"r":["$","$1","c",{"children":[["$","main",null,{"children":[["$","h1",null,{"children":"Notes"}],["$","ul",null,{"children":[["$","li","1",{"children":["Buy milk"," ",["$","$L9",null,{"
```

Three findings. The secret is nowhere in the browser's files; the `LikeButton` code is in one browser chunk. The HTML contains the *rendered* buttons (`♥ 0`) — the Client Component was rendered on the server too, and then hydrated. And the third output is the **RSC payload**, the format in which the server sends a rendered Server Component tree to the browser: elements as JSON-like arrays (`["$","li","1",{…}]`), text already computed, and each Client Component replaced by a *reference* (`$L9`) to a module in a browser chunk (`I[…,"LikeButton"]`) plus its serialised props (`{"id":1}`). The browser uses the payload to build and later update the tree without re-running server code.

### Deep & Tricky

**DT-24. Props passed from a Server Component to a Client Component cross a network boundary, so they must be serialisable by React: strings, numbers, plain objects and arrays, `Date`, `Map`, `Set`, promises, JSX, and Server Functions — but not ordinary functions, class instances, or symbols. Next.js reports an event handler passed across the boundary while *prerendering* the page, not while compiling it.** *(React documentation, "'use client' — Serializable types"; Next.js documentation, "Server and Client Components")*

The `mistakes` application makes the three classic boundary errors:

```jsx
// file: mistakes/app/Clicker.jsx
"use client";
export function Clicker({ onClick }) { return <button onClick={onClick}>Click</button>; }
```

```jsx
// file: mistakes/app/fn/page.jsx
import { Clicker } from "../Clicker.jsx";
// Mistake 1: a Server Component passes a function to a Client Component.
export default function Page() {
  return <Clicker onClick={() => console.log("clicked")} />;
}
```

```jsx
// file: mistakes/app/hook/page.jsx
import { useState } from "react";
// Mistake 2: state in a Server Component (no "use client").
export default function Page() {
  const [n, setN] = useState(0);
  return <button onClick={() => setN(n + 1)}>{n}</button>;
}
```

```jsx
// file: mistakes/app/secret/Panel.jsx
"use client";
import { SECRET_API_KEY } from "../../../rsc/lib/db.js";
// Mistake 3: a Client Component imports a server-only module.
export function Panel() { return <p>{SECRET_API_KEY.length}</p>; }
```

```jsx
// file: mistakes/app/secret/page.jsx
import { Panel } from "./Panel.jsx";
export default function Page() { return <Panel />; }
```

```console
$ cp rsc/next.config.mjs rsc/app/layout.jsx mistakes/ && mv mistakes/layout.jsx mistakes/app/
$ cd mistakes && npx next build 2>&1 | grep -E "^Error"
Error: Turbopack build failed with 3 errors:
Error: You're importing a module that depends on "server-only". This API is only available in Server Components in the App Router, but you are using it in the Pages Router.
Error: You're importing a module that depends on `useState` into a React Server Component module. This API is only available in Client Components. To fix, mark the file (or its parent) with the `"use client"` directive.
Error: 'server-only' cannot be imported from a Client Component module
$ cd mistakes && rm -r app/hook app/secret && npx next build 2>&1 | grep -A3 "^Error: Event"
Error: Event handlers cannot be passed to Client Component props.
  {onClick: function onClick}
            ^^^^^^^^^^^^^^^^
If you need interactivity, consider converting part of this to a Client Component.
```

The compiler catches two: `useState` in a module without `"use client"`, and a Client Component importing the `server-only` database module (reported twice, once for each environment the client module is compiled for). Only after those are removed does the build reach prerendering and report the function prop.

### Pattern

**P-13. Push `"use client"` down to the leaves: keep pages, layouts, and data fetching in Server Components, and make only the interactive pieces (a button, a form, a chart) Client Components. Pass server-rendered content *into* Client Components as `children` rather than importing it there.** *(Next.js documentation, "Server and Client Components — Reducing JS bundle size" and "Interleaving Server and Client Components")* *Trade-off:* many small boundaries mean more props to serialise and more modules to load; a highly interactive area (an editor, a dashboard) is simpler as one Client Component subtree.

### Mastery check

1. **Predict.** `LikeButton` imports a 200 KB charting library. Which grows: the HTML, the RSC payload, or the JavaScript in `.next/static`?
2. **Explain.** Why is a component in a `"use client"` module still rendered on the server?

## Unit 12.2 — Server Functions and forms

**One-line summary:** A function marked `"use server"` (inside a server file, or every export of a file that starts with `"use server"`) can be passed to the browser as a reference and called from there — most often as a form's `action`; Next.js turns the call into a POST request, runs the function on the server, and can refresh the page's data with `revalidatePath` or `updateTag`; and because it is reachable over HTTP, it must validate its input and check authorisation itself.

```jsx
// file: rsc/app/actions/page.jsx
import { revalidatePath } from "next/cache";
import { addNote, getNotes } from "../../lib/db.js";

// A Server Function: called from the browser by a form, runs on the server.
async function create(formData) {
  "use server";
  const text = String(formData.get("text") ?? "").trim();
  if (!text) return;
  await addNote(text);
  console.log(`[server] added "${text}"`);
  revalidatePath("/actions");             // the prerendered page is now stale: render it again
}

export default async function ActionsPage() {
  const notes = await getNotes();
  return <main>
    <h1>Notes ({notes.length})</h1>
    <ul>{notes.map((n) => <li key={n.id}>{n.text}</li>)}</ul>
    <form action={create}>
      <input name="text" aria-label="New note" required />
      <button>Add</button>
    </form>
  </main>;
}
```

```console
$ cd rsc && npx next build 2>&1 | grep -E "/actions"
└ ○ /actions
$ cd rsc && python3 ../../tools/page.py http://localhost:3100/actions "eval:document.querySelector('main').innerText" "eval:[...document.querySelectorAll('form input')].map((i) => i.outerHTML)" "fill:input[name=text]|Write tests" "click:button" "wait:500" "eval:document.querySelector('main').innerText" --serve "../../node_modules/.bin/next start -p 3100" --server-log 2>&1 | grep -vE "Next.js|Local|Network|Ready|took"
=> "Notes (2)\nBuy milk\nRead chapter 12\nAdd"
=> ["<input type=\"hidden\" name=\"$ACTION_ID_40acd0d101451f4f0c50e1124793e870299bf60693\">", "<input aria-label=\"New note\" required=\"\" name=\"text\">"]
[server] added "Write tests"
=> "Notes (3)\nBuy milk\nRead chapter 12\nWrite tests\nAdd"
```

The form works — and contains a hidden input naming the function by an ID. That ID is all a client needs:

```console
$ cd rsc && { ../../node_modules/.bin/next start -p 3100 > /dev/null & S=$!; sleep 1; ID=$(curl -s localhost:3100/actions | grep -o "ACTION_ID_[0-9a-f]*" | head -1); curl -s -o /dev/null -w "POST %{http_code}\n" -F "\$$ID=" -F "text=Added by curl, no browser" localhost:3100/actions; curl -s localhost:3100/actions | grep -o "<ul>.*</ul>"; kill $S; }
⚠ Missing `origin` header from a forwarded Server Actions request.
POST 200
<ul><li>Buy milk</li><li>Read chapter 12</li><li>Added by curl, no browser</li></ul>
$ cd rsc && { ../../node_modules/.bin/next start -p 3100 > /dev/null 2>&1 & S=$!; sleep 1; ID=$(curl -s localhost:3100/actions | grep -o "ACTION_ID_[0-9a-f]*" | head -1); curl -s -o /dev/null -w "POST from another site: %{http_code}\n" -H "Origin: https://evil.example" -F "\$$ID=" -F "text=evil" localhost:3100/actions; kill $S; }
POST from another site: 500
```

### Deep & Tricky

**DT-25. Every Server Function is a public HTTP endpoint. Its ID is in the page (here in a hidden input, which lets the form work before JavaScript loads), and anyone who can load the page can call it with any arguments — `curl` added a note above without a browser or a form. Validate every argument (it is `unknown` input, whatever the TypeScript types say) and check the user's permissions inside the function, not only in the page that renders the form. The warning in the output is Next.js's cross-site request forgery check: it compares the `Origin` header with the host and rejects requests that come from other sites, as the second request shows (status 500, "Invalid Server Actions request" in the server log), but a request with no `Origin` at all, like the first, is let through — it protects users' browsers; it is not authentication.** *(React documentation, "'use server' — Security considerations"; Next.js documentation, "Data Security — Server Actions")*

### Mastery check

1. **Predict.** In the `rsc` app, `create` is changed to skip `revalidatePath`. After submitting the form, does the browser show the new note? After a reload?

## Topic 12B — Rendering and caching

## Unit 12.3 — Streaming and partial prerendering

**One-line summary:** With `cacheComponents` enabled (Next.js 16), a route is prerendered at build time as far as possible; any component that reads per-request data (`cookies()`, `headers()`, `searchParams`, `connection()`, uncached `fetch`) must be inside a `<Suspense>` boundary, whose fallback goes into the static **shell**; at request time the shell is sent at once and each boundary's content is **streamed** into the same response when it is ready.

```js
// file: stream/next.config.mjs
export default { cacheComponents: true, partialPrefetching: true };   // Next.js 16: "use cache", partial prerendering
```

```jsx
// file: stream/app/layout.jsx
export default function RootLayout({ children }) {
  return <html lang="en"><body>{children}</body></html>;
}
```

```js
// file: stream/app/slow.js
export const wait = (ms) => new Promise((r) => setTimeout(r, ms));
export async function getSales() { await wait(800); return { total: 12345 }; }
export async function getReviews() { await wait(1600); return ["Great", "Fast delivery"]; }
```

```jsx
// file: stream/app/report/page.jsx
import { Suspense } from "react";
import { connection } from "next/server";
import { getSales, getReviews } from "../slow.js";

async function Sales() {
  await connection();                       // per-request data: never prerendered
  const { total } = await getSales();
  return <p id="sales">Sales today: {total}</p>;
}
async function Reviews() {
  await connection();
  const reviews = await getReviews();
  return <ul id="reviews">{reviews.map((r) => <li key={r}>{r}</li>)}</ul>;
}

export default function Report() {
  return <main>
    <h1>Daily report</h1>                    {/* static shell: prerendered at build time */}
    <Suspense fallback={<p>Loading sales…</p>}><Sales /></Suspense>
    <Suspense fallback={<p>Loading reviews…</p>}><Reviews /></Suspense>
  </main>;
}
```

```js
// file: stream/timeline.mjs
// Fetch a page and print when each interesting piece of HTML arrives.
const [url, ...marks] = process.argv.slice(2);
const start = performance.now();
const res = await fetch(url);
const decoder = new TextDecoder();
let html = "", seen = new Set();
for await (const chunk of res.body) {
  html += decoder.decode(chunk, { stream: true });
  for (const m of marks) if (!seen.has(m) && html.includes(m)) {
    seen.add(m);
    console.log(`${String(Math.round((performance.now() - start) / 100) * 100).padStart(5)} ms  ${m}`);
  }
}
console.log(`${String(Math.round((performance.now() - start) / 100) * 100).padStart(5)} ms  end of response (${html.length} characters)`);
```

```console
$ cd stream && npx next build 2>&1 | grep -E "Route|[○ƒ◐] "
Route (app)      Revalidate  Expire
┌ ○ /_not-found
├ ○ /product             5s      1h
└ ◐ /report
○  (Static)             prerendered as static content
◐  (Partial Prerender)  prerendered as static HTML with dynamic server-streamed content
$ cd stream && { ../../node_modules/.bin/next start -p 3100 > /dev/null & S=$!; sleep 1; node timeline.mjs http://localhost:3100/report "Daily report" "Loading sales" "Sales today" "Fast delivery"; kill $S; }
  100 ms  Daily report
  100 ms  Loading sales
  900 ms  Sales today
 1700 ms  Fast delivery
 1700 ms  end of response (11079 characters)
```

`/report` is marked `◐` (partial prerender). One response: the shell with both fallbacks arrives at once, the sales at about 0.9 s, the reviews at about 1.7 s, and only then does the response end. Without the boundaries, the first byte would wait for the slowest query; without streaming, a client would make three requests.

### Mastery check

1. **Predict.** Move `<Reviews />` *inside* the first `<Suspense>` boundary. When do the sales appear?

## Unit 12.4 — Caching with `"use cache"`

**One-line summary:** `"use cache"` at the top of an async function (or component, or file) caches its result, keyed by its arguments; `cacheLife` says how long the entry is fresh (`stale`), when it is regenerated in the background (`revalidate`), and when it must be thrown away (`expire`); `cacheTag` labels it so that `updateTag` (in a Server Function) or `revalidateTag` can invalidate it on demand; and a cached result can be part of the prerendered page.

```jsx
// file: stream/app/product/page.jsx
import { cacheLife } from "next/cache";
import { wait } from "../slow.js";

async function getPrice() {
  "use cache";                              // the result is cached and reused across requests
  cacheLife({ stale: 30, revalidate: 5, expire: 3600 });   // seconds
  await wait(300);
  return { price: 42, computedAt: new Date().toISOString() };
}

export default async function Product() {
  const { price, computedAt } = await getPrice();
  return <main><h1>Widget</h1><p>${price} (computed {computedAt})</p></main>;
}
```

```console
$ cd stream && npx next build 2>&1 | grep -E "Revalidate|/product"
Route (app)      Revalidate  Expire
├ ○ /product             5s      1h
$ cd stream && { ../../node_modules/.bin/next start -p 3100 > /dev/null & S=$!; sleep 1; p() { curl -s localhost:3100/product | grep -o "computed <!-- -->[^<]*" | head -1; }; p; sleep 6; p; sleep 0.5; p; kill $S; }
computed <!-- -->2026-11-16T12:47:17.092Z
computed <!-- -->2026-11-16T12:47:17.092Z
computed <!-- -->2026-11-16T12:47:24.546Z
```

The route table shows the page as static, regenerated every 5 s. The first request after 5 s still gets the old page (stale-while-revalidate) and triggers a regeneration in the background; the next request gets the new time.

### Deep & Tricky

**DT-26. A `"use cache"` entry whose lifetime is too short is not put into the prerendered page: in Next.js 16.4, an `expire` under 5 minutes or a `stale` under 30 seconds makes the cached call behave like per-request data, and the build fails with "encountered uncached or runtime data during prerendering" unless the component is inside `<Suspense>`. Measured here: `{ stale: 29, … }` fails; `{ stale: 30, revalidate: 5, expire: 3600 }` prerenders; `{ …, expire: 299 }` fails.** *(Next.js 16.4.6 source, `server/use-cache/constants.js` — `MIN_PREFETCHABLE_STALE = 30`, `MIN_PRERENDERABLE_EXPIRE = 300`; Next.js documentation, "cacheLife")*

### Pattern

**P-14. Decide per piece of data, not per page: static content in the shell; shared data that changes rarely behind `"use cache"` with a tag that writes invalidate; per-user or real-time data streamed behind `<Suspense>`.** *(Next.js documentation, "Cache Components")* *Trade-off:* every cache is a source of stale-data bugs; tag the cache entries that writes affect, and test the write-then-read path (as the project does).

## Chapter 12 Project — A guestbook

### Requirements

A one-page guestbook. The list of messages is read through a cached function (tag `messages`) and prerendered. A Client Component form posts through a Server Function that validates the name (1–40 characters) and message (1–280), returns errors and the submitted values on failure (so nothing typed is lost), and on success stores the message and invalidates the tag, so that the poster — and every later visitor — sees it. The submit button shows a pending state.

### Reference solution

```js
// file: project/next.config.mjs
export default { cacheComponents: true, partialPrefetching: true };   // Next.js 16: "use cache", partial prerendering
```

```js
// file: project/lib/store.js
import "server-only";
import { cacheLife, cacheTag } from "next/cache";

const messages = [{ id: 1, name: "Ana", text: "First!", at: "2026-01-01T10:00:00Z" }];

export async function getMessages() {
  "use cache";
  cacheTag("messages");                     // a label to invalidate this cache entry by
  cacheLife("max");                         // keep until invalidated
  console.log(`[server] reading ${messages.length} message(s) from the store`);
  await new Promise((r) => setTimeout(r, 200));
  return messages.toReversed();
}

export async function addMessage({ name, text }) {
  messages.push({ id: messages.length + 1, name, text, at: new Date().toISOString() });
}
```

```js
// file: project/app/actions.js
"use server";                               // every export is a Server Function (a public endpoint)
import { updateTag } from "next/cache";
import { addMessage } from "../lib/store.js";

export async function postMessage(previous, formData) {
  const values = { name: String(formData.get("name") ?? "").trim(), text: String(formData.get("text") ?? "").trim() };
  const errors = {};
  if (values.name.length < 1 || values.name.length > 40) errors.name = "Name must be 1–40 characters";
  if (values.text.length < 1 || values.text.length > 280) errors.text = "Message must be 1–280 characters";
  if (Object.keys(errors).length) return { errors, values };

  await addMessage(values);
  updateTag("messages");                    // expire the cached list; this request sees the new one
  return { errors: {}, values: { name: values.name, text: "" }, posted: Date.now() };
}
```

```jsx
// file: project/app/MessageForm.jsx
"use client";
import { useActionState } from "react";
import { useFormStatus } from "react-dom";
import { postMessage } from "./actions.js";

function Submit() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? "Posting…" : "Post"}</button>;
}

export function MessageForm() {
  const [state, action] = useActionState(postMessage, { errors: {}, values: { name: "", text: "" } });
  return (
    <form action={action} key={state.posted}>     {/* a new key after success clears the fields */}
      <label>Name <input name="name" defaultValue={state.values.name} aria-invalid={!!state.errors.name} /></label>
      {state.errors.name && <p role="alert">{state.errors.name}</p>}
      <label>Message <textarea name="text" defaultValue={state.values.text} aria-invalid={!!state.errors.text} /></label>
      {state.errors.text && <p role="alert">{state.errors.text}</p>}
      <Submit />
    </form>
  );
}
```

```jsx
// file: project/app/Messages.jsx
import { getMessages } from "../lib/store.js";

export async function Messages() {
  const messages = await getMessages();
  return <ol id="messages">{messages.map((m) => <li key={m.id}><b>{m.name}</b>: {m.text}</li>)}</ol>;
}
```

```jsx
// file: project/app/layout.jsx
export const metadata = { title: "Guestbook" };
export default function RootLayout({ children }) {
  return <html lang="en"><body>{children}</body></html>;
}
```

```jsx
// file: project/app/page.jsx
import { Messages } from "./Messages.jsx";
import { MessageForm } from "./MessageForm.jsx";

export default function Home() {
  return <main>
    <h1>Guestbook</h1>
    <MessageForm />
    <Messages />
  </main>;
}
```

```console
$ cd project && npx next build 2>&1 | grep -E "server\]|Route|[○ƒ◐] "
[server] reading 1 message(s) from the store
Route (app)      Revalidate  Expire
┌ ○ /                   30d      1y
└ ○ /_not-found
○  (Static)  prerendered as static content
$ cd project && python3 ../../tools/page.py http://localhost:3100/ "eval:document.getElementById('messages').innerText" "click:button" "wait:400" "eval:[...document.querySelectorAll('[role=alert]')].map((e) => e.textContent)" "fill:input[name=name]|Bo" "fill:textarea|Hello from the guestbook" "click:button" "wait:600" "eval:document.getElementById('messages').innerText" "eval:[document.querySelector('input[name=name]').value, document.querySelector('textarea').value]" "reload" "eval:document.getElementById('messages').innerText" --serve "../../node_modules/.bin/next start -p 3100" --server-log 2>&1 | grep -vE "Next.js|Local|Network|Ready|took"
=> "Ana: First!"
=> ["Name must be 1–40 characters", "Message must be 1–280 characters"]
[server] reading 2 message(s) from the store
=> "Bo: Hello from the guestbook\nAna: First!"
=> ["Bo", ""]
=> "Bo: Hello from the guestbook\nAna: First!"
```

The list was read once at build time; the empty submission returns both errors without touching the store; the valid one writes, `updateTag` expires the entry, and the store is read again (2 messages) — in the *same* request, so the poster sees their message immediately; the form is reset by its new `key` but keeps the name; and a reload is served from the refreshed cache.

### Review checklist

- `postMessage` accepts any number of posts from anyone. Add a per-IP rate limit, and say where the counter must live when the app runs on several servers.
- The store is in memory: what happens to messages when the server restarts, or when there are two instances? (Chapter 13 adds a database.)
- Make the page show "Posted!" after a successful post without breaking the form reset.

## Answers to the Chapter 12 prediction questions

**Unit 12.1, Q1.** The JavaScript in `.next/static` (the library is bundled for the browser). The HTML and the RSC payload grow only by what the chart renders.

**Unit 12.1, Q2.** To produce HTML for the first paint (server-side rendering); `"use client"` marks where the browser's module graph starts, not "browser only". Use effects for browser-only work.

**Unit 12.2, Q1.** No, and no. The function runs and stores the note (the server logs it), but nothing tells Next.js that the prerendered page is stale: the browser keeps showing two notes, and a reload is served the static page from the build. Checked by removing the line and repeating the browser run.

**Unit 12.3, Q1.** At about 1.7 s: a boundary reveals its content when *everything* inside it is ready.

# Chapter 13 — Databases, authentication, and deployment

*Goal: turn a Next.js application into one that stores real data, knows who its users are, lets each user see only their own data, and runs as a self-contained server. You will query SQLite safely (and watch an unsafe query leak another user's data), hash passwords, create sessions in `HttpOnly` cookies, separate authentication from authorisation, add Route Handlers for API clients, and build a standalone deployment — discovering on the way which environment variables are read at run time and which were frozen at build time.*

**What you will be able to do after this chapter**

- Query a database from Server Components and Server Functions with parameters, never string building.
- Store passwords as salted, slow hashes, and sessions as random tokens in secure cookies.
- Check authentication *and* authorisation where the data is read and written, not only in a proxy.
- Write Route Handlers (`GET`, `POST`, `DELETE`) with correct status codes.
- Build and run a standalone Next.js server, and configure it with environment variables correctly.

The examples use `node:sqlite`, the SQLite module built into Node.js 22 (still marked experimental, so its warning is turned off with `--disable-warning=ExperimentalWarning`); the same code shapes apply to PostgreSQL clients and to ORMs such as Drizzle or Prisma.

## Topic 13A — Data and identity

## Unit 13.1 — Querying a database safely

**One-line summary:** A server-side module opens the database and Server Components and Server Functions query it directly; every value that comes from outside goes in as a **parameter** (`?`), so the database receives the SQL and the data separately and can never mistake data for SQL.

```js
// file: basics/sql.mjs
import { DatabaseSync } from "node:sqlite";

const db = new DatabaseSync(":memory:");
db.exec(`
  CREATE TABLE notes (id INTEGER PRIMARY KEY, owner TEXT NOT NULL, text TEXT NOT NULL);
  INSERT INTO notes (owner, text) VALUES ('ana', 'Ana''s diary'), ('bo', 'Bo''s bank PIN is 4321');
`);

// The user types a search term; we only want Ana's notes.
const search = "%' OR 1=1 --";

const unsafe = db.prepare(`SELECT text FROM notes WHERE owner = 'ana' AND text LIKE '%${search}%'`).all();
console.log("string-built SQL:", unsafe.map((r) => r.text));

const safe = db.prepare("SELECT text FROM notes WHERE owner = ? AND text LIKE ?").all("ana", `%${search}%`);
console.log("parameters:      ", safe.map((r) => r.text));
console.log("what the unsafe query became:");
console.log(`  SELECT text FROM notes WHERE owner = 'ana' AND text LIKE '%${search}%'`);
```

```console
$ node --disable-warning=ExperimentalWarning basics/sql.mjs
string-built SQL: [ "Ana's diary", "Bo's bank PIN is 4321" ]
parameters:       []
what the unsafe query became:
  SELECT text FROM notes WHERE owner = 'ana' AND text LIKE '%%' OR 1=1 --%'
```

The search term closed the string, added `OR 1=1` (true for every row), and commented out the rest with `--`. Because `AND` binds tighter than `OR`, the condition became "(Ana's notes matching nothing) or anything at all", and Bo's PIN was returned to Ana. With parameters, the same text is simply a strange search term that matches nothing.

### Mastery check

1. **Explain.** Why does escaping quotes in the search term not fully solve the problem, when parameters do?

## Unit 13.2 — Passwords and sessions

**One-line summary:** Store a password only as a salted hash from a deliberately slow function (scrypt, Argon2id, or bcrypt), and compare hashes in constant time; after sign-in, give the browser a long random session token in a cookie marked `HttpOnly`, `Secure`, and `SameSite=Lax`, and keep only a hash of the token on the server.

```js
// file: basics/password.mjs
import { scryptSync, randomBytes, timingSafeEqual, createHash } from "node:crypto";

// Store a random salt and a slow, salted hash — never the password, never a fast hash.
export function hashPassword(password) {
  const salt = randomBytes(16);
  const hash = scryptSync(password.normalize("NFKC"), salt, 64);
  return `${salt.toString("base64url")}:${hash.toString("base64url")}`;
}
export function verifyPassword(password, stored) {
  const [salt, hash] = stored.split(":").map((s) => Buffer.from(s, "base64url"));
  const candidate = scryptSync(password.normalize("NFKC"), salt, 64);
  return timingSafeEqual(candidate, hash);      // compares every byte, whatever the first mismatch
}

if (import.meta.main) {
  const a = hashPassword("correct horse"), b = hashPassword("correct horse");
  console.log("same password, different hashes:", a !== b);
  console.log("length of stored value:", a.length);
  console.log("verify right password:", verifyPassword("correct horse", a));
  console.log("verify wrong password:", verifyPassword("correct hors3", a));

  let t = performance.now();
  for (let i = 0; i < 20; i++) scryptSync("pw", "salt", 64);
  const scryptMs = (performance.now() - t) / 20;
  t = performance.now();
  for (let i = 0; i < 20000; i++) createHash("sha256").update("pw").digest();
  const shaMs = (performance.now() - t) / 20000;
  const power = Math.round(Math.log10(scryptMs / shaMs));
  console.log(`scrypt is roughly ${(10 ** power).toLocaleString("en")} times slower than SHA-256 per guess`);
}
```

```console
$ node basics/password.mjs
same password, different hashes: true
length of stored value: 109
verify right password: true
verify wrong password: false
scrypt is roughly 10,000 times slower than SHA-256 per guess
```

The random salt makes two hashes of the same password different, so a stolen table cannot be attacked with precomputed tables or reveal which users share a password. The cost is the point: each guess at a stolen hash takes roughly ten thousand times longer than with SHA-256 (the ratio is measured and rounded; it depends on the machine).

The application uses these functions, a database module, and a session module:

```js
// file: app/lib/password.js
import { scryptSync, randomBytes, timingSafeEqual } from "node:crypto";

// Store a random salt and a slow, salted hash — never the password, never a fast hash.
export function hashPassword(password) {
  const salt = randomBytes(16);
  const hash = scryptSync(password.normalize("NFKC"), salt, 64);
  return `${salt.toString("base64url")}:${hash.toString("base64url")}`;
}
export function verifyPassword(password, stored) {
  const [salt, hash] = stored.split(":").map((s) => Buffer.from(s, "base64url"));
  const candidate = scryptSync(password.normalize("NFKC"), salt, 64);
  return timingSafeEqual(candidate, hash);      // compares every byte, whatever the first mismatch
}

```

```js
// file: app/lib/db.js
import "server-only";
import { DatabaseSync } from "node:sqlite";

export const db = new DatabaseSync(process.env.DATABASE_PATH ?? "notes.db");
db.exec(`
  PRAGMA foreign_keys = ON;
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY, email TEXT NOT NULL UNIQUE, password_hash TEXT NOT NULL);
  CREATE TABLE IF NOT EXISTS sessions (
    token_hash TEXT PRIMARY KEY, user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    expires_at INTEGER NOT NULL);
  CREATE TABLE IF NOT EXISTS notes (
    id INTEGER PRIMARY KEY, user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    text TEXT NOT NULL CHECK (length(text) BETWEEN 1 AND 500));
`);
```

```js
// file: app/lib/session.js
import "server-only";
import { cache } from "react";
import { cookies } from "next/headers";
import { randomBytes, createHash } from "node:crypto";
import { db } from "./db.js";

const COOKIE = "session";
const WEEK = 7 * 24 * 60 * 60;
const sha256 = (s) => createHash("sha256").update(s).digest("base64url");

export async function createSession(userId) {
  const token = randomBytes(32).toString("base64url");          // 256 random bits: unguessable
  db.prepare("INSERT INTO sessions (token_hash, user_id, expires_at) VALUES (?, ?, ?)")
    .run(sha256(token), userId, Date.now() + WEEK * 1000);        // store only a hash of the token
  (await cookies()).set(COOKIE, token, {
    httpOnly: true,                       // not readable by JavaScript in the page
    sameSite: "lax",                      // not sent on cross-site POSTs
    secure: process.env.NODE_ENV === "production",   // HTTPS only (browsers treat localhost as secure)
    path: "/", maxAge: WEEK,
  });
}

// Who is making this request? Cached for the duration of one request.
export const getCurrentUser = cache(async () => {
  const token = (await cookies()).get(COOKIE)?.value;
  if (!token) return null;
  const row = db.prepare(`SELECT users.id, users.email, sessions.expires_at FROM sessions
    JOIN users ON users.id = sessions.user_id WHERE token_hash = ?`).get(sha256(token));
  if (!row || row.expires_at < Date.now()) return null;
  return { id: row.id, email: row.email };
});

export async function destroySession() {
  const store = await cookies();
  const token = store.get(COOKIE)?.value;
  if (token) db.prepare("DELETE FROM sessions WHERE token_hash = ?").run(sha256(token));
  store.delete(COOKIE);
}
```

```js
// file: app/lib/auth.js
import "server-only";
import { db } from "./db.js";
import { hashPassword, verifyPassword } from "./password.js";

const DUMMY_HASH = hashPassword("dummy password for timing");

export function parseCredentials(email, password) {
  email = String(email ?? "").trim().toLowerCase();
  password = String(password ?? "");
  if (!/^\S+@\S+\.\S+$/.test(email)) return { error: "Enter a valid email address", email };
  if (password.length < 8) return { error: "Passwords have at least 8 characters", email };
  return { email, password };
}

// Returns the user, or null. Unknown emails are checked against a dummy hash,
// so that both failures take the same time and give the same answer.
export function authenticate(email, password) {
  const user = db.prepare("SELECT id, email, password_hash FROM users WHERE email = ?").get(email);
  const ok = verifyPassword(password, user?.password_hash ?? DUMMY_HASH);
  return ok && user ? { id: user.id, email: user.email } : null;
}

export function register(email, password) {
  try {
    const { lastInsertRowid } = db.prepare("INSERT INTO users (email, password_hash) VALUES (?, ?)")
      .run(email, hashPassword(password));
    return { id: Number(lastInsertRowid), email };
  } catch (e) {
    if (String(e.message).includes("UNIQUE")) return null;
    throw e;
  }
}
```

### Deep & Tricky

**DT-27. Store a *hash* of each session token, not the token: the token in the cookie is effectively a password, and anyone who can read the `sessions` table (a backup, a log, an SQL injection elsewhere) could otherwise sign in as every user. Because the token is 256 random bits, a fast hash (SHA-256) is enough here — slow hashing protects low-entropy secrets such as passwords, not random tokens.** *(OWASP Session Management Cheat Sheet; Lucia, "Sessions" guide; NIST SP 800-63B §5.1.1.2 on memorised secrets)*

**DT-28. Sign-in must fail the same way, and in about the same time, for an unknown email and a wrong password — "Wrong email or password", with a hash comparison in both cases (against a dummy hash for unknown emails). A different message or a quicker failure tells an attacker which emails have accounts.** *(OWASP Authentication Cheat Sheet, "Authentication and Error Messages")*

### Mastery check

1. **Predict.** The cookie lacked `HttpOnly`. Which attack becomes much worse, and why does it not matter to a `curl` user?

## Unit 13.3 — Authentication, authorisation, and Route Handlers

**One-line summary:** *Authentication* establishes who is making a request (a valid session); *authorisation* decides whether that user may do this to this resource (it is their note). Both checks belong in the code that reads or writes the data — every Server Function, every Route Handler — because a proxy (Next.js 16's `proxy.js`, formerly middleware) sees only the request and can do no more than an optimistic redirect.

```js
// file: app/proxy.js
import { NextResponse } from "next/server";

// Runs before every matching request. An optimistic check only: it sees that a cookie exists,
// not that the session is valid. The real check is getCurrentUser() in the page and actions.
export function proxy(request) {
  if (!request.cookies.has("session")) return NextResponse.redirect(new URL("/login", request.url));
}
export const config = { matcher: ["/"] };
```

```js
// file: app/app/actions.js
"use server";
import { redirect } from "next/navigation";
import { revalidatePath } from "next/cache";
import { db } from "../lib/db.js";
import { authenticate, parseCredentials, register } from "../lib/auth.js";
import { createSession, destroySession, getCurrentUser } from "../lib/session.js";

export async function signUp(previous, formData) {
  const c = parseCredentials(formData.get("email"), formData.get("password"));
  if (c.error) return c;
  const user = register(c.email, c.password);
  if (!user) return { error: "That email is already registered", email: c.email };
  await createSession(user.id);
  redirect("/");
}

export async function signIn(previous, formData) {
  const c = parseCredentials(formData.get("email"), formData.get("password"));
  if (c.error) return c;
  const user = authenticate(c.email, c.password);
  if (!user) return { error: "Wrong email or password", email: c.email };   // never say which
  await createSession(user.id);
  redirect("/");
}

export async function signOut() {
  await destroySession();
  redirect("/login");
}

async function requireUser() {
  const user = await getCurrentUser();
  if (!user) throw new Error("Not signed in");
  return user;
}

export async function addNote(formData) {
  const user = await requireUser();
  const text = String(formData.get("text") ?? "").trim();
  if (text.length < 1 || text.length > 500) return;
  db.prepare("INSERT INTO notes (user_id, text) VALUES (?, ?)").run(user.id, text);
  revalidatePath("/");
}

export async function deleteNote(id) {
  const user = await requireUser();
  // The owner check is part of the query: another user's note id deletes nothing.
  db.prepare("DELETE FROM notes WHERE id = ? AND user_id = ?").run(Number(id), user.id);
  revalidatePath("/");
}
```

```jsx
// file: app/app/layout.jsx
export const metadata = { title: "Private notes" };
export default function RootLayout({ children }) {
  return <html lang="en"><body>{children}</body></html>;
}
```

```jsx
// file: app/app/page.jsx
import { redirect } from "next/navigation";
import { db } from "../lib/db.js";
import { getCurrentUser } from "../lib/session.js";
import { addNote, deleteNote, signOut } from "./actions.js";

export default async function Home() {
  const user = await getCurrentUser();
  if (!user) redirect("/login");
  const notes = db.prepare("SELECT id, text FROM notes WHERE user_id = ? ORDER BY id").all(user.id);
  return <main>
    <p>Signed in as <b id="who">{user.email}</b> <button form="out">Sign out</button></p>
    <form id="out" action={signOut} />
    <h1>Your notes</h1>
    <ul id="notes">{notes.map((n) => <li key={n.id}>{n.text}{" "}
      <form action={deleteNote.bind(null, n.id)} style={{ display: "inline" }}>
        <button aria-label={`Delete note ${n.id}`}>×</button>
      </form></li>)}</ul>
    <form action={addNote}><input name="text" aria-label="New note" /><button>Add</button></form>
  </main>;
}
```

```jsx
// file: app/app/login/AuthForm.jsx
"use client";
import { useActionState } from "react";

export function AuthForm({ action, label }) {
  const [state, formAction, pending] = useActionState(action, { email: "" });
  return (
    <form action={formAction} aria-label={label}>
      <label>Email <input name="email" type="email" defaultValue={state.email} /></label>
      <label>Password <input name="password" type="password" /></label>
      {state.error && <p role="alert">{state.error}</p>}
      <button disabled={pending}>{label}</button>
    </form>
  );
}
```

```jsx
// file: app/app/login/AppName.jsx
"use client";
export function AppName() {
  return <small id="app-name">{process.env.NEXT_PUBLIC_APP_NAME ?? "(unnamed)"}</small>;
}
```

```jsx
// file: app/app/login/page.jsx
import { AuthForm } from "./AuthForm.jsx";
import { AppName } from "./AppName.jsx";
import { signIn, signUp } from "../actions.js";

export default function Login() {
  return <main>
    <h1>Private notes <AppName /></h1>
    <p id="motd">{process.env.MOTD}</p>
    <AuthForm action={signIn} label="Sign in" />
    <AuthForm action={signUp} label="Create account" />
  </main>;
}
```

Route Handlers are files named `route.js` that export functions named after HTTP methods. They serve API clients — mobile apps, scripts, tests — with the same session and the same checks:

```js
// file: app/app/api/session/route.js
// POST /api/session {email, password} → sets the session cookie (for API clients and tests).
import { authenticate, parseCredentials } from "../../../lib/auth.js";
import { createSession } from "../../../lib/session.js";

export async function POST(request) {
  const body = await request.json().catch(() => ({}));
  const c = parseCredentials(body.email, body.password);
  const user = !c.error && authenticate(c.email, c.password);
  if (!user) return Response.json({ error: "Wrong email or password" }, { status: 401 });
  await createSession(user.id);
  return Response.json({ email: user.email });
}
```

```js
// file: app/app/api/notes/route.js
import { db } from "../../../lib/db.js";
import { getCurrentUser } from "../../../lib/session.js";

export async function GET() {
  const user = await getCurrentUser();
  if (!user) return Response.json({ error: "Sign in first" }, { status: 401 });
  return Response.json(db.prepare("SELECT id, text FROM notes WHERE user_id = ? ORDER BY id").all(user.id));
}
```

```js
// file: app/app/api/notes/[id]/route.js
import { db } from "../../../../lib/db.js";
import { getCurrentUser } from "../../../../lib/session.js";

export async function DELETE(request, { params }) {
  const user = await getCurrentUser();                                    // authentication
  if (!user) return Response.json({ error: "Sign in first" }, { status: 401 });
  const { id } = await params;
  const { changes } = db.prepare("DELETE FROM notes WHERE id = ? AND user_id = ?")   // authorisation
    .run(Number(id), user.id);
  // 404, not 403: do not confirm that someone else's note exists.
  return changes ? new Response(null, { status: 204 }) : Response.json({ error: "No such note" }, { status: 404 });
}
```

```js
// file: app/app/api/health/route.js
// For load balancers and uptime checks: is the server up, and can it reach the database?
import { db } from "../../../lib/db.js";

export function GET() {
  const { n } = db.prepare("SELECT count(*) AS n FROM users").get();
  return Response.json({ ok: true, users: n, motd: process.env.MOTD ?? null });
}
```

### Pattern

**P-15. A data access layer: put every query behind functions that take the current user and enforce ownership (`WHERE id = ? AND user_id = ?`), and never query user data from a component directly. Return 404 rather than 403 for another user's resource, so that IDs cannot be probed.** *(Next.js documentation, "Data Security — Data Access Layer"; OWASP API Security Top 10, API1:2023 "Broken Object Level Authorization")* *Trade-off:* a strict layer is more code than inline queries; the cost is repaid the first time a new page reuses a function instead of re-implementing its checks.

## Topic 13B — Deployment

## Unit 13.4 — Standalone builds and environment variables

**One-line summary:** `output: "standalone"` makes `next build` copy into `.next/standalone` a `server.js` and only the files and packages the server needs (traced from its imports), so the folder can be deployed and run with `node server.js` alone; static assets in `.next/static` (and `public/`) must be copied next to it or served by a CDN; configuration comes from environment variables — but some are read when you *build*, not when you *run*.

```js
// file: app/next.config.mjs
export default {
  output: "standalone",                    // a self-contained server for deployment (Unit 13.4)
};
```

The build gets an in-memory database (the modules are imported while pages are prerendered), and two environment variables whose values say when they were read:

```console
$ cd app && DATABASE_PATH=:memory: MOTD="Message at build time" NEXT_PUBLIC_APP_NAME="v1 (build)" NODE_OPTIONS=--disable-warning=ExperimentalWarning npx next build 2>&1 | sed -n '/Route/,$p' | grep -v "^$"
Route (app)
┌ ƒ /
├ ○ /_not-found
├ ƒ /api/health
├ ƒ /api/notes
├ ƒ /api/notes/[id]
├ ƒ /api/session
└ ○ /login
ƒ Proxy (Middleware)
○  (Static)   prerendered as static content
ƒ  (Dynamic)  server-rendered on demand
$ cd app && cp -r .next/static .next/standalone/ch13/app/.next/ && du -sh .next/standalone | cut -f1 && ls .next/standalone .next/standalone/ch13/app
50M
.next/standalone:
ch13  node_modules

.next/standalone/ch13/app:
server.js
```

The standalone folder mirrors the path from the workspace root (this book's `code` folder holds `node_modules`), and weighs about 50 MB including the traced parts of `next`, `react`, and `react-dom` — no `npm install` is needed where it runs. The server is started with `node`:

```console
$ cd app && python3 ../../tools/page.py http://localhost:3100/ "eval:location.pathname" "fill:form[aria-label='Create account'] input[name=email]|ana@example.org" "fill:form[aria-label='Create account'] input[name=password]|correct horse" "click:form[aria-label='Create account'] button" "wait:800" "eval:location.pathname" "fill:input[name=text]|Ana's secret plan" "click:text=Add" "wait:500" "eval:document.getElementById('notes').innerText" "eval:document.cookie" "click:text=Sign out" "wait:500" "eval:location.pathname" "fill:form[aria-label='Create account'] input[name=email]|bo@example.org" "fill:form[aria-label='Create account'] input[name=password]|short" "click:form[aria-label='Create account'] button" "wait:500" "eval:document.querySelector('[role=alert]').textContent" "fill:form[aria-label='Create account'] input[name=password]|bo's password" "click:form[aria-label='Create account'] button" "wait:800" "eval:[location.pathname, document.getElementById('notes').innerText]" --serve "env NODE_OPTIONS=--disable-warning=ExperimentalWarning DATABASE_PATH=$PWD/data.db PORT=3100 node .next/standalone/ch13/app/server.js" --server-log 2>&1 | grep -vE "Next.js|Local|Network|Ready|took"
=> "/login"
=> "/"
=> "Ana's secret plan ×"
=> ""
=> "/login"
=> "Passwords have at least 8 characters"
=> ["/", ""]
$ cd app && { env NODE_OPTIONS=--disable-warning=ExperimentalWarning DATABASE_PATH=$PWD/data.db PORT=3100 node .next/standalone/ch13/app/server.js > /dev/null & S=$!; sleep 1; login() { curl -s -c $1.jar -o /dev/null -w "login $1 ($2): %{http_code}\n" -H "content-type: application/json" -d "{\"email\":\"$1@example.org\",\"password\":\"$2\"}" localhost:3100/api/session; }; login ana "wrong password"; login ana "correct horse"; login bo "bo's password"; curl -s -D - -o /dev/null -H "content-type: application/json" -d '{"email":"ana@example.org","password":"correct horse"}' localhost:3100/api/session | grep -i set-cookie | sed -E "s/session=[^;]+/session=…/"; echo "ana's notes: $(curl -s -b ana.jar localhost:3100/api/notes)"; echo "bo's notes:  $(curl -s -b bo.jar localhost:3100/api/notes)"; echo "no cookie:   $(curl -s localhost:3100/api/notes)"; curl -s -b bo.jar -X DELETE -w " %{http_code}  (bo deletes ana's note 1)\n" localhost:3100/api/notes/1; curl -s -b bo.jar -X DELETE -w " %{http_code}  (bo deletes note 999)\n" localhost:3100/api/notes/999; curl -s -b ana.jar -X DELETE -w "%{http_code}  (ana deletes note 1)\n" localhost:3100/api/notes/1; kill $S; }
login ana (wrong password): 401
login ana (correct horse): 200
login bo (bo's password): 200
set-cookie: session=…; Path=/; Expires=Mon, 23 Nov 2026 12:52:18 GMT; Max-Age=604800; Secure; HttpOnly; SameSite=lax
ana's notes: [{"id":1,"text":"Ana's secret plan"}]
bo's notes:  []
no cookie:   {"error":"Sign in first"}
{"error":"No such note"} 404  (bo deletes ana's note 1)
{"error":"No such note"} 404  (bo deletes note 999)
204  (ana deletes note 1)
$ cd app && { env NODE_OPTIONS=--disable-warning=ExperimentalWarning DATABASE_PATH=$PWD/data.db PORT=3100 node .next/standalone/ch13/app/server.js > /dev/null & S=$!; sleep 1; curl -s -o /dev/null -w "no cookie:   %{http_code} → %{redirect_url}, %{size_download}-byte body\n" localhost:3100/; curl -s -o /dev/null -w "fake cookie: %{http_code} → %{redirect_url}, %{size_download}-byte body\n" -b "session=made-up" localhost:3100/; kill $S; }
no cookie:   307 → http://localhost:3100/login, 6-byte body
fake cookie: 307 → http://localhost:3100/login, 4339-byte body
```

The browser run: an unauthenticated visit goes to `/login`; Ana signs up and adds a note; `document.cookie` is empty because the session cookie is `HttpOnly`; Bo's short password is rejected before anything is stored, and Bo sees no notes. The API run: the wrong password gets 401; the `Set-Cookie` header carries all four attributes; each user sees only their own notes; Bo's attempt on Ana's note gets exactly the same 404 as a note that does not exist; Ana can delete it. Finally, both requests to `/` are redirected, but differently: without a cookie, the proxy answers with a 6-byte body before any rendering; with a made-up cookie, the proxy lets the request through and the page's own `getCurrentUser()` check redirects it.

Now the environment variables, with the server started with *different* values:

```console
$ cd app && { env NODE_OPTIONS=--disable-warning=ExperimentalWarning DATABASE_PATH=$PWD/data.db PORT=3100 MOTD="Message at run time" NEXT_PUBLIC_APP_NAME="v2 (run)" node .next/standalone/ch13/app/server.js > /dev/null & S=$!; sleep 1; curl -s localhost:3100/api/health; echo; curl -s localhost:3100/login | grep -oE '<p id="motd">[^<]*|<small id="app-name">[^<]*'; echo "inlined in browser JavaScript: $(grep -rl 'v1 (build)' .next/static | wc -l) file(s)"; kill $S; }
{"ok":true,"users":2,"motd":"Message at run time"}
<small id="app-name">v1 (build)
<p id="motd">Message at build time
inlined in browser JavaScript: 1 file(s)
```

### Deep & Tricky

**DT-29. Environment variables are not all read at run time. `NEXT_PUBLIC_…` variables are replaced by their values in the browser JavaScript at build time; and *any* variable read by a page that is prerendered (`/login` is static) is read once, at build time, and frozen into the HTML. Only dynamic code — the `/api/health` handler here, or dynamic pages — sees the run-time value. Building one image and promoting it from staging to production therefore needs runtime configuration for anything public: read it in dynamic server code and pass it down as props.** *(Next.js documentation, "Environment Variables — Bundling Environment Variables for the Browser" and "Runtime Environment Variables")*

### Mastery check

1. **Predict.** `/login` used `await connection()` before reading `process.env.MOTD`. Which message would it show, and what would `next build` print for the route?

## Chapter 13 Project — Private notes

### Requirements

The application above is the project: sign-up and sign-in with validation, hashed passwords, database-backed sessions in secure cookies, per-user notes added and deleted through Server Functions, a JSON API for the same data, a health check, and a standalone build. Check it with a browser run and an API run as two users.

### Review checklist

- Sessions never expire on the server unless used after `expires_at`. Add a cleanup query, and *sliding* expiry (extend a session when it is used, at most once a day).
- Add rate limiting to `/api/session` and `signIn`. Where does the counter live (DT-28 still applies)?
- Sign-out deletes one session. Add "sign out everywhere", and change the password-reset flow you would design to call it.
- What happens if two server instances run with SQLite files on different machines? Which parts of this code would change for PostgreSQL?

## Answers to the Chapter 13 prediction questions

**Unit 13.1, Q1.** Escaping depends on knowing every special character and context (quotes, backslashes, encodings, identifiers, `LIKE` wildcards) and on never forgetting a call; parameters remove the question entirely — the data is never parsed as SQL.

**Unit 13.2, Q1.** Cross-site scripting: an injected script could read the token and send it elsewhere, giving the attacker the session from another machine. `HttpOnly` limits the browser's JavaScript; `curl` is not a browser and reads every header anyway — it is the attacker *with* the token that `HttpOnly` stops from getting it.

**Unit 13.4, Q1.** The run-time message — `connection()` makes the page dynamic, so `next build` lists `/login` as `ƒ` (dynamic) instead of `○`. The `NEXT_PUBLIC_` name stays at its build-time value either way, because it was inlined into the browser code.

# Chapter 14 — Accessibility and internationalisation

*Goal: build interfaces that work for people who use a keyboard, a screen reader, or another language and script. You will audit components with an automated checker and find out what it cannot see, build a modal dialog on the native `<dialog>` element and test focus with the keyboard, format numbers, dates, lists, and plurals for any locale with the built-in `Intl` APIs, cause (and fix) a hydration mismatch that only appears in some time zones, and build a localised tabs component that follows the WAI-ARIA pattern — including in right-to-left Arabic.*

**What you will be able to do after this chapter**

- Use semantic HTML first, ARIA second, and test with the keyboard as well as with automated tools.
- Manage focus in dialogs and announce changes with live regions.
- Format values and choose plural forms with `Intl`, and keep messages in catalogues.
- Avoid hydration mismatches caused by locale and time zone.
- Implement a keyboard-accessible widget that mirrors correctly in right-to-left layouts.

```js
// file: vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  logLevel: "warn",
  optimizeDeps: { include: ["react", "react/jsx-dev-runtime", "react-dom/client"] },
});
```

```html
<!-- file: page.html -->
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Chapter 14</title></head>
<body><div id="root"></div><script type="module" src="./main.jsx"></script></body></html>
```

```console
$ for d in audit dialog project; do mkdir -p $d && cp page.html $d/index.html; done
```

## Topic 14A — Accessibility

## Unit 14.1 — Semantic HTML and automated audits

**One-line summary:** Native elements (`<button>`, `<a href>`, `<label>`, `<input>`) come with a role, a name, keyboard behaviour, and focus for free; React adds nothing to accessibility and removes nothing, so a `<div onClick>` is as broken in React as anywhere else; automated checkers such as axe-core find many problems quickly, but only those visible in the markup.

```jsx
// file: audit/main.jsx
import { createRoot } from "react-dom/client";

// Looks fine with a mouse and good eyesight. Everything here is a real, common bug.
function Bad() {
  return <main>
    <h1>Newsletter</h1>
    <img src="data:image/gif;base64,R0lGODlhAQABAAAAACw=" width="40" height="40" />
    <input type="email" placeholder="Your email" />
    <div className="btn" onClick={() => alert("subscribed")}>Subscribe</div>
    <p style={{ color: "#aaa" }}>We send one email a month.</p>
    <a href="/privacy"><svg width="16" height="16"><circle cx="8" cy="8" r="7" /></svg></a>
  </main>;
}

function Good() {
  return <main>
    <h1>Newsletter</h1>
    <img src="data:image/gif;base64,R0lGODlhAQABAAAAACw=" width="40" height="40" alt="" />   {/* decorative */}
    <label htmlFor="email">Email</label>
    <input id="email" type="email" autoComplete="email" />
    <button type="button" onClick={() => console.log("subscribed")}>Subscribe</button>
    <p style={{ color: "#595959" }}>We send one email a month.</p>
    <a href="/privacy" aria-label="Privacy policy"><svg width="16" height="16" aria-hidden="true"><circle cx="8" cy="8" r="7" /></svg></a>
  </main>;
}

const which = new URLSearchParams(location.search).get("v");
createRoot(document.getElementById("root")).render(which === "good" ? <Good /> : <Bad />);
```

```console
$ python3 ../tools/page.py http://localhost:5173/audit/ axe "press:Tab" focus "press:Tab" focus "press:Tab" focus --serve "npx vite --port 5173 --strictPort"
axe: 3 violation(s)
  color-contrast (serious): Elements must meet minimum color contrast ratio thresholds — p
  image-alt (critical): Images must have alternative text — img
  link-name (serious): Links must have discernible text — a
focus: input ""
focus: a ""
focus: body "NewsletterSubscribeWe send one email a m"
$ python3 ../tools/page.py "http://localhost:5173/audit/?v=good" axe "press:Tab" focus "press:Tab" focus "press:Tab" focus --serve "npx vite --port 5173 --strictPort"
axe: 0 violation(s)
focus: input#email ""
focus: button "Subscribe"
focus: a ""
```

The checker found three of the five bugs: the image with no `alt`, the icon-only link with no name, and the grey text (contrast 2.3:1, where 4.5:1 is required). The keyboard found the fourth: pressing Tab goes from the input to the link to the page itself — the "Subscribe" `<div>` can never be reached or activated without a mouse. The fixed version passes both, with `alt=""` marking the image as decorative and `aria-label` naming the link.

### Deep & Tricky

**DT-30. Automated checkers report what they can decide from the markup, which is a minority of real accessibility failures. axe-core cannot know that a `<div>` has a click handler (React attaches listeners at the root, so not even the DOM shows it), and it accepts a placeholder as an input's accessible name — although placeholder text disappears when typing starts and is often low-contrast. "0 violations" means "no *detectable* violations"; keyboard and screen-reader testing remain necessary.** *(axe-core rule descriptions, `label` — "placeholder" as an accessible name; W3C WAI, "Evaluating Web Accessibility — Overview"; WCAG 2.2 SC 2.1.1 Keyboard and SC 3.3.2 Labels or Instructions)*

### Mastery check

1. **Predict.** In `Bad`, you change the `<div>` to `<div role="button">`. Does it become keyboard-accessible?

## Unit 14.2 — Focus, dialogs, and live regions

**One-line summary:** When content appears or disappears, someone must decide where keyboard focus goes; the native `<dialog>` element opened with `showModal()` makes the rest of the page inert, moves focus into the dialog, closes on Escape (firing a `close` event), and returns focus to the element that opened it; a *live region* (`role="status"`, or `aria-live`) makes screen readers announce text that changes without moving focus.

```jsx
// file: dialog/Dialog.jsx
import { useEffect, useId, useRef } from "react";

// A modal dialog built on the native <dialog> element: showModal() makes the rest of the page
// inert, moves focus inside, closes on Escape, and returns focus to the opener when closed.
export function Dialog({ open, onClose, title, children }) {
  const ref = useRef(null);
  const titleId = useId();
  useEffect(() => {
    const dialog = ref.current;
    if (open && !dialog.open) dialog.showModal();
    if (!open && dialog.open) dialog.close();
  }, [open]);
  return (
    <dialog ref={ref} aria-labelledby={titleId} onClose={onClose}>   {/* Escape fires "close" */}
      <h2 id={titleId}>{title}</h2>
      {children}
    </dialog>
  );
}
```

```jsx
// file: dialog/main.jsx
import { useState } from "react";
import { createRoot } from "react-dom/client";
import { Dialog } from "./Dialog.jsx";

function App() {
  const [open, setOpen] = useState(false);
  const [message, setMessage] = useState("");
  function save(event) {
    event.preventDefault();
    setOpen(false);
    setMessage(`Saved “${new FormData(event.currentTarget).get("name")}”`);
  }
  return <main>
    <button id="opener" onClick={() => setOpen(true)}>Rename project</button>
    <button id="other">Something else</button>
    <Dialog open={open} onClose={() => setOpen(false)} title="Rename project">
      <form onSubmit={save}>
        <label>New name <input id="name" name="name" defaultValue="Apollo" /></label>
        <button type="button" onClick={() => setOpen(false)}>Cancel</button>
        <button id="save">Save</button>
      </form>
    </Dialog>
    <p role="status" id="status">{message}</p>    {/* announced by screen readers when it changes */}
  </main>;
}
createRoot(document.getElementById("root")).render(<App />);
```

```console
$ python3 ../tools/page.py http://localhost:5173/dialog/ "click:#opener" focus "press:Tab" focus "press:Tab" focus "press:Tab" focus "press:Tab" focus "press:Escape" focus "eval:document.querySelector('dialog').open" "click:#opener" "fill:#name|Gemini" "press:Enter" focus "eval:document.getElementById('status').textContent" "aria:main" --serve "npx vite --port 5173 --strictPort"
focus: input#name "Apollo"
focus: button "Cancel"
focus: button#save "Save"
focus: body "Rename projectSomething else   Rename pr"
focus: input#name "Apollo"
focus: button#opener "Rename project"
=> false
focus: button#opener "Rename project"
=> "Saved “Gemini”"
- main:
  - button "Rename project"
  - button "Something else"
  - status: Saved “Gemini”
```

Opening moves focus to the first field. Tab moves through Cancel and Save, then *leaves the document* (the `body` line: focus has gone to the browser's own interface, as it does after the last element of any page) and comes back to the first field — "Something else", behind the dialog, is never reached. Escape closes the dialog, React's state follows through `onClose`, and focus returns to "Rename project". Saving does the same and updates the status region, which a screen reader announces.

### Deep & Tricky

**DT-31. A live region must exist in the DOM *before* its content changes: screen readers watch regions that are already present, so rendering `{message && <p role="status">{message}</p>}` — creating the region and its text at the same moment — is often not announced. Render the empty region from the start (as `main.jsx` does) and change only its text.** *(WAI-ARIA 1.2 §6.6 "Live Region Attributes"; MDN, "ARIA live regions")*

### Pattern

**P-16. Prefer native elements that already implement an accessibility pattern — `<dialog>`, `<details>`, `<select>`, `<input type="date">`, the `popover` attribute — over hand-built ARIA widgets; when you must build one, follow the WAI-ARIA Authoring Practices pattern for it exactly, keyboard behaviour included.** *(W3C, "ARIA Authoring Practices Guide — Read Me First: No ARIA is better than bad ARIA")* *Trade-off:* native elements are harder to style (the `<select>` popup, until customisable selects are widely supported) and give less control; that control is precisely what hand-built widgets usually get wrong.

### Mastery check

1. **Predict.** `Dialog` used `dialog.show()` instead of `showModal()`. Starting on Save, you press Tab twice, then Escape. Where is focus, and is the dialog still open?

## Topic 14B — Internationalisation

## Unit 14.3 — Intl and message catalogues

**One-line summary:** The built-in `Intl` objects format numbers, currencies, dates, lists, and relative times for any locale, and `Intl.PluralRules` says which plural *category* a number takes in a language; translated messages live in catalogues keyed by message and category, never assembled from fragments with English grammar.

```js
// file: intl/formats.mjs
// The same values, formatted by the built-in Intl APIs for four locales.
const when = new Date("2026-03-05T14:30:00Z");
for (const locale of ["en-US", "de-DE", "hi-IN", "ar-EG"]) {
  const money = new Intl.NumberFormat(locale, { style: "currency", currency: "EUR" }).format(1234567.891);
  const date = new Intl.DateTimeFormat(locale, { dateStyle: "medium", timeStyle: "short", timeZone: "UTC" }).format(when);
  const list = new Intl.ListFormat(locale, { type: "conjunction" }).format(["Ana", "Bo", "Cy"]);
  const ago = new Intl.RelativeTimeFormat(locale, { numeric: "auto" }).format(-1, "day");
  console.log(`${locale.padEnd(6)} ${money} | ${date} | ${list} | ${ago}`);
}
```

```console
$ node intl/formats.mjs
en-US  €1,234,567.89 | Mar 5, 2026, 2:30 PM | Ana, Bo, and Cy | yesterday
de-DE  1.234.567,89 € | 05.03.2026, 14:30 | Ana, Bo und Cy | gestern
hi-IN  €12,34,567.89 | 5 मार्च 2026, 2:30 pm | Ana, Bo, और Cy | कल
ar-EG  ‏١٬٢٣٤٬٥٦٧٫٨٩ € | ٠٥‏/٠٣‏/٢٠٢٦، ٢:٣٠ م | Ana وBo وCy | أمس
```

Grouping (`1.234.567` in German, `12,34,567` in the Indian system), the decimal separator, the currency's position, the date order, 12- or 24-hour clocks, the digits themselves (Arabic-Indic in `ar-EG`), and the word for "and" all change. Hindi's `कल` means both "yesterday" and "tomorrow" — the formatter knows; string concatenation does not.

```js
// file: intl/messages.mjs
// Plural rules differ by language: English has 2 forms, Polish 3, Arabic 6.
export const messages = {
  en: { files: { one: "{n} file", other: "{n} files" } },
  pl: { files: { one: "{n} plik", few: "{n} pliki", many: "{n} plików", other: "{n} pliku" } },
  ar: { files: { zero: "لا ملفات", one: "ملف واحد", two: "ملفان", few: "{n} ملفات", many: "{n} ملفًا", other: "{n} ملف" } },
};

export function t(locale, key, n) {
  const forms = messages[locale][key];
  const category = new Intl.PluralRules(locale).select(n);       // "zero" | "one" | "two" | "few" | "many" | "other"
  return forms[category].replace("{n}", new Intl.NumberFormat(locale).format(n));
}

if (import.meta.main) {
  for (const locale of ["en", "pl", "ar"]) {
    const categories = new Intl.PluralRules(locale).resolvedOptions().pluralCategories;
    console.log(`${locale}: categories ${categories.join(", ")}`);
    console.log("   ", [0, 1, 2, 3, 5, 11, 22, 100].map((n) => t(locale, "files", n)).join(" · "));
  }
  console.log("naive English rule:", [1, 2, 5].map((n) => `${n} file${n === 1 ? "" : "s"}`).join(", "));
}
```

```console
$ node intl/messages.mjs
en: categories one, other
    0 files · 1 file · 2 files · 3 files · 5 files · 11 files · 22 files · 100 files
pl: categories few, many, one, other
    0 plików · 1 plik · 2 pliki · 3 pliki · 5 plików · 11 plików · 22 pliki · 100 plików
ar: categories few, many, one, two, zero, other
    لا ملفات · ملف واحد · ملفان · 3 ملفات · 5 ملفات · 11 ملفًا · 22 ملفًا · 100 ملف
naive English rule: 1 file, 2 files, 5 files
```

English has two plural forms, Polish four categories (with `2`, `3`, and `22` taking a different form from `5` and `11`), and Arabic six. The `n === 1 ? "" : "s"` rule that every English-speaking developer writes is wrong in most languages.

### Deep & Tricky

**DT-32. Formatting dates or numbers with the *machine's* locale or time zone in server-rendered components produces different text on the server and in the browser, and React reports a hydration mismatch and re-renders the tree on the client (a visible flash, and the server's work wasted). Pass the locale and time zone explicitly, from the user's settings or the request — or render such values only on the client.** *(React documentation, "hydrateRoot — Hydrating server-rendered HTML" and the list of hydration mismatch causes; ECMA-402 §11 `Intl.DateTimeFormat` default time zone)*

```jsx
// file: hydrate/App.jsx
const MEETING = new Date("2026-03-05T14:30:00Z");

// Formats with the time zone of whatever machine runs it: the server's, then the browser's.
export function Bad() {
  return <p>Meeting at {MEETING.toLocaleTimeString("en-GB", { hour: "2-digit", minute: "2-digit" })}</p>;
}

// Formats with an explicit time zone (in a real app: the user's saved preference).
export function Good({ timeZone }) {
  const time = MEETING.toLocaleTimeString("en-GB", { hour: "2-digit", minute: "2-digit", timeZone });
  return <p>Meeting at {time} ({timeZone})</p>;
}
```

```jsx
// file: hydrate/prerender.jsx
// "Server-side rendering": render both components to HTML and write the page.
import { writeFileSync } from "node:fs";
import { renderToString } from "react-dom/server";
import { Bad, Good } from "./App.jsx";

const page = `<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Hydration</title></head>
<body><div id="bad">${renderToString(<Bad />)}</div><div id="good">${renderToString(<Good timeZone="Asia/Tokyo" />)}</div>
<script type="module" src="./main.jsx"></script></body></html>`;
writeFileSync(new URL("./index.html", import.meta.url), page);
console.log(`server time zone: ${Intl.DateTimeFormat().resolvedOptions().timeZone}`);
console.log(page.match(/<div id="bad">.*?<\/div>|<div id="good">.*?<\/div>/g).join("\n"));
```

```jsx
// file: hydrate/main.jsx
import { hydrateRoot } from "react-dom/client";
import { Bad, Good } from "./App.jsx";

console.log(`browser time zone: ${Intl.DateTimeFormat().resolvedOptions().timeZone}`);
const report = (id) => ({ onRecoverableError: (error) => console.log(`#${id}: ${error.message.split("\n")[0]}`) });
hydrateRoot(document.getElementById("bad"), <Bad />, report("bad"));
hydrateRoot(document.getElementById("good"), <Good timeZone="Asia/Tokyo" />, report("good"));
```

```console
$ TZ=UTC npx tsx hydrate/prerender.jsx
server time zone: UTC
<div id="bad"><p>Meeting at <!-- -->14:30</p></div>
<div id="good"><p>Meeting at <!-- -->23:30<!-- --> (<!-- -->Asia/Tokyo<!-- -->)</p></div>
$ python3 ../tools/page.py http://localhost:5173/hydrate/ "wait:300" "eval:document.body.innerText" --tz Asia/Tokyo --serve "npx vite --port 5173 --strictPort"
browser time zone: Asia/Tokyo
#bad: Hydration failed because the server rendered text didn't match the client. As a result this tree will be regenerated on the client. This can happen if a SSR-ed Client Component used:
=> "Meeting at 23:30\n\nMeeting at 23:30 (Asia/Tokyo)"
```

The server (in UTC) rendered 14:30; the browser (in Tokyo) rendered 23:30, so React threw away the server's HTML for `#bad` and rendered it again. `Good` names its time zone, so both sides produce 23:30 and hydration succeeds.

### Mastery check

1. **Predict.** A browser in London (UTC+0 in March) opens the page. Does `#bad` report a mismatch?

## Chapter 14 Project — Localised, accessible tabs

### Requirements

Write a `Tabs` component that follows the WAI-ARIA tabs pattern: a `tablist` with a name, `tab`s with `aria-selected` and `aria-controls`, `tabpanel`s labelled by their tabs, a single tab stop for the tab list (roving `tabindex`), arrow keys that move and select (reversed in right-to-left text), and Home/End. Use it for an order summary in English and Arabic, with the document's `lang` and `dir` set, numbers, currency, dates, and plurals formatted for the locale, and zero axe violations in both.

### Reference solution

```js
// file: project/i18n.js
export const LOCALES = {
  en: { dir: "ltr", name: "English" },
  ar: { dir: "rtl", name: "العربية" },
};

const MESSAGES = {
  en: {
    title: "Order summary", tabs: "Order details", items: "Items", shipping: "Shipping", payment: "Payment",
    count: { one: "{n} item", other: "{n} items" }, total: "Total: {amount}", arrives: "Arrives {date}",
    paid: "Paid by card ending {last4}", language: "Language",
  },
  ar: {
    title: "ملخص الطلب", tabs: "تفاصيل الطلب", items: "المنتجات", shipping: "الشحن", payment: "الدفع",
    count: { zero: "لا منتجات", one: "منتج واحد", two: "منتجان", few: "{n} منتجات", many: "{n} منتجًا", other: "{n} منتج" },
    total: "الإجمالي: {amount}", arrives: "يصل {date}", paid: "دُفع ببطاقة تنتهي بـ {last4}", language: "اللغة",
  },
};

export function makeT(locale) {
  const nf = new Intl.NumberFormat(locale);
  const pr = new Intl.PluralRules(locale);
  return function t(key, values = {}) {
    let message = MESSAGES[locale][key];
    if (typeof message === "object") message = message[pr.select(values.n)] ?? message.other;
    return message.replace(/\{(\w+)\}/g, (_, name) =>
      typeof values[name] === "number" ? nf.format(values[name]) : String(values[name]));
  };
}
```

```jsx
// file: project/Tabs.jsx
import { useId, useRef, useState } from "react";

// The WAI-ARIA tabs pattern: one tab stop for the whole tab list (roving tabindex),
// arrow keys move between tabs (mirrored in right-to-left text), Home and End jump to the ends.
export function Tabs({ label, tabs, dir = "ltr" }) {
  const [selected, setSelected] = useState(0);
  const refs = useRef([]);
  const base = useId();

  function onKeyDown(event) {
    const last = tabs.length - 1;
    const forward = dir === "rtl" ? "ArrowLeft" : "ArrowRight";
    const back = dir === "rtl" ? "ArrowRight" : "ArrowLeft";
    const next = { [forward]: selected === last ? 0 : selected + 1,
                   [back]: selected === 0 ? last : selected - 1, Home: 0, End: last }[event.key];
    if (next === undefined) return;
    event.preventDefault();                       // Home/End would otherwise scroll the page
    setSelected(next);
    refs.current[next].focus();
  }

  return <div>
    <div role="tablist" aria-label={label} onKeyDown={onKeyDown}>
      {tabs.map((tab, i) => (
        <button key={tab.id} role="tab" id={`${base}-tab-${i}`} aria-controls={`${base}-panel-${i}`}
          aria-selected={i === selected} tabIndex={i === selected ? 0 : -1}
          ref={(el) => { refs.current[i] = el; }} onClick={() => setSelected(i)}>{tab.label}</button>
      ))}
    </div>
    {tabs.map((tab, i) => (
      <div key={tab.id} role="tabpanel" id={`${base}-panel-${i}`} aria-labelledby={`${base}-tab-${i}`}
        tabIndex={0} hidden={i !== selected}>{tab.content}</div>
    ))}
  </div>;
}
```

```jsx
// file: project/main.jsx
import { useEffect, useState } from "react";
import { createRoot } from "react-dom/client";
import { Tabs } from "./Tabs.jsx";
import { LOCALES, makeT } from "./i18n.js";

const ORDER = { items: ["Lamp", "Desk", "Chair"], total: 1234.5, arrives: new Date("2026-03-05T12:00:00Z"), last4: "4242" };

function App() {
  const [locale, setLocale] = useState(new URLSearchParams(location.search).get("lang") ?? "en");
  const { dir } = LOCALES[locale];
  const t = makeT(locale);
  useEffect(() => {                                   // the whole document's language and direction
    document.documentElement.lang = locale;
    document.documentElement.dir = dir;
  }, [locale, dir]);

  const money = new Intl.NumberFormat(locale, { style: "currency", currency: "EUR" }).format(ORDER.total);
  const date = new Intl.DateTimeFormat(locale, { dateStyle: "full", timeZone: "UTC" }).format(ORDER.arrives);
  return <main>
    <h1>{t("title")}</h1>
    <label>{t("language")}{" "}
      <select value={locale} onChange={(e) => setLocale(e.target.value)}>
        {Object.entries(LOCALES).map(([code, l]) => <option key={code} value={code} lang={code}>{l.name}</option>)}
      </select>
    </label>
    <Tabs key={locale} label={t("tabs")} dir={dir} tabs={[
      { id: "items", label: t("items"), content: <p>{t("count", { n: ORDER.items.length })} — {t("total", { amount: money })}</p> },
      { id: "shipping", label: t("shipping"), content: <p>{t("arrives", { date })}</p> },
      { id: "payment", label: t("payment"), content: <p>{t("paid", { last4: ORDER.last4 })}</p> },
    ]} />
  </main>;
}
createRoot(document.getElementById("root")).render(<App />);
window.state = () => `${document.documentElement.lang}/${document.documentElement.dir} selected="${document.querySelector('[aria-selected=true]').textContent}" panel="${document.querySelector('[role=tabpanel]:not([hidden])').textContent}"`;
```

```console
$ python3 ../tools/page.py http://localhost:5173/project/ axe "eval:state()" "press:Tab" "press:Tab" focus "press:ArrowRight" "eval:state()" "press:End" "eval:state()" "press:ArrowRight" "eval:state()" "press:Tab" focus "aria:[role=tablist]" "select:select|ar" "eval:state()" --serve "npx vite --port 5173 --strictPort"
axe: 0 violation(s)
=> "en/ltr selected=\"Items\" panel=\"3 items — Total: €1,234.50\""
focus: button#_r_0_-tab-0 "Items"
=> "en/ltr selected=\"Shipping\" panel=\"Arrives Thursday, March 5, 2026\""
=> "en/ltr selected=\"Payment\" panel=\"Paid by card ending 4242\""
=> "en/ltr selected=\"Items\" panel=\"3 items — Total: €1,234.50\""
focus: div#_r_0_-panel-0 "3 items — Total: €1,234.50"
- tablist "Order details":
  - tab "Items" [selected]
  - tab "Shipping"
  - tab "Payment"
=> "ar/rtl selected=\"المنتجات\" panel=\"3 منتجات — الإجمالي: ‏1,234.50 €\""
$ python3 ../tools/page.py "http://localhost:5173/project/?lang=ar" axe "eval:state()" "click:[role=tab]" "press:ArrowLeft" "eval:state()" "press:ArrowLeft" "eval:state()" "press:ArrowRight" "eval:state()" --serve "npx vite --port 5173 --strictPort"
axe: 0 violation(s)
=> "ar/rtl selected=\"المنتجات\" panel=\"3 منتجات — الإجمالي: ‏1,234.50 €\""
=> "ar/rtl selected=\"الشحن\" panel=\"يصل الخميس، 5 مارس 2026\""
=> "ar/rtl selected=\"الدفع\" panel=\"دُفع ببطاقة تنتهي بـ 4242\""
=> "ar/rtl selected=\"الشحن\" panel=\"يصل الخميس، 5 مارس 2026\""
```

In English, Tab reaches the selected tab only once; ArrowRight selects Shipping; End selects Payment; ArrowRight wraps to Items; the next Tab goes to the panel, not to the other tabs. In Arabic, where the tabs run from right to left, ArrowLeft moves *forward* and ArrowRight back. Both pass the automated audit — which, as Unit 14.1 showed, is the minimum, not the proof: the keyboard runs are the real test.

### Review checklist

- The tabs activate on arrow keys ("automatic activation"). When should a tab list use manual activation (arrows move focus, Enter selects) instead?
- `t()` returns the message key's text unchanged if a translation is missing in one language. What should it do, and how would you find missing keys before release?
- The currency is always EUR. Where does the currency come from in a real shop, and why must it never be inferred from the locale?

## Answers to the Chapter 14 prediction questions

**Unit 14.1, Q1.** No. `role="button"` changes what a screen reader announces, but not behaviour: the element still cannot be focused (it needs `tabIndex={0}`) or activated with Enter and Space (it needs a key handler). A `<button>` has all of this.

**Unit 14.2, Q1.** On "Rename project", behind the dialog — and the dialog is still open. `show()` opens a *non-modal* dialog: the rest of the page is not inert, so focus leaves the document and comes back to the page's first button, and Escape closes only modal dialogs. (Checked by changing the one call and repeating the keyboard run.)

**Unit 14.3, Q1.** No: in March London is on UTC, the same as the server, so both render 14:30. The bug is invisible to everyone in the server's time zone — which is usually where its developers are.

# Chapter 15 — Capstone: a full-stack task board

*Goal: put the whole book to work in one application, built the way a small production app is built: TypeScript throughout; Server Components for pages and data; a data access layer that enforces ownership; validated Server Functions; optimistic updates that roll back; streamed slow parts; accessible, keyboard-friendly markup; unit tests for the rules and end-to-end tests for the journeys; and a standalone production build. Three surprises turned up while building it — each is a Deep & Tricky item below, found by running the code, not by reading about it.*

**What you will have built**

- A task board where users register and sign in, add tasks, move them between *To do*, *Doing*, and *Done* with immediate feedback, search them, and delete them — and never see another user's tasks.
- A test suite: 18 unit tests for the validation rules and two end-to-end journeys in Chromium against the production build.

## The design

| Concern | Decision | Chapter |
|---|---|---|
| Language | TypeScript, `strict`; `unknown` input parsed by pure functions in `lib/validate.ts` | 9 |
| Rendering | the board page is a dynamic Server Component; only the form, the board, and the login form are Client Components | 12 |
| Data | SQLite through a data access layer (`lib/tasks.ts`) whose every query includes `user_id` | 13 |
| Identity | scrypt password hashes, hashed session tokens in `HttpOnly` cookies, an optimistic `proxy.ts` | 13 |
| Mutations | Server Functions with `useActionState` (forms) and `useOptimistic` (moves) | 4, 12 |
| Slow parts | the statistics stream in behind `<Suspense>` | 12 |
| Search | a GET form, so a search is a URL that can be shared and reloaded | 7 |
| Accessibility | labelled regions, buttons named by task, live regions for errors, checked with axe and Playwright's role queries | 14 |
| Delivery | `output: "standalone"`; tests run against that build | 11, 13 |

## Unit 15.1 — Types, validation, and the data layer

**One-line summary:** Shared types and pure validation functions are the contract between the browser and the server; the data access layer is the only code that touches the `tasks` table, and it cannot be called without a user.

```jsonc
// file: taskboard/tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022", "lib": ["dom", "dom.iterable", "esnext"],
    "strict": true, "noEmit": true, "skipLibCheck": true, "allowJs": false,
    "module": "esnext", "moduleResolution": "bundler", "resolveJsonModule": true,
    "isolatedModules": true, "esModuleInterop": true, "jsx": "react-jsx", "incremental": true,
    "plugins": [{ "name": "next" }],
    "types": ["node"]
  },
  "include": ["next-env.d.ts", ".next/types/**/*.ts", ".next/dev/types/**/*.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules", ".next"]
}
```

```js
// file: taskboard/next.config.mjs
export default { output: "standalone" };
```

```ts
// file: taskboard/lib/types.ts
export const STATUSES = ["todo", "doing", "done"] as const;
export type Status = (typeof STATUSES)[number];
export type Task = { id: number; title: string; status: Status };
export type User = { id: number; email: string };
export type FormState = { error?: string; values?: Record<string, string> };
```

```ts
// file: taskboard/lib/validate.ts
// Pure functions: everything that comes from a form or a URL is `unknown` until checked.
import { STATUSES, type Status } from "./types";

export type Result<T> = { ok: true; value: T } | { ok: false; error: string };

export function parseTitle(input: unknown): Result<string> {
  const title = typeof input === "string" ? input.trim().replace(/\s+/g, " ") : "";
  if (title.length === 0) return { ok: false, error: "Give the task a title" };
  if (title.length > 120) return { ok: false, error: `Titles have at most 120 characters (this one has ${title.length})` };
  return { ok: true, value: title };
}

export function parseStatus(input: unknown): Result<Status> {
  return STATUSES.includes(input as Status) ? { ok: true, value: input as Status }
    : { ok: false, error: `Unknown status ${JSON.stringify(input)}` };
}

export function parseCredentials(email: unknown, password: unknown): Result<{ email: string; password: string }> {
  const e = typeof email === "string" ? email.trim().toLowerCase() : "";
  const p = typeof password === "string" ? password : "";
  if (!/^\S+@\S+\.\S+$/.test(e)) return { ok: false, error: "Enter a valid email address" };
  if (p.length < 8) return { ok: false, error: "Passwords have at least 8 characters" };
  return { ok: true, value: { email: e, password: p } };
}

/** The search box: a trimmed, length-limited string, or "" for none. */
export function parseQuery(input: unknown): string {
  return typeof input === "string" ? input.trim().slice(0, 50) : "";
}
```

```ts
// file: taskboard/lib/db.ts
import "server-only";
import { DatabaseSync } from "node:sqlite";

export const db = new DatabaseSync(process.env.DATABASE_PATH ?? "taskboard.db");
db.exec(`
  PRAGMA foreign_keys = ON;
  PRAGMA journal_mode = WAL;
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY, email TEXT NOT NULL UNIQUE, password_hash TEXT NOT NULL);
  CREATE TABLE IF NOT EXISTS sessions (
    token_hash TEXT PRIMARY KEY, user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    expires_at INTEGER NOT NULL);
  CREATE TABLE IF NOT EXISTS tasks (
    id INTEGER PRIMARY KEY, user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title TEXT NOT NULL, status TEXT NOT NULL DEFAULT 'todo' CHECK (status IN ('todo', 'doing', 'done')));
  CREATE INDEX IF NOT EXISTS tasks_by_user ON tasks (user_id, status);
`);
```

```ts
// file: taskboard/lib/password.ts
import { scryptSync, randomBytes, timingSafeEqual } from "node:crypto";

export function hashPassword(password: string): string {
  const salt = randomBytes(16);
  const hash = scryptSync(password.normalize("NFKC"), salt, 64);
  return `${salt.toString("base64url")}:${hash.toString("base64url")}`;
}

export function verifyPassword(password: string, stored: string): boolean {
  const [salt, hash] = stored.split(":").map((s) => Buffer.from(s, "base64url"));
  return timingSafeEqual(scryptSync(password.normalize("NFKC"), salt, 64), hash);
}
```

```ts
// file: taskboard/lib/session.ts
import "server-only";
import { cache } from "react";
import { cookies } from "next/headers";
import { randomBytes, createHash } from "node:crypto";
import { db } from "./db";
import { hashPassword, verifyPassword } from "./password";
import type { User } from "./types";

const COOKIE = "session";
const WEEK = 7 * 24 * 60 * 60;
const sha256 = (s: string) => createHash("sha256").update(s).digest("base64url");
const DUMMY_HASH = hashPassword("dummy password for timing");

export async function createSession(userId: number) {
  const token = randomBytes(32).toString("base64url");
  db.prepare("INSERT INTO sessions (token_hash, user_id, expires_at) VALUES (?, ?, ?)")
    .run(sha256(token), userId, Date.now() + WEEK * 1000);
  (await cookies()).set(COOKIE, token, {
    httpOnly: true, sameSite: "lax", secure: process.env.NODE_ENV === "production", path: "/", maxAge: WEEK,
  });
}

export const getCurrentUser = cache(async (): Promise<User | null> => {
  const token = (await cookies()).get(COOKIE)?.value;
  if (!token) return null;
  const row = db.prepare(`SELECT users.id, users.email, sessions.expires_at FROM sessions
    JOIN users ON users.id = sessions.user_id WHERE token_hash = ?`).get(sha256(token)) as
    { id: number; email: string; expires_at: number } | undefined;
  return row && row.expires_at > Date.now() ? { id: row.id, email: row.email } : null;
});

export async function destroySession() {
  const store = await cookies();
  const token = store.get(COOKIE)?.value;
  if (token) db.prepare("DELETE FROM sessions WHERE token_hash = ?").run(sha256(token));
  store.delete(COOKIE);
}

export function authenticate(email: string, password: string): User | null {
  const row = db.prepare("SELECT id, email, password_hash FROM users WHERE email = ?").get(email) as
    { id: number; email: string; password_hash: string } | undefined;
  const ok = verifyPassword(password, row?.password_hash ?? DUMMY_HASH);
  return ok && row ? { id: row.id, email: row.email } : null;
}

export function register(email: string, password: string): User | null {
  try {
    const { lastInsertRowid } = db.prepare("INSERT INTO users (email, password_hash) VALUES (?, ?)")
      .run(email, hashPassword(password));
    return { id: Number(lastInsertRowid), email };
  } catch (e) {
    if (String((e as Error).message).includes("UNIQUE")) return null;
    throw e;
  }
}
```

```ts
// file: taskboard/lib/tasks.ts
// The data access layer: every function takes the user, and every query is limited to them.
import "server-only";
import { db } from "./db";
import type { Status, Task, User } from "./types";

export function listTasks(user: User, query = ""): Task[] {
  const rows = db.prepare(`SELECT id, title, status FROM tasks WHERE user_id = ? AND title LIKE ? ESCAPE '\\'
    ORDER BY id`).all(user.id, `%${query.replace(/[\\%_]/g, (c) => "\\" + c)}%`) as Task[];
  // node:sqlite returns objects with a null prototype, which React will not send to the browser.
  return rows.map(({ id, title, status }) => ({ id, title, status }));
}

export function createTask(user: User, title: string): void {
  db.prepare("INSERT INTO tasks (user_id, title) VALUES (?, ?)").run(user.id, title);
}

/** Returns false if the task does not exist or belongs to someone else. */
export function moveTask(user: User, id: number, status: Status): boolean {
  return db.prepare("UPDATE tasks SET status = ? WHERE id = ? AND user_id = ?").run(status, id, user.id).changes === 1;
}

export function deleteTask(user: User, id: number): boolean {
  return db.prepare("DELETE FROM tasks WHERE id = ? AND user_id = ?").run(id, user.id).changes === 1;
}

export async function stats(user: User): Promise<Record<Status, number>> {
  await new Promise((r) => setTimeout(r, Number(process.env.STATS_DELAY_MS ?? 0)));   // a slow report
  const rows = db.prepare("SELECT status, count(*) AS n FROM tasks WHERE user_id = ? GROUP BY status")
    .all(user.id) as { status: Status; n: number }[];
  return { todo: 0, doing: 0, done: 0, ...Object.fromEntries(rows.map((r) => [r.status, r.n])) };
}
```

`tsconfig.json` already contains everything Next.js would otherwise add (it rewrites the file on the first build if `esModuleInterop` or its type folders are missing). The validation rules are pure functions, so they are tested without a server, a database, or a browser:

```ts
// file: taskboard/vitest.config.ts
import { defineConfig } from "vitest/config";
export default defineConfig({ test: { include: ["unit/**/*.test.ts"], environment: "node" } });
```

```ts
// file: taskboard/unit/validate.test.ts
import { describe, it, expect } from "vitest";
import { parseCredentials, parseQuery, parseStatus, parseTitle } from "../lib/validate";

describe("parseTitle", () => {
  it("trims and collapses whitespace", () => {
    expect(parseTitle("  Write   the\ttests ")).toEqual({ ok: true, value: "Write the tests" });
  });
  it.each([[""], ["   "], [undefined], [42], [null]])("rejects %j", (input) => {
    expect(parseTitle(input)).toEqual({ ok: false, error: "Give the task a title" });
  });
  it("accepts 120 characters and rejects 121", () => {
    expect(parseTitle("x".repeat(120)).ok).toBe(true);
    expect(parseTitle("x".repeat(121))).toEqual({ ok: false, error: "Titles have at most 120 characters (this one has 121)" });
  });
});

describe("parseStatus", () => {
  it.each(["todo", "doing", "done"])("accepts %s", (s) => expect(parseStatus(s)).toEqual({ ok: true, value: s }));
  it.each(["DONE", "archived", "", null, { status: "done" }])("rejects %j", (s) => expect(parseStatus(s).ok).toBe(false));
});

describe("parseCredentials and parseQuery", () => {
  it("normalises the email", () => {
    expect(parseCredentials(" Ana@Example.ORG ", "correct horse"))
      .toEqual({ ok: true, value: { email: "ana@example.org", password: "correct horse" } });
  });
  it("limits the query to 50 characters", () => expect(parseQuery("q".repeat(80))).toHaveLength(50));
  it("treats a repeated parameter (an array) as no query", () => expect(parseQuery(["a", "b"])).toBe(""));
});
```

```console
$ cd taskboard && npx vitest run --reporter=verbose 2>&1 | grep -E "✓|×|Tests " | sed -E "s/ [0-9]+ms$//"
 ✓ unit/validate.test.ts > parseTitle > trims and collapses whitespace
 ✓ unit/validate.test.ts > parseTitle > rejects ""
 ✓ unit/validate.test.ts > parseTitle > rejects "   "
 ✓ unit/validate.test.ts > parseTitle > rejects undefined
 ✓ unit/validate.test.ts > parseTitle > rejects 42
 ✓ unit/validate.test.ts > parseTitle > rejects null
 ✓ unit/validate.test.ts > parseTitle > accepts 120 characters and rejects 121
 ✓ unit/validate.test.ts > parseStatus > accepts todo
 ✓ unit/validate.test.ts > parseStatus > accepts doing
 ✓ unit/validate.test.ts > parseStatus > accepts done
 ✓ unit/validate.test.ts > parseStatus > rejects "DONE"
 ✓ unit/validate.test.ts > parseStatus > rejects "archived"
 ✓ unit/validate.test.ts > parseStatus > rejects ""
 ✓ unit/validate.test.ts > parseStatus > rejects null
 ✓ unit/validate.test.ts > parseStatus > rejects {"status":"done"}
 ✓ unit/validate.test.ts > parseCredentials and parseQuery > normalises the email
 ✓ unit/validate.test.ts > parseCredentials and parseQuery > limits the query to 50 characters
 ✓ unit/validate.test.ts > parseCredentials and parseQuery > treats a repeated parameter (an array) as no query
      Tests  18 passed (18)
```

### Deep & Tricky

**DT-33. `node:sqlite` returns each row as an object with a *null prototype* (`[Object: null prototype] { id: 1, … }`), and React refuses to serialise such objects from a Server Component to a Client Component — the first version of this board crashed with "Only plain objects, and a few built-ins, can be passed to Client Components … null prototypes are not supported" as soon as there was a task to show. Copy rows into plain objects at the data layer boundary (`listTasks` does), which is also where to choose exactly which columns leave the server.** *(Node.js 22 `node:sqlite` — rows from `StatementSync.all()` print as `[Object: null prototype] { … }`, checked with `node -e`; React documentation, "'use client' — Serializable types")*

**DT-34. In SQL `LIKE`, `%` and `_` are wildcards, and parameters do not change that: a parameter stops injection (DT in Unit 13.1) but the *pattern* is still interpreted. Searching for `%` without escaping would match every task. `listTasks` escapes `\`, `%`, and `_` and declares `ESCAPE '\'`; the end-to-end test searches for `%` and expects exactly one task.** *(SQLite documentation, "The LIKE, GLOB, REGEXP, MATCH, and extract operators")*

## Unit 15.2 — Pages, Server Functions, and optimistic updates

**One-line summary:** The page reads the user, the query, and the tasks on the server and passes plain data to the board; Server Functions re-check the user, validate, write through the data layer, and revalidate; the board applies a move at once with `useOptimistic`, marks it pending, and lets the server's next render replace it — or drops it and shows an error.

```ts
// file: taskboard/app/actions.ts
"use server";
import { redirect } from "next/navigation";
import { revalidatePath } from "next/cache";
import { authenticate, createSession, destroySession, getCurrentUser, register } from "../lib/session";
import { createTask, deleteTask, moveTask } from "../lib/tasks";
import { parseCredentials, parseStatus, parseTitle } from "../lib/validate";
import type { FormState } from "../lib/types";

async function requireUser() {
  const user = await getCurrentUser();
  if (!user) redirect("/login");
  return user;
}

export async function signIn(_: FormState, form: FormData): Promise<FormState> {
  const email = String(form.get("email") ?? "");
  const c = parseCredentials(form.get("email"), form.get("password"));
  if (!c.ok) return { error: c.error, values: { email } };
  const user = form.get("intent") === "register" ? register(c.value.email, c.value.password)
    : authenticate(c.value.email, c.value.password);
  if (!user) return {
    error: form.get("intent") === "register" ? "That email is already registered" : "Wrong email or password",
    values: { email },
  };
  await createSession(user.id);
  redirect("/");
}

export async function signOut() {
  await destroySession();
  redirect("/login");
}

export async function addTask(_: FormState, form: FormData): Promise<FormState> {
  const user = await requireUser();
  const title = parseTitle(form.get("title"));
  if (!title.ok) return { error: title.error, values: { title: String(form.get("title") ?? "") } };
  createTask(user, title.value);
  revalidatePath("/");
  return {};
}

export async function changeStatus(id: number, status: unknown): Promise<{ error?: string }> {
  const user = await requireUser();
  const s = parseStatus(status);
  if (!s.ok) return { error: s.error };
  await new Promise((r) => setTimeout(r, Number(process.env.MOVE_DELAY_MS ?? 0)));     // network + database time
  const moved = moveTask(user, Number(id), s.value);
  revalidatePath("/");                     // either way, the board should show the database's truth
  return moved ? {} : { error: "That task no longer exists" };
}

export async function removeTask(id: number): Promise<void> {
  const user = await requireUser();
  deleteTask(user, Number(id));
  revalidatePath("/");
}
```

```tsx
// file: taskboard/app/layout.tsx
import type { ReactNode } from "react";
import "./style.css";

export const metadata = { title: "Taskboard" };
export default function RootLayout({ children }: { children: ReactNode }) {
  return <html lang="en"><body>{children}</body></html>;
}
```

```css
/* file: taskboard/app/style.css */
body { font: 16px/1.5 system-ui, sans-serif; margin: 0 auto; max-width: 60rem; padding: 1rem; color: #1a1a1a; }
.columns { display: grid; grid-template-columns: repeat(auto-fit, minmax(14rem, 1fr)); gap: 1rem; }
.columns section { background: #f3f3f3; border-radius: 8px; padding: 0.5rem 1rem; }
.columns li { background: white; border-radius: 6px; padding: 0.5rem; margin-bottom: 0.5rem; list-style: none; }
.columns ul { padding: 0; }
.pending { opacity: 0.6; }
[role="alert"] { color: #b00020; }
button { font: inherit; }
```

```tsx
// file: taskboard/app/login/page.tsx
import { LoginForm } from "./LoginForm";

export default function Login() {
  return <main><h1>Taskboard</h1><LoginForm /></main>;
}
```

```tsx
// file: taskboard/app/login/LoginForm.tsx
"use client";
import { useActionState } from "react";
import { signIn } from "../actions";

export function LoginForm() {
  const [state, action, pending] = useActionState(signIn, {});
  return (
    <form action={action}>
      <p><label>Email <input name="email" type="email" autoComplete="email" defaultValue={state.values?.email} /></label></p>
      <p><label>Password <input name="password" type="password" autoComplete="current-password" /></label></p>
      {state.error && <p role="alert">{state.error}</p>}
      <button name="intent" value="login" disabled={pending}>Sign in</button>{" "}
      <button name="intent" value="register" disabled={pending}>Create account</button>
    </form>
  );
}
```

```tsx
// file: taskboard/app/page.tsx
import { Suspense } from "react";
import { redirect } from "next/navigation";
import { getCurrentUser } from "../lib/session";
import { listTasks } from "../lib/tasks";
import { parseQuery } from "../lib/validate";
import { Board } from "./Board";
import { NewTask } from "./NewTask";
import { Stats } from "./Stats";
import { signOut } from "./actions";

export default async function Home({ searchParams }: { searchParams: Promise<{ q?: string }> }) {
  const user = await getCurrentUser();
  if (!user) redirect("/login");
  const q = parseQuery((await searchParams).q);
  const tasks = listTasks(user, q);
  return <main>
    <header>
      <h1>Taskboard</h1>
      <form action={signOut}><span id="who">{user.email}</span> <button>Sign out</button></form>
    </header>
    <Suspense fallback={<p id="stats">Counting…</p>}><Stats user={user} /></Suspense>
    <NewTask />
    <form role="search">                                            {/* a GET form: the query is in the URL */}
      <label>Search <input name="q" type="search" defaultValue={q} /></label> <button>Search</button>
    </form>
    <Board tasks={tasks} />
  </main>;
}
```

```tsx
// file: taskboard/app/Stats.tsx
import { stats } from "../lib/tasks";
import type { User } from "../lib/types";

export async function Stats({ user }: { user: User }) {
  const s = await stats(user);
  const total = s.todo + s.doing + s.done;
  const percent = total ? Math.round((s.done / total) * 100) : 0;
  return <p id="stats">{s.done} of {total} done ({percent}%)</p>;
}
```

```tsx
// file: taskboard/app/NewTask.tsx
"use client";
import { useActionState } from "react";
import { addTask } from "./actions";

export function NewTask() {
  const [state, action, pending] = useActionState(addTask, {});
  return (
    <form action={action} aria-label="New task">
      <label>New task <input name="title" defaultValue={state.values?.title}
        aria-invalid={!!state.error} aria-describedby="title-error" /></label>{" "}
      <button disabled={pending}>Add</button>
      <p id="title-error" role="alert">{state.error}</p>
    </form>
  );
}
```

```tsx
// file: taskboard/app/Board.tsx
"use client";
import { useOptimistic, useState, startTransition } from "react";
import { changeStatus, removeTask } from "./actions";
import { STATUSES, type Status, type Task } from "../lib/types";

const LABEL: Record<Status, string> = { todo: "To do", doing: "Doing", done: "Done" };

export function Board({ tasks }: { tasks: Task[] }) {
  // Show a move at once; the server's next render replaces it (or it is dropped on failure).
  const [shown, applyMove] = useOptimistic(tasks, (current, move: { id: number; status: Status }) =>
    current.map((t) => (t.id === move.id ? { ...t, status: move.status, pending: true } : t)));
  const [error, setError] = useState("");

  function move(task: Task, status: Status) {
    setError("");
    startTransition(async () => {
      applyMove({ id: task.id, status });
      const result = await changeStatus(task.id, status);
      if (result.error) setError(`Could not move “${task.title}”: ${result.error}`);
    });
  }

  return <>
    <p role="alert" id="board-error">{error}</p>
    <div className="columns">
      {STATUSES.map((status) => {
        const column = shown.filter((t) => t.status === status);
        return <section key={status} aria-labelledby={`h-${status}`}>
          <h2 id={`h-${status}`}>{LABEL[status]} ({column.length})</h2>
          <ul>{column.map((task) => (
            <li key={task.id} className={"pending" in task ? "pending" : undefined} data-task={task.id}>
              <span>{task.title}</span>{" "}
              {STATUSES.filter((s) => s !== status).map((s) => (
                <button key={s} onClick={() => move(task, s)} aria-label={`Move “${task.title}” to ${LABEL[s]}`}>
                  → {LABEL[s]}</button>
              )).flatMap((b) => [b, " "])}
              <button onClick={() => startTransition(() => removeTask(task.id))} aria-label={`Delete “${task.title}”`}>×</button>
            </li>))}
          </ul>
        </section>;
      })}
    </div>
  </>;
}
```

```ts
// file: taskboard/proxy.ts
import { NextResponse, type NextRequest } from "next/server";

export function proxy(request: NextRequest) {
  if (!request.cookies.has("session")) return NextResponse.redirect(new URL("/login", request.url));
}
export const config = { matcher: ["/"] };
```

### Deep & Tricky

**DT-35. A Server Function's response carries the re-rendered page (after `revalidatePath`), and the transition that called it does not finish until that payload has arrived — including content inside `<Suspense>` boundaries on the page. Here a move takes 1 s on the server, but in the run below the card is still pending at 1.0 s and 1.6 s and settled only by 2.2 s, because the revalidated page also re-runs the 0.8 s statistics query. Keep slow work out of pages that mutations revalidate, cache it, or revalidate more narrowly with tags.** *(Measured below; React documentation, "useOptimistic" and "useTransition"; Next.js documentation, "Updating Data — Revalidating")*

## Unit 15.3 — Building, running, and testing

**One-line summary:** The production build is the thing that is tested: type-checked by `next build`, run as a standalone server with environment variables for the database and the simulated delays, driven by a browser, and finally exercised by Playwright journeys against a fresh database.

```console
$ cd taskboard && DATABASE_PATH=:memory: NODE_OPTIONS=--disable-warning=ExperimentalWarning npx next build 2>&1 | grep -E "TypeScript|Route|[○ƒ◐] "
  Running TypeScript ...
  Finished TypeScript in 656ms ...
Route (app)
┌ ƒ /
├ ○ /_not-found
└ ○ /login
ƒ Proxy (Middleware)
○  (Static)   prerendered as static content
ƒ  (Dynamic)  server-rendered on demand
$ cd taskboard && cp -r .next/static .next/standalone/ch15/taskboard/.next/ && rm -f data.db*
$ cd taskboard && python3 ../../tools/page.py http://localhost:3100/ "eval:location.pathname" "fill:input[name=email]|ana@example.org" "fill:input[name=password]|correct horse" "click:button[value=register]" "wait:1500" "eval:[location.pathname, document.getElementById('stats').textContent]" "fill:input[name=title]|   " "click:text=Add" "wait:400" "eval:document.getElementById('title-error').textContent" "fill:input[name=title]|Write chapter 15" "click:text=Add" "wait:1200" "fill:input[name=title]|Review 100% of PRs" "click:text=Add" "wait:1200" "fill:input[name=title]|Ship it" "click:text=Add" "wait:1200" "eval:document.querySelector('.columns').innerText" "click:[aria-label='Move “Write chapter 15” to Done']" "wait:100" "eval:[document.querySelector('[data-task=\"1\"]').closest('section').querySelector('h2').textContent, document.querySelector('[data-task=\"1\"]').className]" "wait:900" "eval:document.querySelector('[data-task=\"1\"]').className" "wait:600" "eval:document.querySelector('[data-task=\"1\"]').className" "wait:600" "eval:[document.querySelector('[data-task=\"1\"]').className, document.getElementById('stats').textContent]" axe --serve "env NODE_OPTIONS=--disable-warning=ExperimentalWarning DATABASE_PATH=$PWD/data.db PORT=3100 MOVE_DELAY_MS=1000 STATS_DELAY_MS=800 node .next/standalone/ch15/taskboard/server.js"
=> "/login"
=> ["/", "0 of 0 done (0%)"]
=> "Give the task a title"
=> "To do (3)\nWrite chapter 15 → Doing → Done ×\nReview 100% of PRs → Doing → Done ×\nShip it → Doing → Done ×\nDoing (0)\nDone (0)"
=> ["Done (1)", "pending"]
=> "pending"
=> "pending"
=> ["", "1 of 3 done (33%)"]
axe: 0 violation(s)
```

Reading the run: an unauthenticated visit lands on `/login`; registration signs Ana in, and the statistics stream in; a blank title is rejected by the same `parseTitle` the unit tests check; three tasks are added. Moving "Write chapter 15" to *Done* updates the board within 100 ms — the card is already in the Done column, faded as `pending` — while the server takes its second; the card stays pending until the revalidated page, statistics included, arrives (DT-35). The axe audit of the finished board is clean.

Now the failure path. A second process — standing in for another browser tab — deletes "Ship it" while the board is open; then Ana tries to move it:

```console
$ cd taskboard && { (sleep 4; node --disable-warning=ExperimentalWarning -e 'new (require("node:sqlite").DatabaseSync)("data.db").exec("DELETE FROM tasks WHERE id = 3"); console.log("[another tab] deleted task 3")') & python3 ../../tools/page.py http://localhost:3100/login "fill:input[name=email]|ana@example.org" "fill:input[name=password]|correct horse" "click:button[value=login]" "wait:1500" "eval:document.querySelector('.columns').innerText" "wait:3000" "click:[aria-label='Move “Ship it” to Doing']" "wait:100" "eval:document.querySelector('.columns').innerText" "wait:2500" "eval:[document.getElementById('board-error').textContent, document.querySelector('.columns').innerText]" --serve "env NODE_OPTIONS=--disable-warning=ExperimentalWarning DATABASE_PATH=$PWD/data.db PORT=3100 MOVE_DELAY_MS=1000 STATS_DELAY_MS=800 node .next/standalone/ch15/taskboard/server.js"; wait; }
=> "To do (2)\nReview 100% of PRs → Doing → Done ×\nShip it → Doing → Done ×\nDoing (0)\nDone (1)\nWrite chapter 15 → To do → Doing ×"
[another tab] deleted task 3
=> "To do (1)\nReview 100% of PRs → Doing → Done ×\nDoing (1)\nShip it → To do → Done ×\nDone (1)\nWrite chapter 15 → To do → Doing ×"
=> ["Could not move “Ship it”: That task no longer exists", "To do (1)\nReview 100% of PRs → Doing → Done ×\nDoing (0)\nDone (1)\nWrite chapter 15 → To do → Doing ×"]
```

The move is shown at once (Doing has one card), the server finds no such task, and the transition ends: the optimistic state is dropped, the revalidated board no longer has the task, and the error is announced through the live region. No code had to "undo" anything.

```ts
// file: taskboard/playwright.config.ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "e2e",
  reporter: "list",
  use: { baseURL: "http://localhost:3200" },
  webServer: {
    // The production build, against a fresh database for every run.
    command: "rm -f e2e.db* && DATABASE_PATH=$PWD/e2e.db PORT=3200 NODE_OPTIONS=--disable-warning=ExperimentalWarning node .next/standalone/ch15/taskboard/server.js",
    url: "http://localhost:3200/login",
  },
});
```

```ts
// file: taskboard/e2e/board.spec.ts
import { test, expect, type Page } from "@playwright/test";

async function register(page: Page, email: string) {
  await page.goto("/");
  await expect(page).toHaveURL(/\/login$/);
  await page.getByLabel("Email").fill(email);
  await page.getByLabel("Password").fill("correct horse");
  await page.getByRole("button", { name: "Create account" }).click();
  await expect(page.getByRole("heading", { name: "Taskboard" })).toBeVisible();
  await expect(page.locator("#who")).toHaveText(email);
}

const column = (page: Page, name: string) => page.getByRole("region", { name: new RegExp(`^${name}`) });

test("a user adds, moves, finds, and deletes tasks", async ({ page }) => {
  await register(page, "ana@example.org");
  for (const title of ["Write chapter 15", "Review 100% of PRs", "Ship it"]) {
    await page.getByRole("textbox", { name: "New task" }).fill(title);
    await page.getByRole("button", { name: "Add" }).click();
    await expect(column(page, "To do").getByText(title)).toBeVisible();
  }
  await page.getByRole("button", { name: "Move “Ship it” to Done" }).click();
  await expect(column(page, "Done")).toContainText("Ship it");
  await expect(page.locator("#stats")).toHaveText("1 of 3 done (33%)");

  await page.getByRole("searchbox", { name: "Search" }).fill("%");             // a literal %, not a wildcard
  await page.getByRole("button", { name: "Search" }).click();
  await expect(page).toHaveURL(/\?q=%25$/);
  await expect(page.locator("[data-task]")).toHaveCount(1);

  await page.getByRole("button", { name: "Delete “Review 100% of PRs”" }).click();
  await expect(page.locator("[data-task]")).toHaveCount(0);
});

test("users never see each other's tasks", async ({ browser }) => {
  const ana = await (await browser.newContext()).newPage();
  const bo = await (await browser.newContext()).newPage();            // separate cookies
  await register(ana, "ana2@example.org");
  await ana.getByRole("textbox", { name: "New task" }).fill("Ana's private task");
  await ana.getByRole("button", { name: "Add" }).click();
  await expect(ana.locator("[data-task]")).toHaveCount(1);

  await register(bo, "bo@example.org");
  await expect(bo.locator("[data-task]")).toHaveCount(0);
  await expect(bo.locator("#stats")).toHaveText("0 of 0 done (0%)");
});
```

```console
$ cd taskboard && npx playwright test 2>&1 | grep -E "✓|✘|passed|failed" | sed -E "s/ \([0-9.]+m?s\)//"
  ✓  1 e2e/board.spec.ts:16:1 › a user adds, moves, finds, and deletes tasks
  ✓  2 e2e/board.spec.ts:36:1 › users never see each other's tasks
  2 passed
```

The first version of this test failed with a *strict mode violation*: `getByLabel("New task")` matched both the text field (labelled "New task") and the form (whose `aria-label` is also "New task"). Playwright refuses to guess; `getByRole("textbox", { name: "New task" })` says precisely which one.

### Pattern

**P-17. Test at the level where a rule lives: pure rules with unit tests (fast, many cases), components that own interaction with component tests, and the journeys that cross the network, the database, and the session with a few end-to-end tests on the production build.** *(Chapter 11, P-12; Playwright documentation, "Best Practices — Test user-visible behavior")* *Trade-off:* the end-to-end tests here need a build first (about ten seconds); run them in continuous integration and before releases, and the unit tests on every save.

## Capstone review checklist

Treat this as the code review a senior colleague would give the board before it goes to real users.

- **Security.** `signIn` has no rate limit; add one. Sessions are never cleaned up; add expiry cleanup and "sign out everywhere". Would you add a `Content-Security-Policy`? Which inline scripts would it break?
- **Data.** Two people editing the same task: add an `updated_at` column and reject stale moves (optimistic concurrency), showing "This task changed — reload".
- **Performance.** Apply DT-35: cache the statistics with `"use cache"` and a tag per user, invalidated by every mutation, and measure the pending time again.
- **UX and accessibility.** After deleting a task, where does keyboard focus go? (Today: to the `body`.) Move it to the next task or the column heading, and test that with the keyboard.
- **Internationalisation.** Extract the strings into a catalogue (Chapter 14) and add a second language, including plurals for "N of M done".
- **Operations.** Add the `/api/health` route from Chapter 13, structured logging for Server Function failures, and a database migration strategy for the `updated_at` change.

## Where to go next

You now know React from the element objects up and Next.js from the wire format up. The natural next steps: a real database with migrations (PostgreSQL with Drizzle or Prisma), an authentication library or provider for social sign-in and password resets, error monitoring and analytics, and — for the mobile platforms that the remaining books in this series cover — React Native, which reuses everything in Chapters 0–6 with native components instead of the DOM.

## Answers

This chapter has no prediction questions; its tests are the review checklist above. For DT-34, check your understanding by deleting the `.replace(…)` call in `listTasks`, rebuilding, and predicting the result of the end-to-end search test before running it (it should fail, finding three tasks instead of one).

# Appendix A — Index of Deep & Tricky entries and Patterns

Every *Deep & Tricky* entry (DT) and *Pattern* (P) in the book, in number order, with the unit where it is explained. Use it for revision: read each title and explain the entry before looking it up.

## Deep & Tricky

| # | Entry | Where |
| --- | --- | --- |
| DT-1 | `{count && <Badge />}` renders a literal `0` when `count` is 0, because `&&` returns its left operand when it is falsy and React renders numbers — a classic source of stray zeros in lists and layouts (and a crash in React Native, where text outside `<Text>` is not allowed). Use `count > 0 && …` or a ternary. | Unit 0.5 |
| DT-2 | In development, `<StrictMode>` intentionally calls component functions, initialisers of `useState`/`useMemo`/`useReducer`, and the updater functions passed to setters twice, and mounts every component twice (mount, simulated unmount, mount) to check that effects clean up after themselves. Logs, counters, or network requests that appear twice in development and once in production are this check working, not a bug in React — and code that breaks under it would also break when React re-renders, reuses, or restores components in ways StrictMode simulates. | Unit 0.6 |
| DT-3 | Keys identify components among their siblings, so a key that changes on every render — `key={Math.random()}`, or `key={crypto.randomUUID()}` computed during render — is worse than no key: every render destroys every item and creates new ones, losing their state, focus, and DOM, and making long lists slow. Keys must come from the data (database IDs), or be generated once when the item is created. | Unit 1.2 |
| DT-4 | React decides whether state changed with `Object.is(old, new)`, so mutating an object or array and setting it again is invisible: no render happens, and the mutation shows up later, at an unrelated render, as if by magic. State must be treated as immutable — replace it with a new object, array, or copy (`[...a, x]`, `{ ...o, k: v }`, `structuredClone`, or a library such as Immer). | Unit 1.3 |
| DT-5 | Reading or writing `ref.current` during rendering makes a component impure — its output depends on something React does not track — so concurrent rendering, StrictMode's double render, or a discarded render can show stale or doubled values; the render counter above counts *attempts*, and under StrictMode it would count two per update. Read and write refs in event handlers and effects; the one sanctioned exception is initialising a ref lazily (`if (ref.current === null) ref.current = new Thing()`). | Unit 2.2 |
| DT-6 | An effect's dependency list is not a choice of "when to run" — it must contain every reactive value (prop, state, or value derived from them) that the effect reads, and React compares each with `Object.is`; omitting one does not make the effect run less, it makes it use stale values. To run an effect less often, change the code so it does not need the value (updater functions, moving objects inside the effect, `useEffectEvent`), never lie in the list. | Unit 3.1 |
| DT-7 | Effects do not run on the server, and they run only after the component has rendered and committed — so fetching in an effect creates "waterfalls": the parent renders, then fetches, then renders a child, which then starts its own fetch. Every level adds a full round trip, and nothing starts until the JavaScript has loaded. Server components and route loaders (Chapter 12) start all fetches as early as possible, in parallel. | Unit 3.5 |
| DT-8 | `value={undefined}` and `value={null}` mean "uncontrolled", so `useState()` with no initial value, or data that arrives later from a fetch (`value={user?.name}`), makes an input uncontrolled at first and controlled afterwards — React warns, and the input may keep the old DOM value. Initialise controlled fields to `""` (and checkboxes to `false`). | Unit 4.1 |
| DT-9 | React resets a form after its action finishes, whether the action "succeeded" or returned an error — an action that returns validation errors therefore erases what the user typed. Return the submitted values with the errors and use them as the fields' `defaultValue` (as the project does), call `event.preventDefault()` in an `onSubmit` handler and start the action yourself, or make the fields controlled. | Unit 4.2 |
| DT-10 | Context has no selectors: a component that reads one field of a context value re-renders whenever *any* field changes. Splitting one context into several (theme, user, dispatch) or moving fast-changing state into an external store with selectors is the remedy; memoising the value object only removes the re-renders caused by the provider's own re-renders. | Unit 5.2 |
| DT-11 | `useSyncExternalStore` calls `getSnapshot` repeatedly — on every render and on every store change — and requires that it return the *same* value (by `Object.is`) as long as the store has not changed; a selector that builds a new object or array (`s => ({ a: s.a })`, `s => s.items.filter(...)`) therefore causes an infinite re-render loop. Return primitives or existing objects, memoise derived objects, or compare with shallow equality (Zustand's `useShallow`, Redux's `shallowEqual`). | Chapter 5 Project |
| DT-12 | The React Compiler can only memoise code that follows the rules of React — pure rendering, immutable props and state, hooks at the top level — and it silently skips (bails out of) components it cannot prove safe, such as those that mutate values created during render or read refs while rendering; such components keep working, unoptimised. Code that "worked" while breaking the rules can also change behaviour when compiled, because a mutation that used to be repeated on every render now happens only when inputs change. | Unit 6.2 |
| DT-13 | Windowing removes off-screen rows from the DOM, so the browser's find-in-page (Ctrl+F) cannot find them, screen readers cannot browse them, and their state (an open menu, a half-typed note) is destroyed when they scroll away. Expose the full count to assistive technology (`aria-rowcount`/`aria-setsize`), keep row state outside the rows, and prefer pagination or CSS `content-visibility: auto` — which skips rendering work while keeping the DOM — when the list is only hundreds of rows long. | Chapter 6 Project |
| DT-14 | Revalidation after actions is the router's caching strategy: after *every* action — including one that returns validation errors — the loaders of all matched routes run again, so a failed save of one field re-fetches the page's data. That keeps the UI consistent with the server at the cost of extra requests; a route's `shouldRevalidate` function can opt out when the action cannot have changed its data. | Unit 7.3 |
| DT-15 | With TanStack Query's default `staleTime` of 0, every cached query is stale as soon as it arrives, so it is refetched on every mount of a component that uses it and every time the window regains focus — correct but chatty. Set `staleTime` to how long data can reasonably be shown without checking (seconds for prices, minutes for a user profile), which is a product decision, not a technical one. | Unit 8.1 |
| DT-16 | If a query function uses the `AbortSignal` that TanStack Query passes in, the library cancels the request when the last component observing the query unmounts — which saves bandwidth, but also means a quick unmount–remount (a tab switch, StrictMode's double mount in development) aborts and restarts the request, as the log above shows. Without the signal, the first request is allowed to finish and is reused. | Chapter 8 Project |
| DT-17 | `e.target` is typed as `EventTarget` because the event may have been dispatched by any descendant — a click on a button can come from the `<svg>` icon inside it — while `e.currentTarget` is the element whose handler is running, and React's types give it the element's type. The same distinction is the source of event-delegation bugs at run time (Book 1, DT on `currentTarget`): the types are telling the truth. | Unit 9.1 |
| DT-18 | CSS Modules scope *class names*, not *styles*: a rule such as `.card p { … }` or `h2 { … }` in a module still applies to every matching element on the page, because only the class selector is renamed — element selectors, attribute selectors, and the cascade are untouched. A module that styles bare elements leaks exactly as a global stylesheet does. | Unit 10.1 |
| DT-19 | The order of class names in a `class` attribute has no effect on the cascade: when two rules of equal specificity set the same property, the one *later in the stylesheet* wins. Tailwind emits `p-2` before `p-4`, so `p-4` wins in both buttons. This is why a component cannot let callers "override" a utility by appending another one (`className={"p-4 " + props.className}` with `"p-2"`), and why libraries such as `tailwind-merge` exist: they remove the conflicting class instead. | Unit 10.2 |
| DT-20 | Tailwind finds class names by scanning source files as plain text, so a class must appear whole in the source: `` `bg-${color}-600` `` generates nothing, and the element is unstyled. Map props to complete class strings (`{ danger: "bg-red-600" }`) instead. | Unit 10.2 |
| DT-21 | `<input type="password">` has no ARIA role — deliberately, so that assistive technology does not treat its contents like ordinary text — so `getByRole("textbox", { name: "Password" })` cannot find it, and the error lists every role on the page without it. Query password fields with `getByLabelText`. | Unit 11.2 |
| DT-22 | Under Vitest's fake timers, Testing Library's asynchronous helpers can hang even when `user-event` is configured: React Testing Library's `asyncWrapper` drains pending work with `setTimeout(…, 0)` and advances fake timers only if a global named `jest` exists and `setTimeout` is a fake. Vitest does not define `jest`, so the zero-delay timeout never fires. `setup.js` assigns `globalThis.jest = vi`; the alternative is to avoid fake timers and wait for real ones with `findBy…`. | Unit 11.2 |
| DT-23 | Playwright's `expect(locator).toHaveText(…)` retries until the assertion passes (5 seconds by default), but `expect(await locator.textContent()).toBe(…)` reads the text *once* and fails if the page has not caught up: awaiting the value before the assertion throws away the retry. The same mistake in Testing Library is `getBy…` where `findBy…` was needed. | Unit 11.3 |
| DT-24 | Props passed from a Server Component to a Client Component cross a network boundary, so they must be serialisable by React: strings, numbers, plain objects and arrays, `Date`, `Map`, `Set`, promises, JSX, and Server Functions — but not ordinary functions, class instances, or symbols. Next.js reports an event handler passed across the boundary while *prerendering* the page, not while compiling it. | Unit 12.1 |
| DT-25 | Every Server Function is a public HTTP endpoint. Its ID is in the page (here in a hidden input, which lets the form work before JavaScript loads), and anyone who can load the page can call it with any arguments — `curl` added a note above without a browser or a form. Validate every argument (it is `unknown` input, whatever the TypeScript types say) and check the user's permissions inside the function, not only in the page that renders the form. The warning in the output is Next.js's cross-site request forgery check: it compares the `Origin` header with the host and rejects requests that come from other sites, as the second request shows (status 500, "Invalid Server Actions request" in the server log), but a request with no `Origin` at all, like the first, is let through — it protects users' browsers; it is not authentication. | Unit 12.2 |
| DT-26 | A `"use cache"` entry whose lifetime is too short is not put into the prerendered page: in Next.js 16.4, an `expire` under 5 minutes or a `stale` under 30 seconds makes the cached call behave like per-request data, and the build fails with "encountered uncached or runtime data during prerendering" unless the component is inside `<Suspense>`. Measured here: `{ stale: 29, … }` fails; `{ stale: 30, revalidate: 5, expire: 3600 }` prerenders; `{ …, expire: 299 }` fails. | Unit 12.4 |
| DT-27 | Store a *hash* of each session token, not the token: the token in the cookie is effectively a password, and anyone who can read the `sessions` table (a backup, a log, an SQL injection elsewhere) could otherwise sign in as every user. Because the token is 256 random bits, a fast hash (SHA-256) is enough here — slow hashing protects low-entropy secrets such as passwords, not random tokens. | Unit 13.2 |
| DT-28 | Sign-in must fail the same way, and in about the same time, for an unknown email and a wrong password — "Wrong email or password", with a hash comparison in both cases (against a dummy hash for unknown emails). A different message or a quicker failure tells an attacker which emails have accounts. | Unit 13.2 |
| DT-29 | Environment variables are not all read at run time. `NEXT_PUBLIC_…` variables are replaced by their values in the browser JavaScript at build time; and *any* variable read by a page that is prerendered (`/login` is static) is read once, at build time, and frozen into the HTML. Only dynamic code — the `/api/health` handler here, or dynamic pages — sees the run-time value. Building one image and promoting it from staging to production therefore needs runtime configuration for anything public: read it in dynamic server code and pass it down as props. | Unit 13.4 |
| DT-30 | Automated checkers report what they can decide from the markup, which is a minority of real accessibility failures. axe-core cannot know that a `<div>` has a click handler (React attaches listeners at the root, so not even the DOM shows it), and it accepts a placeholder as an input's accessible name — although placeholder text disappears when typing starts and is often low-contrast. "0 violations" means "no *detectable* violations"; keyboard and screen-reader testing remain necessary. | Unit 14.1 |
| DT-31 | A live region must exist in the DOM *before* its content changes: screen readers watch regions that are already present, so rendering `{message && <p role="status">{message}</p>}` — creating the region and its text at the same moment — is often not announced. Render the empty region from the start (as `main.jsx` does) and change only its text. | Unit 14.2 |
| DT-32 | Formatting dates or numbers with the *machine's* locale or time zone in server-rendered components produces different text on the server and in the browser, and React reports a hydration mismatch and re-renders the tree on the client (a visible flash, and the server's work wasted). Pass the locale and time zone explicitly, from the user's settings or the request — or render such values only on the client. | Unit 14.3 |
| DT-33 | `node:sqlite` returns each row as an object with a *null prototype* (`[Object: null prototype] { id: 1, … }`), and React refuses to serialise such objects from a Server Component to a Client Component — the first version of this board crashed with "Only plain objects, and a few built-ins, can be passed to Client Components … null prototypes are not supported" as soon as there was a task to show. Copy rows into plain objects at the data layer boundary (`listTasks` does), which is also where to choose exactly which columns leave the server. | Unit 15.1 |
| DT-34 | In SQL `LIKE`, `%` and `_` are wildcards, and parameters do not change that: a parameter stops injection (DT in Unit 13.1) but the *pattern* is still interpreted. Searching for `%` without escaping would match every task. `listTasks` escapes `\`, `%`, and `_` and declares `ESCAPE '\'`; the end-to-end test searches for `%` and expects exactly one task. | Unit 15.1 |
| DT-35 | A Server Function's response carries the re-rendered page (after `revalidatePath`), and the transition that called it does not finish until that payload has arrived — including content inside `<Suspense>` boundaries on the page. Here a move takes 1 s on the server, but in the run below the card is still pending at 1.0 s and 1.6 s and settled only by 2.2 s, because the revalidated page also re-runs the 0.8 s statistics query. Keep slow work out of pages that mutations revalidate, cache it, or revalidate more narrowly with tags. | Unit 15.2 |

## Patterns

| # | Pattern | Where |
| --- | --- | --- |
| P-1 | Keep the minimum state: store only what cannot be computed from props or other state, and derive everything else during rendering; lift state to the closest common parent of the components that need it; and do not copy props into state (it stops following the prop). | Unit 1.4 |
| P-2 | Extract a custom hook when the same combination of hooks and logic appears in two components, or when a component's logic deserves a name (`useCart`, `useOnlineStatus`); return the smallest useful interface (often a tuple or an object of named values and functions), and keep hooks free of JSX. | Unit 2.4 |
| P-3 | Before writing an effect, ask what it synchronises with; if the answer is "other state", compute during render (with `useMemo` if expensive); if it is "the user did something", put it in the event handler; if it is "a prop changed, reset everything", use a `key`. Keep effects for subscriptions, timers, DOM integrations, network synchronisation, and analytics. | Unit 3.3 |
| P-4 | Validate on the server always (it is the only validation that cannot be bypassed), and on the client for immediate feedback; return machine-readable, per-field errors (HTTP 422 with `{ errors: { field: message } }`), show them next to the fields with `aria-invalid` and `aria-describedby`, and never discard the user's input because of an error. | Unit 4.3 |
| P-5 | Keep state as local as possible: in a component, then lifted to the nearest common parent, then in context for rarely changing global values (theme, locale, the signed-in user), and in an external store only for frequently changing state shared across distant parts of the app; and keep server data out of all of these — it belongs in a server-state cache (Chapter 8). | Unit 5.3 |
| P-6 | Optimise in this order: measure (React DevTools Profiler, the Performance panel, Core Web Vitals); render less (keys, state placed lower in the tree, children passed as props, windowing); render cheaper (memoisation — preferably by the React Compiler); and render later (transitions and deferred values) — and measure again after each change. | Unit 6.3 |
| P-7 | Make the URL the source of truth for navigation state: every screen a user can reach should have a URL that recreates it, so the Back button, bookmarks, links, and reloads work; keep only ephemeral UI state (an open menu, a hover) in component state. | Unit 7.4 |
| P-8 | Keep server state in a server-state cache and nowhere else: no copying query results into `useState`, Redux, or context; derive what you need from the query data; and define query keys in one module (a "key factory") so that invalidations always match the queries they mean. | Unit 8.3 |
| P-9 | Let TypeScript infer what it can (state from initial values, generic parameters from props, return types of components), annotate boundaries explicitly (props, context values, reducer actions, API responses after validation), and never use `any` to silence an error — use `unknown` and narrow, or fix the type. | Unit 9.2 |
| P-10 | Keep a component's visual variants in one table (variant name → classes), choose from it by props, and let callers pass layout-only classes (margin, width, grid placement) through `className` — not colours or padding, which conflict with the variants (DT-19). | Unit 10.3 |
| P-11 | Query priority: `getByRole` (with a name) first, then `getByLabelText`, `getByPlaceholderText`, `getByText`, and `getByTestId` only as a last resort; a test that can only find an element by test ID is often reporting an accessibility problem. | Unit 11.2 |
| P-12 | The testing trophy: many fast component tests of behaviour, a smaller number of end-to-end tests for the journeys that make money or lose data (sign-up, checkout, saving work), unit tests for pure logic, and static checks (TypeScript, ESLint) underneath them all. | Unit 11.3 |
| P-13 | Push `"use client"` down to the leaves: keep pages, layouts, and data fetching in Server Components, and make only the interactive pieces (a button, a form, a chart) Client Components. Pass server-rendered content *into* Client Components as `children` rather than importing it there. | Unit 12.1 |
| P-14 | Decide per piece of data, not per page: static content in the shell; shared data that changes rarely behind `"use cache"` with a tag that writes invalidate; per-user or real-time data streamed behind `<Suspense>`. | Unit 12.4 |
| P-15 | A data access layer: put every query behind functions that take the current user and enforce ownership (`WHERE id = ? AND user_id = ?`), and never query user data from a component directly. Return 404 rather than 403 for another user's resource, so that IDs cannot be probed. | Unit 13.3 |
| P-16 | Prefer native elements that already implement an accessibility pattern — `<dialog>`, `<details>`, `<select>`, `<input type="date">`, the `popover` attribute — over hand-built ARIA widgets; when you must build one, follow the WAI-ARIA Authoring Practices pattern for it exactly, keyboard behaviour included. | Unit 14.2 |
| P-17 | Test at the level where a rule lives: pure rules with unit tests (fast, many cases), components that own interaction with component tests, and the journeys that cross the network, the database, and the session with a few end-to-end tests on the production build. | Unit 15.3 |

# Appendix B — Versions and Recent Features

Every example in this book was run with the versions below, installed in one shared `code/` folder. React and Next.js change quickly; the table of features lists the recent ones the book relies on, with where they are used, so you can tell which explanations depend on a version.

| Tool | Version used |
| --- | --- |
| Node.js | 22.23 |
| React and React DOM | 19.3.1 |
| Next.js (Turbopack) | 16.4.6 |
| Vite / `@vitejs/plugin-react` | 8.3.8 / 6.3.0 |
| React Compiler (`babel-plugin-react-compiler`) | 1.0.0 |
| TypeScript | 7.0.2 (the native compiler) |
| `tsx` (running JSX and TypeScript in Node.js) | 4.23.15 |
| React Router | 8.5.1 |
| TanStack Query | 5.107.1 |
| Zustand / Redux Toolkit | 5.1.0 / 2.14.0 |
| Tailwind CSS | 4.3.5 |
| Vitest / jsdom | 5.1.0 / 31.0.1 |
| Testing Library: React / user-event / jest-dom | 16.3.3 / 14.6.12 / 6.9.1 |
| Playwright (`@playwright/test` and the Python driver) | 1.63, Chromium 153 |
| axe-core | 4.14.1 |

| Feature | Introduced in | Where in this book |
| --- | --- | --- |
| `useActionState`, `useFormStatus`, `useOptimistic`, form `action` functions | React 19 | Chapter 4, Chapters 12–15 |
| `useEffectEvent` | React 19.2 | Chapter 3 |
| Resource preloading (`<link rel=preload>`) emitted during server rendering | React 19 | Chapter 1 |
| React Compiler (automatic memoisation) | React Compiler 1.0 | Chapter 6 |
| Server Components, Server Functions (`"use client"`, `"use server"`) | React 19, Next.js App Router | Chapters 12–15 |
| Cache Components: `"use cache"`, `cacheLife`, `cacheTag`, `updateTag`, partial prerendering | Next.js 16 | Chapter 12 |
| `proxy.js` (formerly `middleware.js`) | Next.js 16 | Chapters 13, 15 |
| Loaders, actions, and `HydrateFallback` in data routers | React Router 7+ | Chapter 7 |
| CSS-first configuration (`@theme`) and the Vite plugin | Tailwind CSS 4 | Chapter 10 |
| `node:sqlite` | Node.js 22 (experimental) | Chapters 13, 15 |

## Habits that changed with these versions

- Reach for a form `action` and `useActionState` before writing `onSubmit` handlers that juggle loading and error state.
- Let the React Compiler memoise; write `memo`, `useMemo`, and `useCallback` by hand only where measurement shows a need.
- Fetch on the server (Server Components, loaders) instead of in effects; use effects to synchronise with things outside React.
- Treat every Server Function as a public endpoint: validate, then authorise.
- Decide caching per piece of data (`"use cache"` with tags), not per page.
