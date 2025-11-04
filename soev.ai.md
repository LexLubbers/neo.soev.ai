_public_

# soev.ai for NEO NL / LibreChat – quick run cheatsheet

For every mode below you need **two tiny setup steps first** – they are identical in all cases:

```bash
# 1  Environment file
cp soevai.env.example .env          # used by both compose and local dev

# 2  Librechat configuration file with presets for NEO NL
librechat.soev.ai.yaml

```

Now open `.env` and set at minimum:

```dotenv
# core runtime
CONFIG_PATH="librechat.merged.yaml"
LIBRECHAT_TAG=feat-adminpanel_ui_improvements   # or any tag you built/pulled

# AI provider (used by chat UI **and** RAG embeddings)
OPENAI_API_KEY=<your-openai-key>
# …or ANTHROPIC_API_KEY / GOOGLE_KEY …
# …or a custom endpoint like UbiOps (see the librechat.example.yaml)
EMBEDDINGS_PROVIDER=ollama
EMBEDDINGS_MODEL=nomic-embed-text
```

Or copy the provided starter environment file from the project lead
---

## 1  Production stack (pre-built image)

```bash
# 0  (one-time) authenticate with GitHub Container Registry
# export a fine-grained PAT that has `read:packages` scope
docker login ghcr.io

# pull a tagged API image built by CI
docker compose -f docker-compose.test.yml pull

# start with production compose
docker compose -f docker-compose.test.yml up -d
```

Mounts declared in `docker-compose.test.yml` map the two YAML files as writable bind mounts so runtime updates persist to the host.

---

That’s all – choose the mode that fits your workflow, ensure the two env vars (`CONFIG_PATH`, `LIBRECHAT_TAG`) and an AI key are present, and LibreChat / soev.ai is ready to chat and embed documents.

Probably running at http://localhost:3080 if you did not change the config.

The Admin panel is reachable from the account settings in the bottom left of the main application. 

## 2  Dev mode (code on host, services in Docker)

Spin up the required backing services once:

```bash
docker compose -f docker-compose.dev.yml up -d  # MongoDB, MeiliSearch, RAG, etc.
```

Then, in your working tree:

```bash
# install & build shared workspaces
npm i
npm run frontend

# Not needed for NEO NL yet build the soev.ai admin package & admin frontend
# cd packages/librechat-admin && npm run build
# cd ../../admin-frontend       && npm run build
# cd ..                         # back to repo root

# launch backend with hot-reload
npm run backend:dev

# (optional) second terminal – live-reload React client
npm run frontend:dev           # http://localhost:3090
```

The `docker-compose.dev.yml` stack exposes the same ports as the local and prod stacks, so no further configuration is required.

---

## User creation

**development**
npm run create-user

**production**
docker exec -it LibreChat-API /bin/sh -c "cd .. && npm run create-user"

---

## Project Structure

```
neo.soev.ai/
├── api/                          # LibreChat backend (Express)
├── client/                       # Main user-facing React app
├── admin-frontend/               # React admin dashboard
├── packages/
│   ├── librechat-admin/          # Express router + admin logic
│   ├── data-provider/            # React-Query hooks & data utils
│   ├── data-schemas/             # Shared TypeScript/Zod schemas
│   ├── api/                      # Client-side API helpers
│   └── custom/                   # Mounts additional routes (e.g. admin) into core
├── config/                       # Configuration utilities & scripts
│   └── translations/             # Translation files
├── e2e/                          # End-to-end tests (Playwright)
├── helm/                         # Kubernetes deployment charts
│   ├── librechat/
│   └── librechat-rag-api/
├── redis-config/                 # Redis cluster configuration
├── reranker-proxy/               # Python proxy for reranking service
├── searxng/                      # SearXNG search engine config
├── utils/                        # Utility scripts
│   └── docker/
├── .github/workflows/            # CI/CD workflows
│   ├── soevai-images.yml         # Builds & pushes Docker images
│   └── tag-images.yml
├── docker-compose.yml            # Base compose file
├── docker-compose.dev.yml        # Development services stack
├── docker-compose.test.yml       # Test/staging stack
├── deploy-compose.yml            # Production deployment
├── deploy-compose.soev.ai.yml    # Production deployment for soev.ai
├── Dockerfile.multi              # Multi-stage Docker build
├── Dockerfile.soev.ai.multi      # Custom Docker build for soev.ai
├── librechat.soev.ai.yaml        # Runtime config for soev.ai
├── librechat.example.yaml        # Example configuration
├── rag.yml                       # RAG service configuration
└── soev.ai.md                    # This quick-start guide
```