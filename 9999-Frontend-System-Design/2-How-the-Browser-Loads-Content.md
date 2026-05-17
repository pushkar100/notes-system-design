<!-- TOC --><a name="how-the-browser-loads-content"></a>
# How the browser loads content

<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [How the browser loads content](#how-the-browser-loads-content)
   * [1. HTML parsing and the DOM](#1-html-parsing-and-the-dom)
      + [The Need and The Outcome](#the-need-and-the-outcome)
      + [How HTML Parsing Works](#how-html-parsing-works)
      + [How Parsing is BLOCKED!](#how-parsing-is-blocked)
      + [Strategies to Solve HTML Parsing Blockers](#strategies-to-solve-html-parsing-blockers)
         - [Strategy 1: The `defer` Attribute (Most Common & Recommended)](#strategy-1-the-defer-attribute-most-common-recommended)
         - [Strategy 2: The `async` Attribute (For Independent Scripts)](#strategy-2-the-async-attribute-for-independent-scripts)
         - [Strategy 3: Place Scripts at the Bottom of the Body (Legacy Approach)](#strategy-3-place-scripts-at-the-bottom-of-the-body-legacy-approach)
         - [Strategy 4: Resource Hints (`preload`)](#strategy-4-resource-hints-preload)
      + [How Modern Frameworks Solve This](#how-modern-frameworks-solve-this)
   * [2. CSS parsing and the CSSOM](#2-css-parsing-and-the-cssom)
      + [The Need and The Outcome](#the-need-and-the-outcome-1)
      + [How CSS Parsing Works](#how-css-parsing-works)
      + [How Parsing is BLOCKED!](#how-parsing-is-blocked-1)
      + [Strategies to Solve CSS Blocker Issues](#strategies-to-solve-css-blocker-issues)
         - [Strategy 1: Extract and Inline "Critical CSS" (Most Effective)](#strategy-1-extract-and-inline-critical-css-most-effective)
         - [Strategy 2: Asynchronous Loading for Non-Critical CSS](#strategy-2-asynchronous-loading-for-non-critical-css)
         - [Strategy 3: Avoid `@import` in CSS Files](#strategy-3-avoid-import-in-css-files)
      + [How Modern Frameworks Solve This](#how-modern-frameworks-solve-this-1)
   * [3. JavaScript fetching and execution](#3-javascript-fetching-and-execution)
      + [The Need and The Outcome](#the-need-and-the-outcome-2)
      + [How Fetching and Parsing Works](#how-fetching-and-parsing-works)
      + [How JS Execution Can Be BLOCKED](#how-js-execution-can-be-blocked)
      + [How JS Execution Blocks Other Things](#how-js-execution-blocks-other-things)
      + [How to Solve Blocking Issues](#how-to-solve-blocking-issues)
      + [How Modern Frameworks Solve This](#how-modern-frameworks-solve-this-2)
   * [Summary of different blocking scenarios](#summary-of-different-blocking-scenarios)
      + [1. JavaScript Blocks HTML Parsing](#1-javascript-blocks-html-parsing)
      + [2. CSS Blocks Rendering (and JavaScript Execution)](#2-css-blocks-rendering-and-javascript-execution)
      + [3. Long JS Tasks Block the Main Thread (UI Responsiveness)](#3-long-js-tasks-block-the-main-thread-ui-responsiveness)
      + [Summary Table of Web Performance Blockers](#summary-table-of-web-performance-blockers)
   * [Progressive rendering](#progressive-rendering)
      + [How Progressive Rendering Works](#how-progressive-rendering-works)
      + [The Catch: Render-Blocking Resources](#the-catch-render-blocking-resources)
      + [How Developers Optimize This](#how-developers-optimize-this)

<!-- TOC end -->

<!-- TOC --><a name="1-html-parsing-and-the-dom"></a>
## 1. HTML parsing and the DOM

Here is a deep dive into how browsers translate raw network data into the interactive pages we see, the bottlenecks that occur along the way, and how to resolve them.

<!-- TOC --><a name="the-need-and-the-outcome"></a>
### The Need and The Outcome

When a web server sends a webpage to a browser, it doesn't send visual elements; it sends a stream of raw bytes (0s and 1s) representing HTML characters. The browser cannot display raw text directly, nor can JavaScript interact with a string of text. 

**The Need:** The browser must convert this raw text string into a structured, easily queryable, and manipulable format. 
**The Outcome:** The Document Object Model (DOM). The DOM is an internal tree structure—an API—that represents the entire document as a hierarchy of nodes and objects. This tree is what the browser actually renders to the screen and what JavaScript manipulates.

<!-- TOC --><a name="how-html-parsing-works"></a>
### How HTML Parsing Works

The journey from raw network bytes to a fully constructed DOM tree happens in a strict, multi-step pipeline.

```text
=============================================================================
                     THE DOM GENERATION PIPELINE
=============================================================================

[ 1. Bytes ]      01010011 01101000 01100101 ... (Raw Network Data)
      |
      v  (Encoding - e.g., UTF-8)
[ 2. Characters ] <html><head><title>My Page</title>...
      |
      v  (Tokenization - W3C HTML5 Standard)
[ 3. Tokens ]     StartTag:html, StartTag:head, StartTag:title, Text:"My Page"...
      |
      v  (Lexing/Node Creation)
[ 4. Nodes ]      HTMLHtmlElement, HTMLHeadElement, HTMLTitleElement...
      |
      v  (Tree Construction)
[ 5. DOM Tree ]   The final hierarchical object structure.

=============================================================================
```

**Visualizing the Tree Construction:**

Given a simple HTML snippet:
```html
<html>
  <head>
    <title>Hello</title>
  </head>
  <body>
    <h1>Title</h1>
    <p>Some text</p>
  </body>
</html>
```

The parser continuously consumes tokens and builds this tree structure in memory:

```text
                           [ Document ]
                                |
                           [ html ]
                          /        \
                         /          \
                  [ head ]          [ body ]
                     |               /    \
                     |              /      \
                 [ title ]      [ h1 ]    [ p ]
                     |            |         |
                  "Hello"      "Title"  "Some text"
```

<!-- TOC --><a name="how-parsing-is-blocked"></a>
### How Parsing is BLOCKED!

HTML parsing is **synchronous and sequential**. The parser reads the document from top to bottom. The biggest bottleneck in modern web development is that **JavaScript is parser-blocking**.

When the HTML parser encounters a `<script>` tag, it absolutely must pause. Why? Because JavaScript has the power to alter the DOM (e.g., using `document.write()` or `element.appendChild()`). The browser cannot safely continue building the DOM until it downloads and executes that script, just in case the script changes what the DOM is supposed to look like.

**The Blocking Timeline (The Problem):**

```text
SCENARIO: <script src="heavy-logic.js"></script> encountered in the <head>

HTML Parsing:  [========|       PARSER BLOCKED       |===============>
                        |                            |
Network Fetch:          [====== heavy-logic.js =====]|
                        |                            |
JS Execution:                                        [=======>
                                                     |
                                                     (HTML parsing resumes)
```

**Code Example of the Blocker:**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Blocked Page</title>
  <!-- THE BLOCKER: The browser stops parsing HTML here. -->
  <script src="https://example.com/massive-script.js"></script>
</head>
<body>
  <!-- The user stares at a blank white screen because the parser 
       hasn't reached this h1 tag yet. -->
  <h1>Welcome to my site</h1>
</body>
</html>
```

<!-- TOC --><a name="strategies-to-solve-html-parsing-blockers"></a>
### Strategies to Solve HTML Parsing Blockers

To prevent users from staring at blank screens, we must remove JavaScript from the critical rendering path.

<!-- TOC --><a name="strategy-1-the-defer-attribute-most-common-recommended"></a>
#### Strategy 1: The `defer` Attribute (Most Common & Recommended)
Adding `defer` tells the browser: *"Download this script in the background while you continue parsing the HTML. Wait to execute it until the HTML is fully parsed."*

```html
<head>
  <!-- Non-blocking! Downloads in parallel, executes at the end. -->
  <script src="app-logic.js" defer></script>
</head>
```

**The `defer` Timeline:**
```text
HTML Parsing:  [=====================================================>
Network Fetch:          [====== app-logic.js ======]
JS Execution:                                                        [=======>
                                                             (Executes after DOM finishes)
```

<!-- TOC --><a name="strategy-2-the-async-attribute-for-independent-scripts"></a>
#### Strategy 2: The `async` Attribute (For Independent Scripts)
Adding `async` tells the browser: *"Download this script in the background. As soon as it finishes downloading, pause the HTML parser and execute it immediately."* This is best for scripts that don't depend on the DOM or other scripts, like Google Analytics.

```html
<head>
  <!-- Non-blocking fetch, but blocks parser during execution. -->
  <script src="analytics.js" async></script>
</head>
```

**The `async` Timeline:**
```text
HTML Parsing:  [=================|  BLOCKED  |=======================>
Network Fetch:       [====== analytics.js ===]
JS Execution:                                [=======>
```

<!-- TOC --><a name="strategy-3-place-scripts-at-the-bottom-of-the-body-legacy-approach"></a>
#### Strategy 3: Place Scripts at the Bottom of the Body (Legacy Approach)
Before `async` and `defer` were widely supported, developers simply moved all `<script>` tags to the very end of the document, just before the closing `</body>` tag. The parser would build the whole visual page before it ever encountered the scripts.

```html
<body>
  <h1>Welcome to my site</h1>
  <p>Content is visible immediately.</p>
  
  <!-- Parser hits this last, so the UI is already painted -->
  <script src="app-logic.js"></script>
</body>
```

<!-- TOC --><a name="strategy-4-resource-hints-preload"></a>
#### Strategy 4: Resource Hints (`preload`)
If a script is critical for the initial load but resides at the bottom of the page, the browser discovers it very late. You can tell the browser to fetch it immediately in the `<head>` without executing it.

```html
<head>
  <link rel="preload" href="critical-app.js" as="script">
</head>
```

<!-- TOC --><a name="how-modern-frameworks-solve-this"></a>
### How Modern Frameworks Solve This

Modern libraries like React (via Next.js) or Vue (via Nuxt.js) largely abstract this problem away through two main architectural patterns:

1.  **Automated Script Injection:** Frameworks automatically bundle your JavaScript and inject it into the final HTML document with the `defer` attribute applied by default. You rarely manually write `<script>` tags.
2.  **Server-Side Rendering (SSR) & Static Site Generation (SSG):** Instead of sending a nearly empty HTML file and relying on a massive JavaScript bundle to build the DOM on the client (which leads to terrible blocking and blank screens), the framework executes the initial JavaScript on the server. The server sends a fully formed, populated HTML document. The browser parses and displays the UI instantly, and then a deferred script runs in the background to attach event listeners (a process called "hydration").

<!-- TOC --><a name="2-css-parsing-and-the-cssom"></a>
## 2. CSS parsing and the CSSOM

<!-- TOC --><a name="the-need-and-the-outcome-1"></a>
### The Need and The Outcome

When a browser builds the DOM from HTML, it only understands the structure of the page. It knows there is a `<p>` tag inside a `<div>`, but it has no idea what color the text should be, how large the font is, or where the element sits on the screen. 

**The Need:** The browser must parse the CSS rules (from external stylesheets, `<style>` tags, and inline styles) and resolve how they cascade and apply to specific HTML elements.
**The Outcome:** The CSS Object Model (CSSOM). Just like the DOM, the CSSOM is a tree structure. It contains all the styling information, mapped out hierarchically, so the browser can calculate the final computed styles for every node before painting them to the screen.

<!-- TOC --><a name="how-css-parsing-works"></a>
### How CSS Parsing Works

The process of building the CSSOM looks almost identical to building the DOM. The browser takes the raw CSS bytes and pushes them through a conversion pipeline.

```text
=============================================================================
                     THE CSSOM GENERATION PIPELINE
=============================================================================

[ 1. Bytes ]      01100010 01101111 01100100 ... (Raw Network Data)
      |
      v  (Encoding)
[ 2. Characters ] body { font-size: 16px; } p { color: blue; } ...
      |
      v  (Tokenization)
[ 3. Tokens ]     Selector:body, Property:font-size, Value:16px ...
      |
      v  (Node Creation)
[ 4. Nodes ]      bodyNode(font-size:16px), pNode(color:blue) ...
      |
      v  (Tree Construction)
[ 5. CSSOM ]      The final cascading style tree.

=============================================================================
```

**Visualizing the Tree Construction (The Cascade):**

CSS stands for *Cascading* Style Sheets. The CSSOM must be a tree because styles cascade down. If you apply a font size to the `body`, every child node inherits it unless explicitly overridden.

Given this CSS:
```css
body { font-size: 16px; }
p { font-weight: bold; }
p span { color: red; }
```

The browser builds this CSSOM tree in memory:

```text
                           [ CSSOM Tree ]
                                 |
                          [ body: node ]
                        (font-size: 16px)
                                 |
                                 |
                           [ p: node ]
                      (font-size: 16px) <-- Inherited
                      (font-weight: bold)
                                 |
                                 |
                         [ span: node ]
                      (font-size: 16px) <-- Inherited
                      (font-weight: bold) <-- Inherited
                      (color: red)
```

<!-- TOC --><a name="how-parsing-is-blocked-1"></a>
### How Parsing is BLOCKED!

This is a critical nuance in web performance: **CSS does not directly block HTML parsing, but it DOES block Rendering and JavaScript execution (which in turn blocks HTML parsing).**

1.  **Render-Blocking:** The browser will *never* paint the page to the screen until both the DOM and the CSSOM are fully built. If it did, you would see an ugly, unstyled page that suddenly snaps into place (a Flash of Unstyled Content, or FOUC). 
2.  **Script-Blocking:** If the HTML parser hits a `<script>` tag, the browser must execute it. However, JavaScript can query CSS (e.g., `element.style.color`). Therefore, the browser forces the JavaScript engine to wait until the CSSOM is completely downloaded and built. Because the JavaScript is paused, the HTML parser is also paused.

**The Blocking Timeline:**

```text
SCENARIO: A stylesheet is followed by a script in the <head>

HTML Parsing:  [====|   WAITING FOR JS TO FINISH   |================>
                    |                              |
Network (CSS):      [====== styles.css ======]     |
                    |                        |     |
CSSOM Building:                              [====]|
                                                   |
JS Execution:                                      [=======>
                                                           (HTML resumes)
```

**Code Example of the Blocker:**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Blocked CSS Example</title>
  
  <!-- 1. Browser starts downloading this massive CSS file -->
  <link rel="stylesheet" href="massive-theme.css">
  
  <!-- 2. The parser hits this script. It pauses HTML parsing.
          It also CANNOT run this script until 'massive-theme.css' 
          is completely downloaded and the CSSOM is built. -->
  <script src="app.js"></script>
</head>
<body>
  <!-- 3. The user stares at a blank screen for seconds. -->
  <h1>Welcome</h1>
</body>
</html>
```

<!-- TOC --><a name="strategies-to-solve-css-blocker-issues"></a>
### Strategies to Solve CSS Blocker Issues

To get the page painted to the screen as fast as possible, you must optimize how CSS is delivered.

<!-- TOC --><a name="strategy-1-extract-and-inline-critical-css-most-effective"></a>
#### Strategy 1: Extract and Inline "Critical CSS" (Most Effective)
Identify the exact CSS needed to style the "above-the-fold" content (what the user sees immediately without scrolling). Remove it from the external stylesheet and inject it directly into a `<style>` tag in the HTML `<head>`.

```html
<head>
  <!-- 1. Inline Critical CSS. No network request needed. 
          Browser builds CSSOM for this instantly and paints the hero section. -->
  <style>
    body { margin: 0; font-family: sans-serif; }
    .hero { background: #333; color: #fff; padding: 2rem; }
    h1 { font-size: 2rem; }
  </style>

  <!-- 2. Defer non-critical CSS (See Strategy 2) -->
</head>
<body>
  <div class="hero"><h1>Instantly Visible!</h1></div>
</body>
```

<!-- TOC --><a name="strategy-2-asynchronous-loading-for-non-critical-css"></a>
#### Strategy 2: Asynchronous Loading for Non-Critical CSS
For styles that style the footer, modals, or below-the-fold content, you can trick the browser into downloading the CSS asynchronously without blocking the render.

```html
<head>
  <!-- The media="print" tells the browser: "This CSS is for printing, 
       so don't block the screen render waiting for it." 
       Once it finishes downloading (onload), we flip it to media="all" 
       so it applies to the screen. -->
  <link rel="stylesheet" 
        href="non-critical-footer-styles.css" 
        media="print" 
        onload="this.media='all'">
        
  <!-- Fallback for users with JavaScript disabled -->
  <noscript>
    <link rel="stylesheet" href="non-critical-footer-styles.css">
  </noscript>
</head>
```

<!-- TOC --><a name="strategy-3-avoid-import-in-css-files"></a>
#### Strategy 3: Avoid `@import` in CSS Files
Never use `@import` inside a CSS file to load another CSS file. It creates a serialized network chain. The browser cannot even start downloading the second file until it finishes downloading and parsing the first one.

**BAD (Serialized Waterfall):**
```css
/* inside main.css */
@import url("typography.css");
@import url("layout.css");
```

**GOOD (Parallel Fetching):**
```html
<!-- inside index.html -->
<link rel="stylesheet" href="typography.css">
<link rel="stylesheet" href="layout.css">
<link rel="stylesheet" href="main.css">
```

<!-- TOC --><a name="how-modern-frameworks-solve-this-1"></a>
### How Modern Frameworks Solve This

Writing custom scripts to extract Critical CSS manually is incredibly difficult. Modern frameworks handle this automatically during the build process:

1.  **Next.js / Nuxt (Component-Level CSS):** When you build a page in these frameworks, the build tool (like Webpack or Vite) analyzes exactly which components are used on that specific page. It automatically strips out unused CSS and injects the necessary critical styles directly into the `<head>` of the server-rendered HTML. 
2.  **CSS-in-JS (e.g., Styled Components):** Libraries like Styled Components track which components are rendered during the Server-Side Render (SSR) pass. They collect all the generated styles and inject them as a single inline `<style>` tag, completely eliminating render-blocking network requests for the initial paint.
3.  **Utility-First CSS (Tailwind CSS):** Tailwind's compiler scans your HTML/JS files for class names. It generates a single, incredibly small CSS file containing *only* the classes you actually used. Because the final CSS file is often under 10kb, the CSSOM is built in fractions of a millisecond, making render-blocking a non-issue.

<!-- TOC --><a name="3-javascript-fetching-and-execution"></a>
## 3. JavaScript fetching and execution

Here is a deep dive into how browsers fetch, parse, and execute JavaScript, why it creates massive performance bottlenecks, and how to unblock it.

<!-- TOC --><a name="the-need-and-the-outcome-2"></a>
### The Need and The Outcome

While HTML builds the structure and CSS paints the visuals, a webpage without JavaScript is static. 

*   **The Need:** We need a way to add logic, handle user interactions (clicks, typing), manage application state, and fetch new data from servers without reloading the entire page.
*   **The Outcome:** The browser's JavaScript engine (like Google Chrome's V8) downloads the code, translates it into machine-readable instructions, and executes it. The result is a dynamic page where the DOM and CSSOM can be manipulated on the fly, creating interactive web applications.

<!-- TOC --><a name="how-fetching-and-parsing-works"></a>
### How Fetching and Parsing Works

JavaScript does not run immediately upon download. It goes through a complex, multi-stage pipeline inside the browser's JS engine.

1.  **Network Fetch:** The browser's network thread downloads the `.js` file as a stream of bytes.
2.  **Lexical Analysis (Tokenization):** The raw text is broken down into meaningful chunks called tokens (e.g., keywords, variables, operators).
3.  **Syntax Analysis (Parsing):** The tokens are arranged into an Abstract Syntax Tree (AST), which maps out the logical structure of the code.
4.  **Compilation & Execution (JIT):** Modern engines use Just-In-Time (JIT) compilation. An interpreter quickly turns the AST into unoptimized "Bytecode" to start running immediately. Meanwhile, a background compiler watches for frequently used functions and upgrades them into highly optimized Machine Code.

**ASCII Visualization of the JS Engine Pipeline:**

```text
========================================================================
                     JAVASCRIPT PARSING PIPELINE
========================================================================

[ Network ]     bytes -> const x = 10;
                    |
[ Lexer ]           Tokens: [const] [x] [=] [10] [;]
                    |
[ Parser ]          Abstract Syntax Tree (AST)
                         (VariableDeclaration)
                              /         \
                         (Identifier:x) (Literal:10)
                    |
[ Interpreter ]     Generates base Bytecode ---> (Execution Starts)
                    |                              ^
[ JIT Compiler ]    Monitors execution             |
                    Optimizes hot code into Native Machine Code
                    
========================================================================
```

<!-- TOC --><a name="how-js-execution-can-be-blocked"></a>
### How JS Execution Can Be BLOCKED

A major quirk of the browser is that **CSS blocks JavaScript execution**. 

Because JavaScript has the power to ask the browser for the layout or styling of an element (e.g., `window.getComputedStyle(element)`), the browser must guarantee that the styles are accurate before letting the JS run. Therefore, if the browser is downloading a stylesheet, it will pause JS execution until the CSS Object Model (CSSOM) is fully built.

**Sample Code Highlighting the Problem:**

```html
<head>
  <!-- 1. Starts downloading massive CSS -->
  <link rel="stylesheet" href="massive-theme.css">
  
  <!-- 2. Browser downloads this script quickly, but REFUSES to run it
          until massive-theme.css is fully parsed. -->
  <script src="app.js"></script>
</head>
```

**ASCII Visualization of CSS Blocking JS:**

```text
Network (CSS):    [=============== massive-theme.css ===============]
CSSOM Building:                                                     [====]
JS Fetch (app):   [=== app.js ===]
JS Execution:                    |----- WAITING FOR CSSOM ----------|    [======>
```

<!-- TOC --><a name="how-js-execution-blocks-other-things"></a>
### How JS Execution Blocks Other Things

This is the most critical performance bottleneck on the web: **The browser's Main Thread is single-threaded**.

The Main Thread handles parsing HTML, calculating CSS, painting pixels to the screen, AND executing JavaScript. It can only do one thing at a time. If you write a long-running JavaScript function (a "Long Task"), everything else stops. HTML parsing halts, animations freeze, and user clicks are ignored.

**Sample Code Highlighting the Problem:**

```javascript
document.getElementById('btn').addEventListener('click', () => {
  // We want to show a loading spinner
  document.getElementById('spinner').style.display = 'block';

  // PROBLEM: A massive synchronous loop blocks the thread for 3 seconds.
  const start = Date.now();
  while (Date.now() - start < 3000) {
    // heavy math calculation
  }
  
  // The user NEVER saw the spinner. The thread was too busy to paint it.
  document.getElementById('spinner').style.display = 'none';
});
```

**ASCII Visualization of the Main Thread Traffic Jam:**

```text
User Action:      *Clicks Button*                                *UI Finally Updates*
                  |                                              |
Main Thread:      [==== JS EVENT HANDLER (Long Task) ===][ PAINT ]
                  |                                              |
                  <--------- 3 Second Total Freeze -------------->
```

<!-- TOC --><a name="how-to-solve-blocking-issues"></a>
### How to Solve Blocking Issues

Here are the primary strategies to unblock the main thread and the parsing pipeline.

**Strategy 1: Network & Parsing Unblocking (`defer`)**
To stop JS from blocking the HTML parser, always use `defer`. It forces the script to download in the background and guarantees it won't execute until the DOM is fully built.

```html
<!-- The browser parses HTML continuously while this downloads -->
<script src="app.js" defer></script>
```

**Strategy 2: Yielding the Main Thread (Chunking)**
If you must run heavy logic, break it into smaller pieces using `setTimeout`. This allows the browser to briefly pause your JS, paint any pending UI updates, and then resume.

```javascript
function processDataInChunks(data) {
  // Process just a little bit of data
  let chunk = data.splice(0, 100); 
  process(chunk);

  if (data.length > 0) {
    // SOLUTION: Push the rest to the back of the queue.
    // The browser uses this micro-gap to update the screen.
    setTimeout(() => processDataInChunks(data), 0);
  }
}
```

**Strategy 3: Web Workers (True Multithreading)**
For actual heavy lifting (complex math, massive data parsing), move the JavaScript entirely off the Main Thread into a Web Worker.

```javascript
// main.js
const worker = new Worker('heavy-math.js');
worker.postMessage({ start: true }); // Main thread remains 100% free

// heavy-math.js (Runs in a separate CPU core)
self.onmessage = function() {
  let result = doMassiveCalculation(); // Does not block the UI
  self.postMessage(result);
}
```

<!-- TOC --><a name="how-modern-frameworks-solve-this-2"></a>
### How Modern Frameworks Solve This

Modern libraries (React, Vue, Next.js) take these concepts and automate them so you don't have to rewire your codebase manually:

*   **Automated Bundling & Deferring:** Framework bundlers (Webpack/Vite) automatically inject your compiled scripts at the bottom of the HTML or add `defer` attributes, ensuring HTML parsing is never blocked.
*   **React Server Components (RSC):** Next.js and React 19 allow you to run heavy data-fetching and logic components exclusively on the server. The server sends pure, pre-rendered HTML to the browser, drastically reducing the amount of JavaScript the client has to fetch, parse, and execute.
*   **Concurrent Rendering (`useTransition`):** React natively implements thread-yielding. If you mark a state update as a "transition," React will slice the heavy rendering work into tiny chunks under the hood. If the user clicks a button mid-render, React immediately pauses the JavaScript render, handles the click, paints the UI, and resumes the render later.

<!-- TOC --><a name="summary-of-different-blocking-scenarios"></a>
## Summary of different blocking scenarios

Here is a summary of the critical blocking scenarios that occur during page load and execution.

<!-- TOC --><a name="1-javascript-blocks-html-parsing"></a>
### 1. JavaScript Blocks HTML Parsing

When the browser's HTML parser encounters a synchronous `<script>` tag, it must immediately pause building the DOM. It waits for the script to download over the network and execute completely before it resumes parsing the rest of the HTML document.

```text
===================================================================
                SCENARIO 1: JS BLOCKS HTML PARSER
===================================================================

HTML Parser:   [======] (PAUSED)                       [==========>
Network:               [--- Fetch script.js ---]
JS Execution:                                  [=====]

Result: The user stares at a blank or partially loaded screen.
===================================================================
```

<!-- TOC --><a name="2-css-blocks-rendering-and-javascript-execution"></a>
### 2. CSS Blocks Rendering (and JavaScript Execution)

CSS is render-blocking. The browser refuses to paint any pixels to the screen until the CSS Object Model (CSSOM) is fully built to avoid showing unstyled text. Furthermore, if a script is found after a stylesheet, the browser will pause the script's execution (and therefore HTML parsing) until the stylesheet finishes downloading, just in case the script needs to ask for CSS properties.

```text
===================================================================
          SCENARIO 2: CSS BLOCKS RENDERING & JS EXECUTION
===================================================================

Network (CSS): [------- Fetch styles.css -------]
CSSOM Build:                                     [==]
Network (JS):  [-- Fetch app.js --]
JS Execution:                     (WAITING)          [=====]
Screen Paint:                                               [PAINT]

Result: The entire pipeline is bottlenecked by the CSS download.
===================================================================
```

<!-- TOC --><a name="3-long-js-tasks-block-the-main-thread-ui-responsiveness"></a>
### 3. Long JS Tasks Block the Main Thread (UI Responsiveness)

The browser has a single Main Thread that handles parsing, painting, and executing JavaScript. If a JavaScript function takes too long to run (a "Long Task"), it monopolizes the thread. The browser is physically unable to paint UI updates, run animations, or respond to user clicks until the script finishes.

```text
===================================================================
            SCENARIO 3: LONG JS TASKS BLOCK MAIN THREAD
===================================================================

User Action:   *Click!*                                   *Screen Updates*
                  |                                              |
Main Thread:   [======== HEAVY JS LOOP (Long Task) ========][ PAINT ]
                  |                                              |
                  <----------- UI Freeze / INP ---------------->

Result: The page feels frozen, sluggish, and unresponsive.
===================================================================
```

<!-- TOC --><a name="summary-table-of-web-performance-blockers"></a>
### Summary Table of Web Performance Blockers

| Blocker Element | What Gets Blocked? | Why Does It Happen? | Primary Solution |
| :--- | :--- | :--- | :--- |
| **Synchronous `<script>`** | HTML Parsing | JS might modify the DOM (e.g., `document.write`), so the parser must wait. | Use `defer` or `async` attributes on the script tag. |
| **External `<link rel="stylesheet">`** | Visual Rendering | Browser waits for all styles to prevent a Flash of Unstyled Content (FOUC). | Inline Critical CSS; load non-critical CSS asynchronously. |
| **Pending CSSOM Build** | JavaScript Execution | JS might query layout/styles, so the engine waits for the CSSOM to be accurate. | Optimize CSS delivery; ensure CSS loads before JS in the `<head>`. |
| **Heavy JavaScript Logic (Long Tasks)** | Main Thread (UI Updates & Interactions) | The single thread can only do one thing at a time. A heavy loop stops the paint cycle. | Break tasks into chunks with `setTimeout` or offload to Web Workers. |

## Progressive rendering

**Q: Can and does the browser render the page while the HTML is still being parsed?**

**Yes**, modern browsers absolutely render the page while the HTML is still being parsed. This process is known as **progressive rendering**.

If browsers waited for the entire HTML document (and all its linked resources) to finish downloading and parsing before painting anything to the screen, users would be staring at blank white screens for a noticeable amount of time, especially on slower network connections.

Here is a breakdown of how this process works and the exceptions that can interrupt it.

### How Progressive Rendering Works

Browsers process web pages in chunks. As data streams in over the network, the browser's rendering engine gets to work immediately:

1. **Building the DOM (Document Object Model):** As soon as the browser receives HTML tokens, it starts constructing the DOM tree.
2. **Building the CSSOM (CSS Object Model):** Simultaneously, it parses any CSS it encounters to figure out how elements should look.
3. **Constructing the Render Tree:** The browser combines the DOM and CSSOM into a "Render Tree," which only includes the nodes needed to display the page (e.g., elements with `display: none` are left out).
4. **Layout:** The browser calculates the exact size and position of every visible element on the page.
5. **Paint:** Finally, the browser paints the pixels to the screen.

Because this happens continuously, you will often see a web page load top-to-bottom. The text and layout at the top of an article might be fully visible and readable while the browser is still actively downloading and parsing the HTML for the footer.

### The Catch: Render-Blocking Resources

While the browser *wants* to render progressively, certain resources can force it to halt parsing or rendering. These are called **render-blocking resources**.

* **CSS:** By default, CSS is treated as a render-blocking resource. The browser will not paint anything to the screen until it has constructed the CSSOM. It does this to prevent a "Flash of Unstyled Content" (FOUC), where you briefly see a messy, plain-text version of the site before the layout snaps into place.
* **Synchronous JavaScript:** If the HTML parser encounters a standard `<script>` tag, it must immediately pause parsing the HTML, download the JavaScript file, and execute it before it can continue. This is because JavaScript has the power to alter the DOM and CSSOM (e.g., using `document.write()`), so the browser has to wait to see what the script does.

### How Developers Optimize This

To keep progressive rendering running smoothly, web developers use a few tricks:

* **Putting CSS in the `<head>`:** This ensures the browser discovers styles as early as possible.
* **Using `async` or `defer` on scripts:** Adding these attributes to a `<script>` tag tells the browser, "Keep parsing the HTML in the background while this script downloads."
* **Putting scripts at the bottom:** Before `async` and `defer` were widely supported, developers traditionally placed `<script>` tags at the very end of the `<body>` so that all the HTML above it could be parsed and rendered first.
