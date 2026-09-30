# Zwaylo — Full Project Codebase Manifest

This document serves as the complete codebase manifest for **Zwaylo**, an open-source content platform built with Node.js, Express, TypeScript, Prisma, React, Vite, and Tailwind CSS.

---

## 📁 Repository Directory Structure

```text
Zwaylo/
├── .env.example
├── .gitignore
├── LICENSE
├── README.md
├── docker-compose.yml
├── package.json
├── .github/
│   ├── CODE_OF_CONDUCT.md
│   ├── CONTRIBUTING.md
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── SECURITY.md
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── workflows/
│       └── ci.yml
├── docs/
│   ├── API.md
│   ├── ARCHITECTURE.md
│   └── DEVELOPMENT.md
├── backend/
│   ├── package.json
│   ├── tsconfig.json
│   ├── prisma/
│   │   └── schema.prisma
│   ├── src/
│   │   ├── app.ts
│   │   ├── index.ts
│   │   ├── config/
│   │   │   └── env.ts
│   │   ├── controllers/
│   │   │   ├── auth.controller.ts
│   │   │   ├── post.controller.ts
│   │   │   └── user.controller.ts
│   │   ├── db/
│   │   │   └── prisma.ts
│   │   ├── middlewares/
│   │   │   ├── auth.middleware.ts
│   │   │   ├── error.middleware.ts
│   │   │   └── validate.middleware.ts
│   │   ├── routes/
│   │   │   ├── auth.routes.ts
│   │   │   ├── index.ts
│   │   │   ├── post.routes.ts
│   │   │   └── user.routes.ts
│   │   ├── services/
│   │   │   ├── auth.service.ts
│   │   │   └── post.service.ts
│   │   └── utils/
│   │       ├── errors.ts
│   │       └── logger.ts
│   └── tests/
│       └── app.test.ts
└── frontend/
    ├── index.html
    ├── package.json
    ├── postcss.config.js
    ├── tailwind.config.js
    ├── tsconfig.json
    ├── vite.config.ts
    └── src/
        ├── App.tsx
        ├── main.tsx
        ├── components/
        │   ├── AuthModal.tsx
        │   ├── CreatePostModal.tsx
        │   ├── Navbar.tsx
        │   ├── PostCard.tsx
        │   └── Sidebar.tsx
        ├── context/
        │   └── AuthContext.tsx
        ├── services/
        │   └── api.ts
        ├── styles/
        │   └── index.css
        └── types/
            └── index.ts
```

---

## 🛠️ Root Configuration Files

### `package.json`
```json
{
  "name": "zwaylo-monorepo",
  "version": "0.1.0",
  "private": true,
  "description": "Zwaylo — A modern content platform built for discovering and sharing engaging content.",
  "scripts": {
    "dev": "concurrently \"npm run dev --prefix backend\" \"npm run dev --prefix frontend\"",
    "build": "npm run build --prefix backend && npm run build --prefix frontend",
    "test": "npm run test --prefix backend && npm run test --prefix frontend",
    "lint": "npm run lint --prefix backend && npm run lint --prefix frontend",
    "docker:up": "docker-compose up -d",
    "docker:down": "docker-compose down"
  },
  "devDependencies": {
    "concurrently": "^8.2.2"
  },
  "workspaces": [
    "frontend",
    "backend"
  ],
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  },
  "license": "MIT"
}
```

### `docker-compose.yml`
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: zwaylo_postgres
    restart: always
    environment:
      POSTGRES_USER: zwaylo_user
      POSTGRES_PASSWORD: zwaylo_password
      POSTGRES_DB: zwaylo_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U zwaylo_user -d zwaylo_db"]
      interval: 5s
      timeout: 5s
      retries: 5

  adminer:
    image: adminer:latest
    container_name: zwaylo_adminer
    restart: always
    ports:
      - "8080:8080"
    depends_on:
      - postgres

volumes:
  postgres_data:
```

### `.github/workflows/ci.yml`
```yaml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  backend-lint-and-test:
    name: Backend CI
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_USER: zwaylo_user
          POSTGRES_PASSWORD: zwaylo_password
          POSTGRES_DB: zwaylo_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Build Backend
        run: npm run build --prefix backend

      - name: Run Backend Tests
        env:
          DATABASE_URL: postgresql://zwaylo_user:zwaylo_password@localhost:5432/zwaylo_db
          JWT_SECRET: supersecret_ci_jwt_key
          PORT: 5000
        run: npm run test --prefix backend --if-present

  frontend-lint-and-build:
    name: Frontend CI
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Build Frontend
        run: npm run build --prefix frontend
```

---

## ⚙️ Backend Core Files (`backend/`)

### `backend/prisma/schema.prisma`
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id           String    @id @default(uuid())
  email        String    @unique
  username     String    @unique
  passwordHash String
  avatarUrl    String?
  bio          String?
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt

  posts        Post[]
  comments     Comment[]
  likes        Like[]

  @@map("users")
}

model Post {
  id        String    @id @default(uuid())
  title     String
  content   String
  category  String    @default("General")
  authorId  String
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  author    User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  comments  Comment[]
  likes     Like[]

  @@map("posts")
}

model Comment {
  id        String   @id @default(uuid())
  content   String
  authorId  String
  postId    String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)

  @@map("comments")
}

model Like {
  id        String   @id @default(uuid())
  userId    String
  postId    String
  createdAt DateTime @default(now())

  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)

  @@unique([userId, postId])
  @@map("likes")
}
```

### `backend/src/app.ts`
```typescript
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';
import routes from './routes/index.js';
import { errorHandler } from './middlewares/error.middleware.js';

export const createApp = () => {
  const app = express();

  app.use(helmet());
  app.use(cors());
  app.use(express.json());
  if (process.env.NODE_ENV !== 'test') {
    app.use(morgan('dev'));
  }

  app.get('/health', (_req, res) => {
    res.status(200).json({ status: 'ok', timestamp: new Date().toISOString() });
  });

  app.use('/api/v1', routes);
  app.use(errorHandler);

  return app;
};
```

---

## 🎨 Frontend Core Files (`frontend/`)

### `frontend/src/App.tsx`
```tsx
import React, { useState, useEffect } from 'react';
import { Navbar } from './components/Navbar';
import { Sidebar } from './components/Sidebar';
import { PostCard } from './components/PostCard';
import { CreatePostModal } from './components/CreatePostModal';
import { AuthModal } from './components/AuthModal';
import { Post } from './types';
import api from './services/api';
import { Compass, Flame, Sparkles } from 'lucide-react';

export const App: React.FC = () => {
  const [posts, setPosts] = useState<Post[]>([]);
  const [selectedCategory, setSelectedCategory] = useState('All');
  const [searchQuery, setSearchQuery] = useState('');
  const [isAuthOpen, setIsAuthOpen] = useState(false);
  const [isCreateOpen, setIsCreateOpen] = useState(false);

  const fetchPosts = async () => {
    try {
      const res = await api.get('/posts', {
        params: {
          category: selectedCategory === 'All' ? undefined : selectedCategory,
          search: searchQuery.trim() || undefined,
        },
      });
      if (res.data.data) {
        setPosts(res.data.data);
      }
    } catch (err) {
      console.log('Error fetching posts');
    }
  };

  useEffect(() => {
    fetchPosts();
  }, [selectedCategory, searchQuery]);

  return (
    <div className="min-h-screen bg-slate-900 text-slate-100 flex flex-col">
      <Navbar
        onOpenAuth={() => setIsAuthOpen(true)}
        onOpenCreatePost={() => setIsCreateOpen(true)}
        searchQuery={searchQuery}
        setSearchQuery={setSearchQuery}
      />

      <main className="flex-1 max-w-7xl w-full mx-auto px-4 py-6 flex gap-6">
        <Sidebar
          selectedCategory={selectedCategory}
          setSelectedCategory={setSelectedCategory}
        />

        <section className="flex-1 space-y-6">
          <div className="bg-gradient-to-r from-sky-900/40 via-indigo-900/30 to-purple-900/40 border border-slate-800 rounded-2xl p-6">
            <h1 className="text-2xl font-bold text-white">Discover & Share Content</h1>
            <p className="text-sm text-slate-300">Zwaylo modern open-source platform.</p>
          </div>

          <div className="space-y-4">
            {posts.map((post) => (
              <PostCard key={post.id} post={post} onPostUpdated={fetchPosts} />
            ))}
          </div>
        </section>
      </main>

      <AuthModal isOpen={isAuthOpen} onClose={() => setIsAuthOpen(false)} />
      <CreatePostModal
        isOpen={isCreateOpen}
        onClose={() => setIsCreateOpen(false)}
        onPostCreated={fetchPosts}
      />
    </div>
  );
};
```
