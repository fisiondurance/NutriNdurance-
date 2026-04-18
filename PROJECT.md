# NutriNdurance — Sistema inteligente de planificación nutricional y recomendación intra-entreno para deportistas de resistencia

## Autor

Mario Godoy — Fisioterapeuta y nutricionista deportivo especializado en deportistas de resistencia (ciclismo, running, triatlón).

---

## 1. Descripción del proyecto

Sistema que, a partir del perfil del deportista y su plan de entrenamiento semanal, genera un plan nutricional completo de forma automatizada e individualizada. Cubre el ciclo semanal de decisiones nutricionales: cuánto comer cada día, cómo repartirlo en comidas, qué ingerir durante cada sesión de entrenamiento y cómo preparar los días previos a una competición.

Público objetivo: nutricionistas deportivos, entrenadores, fisioterapeutas y deportistas de resistencia (ciclismo, running, triatlón, natación de aguas abiertas, esquí de fondo, ultra-resistencia).

---

## 2. Arquitectura del sistema — 5 módulos

### Módulo 1 — Planificación nutricional diaria

**Qué hace:** Recibe el perfil del deportista y su plan semanal. Para cada día, clasifica el tipo de jornada y calcula necesidades energéticas y macronutrientes.

**Variables de entrada (perfil):**
- Peso, altura, edad, sexo
- Nivel de actividad basal (sedentario fuera de entrenos / activo por trabajo)
- Deporte principal
- Nivel deportivo (recreativo, amateur, competitivo, élite)
- Objetivo nutricional (rendimiento, recomposición corporal, pérdida de peso)

**Variables de entrada (por día):**
- Tipo de sesión (descanso, técnica/movilidad, rodaje suave, entreno moderado, entreno intenso/intervalos, entreno largo, competición)
- Duración (minutos)
- Intensidad (zona de entrenamiento o RPE)
- Doble sesión (sí/no)

**Clasificación automática del día:**

| Categoría | Criterio | Factor de actividad deportiva |
|---|---|---|
| Descanso | Sin sesión | 0 |
| Ligero | Técnica, movilidad, <45 min suave | Bajo |
| Moderado | Rodaje 45-90 min, fuerza | Medio |
| Intenso | Series, intervalos, >Z3 sostenido | Alto |
| Largo | >2h independientemente de intensidad | Alto-Muy alto |
| Competición | Marcado como competición | Muy alto |

**Lógica de cálculo:**
1. Gasto energético basal (Mifflin-St Jeor)
2. Factor de actividad no deportiva (NEAT)
3. Gasto de la sesión (MET × duración × peso, ajustado por intensidad y deporte)
4. Distribución de macros según tipo de día + objetivo (ej: día intenso → 6-8 g CHO/kg; descanso → 3-4 g CHO/kg; proteína 1.6-2.2 g/kg constante)

**Input opcional de kJ reales:** Si el deportista importa datos de sesiones completadas (vía .FIT o Strava en futuro), el gasto estimado por MET se sustituye por el gasto real medido.

**Output:** kcal totales del día, g CHO/kg, g PRO/kg, g FAT/kg, g totales de cada macro.

**NOTA:** El sistema calcula rangos recomendados de g/kg para cada macronutriente según el tipo de día, el perfil del deportista y su objetivo. El nutricionista puede aceptar los valores sugeridos, ajustarlos dentro del rango o introducir valores fuera de rango. Al modificar cualquier parámetro, el sistema recalcula en tiempo real los gramos totales, las kcal resultantes y la diferencia respecto al TDEE. Opcionalmente, el nutricionista puede guardar sus criterios personalizados por tipo de día como perfil reutilizable para futuros deportistas. La misma lógica de selección y ajuste se aplica a las recomendaciones de M3 (intra-entreno).

---

### Módulo 2 — Distribución de comidas (timing nutricional)

**Qué hace:** Toma el output de M1 y lo reparte en comidas concretas adaptándose a la hora del entrenamiento.

**Variables de entrada adicionales:**
- Hora de la sesión (mañana / mediodía / tarde)
- Hora de despertar (aproximada)
- Número de comidas preferido (3, 4, 5 o 6)

**Lógica de distribución según bloque horario:**

| Entreno por la mañana | Entreno por la tarde |
|---|---|
| Pre-entreno (ligero) | Desayuno |
| Durante (si aplica → M3) | Comida (con carga pre-entreno) |
| Post-entreno / Desayuno-2 | Pre-entreno (snack) |
| Comida | Durante (si aplica → M3) |
| Merienda (si aplica) | Post-entreno / Merienda |
| Cena | Cena |

**Reglas de reparto:**
- Proteína distribuida equitativamente entre comidas (cada 3-4h, 0.3-0.5 g/kg por toma)
- CHO concentrados en pre-entreno (1-4 g/kg, 1-4h antes) y post-entreno (1-1.2 g/kg en las primeras 2h)
- Grasa alejada del peri-entreno (ralentiza vaciado gástrico)
- Días de descanso: reparto uniforme sin picos

**Base de datos de alimentos:**
- 80 alimentos frecuentes en deportistas de resistencia
- Campos por alimento: nombre, kcal/100g, CHO, PRO, FAT, fibra, sodio, tags de restricciones
- Fuente: BEDCA o USDA FoodData Central (citar procedencia)
- Filtrado por restricciones: vegetariano, vegano, sin gluten, sin lactosa

**Output:** Tabla con comida → hora aproximada → g CHO, g PRO, g FAT, kcal + sugerencia de alimentos.

---

### Módulo 3 — Recomendación intra-entreno

**IMPORTANTE: Se activa para CUALQUIER sesión de entrenamiento, sin mínimo de duración.**

La recomendación se adapta como un continuo según duración + intensidad + contexto:
- Sesiones <30 min: generalmente no requieren CHO (posible enjuague bucal en alta intensidad), pero sí valoración de hidratación
- Sesiones 30-60 min: hasta 30 g CHO/h en alta intensidad o contextos específicos (ayuno, calor, competición)
- Sesiones 60-90 min: 30-60 g CHO/h
- Sesiones 90 min - 2.5h: 60 g CHO/h
- Sesiones >2.5h: hasta 90-120 g/h con entrenamiento intestinal avanzado

**Variables de entrada:**
- Peso del deportista
- Duración de la sesión
- Intensidad (zona / RPE)
- Tipo de actividad
- Temperatura ambiente (frío <15°C, templado 15-25°C, calor >25°C, calor extremo >35°C)
- Tolerancia digestiva (baja, normal, alta)
- Experiencia con nutrición intra-entreno (principiante, intermedio, avanzado)
- Historial de problemas GI (sí/no)
- Nivel de entrenamiento intestinal (principiante / en entrenamiento / entrenado) — ajusta el techo de CHO/h

**Output:**
- g CHO/hora (rango personalizado)
- ml líquidos/hora
- mg sodio/hora
- Nivel de riesgo GI (bajo / medio / alto)
- Recomendación de ratio glucosa:fructosa (si CHO/h > 60g)

**Lógica:**
- CHO/h según duración e intensidad (Jeukendrup)
- Ajuste por tolerancia y experiencia
- Líquidos: tasa de sudoración estimada según peso, intensidad, temperatura (400-800 ml/h base, escalado por calor)
- Sodio: 300-700 mg/L según duración y condiciones
- Riesgo GI: clasificación basada en tipo de actividad (running = mayor riesgo mecánico), calor, historial, alta ingesta con baja tolerancia

---

### Módulo 4 — Protocolo pre-competición

**Se activa cuando:** el plan semanal incluye un día marcado como competición.

**Protocolo condicional a la duración de la competición:**

**Perfil A — Competición corta (< 60 min):**
- D-2 y D-1: dieta normal (5-7 g CHO/kg)
- Sin carga
- Día D: enfoque en confort digestivo

**Perfil B — Competición media (60-90 min):**
- D-1: 6-7 g CHO/kg, reducción ligera de fibra
- Sin supercompensación
- Día D pre-carrera: 1-3 g CHO/kg 3-4h antes

**Perfil C — Competición moderada (90 min - 3h):**
- D-2: 7-8 g CHO/kg, fibra reducida
- D-1: 8-10 g CHO/kg, fibra muy reducida, grasa reducida
- Entreno: muy ligero D-2, descanso/activación D-1
- Día D pre-carrera: 2-4 g CHO/kg 3-4h antes + cafeína opcional (3-6 mg/kg 60 min antes) + hidratación 5-10 ml/kg 2-4h antes

**Perfil D — Competición larga (> 3h):**
- D-3: 7-8 g CHO/kg
- D-2: 8-10 g CHO/kg, fibra reducida
- D-1: 10-12 g CHO/kg, fibra muy baja, grasa mínima, residuos mínimos
- Entreno: rodaje corto D-3, muy ligero D-2, descanso D-1
- Día D: 2-4 g CHO/kg 3-4h antes + hiperhidratación con sodio + cafeína opcional
- Activa M3 con configuración de alta ingesta

**Pauta pre-carrera (todos los perfiles que lo requieran):**
- Ingesta 3-4h antes: 1-4 g/kg CHO según perfil
- Cafeína opcional: 3-6 mg/kg 60 min antes (sí/no/no sabe como input del deportista)
- Hidratación: 5-10 ml/kg 2-4h antes

**Variables de entrada:** fecha de competición, duración estimada, peso, preferencia de cafeína, historial GI.

**Output:** plan nutricional modificado para D-3 a D-1 (sobreescribe M1 y M2 para esos días), pauta pre-carrera del día D, lista de alimentos recomendados y a evitar (filtro "bajo en fibra" y "bajo en residuos" de la BD de alimentos).

---

### Módulo 5 — Retroalimentación del deportista

**Implementación simple basada en reglas de ajuste predefinidas.**

**Input del deportista tras cada sesión:**
- Nivel de energía durante la sesión (1-5)
- Tolerancia GI (sin problemas / molestias leves / molestias moderadas / problemas serios)
- Notas libres

**Reglas de ajuste (ejemplos):**
- Si reporta molestias GI dos sesiones seguidas → reducir CHO/h un 10-15% en la siguiente recomendación
- Si reporta baja energía dos sesiones seguidas → incrementar CHO pre-entreno un 10%
- Si reporta buena tolerancia de forma consistente → permitir incremento gradual del techo de CHO/h

**Output:** recomendaciones ajustadas en M3 y M2. Trazabilidad del ajuste (qué se cambió, por qué, basado en qué feedback).

---

## 3. Flujo completo del sistema

```
Plan semanal del deportista
        │
        ▼
   ┌─────────┐
   │   M1    │  Clasifica cada día → calcula kcal + macros
   └────┬────┘
        │
        ▼
   ┌─────────┐
   │   M4    │  ¿Hay competición? → Ajusta D-3, D-2, D-1
   └────┬────┘
        │
        ▼
   ┌─────────┐
   │   M2    │  Distribuye macros en comidas según hora de entreno
   └────┬────┘
        │
        ▼
   ┌─────────┐
   │   M3    │  Para cada sesión → pauta intra-entreno
   └─────────┘
        │
        ▼
   Output final: plan semanal completo
   (día a día, comida a comida, con pauta intra)
        │
        ▼
   ┌─────────┐
   │   M5    │  Feedback post-sesión → ajuste progresivo
   └─────────┘
```

---

## 4. Arquitectura técnica

### Motor dual: reglas + ML

- **Capa 1 — Motor de reglas:** funciones Python que implementan las reglas fisiológicas documentadas en la literatura. Cada regla tiene trazabilidad bibliográfica.
- **Capa 2 — Modelos ML:** entrenados sobre un dataset sintético generado a partir de las reglas + variabilidad controlada. Comparados frente al baseline de reglas.
- **Detección de discrepancias:** si ML y reglas difieren significativamente en una recomendación, el sistema lo señala. (Tercer nivel de prioridad.)
- **Explicabilidad:** SHAP values aplicados a los modelos para mostrar qué variables pesan en cada predicción.

### Diseño API-ready

Las funciones del núcleo (reglas, ML, cálculos) reciben y devuelven diccionarios tipados o dataclasses serializables a JSON. Streamlit es una "capa fina" que llama a esas funciones y pinta el resultado. Si en el futuro se quiere exponer como API REST (FastAPI), se envuelven las mismas funciones en endpoints sin reescribir nada.

### Edición libre con alertas informativas

Todas las recomendaciones son editables sin límites duros. El sistema muestra alertas en tres niveles:
- Verde: valor dentro del rango de evidencia
- Amarillo: fuera del rango típico pero justificable en ciertos perfiles
- Rojo: incoherente con principios fisiológicos básicos

Nunca se bloquea un valor. El profesional tiene la última palabra.

### Persistencia

- SQLAlchemy como ORM sobre SQLite
- Migrable a PostgreSQL cambiando solo la cadena de conexión

**Modelo de datos mínimo:**

```
perfil_deportista → id, nombre, peso, altura, edad, sexo, nivel, deporte, objetivo, preferencias_alimentarias
plan_entrenamiento → id, perfil_id, fecha_inicio, fecha_fin
sesion → id, plan_id, fecha, tipo, duracion_min, intensidad, es_competicion, kj_reales, fc_media
recomendacion_diaria → id, perfil_id, fecha, tipo_dia, kcal, cho_g, pro_g, fat_g, editado
recomendacion_intra → id, sesion_id, cho_h, liquidos_h, sodio_h, riesgo_gi
comida → id, recomendacion_diaria_id, nombre, hora, cho_g, pro_g, fat_g, kcal
feedback → id, sesion_id, energia, tolerancia_gi, notas, fecha
alimento → id, nombre, kcal_100g, cho, pro, fat, fibra, sodio, vegetariano, vegano, sin_gluten, sin_lactosa
```

---

## 5. Stack tecnológico

| Componente | Tecnología |
|---|---|
| Lenguaje | Python |
| ORM + BD | SQLAlchemy + SQLite |
| ML | scikit-learn, XGBoost, LightGBM |
| Explicabilidad | SHAP |
| Visualización (dashboard) | Plotly |
| Interfaz / prototipo | Streamlit |
| Exportación HTML | Jinja2 + Plotly embebido |
| Exportación .docx | python-docx |
| Lectura .FIT | fitparse |
| Control de versiones | Git / GitHub |
| Empaquetado (tercer nivel) | Docker |

---

## 6. Estructura del proyecto

```
NutriNdurance/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   ├── alimentos.csv              # Base de datos de 80 alimentos
│   └── perfiles_demo.json         # 3-5 personas precargadas para demo
├── src/
│   ├── __init__.py
│   ├── core/                      # Lógica de dominio
│   │   ├── perfil.py              # Dataclasses del deportista
│   │   ├── sesion.py              # Dataclasses de sesión
│   │   └── plan.py                # Dataclasses del plan semanal
│   ├── rules/                     # Motor de reglas (un archivo por módulo)
│   │   ├── modulo1_diaria.py
│   │   ├── modulo2_comidas.py
│   │   ├── modulo3_intra.py
│   │   ├── modulo4_precompeticion.py
│   │   └── modulo5_feedback.py
│   ├── ml/                        # Modelos ML
│   │   ├── dataset_generator.py   # Generación del dataset sintético
│   │   ├── train.py               # Entrenamiento de modelos
│   │   ├── evaluate.py            # Evaluación y comparativa
│   │   └── shap_explain.py        # Explicabilidad SHAP
│   ├── db/                        # Capa de datos
│   │   ├── models.py              # Modelos SQLAlchemy
│   │   └── database.py            # Conexión y sesión
│   ├── fit/                       # Importación .FIT
│   │   └── fit_parser.py
│   ├── export/                    # Exportación
│   │   ├── html_generator.py      # HTML interactivo con Plotly
│   │   ├── docx_generator.py      # .docx editable
│   │   └── templates/             # Plantillas Jinja2
│   └── ui/                        # Streamlit (solo interfaz)
│       ├── app.py                 # Entry point
│       └── components/            # Componentes reutilizables
├── notebooks/                     # EDA y análisis
├── tests/                         # Tests unitarios (pytest)
└── docs/                          # Documentación del proyecto
```

---

## 7. Dataset sintético

### Justificación

No existe un dataset público de planificación nutricional deportiva individualizada. Las reglas fisiológicas están bien documentadas en la literatura. Se genera un dataset sintético fundamentado en esas reglas, con variabilidad controlada para simular la dispersión real entre deportistas. Se declara y justifica explícitamente.

### Proceso de generación

1. Definir las reglas fisiológicas con sus parámetros
2. Generar N registros (perfiles × días × sesiones) con combinaciones realistas de variables
3. Aplicar las reglas para calcular la recomendación "ideal" de cada registro
4. Añadir variabilidad: ruido gaussiano, variación en tolerancia individual, dispersión en cumplimiento, interacciones no lineales (ej: efecto del calor sobre sodio)
5. El dataset resultante tiene inputs (perfil + sesión + contexto) y targets (recomendaciones)

### Variables target

- **Regresión:** kcal/día, g CHO/día, g PRO/día, g FAT/día, CHO/h intra, líquidos/h, sodio/h
- **Clasificación:** tipo de día (descanso/suave/moderado/intenso/competición), riesgo GI (bajo/medio/alto)

### Modelos a entrenar y comparar

1. Baseline: reglas puras
2. Regresión lineal
3. Decision Tree
4. Random Forest
5. Gradient Boosting (XGBoost o LightGBM)
6. Clasificación para riesgo GI: logistic regression o random forest classifier

### Métricas de evaluación

- Regresión: MAE, RMSE, R²
- Clasificación: accuracy, precision, recall, F1, matriz de confusión
- Validación cruzada (k-fold, k=5 o 10)
- Comparativa ML vs. baseline de reglas (el contenido analítico principal)

---

## 8. Visualización y dashboard (Plotly en Streamlit)

**M1 — Planificación diaria:**
- Barras apiladas: semana completa, eje X = días, eje Y = kcal, colores = CHO/PRO/FAT
- Líneas superpuestas: gasto energético estimado vs. ingesta planificada

**M2 — Distribución de comidas:**
- Timeline horizontal o barras por comida, tamaño proporcional a kcal, colores por macro
- Donut/pie por comida: proporción CHO/PRO/FAT

**M3 — Intra-entreno:**
- Barras horizontales: CHO/h, líquidos/h, sodio/h con franja de rango de evidencia
- Gauge/velocímetro para riesgo GI

**M4 — Pre-competición:**
- Timeline de D-3 a Día D: progresión de la carga CHO, reducción de fibra/grasa
- Comparación visual: protocolo sugerido vs. valores editados

**M5 — Feedback:**
- Líneas temporales: evolución de energía y tolerancia GI sesión a sesión
- Scatter: CHO/h vs. valoración subjetiva de energía

**Dashboard resumen semanal:**
- KPIs: kcal totales, adherencia al plan, alertas, riesgo GI medio, tendencia de feedback

---

## 9. Exportación del plan

**Formato principal — HTML interactivo:**
- Archivo HTML autocontenido con gráficos Plotly embebidos
- Tablas colapsables por día
- Diseño responsive (móvil + escritorio)
- Generado con plantilla Jinja2

**Formato secundario — .docx editable:**
- Portada personalizada, resumen visual de la semana
- Detalle día a día con tablas de comidas
- Pauta intra-entreno cuando corresponda
- Notas editables por el profesional
- Generado con python-docx

---

## 10. Priorización del desarrollo

### Núcleo (se hace sí o sí)

- M1 (planificación diaria) + M3 (intra-entreno)
- Motor de reglas completo para M1 y M3
- Dataset sintético + pipeline ML + evaluación
- Prototipo Streamlit básico funcional
- Git + estructura modular

### Segundo nivel (se hace si el núcleo va bien)

- M2 (distribución de comidas) + M4 (pre-competición)
- Base de datos de 80 alimentos
- Dashboard interactivo con Plotly
- Exportación HTML + .docx
- SHAP para explicabilidad
- SQLite + SQLAlchemy

### Tercer nivel (si hay tiempo)

- M5 (feedback loop con reglas simples)
- Integración Strava
- Importación .FIT
- Motor dual con detección de discrepancias
- Docker
- 3-5 perfiles precargados para demo
- Edición libre con alertas informativas

---

## 11. Fases de desarrollo

| Fase | Contenido | Entregable |
|---|---|---|
| 1 | Fundamentación + reglas base + setup Git | Árbol de reglas M1 y M3 con referencias, repositorio configurado |
| 2 | Base de datos de alimentos + modelo de datos | alimentos.csv (80 items), modelos SQLAlchemy, esquema relacional |
| 3 | Dataset sintético + EDA | Script de generación, notebook con análisis exploratorio |
| 4 | Motor de reglas funcional | Módulos Python para M1, M2, M3, M4 con tests básicos |
| 5 | Pipeline ML + SHAP | Entrenamiento, comparativa baseline vs. modelos, SHAP plots |
| 6 | Motor dual e integración | Sistema combinado reglas + ML |
| 7 | Prototipo Streamlit + dashboard Plotly | Interfaz funcional con visualización interactiva |
| 8 | Exportación HTML + .docx | Generadores funcionales con plantillas |

---

## 12. Referencias científicas de base

Estas son las fuentes que fundamentan las reglas del sistema:

- **ACSM** — American College of Sports Medicine: posicionamiento sobre nutrición y rendimiento deportivo
- **ISSN** — International Society of Sports Nutrition: posicionamientos sobre CHO, proteína, hidratación
- **IOC** — International Olympic Committee: consenso 2018 sobre nutrición deportiva
- **Jeukendrup, A.** — Recomendaciones de CHO intra-ejercicio, periodización, gut training
- **Burke, L.M.** — Protocolos de carga de glucógeno, periodización nutricional
- **Thomas, D.T. et al. (2016)** — Joint Position Statement ACSM/Academy of Nutrition and Dietetics/Dietitians of Canada
- **Mifflin, M.D. et al. (1990)** — Ecuación de gasto energético basal (Mifflin-St Jeor)
- **Aragón, Schoenfeld et al.** — Evidencia actualizada sobre ventana post-ejercicio

Bibliografía completa en APA 7 en [docs/referencias.md](docs/referencias.md). Fundamentación detallada por regla en [docs/evidencia_cientifica.md](docs/evidencia_cientifica.md).

**Nota:** NO inventar referencias. Citar solo fuentes verificadas.

---

## 13. Líneas futuras declaradas

### Funcionales

- Prueba piloto con deportistas y profesionales
- Integración con APIs de Strava, Garmin Connect, TrainingPeaks
- Safety rails clínicas (límites duros configurables)
- Autocalibración de tasa de sudoración individual
- Alertas de baja disponibilidad energética (RED-S)
- Suplementación (cafeína, beta-alanina, creatina)
- Sugerencia completa de alimentos con base de datos amplia
- Multi-idioma (español + inglés)
- Modo deportista vs. modo profesional
- Notificaciones y recordatorios
- Comparación de planes (original vs. editado, semana actual vs. anterior)

### Arquitectónicas

- FastAPI como backend (API REST)
- Migración de SQLite a PostgreSQL
- Frontend moderno en React o Vue.js
- PWA instalable con consulta offline
- Despliegue en la nube con orquestación de contenedores

---

## 14. Decisiones de diseño tomadas

1. **Nutrición sobre biomecánica.** El dataset sintético se defiende mejor en nutrición (reglas con respaldo cuantitativo claro) que en biomecánica (rangos normativos difusos).
2. **Intra-entreno sin mínimo de duración.** El sistema calcula recomendaciones para cualquier sesión, adaptando como un continuo según duración + intensidad + contexto.
3. **Sin límites duros en la edición.** El profesional puede introducir cualquier valor. El sistema informa, no bloquea.
4. **Dataset sintético declarado.** Justificado como legítimo cuando se declara y fundamenta. No se presenta como datos reales.
5. **Motor dual reglas + ML.** El ML no sustituye las reglas; las complementa. La comparativa reglas vs. ML es el contenido analítico principal.
6. **Diseño API-ready.** Funciones con entrada/salida serializable para facilitar la futura exposición como API REST.
7. **SQLite + SQLAlchemy.** Persistencia relacional desde el inicio, migrable a PostgreSQL sin cambios de código.
8. **Plotly para dashboards.** Buena integración con Streamlit.
9. **HTML interactivo como exportación principal.** Más profesional que .docx para el deportista. El .docx se mantiene como formato editable para el profesional.
10. **80 alimentos.** Base de datos acotada y viable. Ampliable en futuras versiones.
11. **M5 simple.** Reglas de ajuste predefinidas, no ML. Suficiente para demostrar el concepto.
12. **Strava en tercer nivel.** API pública y viable, pero no es núcleo.
13. **Sin prueba piloto.** Eliminada del alcance, pasa a líneas futuras.
14. **Priorización en tres niveles.** Núcleo → segundo nivel → tercer nivel. Permite entregar una versión sólida aunque no dé tiempo a todo.

---

## 15. Tests

Tests mínimos con pytest para las funciones del motor de reglas:
- Entrada conocida (perfil tipo + sesión tipo) → salida dentro de rango esperado
- Cubrir: M1 (cálculo de kcal y macros por tipo de día), M3 (CHO/h para distintas duraciones e intensidades), M4 (protocolo correcto según duración de competición)
- 10-15 tests cubren los casos principales
- Ubicación: `tests/`
