# dcs-back-v0

Module: `dcs-back-v0`, Go 1.21, Gin v1.10.

## Entrypoint

`main.go` — wires Handler → Service → Store (3-layer, same `internal/image` package).

## Commands

```bash
go run main.go
go build ./...
```

No tests, linters, or CI config present.

## Architecture

All image logic lives in `internal/image/`. No database — purely filesystem-backed.

| Layer | File | Role |
|-------|------|------|
| Store | `store.go` | FS read/write/delete, thumbnail generation via `disintegration/imaging` |
| Service | `service.go` | Validation (ext, size), UUID naming, orchestrates store |
| Handler | `handler.go` | Gin handlers for all endpoints |

## API (`/api/v1/images`)

- **POST `/upload`**: multipart field `"image"` (form-data) **or** JSON `{"filename":"...","data":"base64..."}`. Auto-detecta por `Content-Type`. Acepta data URIs (`data:image/png;base64,...`). Returns `{filename, url, thumbnail_url, size}`
- **GET `/list`**: returns array of `{filename, url, thumbnail_url}`
- **GET `/:filename`**: serves original file
- **GET `/thumbnails/:filename`**: serves 300×300 Lanczos thumbnail
- **DELETE `/:filename`**: deletes original + thumbnail

## Config (env vars)

| Var | Default |
|-----|---------|
| `PORT` | `9099` |
| `UPLOAD_DIR` | `./uploads` |
| `BASE_URL` | `http://localhost:{PORT}` |

Hardcoded: max 10MB, allowed exts `.jpg/.jpeg/.png/.gif/.webp`, thumbnails always in `{UPLOAD_DIR}/thumbnails/`.

## Notable

- Filenames are random UUIDs — original name is discarded.
- Thumbnails use `imaging.Fit` with Lanczos resampling.
- Delete ignores thumbnail removal failure; only original removal error propagates.
- No auth, no rate limiting, no EXIF handling.
