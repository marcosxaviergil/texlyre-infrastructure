# syntax=docker/dockerfile:1
# Dockerfile do frontend — usado pelo texlyre-infrastructure
# contexto de build: raiz do texlyre-infrastructure

# ─── Stage 1: build ───────────────────────────────────────────────────────────
FROM node:24-alpine AS builder

WORKDIR /app

# Dependências (cache de layer)
COPY frontend/texlyre/package.json frontend/texlyre/package-lock.json ./
RUN npm ci

# Código-fonte do submodule
COPY frontend/texlyre/ ./

# Userdata do homelab (sobrescreve o padrão do submodule)
COPY frontend/userdata.pi.json ./userdata.json
COPY frontend/userdata.pi.mobile.json ./userdata.mobile.json

# Build de produção
RUN npm run build:prod

# ─── Stage 2: serve ───────────────────────────────────────────────────────────
FROM nginx:stable-alpine

RUN rm -f /etc/nginx/conf.d/default.conf

# Fix MIME type para módulos ES (.mjs) — workers WASM
RUN sed -i '/application\/javascript/a\    application\/javascript                   mjs;' /etc/nginx/mime.types

COPY nginx/texlyre.conf /etc/nginx/conf.d/texlyre.conf

COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]