# ---- Stage 1: build the React frontend ----
FROM node:20-alpine AS web
WORKDIR /web
COPY dev/frontend/package.json dev/frontend/package-lock.json ./
RUN npm ci --no-audit --no-fund
COPY dev/frontend/ ./
RUN npm run build

# ---- Stage 2: FastAPI serves the API + the built frontend on one URL ----
FROM python:3.11-slim
WORKDIR /app
COPY dev/backend/requirements-prod.txt ./
RUN pip install --no-cache-dir -r requirements-prod.txt
COPY dev/backend/ ./dev/backend/
COPY --from=web /web/dist ./dev/frontend/dist
ENV SERVE_FRONTEND=1
WORKDIR /app/dev/backend
EXPOSE 8000
# Render / Railway inject $PORT; default 8000 for local `docker run`
CMD ["sh", "-c", "uvicorn app.main:app --host 0.0.0.0 --port ${PORT:-8000}"]
