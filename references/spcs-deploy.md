# SPCS Deployment Reference

## Prerequisites

- Docker installed and running
- Snowflake connection configured in `~/.snowflake/config.toml`
- Image repository exists in the target database/schema
- Compute pool accessible to the service-owning role

## Docker Registry Login

The `snow spcs image-registry login` command may hang. Use this Python fallback:

```bash
python3 -c '
import subprocess, tomllib, os
f = open(os.path.expanduser("~/.snowflake/config.toml"), "rb")
c = tomllib.load(f)["connections"]["<CONNECTION_NAME>"]
subprocess.run(
    ["docker", "login",
     "<ACCOUNT>.registry.snowflakecomputing.com",
     "-u", c["user"], "--password-stdin"],
    input=c["password"], text=True
)'
```

Replace `<CONNECTION_NAME>` and `<ACCOUNT>` with actual values.

## Build and Push

```bash
# Build for linux/amd64 (SPCS requires this platform)
docker build --platform linux/amd64 \
  -t <ACCOUNT>.registry.snowflakecomputing.com/<DB>/<SCHEMA>/<REPO>/<IMAGE>:latest .

# Push to Snowflake registry
docker push <ACCOUNT>.registry.snowflakecomputing.com/<DB>/<SCHEMA>/<REPO>/<IMAGE>:latest
```

## Create Image Repository (if needed)

```sql
CREATE IMAGE REPOSITORY IF NOT EXISTS <DB>.<SCHEMA>.<REPO_NAME>;
GRANT READ ON IMAGE REPOSITORY <DB>.<SCHEMA>.<REPO_NAME> TO ROLE <SERVICE_ROLE>;
```

## Service Specification

```yaml
spec:
  containers:
  - name: app
    image: /<DB>/<SCHEMA>/<REPO>/<IMAGE>:latest
    resources:
      requests:
        cpu: "0.25"
        memory: 256M
      limits:
        cpu: "0.5"
        memory: 512M
    env:
      SNOWFLAKE_ACCOUNT: <ACCOUNT>
      SNOWFLAKE_WAREHOUSE: <WAREHOUSE>
      SNOWFLAKE_DATABASE: <DATABASE>
      SNOWFLAKE_SCHEMA: <SCHEMA>
      NODE_ENV: production
  endpoints:
  - name: app
    port: 8080
    public: true
  networkPolicyConfig:
    allowInternetEgress: true
```

## First Deploy (CREATE SERVICE)

```sql
CREATE SERVICE <DB>.<SCHEMA>.<SERVICE_NAME>
  IN COMPUTE POOL <POOL_NAME>
  FROM SPECIFICATION $$ <yaml-spec> $$
  MIN_INSTANCES = 1
  MAX_INSTANCES = 1;
```

## Update Deploy (ALTER SERVICE — preserves URL)

**CRITICAL: Always use ALTER SERVICE for updates. DROP + CREATE changes the endpoint URL.**

```sql
ALTER SERVICE <DB>.<SCHEMA>.<SERVICE_NAME> FROM SPECIFICATION $$ <yaml-spec> $$;
```

## Grant Access

```sql
-- Grant service endpoint access to consumer role
GRANT SERVICE ROLE <DB>.<SCHEMA>.<SERVICE_NAME>!ALL_ENDPOINTS_USAGE TO ROLE <CONSUMER_ROLE>;

-- Grant table access if service writes data
GRANT SELECT, INSERT, UPDATE ON TABLE <DB>.<SCHEMA>.<TABLE> TO ROLE <SERVICE_ROLE>;
```

## Verify Deployment

```sql
-- Check service status
SELECT SYSTEM$GET_SERVICE_STATUS('<DB>.<SCHEMA>.<SERVICE_NAME>');

-- Get endpoint URL
SHOW ENDPOINTS IN SERVICE <DB>.<SCHEMA>.<SERVICE_NAME>;

-- View logs if issues
SELECT SYSTEM$GET_SERVICE_LOGS('<DB>.<SCHEMA>.<SERVICE_NAME>', 0, 'app');
```

## Common Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| DNS resolution failure | `accessUrl` set in SPCS | Remove `accessUrl` when SPCS token detected |
| Service won't start | Compute pool wrong type | Use CPU_X64_XS pools, not SYSTEM_COMPUTE_POOL |
| Image not found | Wrong image path in spec | Must include leading `/` and match repo exactly |
| Permission denied | Service role missing grants | Grant on image repo, tables, and service endpoint |
| OAuth token expired | Connection not refreshing | Implement token change detection and reconnect |
