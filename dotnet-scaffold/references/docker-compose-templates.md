# Docker Compose Service Templates

Use these snippets to build the `docker-compose.yml` based on the user's infrastructure choices.
Combine only the services the user selected. Always include the network definition at the bottom.

---

## Base Structure

```yaml
version: "3.9"

services:
  # ... services go here based on user choices

networks:
  app-net:
    driver: bridge

volumes:
  # ... named volumes for each service that needs persistence
```

---

## PostgreSQL

```yaml
  postgres:
    image: postgres:16-alpine
    container_name: app-postgres
    restart: unless-stopped
    ports:
      - "${POSTGRES_PORT:-5432}:5432"
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-appuser}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-apppass}
      POSTGRES_DB: ${POSTGRES_DB:-appdb}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-appuser}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-net
```

Volume: `postgres_data:`

---

## Redis

```yaml
  redis:
    image: redis:7-alpine
    container_name: app-redis
    restart: unless-stopped
    ports:
      - "${REDIS_PORT:-6379}:6379"
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD:-redispass}
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD:-redispass}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-net
```

Volume: `redis_data:`

---

## RabbitMQ

```yaml
  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: app-rabbitmq
    restart: unless-stopped
    ports:
      - "${RABBITMQ_PORT:-5672}:5672"
      - "${RABBITMQ_MGMT_PORT:-15672}:15672"
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER:-guest}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD:-guest}
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "-q", "ping"]
      interval: 15s
      timeout: 10s
      retries: 5
    networks:
      - app-net
```

Volume: `rabbitmq_data:`

Management UI: http://localhost:15672

---

## ElasticSearch

```yaml
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.13.0
    container_name: app-elasticsearch
    restart: unless-stopped
    ports:
      - "${ELASTIC_PORT:-9200}:9200"
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    healthcheck:
      test: ["CMD-SHELL", "curl -sf http://localhost:9200/_cluster/health || exit 1"]
      interval: 15s
      timeout: 10s
      retries: 5
    networks:
      - app-net
```

Volume: `elasticsearch_data:`

---

## Grafana + Loki

```yaml
  loki:
    image: grafana/loki:2.9.0
    container_name: app-loki
    restart: unless-stopped
    ports:
      - "${LOKI_PORT:-3100}:3100"
    command: -config.file=/etc/loki/local-config.yaml
    volumes:
      - loki_data:/loki
    healthcheck:
      test: ["CMD-SHELL", "wget --spider -q http://localhost:3100/ready || exit 1"]
      interval: 15s
      timeout: 10s
      retries: 5
    networks:
      - app-net

  grafana:
    image: grafana/grafana:10.4.0
    container_name: app-grafana
    restart: unless-stopped
    ports:
      - "${GRAFANA_PORT:-3000}:3000"
    environment:
      GF_SECURITY_ADMIN_USER: ${GRAFANA_USER:-admin}
      GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_PASSWORD:-admin}
    volumes:
      - grafana_data:/var/lib/grafana
      - ./docker/grafana/provisioning:/etc/grafana/provisioning
    depends_on:
      loki:
        condition: service_healthy
    healthcheck:
      test: ["CMD-SHELL", "wget --spider -q http://localhost:3000/api/health || exit 1"]
      interval: 15s
      timeout: 10s
      retries: 5
    networks:
      - app-net
```

Volumes: `loki_data:`, `grafana_data:`

Grafana UI: http://localhost:3000

### Grafana Provisioning (Loki Datasource)

Create `docker/grafana/provisioning/datasources/loki.yml`:

```yaml
apiVersion: 1
datasources:
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    isDefault: true
```

---

## Port Summary

| Service       | Default Port | Env Var             |
|---------------|-------------|---------------------|
| PostgreSQL    | 5432        | POSTGRES_PORT       |
| Redis         | 6379        | REDIS_PORT          |
| RabbitMQ      | 5672        | RABBITMQ_PORT       |
| RabbitMQ Mgmt | 15672      | RABBITMQ_MGMT_PORT  |
| ElasticSearch | 9200        | ELASTIC_PORT        |
| Loki          | 3100        | LOKI_PORT           |
| Grafana       | 3000        | GRAFANA_PORT        |
