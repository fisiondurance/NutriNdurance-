# NutriNdurance — Sistema de planificación nutricional para deportistas de resistencia

## Alcance de este directorio

Este directorio alberga el **proyecto práctico**: código, datos, modelos, interfaz, tests y documentación técnica.

El archivo [PROJECT.md](PROJECT.md) contiene la especificación íntegra: reglas fisiológicas detalladas, tablas de clasificación de días, protocolos pre-competición, modelo de datos y decisiones de diseño. Consúltalo cuando necesites detalle que no esté en este CLAUDE.md.

## Qué hace el sistema

A partir del perfil del deportista y su plan semanal, genera un plan nutricional individualizado: kcal y macros por día, distribución en comidas según hora de entreno, pauta intra-entreno, protocolo pre-competición y feedback post-sesión.

Público objetivo: nutricionistas deportivos, entrenadores, fisioterapeutas y deportistas de resistencia.

## Arquitectura: 5 módulos

| Módulo | Función | Prioridad |
|---|---|---|
| M1 | Planificación diaria (kcal + macros por tipo de día) | Núcleo |
| M2 | Distribución en comidas según hora de entreno | Segundo nivel |
| M3 | Recomendación intra-entreno (CHO/h, líquidos, sodio, riesgo GI) | Núcleo |
| M4 | Protocolo pre-competición (D-3 a Día D) | Segundo nivel |
| M5 | Feedback del deportista con reglas de ajuste | Tercer nivel |

Flujo: Plan semanal → M1 → M4 (si competición) → M2 → M3 → M5 (feedback).

## Motor dual: reglas + ML

- **Capa 1 — Reglas**: funciones Python con trazabilidad bibliográfica. Fuente de verdad.
- **Capa 2 — ML**: entrenado sobre dataset sintético generado desde las reglas + variabilidad controlada. Comparado contra baseline de reglas.
- **Detección de discrepancias**: si ML y reglas difieren, se señala.
- **Explicabilidad**: SHAP sobre los modelos.
- **Edición libre con alertas**: verde/amarillo/rojo. Nunca bloquear. El profesional decide.

## Stack

Python · SQLAlchemy + SQLite · scikit-learn + XGBoost + LightGBM · SHAP · Plotly · Streamlit · Jinja2 · python-docx · fitparse · pytest · Git.

## Estructura del proyecto

```
NutriNdurance/
├── data/                   # Crudo, read-only (alimentos.csv, perfiles_demo.json)
├── artifacts/              # Generado, gitignored salvo .gitkeep
│   ├── datasets/           # Dataset sintético generado
│   ├── models/             # .pkl de modelos ML entrenados
│   └── exports/            # HTML + .docx generados
├── src/
│   ├── core/               # dataclasses: perfil, sesion, plan
│   ├── rules/              # un archivo por módulo + config/macros.yaml (rangos editables)
│   ├── ml/                 # dataset_generator, train, evaluate, shap_explain
│   ├── db/                 # models, database (SQLAlchemy)
│   ├── fit/                # fit_parser
│   ├── export/
│   │   ├── html_generator.py
│   │   ├── docx_generator.py
│   │   └── templates/      # Jinja2 (dentro del módulo que las consume)
│   └── ui/                 # app.py Streamlit + components/
├── notebooks/              # EDA
├── tests/                  # pytest (plana: test_modulo1.py, test_modulo3.py, ...)
└── docs/
```

## Módulo 1 — contrato de salida

Cada recomendación diaria devuelve un dict con esta forma:

```python
{
    "fecha": str,
    "tipo_dia": str,              # descanso|ligero|moderado|intenso|largo|competicion
    "kcal_objetivo": float,       # TDEE ajustado por objetivo
    "kcal_resultantes": float,    # de los macros finales
    "diferencia_kcal": float,
    "alerta": str,                # verde|amarillo|rojo (sobre kcal totales)
    "macros": {
        "cho": {"min": float, "max": float, "sugerido": float,
                "final": float, "g_total": float, "kcal": float,
                "editado": bool, "justificacion": str | None},
        "pro": {...},
        "fat": {...}
    }
}
```

- `min/max/sugerido` vienen de `src/rules/config/macros.yaml` (tablas por objetivo × tipo_dia, editables sin tocar código).
- `final` = `sugerido` si no se edita; valor del profesional si edita.
- El profesional puede fijar `final` fuera del rango `min/max`. La alerta lo refleja, nunca se bloquea.

**Umbrales de alerta sobre kcal totales:**

| Nivel | Condición |
|---|---|
| Verde | \|diferencia_kcal\| ≤ 5% de `kcal_objetivo` |
| Amarillo | 5-15% (ajuste consciente: déficit o superávit intencional) |
| Rojo | >15% **o** `kcal_resultantes < BMR × 1.1` (safety rail LEA/RED-S) |

## Diseño API-ready

Las funciones del núcleo (reglas, ML, cálculos) reciben y devuelven dataclasses o dicts serializables a JSON. Streamlit es capa fina. Debe poder envolverse en FastAPI sin reescribir la lógica.

## Priorización (entregar por capas)

1. **Núcleo**: M1 + M3, motor de reglas completo para ambos, dataset sintético + pipeline ML + evaluación, prototipo Streamlit, Git.
2. **Segundo nivel**: M2 + M4, BD de 80 alimentos, dashboard Plotly, export HTML + .docx, SHAP, SQLite + SQLAlchemy.
3. **Tercer nivel**: M5, Strava, .FIT, motor dual con discrepancias, Docker, perfiles demo, alertas de edición.

No empezar por el tercer nivel hasta que los dos primeros estén cerrados.

## Dataset sintético

No existe dataset público de planificación nutricional individualizada. Se genera desde reglas fisiológicas + variabilidad controlada (ruido gaussiano, tolerancia individual, interacciones no lineales). Se declara explícitamente como sintético — es defendible, no se presenta como datos reales.

Targets:
- Regresión: kcal/día, CHO/PRO/FAT g/día, CHO/h intra, líquidos/h, sodio/h.
- Clasificación: tipo de día, riesgo GI.

Modelos a comparar: baseline reglas, regresión lineal, decision tree, random forest, XGBoost/LightGBM. Métricas: MAE/RMSE/R² (regresión), accuracy/precision/recall/F1 (clasificación), k-fold con k=5.

## Referencias científicas base

Citar fuentes reales, no inventar. Bibliografía obligatoria: ACSM, ISSN, IOC consensus 2018, Jeukendrup, Burke, Thomas et al. 2016, Mifflin-St Jeor 1990, Aragón/Schoenfeld.

## Decisiones de diseño ya tomadas (no reabrir sin motivo)

1. Nutrición sobre biomecánica (reglas cuantitativas más defendibles).
2. Intra-entreno sin mínimo de duración: continuo según duración + intensidad + contexto.
3. Sin límites duros en edición: el sistema informa, no bloquea.
4. Dataset sintético declarado, fundamentado en literatura.
5. Motor dual: ML complementa las reglas, no las sustituye.
6. Diseño API-ready desde el inicio.
7. SQLite + SQLAlchemy, migrable a PostgreSQL sin cambios de código.
8. Plotly + Streamlit como prototipo.
9. HTML interactivo como exportación principal; .docx como editable para el profesional.
10. 80 alimentos, ampliable en futuro.
11. M5 con reglas simples, no ML.
12. Strava en tercer nivel.
13. Sin prueba piloto en el alcance.
14. Priorización en tres niveles para garantizar entrega mínima sólida.
15. Sin entidad `nutricionista`/`perfil_nutri` en la BD. La customización de rangos se hace en `src/rules/config/macros.yaml`, no en multi-tenant. El profesional edita por día/deportista en la UI y la trazabilidad vive en el propio registro (`editado`, `justificacion`).

## Convenciones específicas del proyecto

- Reglas fisiológicas trazables: cada función del motor de reglas cita su referencia bibliográfica en docstring.
- Dataclasses en [src/core/](src/core/) como contrato: cualquier cambio afecta a rules, ml, db, export.
- Un archivo por módulo en [src/rules/](src/rules/) — no mezclar lógica entre módulos.
- Datos crudos (CSV, JSON, .FIT) viven en [data/](data/) y son de solo lectura.
- Tests mínimos (pytest): entrada conocida → salida en rango esperado. 10-15 tests cubriendo M1, M3, M4.

## Gotchas

- Nunca inventar referencias bibliográficas: si no está verificada, no se cita.
- M4 sobreescribe M1 y M2 en D-3/D-2/D-1 cuando hay competición — respetar el orden del flujo.
- M3 se activa para cualquier sesión sin mínimo: no poner umbrales duros.
- kJ reales (importados de .FIT/Strava) sustituyen el gasto estimado por MET cuando existen.
