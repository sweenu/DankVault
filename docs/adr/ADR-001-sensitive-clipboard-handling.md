# ADR-001: Sensitive Clipboard Handling

**Status:** Accepted (revised 2026-10-29)
**Date:** 2026-04-22
**Applies to:** `DankVault.qml`

## Context

Passwords copied via the Bitwarden plugin were persisted in the Wayland clipboard history (e.g. cliphist), making secrets visible long after use. The clipboard also retained the value indefinitely.

The original decision also used `wl-copy --paste-once`. That mode breaks pasting into browsers (Firefox, Chromium, Brave) running as native Wayland clients: they request the clipboard contents more than once per paste, the first request consumes the single serve, and the paste comes up empty while the clipboard is cleared ([wl-clipboard#107](https://github.com/bugaevc/wl-clipboard/issues/107)). The wl-clipboard maintainer considers this expected behavior and recommends a timed `wl-copy --clear` for passwords instead.

## Decision

Use `wl-copy --sensitive` when copying secrets, and clear the clipboard after 15 seconds.

- `--sensitive`: hints to clipboard managers not to store the entry in history.
- `sleep 15 && wl-copy --clear`: removes the secret from the clipboard once the window has passed, whether or not it was pasted.

## Alternatives Considered

- **`--paste-once` plus timed clear** (the original decision): Limits the secret to a single paste, but pastes into browsers fail, which is where most passwords are entered.
- **`--sensitive` only**: Depends on clipboard manager honoring the hint; the value stays available indefinitely.
- **Timer-only clearing**: No protection against clipboard history logging; value visible until timeout.
- **No change**: Unacceptable — passwords should not persist in clipboard history.

## Consequences

- Copied secrets paste correctly into browsers as well as other Wayland clients.
- A secret can be pasted any number of times within the 15-second window, so users filling several fields need not re-copy, but any client that reads the clipboard during that window can read it.
- Clipboard managers that respect the `sensitive` hint (cliphist does) will not log the entry.
- The 15-second timer clears the clipboard after every copy; anything the user copies in the meantime is cleared too.
- The toast message indicates the 15s lifetime so the user knows to paste promptly.
