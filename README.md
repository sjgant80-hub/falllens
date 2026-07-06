# FallLens

**Peer-relay URL viewer. A sovereign browsing primitive.**

Type any URL. A trusted peer in your network fetches it, signs the bytes, and streams them back. Your local page verifies the content by CID, then renders inside a sandboxed iframe.

> When your network can't, a friend's can.

Live: <https://sjgant80-hub.github.io/falllens/>

---

## What it is

FallLens is a small, single-page tool for **peer-assisted web browsing**. Two roles in one page:

- **Viewer mode (default):** You type a URL. A connected peer fetches it on your behalf, signs the response bundle with their local Ed25519 key, and returns the bytes. Your page verifies the CID, then renders the content in a strict sandbox.
- **Relay mode (toggleable, off by default):** You consent to act as a relay for your peers. Incoming fetch requests are rate-limited, size-capped, and passed through your own domain blocklist before your browser fetches on their behalf.

Everything is local. There is no server. Peer transport is WebRTC data channels (FallLink). Content addressing is SHA-256 of the response bytes (FallStore). Peer trust is Ed25519 signatures (FallSignature).

## Why

Not every network can reach every URL. A coffee-shop wifi blocks parts of the internet. A hotel connection throttles video. A friend on fibre has a faster path to the source than you do. A researcher in one region wants to compare what an article looks like from another region.

FallLens lets your **trusted peer graph** help each other read the open web. This is what a browser has always been quietly asking for: a way to see through a friend's eyes.

## Why it is not a censorship-circumvention tool

FallLens is a general primitive. Relayers **choose their own blocklist** and can refuse to relay any domain. Rate limits and size caps protect relays from abuse. Users are responsible for what they ask their peers to fetch. Nothing about FallLens is about defeating specific network policies — it is about letting peers help each other with normal browsing when their own network is degraded, slow, or missing something.

Sandboxing is strict. The rendered iframe uses `sandbox="allow-scripts"` with no `allow-same-origin`, meaning a relayed page cannot touch your cookies, your storage, or the parent page. The CID-verify step means a relay peer cannot silently swap the bytes — if they do, the render is aborted.

## How it works

```
you --(fetch req)--> peer graph --(peer picks it up)--> peer.fetch(url)
                                                              |
                                            [sign+CID bundle sent back over data channel]
                                                              v
you.verify(CID) --> FallStore cache --> sandboxed iframe render
```

1. Local **Ed25519 keypair** is generated on first load (Web Crypto). Your DID is the base64 of your public key.
2. When you type a URL and hit Fetch, a `fetch.req` message is broadcast to connected peers via FallLink.
3. Any consenting peer whose relay is on will fetch the URL, hash the bytes to get a CID, sign the CID with their Ed25519 key, and send back a `fetch.res` bundle.
4. Your local page **verifies** the CID by re-hashing the bytes. Mismatch → render aborted.
5. Verified bytes are cached (FallStore, IndexedDB) so a repeat view of the same URL is instant and independently re-verifiable.
6. Rendered inside a sandboxed iframe with `srcdoc` (so no request-level identity leaks).

## Estate primitives it uses

- [FallLink](https://sjgant80-hub.github.io/falllink/) — WebRTC data-channel peer transport
- [FallStore](https://sjgant80-hub.github.io/fallstore/) — content-addressed local cache
- [FallSignature](https://sjgant80-hub.github.io/fallsignature/) — Ed25519 peer signatures

The single-file build has minimal fallbacks so the page still works standalone. Wire in the full estate scripts for real multi-peer routing.

## Trust model

- Peers are trusted to **deliver**, not to be **authentic**. Every response is verified by CID against the bytes you actually receive.
- The relayer's DID + Ed25519 signature is attached to every relay bundle, so if a peer misbehaves, they are traceable and can be dropped from your graph.
- Nothing is trusted at the network layer. The primary auth is content-addressing.

## Running locally

Just open `index.html` in a modern browser. No build step. No dependencies.

```bash
git clone https://github.com/sjgant80-hub/falllens.git
cd falllens
open index.html
```

## Deploying

This is designed for GitHub Pages. `.nojekyll` is included. Enable Pages on `main`, root `/`.

## Limits

- **CORS** applies. If a target site refuses cross-origin fetches, no browser relay can bypass that — that is a browser-security boundary, not a FallLens limit.
- **Interactivity** in the rendered iframe is limited by sandbox. FallLens is a lens (reads), not a browser (interacts). Forms, logins, and dynamic APIs are out of scope by design.
- **Single-tab demo mode**: with no external peers connected, FallLens falls back to a local demo relay so you can see the flow end-to-end. In production, connect real peers via the Settings tab.

## Licence

MIT. Use it, fork it, embed it. Attribution to AI-Native Solutions appreciated but not required.

---

◊ FallLens · a sovereign browsing primitive · [ai-nativesolutions.com](https://ai-nativesolutions.com)
