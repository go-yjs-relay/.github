<p align="center"><img src="https://raw.githubusercontent.com/go-yjs-relay/brand/main/social/go-yjs-relay.png" alt="go-yjs-relay" width="640"></p>

<h1 align="center">go-yjs-relay</h1>
<p align="center"><strong>A pure-Go Yjs y-websocket relay hub — transport-agnostic, forward-compatible, no cgo.</strong></p>

<p align="center">
  🌐 <a href="https://go-yjs-relay.github.io">Website</a> ·
  📚 <a href="https://go-yjs-relay.github.io/docs/">Documentation</a>
</p>

<p align="center">
  <a href="https://go-yjs-relay.github.io/docs/"><img alt="Docs" src="https://img.shields.io/badge/docs-mkdocs--material-14B8A6?style=flat-square"></a>
  <a href="https://github.com/go-yjs-relay/yrelay/blob/main/LICENSE"><img alt="License: BSD-3-Clause" src="https://img.shields.io/badge/license-BSD--3--Clause-blue?style=flat-square"></a>
  <img alt="Go 1.26.4+" src="https://img.shields.io/badge/go-1.26.4%2B-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img alt="Coverage 100%" src="https://img.shields.io/badge/coverage-100%25-1a7f37?style=flat-square">
</p>

---

go-yjs-relay is a **pure-Go (no cgo)** implementation of the server side of the
[Yjs](https://github.com/yjs/yjs)
[y-websocket](https://github.com/yjs/y-websocket) protocol as a **relay hub**. It
multiplexes collaborative-editing connections into rooms and fans each client's
binary frames out to the rest of the room — the exact relay-only deployment mode
the upstream y-websocket server uses.

It is **transport-agnostic**: it imports no WebSocket library. The caller owns
the socket and pushes/pulls binary frames through a `Membership` handle, so it
drops into any Go WebSocket stack (`nhooyr.io/websocket`, `gorilla/websocket`,
`net/http`'s own upgrader, or an in-memory pipe in tests).

> **A relay, not a server-side CRDT.** It does **not** decode Yjs updates
> server-side; the clients reconcile among themselves via Yjs's update +
> sync-step algebra. That is what makes it forward-compatible with new
> y-protocols versions for free — and a few hundred lines instead of a
> several-thousand-line Go Yjs implementation.

## Repositories

| Repo | What it is |
|------|------------|
| [**yrelay**](https://github.com/go-yjs-relay/yrelay) | the relay hub: `Hub` / `Room` / `Membership`, room multiplexing, fan-out broadcast, slow-peer drop, room GC, context-driven leave |
| [**docs**](https://github.com/go-yjs-relay/docs) | MkDocs Material documentation, versioned with [mike], served at [/docs/](https://go-yjs-relay.github.io/docs/) |
| [**go-yjs-relay.github.io**](https://github.com/go-yjs-relay/go-yjs-relay.github.io) | the Hugo landing page |
| [**brand**](https://github.com/go-yjs-relay/brand) | logos and brand assets |

## Principles

- **Pure Go, zero cgo.** Cross-compiles and embeds anywhere; a static binary by
  default.
- **A relay, not a CRDT.** The server shuttles opaque binary frames; clients own
  the CRDT. That keeps the server forward-compatible with new y-protocols
  versions and small.
- **Transport-agnostic.** No WebSocket library baked in — the `Membership` handle
  is the seam any transport plugs into.
- **Robust under load.** Slow peers are dropped rather than blocking the hub;
  empty rooms are GC'd; `Leave` is idempotent and context-driven.
- **100% test coverage** under `-race` is the target, enforced as a CI gate.

## Status

**Relay hub complete.** `Hub` / `Room` / `Membership` with room multiplexing,
echo-suppressed fan-out broadcast, non-blocking slow-peer drop, automatic room GC
on the last leave, idempotent `Leave`, and `LeaveOnContextDone`. Pure stdlib
(`context` + `sync`), CGO-free, 100% test coverage under the race detector,
`gofmt` + `go vet` clean, CI green across the six 64-bit Go targets (amd64,
arm64, riscv64, loong64, ppc64le, s390x). Persistence (snapshotting the last
update to a backing store) is a downstream, opt-in concern by design.

BSD-3-Clause.

[mike]: https://github.com/jimporter/mike
