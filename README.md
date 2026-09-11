# fastAPI-1

A small FastAPI service exposing an in-memory collection of books.

## Run locally

```bash
python -m venv fastapienv
fastapienv/Scripts/activate        # Linux/macOS: source fastapienv/bin/activate
pip install -r requirements.txt
uvicorn books:app --reload
```

Interactive docs: http://127.0.0.1:8000/docs

## Endpoints

| Method | Path | Notes |
|---|---|---|
| GET | `/books` | All books |
| GET | `/books/{title}` | 404 if missing |
| GET | `/books/?category=` | Filter by category |
| GET | `/books/{author}/?category=` | Filter by author + category |
| GET | `/books/byauthor/{author}` | Filter by author |
| POST | `/books/create_book` | 201 + created book |
| PUT | `/books/update_book` | 404 if missing |
| DELETE | `/books/delete_book/{title}` | 204, or 404 if missing |

## Deployment (Azure App Service)

Live: https://fastapi-books-vbu.azurewebsites.net

Redeploy after a change:

```bash
az webapp up
```

The defaults (resource group, plan, region, app name) live in `.azure/config`,
which is gitignored because it is machine-local.

### Startup command

The startup command must be set explicitly, once per app:

```bash
az webapp config set -n fastapi-books-vbu -g rg-fastapi-books \
  --startup-file "python -m uvicorn books:app --host 0.0.0.0 --port 8000"
```

Without it App Service falls back to `gunicorn` with multiple workers. Because
`BOOKS` is a plain in-memory list, every worker would hold its own copy and
writes would appear and disappear depending on which worker served the request.

For the same reason: do not scale past one instance, and expect the list to
reset to its six initial entries on every restart or redeploy. Persisting data
requires a real database, which is a code change rather than a deployment one.
