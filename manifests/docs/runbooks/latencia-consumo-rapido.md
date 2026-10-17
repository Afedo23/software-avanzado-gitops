# Runbook · SLOLatenciaConsumoRapido (page)

**Significa:** más del 72 % de las peticiones del borde superan 500 ms en la última hora y sigue ocurriendo ahora (burn rate > 14.4 sobre un presupuesto del 5 %). El usuario percibe el sistema como caído aunque responda 200.
**Meta de atención:** causa identificada en < 3 min.

## Herramientas
- Grafana (`kubectl -n observabilidad port-forward svc/kube-prometheus-stack-grafana 3000:80`) → tablero **Operación — SA Platform**.
- Alertmanager (`kubectl -n observabilidad port-forward svc/kube-prometheus-stack-alertmanager 9093`).
- Explore → Loki / Tempo (datasources ya provisionados y correlacionados por `trace_id`).

## Diagnóstico
1. **Tablero, fila 2:** ¿sube solo el gateway o también auth/solicitudes? Si sube solo el gateway → problema local (CPU, GC); si suben los internos → la causa está aguas abajo.
2. **Fila 5 (saturación):** CPU cercana a 100 % del límite = throttling; memoria cercana a 100 % = GC/OOM inminente.
3. **Traza lenta (Explore → Tempo):** busque `{ duration > 500ms }` filtrando por `resource.service.name`. El span con mayor duración propia (no la suma de hijos) es el cuello de botella.
4. **Registros de esa traza:** `{service_name="<svc>"} | trace_id="<id>"`.
5. ¿Despliegue reciente? `kubectl argo rollouts get rollout <svc> -n sa-p8`.

## Mitigación
- Versión nueva lenta: `kubectl argo rollouts abort <svc> -n sa-p8` + revertir el commit en GitOps.
- Saturación: subir `replicas` / límites en el manifiesto vía PR.
- Dependencia lenta (BD): ver `P9/docs/runbook.md`.
