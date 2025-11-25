# 15 Frontend Concepts Every Senior Dev Has Mastered

![core mental models](assets/core-mental-models.png)

15 Frontend Concepts Every Senior Dev Has Mastered

## 1. The Critical Rendering Path

Its the steps the browser needs to do in order to go from HTML, CSS and JavaScript to pixels on the screen.

PENDING: short description of each step the critical rendering path does, based on image below:

![alt text](assets/15_Frontend_Concepts/Critical-Render-Path.png)
![alt text](assets/15_Frontend_Concepts/render-blocking.png)

> Senior Developer Tip 1: Dont jump to framework talk immediately (like React). Focus on the underlying principles first (eg: browser mechanics), and use frameworks as an implementation example.

## 2. Core Web Vitals

Empirical metrics that measure how fast a page loads (CRP) and how fast it responds to input.

PENDING: add a short description of what measures each CWV metric.

![alt text](assets/15_Frontend_Concepts/CWV.png)
![alt text](assets/15_Frontend_Concepts/CWV2.png)
![alt text](assets/15_Frontend_Concepts/CWV3.png)

## 3. HTTP Caching

> Catching in general, is a pattern based in a core mental model named Memoization

Memoization: reusing the output of an operation for future queries with the same input

Caching (file level): reusing a file (JS, CSS) for a predefined time after first received it (TTL)

TTL (Time to live): The time we should consider a cached asset “fresh”

### HTTP caching

![alt text](assets/15_Frontend_Concepts/HTTP-caching.png)

> TTL its managed by the "cache-tag" header?
PENDING: confirm and extend a little this info.

![alt text](assets/15_Frontend_Concepts/TTL-cache.png)

We dont usually set the caching policy ourselves, because most times our CDN will do it for us

CDN: Distribute static assets to "edge locations" across the globe for fast access with out of the box efficient cache policies and compression (content negotiation)

![alt text](assets/15_Frontend_Concepts/CDN.png)

## 4. Content Negotiation

Is the process in which the client speaks to the server and when it needs some files, it would send some Header, usually Accept Headers

![alt text](assets/15_Frontend_Concepts/content-negotiation.png)

> Its cheaper to decompress things then to download them in the original version
> Senior Developer Tip 2: Senior Frontend Developers are expected to have good fundamentals of the data layer: HTTP, REST, GraphQL, SEE & WebSockets.

## 5. Lazy Loading

Eager loading: load/fetch everything at once.
Lazy loading: load assets when the are needed (user actions like scroll, navigate, new page)

PENDING: detail deeper about lazy loading types and their use cases.

We can defer JavaScript, but we need to relay on dynamic imports.

Static import: Import js script at "build time"

![alt text](assets/15_Frontend_Concepts/static-import.png)

Dynamic import: Import a js script at "run time"

![alt text](assets/15_Frontend_Concepts/dynamic-import.png)

## 6. Bundle Splitting

Traditional Module Bundler job:
![alt text](assets/15_Frontend_Concepts/Module-Bundler.png)
With Bundle Splitting:
![alt text](assets/15_Frontend_Concepts/Module-Bundler-splitting.png)
> Senior Developer Tip 3: Think in systems, components and relationships rather than memorizing separate concepts

## 7. Critical CSS

CSS is render blocking by design so, the browser had to compute all of CSS before continue rendering the page:
![alt text](assets/15_Frontend_Concepts/critical-css.png)
To avoid problems, we need to focus on CSS "Above the Fold"
![alt text](assets/15_Frontend_Concepts/above-the-fold.png)

### Critical CSS Extraction

![alt text](assets/15_Frontend_Concepts/Critical-CSS-Extraction.png)

PENDING: which technology/library/tool helps with this critical css extraction or pre-rendering? How does Tailwind help with any of this?

## 8. Essential State

The minimum data representation needed to achieve a given UI

> Senior Developer Tip 4: The first thing a Senior Developer sees when looking at an UI is the essential State

![alt text](assets/15_Frontend_Concepts/NETFLIX.png)

State is not often the best thing because it can cause re-render, so we have to try to make calculations on Frontend to use the minimum state quantity as shown below:

![alt text](assets/15_Frontend_Concepts/Product-Info.png)

## 9. Reducer Pattern

A reducer its a pure function that takes old state and can compute a new state.
Deterministic, centralized and can be tested

Most used pattern by State management libraries today

> Pure Function: Inmutable, Deterministic, No side-effects

## 10. Windowing
...
