# Filtro global por país de origen

Selector 🌎 en la barra superior que segmenta las vistas del dashboard por país.

## Cómo funciona

- Cada vista se activa **sola** si sus filas traen `country_code` válido (código ISO
  de 2 letras). El guard es `hasValidCountry(rows, campo)`.
- Al elegir un país, las vistas soportadas **recalculan tablas y gráficos desde las
  filas** (`computeClientDist`, `renderTipKpisCharts`, `renderUniverse`).
- Las vistas sin país válido **no se vacían**: muestran un aviso y mantienen los
  totales globales. Nunca se muestran cifras engañosas.
- `mkChart(id, cfg)` mantiene un registro de charts y los destruye antes de
  recrearlos, evitando el error "Canvas is already in use" de Chart.js.

## Estado por vista

| Vista | Campo país | Filtra hoy |
|---|---|---|
| Clientes | `top_clients.country_code` | ✅ Sí |
| Universo | `universe.country_code` | ⚠️ No — dato inválido en el export |
| Tipologías | `typologies.country_of_residence` | ⚠️ No — dato inválido en el export |
| Resumen, Alertas, Flujos, Redes, Perfiles, Segmentos | — | ❌ Agregados sin país |

## Para habilitar el filtro en todas las vistas (repo del modelo `av-kyt-sfc`)

El frontend ya está listo; falta que el JSON traiga país válido. En el export dbt/Python:

1. **Bug `True`**: `universe.country_code` y `typologies.country_of_residence` están
   emitiendo un booleano `true` en vez del código de país. Revisar el `select` —
   probablemente un alias cruzado o un `case when ... then true`. Debe devolver el
   código ISO real del cliente.
2. **Limpieza**: `top_clients.country_code` tiene ~10 filas con `"True"`; corregir en
   origen.
3. **Nueva dimensión país** en los datasets agregados que se quieran filtrar
   (`alerts_detail`, `flows_monthly`, `mart_av_network`, y como `group by` adicional
   en `segment_stats`, `client_profiles`, `alerts_timeline`).

Cuando el JSON traiga `country_code` válido en un dataset, su vista se activa sin
cambios de código.
