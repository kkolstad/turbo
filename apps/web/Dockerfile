# This Dockerfile is copy-pasted into our main docs at /docs/handbook/deploying-with-docker.
# Make sure you update both files!

FROM node:alpine AS base
RUN apk add --no-cache libc6-compat
RUN apk update
RUN npm install -g pnpm turbo



FROM base AS deps
WORKDIR /app
# COPY ./pnpm-lock.yaml .npmrc ./
COPY ./pnpm-lock.yaml ./
RUN pnpm fetch


FROM deps AS builder
COPY . .
RUN pnpm recursive install --offline --frozen-lockfile
RUN pnpm build

RUN ls -al /app/apps/web/.next
RUN ls -al /app/apps/web/.next/standalone

FROM node:alpine AS runner
WORKDIR /app

# Don't run production as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

COPY --from=builder /app/apps/web/next.config.js .
COPY --from=builder /app/apps/web/package.json .

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static
# COPY --from=builder --chown=nextjs:nodejs /app/apps/web/public ./apps/web/public

CMD node apps/web/server.js
