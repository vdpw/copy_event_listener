# AGENTS.md

## Purpose

This repository contains a small Rust library for listening to clipboard copy events on macOS. The public API is intentionally narrow:

- `copy_event_listener::clipboard::ClipboardListener`
- `copy_event_listener::event::{Event, Item, Data}`

The implementation currently only supports macOS and polls `NSPasteboard` for change count updates.

## Repo Layout

- `Cargo.toml`: crate metadata and dependencies
- `src/lib.rs`: public module exports
- `src/clipboard/mod.rs`: platform export boundary
- `src/clipboard/macos.rs`: macOS clipboard listener and pasteboard write logic
- `src/event.rs`: serializable event model returned by the listener
- `examples/listen.rs`: minimal example for manual testing
- `README.md`: short project overview and TODOs

## Verified Commands

Run these from the repository root:

- `cargo test`
- `cargo run --example listen`
- `cargo fmt`
- `cargo check`

`cargo test` currently passes, but there are no unit tests or doctests yet.

## Platform Notes

- This crate is macOS-only in practice. The clipboard module only re-exports the implementation under `#[cfg(target_os = "macos")]`.
- The implementation depends on `objc2`, `objc2-app-kit`, and `objc2-foundation`.
- The listener runs an infinite polling loop in `ClipboardListener::run` and does not currently support graceful shutdown.
- The default polling interval is 500ms. `with_interval(0)` normalizes back to 500ms.

## Editing Guidance

- Keep platform-specific Objective-C and AppKit interaction isolated in `src/clipboard/macos.rs`.
- Minimize changes to `unsafe` blocks. If behavior changes, prefer tightening scope and documenting the safety assumption in code.
- Preserve the current `Event -> Item -> Data` shape unless the task explicitly requires a breaking API change.
- If you add fields to `Event`, `Item`, or `Data`, consider the effect on `serde` consumers.
- When adding cross-platform support, do it behind `cfg` gates and keep the macOS implementation working unchanged unless the task requires a refactor.
- Avoid adding background threads or async runtimes unless there is a clear need; the current design is synchronous and callback-based.

## Validation Expectations

For code changes, prefer this sequence:

1. `cargo fmt`
2. `cargo check`
3. `cargo test`
4. If clipboard behavior changed, manually verify with `cargo run --example listen` on macOS

## Known Gaps

- No graceful shutdown path for the listener loop
- No automated tests covering clipboard behavior
- No non-macOS implementation
- No benchmark or profiling coverage for polling interval tradeoffs
