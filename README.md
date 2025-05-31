// README.md
# SigNoz One-Click App for CapRover

Sigue estos pasos para desplegar SigNoz en tu servidor CapRover:

1. Copia la carpeta `signoz-oneclick` en un repositorio Git accesible por tu servidor.
2. En tu dashboard de CapRover, navega a **Apps > One-Click Apps > Add App**.
3. Rellena los campos:
   - **App Name**: `signoz` (o el nombre que prefieras)
   - **Git Repository**: URL de tu repositorio
   - **Branch**: `main` (o el branch donde subiste los archivos)
   - **Captain Definition Path**: `signoz-oneclick/captain-definition.json`
4. Haz clic en **Save & Update**. CapRover descargará el repo y levantará los servicios definidos.

Durante el despliegue CapRover inicializará ClickHouse con el script de histogram-quantile, levantará Zookeeper, ClickHouse, SigNoz y el Otel Collector.

Una vez completado, podrás acceder a:
- **Frontend de SigNoz**: `http://<your-captain-root-domain>:3301`
- **Backend API (OpenTelemetry ingest)**: `http://<your-captain-root-domain>:8080`

¡Y eso es todo! Ahora tienes SigNoz corriendo con un solo clic en CapRover.

---

# Infraestructura Complementaria

Para desplegar agentes de infraestructura, crea otro archivo `infra-docker-compose.yml`:

```yaml
# File: infra-docker-compose.yml
version: "3"
x-common: &common
  networks:
    - signoz-net
  extra_hosts:
    - host.docker.internal:host-gateway
  logging:
    options:
      max-size: 50m
      max-file: "3"
  restart: unless-stopped
services:
  otel-agent:
    <<: *common
    image: otel/opentelemetry-collector-contrib:0.111.0
    command:
      - --config=/etc/otel-collector-config.yaml
    volumes:
      - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml
      - /:/hostfs:ro
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - SIGNOZ_COLLECTOR_ENDPOINT=http://host.docker.internal:4317
      - OTEL_RESOURCE_ATTRIBUTES=host.name=signoz-host,os.type=linux
  logspout:
    <<: *common
    image: "gliderlabs/logspout:v3.2.14"
    volumes:
      - /etc/hostname:/etc/host_hostname:ro
      - /var/run/docker.sock:/var/run/docker.sock
    command: syslog+tcp://otel-agent:2255
    depends_on:
      - otel-agent

networks:
  signoz-net:
    name: signoz-net
    external: true