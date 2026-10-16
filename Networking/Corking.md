Related to [[Lapideus Source]], [[Lapideus Documentation]] , [[Nagles Algorithm]]

### (TCP cork / nopush) in this context

**Corking** means telling the TCP stack:

“Don’t immediately send out small chunks I write; buffer them briefly so you can send a larger, better-packed TCP segment.”

In Nginx terms, this is usually done with an OS socket option:

- Linux: `TCP_CORK`
- BSD/macOS-ish: `TCP_NOPUSH`

They’re slightly different in details, but the practical effect Nginx cares about is the same: **coalescing output into fewer packets**.

It’s easy to mix these up:

- **Nagle (`TCP_NODELAY` off)**: TCP _itself_ tries to merge small writes by waiting for ACKs, to avoid lots of tiny packets.
- **Corking (`TCP_CORK`/`TCP_NOPUSH`)**: the _application_ tells TCP “hold sending until I say so (or until there’s a full segment)”.

Rule of thumb:

- **Corking** is great when you know you’re about to send “headers + body” and want them to leave efficiently.
- **`TCP_NODELAY`** is great when you want low latency for small writes (interactive/streaming-ish patterns).

Nginx often toggles between these modes depending on what it’s sending