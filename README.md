## 🐳 Docker-Ready Fork

> **This fork adds complete Docker support to the Medusa B2B Starter.** Includes fixes for SSL issues, Yarn 4 workspaces, and Vite admin UI path resolution in containers.

<h1 align="center">
  <a href="http://www.amitmerchant.com/electron-markdownify"><img src="https://github.com/user-attachments/assets/38ba3a7b-e07b-4117-8187-7b171eae3769" alt="B2B Commerce Starter" width="80" height="80"></a>
  <br>
  <br>
  Medusa B2B Commerce Starter (Now with Docker)
  <br>
</h1>

<p align="center">Customizable B2B ecommerce built with <a href="https://medusajs.com/" target="_blank">Medusa 2.0</a> & Next.js Storefront</p>

<p align="center">
  <a href="https://github.com/medusajs/medusa/blob/master/CONTRIBUTING.md">
    <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat" alt="PRs welcome!" />
  </a>
    
  <a href="https://discord.gg/xpCwq3Kfn8">
    <img src="https://img.shields.io/badge/chat-on%20discord-7289DA.svg" alt="Discord Chat" />
  </a>

  <a href="https://twitter.com/intent/follow?screen_name=medusajs">
    <img src="https://img.shields.io/twitter/follow/medusajs.svg?label=Follow%20@medusajs" alt="Follow @medusajs" />
  </a>
</p>

<p align="center">
  <video src="https://github.com/user-attachments/assets/833b26a5-4b52-447f-ac30-6ae02cbe8f05" controls="controls" muted="muted" playsinline="playsinline">
</video>
</p>

<br>

## Table

- [Update](#update)
  - [Update packages](#update-packages)
  - [Run migrations](#run-migrations)
- [Resources](#resources)
      - [Learn more about Medusa](#learn-more-about-medusa)
      - [Learn more about Next.js](#learn-more-about-nextjs)
  - [Contributors](#contributors)

&nbsp;

## 🐳 Docker Setup

This fork adds complete Docker support for the Medusa B2B starter with fixes for common containerization issues.

### Quick Start

```bash
# Clone this Docker-ready fork
git clone https://github.com/christian561/b2b-starter-medusa__docker.git
cd b2b-starter-medusa__docker

# Create docker-compose.yml by copying from section below

# Start all services with Docker
docker-compose up -d

# Wait for services to start, then visit:
# - Backend + Admin: http://localhost:9000/app
# - Storefront: http://localhost:3000
```

### Default Credentials

- **Admin Email**: `admin@test.com`
- **Admin Password**: `supersecret`

### Services

| Service | URL | Purpose |
|---------|-----|---------|
| **Backend + Admin** | http://localhost:9000/app | Medusa API & Admin Dashboard |
| **Storefront** | http://localhost:3000 | Next.js B2B Storefront |
| **PostgreSQL** | localhost:5432 | Database |
| **Redis** | localhost:6379 | Cache & Sessions |

### Key Docker Fixes

This fork solves three major issues when running Medusa B2B in Docker:

1. **🔒 SSL Connection Issues**: Fixed `databaseDriverOptions` configuration for containerized PostgreSQL
2. **📦 Yarn 4 Workspace Resolution**: Proper file copying order and `--immutable` installs for workspace compatibility
3. **🎨 Admin UI Blank Page**: Added symlink `/src → /app/src` to fix Vite's absolute import path resolution

### Development Mode

The Docker setup includes volume mounts for hot-reloading:

```bash
# Backend source code changes are reflected immediately
# Storefront changes trigger Next.js hot-reload
docker-compose logs -f backend    # View backend logs
docker-compose logs -f storefront # View storefront logs
```

### Production Usage

For production deployments, consider:
- Using a managed PostgreSQL service
- Setting up proper environment variables
- Enabling SSL/TLS
- Using a reverse proxy (nginx)

### Docker Configuration

The `docker-compose.yml` includes all necessary services:

```yaml
services:
  # ───────── Infrastructure ─────────
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: medusa
      POSTGRES_USER: medusa
      POSTGRES_PASSWORD: secret
    ports: ["5432:5432"]
    volumes: [pgdata:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  # ───────── Medusa B2B Backend + Admin ─────────
  backend:
    build: ./backend
    env_file: ./backend/.env
    environment:
      - DATABASE_URL=postgres://medusa:secret@postgres:5432/medusa
      - REDIS_URL=redis://redis:6379
    depends_on: [postgres, redis]
    ports: ["9000:9000"]
    volumes:
      - ./backend/src:/app/src
      - ./backend/.medusa:/app/.medusa

  # ───────── Next.js Storefront ─────────
  storefront:
    build: ./storefront
    environment:
      - NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000
      - NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_01JJBQBV9Y2W3BGQFZZQ2F3T4W
      - NEXT_PUBLIC_BASE_URL=http://localhost:3000
      - NEXT_PUBLIC_DEFAULT_REGION=us
      - REVALIDATE_SECRET=supersecret
    depends_on: [backend]
    ports: ["3000:3000"]
    volumes:
      - ./storefront:/app
      - /app/node_modules
      - /app/.next

volumes:
  pgdata:
```

### Troubleshooting

**Common Issues:**

1. **Port conflicts**: If you get "port already in use" errors:
   ```bash
   # Stop conflicting services
   lsof -ti:9000 | xargs kill -9  # Backend
   lsof -ti:3000 | xargs kill -9  # Storefront
   ```

2. **Database connection issues**: Ensure PostgreSQL is running:
   ```bash
   docker-compose logs postgres
   ```

3. **Admin UI blank page**: This fork fixes the Vite path resolution issue, but ensure the symlink was created:
   ```bash
   docker-compose exec backend ls -la /src
   # Should show: /src -> /app/src
   ```

&nbsp;

## Prerequisites

⚠️ We have tested this repo with the below versions:

- ✅ Node 20
- ✅ Postgres 15
- ✅ Medusa 2.4
- ✅ Next.js 15

&nbsp;

## Overview
For a full feature overview, please visit [the project wiki](https://github.com/medusajs/b2b-starter-medusa/wiki).

#### Core features

- **Company Management**. Customers can manage their company and invite employees.
- **Spending Limits**. Company admins can assign spending limits to its employees.
- **Bulk add-to-cart**. Customers can add multiple variants of a product to their cart at once.
- **Quote Management**. Customers & Merchants can communicate, accept or reject quotes.
- **Order Edit**. Merchants can edit orders or quotes - add/remove item, update quantity & price management and more.
- **Company Approvals**. Companies can mandate approvals from company admins before employees can finalize a cart.
- **Merchant Approvals**. Merchants can set up approval processes for orders, ensuring compliance with business rules before fulfillment.
- **Promotions**. Customers can apply manual and automatic promotions to their cart.
- **Free Shipping Nudge**. Displays a component showing progress toward free shipping.
- **Full ecommerce support**
  - Product Pages
  - Product Collections & Categories
  - Cart & Checkout
  - User Accounts
  - Order Details
- **Full Next.js 15 support**
  - App Router
  - Caching
  - Server components/actions
  - Streaming
  - Static Pre-Rendering

&nbsp;

#### Demo

#### Quote Management

<img align="right" src="https://github.com/user-attachments/assets/110c99e8-18ba-49e5-8955-84a058b597c7" alt="image" style=: />
&nbsp;

#### Company Management

<img align="right" src="https://github.com/user-attachments/assets/361702ce-d491-4509-a930-4361ab3b4126" alt="image" style=: />
&nbsp;

#### Approval Management

<img align="right" src="https://github.com/user-attachments/assets/b93b7b94-41a9-4c5f-bd6b-abf87492ed46" alt="image" style=: />
&nbsp;

#### Product Page

<img align="right" src="https://github.com/user-attachments/assets/2cd8a3ff-5999-49af-890a-4bac7b6f2f15" alt="image" style=: />
&nbsp;

#### Cart Summary

<img align="right" src="https://github.com/user-attachments/assets/095f5565-992e-4c74-acdc-a44bd905e59b" alt="image" style=: />
&nbsp;

&nbsp;

## Quickstart

#### Setup Medusa project

```bash
# Clone the repository
git clone https://github.com/medusajs/b2b-starter-medusa.git

## Setup Backend

# Go to the folder
cd ./backend

# Clone .env.template
cp .env.template .env

# Install dependencies
yarn install

# Install dependencies, setup database & seed data
yarn install && yarn medusa db:create && yarn medusa db:migrate && yarn run seed && yarn medusa user -e admin@test.com -p supersecret -i admin

# Start Medusa project - backend & admin
yarn dev

## Setup Storefront

# Go to folder
cd ../storefront

# Clone .env.template
cp .env.template .env

# Install dependencies
yarn install
```

#### Setup publishable key

- ✅ Visit [Admin: Publishable Key](http://localhost:9000/app/settings/publishable-api-keys)
  - <b>Credentials</b>:
    - <b>email</b>: `admin@test.com`
    - <b>password</b>: `supersecret`
- ✅ Copy token key of "Webshop"
- ✅ Open file - `storefront/.env`
- ✅ Add token to this var - `NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY`

```
# Start Medusa storefront
yarn dev
```

Visit the following links to see the Medusa storefront & admin

- [Medusa Admin](http://localhost:9000/app)
- [Medusa Storefront](http://localhost:8000)

&nbsp;

# Update

Some general guidelines for when you're updating this Starter to a newer version.

## Update packages

Run `yarn install` in both projects to update you're packages to the latest versions.

## Run migrations

To reflect any changes made to data models, make sure to run `npx medusa db:migrate` in the backend project.

> Note: are you updating from a version of this Starter that didn't have the Approval module yet? Run `npx medusa exec src/scripts/create-approval-settings.ts` in the backend project to add approval settings to all existing companies.

# Resources

#### Learn more about Medusa

- [Website](https://www.medusajs.com/)
- [GitHub](https://github.com/medusajs)
- [2.0 Documentation](https://docs.medusajs.com/v2)

#### Learn more about Next.js

- [Website](https://nextjs.org/)
- [GitHub](https://github.com/vercel/next.js)
- [Documentation](https://nextjs.org/docs)

&nbsp;

## Contributors

<a href = "https://github.com/medusajs/b2b-starter-medusa/graphs/contributors">
  <img src = "https://contrib.rocks/image?repo=medusajs/b2b-starter-medusa"/>
</a>
