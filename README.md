# config-server

Centralized configuration server for the microservices system. Serves shared
and per-service config from a local filesystem-backed repo, so settings like
Eureka's URL or Kafka's broker address live in one place instead of being
duplicated across every service's `application.yml`.

## How it works
- `config-repo/application.yml` — shared across ALL client services
  (Eureka client settings, actuator exposure, tracing config)
- `config-repo/<service-name>.yml` — specific to one service (port,
  datasource, Kafka topics, AI model config, gateway routes), merged on
  top of the shared file

Each client service's own local `application.yml` only contains:
```yaml
spring:
  application:
    name: <service-name>
  config:
    import: "optional:configserver:http://localhost:8888"
```
The `optional:` prefix means a client still starts (using local-only config)
even if this server is down, rather than refusing to boot.

## Run
    mvn spring-boot:run
Runs on port 8888. **Start this before every other service** except
`eureka-server`, since clients fetch their config from here at startup.

## Verify
    GET http://localhost:8888/<service-name>/default
Returns the merged JSON config for that service.

## Tech
Spring Boot 4.1, Spring Cloud 2025.1.2 (Config Server, native/filesystem backend).

## Exceptions
`eureka-server` and this service itself do NOT import config from here —
both keep full local config, to avoid a circular startup dependency
(each needing the other before it can configure itself).
