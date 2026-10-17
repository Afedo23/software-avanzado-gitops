# Objetivos de nivel de servicio (SLO)

Ambos se miden en el **borde (api-gateway)**: es lo que el usuario experimenta. `/health` y `/metrics` se excluyen para no diluir el indicador. Reglas en `platform/observability/prometheusrule-slo.yaml` (repo GitOps).

| | SLO 1 · Disponibilidad | SLO 2 · Latencia |
|---|---|---|
| Objetivo | 99.5 % de peticiones sin 5xx | 95 % de peticiones en ≤ 500 ms |
| Ventana | 30 días móviles | 30 días móviles |
| Presupuesto de error | 0.5 % (≈ 3 h 36 min de fallo total al mes) | 5 % de peticiones lentas |
| SLI (regla de registro) | `sli:disponibilidad:ratio_rate5m` | `sli:latencia:ratio_rate5m` |
| Presupuesto restante | `slo:disponibilidad:error_budget_remaining:ratio30d` | `slo:latencia:error_budget_remaining:ratio30d` |

## Justificación — completar con SUS datos
- **Por qué 99.5 % y no 99.9 %:** <el sistema corre en pocos nodos (P9 midió RTO de ...); 99.9 % (≈ 43 min/mes) no es defendible con su RTO medido>.
- **Por qué 500 ms:** <qué latencia p95 midió k6 en P8 y a partir de cuándo el usuario nota lentitud>.
- **Alertas:** burn rate 14.4× (1 h + 5 min) = página; 6× (6 h + 30 min) = ticket. Con 14.4× se consume el 2 % del presupuesto mensual en 1 hora.
