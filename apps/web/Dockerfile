# Use a specific version of Node
FROM node:18-alpine AS base

# Set up environment for pnpm
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"

# Install corepack, Turbo, and OpenSSL
RUN apk add --no-cache libc6-compat openssl \
    && apk update \
    && corepack enable \
    && pnpm i -g turbo@2.3.4

# Create a builder stage
FROM base AS builder

# Install libc6-compat and OpenSSL (required for some dependencies)
RUN apk add --no-cache libc6-compat openssl \
    && apk update

# Set working directory and copy project files
WORKDIR /app
COPY . .

# Prune dependencies using Turbo
RUN turbo prune web --docker

# Create an installer stage
FROM base AS installer

# Install libc6-compat and OpenSSL (required for some dependencies)
RUN apk add --no-cache libc6-compat openssl \
    && apk update

# Set working directory and copy project files
WORKDIR /app
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/pnpm-lock.yaml ./pnpm-lock.yaml
COPY --from=builder /app/out/pnpm-workspace.yaml ./pnpm-workspace.yaml

# Copy project files from the builder stage
COPY --from=builder /app/out/full/ .

# Install dependencies using pnpm
RUN corepack enable \
    && pnpm install -w turbo@2.3.4 && pnpm install

# Build the project
RUN pnpx turbo run build --filter web

# Create a runner stage
FROM base AS runner

# Set working directory
WORKDIR /app

# Add a system group and user for running the app
RUN addgroup --system --gid 1001 nodejs \
    && adduser --system --uid 1001 nextjs

# Copy project files from the installer stage
COPY --from=installer --chown=nextjs:nodejs /app/ .

RUN find /app -type f ! -path "*/node_modules/*"

# Switch to the nextjs user and set the default command
USER nextjs
CMD ["node", "/app/apps/web/.next/standalone/apps/web/server.js"]