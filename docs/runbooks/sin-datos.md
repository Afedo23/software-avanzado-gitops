# Runbook · ObservabilidadSinDatosGateway (page)

**Significa:** Prometheus no ve `http_requests_total` del api-gateway hace 5 min. Los SLO y sus alertas están ciegos.

## Herramientas
- Grafana (`kubectl -n observabilidad port-forward svc/kube-prometheus-stack-grafana 3000:80`) → tablero **Operación — SA Platform**.
- Alertmanager (`kubectl -n observabilidad port-forward svc/kube-prometheus-stack-alertmanager 9093`).
- Explore → Loki / Tempo (datasources ya provisionados y correlacionados por `trace_id`).

## Diagnóstico
1. ¿Hay pods? `kubectl -n sa-p8 get pods -l app=api-gateway`. Si no, es una caída total: trate como incidente de disponibilidad (runbook de consumo rápido, paso 2).
2. ¿Prometheus lo descubre? Prometheus UI (`port-forward svc/prometheus-operated 9090`) → *Status → Targets* → `podMonitor/observabilidad/sa-servicios`. Si el target está *down*: `kubectl -n sa-p8 exec deploy/loadgen -- curl -s <pod-ip>:9464/metrics | head`.
3. ¿La app expone el puerto `metrics` (9464) y los nombres `http_requests_total` / `http_request_duration_seconds`? Cambios de nombre rompen todas las reglas.
4. ¿Prometheus vivo? `kubectl -n observabilidad get pods`.

## Mitigación
Restaurar el endpoint `/metrics` o el PodMonitor. Mientras tanto, monitoree a mano con `kubectl logs`.
