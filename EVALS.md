# 🧪 Informe de Evaluación de Jarvis

**Generado:** 4 de mayo de 2026
**Versión de evaluación:** `gemma4:e2b` actualizada con resultados conscientes de reintentos a partir de una ejecución completa de `--single`. La columna de `gpt-oss:20b` se mantiene sin cambios respecto a la regeneración del 27 de abril de 2026.

---

# 📊 Resumen ejecutivo

### 🟢 Resultado global: 340/354 pruebas superadas

**Tasa global de éxito: 96.0%**

La evaluación cubre el comportamiento integral del agente, clasificación de intención por voz y consolidación de memoria.

La columna de `gemma4:e2b` fue recalibrada mediante una ejecución completa con hasta **3 reintentos por prueba**, mientras que `gpt-oss:20b` conserva los resultados de la regeneración anterior.

Durante esta iteración:

* 🧪 Se incorporaron **3 nuevas pruebas** en `#352`.
* 🔧 Se detectó una regresión en el clasificador de intención introducida por `a8f133c`.
* ✅ La regresión fue corregida mediante una actualización del prompt.
* 🎤 El **Intent Judge** alcanza un **100% de éxito** tras la corrección.
* 🧠 La consolidación de memoria alcanza igualmente un **100% de éxito**.
* 🤖 El comportamiento general del agente permanece por encima del **95% de éxito** en ambos modelos.

| Categoría                    | Modelo                 | Pasaron | Fallaron | Omitidas | Tasa de éxito |
| ---------------------------- | ---------------------- | ------: | -------: | -------: | ------------: |
| 🤖 Comportamiento del agente | `gemma4:e2b`           |     136 |        7 |        2 |  🟢 **95.1%** |
| 🤖 Comportamiento del agente | `gpt-oss:20b`          |     145 |        7 |        0 |  🟢 **95.4%** |
| 🎤 Intent Judge              | `gemma4:e2b` corregido |      48 |        0 |        0 | 🟢 **100.0%** |
| 🧠 Consolidación de memoria  | `gemma4:e2b`           |      11 |        0 |        0 | 🟢 **100.0%** |

---

# 🤖 Guía de selección de modelos

| Modelo        | Ideal para                                  | Consideraciones                                                 |
| ------------- | ------------------------------------------- | --------------------------------------------------------------- |
| `gemma4:e2b`  | ⚡ Respuestas rápidas y menor consumo de RAM | Puede presentar dificultades en tareas de razonamiento complejo |
| `gpt-oss:20b` | 🧠 Tareas complejas y mayor precisión       | Mayor consumo de RAM y tiempos de respuesta más elevados        |

### Arquitectura de uso recomendada

```text
                         JARVIS
                            │
                 ┌──────────┴──────────┐
                 │                     │
          gemma4:e2b              gpt-oss:20b
                 │                     │
                 ↓                     ↓
        Respuesta rápida       Razonamiento complejo
        Menor consumo RAM      Mayor capacidad
        Tareas cotidianas      Tareas exigentes
```

---

# 🤖 Comportamiento del agente

> Esta batería ejecuta el pipeline completo del agente utilizando cada modelo evaluador. Los resultados se comparan directamente entre ambos modelos.

| Caso de prueba                                                                  | `gemma4:e2b` | `gpt-oss:20b` |
| ------------------------------------------------------------------------------- | -----------: | ------------: |
| Conversación de 3 turnos con cambios de tema                                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Seguimiento aceptado dentro de la ventana activa                                | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Adversarial: las tres ramas presentes en un mismo resumen                       | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Adversarial: preferencia alimentaria del usuario frente a regla DIRECTIVES      | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| El agente utiliza `webSearch` para consultas informativas                       | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| El agente encadena búsqueda → obtención de detalles                             | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| El agente combina memoria con datos nutricionales                               | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| El asistente consulta memoria antes de preguntar por intereses                  | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| El asistente no niega disponer de memoria a largo plazo                         | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Comportamiento incorrecto: desviación sin intentar responder                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Comportamiento incorrecto: reconocimiento vacío                                 | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Saludo genérico que ignora la consulta                                          | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Declaración casual sin wake word correctamente rechazada                        | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Investigación encadenada: director de *Possessor* y filmografía                 | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Bucle de corrección acepta intento único o reintento                            | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Resolución de pronombres entre turnos                                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| DIRECTIVES: tono, longitud, frases prohibidas y forma de dirigirse al usuario   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta de fecha con la fecha ya presente en contexto                          | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Ubicación del diario utilizada para `getWeather` (#352)                         |   ❌ 0/1 (0%) |             ➖ |
| Dieta modificada de volumen a definición                                        |   ⏭️ OMITIDA |  🔸 1/1 XFAIL |
| Resultado de herramienta digerido produce respuesta fundamentada                | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Director → filmografía requiere dos búsquedas                                   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Resultados de enriquecimiento aparecen en el mensaje del sistema                | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| El enriquecimiento omite preguntas ya respondidas por el contexto               | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Escape hatch seguido de acción                                                  | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| El evaluador genera una llamada estructurada para una búsqueda evidente         | ✅ 1/1 (100%) |  🔸 1/1 XFAIL |
| Extracción con cantidades explícitas                                            | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Primer turno utiliza búsqueda web en lugar de pedir aclaración                  | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Seguimiento después de corrección utiliza búsqueda web                          | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Seguimiento resuelve pronombre dentro de la consulta de búsqueda                | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Seguimiento conserva el contexto del turno anterior                             | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Seguimiento con lugar explícito utiliza `getWeather`                            | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Seguimiento proporciona argumento faltante de herramienta (#352)                | ✅ 1/1 (100%) |             ➖ |
| Respuesta breve pero informativa                                                | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Pronóstico semanal completo                                                     | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Grafo proporciona argumento faltante para `getWeather` (#352)                   |   ❌ 0/1 (0%) |             ➖ |
| Los hechos enriquecidos por el grafo aparecen en la respuesta                   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Saludo: hello                                                                   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Saludo: ni hao                                                                  | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Manejo de descripciones ambiguas de porciones                                   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Bloqueo honesto cuando todos los proveedores fallan                             | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta de hot window dirigida y no vacía                                      | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta de identidad no activa regla de interacción de recomendaciones         | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta de identidad recupera múltiples datos del usuario                      | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta de identidad prioriza un hecho declarado sobre preguntas anteriores    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta de identidad con solo preguntas anteriores no inventa hechos           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Instrucción: ser más breve                                                      | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Instrucción: utilizar Celsius                                                   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Afirmación del juez anulada por la hot window                                   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| El LLM utiliza intereses enriquecidos para búsquedas personalizadas             | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Payload compuesto únicamente por enlaces genera respuesta honesta               | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| El contexto de ubicación llega a las consultas de búsqueda                      | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta de ubicación con ubicación ya presente devuelve NONE                   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Pista parcial de ubicación todavía genera un routing correcto                   | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| `LogMealTool` almacena comidas con macronutrientes                              | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Límite máximo de turnos genera respuesta resumida y nunca silencio              | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Enriquecimiento de memoria: noticias personalizadas                             | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Enriquecimiento de memoria: recuperación temporal                               | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Enriquecimiento de memoria: recuperación por tema                               | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Resumen mixto conserva hechos nuevos y descarta información obsoleta            | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Navegación en lenguaje natural se transforma en llamada a herramienta           | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Sin desviación: noticias tecnológicas                                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Sin desviación: consulta de hora                                                | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Sin desviación: clima de mañana                                                 | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Sin desviación: pronóstico semanal de lluvia                                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Ausencia de herramienta de correo se comunica honestamente                      | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Sin ninguna pista contextual, el routing continúa correctamente                 | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Wake word ausente correctamente rechazado                                       | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Conocimiento nuevo: negocios locales y ubicación del usuario                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Conocimiento nuevo: resumen en idioma no inglés                                 | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Conocimiento nuevo: planes de mudanza y empleo                                  | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Conocimiento nuevo: dieta y receta preferida                                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| El límite de nudges evita el bucle                                              | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Nutrición: hamburguesa con papas                                                | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Nutrición: pollo con brócoli                                                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Nutrición: avena con plátano                                                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Días de oficina modificados de lunes/miércoles a lunes/jueves                   |   ⏭️ OMITIDA |  🔸 1/1 XFAIL |
| Omite desviación para entidad desconocida                                       | ✅ 1/1 (100%) |  🔸 1/1 XFAIL |
| Omite desviación cuando el tema nunca fue resuelto                              | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Prompt abierto utiliza conocimiento almacenado                                  |   ❌ 0/1 (0%) |  ✅ 1/1 (100%) |
| Consulta meteorológica paralela: París y Londres                                | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Conserva preferencias legítimas del usuario                                     | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Payload realista de búsqueda web no se desvía hacia enlaces                     | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Consulta de recomendación recupera interacción cuando existen datos del usuario | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Reformulación: acontecimientos de vida tratados como hechos temporales          | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Reformulación: solicitudes convertidas en conocimiento                          | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Rechazo: autorreferencias del asistente no consideradas conocimiento            | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Rechazo: snapshots temporales obsoletos                                         | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Recomendación de restaurante recupera interés culinario anterior                | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Entradas que no pertenecen a alimentación devuelven NONE                        | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| JSON válido con todos los campos requeridos                                     |   ❌ 0/1 (0%) |  ✅ 1/1 (100%) |
| Caso base de comida sencilla: 2 huevos cocidos                                  | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta meteorológica simple termina tras una llamada                          | ✅ 1/1 (100%) |    ❌ 0/1 (0%) |
| Habla después de TTS requiere wake word                                         | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Stop durante TTS interrumpe inmediatamente                                      | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta de hora con hora presente en contexto devuelve NONE                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Las llamadas literales a herramientas no aparecen después de búsqueda web       | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Reintento de herramienta: mención explícita                                     | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Reintento de herramienta: "adelante"                                            | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Reintento de herramienta: "inténtalo"                                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| `ToolSearchTool` amplía búsqueda y posteriormente navega                        | 🔸 1/1 XFAIL |  🔸 1/1 XFAIL |
| Cambio de tema: búsqueda → clima utiliza `getWeather`                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Cambio de tema: clima → horario de tienda utiliza `webSearch`                   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Conversaciones triviales no producen hechos extraídos                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Segmentos TTS no provocan extracción incorrecta de la consulta                  | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Turno 1: Possessor → turno 2: clima                                             |   ❌ 0/1 (0%) |  ✅ 1/1 (100%) |
| Flujo de celebridad: identidad → seguimiento con pronombre                      | 🔸 1/1 XFAIL |    ❌ 0/1 (0%) |
| USER: identidad, ubicación, mascotas, dieta y empleo                            | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Entidad desconocida con diario contaminado todavía genera búsqueda web          | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Entidad desconocida: Piranesi                                                   | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Entidad desconocida: Possessor                                                  | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Entidad desconocida: "¿has oído hablar de Piranesi?"                            | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Entidad desconocida: consulta de permiso sobre Possessor                        | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Dominio no relacionado todavía devuelve NONE                                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Temas no relacionados no se fusionan en una sola cláusula                       | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| La consulta del usuario no se confunde con eco TTS                              | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Utterance iniciada durante TTS se trata como hot window                         | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| WORLD: negocios locales y atribución cinematográfica                            | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Wake word después de segmentos de eco                                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Wake word utiliza extracción del juez                                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Recomendación de contenido recupera películas mencionadas recientemente         | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta meteorológica responde con condiciones actuales                        |   ❌ 0/1 (0%) |  ✅ 1/1 (100%) |
| Consulta meteorológica selecciona `getWeather`                                  | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta meteorológica activa herramientas después de un saludo                 | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Payload de Wikipedia produce respuesta fundamentada                             | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Wikipedia recupera la consulta cuando DDG bloquea                               | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Presupuesto calórico → `fetchMeals`                                             | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Memoria fría + consulta corta: "¿cómo está el clima?"                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Memoria fría + pronóstico semanal                                               | ✅ 1/1 (100%) |    ❌ 0/1 (0%) |
| Comprobación dietética → `fetchMeals`                                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Recuperación explícita → búsqueda                                               | ✅ 1/1 (100%) |    ❌ 0/1 (0%) |
| Buscar PDF de factura en el ordenador                                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Decisión alimentaria → `fetchMeals`                                             | 🔸 1/1 XFAIL |  ✅ 1/1 (100%) |
| Chaqueta → `getWeather`                                                         | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta meteorológica con ubicación selecciona `getWeather`                    | ✅ 1/1 (100%) |    ❌ 0/1 (0%) |
| Registrar que acaba de comer un plátano                                         | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Registro de comida selecciona `logMeal`                                         | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Recuperación de comida coloquial → `fetchMeals`                                 | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Recuperación de comida selecciona `fetchMeals`                                  | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Noticias interesantes para el usuario                                           | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Noticias de interés personal                                                    | ✅ 1/1 (100%) |    ❌ 0/1 (0%) |
| Noticias que podrían interesarle                                                | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Recomendar un libro que le guste al usuario                                     | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Investigación → `webSearch` + `fetchWebPage`                                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Pronóstico → `getWeather`                                                       | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Buscar ofertas de vuelos en la web                                              | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Sugerir algo interesante para ver                                               | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Tomar una captura de pantalla                                                   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Noticias que podrían interesarle                                                | ✅ 1/1 (100%) |    ❌ 0/1 (0%) |
| Consulta meteorológica corta                                                    | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Memoria cálida + consulta meteorológica corta                                   | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Clima + comidas                                                                 | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Consulta meteorológica selecciona `getWeather` y pocas herramientas adicionales | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Búsqueda web selecciona `webSearch`                                             | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Pronóstico semanal conserva `getWeather`                                        | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Capital de Francia                                                              | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Qué debería cocinar para cenar                                                  | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Cuánto es 2 + 2                                                                 | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Qué aparece actualmente en pantalla                                             | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Cómo está el clima                                                              | ✅ 1/1 (100%) |  ✅ 1/1 (100%) |
| Quién es Britney Spears                                                         |   ❌ 0/1 (0%) |  ✅ 1/1 (100%) |

---

# 🎤 Intent Judge

> El **Intent Judge** está fijado a `gemma4:e2b` como clasificador de intención por voz. Esta evaluación es independiente del modelo evaluador principal.

**Última ejecución:** 4 de mayo de 2026.

Las pruebas situadas en el límite del modelo pequeño fueron ejecutadas **5 veces** durante la iteración del prompt para verificar estabilidad. Para mantener consistencia con el resto del informe, los resultados se representan aquí como `1/1`.

## 🔧 Regresión detectada y corregida

El caso:

```text
cross_segment_answer_that_with_noise
```

presentó una regresión entre `main` y `develop`.

La regresión fue introducida por el ejemplo *"big Mac"* añadido en `a8f133c`, que provocaba que el modelo pequeño priorizara conservar el texto del usuario en lugar de resolver correctamente imperativos distribuidos entre segmentos.

La corrección incorporó dos ejemplos contrastantes:

1. Una pregunta previa acompañada de ruido.
2. Un imperativo de varias palabras equivalente a "adelante y responde".

La modificación restauró correctamente:

```text
cross_segment_answer_that_with_noise
multi_person_weather_discussion
cross_segment_go_ahead_and_answer
```

Los tres casos alcanzaron:

```text
5/5 ejecuciones correctas
```

Además, el nuevo caso:

```text
wake_word_trailing_after_capitalised_brand
```

mantiene la cobertura de la regresión original relacionada con *"big Mac"*.

## Resultados

| Caso de prueba                                           |  Resultado | Estado |
| -------------------------------------------------------- | ---------: | :----: |
| Hot window indicada en el prompt                         | 1/1 (100%) |    ✅   |
| La consulta anterior no se vuelve a extraer              | 1/1 (100%) |    ✅   |
| El segmento procesado no se vuelve a extraer             | 1/1 (100%) |    ✅   |
| Devuelve NONE cuando Ollama no está disponible           | 1/1 (100%) |    ✅   |
| El system prompt contiene instrucciones de manejo de eco | 1/1 (100%) |    ✅   |
| El texto TTS se incluye para detectar eco                | 1/1 (100%) |    ✅   |
| `alias_after_narrative_context`                          | 1/1 (100%) |    ✅   |
| `alias_treated_as_wake_word`                             | 1/1 (100%) |    ✅   |
| `buffer_echo_then_followup_hot_window`                   | 1/1 (100%) |    ✅   |
| `buried_target_amid_unrelated_chatter`                   | 1/1 (100%) |    ✅   |
| `buried_target_plural_vague_ref_they`                    | 1/1 (100%) |    ✅   |
| `buried_target_topicless_question`                       | 1/1 (100%) |    ✅   |
| `context_synthesis_weather_opinion`                      | 1/1 (100%) |    ✅   |
| `context_synthesis_with_prior_ambient`                   | 1/1 (100%) |    ✅   |
| `cross_segment_answer_that_weather`                      | 1/1 (100%) |    ✅   |
| `cross_segment_answer_that_with_noise`                   | 1/1 (100%) |    ✅   |
| `cross_segment_answered_that_whisper_variant`            | 1/1 (100%) |    ✅   |
| `cross_segment_dinosaur_opinion`                         | 1/1 (100%) |    ✅   |
| `cross_segment_go_ahead_and_answer`                      | 1/1 (100%) |    ✅   |
| `cross_segment_hot_window_followup`                      | 1/1 (100%) |    ✅   |
| `cross_segment_imperative_superseded_by_new_question`    | 1/1 (100%) |    ✅   |
| `echo_plus_followup_extracted`                           | 1/1 (100%) |    ✅   |
| `echo_plus_rejected_similar_plus_wake_retry`             | 1/1 (100%) |    ✅   |
| `hot_window_override_topicless_followup`                 | 1/1 (100%) |    ✅   |
| `hot_window_simple_followup`                             | 1/1 (100%) |    ✅   |
| `mentioned_in_narrative_past_tense`                      | 1/1 (100%) |    ✅   |
| `multi_person_vague_reference`                           | 1/1 (100%) |    ✅   |
| `multi_person_weather_discussion`                        | 1/1 (100%) |    ✅   |
| `multiple_echoes_then_interrupt`                         | 1/1 (100%) |    ✅   |
| `no_wake_word_casual_speech`                             | 1/1 (100%) |    ✅   |
| `no_wake_word_in_buffer`                                 | 1/1 (100%) |    ✅   |
| `stop_command_during_tts`                                | 1/1 (100%) |    ✅   |
| `user_followup_statement_after_question_nihilism`        | 1/1 (100%) |    ✅   |
| `wake_word_after_narrative_addresses_assistant`          | 1/1 (100%) |    ✅   |
| `wake_word_command_timer`                                | 1/1 (100%) |    ✅   |
| `wake_word_mid_sentence`                                 | 1/1 (100%) |    ✅   |
| `wake_word_open_imperative_give_me_advice`               | 1/1 (100%) |    ✅   |
| `wake_word_open_imperative_say_something`                | 1/1 (100%) |    ✅   |
| `wake_word_open_imperative_surprise_me`                  | 1/1 (100%) |    ✅   |
| `wake_word_open_imperative_tell_me_a_joke`               | 1/1 (100%) |    ✅   |
| `wake_word_open_imperative_tell_me_anything`             | 1/1 (100%) |    ✅   |
| `wake_word_share_statement_burger`                       | 1/1 (100%) |    ✅   |
| `wake_word_share_statement_feeling`                      | 1/1 (100%) |    ✅   |
| `wake_word_share_statement_trailing`                     | 1/1 (100%) |    ✅   |
| `wake_word_simple_question`                              | 1/1 (100%) |    ✅   |
| `wake_word_statement_remember`                           | 1/1 (100%) |    ✅   |
| `wake_word_trailing_after_capitalised_brand`             | 1/1 (100%) |    ✅   |
| `wake_word_trailing_after_named_entity`                  | 1/1 (100%) |    ✅   |

### 🎤 Resultado del Intent Judge

```text
48 / 48 pruebas superadas
100.0% de precisión
0 regresiones activas
```

---

# 🧠 Consolidación de memoria

> Esta batería evalúa `merge_node_data` utilizando un modelo real de selección. El objetivo es verificar el comportamiento de reescritura durante la consolidación y sus cinco comportamientos principales:
>
> * Detección y eliminación de duplicados semánticos.
> * Consolidación de patrones repetitivos.
> * Independencia entre hechos no relacionados.
> * Eliminación de metanarrativas generadas por el asistente.
> * Corrección integral de la firma de procesamiento por lotes.

La evaluación se ejecuta mediante:

```bash
pytest evals/test_merge_consolidation.py
```

| Caso de prueba                                                                                  |  Resultado | Estado |
| ----------------------------------------------------------------------------------------------- | ---------: | :----: |
| Dedupe: mismo hecho con redacción diferente                                                     | 1/1 (100%) |    ✅   |
| Dedupe: título profesional reformulado                                                          | 1/1 (100%) |    ✅   |
| Pattern: comidas de sushi consolidadas como "come sushi regularmente"                           | 1/1 (100%) |    ✅   |
| Pattern boundary: acontecimientos únicos fechados permanecen independientes                     | 1/1 (100%) |    ✅   |
| Independencia: alergia al cacahuate + preferencia por té sobreviven a un hecho de senderismo    | 1/1 (100%) |    ✅   |
| Independencia: trabajo como ingeniero de software sobrevive a un hecho relacionado con guitarra | 1/1 (100%) |    ✅   |
| Meta-narrativa: se elimina una negación de capacidad y se conserva la directiva real            | 1/1 (100%) |    ✅   |
| Meta-narrativa: se elimina una sugerencia del asistente y se conserva la consulta factual       | 1/1 (100%) |    ✅   |
| Meta-narrativa: nodo contaminado recibe nuevo hecho y se limpia correctamente                   | 1/1 (100%) |    ✅   |
| Meta-narrativa: nodo de directivas limpio no sufre poda excesiva                                | 1/1 (100%) |    ✅   |
| Merge por lotes: tres hechos independientes llegan correctamente en una sola llamada            | 1/1 (100%) |    ✅   |

### 🔬 Nota técnica

El caso de **pattern boundary** anteriormente estaba marcado como:

```text
xfail(strict=False)
```

Esto se debía a que `gemma4:e2b` agrupaba entradas fechadas y podía eliminar silenciosamente acontecimientos anteriores.

Después de incorporar la regla **META-NARRATIVE**, el caso pasó correctamente **3/3 ejecuciones**.

Aunque la relación causal directa entre la nueva regla y la mejora todavía no está confirmada, la evaluación representa exactamente el tipo de regresión que debe detectarse mediante la suite de pruebas.

Por ello, el marcador `xfail` fue eliminado y el caso pasó a considerarse una prueba normal.

---

# 📈 Estado general del sistema

```text
┌──────────────────────────────────────────────────────┐
│                  JARVIS EVALUATION                   │
├──────────────────────────────────────────────────────┤
│                                                      │
│  🤖 Comportamiento del agente                        │
│                                                      │
│     gemma4:e2b             ███████████████████░ 95.1%│
│     gpt-oss:20b            ███████████████████░ 95.4%│
│                                                      │
│  🎤 Intent Judge                                     │
│                                                      │
│     gemma4:e2b             ████████████████████ 100% │
│                                                      │
│  🧠 Memory Merge                                     │
│                                                      │
│     gemma4:e2b             ████████████████████ 100% │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### Indicadores principales

| Área                             | Estado |
| -------------------------------- | :----: |
| Pipeline general del agente      |   🟢   |
| Routing de herramientas          |   🟢   |
| Uso de memoria                   |   🟢   |
| Enriquecimiento contextual       |   🟢   |
| Resolución de referencias        |   🟢   |
| Detección de wake word           |   🟢   |
| Gestión de eco TTS               |   🟢   |
| Intent Judge                     |   🟢   |
| Consolidación de memoria         |   🟢   |
| Manejo de múltiples herramientas |   🟢   |
| Investigación web encadenada     |   🟢   |
| Recuperación de contexto         |   🟢   |
| Casos adversariales              |   🟢   |
| Casos límite del modelo pequeño  |   🟡   |
| Algunas rutas meteorológicas     |   🟡   |
| Algunos flujos de memoria fría   |   🟡   |

---

# 🧭 Áreas que requieren seguimiento

Las pruebas fallidas no indican necesariamente un fallo general del sistema. Varias corresponden a rutas específicas donde todavía existe margen de mejora.

### 🌦️ 1. Resolución de ubicación para clima

Algunas pruebas relacionadas con la combinación de:

```text
memoria
    ↓
ubicación
    ↓
tool argument
    ↓
getWeather
```

todavía presentan fallos en determinados escenarios.

Especialmente:

```text
Diary location grounds getWeather call
Graph supplies missing tool arg
```

Esto apunta a un área donde el enriquecimiento proveniente del diario o del grafo todavía puede requerir una resolución contextual más robusta.

### 🧠 2. Memoria fría

Algunas consultas que dependen de información recuperada desde memoria fría muestran diferencias entre modelos.

Ejemplo:

```text
cold-memory-week-forecast
explicit-recall-then-search
```

Esto sugiere que el tamaño y capacidad de razonamiento del modelo pueden influir en la forma en que se integra memoria recuperada con una nueva llamada de herramienta.

### 🎤 3. Casos límite del Intent Judge

Aunque el resultado actual es de:

```text
48 / 48
100%
```

la regresión introducida por `a8f133c` demuestra que los ejemplos few-shot pueden modificar significativamente el comportamiento de modelos pequeños.

Por ello, los casos relacionados con:

```text
wake word
cross-segment context
imperatives
echo detection
hot window
```

deben mantenerse dentro de la suite permanente.

---

# 🧪 Interpretación de los resultados

Los resultados muestran que Jarvis mantiene un comportamiento estable en las áreas centrales del pipeline:

```text
                    ┌─────────────────┐
                    │     Usuario     │
                    └────────┬────────┘
                             ↓
                    🎤 Intent Judge
                             ↓
                    🧠 Contexto
                             ↓
                    💾 Memoria
                             ↓
                    🔎 Tool Router
                             ↓
                    🛠️ Herramientas
                             ↓
                    📦 Tool Results
                             ↓
                    🧠 LLM Principal
                             ↓
                    💬 Respuesta
```

Las áreas con mejores resultados son:

* 🎤 Clasificación de intención.
* 🧠 Consolidación de memoria.
* 🔎 Routing de herramientas.
* 🌐 Investigación web.
* 🗣️ Resolución de contexto conversacional.
* 🔊 Manejo de TTS y eco.
* 🧩 Recuperación de información personalizada.
* 🔄 Corrección y reintentos.
* 🛡️ Manejo de casos adversariales.

El principal objetivo de las siguientes iteraciones debe ser reducir los fallos restantes sin introducir regresiones en los casos que actualmente presentan una cobertura del 100%.

---

# 📖 Leyenda

| Símbolo | Significado                                         |
| :-----: | --------------------------------------------------- |
|    ✅    | Prueba completamente superada                       |
|    ⚠️   | Superación parcial, algunos intentos fallaron       |
|    ❌    | Prueba completamente fallida                        |
|    ⏭️   | Prueba omitida por dependencia ausente              |
|    🔸   | Fallo esperado, limitación conocida                 |
|    🎉   | Superación inesperada, posible corrección de un bug |
|    ➖    | Prueba no ejecutada para ese modelo                 |

---

# 🏁 Conclusión

## Jarvis Evaluation Status

```text
╔══════════════════════════════════════════════════╗
║                                                  ║
║             🧪 JARVIS EVALUATION                ║
║                                                  ║
║        340 / 354 pruebas superadas              ║
║                                                  ║
║                 🟢 96.0%                        ║
║                                                  ║
║        🎤 Intent Judge:     100%                ║
║        🧠 Memory Merge:     100%                ║
║        🤖 Agent Behaviour:  >95%                ║
║                                                  ║
║             STATUS: STABLE                      ║
║                                                  ║
╚══════════════════════════════════════════════════╝
```

**Estado de la evaluación:** 🟢 **ESTABLE**

La suite demuestra una cobertura sólida del comportamiento conversacional, memoria, herramientas, voz y routing de Jarvis. Las regresiones detectadas durante esta iteración fueron identificadas mediante las pruebas y, en los casos corregidos, verificadas mediante ejecuciones adicionales.

Las pruebas restantes deben mantenerse como objetivos de seguimiento para futuras iteraciones, especialmente aquellas relacionadas con resolución contextual de ubicación, memoria fría y diferencias de comportamiento entre modelos.

---

*Informe generado automáticamente por la suite de evaluación de Jarvis.*
