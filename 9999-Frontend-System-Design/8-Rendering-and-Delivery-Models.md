<!-- TOC --><a name="rendering-delivery-models"></a>
# Rendering & Delivery models

<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [Rendering & Delivery models](#rendering-delivery-models)
   * [Introduction](#introduction)
   * [Prerequisite - The early days of MPAs leading to SPAs](#prerequisite-the-early-days-of-mpas-leading-to-spas)
      + [The Early Days: Multi-Page Applications (MPAs)](#the-early-days-multi-page-applications-mpas)
      + [The Solution: The Single Page Application (SPA)](#the-solution-the-single-page-application-spa)
      + [Pros and Cons of the SPA Model](#pros-and-cons-of-the-spa-model)
      + [The Connection to Delivery Models](#the-connection-to-delivery-models)
   * [1. CSR (Client-Side Rendering) - The Original SPA Way](#1-csr-client-side-rendering-the-original-spa-way)
   * [2. SSR + Hydration (Server-Side Rendering)](#2-ssr-hydration-server-side-rendering)
   * [3. SSG (Static Site Generation)](#3-ssg-static-site-generation)
   * [4. ISR (Incremental Static Regeneration)](#4-isr-incremental-static-regeneration)
   * [5. Server Components (RSC in React / App Router)](#5-server-components-rsc-in-react-app-router)
   * [Comprehensive Summary: When to Use What](#comprehensive-summary-when-to-use-what)
   * [The Trend: What is becoming more popular and why?](#the-trend-what-is-becoming-more-popular-and-why)
   * [Deep-dive into NextJS component delivery models](#deep-dive-into-nextjs-component-delivery-models)
      + [1. Client-Side Rendering (CSR)](#1-client-side-rendering-csr)
      + [2. Server-Side Rendering (SSR)](#2-server-side-rendering-ssr)
      + [3. Static Site Generation (SSG)](#3-static-site-generation-ssg)
      + [4. Incremental Static Regeneration (ISR)](#4-incremental-static-regeneration-isr)
      + [5. React Server Components (RSC)](#5-react-server-components-rsc)
      + [Combinations of Delivery Models (Hybrid Architecture)](#combinations-of-delivery-models-hybrid-architecture)
      + [Best Practices for Senior Engineers](#best-practices-for-senior-engineers)

<!-- TOC end -->

<!-- TOC --><a name="introduction"></a>
## Introduction

The core challenge of modern web development is balancing two things: getting pixels onto the screen as fast as possible **(First Contentful Paint)** and making those pixels interactive **(Time to Interactive)**, all while managing JavaScript bundle sizes.

Here is how the industry has tackled this over time:
1. CSR (Client-Side Rendering) - The Original SPA Way
2. SSR + Hydration (Server-Side Rendering)
3. SSG (Static Site Generation)
4. ISR (Incremental Static Regeneration)'
5. Server Components (RSC in React / App Router)

<!-- TOC --><a name="prerequisite-the-early-days-of-mpas-leading-to-spas"></a>
## Prerequisite - The early days of MPAs leading to SPAs

To fully understand the evolution of web delivery models (like CSR, SSR, and RSC), we have to look at the architectural shift that forced them to exist: **The Single Page Application (SPA).**

Here is a breakdown of what SPAs are, why the old way of building websites broke down, and how SPAs changed the internet forever.

<!-- TOC --><a name="the-early-days-multi-page-applications-mpas"></a>
### The Early Days: Multi-Page Applications (MPAs)

In the beginning, the web was built entirely on a **Multi-Page Application (MPA)** model. Every time a user clicked a link, the browser threw away the current page completely and asked the server for a brand new one.

**Why sending down the whole HTML wasn't sufficient:**
1.  **The "White Flash":** Every navigation resulted in the browser clearing the screen, leaving the user staring at a blank white page while waiting for the server to respond. 
2.  **Massive Redundancy:** If you clicked from the "Home" page to the "About" page, the server sent down the *entire* HTML file again. This included the header, the navigation bar, the footer, and the sidebar—which hadn't changed at all. Re-downloading and re-rendering identical UI was a massive waste of bandwidth and CPU.
3.  **Poor User Experience:** As the web transitioned from "document viewers" to "interactive applications" (like Gmail or Google Maps), users expected them to feel fast and seamless, exactly like desktop software. The clunky, page-reloading MPA model couldn't provide that native feel.

**ASCII Diagram: The MPA Flow**
```text
========================================================================
                      TRADITIONAL MPA (The Old Way)
========================================================================
Time ->
1. User clicks "Profile"
[ Browser ] ----------------- (Request: GET /profile) ----------------> [ Server ]
                                                                             |
      (Browser clears screen)                                                | (Server generates
      (User stares at white flash)                                           |  FULL HTML page)
                                                                             |
[ Browser ] <----------- (Response: <html>...Full Page...</html>) -----------+
      |
      v
Browser parses HTML, CSS, JS, and completely redraws the screen.
========================================================================
```

<!-- TOC --><a name="the-solution-the-single-page-application-spa"></a>
### The Solution: The Single Page Application (SPA)

The SPA was invented to solve the "White Flash" and make web apps feel like native desktop apps. 

**How it works:** 
The server sends exactly *one* HTML file (usually containing nothing but an empty `<div id="root"></div>`) along with a massive JavaScript application bundle. 

From that moment on, the browser **never asks the server for another HTML page**. When the user clicks "Profile", JavaScript intercepts the click, stops the browser from reloading, asks an API for raw JSON data, and updates the screen instantly.

**ASCII Diagram: The SPA Flow**
```text
========================================================================
                      MODERN SPA (The New Way)
========================================================================
Time ->
1. INITIAL LOAD (Happens once)
[ Browser ] ----------------- (Request: GET /) -----------------------> [ Server ]
[ Browser ] <----------- (Response: Empty HTML + 5MB JS App) -----------+

2. USER CLICKS "Profile" (Navigation)
[ Browser JS Engine ] 
      |
      |-- Intercepts click (No page reload!)
      |-- (Request: GET /api/user/123) ---------------------------------> [ API Server ]
      |                                                                        |
      |<--------- (Response: { "name": "Pushkar", "role": "Eng" }) ------------+
      |
      v
JS dynamically swaps out the middle of the page.
Header and Footer stay frozen. Transition is 0ms.
========================================================================
```

<!-- TOC --><a name="pros-and-cons-of-the-spa-model"></a>
### Pros and Cons of the SPA Model

**Pros:**
*   **Buttery Smooth Routing:** Because the browser never reloads, navigating between pages is virtually instantaneous. It feels like a native app.
*   **No UI Redundancy:** The header, footer, and sidebar are only rendered once. Only the specific data that changes is swapped out.
*   **Decoupled Backend:** The frontend becomes a completely independent app. The backend becomes a pure JSON API, meaning a company's iOS, Android, and Web apps can all use the exact same backend servers.

**Cons (The Catalyst for Modern Delivery Models):**
*   **Terrible Initial Load Times:** Because the app is driven by JavaScript, the browser has to download, parse, and execute a massive JS bundle before the user sees *anything*. This causes a prolonged blank screen on the very first visit.
*   **SEO Nightmares:** In the early SPA days, when a Google search crawler visited a site, it saw an empty `<div id="root"></div>` because crawlers couldn't execute heavy JavaScript. 
*   **Client-Side CPU Tax:** You are forcing the user's device (which might be an old, slow smartphone) to do all the heavy lifting of building the UI, draining their battery and causing sluggishness.

<!-- TOC --><a name="the-connection-to-delivery-models"></a>
### The Connection to Delivery Models

Understanding SPAs is the key to understanding modern delivery models. **The entire modern JavaScript ecosystem (SSR, SSG, RSC) is simply a series of attempts to fix the fatal flaws of the SPA (Initial Load Time & SEO) while keeping its greatest strength (Instant Routing).** 

*   **CSR** is just the raw SPA model.
*   **SSR/SSG** were invented to pre-paint the SPA on the server so users didn't have to wait for the JS bundle to see the UI.
*   **RSC** was invented because those JS bundles were still getting too large, and we needed a way to keep parts of the SPA on the server permanently.

<!-- TOC --><a name="1-csr-client-side-rendering-the-original-spa-way"></a>
## 1. CSR (Client-Side Rendering) - The Original SPA Way

**What it is:** 
The server sends a nearly empty HTML document containing a single `<div id="root"></div>` and a `<script>` tag pointing to a massive JavaScript bundle. The browser downloads the JS, parses it, and builds the entire UI dynamically on the client device.

**Why it exists:**
It enabled Single Page Applications (SPAs). Once the initial heavy load is done, navigating between pages is instant because the browser doesn't have to ask the server for new HTML; JavaScript just swaps out the DOM elements.

**ASCII Diagram:**
```text
========================================================================
                      CLIENT-SIDE RENDERING (CSR)
========================================================================

[ SERVER ]                                [ BROWSER ]
    |                                          |
    |-- 1. Returns empty HTML & JS Link ---->  | (Screen is blank white)
    |                                          |
    |                                          |-- 2. Downloads JS Bundle
    |                                          |-- 3. Executes JS
    |                                          |-- 4. Fetches API Data
    |                                          |-- 5. Builds DOM elements
    |                                          v
                                          [ UI RENDERS & IS INTERACTIVE ]
                                          (Time to see UI is very slow)
```

**Code Snippet:**
```javascript
// index.html (What the server sends)
<!DOCTYPE html>
<html>
  <body>
    <!-- The user sees a blank screen until JS fills this div -->
    <div id="root"></div> 
    <script src="/bundle.js"></script>
  </body>
</html>

// index.js (Runs entirely in the browser)
import { createRoot } from 'react-dom/client';
import App from './App';

const root = createRoot(document.getElementById('root'));
// React takes over and builds the entire DOM here
root.render(<App />);
```

<!-- TOC --><a name="2-ssr-hydration-server-side-rendering"></a>
## 2. SSR + Hydration (Server-Side Rendering)

**What it is:**
The server runs your React/Vue code *on the server* for every single request. It builds the full HTML string and sends it to the browser. The user sees the UI instantly. However, the UI is essentially a "dead" screenshot until the browser downloads the JS bundle and attaches event listeners (onClick, state) in a process called **Hydration**.

**Why it exists:**
CSR was terrible for SEO (crawlers saw empty divs) and terrible for users on slow phones (staring at white screens). SSR fixes the First Contentful Paint.

**ASCII Diagram:**
```text
========================================================================
                      SSR + HYDRATION
========================================================================

[ SERVER ]                                [ BROWSER ]
    |                                          |
    |-- 1. Fetches DB/API                      |
    |-- 2. Renders HTML string                 |
    |-- 3. Returns FULL HTML & JS Link ---->   | 
                                               |-- 4. [ UI PAINTS INSTANTLY! ]
                                               |      (But buttons don't work yet)
                                               |
                                               |-- 5. Downloads JS Bundle
                                               |-- 6. "Hydrates" the HTML
                                               v
                                          [ UI IS NOW INTERACTIVE ]
```

**Code Snippet (Next.js Pages Router):**
```javascript
// Runs on every request on the SERVER
export async function getServerSideProps() {
  const data = await fetchDatabase();
  return { props: { data } }; // Passes data to the component
}

// Runs on the server to generate HTML, then runs on the client to Hydrate
export default function Page({ data }) {
  return (
    <div>
      <h1>{data.title}</h1>
      <button onClick={() => alert('Hydrated!')}>Click Me</button>
    </div>
  );
}
```

<!-- TOC --><a name="3-ssg-static-site-generation"></a>
## 3. SSG (Static Site Generation)

**What it is:**
Instead of generating HTML on every single user request (like SSR), the framework generates the HTML files exactly *once* during the build process (when you run `npm run build`). These static files are pushed to a global CDN.

**Why it exists:**
It is incredibly fast and cheap. Serving a static file from a CDN takes milliseconds and requires zero backend server processing power.

**ASCII Diagram:**
```text
========================================================================
                      STATIC SITE GENERATION (SSG)
========================================================================

[ BUILD TIME (Developer's Machine / CI) ]
    |
    |-- Fetches all data
    |-- Compiles HTML files
    |-- Pushes to CDN 

[ RUNTIME (User visits site) ]
[ CDN Node in User's City ]               [ BROWSER ]
    |                                          |
    |-- Returns pre-built HTML ------------>   | 
                                               |-- [ UI PAINTS INSTANTLY ]
                                               |-- Downloads JS & Hydrates
```

**Code Snippet (Next.js):**
```javascript
// Runs EXACTLY ONCE during `npm run build`
export async function getStaticProps() {
  const data = await fetchCMSData();
  return { props: { data } };
}

export default function Blog({ data }) {
  return <article>{data.post}</article>;
}
```

<!-- TOC --><a name="4-isr-incremental-static-regeneration"></a>
## 4. ISR (Incremental Static Regeneration)

**What it is:**
SSG is great, but if your data changes, you have to rebuild and redeploy the entire site. ISR solves this. It serves static SSG pages, but if the cache is older than a specified time (e.g., 60 seconds), the server will automatically rebuild *just that specific page* in the background for the next visitor.

**Why it exists:**
To combine the instant performance of SSG with the dynamic, up-to-date data of SSR, without the massive infrastructure cost of rendering every single request.

**ASCII Diagram:**
```text
========================================================================
                 INCREMENTAL STATIC REGENERATION (ISR)
========================================================================

Visitor 1 (Time 0s): 
[ CDN ] -> (Returns Static HTML) -> Browser

Visitor 2 (Time 61s - Cache Expired):
[ CDN ] -> (Returns STALE Static HTML instantly to user) -> Browser
    |
    |-- (Triggers Background Rebuild)
    v
[ SERVER ] -> Regenerates HTML -> Updates CDN Cache

Visitor 3 (Time 62s):
[ CDN ] -> (Returns NEW Static HTML) -> Browser
```

**Code Snippet (Next.js):**
```javascript
export async function getStaticProps() {
  const data = await fetchLiveScores();
  return { 
    props: { data },
    revalidate: 60 // Rebuild this page in the background at most once every 60s
  };
}
```

<!-- TOC --><a name="5-server-components-rsc-in-react-app-router"></a>
## 5. Server Components (RSC in React / App Router)

**What it is:**
The newest paradigm. Unlike traditional SSR (where the whole page renders on the server *and* hydrates on the client), React Server Components **never send their JavaScript to the browser**. They render on the server, fetch databases directly, and send a special UI format to the browser. Only specific "Client Components" (marked with `"use client"`) send JS to the browser for interactivity.

**Why it exists:**
Hydration is mathematically expensive. As apps grew, JS bundles got massive. RSC allows you to keep heavy dependencies (like markdown parsers or database drivers) entirely on the server, drastically reducing the JS sent to the browser.

**ASCII Diagram:**
```text
========================================================================
                      REACT SERVER COMPONENTS (RSC)
========================================================================

[ SERVER ]                                      [ BROWSER ]
    |
    |-- <ServerComponent> (Direct DB access, 
    |    imports heavy libraries. NO JS bundle 
    |    generated for this!)
    |
    |-- <ClientComponent> (Contains state/onClick.
    |    Generates a JS bundle.)
    |
    |-- Streams custom UI payload ------------> |
                                                |-- Paints UI
                                                |-- Hydrates ONLY the 
                                                |   <ClientComponent> parts
```

**Code Snippet (Next.js App Router):**
```javascript
// page.js (SERVER COMPONENT by default)
// This code never reaches the browser. You can talk to the DB securely.
import db from './db';
import InteractiveButton from './InteractiveButton';

export default async function Page() {
  const users = await db.query('SELECT * FROM users'); 
  
  return (
    <div>
      {users.map(u => <h1>{u.name}</h1>)}
      <InteractiveButton /> 
    </div>
  );
}

// InteractiveButton.js (CLIENT COMPONENT)
// Only this small file is sent to the browser.
"use client"; 
import { useState } from 'react';

export default function InteractiveButton() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

<!-- TOC --><a name="comprehensive-summary-when-to-use-what"></a>
## Comprehensive Summary: When to Use What

| Paradigm | Best For | Do NOT Use When (Terrible Choice) |
| :--- | :--- | :--- |
| **CSR** | Highly interactive, state-heavy dashboards behind a login screen (e.g., Figma, Spotify web, complex admin panels). | You need SEO, or the site is public-facing and performance/First Contentful Paint is critical. |
| **SSR** | Highly dynamic public pages where data changes every second and SEO matters (e.g., Twitter feed, Stock market ticker). | You have low server budget/infrastructure, or the data rarely changes. SSR is expensive to scale. |
| **SSG** | Marketing pages, blogs, documentation, portfolios. | You have millions of pages (build times will take hours) or data that changes constantly (e.g., a live sports site). |
| **ISR** | Large e-commerce sites (millions of product pages), news websites, large blogs. | You need absolutely guaranteed real-time data for every single request (e.g., banking ledger). |
| **RSC** | Modern full-stack apps. Anything where you want to minimize JS bundle size while mixing static content with highly interactive islands. | You are building a pure SPA with no backend, or migrating a massive legacy CSR codebase (the migration is incredibly difficult). |

<!-- TOC --><a name="the-trend-what-is-becoming-more-popular-and-why"></a>
## The Trend: What is becoming more popular and why?

The web is aggressively moving toward **Server Components (RSC)** and **Hybrid Architectures** (powered by frameworks like Next.js App Router, Nuxt 3, and SvelteKit).

**The "Why" boils down to the Hydration Bottleneck.**
For years, the industry leaned heavily on SSR. However, developers realized that SSR had a massive flaw: the "Uncanny Valley." The server would paint the HTML instantly, but the browser would lock up for 2-3 seconds downloading and executing a 5MB JavaScript bundle just to attach `onClick` handlers (hydration). During this time, the user clicks buttons and nothing happens.

**Server Components solve the ultimate contradiction:**
We want the backend power and instant HTML delivery of SSR, but we want the tiny JavaScript footprint of a simple static site. RSC allows developers to seamlessly weave server-only code (zero JS sent to client) with client-only code (interactivity) in the same file tree. This drastically shrinks bundle sizes, improves SEO, and provides the best possible Core Web Vitals (specifically Largest Contentful Paint and Interaction to Next Paint).

<!-- TOC --><a name="deep-dive-into-nextjs-component-delivery-models"></a>
## Deep-dive into NextJS component delivery models

Here is a deep-dive into the component delivery and rendering models in Next.js, tailored for senior-level system design and performance optimization. 

We will focus primarily on the modern **App Router (Next.js 13+)**, as it represents the current standard, though the architectural concepts apply broadly.

<!-- TOC --><a name="1-client-side-rendering-csr"></a>
### 1. Client-Side Rendering (CSR)

**How it works:**
The server sends a bare-bones HTML document and a JavaScript bundle. The browser downloads the JS, executes it, fetches the necessary data from an API, and finally builds the DOM. 

**When to use:**
Highly interactive, state-heavy dashboards behind an authentication wall where SEO is irrelevant, and the user expects an app-like experience (e.g., an internal admin tool or a rich text editor).

```text
================================================================================
                              CLIENT-SIDE RENDERING (CSR)
================================================================================

[ BROWSER ]                                           [ SERVER / API ]
    |
    |-- 1. GET /dashboard -----------------------------> |
    |                                                    |
    |<-- 2. Returns blank HTML + JS Bundle --------------|
    |
    |-- 3. Browser executes JS (Screen is blank)
    |
    |-- 4. JS calls API (fetch data) ------------------> |
    |                                                    |
    |<-- 5. Returns JSON data { "user": "Pushkar" } -----|
    |
    |-- 6. React builds DOM
    v
[ UI PAINTS & IS INTERACTIVE ]
(Very slow First Contentful Paint)
================================================================================
```

**Code Snippet (App Router):**
In the App Router, you must explicitly opt into CSR using the `"use client"` directive.

```javascript
"use client"; // Opts into the client boundary

import { useState, useEffect } from 'react';

export default function ClientDashboard() {
  const [data, setData] = useState(null);
  const [isLoading, setLoading] = useState(true);

  useEffect(() => {
    // Data is fetched strictly on the client device
    fetch('https://api.example.com/user')
      .then((res) => res.json())
      .then((data) => {
        setData(data);
        setLoading(false);
      });
  }, []);

  if (isLoading) return <p>Loading UI on Client...</p>;
  return <div>Welcome, {data.name}</div>;
}
```

<!-- TOC --><a name="2-server-side-rendering-ssr"></a>
### 2. Server-Side Rendering (SSR)

**How it works:**
The HTML is generated on the server for *every single request*. The server fetches the data, compiles the React components into an HTML string, and sends it to the browser. The browser paints the UI instantly, then downloads the JS bundle to "hydrate" the page (attach event listeners).

**When to use:**
Highly dynamic pages where data changes constantly, and SEO is absolutely critical (e.g., a real-time stock ticker, a customized news feed).

```text
================================================================================
                              SERVER-SIDE RENDERING (SSR)
================================================================================

[ BROWSER ]                                           [ NEXT.JS SERVER ]
    |                                                        |
    |-- 1. GET /live-feed ---------------------------------> |
                                                             |-- 2. Server fetches DB/API
                                                             |-- 3. Renders HTML string
    |<-- 4. Returns Fully Populated HTML --------------------|
    |
    |-- 5. [ UI PAINTS INSTANTLY ] (But buttons are dead)
    |
    |-- 6. Downloads JS Bundle
    |-- 7. React Hydrates the DOM
    v
[ UI IS FULLY INTERACTIVE ]
================================================================================
```

**Code Snippet (App Router):**
In the App Router, Server Components do this automatically if you tell the fetch request *not* to cache the data.

```javascript
// By default, this is a Server Component.
// The 'no-store' cache directive forces SSR on every request.

export default async function LiveFeed() {
  const res = await fetch('https://api.example.com/stocks', {
    cache: 'no-store' // Forces dynamic server-rendering
  });
  const stocks = await res.json();

  return (
    <main>
      <h1>Live Market Data</h1>
      <ul>
        {stocks.map(stock => <li key={stock.id}>{stock.ticker}: {stock.price}</li>)}
      </ul>
    </main>
  );
}
```

<!-- TOC --><a name="3-static-site-generation-ssg"></a>
### 3. Static Site Generation (SSG)

**How it works:**
The HTML is generated exactly *once* during the build process (`next build`). The pre-rendered HTML and JSON data are pushed to a CDN. At runtime, the server does no computation; it just serves static files instantly.

**When to use:**
Marketing pages, blogs, documentation, or e-commerce product catalogs where data does not change frequently. 

```text
================================================================================
                         STATIC SITE GENERATION (SSG)
================================================================================

[ BUILD TIME (CI/CD Pipeline) ]
  |-- Next.js fetches all required data
  |-- Renders all pages into static HTML
  |-- Pushes to Global CDN

[ RUNTIME (User visits site) ]
[ BROWSER ]                                           [ CDN EDGE NODE ]
    |                                                        |
    |-- 1. GET /about-us ----------------------------------> |
    |<-- 2. Returns pre-built HTML instantly ----------------|
    |
    |-- 3. [ UI PAINTS INSTANTLY ]
    |-- 4. Downloads minimal JS & Hydrates
    v
[ UI IS FULLY INTERACTIVE ]
(Zero server backend compute utilized)
================================================================================
```

**Code Snippet (App Router):**
In the App Router, `fetch` caches data by default, meaning standard Server Components are statically generated at build time unless specified otherwise.

```javascript
// This component will be pre-rendered into static HTML at build time.

export default async function AboutPage() {
  // 'force-cache' is the default behavior in Next.js 13+
  const res = await fetch('https://api.example.com/company-info', {
    cache: 'force-cache' 
  });
  const info = await res.json();

  return (
    <article>
      <h1>About Us</h1>
      <p>{info.missionStatement}</p>
    </article>
  );
}
```

<!-- TOC --><a name="4-incremental-static-regeneration-isr"></a>
### 4. Incremental Static Regeneration (ISR)

**How it works:**
ISR gives you the performance of SSG with the flexibility of SSR. Pages are statically generated at build time. However, you assign a "revalidation" timeframe (e.g., 60 seconds). If a user requests the page after 60 seconds, they receive the *stale* static page instantly, but it triggers the Next.js server to rebuild that specific page in the background. The *next* user gets the newly rebuilt page.

**When to use:**
Massive websites (like a 100,000-item e-commerce store) where rebuilding the whole site takes hours, but prices/inventory need to update reasonably fast without crushing the database with SSR requests.

```text
================================================================================
                   INCREMENTAL STATIC REGENERATION (ISR)
================================================================================

[ BROWSER 1 (Time: 0s) ]
    |-- GET /product/shoes ---> [ CDN ] ---> Returns Static HTML V1

[ BROWSER 2 (Time: 65s) ] Cache has expired!
    |-- GET /product/shoes ---> [ CDN ] ---> Returns Static HTML V1 (STALE)
                                  |
                                  |-- (Background trigger sent to Server)
                                  v
                            [ NEXT.JS SERVER ]
                            - Fetches new DB data
                            - Recompiles HTML V2
                            - Updates the CDN cache secretly

[ BROWSER 3 (Time: 68s) ]
    |-- GET /product/shoes ---> [ CDN ] ---> Returns Static HTML V2 (FRESH)
================================================================================
```

**Code Snippet (App Router):**
Use the `next.revalidate` option in your fetch call.

```javascript
export default async function ProductCatalog() {
  // Fetch data, cache it, but revalidate every 60 seconds
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 60 } 
  });
  const products = await res.json();

  return (
    <div className="grid">
      {products.map(p => <Card key={p.id} title={p.name} />)}
    </div>
  );
}
```

<!-- TOC --><a name="5-react-server-components-rsc"></a>
### 5. React Server Components (RSC)

**How it works:**
RSC is a fundamental paradigm shift. These components run **exclusively on the server** and never send any JavaScript to the browser. Instead of compiling to HTML, they compile to a special binary JSON format (React Flight payload). The browser receives this payload and seamlessly merges it with the Client Components. 

**When to use:**
This is the default architectural pattern for Next.js App Router. Use RSC for anything that fetches data, accesses the file system, or imports heavy utility libraries (like Markdown parsers or date formatting libraries) that you don't want bloating the client bundle.

```text
================================================================================
                         REACT SERVER COMPONENTS (RSC)
================================================================================

[ NEXT.JS SERVER ]
    |
    |-- <ServerLayout> (Reads DB, huge dependencies. ZERO JS sent to client)
    |     |
    |     |-- <ClientSidebar> ("use client", has onClick state)
    |     |
    |     |-- <ServerArticle> (Fetches markdown. ZERO JS sent to client)
    |
    |-- Serializes Server Components into React Flight Payload
    |-- Sends Payload + Client Component JS Bundles to Browser
    v

[ BROWSER ]
    |-- React parses the Flight Payload
    |-- Paints the UI
    |-- Hydrates ONLY the <ClientSidebar>
    v
(Massive reduction in JavaScript bundle size. Extremely fast TTI).
================================================================================
```

**Code Snippet (App Router):**
You can securely write direct database queries inside an RSC because the code never leaves the server.

```javascript
// Server Component (Default in App Router)
import db from '@/lib/db'; 
import InteractiveLikeButton from './LikeButton'; // A Client Component

export default async function BlogPost({ id }) {
  // Securely query the database directly. 
  // No API route needed!
  const post = await db.query('SELECT * FROM posts WHERE id = ?', [id]);

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      
      {/* We pass data to the client component, which handles interactivity */}
      <InteractiveLikeButton initialLikes={post.likes} postId={id} />
    </article>
  );
}
```

<!-- TOC --><a name="combinations-of-delivery-models-hybrid-architecture"></a>
### Combinations of Delivery Models (Hybrid Architecture)

Modern Next.js applications do not use just one model; they combine them on a per-route or even per-component basis.

**1. The "Islands" Combination (RSC + CSR)**
You render the heavy, static shell of the page (navbars, footers, article bodies) using RSC to keep the JS bundle near zero. You then sprinkle "islands" of interactivity using Client Components (CSR) only where needed (e.g., an "Add to Cart" button, an image carousel).

**2. The E-Commerce Combination (ISR + SSR + CSR)**
*   **Homepage (ISR):** Cached and revalidated every 5 minutes for extreme speed.
*   **Checkout Flow (SSR):** Absolutely no caching. Dynamic SSR ensures the user sees accurate shipping rates and tax calculations.
*   **Search Bar (CSR):** Client-side fetching (like Algolia or a debounce API call) handles the rapid keystrokes without full page reloads.

<!-- TOC --><a name="best-practices-for-senior-engineers"></a>
### Best Practices for Senior Engineers

To architect a highly optimized Next.js App Router application, adhere to these rules:

1.  **Push Client Boundaries Down the Tree:**
    Never put `"use client"` at the top of your page hierarchy (like `layout.js` or `page.js`). Doing so forces every child component to also become a Client Component, destroying the benefits of RSC. Keep Client Components as tiny "leaves" at the edges of your component tree.

2.  **Pass Server Components as Props (Children):**
    If a Client Component needs to wrap a Server Component (e.g., a Context Provider or a draggable layout wrapper), pass the Server Component via the `children` prop. This prevents the Server Component from being absorbed into the client bundle.

3.  **Leverage Streaming and Suspense:**
    Don't let one slow API call block your entire SSR page from rendering. Wrap slow Server Components in React `<Suspense>`. Next.js will stream the fast parts of the HTML immediately, and then stream the slow data in chunks as it resolves.
    ```javascript
    export default function Dashboard() {
      return (
        <main>
          <FastHeader />
          <Suspense fallback={<SkeletonLoader />}>
            <SlowDatabaseQueryComponent />
          </Suspense>
        </main>
      );
    }
    ```

4.  **Colocate Data Fetching:**
    In the older Pages Router, you had to fetch all data at the top level (`getServerSideProps`) and drill it down. In the App Router, **fetch data exactly where you need it**. Next.js automatically dedupes identical `fetch` requests in the same render pass. If three nested Server Components request the same user profile, the network call is only executed once.
