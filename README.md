## 1 Create Dockerfile
```
# Dockerfile
FROM python:3.11-slim

# —–– Fast, reliable Python in containers
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

# Install deps first (better layer caching)
COPY requirements.txt .
RUN pip install --upgrade pip \
 && pip install -r requirements.txt \
 && pip install "uvicorn[standard]"

# Copy code
COPY app ./app

# Expose FastAPI default port
EXPOSE 8000

# Start server
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```
Also add a .dockerignore (important!)
```
__pycache__/
*.pyc
*.pyo
*.pyd
.env
.git
.gitignore
.vscode/
.idea/
dist/
build/
node_modules/
```

## 2 Build the image
```
docker build -t myapp:latest .
```

## 3 Run the container (manual workflow)
```
docker run --name myapp -p 8000:8000 myapp:latest
```

### With environment from .env (recommended)
```
docker rm -f myapp 2>$null
docker run --name myapp -p 8000:8000 --env-file .env myapp:latest
```

## 4 Rebuild & restart after code changes (manual, prod-style)
```
docker rm -f myapp
docker build -t myapp:latest .
docker run --name myapp -p 8000:8000 --env-file .env myapp:latest
```

## 5 Compose workflow (clean, professional)
Create docker-compose.yml in the project root:
```
services:
  api:
    build: .
    container_name: myapp
    ports:
      - "8000:8000"     # change left side to expose on a different host port (e.g. "3000:8000")
    env_file:
      - .env
    restart: unless-stopped
    # For dev hot-reload (optional) mount the code and override command:
    # volumes:
    #   - ./app:/app/app
    # command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```
Dev cycle with Compose
# First time or if Dockerfile/requirements changed
docker compose up --build -d

# Next times (just start/attach)
docker compose up -d

# Stop everything
docker compose down

# Logs
docker compose logs -f





