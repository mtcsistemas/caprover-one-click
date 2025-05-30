SigNoz One-Click App for CapRover

Sigue estos pasos para desplegar SigNoz en tu servidor CapRover:

Copia la carpeta signoz-oneclick en un repositorio Git accesible por tu servidor.

En tu dashboard de CapRover, navega a Apps > One-Click Apps > Add App.

Rellena los campos:

App Name: signoz (o el nombre que prefieras)

Git Repository: URL de tu repositorio

Branch: main (o el branch donde subiste los archivos)

Captain Definition Path: signoz-oneclick/captain-definition.json

Haz clic en Save & Update. CapRover descargará el repo y levantará los servicios definidos.

Durante el despliegue CapRover inicializará ClickHouse con el script de histogram-quantile, levantará Zookeeper, ClickHouse, SigNoz y el Otel Collector.

Una vez completado, podrás acceder a:

Frontend de SigNoz: http://<your-captain-root-domain>:3301

Backend API (OpenTelemetry ingest): http://<your-captain-root-domain>:8080