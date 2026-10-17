# Runbook · SLODisponibilidadConsumoLento (ticket)

**Significa:** la disponibilidad se degrada de forma sostenida (burn rate > 6 durante 6 h y 30 min). Agotaría el presupuesto en ~5 días.
**Meta de atención:** ticket atendido el mismo día laboral.

## Herramientas
- Grafana (`kubectl -n observabilidad port-forward svc/kube-prometheus-stack-grafana 3000:80`) → tablero **Operación — SA Platform**.
- Alertmanager (`kubectl -n observabilidad port-forward svc/kube-prometheus-stack-alertmanager 9093`).
- Explore → Loki / Tempo (datasources ya provisionados y correlacionados por `trace_id`).

## Diagnóstico (del síntoma a la causa)
1. **Tablero, fila 4:** ¿qué servicio tiene más tasa 5xx? ¿en qué ruta? Anote la hora de inicio (T0).
2. **¿Hubo un despliegue?** `kubectl argo rollouts list rollouts -n sa-p8` y `kubectl argo rollouts get rollout <svc> -n sa-p8`. Un canary en curso con AnalysisRun fallando → ya se está revirtiendo solo; verifique con `kubectl get analysisrun -n sa-p8`.
3. **Registro del error (Grafana → Explore → Loki):** `{service_name="<svc>"} | json | level="error"`. Copie el `trace_id` de un registro.
4. **Traza (Explore → Tempo):** pegue el `trace_id`. El span en rojo más profundo es la causa (servicio, operación, mensaje de excepción). Desde el span use *Logs for this span* para volver al registro.
5. Confirme con `kubectl -n sa-p8 get pods` y `kubectl -n sa-p8 logs deploy/<svc>` solo si falta detalle.

## Mitigación
- **Causa = versión nueva:** `kubectl argo rollouts abort <svc> -n sa-p8` y, en el repo GitOps, revierta el commit del tag (GitOps sigue siendo la fuente de verdad; no deje el clúster divergente).
- **Causa = dependencia (BD / auth):** restaure la dependencia; consulte runbook de P9 (`P9/docs/runbook.md`) si es el ledger.
- **Causa = saturación:** fila 5 del tablero; escale réplicas en el manifiesto y suba el PR.

## Cierre
Registre T0, hora de la alerta, hora de causa y hora de recuperación en la evidencia (`P10/evidencia/`). Si la alerta tardó más de lo debido, anótelo en "Alerta faltante o sobrante" del informe.

## Notas específicas del consumo lento
Suele ser un fallo parcial y constante (una ruta, un pod, un cliente concreto) que no justifica despertar a nadie. Busque la ruta con \`sum by (route)(rate(http_requests_total{status_code=~"5.."}[6h]))\` y cree un ticket con la traza de ejemplo adjunta.
