# Agent Note: Keep request-time Web indexes out of persistent caches

Status: implemented

English | [中文](2026-08-15-dynamic-index-no-store.zh.md)

## Problem

The Web fallback reads and transforms `index.html` for every request, but its response carried no cache directive. The resulting document is not a static asset: index taps inject the current client-plugin graph and the pre-plugin theme bootstrap. A long-lived browser profile could therefore retain an entry document whose boot data no longer matched the Host that answered the next launch. A damaged stored response could also expose inline boot or style text before the application mounted, even while a separate browser profile continued to load the same Host correctly.

## Decision

`dsh-host-frontend-static` sends `Cache-Control: no-store` on every index response: the root path, explicit `index.html`, SPA fallback paths, and their HEAD forms. The plugin still reads the dist index and applies the current index taps before answering each request.

Static asset responses keep their existing behavior. Their content-hashed Vite names and revisioned client-plugin URLs remain the cache identities; the entry document alone is excluded from persistent caches because it contains request-time composition data.

## Alternatives considered

**Use `no-cache`.** Rejected because it still permits storage and relies on revalidation, while this server does not emit an index validator. The entry document is small and has no offline contract, so retaining it provides little benefit.

**Version the index URL.** Rejected because browsers and desktop shells enter through a stable root URL and should not need knowledge of the current Host graph before loading it. Versioned child assets already carry their own identities.

**Clear each client's complete cache at startup.** Rejected because the server owns the response semantics, and clearing a profile-wide cache discards unrelated immutable assets while failing to protect other clients.

## Consequences

Each page navigation fetches a current entry document and receives the current boot graph and theme bootstrap. Reloads cannot reuse a persistently stored index, while scripts, styles, plugin bundles, icons, and the manifest retain their existing cache behavior.

The real Loader composition test pins `no-store` on root, explicit-index, SPA-fallback, and HEAD responses and pins that a JavaScript asset receives no new cache directive.
