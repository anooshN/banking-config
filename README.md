# Banking Config Repository

Centralized configuration for all Banking App microservices, served by Spring Cloud Config Server.

## Structure

```
banking-config/
├── application.yml              ← shared defaults for ALL services
├── application-dev.yml          ← shared overrides for dev profile
├── application-prod.yml         ← shared overrides for prod profile
│
├── auth-service/
│   ├── auth-service.yml         ← auth-service defaults
│   ├── auth-service-dev.yml     ← auth-service dev overrides
│   └── auth-service-prod.yml    ← auth-service prod overrides
│
├── account-service/
│   ├── account-service.yml
│   ├── account-service-dev.yml
│   └── account-service-prod.yml
│
... (same pattern for all 14 services)
```

## How the Config Server Reads This

When auth-service starts with `spring.profiles.active=dev`, the config server returns:
1. `application.yml` (shared defaults)
2. `application-dev.yml` (shared dev overrides)
3. `auth-service/auth-service.yml` (service-specific defaults)
4. `auth-service/auth-service-dev.yml` (service-specific dev overrides)

Later files override earlier ones.

## Encrypted Values

Sensitive values (passwords, secrets) are stored encrypted:
```yaml
spring:
  datasource:
    password: '{cipher}AQB3x7...'   # encrypted with config server's encrypt.key
```

To encrypt a value:
```bash
curl -X POST http://localhost:8888/encrypt -d 'my-secret-password'
```

To decrypt (automatic when config server serves the value):
```bash
curl http://localhost:8888/auth-service/dev
```

## Adding a New Service

1. Create a folder: `mkdir new-service`
2. Add `new-service/new-service.yml` with service-specific config
3. Add `new-service/new-service-dev.yml` and `new-service/new-service-prod.yml`
4. The service automatically picks up config on startup — no config-server restart needed

## Security

This repository should be **private** in production. The config-server authenticates to GitHub using a deploy key or personal access token set via `CONFIG_GIT_USERNAME` / `CONFIG_GIT_PASSWORD` environment variables.
