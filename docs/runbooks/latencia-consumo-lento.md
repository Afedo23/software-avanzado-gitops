# Runbook · SLOLatenciaConsumoLento (ticket)

**Significa:** degradación sostenida (burn rate > 6 durante 6 h y 30 min). Tendencia que agotaría el presupuesto de latencia en ~5 días. No despierta a nadie.
**Meta de atención:** ticket atendido el mismo día laboral.

## Herramientas
- Grafana (`kubectl -n observabilidad port-forward svc/kube-prometheus-stack-grafana 3000:80`) → tablero **Operación — SA Platform**.
- Alertmanager (`kubectl -n observabilidad port-forward svc/kube-prometheus-stack-alertmanager 9093`).
- Explore → Loki / Tempo (datasources ya provisionados y correlacionados por `trace_id`).

## Diagnóstico
1. Tablero fila 2: ¿desde cuándo? Compare con despliegues (`kubectl argo rollouts history`) y con la curva de tráfico (fila 3): un aumento de latencia con tráfico estable apunta a regresión; con tráfico creciente, a falta de capacidad.
2. Traza representativa: Tempo `{ duration > 500ms }`; identifique el span dominante.
3. Registre hallazgo y propuesta (optimizar consulta, subir réplicas, ajustar el SLO si el 500 ms resultó irreal — **justificando, nunca ajustando para que el número cuadre**).
