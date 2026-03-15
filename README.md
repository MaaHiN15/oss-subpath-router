# OSS Subpath Router

Single nginx reverse proxy serving multiple OSS tools at subpaths. All services are accessible at `http://localhost/<service>/` through a unified nginx gateway running on port 80.

## Services

| Service | Image | Subpath | Internal Port |
|---|---|---|---|
| Alertmanager | `prom/alertmanager:v0.31.1` | `/alertmanager/` | 9093 |
| Consul | `hashicorp/consul:1.22` | `/consul/` | 8500 |
| Grafana | `grafana/grafana:12.3` | `/grafana/` | 3000 |
| Jenkins | `jenkins/jenkins:2.554` | `/jenkins/` | 8080 |
| Kafka UI | `provectuslabs/kafka-ui:v0.7.2` | `/kafka/` | 8080 |
| MinIO Console | `minio/minio:RELEASE.2025-09-07T16-13-09Z` | `/minio/` | 9001 |
| Nexus | `sonatype/nexus3:3.90.1` | `/nexus/` | 8081 |
| Prometheus | `prom/prometheus:v3.10.0` | `/prometheus/` | 9090 |
| Temporal UI | `temporalio/ui:2.47.3` | `/temporal/` | 8080 |
| Traefik Dashboard | `traefik:v3.6` | `/dashboard/` | 8080 |

## Quick Start

Start all services together behind nginx:

```bash
docker compose up -d
```

Check that nginx and all services are running:

```bash
docker compose ps
```

## Subpath Configuration Notes

Each service requires specific configuration to work correctly behind a subpath proxy.

### Alertmanager
- `--web.external-url=http://localhost/alertmanager` — tells Alertmanager its public URL so links and redirects are correct.

### Consul
- `-ui-content-path=/consul/` — sets the UI content path so the UI assets load from the correct subpath.

### Grafana
- `GF_SERVER_ROOT_URL=%(protocol)s://%(domain)s/grafana` — sets the root URL for correct link generation.
- `GF_SERVER_SERVE_FROM_SUB_PATH=true` — enables serving from a subpath prefix.

### Jenkins
- `JENKINS_OPTS=--prefix=/jenkins` — configures the Jenkins context path so all URLs and redirects use `/jenkins`.

### Kafka UI
- `SERVER_SERVLET_CONTEXT_PATH=/kafka` — sets the servlet context path for the Spring Boot application.

### MinIO
- `MINIO_BROWSER_REDIRECT_URL=http://localhost/minio` — sets the browser redirect URL for the console.
- nginx rewrites `/minio/(.*)` to `/$1` before proxying to the console port (9001).

### Nexus
- `INSTALL4J_ADD_VM_PARAMS=-Dnexus-context-path=/nexus` — sets the Nexus context path via JVM property.

### Prometheus
- `--web.external-url=http://localhost/prometheus` — sets the external URL for correct link generation.
- `--web.route-prefix=/` — keeps internal routing at `/` while the external URL includes the subpath.
- nginx rewrites `/prometheus/(.*)` to `/$1` before proxying.

### Temporal UI
- nginx proxies `/temporal/` to `http://temporal:8080/` (trailing slash strips the prefix).
- Set `TEMPORAL_ADDRESS` to point to your Temporal server frontend gRPC address.

### Traefik
- `--api=true` and `--api.dashboard=true` — enable the API and dashboard.
- Dashboard accessible at `/dashboard/` and API at `/api/`.

## Directory Structure

```
.
├── docker-compose.yml              # root compose — all services + nginx on oss-net
├── nginx.conf                      # nginx http block
├── ops.nginx.conf                  # combined server block with all location blocks
├── <service>/
│   ├── <service>.conf              # standalone nginx server block
│   ├── docker-compose.yml          # standalone compose
│   └── config/
│       └── <alertmanager>.yml      # service configuration
```
