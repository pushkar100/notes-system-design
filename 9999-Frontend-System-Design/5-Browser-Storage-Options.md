<!-- TOC --><a name="browser-storage-options"></a>
# Browser Storage Options

<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [Browser Storage Options](#browser-storage-options)
   * [Introduction](#introduction)
      + [Explicit Storage (Developer Controlled)](#explicit-storage-developer-controlled)
      + [Implicit Storage (Browser Managed)](#implicit-storage-browser-managed)
   * [LocalStorage](#localstorage)
      + [Introduction to LocalStorage](#introduction-to-localstorage)
      + [Browser Support](#browser-support)
      + [Pros and Cons](#pros-and-cons)
      + [When to Use (Practical Use Cases)](#when-to-use-practical-use-cases)
      + [When NOT to Use](#when-not-to-use)
      + [Security Risks & TTL](#security-risks-ttl)
   * [SessionStorage](#sessionstorage)
      + [Introduction to SessionStorage](#introduction-to-sessionstorage)
      + [Browser Support](#browser-support-1)
      + [Pros and Cons ](#pros-and-cons-1)
      + [When to Use (Practical Use Cases)](#when-to-use-practical-use-cases-1)
      + [When NOT to Use](#when-not-to-use-1)
      + [Security Risks & TTL](#security-risks-ttl-1)
   * [Cookies](#cookies)
      + [Introduction to Cookies](#introduction-to-cookies)
      + [Browser Support](#browser-support-2)
      + [Pros and Cons](#pros-and-cons-2)
      + [When to Use (Practical Use Cases)](#when-to-use-practical-use-cases-2)
      + [When NOT to Use](#when-not-to-use-2)
      + [Security Risks & TTL](#security-risks-ttl-2)
   * [IndexedDB](#indexeddb)
      + [Introduction to IndexedDB](#introduction-to-indexeddb)
      + [Browser Support](#browser-support-3)
      + [Pros and Cons (The Problem it Solves)](#pros-and-cons-the-problem-it-solves)
      + [When to Use (Practical Use Cases)](#when-to-use-practical-use-cases-3)
      + [When NOT to Use](#when-not-to-use-3)
      + [Security Risks & TTL](#security-risks-ttl-3)
   * [Cache API](#cache-api)
      + [Introduction to the Cache API](#introduction-to-the-cache-api)
      + [Browser Support](#browser-support-4)
      + [Pros and Cons ](#pros-and-cons-3)
      + [When to Use (Practical Use Cases)](#when-to-use-practical-use-cases-4)
      + [When NOT to Use](#when-not-to-use-4)
      + [Security Risks & TTL](#security-risks-ttl-4)
   * [HTTP Cache](#http-cache)
      + [Introduction to the HTTP Cache](#introduction-to-the-http-cache)
      + [Browser Support](#browser-support-5)
      + [Pros and Cons](#pros-and-cons-4)
      + [When to Use (Practical Use Cases)](#when-to-use-practical-use-cases-5)
      + [When NOT to Use](#when-not-to-use-5)
      + [Security Risks & TTL](#security-risks-ttl-5)
   * [Bfcache](#bfcache)
      + [Introduction to Bfcache (Back/Forward Cache)](#introduction-to-bfcache-backforward-cache)
      + [Browser Support](#browser-support-6)
      + [Pros and Cons ](#pros-and-cons-5)
      + [When to Use (Optimize for it)](#when-to-use-optimize-for-it)
      + [When NOT to Use (Prevent it)](#when-not-to-use-prevent-it)
      + [Security Risks & TTL](#security-risks-ttl-6)
   * [Origin Private File System (OPFS)](#origin-private-file-system-opfs)
      + [Introduction to Origin Private File System (OPFS)](#introduction-to-origin-private-file-system-opfs)
      + [Browser Support](#browser-support-7)
      + [Pros and Cons ](#pros-and-cons-6)
      + [When to Use (Practical Use Cases)](#when-to-use-practical-use-cases-6)
      + [When NOT to Use](#when-not-to-use-6)
      + [Security Risks & TTL](#security-risks-ttl-7)

<!-- TOC end -->

<!-- TOC --><a name="introduction"></a>
## Introduction

Here is a rapid breakdown of all modern browser storage mechanisms.

<!-- TOC --><a name="explicit-storage-developer-controlled"></a>
### Explicit Storage (Developer Controlled)

*   **1. LocalStorage:** Stores simple string key-value pairs (~5MB limit). Data persists indefinitely until explicitly cleared by the user or code. Synchronous and blocks the main thread.
*   **2. SessionStorage:** Identical to LocalStorage, but data is automatically destroyed the moment the specific browser tab or window is closed.
*   **3. Cookies:** Tiny text files (~4KB limit) primarily used for authentication. They are automatically attached and sent with every HTTP request to the server.
*   **4. IndexedDB:** A powerful, asynchronous NoSQL database for storing large amounts of structured data, blobs, and files without blocking the main thread. 
*   **5. Cache API:** Specifically designed to store network Request/Response pairs. It is the backbone of Service Workers, allowing websites to function offline.
*   **6. Origin Private File System (OPFS):** A highly optimized, origin-specific virtual file system that allows web apps (like video editors or IDEs) to read and write files with near-native performance.

<!-- TOC --><a name="implicit-storage-browser-managed"></a>
### Implicit Storage (Browser Managed)

*   **1. HTTP Cache (Disk/Memory Cache):** The browser automatically saves static assets (images, CSS, JS files) based on server response headers to avoid downloading them again on future visits.
*   **2. Bfcache (Back/Forward Cache):** The browser takes a complete, frozen memory snapshot of an entire webpage (including JavaScript state). When a user clicks the back or forward button, the page restores instantly without reloading.

<!-- TOC --><a name="localstorage"></a>
## LocalStorage

<!-- TOC --><a name="introduction-to-localstorage"></a>
### Introduction to LocalStorage

**What it is:**
LocalStorage is a web storage API that allows JavaScript to store data as key-value pairs directly in the user's web browser. Unlike SessionStorage, data in LocalStorage has no expiration time; it persists even after the browser window or tab is closed.

**Why it exists:**
Before HTML5, the only way to store data on the client side was using Cookies. Cookies are small (4KB) and are sent back and forth to the server with every single HTTP request, wasting bandwidth. LocalStorage was created to provide a much larger storage capacity (~5MB) that sits entirely on the client, never touching the network unless you explicitly write code to send it.

**How it works (ASCII Diagram):**

```text
+----------------------------------------------------+
|                   BROWSER WINDOW                   |
|                                                    |
|  [ Web Page (JS) ] <---+                           |
|      |                 | (Read/Write)              |
|      v                 |                           |
|  +----------------------------------------------+  |
|  |             window.localStorage              |  |
|  |----------------------------------------------|  |
|  | Key              | Value (Strings ONLY)      |  |
|  |------------------|---------------------------|  |
|  | "theme"          | "dark"                    |  |
|  | "userSettings"   | "{\"fontSize\": 14}"      |  |
|  | "draft_post"     | "Hello world..."          |  |
|  +----------------------------------------------+  |
+----------------------------------------------------+
```

**How to use it (Code Snippets):**

Because LocalStorage only stores strings, you must serialize complex objects.

```javascript
// 1. Storing data
localStorage.setItem('theme', 'dark');

// Storing objects (requires serialization)
const settings = { layout: 'grid', volume: 80 };
localStorage.setItem('appSettings', JSON.stringify(settings));

// 2. Retrieving data
const currentTheme = localStorage.getItem('theme'); // Returns "dark"

// Retrieving objects (requires parsing)
const savedSettings = JSON.parse(localStorage.getItem('appSettings'));

// 3. Removing a specific item
localStorage.removeItem('theme');

// 4. Clearing the entire storage for this domain
localStorage.clear();
```

<!-- TOC --><a name="browser-support"></a>
### Browser Support

LocalStorage is universally supported. It was introduced with HTML5 and works across all modern browsers (Chrome, Firefox, Safari, Edge) and even legacy browsers like IE8. 

<!-- TOC --><a name="pros-and-cons"></a>
### Pros and Cons

**Pros:**
*   **Simple API:** `setItem` and `getItem` are incredibly easy to implement.
*   **Capacity:** Offers significantly more storage (~5MB per domain) compared to Cookies (4KB).
*   **Bandwidth Saver:** Data is stored strictly on the client and is not automatically sent to the server with HTTP requests.
*   **Persistence:** Survives browser restarts and system reboots.

**Cons:**
*   **Synchronous & Blocking:** LocalStorage operations run synchronously on the Main Thread. Reading/writing large amounts of data can cause UI freezes (ruining the Interaction to Next Paint metric).
*   **String Only:** Requires `JSON.stringify` and `JSON.parse` for objects, which adds CPU overhead.
*   **Capacity Limit:** 5MB is not enough for large datasets, files, or blobs.

<!-- TOC --><a name="when-to-use-practical-use-cases"></a>
### When to Use (Practical Use Cases)

Use LocalStorage for small, non-critical, non-sensitive data that enhances the user experience.

*   **UI Preferences:** Saving a user's choice of "Dark Mode" or a collapsed sidebar state so the site looks correct instantly on their next visit.
*   **Form Autosave:** Saving the contents of a long text area as the user types. If their browser crashes, you can restore their draft from LocalStorage.
*   **Non-critical Caching:** Storing an API response for a static list (e.g., a list of countries) to avoid unnecessary network requests on subsequent visits.

<!-- TOC --><a name="when-not-to-use"></a>
### When NOT to Use

*   **Storing PII or Sensitive Data:** Never store passwords, credit card numbers, or personal identifying information.
*   **Storing Authentication Tokens (JWTs):** While heavily debated, storing JWTs in LocalStorage is dangerous due to XSS vulnerabilities (see Security Risks below). 
*   **High-Frequency Operations:** Do not use it for data that updates dozens of times per second (like a game loop state or mouse coordinates) due to its synchronous, main-thread-blocking nature.
*   **Large Datasets:** If you need to store megabytes of data, search indexes, or blobs, use **IndexedDB** instead.

<!-- TOC --><a name="security-risks-ttl"></a>
### Security Risks & TTL

**Security Risks (XSS)**
The biggest flaw of LocalStorage is that it is accessible via global JavaScript (`window.localStorage`). If your website has a Cross-Site Scripting (XSS) vulnerability—meaning a malicious user manages to inject their own JavaScript into your page—that malicious script can instantly read everything in LocalStorage and send it to an external server. This is why storing sensitive tokens here is risky; an `HttpOnly` cookie is generally safer for session tokens because JavaScript cannot access it.

**TTL (Time To Live)**
By default, LocalStorage is permanent. It **does not support a built-in TTL or expiration mechanism**.

If you want data to expire, you must build the logic yourself by storing a timestamp alongside your data and checking it during retrieval:

```javascript
// Implementation of LocalStorage with TTL
function setItemWithExpiry(key, value, ttlInMilliseconds) {
  const now = new Date();
  const item = {
    value: value,
    expiry: now.getTime() + ttlInMilliseconds,
  };
  localStorage.setItem(key, JSON.stringify(item));
}

function getItemWithExpiry(key) {
  const itemStr = localStorage.getItem(key);
  if (!itemStr) return null;

  const item = JSON.parse(itemStr);
  const now = new Date();

  // Check if the item is expired
  if (now.getTime() > item.expiry) {
    localStorage.removeItem(key); // Clean up
    return null;
  }
  return item.value;
}

// Usage: Store "token" for 1 hour (3600000 ms)
setItemWithExpiry('sessionData', 'abc-123', 3600000);
```

<!-- TOC --><a name="sessionstorage"></a>
## SessionStorage

<!-- TOC --><a name="introduction-to-sessionstorage"></a>
### Introduction to SessionStorage

**What it is:**
SessionStorage is a web storage API identical in syntax to LocalStorage, but with one critical difference: **its lifespan and scope are strictly tied to the specific browser tab (or window) it was created in.** When the tab is closed, the data is instantly destroyed.

**Why it exists:**
While LocalStorage solved the capacity and network-overhead problems of Cookies, it introduced a new issue: data permanence and cross-tab leaking. If a user is booking a flight in Tab A, and opens the same site in Tab B to look at a different flight, LocalStorage would mix their states together. SessionStorage isolates data so each tab has its own pristine environment, and it cleans up automatically so developers don't have to write manual deletion logic.

**How it works (ASCII Diagram):**

```text
========================================================================
                      THE BROWSER (One Website)
========================================================================

    [ TAB 1: Checkout ]                   [ TAB 2: Browsing ]
            |                                     |
            v                                     v
+-----------------------+             +-----------------------+
| window.sessionStorage |             | window.sessionStorage |
|-----------------------|             |-----------------------|
| "step": "payment"     |  <-- NO --> | "step": "search"      |
| "cartId": "xyz987"    |    ACCESS   | "cartId": "abc123"    |
+-----------------------+             +-----------------------+
            |                                     |
    (Tab Closed -> PURGED)                (Tab Closed -> PURGED)

========================================================================
```

**How to use it (Code Snippets):**

Because it shares the exact same `Storage` interface as LocalStorage, you still must serialize complex objects.

```javascript
// 1. Storing data for the current tab session
sessionStorage.setItem('checkoutStep', '2');

// Storing objects (requires serialization)
const temporaryFilters = { category: 'shoes', maxPrice: 100 };
sessionStorage.setItem('catalogFilters', JSON.stringify(temporaryFilters));

// 2. Retrieving data
const currentStep = sessionStorage.getItem('checkoutStep'); // Returns "2"

// Retrieving objects (requires parsing)
const savedFilters = JSON.parse(sessionStorage.getItem('catalogFilters'));

// 3. Removing a specific item
sessionStorage.removeItem('checkoutStep');

// 4. Clearing the entire storage for this specific tab
sessionStorage.clear();
```

<!-- TOC --><a name="browser-support-1"></a>
### Browser Support

Like LocalStorage, SessionStorage is universally supported across all modern and legacy browsers (HTML5 standard, IE8+).

<!-- TOC --><a name="pros-and-cons-1"></a>
### Pros and Cons 

**The Problem it Solves:** It elegantly solves the "dirty state" problem. Before SessionStorage, developers had to use complex cookie-scoping or URL query parameters to ensure a user running two simultaneous workflows in different tabs didn't corrupt their own data.

**Pros:**
*   **Automatic Cleanup:** No manual garbage collection required; the browser deletes it when the tab closes.
*   **Strict Tab Isolation:** Prevents data collision when users open the same web app in multiple tabs.
*   **Generous Capacity:** Offers ~5MB of storage, far exceeding the 4KB limit of cookies.
*   **Zero Network Overhead:** Data never leaves the client's browser.

**Cons:**
*   **Synchronous & Blocking:** Read/write operations block the Main Thread.
*   **Strings Only:** Requires parsing and stringifying for JSON objects.
*   **Volatility:** If a user accidentally closes the tab, their temporary state is immediately lost (though modern browsers sometimes restore SessionStorage if you use the "Undo Closed Tab" feature).

<!-- TOC --><a name="when-to-use-practical-use-cases-1"></a>
### When to Use (Practical Use Cases)

Use SessionStorage for temporary state that is highly specific to a single workflow.

*   **Multi-Step Forms & Wizards:** Saving form inputs during a checkout process so if the user hits the browser's "Back" button, their data is still there, but is wiped clean once they close the site.
*   **Temporary UI State:** Remembering if a user has expanded a specific accordion menu or applied specific search filters on a data table during their current browsing session.
*   **Banking or Secure Workflows:** Enforcing that a workflow (like a wire transfer) is isolated to one tab. If the user opens a new tab, it forces them to start a fresh, secure session.

<!-- TOC --><a name="when-not-to-use-1"></a>
### When NOT to Use

*   **Cross-Tab Communication:** If you need to share data or trigger events across multiple open tabs, use LocalStorage (which fires a `storage` event) or a BroadcastChannel. SessionStorage is strictly isolated.
*   **Long-Term Preferences:** Do not use it for things like "Dark Mode" preferences, as the user will have to re-select it every time they open a new tab.
*   **Large Datasets:** Never use it to cache massive API payloads, as the synchronous stringification will cause Main Thread freezing (Jank).

<!-- TOC --><a name="security-risks-ttl-1"></a>
### Security Risks & TTL

**Security Risks (XSS)**
Just like LocalStorage, SessionStorage is entirely vulnerable to Cross-Site Scripting (XSS). If an attacker injects malicious JavaScript into your webpage, they can read everything inside `window.sessionStorage`. For this reason, you should avoid storing highly sensitive data, unencrypted PII, or raw authentication tokens here. 

**TTL (Time To Live)**
SessionStorage has no programmatic, time-based TTL (e.g., "delete this in 5 minutes"). **Its TTL is inherently tied to the lifecycle of the browser tab.** It lives as long as the tab remains open, surviving page reloads and back/forward navigation within that specific tab, but dies the exact millisecond the user clicks the "X" on the tab.

<!-- TOC --><a name="cookies"></a>
## Cookies

<!-- TOC --><a name="introduction-to-cookies"></a>
### Introduction to Cookies

**What it is:**
Cookies are small pieces of text data (maximum 4KB) stored by the user's web browser. Unlike LocalStorage or SessionStorage, cookies are inherently tied to the network layer: they are automatically attached to the `Cookie` header of every single HTTP request sent to the domain that created them.

**Why it exists:**
HTTP is a stateless protocol. When a server receives a request, it has no memory of the previous requests from that same user. Cookies were the very first web storage mechanism invented (in 1994) to solve this fundamental problem. They allowed servers to "tag" a user and remember them across multiple page navigations, making concepts like shopping carts and user logins possible.

**How it works (ASCII Diagram):**

```text
========================================================================
                      THE COOKIE LIFECYCLE
========================================================================

[ CLIENT BROWSER ]                                  [ BACKEND SERVER ]
                                                      (e.g., Node.js /
                                                       Spring Boot)

1. POST /login (username, password) ---------------->
                                                      Validates user.
                                                      Generates Session ID.

                                   <----------------  2. HTTP Response
                                                      Header: 
                                                      Set-Cookie: sessionId=xyz123

3. GET /dashboard ---------------------------------->
   Header:                                            Reads 'xyz123'.
   Cookie: sessionId=xyz123                           Knows exactly who is 
                                                      making the request.

                                   <----------------  4. HTTP Response (Dashboard UI)

========================================================================
```

**How to use it (Code Snippets):**

While you can manipulate cookies in JavaScript, they are most securely and commonly set by the backend server.

**1. Server-Side Setup (The standard approach in modern system design):**
The server sends an HTTP response header to instruct the browser to store the cookie.
```http
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: auth_token=jwt_header_payload_signature; Secure; HttpOnly; SameSite=Strict; Max-Age=3600
```

**2. Client-Side Setup (JavaScript):**
You can read and write cookies in the browser, though the API is notoriously clunky because `document.cookie` acts as a giant string.
```javascript
// Writing a cookie
document.cookie = "theme=dark; max-age=86400; path=/";

// Reading all cookies (Returns a single string: "theme=dark; another=value")
const allCookies = document.cookie;

// Helper function to extract a specific cookie
function getCookie(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop().split(';').shift();
  return null;
}
```

<!-- TOC --><a name="browser-support-2"></a>
### Browser Support

Universally supported. Cookies are a foundational standard of the internet and have been supported by every web browser since Netscape Navigator in 1994.

<!-- TOC --><a name="pros-and-cons-2"></a>
### Pros and Cons

**The Problem it Solved:** Before Web Storage (LocalStorage/SessionStorage) existed, cookies were the *only* way to persist data on the client. Their primary breakthrough was solving the statelessness of HTTP, enabling persistent user sessions.

**Pros:**
*   **Automatic Transport:** You don't need to write fetch interceptors to attach them; the browser sends them to the server automatically.
*   **Security Controls:** Unlike LocalStorage, cookies have robust security flags (`HttpOnly`, `Secure`, `SameSite`) that can protect them from client-side attacks.
*   **Server-Side Readability:** They are the only storage mechanism the backend can read directly before the HTML/JS even loads.

**Cons:**
*   **Network Overhead:** Because they are sent with *every* request (even for static images or CSS), storing too much data in cookies creates a massive bandwidth bottleneck.
*   **Tiny Capacity:** Strictly limited to 4KB per domain.
*   **Clunky JS API:** Parsing strings manually via `document.cookie` is inefficient.

<!-- TOC --><a name="when-to-use-practical-use-cases-2"></a>
### When to Use (Practical Use Cases)

In scalable, distributed backend architectures, cookies are almost exclusively reserved for identity and session management.

*   **Authentication (JWTs or Session IDs):** The gold standard for secure logins. Storing an authentication token in an `HttpOnly` cookie prevents malicious JavaScript from accessing it, which is critical for secure applications like a bank ledger or a payment gateway.
*   **Server-Side Personalization:** Storing a user's language preference or A/B testing cohort. Because the server reads the cookie on the very first request, it can immediately return the correct server-side rendered (SSR) HTML without waiting for client-side JavaScript to figure it out.

<!-- TOC --><a name="when-not-to-use-2"></a>
### When NOT to Use

*   **Storing Application State:** Never use cookies to store UI state (like whether a sidebar is open) or complex JSON objects. Use LocalStorage or SessionStorage to prevent bloating your network requests.
*   **Storing Sensitive Data in Plain Text:** Never store an unencrypted email, password, or account balance in a cookie.

<!-- TOC --><a name="security-risks-ttl-2"></a>
### Security Risks & TTL

**Security Risks (XSS & CSRF)**
Cookies are a double-edged sword when it comes to security.
1.  **XSS (Cross-Site Scripting):** If an attacker injects a script into your page, they can read `document.cookie`. **Mitigation:** Setting the `HttpOnly` flag on the server completely hides the cookie from JavaScript, neutralizing this threat.
2.  **CSRF (Cross-Site Request Forgery):** Because cookies are sent automatically, an attacker can trick a user into clicking a link on a malicious site, which sends a request to your bank's API. Because the user is logged in, the browser automatically attaches the auth cookie, and the unauthorized transfer goes through. **Mitigation:** Setting the `SameSite=Strict` or `Lax` flag ensures the cookie is only sent if the request originates from your actual domain.

**TTL (Time To Live)**
Cookies have the most flexible expiration controls of all storage options, defined when the cookie is created:
*   **Session Cookies:** If no expiration is set, the cookie lives in memory and is automatically deleted the moment the user completely closes the browser window.
*   **Persistent Cookies:** You can set a precise TTL using the `Max-Age` attribute (in seconds) or the `Expires` attribute (a specific date/time). Once that time hits, the browser automatically deletes the cookie.

<!-- TOC --><a name="indexeddb"></a>
## IndexedDB

<!-- TOC --><a name="introduction-to-indexeddb"></a>
### Introduction to IndexedDB

**What it is:**
IndexedDB is a low-level, powerful, and asynchronous NoSQL database built directly into the user's web browser. It allows you to store significant amounts of structured data, including complex JavaScript objects, arrays, and even raw files (Blobs).

**Why it exists:**
As web applications evolved into complex Single Page Applications (SPAs) and Progressive Web Apps (PWAs), the limitations of LocalStorage became obvious. LocalStorage is strictly limited to ~5MB, forces you to convert everything to strings, and critically, it is **synchronous**—meaning reading or writing large amounts of data freezes the browser's Main Thread. IndexedDB was created to provide massive, asynchronous storage that doesn't cause UI jank.

**How it works (ASCII Diagram):**

Unlike a simple key-value dictionary, IndexedDB is hierarchical and transactional.

```text
========================================================================
                      INDEXED DB ARCHITECTURE
========================================================================

[ Browser Origin ] (e.g., https://myapp.com)
       |
       v
  [ Database: "MyOfflineWorkspace" (v1.0) ]
       |
       |--> [ Object Store: "Users" ]  (Like a NoSQL Table)
       |       |-- Key: 1, Value: { name: "Alice", age: 28, role: "admin" }
       |       |-- Key: 2, Value: { name: "Bob", age: 34, role: "user" }
       |       +-- (Index: "by_role" -> allows fast searching for "admin")
       |
       |--> [ Object Store: "UserUploads" ] 
               |-- Key: "file_99", Value: [ Binary Blob / Image Data ]

========================================================================
```

**How to use it (Code Snippets):**

The native IndexedDB API relies heavily on event listeners (`onsuccess`, `onerror`), which makes it notoriously verbose. 

```javascript
// Native IndexedDB Approach (Verbose)
const request = indexedDB.open("MyDatabase", 1);

// This event fires if the database needs to be created or upgraded
request.onupgradeneeded = (event) => {
  const db = event.target.result;
  // Create an object store (table) with 'id' as the primary key
  const store = db.createObjectStore("users", { keyPath: "id" });
  store.createIndex("nameIndex", "name", { unique: false });
};

// This event fires when the connection is successfully opened
request.onsuccess = (event) => {
  const db = event.target.result;
  
  // To write data, you must open a Transaction
  const transaction = db.transaction(["users"], "readwrite");
  const store = transaction.objectStore("users");
  
  // Add a native JS object (no JSON.stringify needed!)
  store.add({ id: 101, name: "Pushkar", occupation: "Engineer" });
};
```

> **Developer Note:** Because the raw API is so clunky, modern web development almost universally relies on tiny wrapper libraries like **`idb`** (by Jake Archibald), which converts the event-based API into modern `async/await` Promises.

<!-- TOC --><a name="browser-support-3"></a>
### Browser Support

IndexedDB is universally supported. It is the W3C standard for browser databases and works fully across modern versions of Chrome, Firefox, Safari, Edge, and all major mobile browsers.

<!-- TOC --><a name="pros-and-cons-the-problem-it-solves"></a>
### Pros and Cons (The Problem it Solves)

**The Problem it Solves:** It completely removes the 5MB string-only ceiling of LocalStorage and prevents the browser from freezing during heavy data reads/writes by offloading the work from the Main Thread.

**Pros:**
*   **Asynchronous:** Operations run in the background; they do not block the UI or the Critical Rendering Path.
*   **Massive Capacity:** Depending on the browser and the user's available disk space, you can store hundreds of megabytes or even gigabytes of data.
*   **Native Object Storage:** You can store JavaScript objects, Maps, Sets, and Date objects directly without serialization overhead.
*   **Binary Support:** Native support for storing `ArrayBuffer` and `Blob` data (ideal for images, audio, and video).
*   **Transactions & Indexes:** Ensures data integrity (if one part of a multi-step write fails, the whole transaction rolls back) and allows high-speed querying via indexes.

**Cons:**
*   **High Complexity:** The learning curve is steep, and the boilerplate code required for simple operations is massive compared to `localStorage.setItem()`.
*   **No Native Sync:** It does not automatically sync with your backend server. You must write the logic to push offline changes to your API when the network returns.

<!-- TOC --><a name="when-to-use-practical-use-cases-3"></a>
### When to Use (Practical Use Cases)

IndexedDB is the backbone of the offline web. Use it when you are dealing with large datasets or files.

*   **Offline-First PWAs:** Caching massive API responses (like an entire user's inbox or a product catalog) so the app loads instantly, even on a subway with no internet.
*   **Rich Media Web Apps:** In-browser video editors, audio workstations, or photo editors that need to save heavy user-uploaded files locally before processing them.
*   **State Persistence for Heavy Apps:** Saving complex, multi-megabyte Redux or Vuex state trees to prevent data loss on browser refresh.

<!-- TOC --><a name="when-not-to-use-3"></a>
### When NOT to Use

*   **Simple Key-Value Preferences:** If you just need to remember `theme = dark`, using IndexedDB is massive overkill. Stick to LocalStorage.
*   **Highly Relational Data:** While it has "indexes," IndexedDB is not a SQL database. If your app relies heavily on complex `JOIN` operations across dozens of tables, the client-side code will become very slow and complicated.
*   **Authentication:** Never use it to store secure session IDs. Use `HttpOnly` cookies.

<!-- TOC --><a name="security-risks-ttl-3"></a>
### Security Risks & TTL

**Security Risks (XSS)**
IndexedDB adheres to the Same-Origin Policy (a script from `evil.com` cannot read the database of `yourbank.com`). However, just like LocalStorage, it is completely vulnerable to **Cross-Site Scripting (XSS)**. Any malicious JavaScript that successfully executes on your domain has full, unrestricted read/write access to your IndexedDB. You should never store unencrypted Personally Identifiable Information (PII), credit card data, or passwords inside it.

**TTL (Time To Live)**
IndexedDB **does not have a programmatic TTL feature**. Data persists indefinitely until one of three things happens:
1.  **Developer Deletion:** You write custom JavaScript to check timestamps and delete old records.
2.  **User Deletion:** The user manually clears their browser data/cache.
3.  **Browser Eviction (Storage Pressure):** If the user's hard drive is running completely out of space, the browser will act in self-defense. It will automatically start deleting IndexedDB databases from the "least recently used" websites to free up room for the operating system.

<!-- TOC --><a name="cache-api"></a>
## Cache API

<!-- TOC --><a name="introduction-to-the-cache-api"></a>
### Introduction to the Cache API

**What it is:**
The Cache API is a system specifically designed to store and retrieve network `Request` and `Response` objects. While it is available to the main browser window (`window.caches`), it was built hand-in-hand with **Service Workers** to be the core engine behind Progressive Web Apps (PWAs).

**Why it exists:**
Historically, developers had zero control over the browser's implicit HTTP cache. If the internet went down, the browser would immediately show the dreaded "No Internet Connection" dinosaur game. 
The Cache API exists to give developers programmable, explicit control over network requests. It allows you to intercept a request, save the server's response, and then serve that saved response in the future—meaning your website can load instantly and function completely offline.

**How it works (ASCII Diagram):**

```text
========================================================================
                      THE CACHE API & SERVICE WORKER
========================================================================

[ User Clicks a Link / Image ]
            |
            v  (Fetch Event: "GET /hero.jpg")
+--------------------------------------------------+
|               SERVICE WORKER                     |
|                                                  |
|  1. Intercepts the request                       |
|  2. Checks the Cache API first (Cache-First)     |
+--------------------------------------------------+
            /                              \
     (Found in Cache)                 (Not in Cache)
          /                                  \
         v                                    v
[ THE CACHE API ]                     [ THE INTERNET ]
(Returns stored Response)             (Fetches from Server)
         \                                    /
          \                                  /
           +------------> [ SCREEN ] <------+

========================================================================
```

**How to use it (Code Snippets):**

The API is entirely asynchronous (Promise-based) and uses native HTTP Request/Response objects.

```javascript
// 1. Storing Assets (Usually done during Service Worker installation)
async function cacheCriticalAssets() {
  // Open a specific cache "bucket" (e.g., v1)
  const cache = await caches.open('my-site-cache-v1');
  
  // Fetch and store an array of URLs instantly
  await cache.addAll([
    '/',
    '/styles.css',
    '/app.js',
    '/offline.html'
  ]);
}

// 2. Retrieving Assets (Intercepting a fetch)
async function handleFetch(request) {
  // Check if the exact request exists in any cache
  const cachedResponse = await caches.match(request);
  
  if (cachedResponse) {
    return cachedResponse; // Serve from cache (Instant/Offline)
  }
  
  // If not, go to the network
  return fetch(request);
}
```

<!-- TOC --><a name="browser-support-4"></a>
### Browser Support

The Cache API is universally supported across all modern browsers (Chrome, Firefox, Safari, Edge) alongside the Service Worker specification. 

<!-- TOC --><a name="pros-and-cons-3"></a>
### Pros and Cons 

**The Problem it Solves:** 
Before the Cache API, developers tried to cache application files using `ApplicationCache` (AppCache), which was notoriously buggy and inflexible, or by dumping strings of HTML into LocalStorage. The Cache API solved this by natively supporting HTTP Streams, meaning it can store massive video files, images, and fonts without converting them to strings or blocking the main thread.

**Pros:**
*   **Offline Capabilities:** The only API that can intercept network requests and return cached responses to keep a site running without an internet connection.
*   **Asynchronous:** Promise-based; it never blocks the Main Thread or the Critical Rendering Path.
*   **Massive Capacity:** Shares a generous storage quota with IndexedDB (often hundreds of megabytes or gigabytes, depending on the device's free space).
*   **Native HTTP Support:** Works natively with standard `Request` and `Response` objects.

**Cons:**
*   **Complex Lifecycle:** Managing cache versions (e.g., deleting `v1` when `v2` is released) is entirely manual and easy to mess up.
*   **Stream Consumption:** A `Response` body is a readable stream and can only be consumed *once*. If you want to put a response in the cache AND return it to the browser simultaneously, you have to explicitly clone it (`response.clone()`), which catches many developers off guard.

<!-- TOC --><a name="when-to-use-practical-use-cases-4"></a>
### When to Use (Practical Use Cases)

Use the Cache API strictly for network-level assets and offline routing.

*   **Serving the App Shell:** Caching your core HTML, CSS, JavaScript, and logos so the basic UI of your web app loads instantly on repeat visits, regardless of network speed.
*   **Offline Fallbacks:** Storing an `offline.html` page. If a user tries to visit a page they haven't cached while on an airplane, the Service Worker can catch the failed network request and serve the offline page instead of the browser's default error.
*   **Advanced Caching Strategies:** Implementing patterns like "Stale-While-Revalidate" (serve the fast cached version to the user immediately, while silently fetching the newest version from the network in the background to update the cache for next time).

<!-- TOC --><a name="when-not-to-use-4"></a>
### When NOT to Use

*   **Storing Application State:** Do not use it to store a user's shopping cart, a draft of a document, or UI preferences. That is the job of IndexedDB or LocalStorage.
*   **Storing Raw JSON Data:** While you *can* wrap JSON in a synthetic `Response` object and store it here, it is not searchable or indexable. If you have a massive JSON list of 10,000 products, use IndexedDB.

<!-- TOC --><a name="security-risks-ttl-4"></a>
### Security Risks & TTL

**Security Risks (Cache Poisoning)**
The Cache API adheres to the Same-Origin Policy. However, if your site suffers from Cross-Site Scripting (XSS), an attacker can execute JavaScript to write malicious `Response` objects directly into your cache. This is known as **Cache Poisoning**. If they overwrite your cached `app.js` with a malicious script, that malicious script will run every time the user visits your site, even if the original XSS vulnerability is fixed on the server.

Another quirk involves **Opaque Responses** (fetching cross-origin resources without CORS headers, like a third-party image). The Cache API will store these, but for security and privacy reasons, the browser artificially pads their size (sometimes inflating a 10kb image to 7MB in the cache) to prevent attackers from guessing the file contents based on exact byte sizes. Storing too many opaque responses will rapidly exhaust your user's hard drive space.

**TTL (Time To Live)**
Like IndexedDB, the Cache API has **no programmatic TTL**. 
*   It does not respect HTTP Cache headers (like `max-age` or `Cache-Control`). 
*   Once a response is stored in the Cache API, it stays there forever until you explicitly write JavaScript to call `cache.delete(request)` or the user wipes their browser data.
*   If the device's hard drive reaches critical capacity, the browser will act in self-defense and delete entire origin buckets (IndexedDB + Cache API) to keep the operating system stable.

<!-- TOC --><a name="http-cache"></a>
## HTTP Cache

<!-- TOC --><a name="introduction-to-the-http-cache"></a>
### Introduction to the HTTP Cache

**What it is:**
The HTTP Cache (often referred to as the Disk Cache or Memory Cache) is the browser's native, implicit storage mechanism. Unlike LocalStorage or IndexedDB, you do not write JavaScript to put data here. Instead, the browser automatically saves network responses (images, CSS, JS) based on the HTTP headers sent by your backend server.

**Why it exists:**
Fetching assets over a network is slow and expensive. If your website's logo and CSS file never change, forcing the user to re-download them on every single page navigation is a massive waste of bandwidth. The HTTP Cache solves this by saving the files locally so subsequent loads are nearly instantaneous.

**How it works (ASCII Diagram):**

```text
========================================================================
                      THE HTTP CACHE LIFECYCLE
========================================================================

[ FIRST VISIT ]
Browser: "I need styles.css"
    |
    v (Network)
Server:  "Here is styles.css. Please cache it for 1 year."
         (Header: Cache-Control: max-age=31536000)
    |
    v
Browser: *Saves styles.css to local hard drive (Disk Cache)*

[ SECOND VISIT (Or navigating to a new page) ]
Browser: "I need styles.css"
    |
    v (Checks local cache first)
Cache:   "I have it! It's only been 2 days, so it's still fresh."
    |
    v (Network Bypassed!)
Browser: *Loads styles.css instantly from local disk in 0ms*

========================================================================
```

**How to use it (Code Snippets):**

You control the HTTP Cache entirely from the backend server configuration or CDN, not via frontend JavaScript.

```http
# 1. Cache this forever (e.g., an image or hashed JS bundle)
Cache-Control: public, max-age=31536000, immutable

# 2. Never cache this (e.g., real-time banking API response)
Cache-Control: no-store

# 3. Check with the server before using the cached version (Validation)
# The server will return a 304 Not Modified if the file hasn't changed
Cache-Control: no-cache
ETag: "W/123456789"
```

<!-- TOC --><a name="browser-support-5"></a>
### Browser Support

Universally supported. It is a fundamental part of the HTTP protocol and works in every browser ever created. Modern browsers split this into two tiers automatically:
*   **Memory Cache:** Stores assets in RAM for the duration of the page session (ultra-fast, ~0ms).
*   **Disk Cache:** Stores assets on the hard drive for long-term persistence across browser restarts (fast, ~2-5ms).

<!-- TOC --><a name="pros-and-cons-4"></a>
### Pros and Cons

**The Problem it Solves:** It was the original solution to the fundamental slowness of the internet, allowing static files to be downloaded exactly once.

**Pros:**
*   **Zero JavaScript:** Requires zero frontend code. It is managed entirely by standard network headers.
*   **Automatic:** The browser handles all the complex logic of storing, retrieving, and evicting files.
*   **Massive Performance Boost:** Bypassing the network is the single best way to optimize the Critical Rendering Path.

**Cons:**
*   **Hard to Invalidate:** "Cache invalidation" is notoriously difficult. If you tell the browser to cache `app.js` for a year, and you deploy a critical bug fix tomorrow, the user's browser will stubbornly use the broken cached version for a year. 
*   **Implicit:** Frontend developers have no API to explicitly say `cache.delete('app.js')`.

<!-- TOC --><a name="when-to-use-practical-use-cases-5"></a>
### When to Use (Practical Use Cases)

*   **Hashed Static Assets:** Modern bundlers (Webpack/Vite) output files like `main.a3b4c.js`. Because the filename changes every time the code changes, you can safely set the HTTP Cache to cache these files permanently.
*   **Images and Fonts:** Logos, hero images, and web fonts rarely change and should be heavily cached.

<!-- TOC --><a name="when-not-to-use-5"></a>
### When NOT to Use

*   **HTML Documents (Usually):** You generally do not want to heavily cache your main `index.html`. If the HTML is cached, the browser won't know to look for your newly deployed `main.new-hash.js` file.
*   **Dynamic API Data:** Real-time dashboards, user profiles, or shopping cart totals must never be cached by the implicit HTTP cache.

<!-- TOC --><a name="security-risks-ttl-5"></a>
### Security Risks & TTL

**Security Risks (Shared Devices)**
If a user generates a sensitive PDF (like a tax return or a bank statement) on a shared library computer, the browser might save that PDF in the Disk Cache. The next person to use that computer could potentially inspect the cache and view the sensitive document. 
*Mitigation:* Backend servers must explicitly send `Cache-Control: private` (meaning only a single-user device can cache it) or `Cache-Control: no-store` (absolutely no caching allowed) for sensitive data.

**TTL (Time To Live)**
The TTL is explicitly controlled by the `max-age` directive in the `Cache-Control` header. 
*   `max-age` is measured in seconds. 
*   `max-age=86400` means the cache is valid for exactly 24 hours. 
*   Once the TTL expires, the cache becomes "stale." The next time the browser needs the file, it will reach out to the network to fetch a fresh copy.

<!-- TOC --><a name="bfcache"></a>
## Bfcache

<!-- TOC --><a name="introduction-to-bfcache-backforward-cache"></a>
### Introduction to Bfcache (Back/Forward Cache)

**What it is:**
Bfcache is an implicit, browser-managed, in-memory cache. Unlike the HTTP cache that saves files, bfcache takes a complete, frozen snapshot of an entire webpage—including the parsed DOM, rendered layout, and the exact state of the JavaScript heap—when a user navigates away from it.

**Why it exists:**
Historically, clicking the browser's "Back" button forced the page to entirely reload and re-execute its JavaScript from scratch, which felt slow and wasteful. Bfcache exists to make back and forward navigations absolutely instantaneous. 

**How it works (ASCII Diagram):**

```text
========================================================================
                      THE BFCACHE LIFECYCLE
========================================================================

1. User is on Page A.
   [ Page A (Active in Memory) ]

2. User clicks a link to Page B.
   Page A is frozen and moved to Bfcache.
   [ Page A (Paused in Bfcache) ] ----> [ Page B (Active in Memory) ]

3. User clicks the browser "Back" button.
   Page B is destroyed (or bfcached). Page A is instantly resumed!
   [ Page A (Restored! 0ms Load Time) ] <---- [ Back Button ]

========================================================================
```

**How to use it (Code Snippets):**

You do not explicitly "save" data to bfcache. Instead, you write JavaScript to listen for when your page is restored from it, so you can update stale data (like a shopping cart count).

```javascript
// Listen for the page becoming visible
window.addEventListener('pageshow', (event) => {
  // event.persisted is TRUE if the page was restored from bfcache
  if (event.persisted) {
    console.log('Restored from bfcache! Refreshing user data...');
    fetchLatestCartCount();
  } else {
    console.log('Fresh page load.');
  }
});

// Listen for the page freezing (moving into bfcache)
window.addEventListener('pagehide', (event) => {
  if (event.persisted) {
    console.log('Page frozen and saved to bfcache.');
    // Pause expensive timers or close open WebSockets here!
  }
});
```

<!-- TOC --><a name="browser-support-6"></a>
### Browser Support

Bfcache is universally supported across modern browsers. Safari has championed it for over a decade, Firefox followed suit, and Chrome fully rolled it out for desktop and mobile in recent years.

<!-- TOC --><a name="pros-and-cons-5"></a>
### Pros and Cons 

**The Problem it Solves:** It eliminates the loading screen for the most common user interaction on the web (the back button), solving the problem of redundant network requests and CPU execution.

**Pros:**
*   **Zero Latency:** Back/forward navigations take literally 0 milliseconds.
*   **Saves Battery & Data:** Bypasses the network, HTML parsing, and JavaScript execution completely.
*   **Maintains Scroll Position & State:** The user lands exactly where they left off, with form inputs and UI state perfectly preserved.

**Cons:**
*   **Easily Broken (Ineligible):** The browser will silently refuse to bfcache your page if you use legacy features (like the `unload` event listener) or leave active connections open (like WebSockets or IndexedDB transactions).
*   **Stale Data:** Because the page is frozen, any data on the screen (like stock prices or inventory) will be exactly as old as when the user left the page.

<!-- TOC --><a name="when-to-use-optimize-for-it"></a>
### When to Use (Optimize for it)

You want almost every public-facing page to be eligible for bfcache.

*   **Content Sites & Blogs:** Perfect for users jumping back and forth between a search results page and individual articles.
*   **E-Commerce:** Great for users browsing a product catalog, clicking an item, and hitting "back" to keep browsing.
*   **Single Page Applications (SPAs):** While SPAs handle routing internally, they should still optimize for bfcache for when users navigate *away* from the SPA entirely and then hit the back button to return.

<!-- TOC --><a name="when-not-to-use-prevent-it"></a>
### When NOT to Use (Prevent it)

*   **Highly Sensitive Applications:** For banking portals or medical dashboards, you do not want a frozen snapshot of a logged-in state sitting in memory if the user navigates away. 
*   **How to prevent it:** Set the HTTP response header `Cache-Control: no-store` on the server. The browser respects this and will absolutely refuse to bfcache the page.

<!-- TOC --><a name="security-risks-ttl-6"></a>
### Security Risks & TTL

**Security Risks (Shared Devices)**
The primary risk is exposing sensitive data on a shared device. If a user logs into a bank, views their account, navigates to a public website (like Google), and walks away without closing the tab, the next person to use the computer could simply click the "Back" button. Because bfcache bypasses the network, the bank's server cannot intercept the request to verify the session, and the frozen page will instantly render the sensitive data. 

**TTL (Time To Live)**
Bfcache has **no exact TTL**, but it is extremely short-lived compared to other storage. 
*   It is completely destroyed if the user closes the tab.
*   It is dynamically managed by the browser's memory allocator. If a user opens a heavy 3D game in a new tab, the browser will instantly purge background bfcache snapshots to free up RAM. 
*   Generally, a page will only survive in bfcache for a few minutes before the browser quietly evicts it.

<!-- TOC --><a name="origin-private-file-system-opfs"></a>
## Origin Private File System (OPFS)

<!-- TOC --><a name="introduction-to-origin-private-file-system-opfs"></a>
### Introduction to Origin Private File System (OPFS)

**What it is:**
The Origin Private File System (OPFS) is a storage endpoint provided by the modern File System API. It acts like a virtual, hidden hard drive that is entirely private to the website's origin. It allows web applications to read, write, and organize files and directories with performance that rivals native desktop applications.

**Why it exists:**
As heavy desktop applications (like AutoCAD, Photoshop, or complex SQLite databases) began porting to the web using WebAssembly (Wasm), they hit a wall. IndexedDB is great for web data, but it is too slow for high-frequency, byte-level file manipulations (like editing a 4GB video file). OPFS was created to provide a low-level, high-performance file system that gives WebAssembly and heavy web apps direct, optimized access to the device's storage.

**How it works (ASCII Diagram):**

```text
========================================================================
                      OPFS ARCHITECTURE
========================================================================

[ Web App / WebAssembly ] <--- (Fast Byte-Level Read/Write) ---> [ Web Worker ]
                                                                       |
                                                                       v
+----------------------------------------------------------------------+
|                 ORIGIN PRIVATE FILE SYSTEM (OPFS)                    |
|----------------------------------------------------------------------|
| / (Root)                                                             |
| ├── /projects/                                                       |
| │   └── massive-video.mp4  (Accessed via SyncAccessHandle)           |
| └── /database/                                                       |
|     └── app-data.sqlite                                              |
+----------------------------------------------------------------------+
                                  |
                                  v
[ User's Actual OS Hard Drive (Files are obfuscated and hidden from user) ]

========================================================================
```

**How to use it (Code Snippets):**

OPFS offers asynchronous methods for the main thread, and ultra-fast *synchronous* methods designed exclusively for Web Workers.

```javascript
// Example: Creating and writing to a file in OPFS
async function writeToOPFS() {
  // 1. Get access to the root of the private virtual drive
  const opfsRoot = await navigator.storage.getDirectory();
  
  // 2. Create or open a file
  const fileHandle = await opfsRoot.getFileHandle('draft.txt', { create: true });
  
  // 3. Create a writable stream
  const writable = await fileHandle.createWritable();
  
  // 4. Write data and close the file
  await writable.write('This is highly optimized local storage.');
  await writable.close();
}
```

*Note: For the absolute highest performance, developers use `createSyncAccessHandle()` inside a Web Worker to perform synchronous, byte-level operations.*

<!-- TOC --><a name="browser-support-7"></a>
### Browser Support

OPFS is a newer standard but is well-supported across modern browsers. It is available in Chrome (102+), Safari (15.2+), Firefox (111+), and modern Chromium-based Edge. 

<!-- TOC --><a name="pros-and-cons-6"></a>
### Pros and Cons 

**The Problem it Solves:** It solves the performance bottleneck of web storage. Earlier options (like IndexedDB) required too much overhead for byte-by-byte file manipulations, making it impossible to run heavy database engines (like SQLite) efficiently in the browser. 

**Pros:**
*   **Blazing Performance:** The `SyncAccessHandle` provides near-native read/write speeds, making it the fastest storage option on the web.
*   **File System Semantics:** You can create real folder structures, directories, and files, rather than stuffing blobs into a NoSQL database.
*   **Wasm Integration:** Perfect for WebAssembly applications that expect a standard C/C++ style file system.

**Cons:**
*   **Hidden from Users:** Files in OPFS cannot be viewed by the user in their Windows Explorer or Mac Finder. It is strictly internal to the browser.
*   **Complex API:** Managing file handles, streams, and Web Workers is significantly more complex than standard web storage.
*   **Worker Requirement:** To get the true performance benefits (synchronous access), you must write the code inside a background Web Worker.

<!-- TOC --><a name="when-to-use-practical-use-cases-6"></a>
### When to Use (Practical Use Cases)

OPFS is designed for heavy, desktop-class web applications.

*   **In-Browser Databases:** Running a full SQLite database directly in the browser. OPFS allows SQLite to lock and update specific bytes of a database file instantly.
*   **Heavy Media Editors:** Web-based video editors, audio Digital Audio Workstations (DAWs), or 3D rendering engines that need to stream gigabytes of assets without crashing the browser's memory.
*   **Web IDEs:** Browser-based code editors that need to compile and save thousands of tiny files instantly.

<!-- TOC --><a name="when-not-to-use-6"></a>
### When NOT to Use

*   **Standard Web Apps:** If you are building a standard React/Next.js dashboard, e-commerce site, or blog, OPFS is massive overkill. Use IndexedDB for caching and LocalStorage for state.
*   **User-Accessible Files:** If you want the user to download a generated PDF and see it in their "Downloads" folder, do not use OPFS. (Use the File System Access API `showSaveFilePicker()` instead).

<!-- TOC --><a name="security-risks-ttl-7"></a>
### Security Risks & TTL

**Security Risks (XSS & Isolation)**
OPFS is heavily sandboxed. It strictly enforces the Same-Origin Policy, meaning `site-A.com` cannot access the OPFS of `site-B.com`. However, like all web storage, it is vulnerable to Cross-Site Scripting (XSS). If malicious code runs on your site, it can request the OPFS root directory and read or corrupt the user's local files. Furthermore, because it is invisible to the user's OS antivirus scanners, it could theoretically be used by a malicious site to cache malware payloads (though the payload couldn't easily execute outside the browser).

**TTL (Time To Live)**
OPFS **does not have a programmatic TTL**. 
Data stored in OPFS persists indefinitely until:
1.  Your application logic deletes the files or directories.
2.  The user manually clears their browser's site data/cookies.
3.  The browser's Storage Manager evicts the data automatically because the user's physical hard drive is dangerously low on space (Browser Eviction).

