# ShopEase Product Service

A Docker-ready FastAPI application implementing all five product CRUD
operations with **in-memory storage**. This practical assignment does not use
MySQL or any other database. Data is kept in the running process and resets
when the container restarts.

[GitHub repository](https://github.com/vivithkumar7/shopease-product-service)

## Technology

- Python 3.11+
- FastAPI, Uvicorn, Pydantic
- Docker and Docker Compose
- pytest, Swagger UI, and Postman
- Git and GitHub

## Project structure

```text
app/
  config/settings.py
  models/product_model.py
  repositories/product_repository.py
  routers/product_routes.py
  schemas/product_schema.py
  services/product_service.py
  main.py
scripts/smoke_test.py
tests/test_product_api.py
Dockerfile
.dockerignore
compose.yaml
requirements.txt
requirements-dev.txt
README.md
PROJECT_GUIDE.md
evidence/                 Docker-hosted test output and screenshots
reports/                  Project report and source
```

Routes, schemas, business logic, and storage are separate. See
[PROJECT_GUIDE.md](PROJECT_GUIDE.md) for a file-by-file explanation, test
procedure, and team discussion questions.

## Product API

Base URL when running Docker: `http://localhost:8003`

| Method | Path | Purpose | Success |
|---|---|---|---|
| POST | `/api/products/` | Create a product | 201 |
| GET | `/api/products/` | List, filter, and paginate | 200 |
| GET | `/api/products/{product_id}` | Get one product | 200 |
| PUT | `/api/products/{product_id}` | Replace product details | 200 |
| DELETE | `/api/products/{product_id}` | Delete a product | 204 |

Category filtering is case-insensitive. List pagination accepts `skip` and
`limit`. Unknown IDs return 404; invalid request data or non-positive IDs
return 422.

Example request for POST and PUT:

```json
{
  "name": "Headphones",
  "description": "Wireless headphones",
  "price": 25.5,
  "stock_quantity": 20,
  "category": "Accessories",
  "status": "Active"
}
```

Price must be positive with at most four decimal places; stock must be a
non-negative integer; status must be `Active` or `Inactive`. Names and
categories cannot be blank. Extra request fields are rejected.

## Install and run locally

Use Python 3.11 or newer. From PowerShell in the project folder:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pip install -r requirements-dev.txt
uvicorn app.main:app --host 127.0.0.1 --port 8003 --reload
```

Open <http://localhost:8003/docs>. Stop the local server with Ctrl+C.

## Dockerfile and Docker ignore

The Dockerfile builds a small Python 3.12 image and runs the API on port 8003:

| Instruction | Why it is used |
|---|---|
| `FROM python:3.12-slim` | Starts with Python 3.12 on a smaller Debian base image. |
| `WORKDIR /service` | Sets the directory used by subsequent build and run instructions. |
| `COPY requirements.txt .` | Copies the production dependency list before application code, allowing Docker to reuse the dependency layer when only source files change. |
| `RUN pip install --no-cache-dir -r requirements.txt` | Installs the FastAPI/Uvicorn dependencies without retaining pip's download cache in the image. |
| `COPY app ./app` | Copies only the application package needed to serve requests. |
| `EXPOSE 8003` | Documents the port the containerized API listens on; publishing it to the host is done with `-p`. |
| `CMD [...]` | Starts Uvicorn with the FastAPI `app` object, listening on all container interfaces at port 8003. |

`.dockerignore` excludes local environment files, virtual environments, Git
metadata, caches, tests, reports, screenshots, and other files not needed by
the image build. This reduces the build context and prevents local `.env`
configuration and credentials from being copied into the image. The Dockerfile
copies only `app` and `requirements.txt`.

## Build and run in Docker

Build the image and confirm it exists:

```powershell
docker build -t shopease-product-service .
docker image ls shopease-product-service
```

Run the standalone container (the host port 8003 must be free):

```powershell
docker run -d --name shopease-product-container -p 8003:8003 shopease-product-service
docker ps
docker logs shopease-product-container
```

Open Swagger UI at <http://localhost:8003/docs>. To stop and resume the same
container, then remove it when finished:

```powershell
docker stop shopease-product-container
docker start shopease-product-container
docker rm -f shopease-product-container
```

`docker rm -f` stops and removes that named container. It does not remove the
image. Do not run this standalone container and Docker Compose at the same
time while both attempt to publish host port 8003.

Docker Compose is an alternative that builds and starts the same service:

From PowerShell in this project folder, with Docker Desktop running:

```powershell
docker compose up -d --build
docker compose ps
docker compose logs --tail 50 product-service
```

Stop the Compose service with `docker compose down`.

## Test all APIs through Docker

```powershell
.\.venv\Scripts\python.exe scripts\smoke_test.py --base-url http://localhost:8003
```

This sends real HTTP requests through the published container port and checks:

1. Valid product creation.
2. Empty name, negative price, zero price, negative stock, invalid status, and missing-name validation.
3. Listing all products, category filtering, and pagination.
4. Reading a product with a valid ID and checking invalid/missing IDs.
5. Updating a product and preserving its creation timestamp.
6. Deleting a product and confirming the deleted product returns 404.

Every check prints its HTTP status; the script exits with an error if a
connection or assertion fails. The same scenarios can be performed in Swagger
UI or Postman with `http://localhost:8003` as the base URL. API status codes
are `201` for create, `200` for list/read/update, `204` for delete, `422` for
invalid input, and `404` for an unknown product.

Useful checks:

```powershell
curl.exe -i http://localhost:8003/
curl.exe -i http://localhost:8003/api/products/
```

Since storage is in memory, product data is intentionally temporary and is
cleared when the application process or container restarts.

## Run automated tests

```powershell
python -m pip install -r requirements-dev.txt
python -m pytest -q
```

These tests verify API validation and behavior. The smoke-test script above
separately verifies the API over HTTP through Docker.

## GitHub submission

The source repository should include application code, tests, Docker files,
documentation, and sanitized evidence. Never commit `.env`, passwords, access
tokens, `.venv`, or caches. Capture your own Docker and Swagger screenshots
after running the container, then add only those non-secret evidence files.

```powershell
git status
git add README.md PROJECT_GUIDE.md app scripts tests Dockerfile compose.yaml requirements.txt requirements-dev.txt evidence reports
git commit -m "Complete in-memory Docker product CRUD"
git push
```
