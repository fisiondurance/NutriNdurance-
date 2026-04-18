# Evidencia científica del sistema

Documento centralizado de fundamentación. Cada regla del motor cita una sección de este archivo desde su docstring.

**Convención de anclas:** `#m<n>-<tema-kebab>` (ej. `#m1-cho-por-tipo-de-dia`). Estables — no renombrar sin actualizar docstrings.

**Aviso:** referencias en APA 7 disponibles en [referencias.md](referencias.md). Verificar DOI/páginas contra la fuente original antes de citar externamente.

---

## M1 — Planificación nutricional diaria

### M1 — BMR (ecuación Mifflin-St Jeor) {#m1-bmr}

**Qué dice la evidencia:** la ecuación de Mifflin-St Jeor predice el gasto metabólico basal con menor error que Harris-Benedict en adultos sanos, tanto con peso normal como con sobrepeso/obesidad.

**Fuente principal:** Mifflin et al. (1990).

**Fórmula:**
```
hombre:  BMR = 10·peso(kg) + 6.25·altura(cm) − 5·edad + 5
mujer:   BMR = 10·peso(kg) + 6.25·altura(cm) − 5·edad − 161
```

**Rango de error documentado:** ±10% vs. calorimetría indirecta en población sana.

**Decisión de diseño:** se usa Mifflin-St Jeor como única fórmula de BMR. No ofrecemos selector con Harris-Benedict u otras — el rango de error es equivalente o peor y añade complejidad sin aportar precisión.

---

### M1 — Factor NEAT (actividad no deportiva) {#m1-neat}

**Qué dice la evidencia:** el gasto no asociado al entrenamiento estructurado (NEAT: non-exercise activity thermogenesis) varía ampliamente entre individuos. La tradición clínica aplica factores multiplicativos sobre el BMR.

**Fuente:** Harris & Benedict (factores tradicionales actualizados en posicionamientos). Referenciado en Thomas et al. (2016).

**Rangos:**

| Nivel | Factor sobre BMR |
|---|---|
| Sedentario (oficina, teletrabajo) | 1.2 |
| Ligeramente activo (camina, de pie a ratos) | 1.35 |
| Activo por trabajo físico (manual, reparto) | 1.55 |

**Decisión de diseño:** solo 3 niveles. El entrenamiento deportivo **no** se incluye en este factor; se calcula aparte (ver [M1 — gasto de la sesión](#m1-gasto-sesion)). Esto evita la doble contabilidad clásica de las ecuaciones "TDEE con deporte incluido".

---

### M1 — Gasto energético de la sesión {#m1-gasto-sesion}

**Qué dice la evidencia:** el gasto de una sesión se puede estimar con equivalentes metabólicos (MET) según deporte e intensidad. En ciclismo con potenciómetro, el trabajo mecánico en kJ ≈ kcal totales por la eficiencia ~24% del movimiento.

**Fuentes:** Ainsworth et al. (2011) para MET por actividad; convención de eficiencia en ciclismo documentada en Burke et al. (2011).

**Fórmulas:**
```
vía estimada:  kcal_sesion = MET × duración(h) × peso(kg)
vía real:      kcal_sesion = kJ_reales    # si hay importación .FIT/Strava
```

**Rango MET orientativo:**

| Actividad | MET aprox. |
|---|---|
| Ciclismo Z2 (suave) | 6 |
| Ciclismo Z3-Z4 | 8-10 |
| Ciclismo HIIT | 10-12 |
| Running 10 km/h | 10 |
| Running intervalos | 12-14 |
| Fuerza moderada | 5 |

**Decisión de diseño:** la vía real (kJ) siempre prevalece si existe. Es más precisa por individuo y elimina el error del MET.

---

### M1 — CHO por tipo de día {#m1-cho-por-tipo-de-dia}

**Qué dice la evidencia:** la ingesta de carbohidratos debe periodizarse según la carga diaria de entrenamiento. Los rangos son consistentes entre los principales posicionamientos.

**Fuentes principales:** Thomas et al. (2016), Burke et al. (2011), ACSM/IOC consensus.

**Rangos (g/kg/día):**

| Tipo de día | CHO g/kg |
|---|---|
| Descanso | 3-4 |
| Ligero (<45 min suave, técnica) | 4-5 |
| Moderado (rodaje 45-90 min, fuerza) | 5-6 |
| Intenso (series, >Z3 sostenido) | 6-8 |
| Largo (>2h) | 7-10 |
| Competición | 8-12 |

**Decisión de diseño:** valores configurables en `src/rules/config/macros.yaml`. El `sugerido` por defecto es el punto medio del rango. El objetivo del deportista (rendimiento/recomposición/pérdida) modula estos valores: en déficit se tiende al extremo inferior, en carga al superior.

---

### M1 — Proteína {#m1-proteina}

**Qué dice la evidencia:** la ingesta óptima para deportistas de resistencia está entre 1.6 y 2.0 g/kg/día en mantenimiento. En déficit calórico o recomposición corporal, subir hasta 2.2 g/kg preserva masa muscular.

**Fuentes:** Jäger et al. (2017) (ISSN Position Stand), Morton et al. (2018) (meta-análisis).

**Rangos (g/kg/día):**

| Objetivo | PRO g/kg |
|---|---|
| Rendimiento / mantenimiento | 1.6-2.0 |
| Recomposición corporal | 1.8-2.2 |
| Pérdida de peso (déficit) | 2.0-2.2 |

**Decisión de diseño:** la proteína se calcula **independientemente del tipo de día** — es función solo del objetivo y el peso. Esto simplifica la UI y se alinea con la literatura (no se periodiza como el CHO).

---

### M1 — Grasa (macro residual) {#m1-grasa}

**Qué dice la evidencia:** la grasa completa las kcal del día tras fijar CHO y proteína, con un mínimo clínico de 0.8-1 g/kg para asegurar función hormonal y absorción de vitaminas liposolubles.

**Fuente:** Thomas et al. (2016).

**Cálculo:**
```
kcal_fat = kcal_objetivo − (g_cho × 4) − (g_pro × 4)
g_fat   = kcal_fat / 9
```

**Suelo clínico:** `g_fat >= 0.8 × peso`. Si no se cumple, el sistema marca alerta roja (ver [safety rail LEA/RED-S](#m1-safety-rail)).

**Decisión de diseño:** no se fija rango superior — la grasa absorbe el balance restante. El rango inferior es safety rail, no restricción operativa.

---

### M1 — Ajuste por objetivo {#m1-ajuste-por-objetivo}

**Qué dice la evidencia:**

- **Rendimiento:** mantenimiento energético. Déficits comprometen calidad de entreno.
- **Recomposición:** déficit leve (5-10%) con proteína alta (≥1.8 g/kg).
- **Pérdida de peso:** déficit 15-25%, nunca por debajo de BMR × 1.1 para evitar LEA/RED-S.

**Fuentes:** Helms et al. (2014), Aragon & Schoenfeld (2020), Mountjoy et al. (2018) (IOC RED-S).

**Aplicación sobre TDEE:**

| Objetivo | Ajuste |
|---|---|
| Rendimiento | 0% |
| Recomposición | −5% a −10% |
| Pérdida de peso | −15% a −25% |

**Safety rail {#m1-safety-rail}:** `kcal_objetivo >= BMR × 1.1`. Por debajo, alerta roja permanente hasta que el profesional lo justifique (el sistema nunca bloquea).

---

## M2 — Distribución de comidas

> **TODO (v0.2):** poblar con evidencia de timing nutricional.
>
> Temas a cubrir:
> - Ventana peri-entreno (pre, durante, post)
> - Reparto de proteína cada 3-4h (0.3-0.5 g/kg/toma)
> - CHO concentrados en peri-entreno
> - Grasa alejada del peri-entreno (vaciado gástrico)
> - Reparto en días de descanso
>
> Referencias a revisar: Jäger et al. 2017, Aragón & Schoenfeld 2013, Kerksick et al. 2017.

---

## M3 — Recomendación intra-entreno

### M3 — CHO/h según duración {#m3-cho-h-segun-duracion}

**Qué dice la evidencia:** la ingesta de CHO durante el ejercicio se adapta como un continuo según la duración y la intensidad. La tasa máxima de oxidación de glucosa exógena es ~60 g/h; superar ese umbral requiere combinación glucosa:fructosa (2:1) que utiliza transportadores distintos (SGLT1 + GLUT5).

**Fuente principal:** Jeukendrup (2014), Jeukendrup & McLaughlin (2011).

**Rangos (g CHO/h):**

| Duración | CHO/h sugerido | Notas |
|---|---|---|
| <30 min | 0 (posible enjuague bucal) | Activa vía neural sin ingestión |
| 30-60 min | 0-30 | Opcional; alto valor en calor, ayuno o competición |
| 60-90 min | 30-60 | Glucosa o glucosa + maltodextrina |
| 90 min - 2.5h | 60 | Techo de un solo tipo de CHO |
| >2.5h | 60-90 (hasta 120 con gut training) | Requiere ratio glucosa:fructosa 2:1 |

**Decisión de diseño:** el sistema calcula recomendación para cualquier sesión sin umbral mínimo — se adapta continuamente. El techo superior se ajusta por nivel de "entrenamiento intestinal" del deportista (principiante 60, intermedio 90, entrenado 120).

---

### M3 — Hidratación {#m3-hidratacion}

**Qué dice la evidencia:** la tasa de sudoración varía 400-2000+ ml/h según tamaño corporal, intensidad, temperatura y humedad. La reposición debe ajustarse al peso perdido tras sesión (pérdida máxima 2% peso corporal).

**Fuentes:** Sawka et al. (2007) (ACSM Position Stand), Thomas et al. (2016).

**Rangos base (ml/h):**

| Temperatura | Líquidos sugeridos |
|---|---|
| Frío (<15°C) | 400-600 |
| Templado (15-25°C) | 500-700 |
| Calor (25-35°C) | 700-1000 |
| Calor extremo (>35°C) | 1000-1500+ |

**Decisión de diseño:** el sistema estima la tasa base por peso × intensidad × temperatura. La tasa personal exacta requiere medición individual (pesar antes/después) — marcado como línea futura (autocalibración).

---

### M3 — Sodio {#m3-sodio}

**Qué dice la evidencia:** la concentración de sodio en bebida deportiva se recomienda entre 300 y 700 mg/L según duración, calor y tasa individual de sudoración. Sesiones >2h en calor aumentan la necesidad.

**Fuentes:** Thomas et al. (2016), IOC Consensus Statement (2018).

**Rangos (mg Na/L):**

| Contexto | Sodio |
|---|---|
| Sesión corta, templada | 300-400 |
| Sesión larga o templada | 400-600 |
| Sesión larga en calor | 500-700 |
| Ultra en calor extremo | 700-1000+ |

**Decisión de diseño:** salida en mg Na/h (no por litro), calculada como `sodio_h = concentración(mg/L) × líquidos_h(L)`. Más operativo para planificar.

---

### M3 — Riesgo GI {#m3-riesgo-gi}

**Qué dice la evidencia:** los trastornos gastrointestinales en ejercicio tienen origen multifactorial: mecánico (running > ciclismo por impacto), isquémico (redistribución de flujo sanguíneo en calor e intensidad alta), nutricional (alta carga osmótica, baja tolerancia, fibra reciente) y psicológico.

**Fuentes:** Costa et al. (2017) (revisión sistemática), de Oliveira & Burini (2014).

**Clasificación de riesgo:**

| Nivel | Criterio |
|---|---|
| Bajo | Ciclismo/natación, <2h, temperatura templada, historial sin problemas |
| Medio | Running moderado, 2-3h, calor ligero, ingesta cercana al techo de tolerancia |
| Alto | Running largo, calor, historial GI, CHO/h alto con baja tolerancia, alta concentración osmótica |

**Decisión de diseño:** tres niveles, sin ML. Es input para:
- Recomendar ratio glucosa:fructosa 2:1 cuando CHO/h > 60
- Sugerir entrenamiento intestinal progresivo si riesgo alto con CHO objetivo alto
- Alertar al profesional en la UI

---

## M4 — Protocolo pre-competición

> **TODO (v0.2):** poblar con protocolos de carga de CHO por duración de competición.
>
> Temas a cubrir:
> - Perfiles A-D según duración (<60', 60-90', 90-180', >180')
> - Carga de CHO (glycogen loading) D-3 a D-1
> - Reducción progresiva de fibra y grasa
> - Pauta pre-carrera 3-4h antes
> - Cafeína opcional (3-6 mg/kg 60 min antes)
> - Hiperhidratación con sodio
>
> Referencias a revisar: Burke et al. 2011, Jeukendrup 2014, Bean 2017, IOC consensus 2018.

---

## M5 — Feedback del deportista

> **TODO (tercer nivel):** poblar con evidencia sobre auto-regulación y tolerancia GI.
>
> Temas a cubrir:
> - Validez de escalas RPE y energía subjetiva
> - Tolerancia GI como marcador de sobreingestión
> - Principio de ajuste progresivo en gut training
>
> Referencias a revisar: Costa et al. 2017, Jeukendrup 2017 (gut training).
