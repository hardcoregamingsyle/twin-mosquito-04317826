# WhatsApp Clone

A fully-featured, real-time messaging web app that mimics the core user experience of WhatsApp.

### Table of Contents

* [Goal](#goal)
* [Directory Structure](#directory-structure)
* [Environment Variables](#environment-variables)
* [Dependencies](#dependencies)

### Goal

Build a fully-featured, real-time messaging web app that mimics the core user experience of WhatsApp.

### Directory Structure

```
whatsapp-clone/
├─ .github/
│   └─ workflows/
│       ├─ ci.yml               # GitHub Actions CI (lint, test, build)
│       └─ cd.yml               # CD to Vercel/Render/AWS (optional)
├─ .env.example                 # Template for environment variables
├─ .eslintrc.js                 # ESLint config (Airbnb + plugin)
├─ .prettierrc                  # Prettier formatting
├─ Dockerfile                   # Multi-stage build for backend
├─ docker-compose.yml           # Local dev (DB, Redis, backend, frontend)
├─ README.md                    # Project overview, setup, scripts
├─ package.json                 # Root workspace (if using monorepo) or frontend only
├─ tsconfig.json                # TypeScript config (shared)
└─ /apps
    ├─ /frontend                # React + Vite (or Next.js) SPA/PWA
    │   ├─ public/
    │   │   ├─ icons/           # PWA icons
    │   │   └─ manifest.json    # Web app manifest
    │   ├─ src/
    │   │   ├─ assets/          # Images, SVGs, fonts
    │   │   ├─ components/      # Reusable UI atoms/molecules
    │   │   │   ├─ ui/          # shadcn-ui / Radix primitives (Button, Input, etc.)
    │   │   │   ├─ layout/      # Header, Sidebar, Footer
    │   │   │   ├─ chat/        # ChatList, ChatWindow, MessageInput
    │   │   │   └─ pages/       # Route components
    │   │   └─ services/        # API services (e.g., fetch users, messages)
    │   ├─ vite.config.js        # Vite config for frontend
    │   └─ webpack.config.js    # Webpack config for Next.js (if using)
    └─ /backend                  # Node.js Express + Socket.io server
        ├─ src/
        │   ├─ models/          # Mongoose models for DB interaction
        │   ├─ routes/          # Express routes (e.g., fetch users, messages)
        │   ├─ services/        # API services (e.g., fetch users, messages)
        │   └─ utils/           # Utility functions (e.g., encryption, hashing)
        ├─ app.js                # Main application entry point
        ├─ server.js             # Server setup and configuration
        └─ package.json          # Backend dependencies
```

### Environment Variables

Create a `.env.example` file with template environment variables.

```bash
# Template for environment variables
PORT=3000
HOST=0.0.0.0
DB_URL=mongodb://localhost:27017/whatsapp-clone
REDIS_URL=redis://localhost:6379
JWT_SECRET=my-secret-key
SESSION_SECRET=my-session-secret
```

### Dependencies

List dependencies in `package.json`.

```json
{
  "name": "whatsapp-clone",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js",
    "dev": "vite dev",
    "build": "vite build",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.17.1",
    "mongoose": "^5.9.10",
    "socket.io": "^2.3.0",
    "bcrypt": "^5.0.1",
    "jsonwebtoken": "^8.5.1",
    "session": "^0.35.1",
    "typescript": "^4.1.3"
  },
  "devDependencies": {
    "@types/express": "^4.17.9",
    "@types/mongoose": "^5.9.10",
    "@types/socket.io": "^2.3.0",
    "@types/bcrypt": "^5.0.1",
    "@types/jsonwebtoken": "^8.5.1",
    "@types/session": "^0.35.1",
    "@types/typescript": "^4.1.3",
    "ts-jest": "^27.0.3",
    "jest": "^27.0.6",
    "prettier": "^2.3.2",
    "eslint": "^7.14.0",
    "eslint-plugin-jest": "^28.0.0",
    "jest-preset": "^1.5.0"
  }
}
```

### Dockerfile

Create a `Dockerfile` for the backend.

```dockerfile
# Stage 1: Build the frontend
FROM node:14 AS frontend

# Set working directory to /app
WORKDIR /app

# Copy package.json and yarn.lock
COPY package*.json yarn.lock ./

# Install dependencies
RUN yarn install

# Copy the rest of the files
COPY . .

# Build the frontend
RUN yarn build

# Stage 2: Build the backend
FROM node:14 AS backend

# Set working directory to /app
WORKDIR /app

# Copy package.json and yarn.lock
COPY package*.json yarn.lock ./

# Install dependencies
RUN yarn install

# Copy the rest of the files
COPY . .

# Build the backend
RUN yarn build

# Stage 3: Final image
FROM --platform=linux/amd64 node:14

# Set working directory to /app
WORKDIR /app

# Copy the frontend and backend
COPY --from=frontend /app/dist /app/dist
COPY --from=backend /app/build /app/build

# Expose the port
EXPOSE 3000

# Run the command
CMD ["node", "server.js"]
```

### docker-compose.yml

Create a `docker-compose.yml` file for local development.

```yml
version: "3"

services:
  db:
    image: mongo:latest
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - ./mongo-data:/data/db

  redis:
    image: redis:alpine
    restart: always

  backend:
    build: .
    restart: always
    environment:
      DB_URL: mongodb://db:27017
      REDIS_URL: redis://redis:6379
    ports:
      - "3000:3000"
    depends_on:
      - db
      - redis

  frontend:
    build: ./frontend
    restart: always
    ports:
      - "3001:3001"
    depends_on:
      - backend
```