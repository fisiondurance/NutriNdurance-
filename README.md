# NutriNdurance

Sistema de planificación nutricional para deportistas de resistencia (ciclismo, running, triatlón).

A partir del perfil del deportista y su plan semanal, el sistema genera un plan nutricional individualizado: kilocalorías y macronutrientes por día, distribución en comidas según la hora de entreno, pauta intra-entreno, protocolo pre-competición y feedback post-sesión.

## Arquitectura

Motor dual: reglas fisiológicas (fuente de verdad, con trazabilidad bibliográfica) + modelos de ML entrenados sobre dataset sintético generado a partir de las reglas. Diseño API-ready: la lógica de dominio vive en funciones serializables; Streamlit actúa como capa fina.

Cinco módulos:

- M1 — Planificación diaria (kcal + macros por tipo de día).
- M2 — Distribución en comidas según hora de entreno.
- M3 — Recomendación intra-entreno (CHO/h, líquidos, sodio, riesgo GI).
- M4 — Protocolo pre-competición (D-3 a Día D).
- M5 — Feedback del deportista con reglas de ajuste.

## Stack

Python · SQLAlchemy + SQLite · scikit-learn + XGBoost + LightGBM · SHAP · Plotly · Streamlit · Jinja2 · python-docx · fitparse · pytest.

## Estructura

```
data/                # Datos crudos, read-only (alimentos.csv, perfiles_demo.json)
artifacts/           # Generados (dataset, modelos ML, exports), gitignored
src/
  core/              # Dataclasses de dominio
  rules/             # Motor de reglas (un archivo por módulo)
  ml/                # Dataset sintético, entrenamiento, evaluación, SHAP
  db/                # SQLAlchemy
  fit/               # Importación .FIT
  export/            # HTML y .docx + plantillas Jinja2
  ui/                # Streamlit
notebooks/           # EDA
tests/               # pytest
docs/                # Documentación
```

## Instalación

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Documentación

- [PROJECT.md](PROJECT.md): especificación íntegra del sistema.
- [CLAUDE.md](CLAUDE.md): contexto operativo para sesiones de desarrollo.
- [docs/evidencia_cientifica.md](docs/evidencia_cientifica.md): fundamentación científica de las reglas.
- [docs/referencias.md](docs/referencias.md): bibliografía APA 7.

## Autor

Mario Godoy — fisioterapeuta y nutricionista deportivo especializado en deportistas de resistencia.
