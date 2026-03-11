# Tech Stack Reference

## Package Dependencies

```json
{
  "dependencies": {
    "next": "^16.1.6",
    "react": "^19.2.4",
    "react-dom": "^19.2.4",
    "snowflake-sdk": "^2.3.4"
  },
  "devDependencies": {
    "@tailwindcss/postcss": "^4.2.1",
    "@types/node": "^25.4.0",
    "@types/react": "^19.2.14",
    "daisyui": "^5.5.19",
    "postcss": "^8.5.8",
    "tailwindcss": "^4.2.1",
    "typescript": "^5.9.3"
  }
}
```

## next.config.ts

```typescript
import type { NextConfig } from "next";
const nextConfig: NextConfig = { output: "standalone" };
export default nextConfig;
```

`output: "standalone"` is required for Docker/SPCS deployment. Do NOT add `serverExternalPackages` — snowflake-sdk works without it in Next.js 16.

## postcss.config.mjs

```javascript
const config = { plugins: { "@tailwindcss/postcss": {} } };
export default config;
```

## Snowflake Connection (src/lib/snowflake.ts)

Key patterns:
- **SPCS detection**: Check if `/snowflake/session/token` exists. If yes, use OAuth.
- **CRITICAL**: When in SPCS, do NOT set `accessUrl`. The SDK resolves internally via account name. Setting the external hostname causes DNS resolution failure inside the container.
- **Token refresh**: Cache the token, reconnect when it changes.
- **Retry logic**: Retry once on OAuth expiry or terminated connection errors.
- **Local dev**: Use `EXTERNALBROWSER` authenticator (SSO popup).

```typescript
import snowflake from "snowflake-sdk";
import fs from "fs";

snowflake.configure({ logLevel: "ERROR" });

let connection: snowflake.Connection | null = null;
let cachedToken: string | null = null;

function getSPCSToken(): string | null {
  try {
    if (fs.existsSync("/snowflake/session/token")) {
      return fs.readFileSync("/snowflake/session/token", "utf8");
    }
  } catch { /* not in SPCS */ }
  return null;
}

function getConfig(): snowflake.ConnectionOptions {
  const base = {
    account: process.env.SNOWFLAKE_ACCOUNT || "",
    warehouse: process.env.SNOWFLAKE_WAREHOUSE || "",
    database: process.env.SNOWFLAKE_DATABASE || "",
    schema: process.env.SNOWFLAKE_SCHEMA || "",
  };
  const token = getSPCSToken();
  if (token) {
    // SPCS: OAuth token, NO accessUrl (internal DNS)
    return { ...base, token, authenticator: "oauth" };
  }
  // Local dev: browser SSO
  return { ...base, username: process.env.SNOWFLAKE_USER || "", authenticator: "EXTERNALBROWSER" };
}

async function getConnection(): Promise<snowflake.Connection> {
  const token = getSPCSToken();
  if (connection && (!token || token === cachedToken)) return connection;
  if (connection) { connection.destroy(() => {}); }
  const conn = snowflake.createConnection(getConfig());
  await conn.connectAsync(() => {});
  connection = conn;
  cachedToken = token;
  return connection;
}

export async function query<T>(sql: string, retries = 1): Promise<T[]> {
  try {
    const conn = await getConnection();
    return await new Promise<T[]>((resolve, reject) => {
      conn.execute({
        sqlText: sql,
        complete: (err, _stmt, rows) => err ? reject(err) : resolve((rows || []) as T[]),
      });
    });
  } catch (err) {
    const msg = (err as Error).message || "";
    if (retries > 0 && (msg.includes("OAuth access token expired") || msg.includes("terminated"))) {
      connection = null;
      return query(sql, retries - 1);
    }
    throw err;
  }
}
```

## Dockerfile

```dockerfile
FROM node:22-alpine AS base

FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci --ignore-scripts

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production
ENV HOSTNAME="0.0.0.0"
ENV PORT=8080
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
USER nextjs
EXPOSE 8080
CMD ["node", "server.js"]
```

## .dockerignore

```
node_modules
.next
.git
.env
*.md
```
