# go-gin-lincolng4

- **Language**: Go 1.27.2 (pinned in `install.sh`, `go 1.26.0` directive in `go.mod`)
- **Framework**: gin 1.12.0
- **SQLite driver**: mattn/go-sqlite3 1.14.52 (CGO)

## Nginx or direct

Both work — the server binds `HOST:PORT` from the environment, so run it behind Nginx on
`127.0.0.1:3000` (as in the video) or directly on `0.0.0.0:80`. The server never idle-closes
connections, satisfying Nginx's 65 s keep-alive requirement.

## Optimizations

- **Prepared statements** for all five queries: the query plan is built once at startup
  (explicitly allowed by rule 5 — every request still reads from SQLite).
- **WAL + `synchronous=NORMAL`** with a 5 s busy timeout, set via DSN pragmas, so readers
  don't block behind the single writer (rule 6 compliant).
- **Connection pool capped at 16**: plenty for 1 vCPU, bounds memory and file handles.
- **Hand-rolled JWT verification**: HMAC-SHA256 from the standard library with a pooled
  `hash.Hash`, signature checked before any JSON parsing, and the alg pinned to HS256.
  No JWT library, no reflection.
- **Structs instead of maps** for responses, so JSON key order is exact and encoding is
  allocation-light; error bodies on auth paths are pre-serialized byte slices.
- **gin release mode, no logger middleware**: nothing written to stdout per request.

The database schema is used exactly as given; `created_at` values come from the column
default (`strftime('%Y-%m-%dT%H:%M:%fZ','now')`) via `INSERT ... RETURNING`.
