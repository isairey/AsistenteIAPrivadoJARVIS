# Mapa de Contextos de LLM

Cada llamada distinta a un LLM en Jarvis, qué información la alimenta, qué consume su salida y bajo qué condiciones se ejecuta. Esta es la referencia principal para optimizar el cuello de botella más importante de la aplicación: la latencia del LLM. Mantén este documento sincronizado con el código; consulta la nota al final.

---

## 1. Bucle Principal de Respuesta (bucle de mensajes agénticos)

* **Archivo**: `src/jarvis/reply/engine.py` — `reply()` y el bucle alrededor de las líneas 1370-1650; ruta nativa de llamadas a herramientas en `chat_with_messages()` (~1424, 1455).
* **Disparador**: cada mensaje del usuario. Ejecuta hasta `agentic_max_turns` (por defecto 8) iteraciones por respuesta.
* **Modelo / control**: `cfg.ollama_chat_model` (modelo grande). No es opcional. No existe una bifurcación por tamaño en el propio bucle; la bifurcación por tamaño afecta a los resúmenes/digests y al evaluador que lo rodean.
* **Entradas**:

  * Consulta del usuario con datos sensibles redactados.
  * Diálogo reciente (últimos 5 minutos), incluyendo llamadas a herramientas y mensajes con rol `tool` de respuestas anteriores dentro de la conversación activa (herencia de herramientas, `DialogueMemory.record_tool_turn` / `get_recent_turns_with_tools` en `src/jarvis/memory/conversation.py`; límite por prompt mediante `cfg.tool_carryover_max_turns` / `tool_carryover_per_entry_chars`; límite de almacenamiento `_tool_turns_max_storage = 16`; se limpia con la señal `stop` y al iniciar una nueva conversación; los marcadores de fence `UNTRUSTED WEB EXTRACT` se conservan al truncar; tanto `content` como `tool_calls[*].function.arguments` se limpian al escribir).
  * Prompt de sistema unificado de `src/jarvis/system_prompt.py` + nota de ASR + instrucciones del protocolo de herramientas.
  * **Bloque de perfil en caliente (Warm Profile)**: fragmento independiente de la consulta con información del usuario + directivas procedentes del grafo de conocimiento, generado mediante `build_warm_profile()` / `format_warm_profile_block()` en `src/jarvis/memory/graph_ops.py`, en el paso 3.5 de `reply()`. No realiza ninguna llamada al LLM; es una lectura directa de SQLite. Se inyecta siempre para que la personalización sea el comportamiento predeterminado.

    * El resultado se almacena en caché en `DialogueMemory._hot_cache` usando `DialogueMemory.WARM_PROFILE_CACHE_KEY` durante toda la conversación activa.
    * Se invalida al ejecutar `stop`, al iniciar una nueva conversación y cuando se modifican nodos de User/Directives del grafo mediante el listener registrado en `src/jarvis/daemon.py` contra `register_graph_mutation_listener` de `src/jarvis/memory/graph.py`.
    * Las escrituras de la rama World se ignoran.
  * Enriquecimiento mediante memoria resumida/digerida (opcional, consultar #4).
  * Contexto de hora + ubicación, reinyectado en cada turno.
  * Esquema de herramientas: nativo mediante `generate_tools_json_schema()` (`src/jarvis/tools/registry.py`) o fallback de texto mediante `_text_tool_call_guidance()` (`engine.py:68`).
  * Resultados de herramientas de turnos anteriores (sin procesar o digeridos; consultar #5).
* **Salida**: `{content, tool_calls, thinking}` con formato similar al de OpenAI. La salida es consumida por el orquestador de herramientas y el pipeline TTS. El contenido en lenguaje natural se entrega inmediatamente; no se ejecuta un evaluador posterior al turno.
* **Límites**:

  * `num_ctx: 8192` explícito.
  * Timeout `llm_chat_timeout_sec` (45 s).
  * Fallback automático de llamadas nativas a herramientas hacia llamadas de texto cuando HTTP devuelve 400 (`ToolsNotSupportedError`), quedando activo para el resto de la sesión.
  * Riesgo: `fetch_web_page` trunca a 50.000 caracteres (~37k tokens). Esto se mitiga en modelos SMALL mediante el digest del resultado de herramienta (#5), que comprime el contenido antes de introducirlo en el historial de mensajes.
  * Los modelos LARGE reciben el contenido sin procesar y pueden encontrarse silenciosamente con un contexto truncado.

---

## 2. Evaluador de Intención

* **Archivo**: `src/jarvis/listening/intent_judge.py` — `IntentJudge.evaluate()`.
* **Disparador**: en un segmento de voz únicamente si existe una señal de interacción (se detectó la palabra de activación, la ventana caliente está activa o se está reproduciendo TTS). El habla ambiental pura evita esta llamada.
* **Modelo / control**: `cfg.intent_judge_model` (por defecto `gemma4:e2b`, ~2B). Si Ollama no está disponible, utiliza la detección de palabra de activación basada en texto.
* **Entradas**:

  * Buffer de transcripción móvil de los últimos 120 s, con marcas de tiempo.
  * Marca de tiempo de la palabra de activación, si existe, con alias normalizados.
  * Último texto reproducido mediante TTS + momento de finalización, para rechazo de eco.
  * Indicadores de estado (`wake_word_mode`, `hot_window_mode`, `during_tts`).
* **Prompt de sistema**: `SYSTEM_PROMPT_TEMPLATE` en `intent_judge.py:135`. Enseña extracción de consultas, detección de eco, comandos de parada, desambiguación de pronombres/temas, reasignación mediante imperativos y declaraciones dirigidas a la palabra de activación.
* **Salida**: JSON estricto `IntentJudgment{directed, query, stop, confidence, reasoning}` (`intent_judge.py:94`). Lo consume la máquina de estados de escucha, que lo envía al motor de respuesta.
* **Límites**:

  * `intent_judge_timeout_sec` (15 s).
  * `num_ctx: 8192` explícito.
  * El prompt de sistema ocupa ~2k tokens después del PR #362 y el buffer de transcripción, con `transcript_buffer_duration_sec=120`, puede alcanzar ~1.5k tokens en escenas con varios hablantes.
  * 4096 tokens dejaba solo ~10 % de margen y podía provocar truncamiento silencioso de Ollama en la parte final del prompt, donde se encuentran los ejemplos few-shot y el bloque `TRANSCRIPT NOISE`.

---

## 3. Extractor de Enriquecimiento de Memoria

* **Archivo**: `src/jarvis/reply/enrichment.py` — `extract_search_params_for_memory()` (~línea 71).
* **Disparador**: una vez por respuesta, **solo cuando el planificador previo (#12) genera una directiva `searchMemory` o devuelve un plan vacío (fail-open)**.
* Los planes de respuesta pura omiten completamente esta llamada, ahorrando una llamada al LLM en saludos y conversaciones pequeñas.
* **Modelo / control**: resuelto mediante `resolve_tool_router_model(cfg)`:

  * `tool_router_model`
  * `intent_judge_model`
  * `ollama_chat_model`
* Es una tarea pequeña de clasificación y utiliza el mismo modelo pequeño/cálido que el router.
* **Entradas**:

  * Consulta del usuario, con la sugerencia `topic` del planificador cuando existe.
  * Sugerencia contextual opcional, como resumen compacto del contexto actual.
  * Fecha/hora actual en UTC.
* **Prompt de sistema**: definido directamente en `enrichment.py:35-63`.
* **Salida**: `{keywords, from?, to?, questions?}`. La consume la búsqueda de memoria del motor de respuestas.
* **Límites**:

  * Hasta 2 reintentos.
  * Timeout proveniente de `llm_tools_timeout_sec`.
* **Caché**:

  * Se almacena en `DialogueMemory._hot_cache`.
  * Clave: `enrichment:{redacted_query[+topic_hint]}`.
  * Permanece durante toda la conversación activa.
  * Consultas de seguimiento idénticas reutilizan el diccionario y evitan otra llamada al LLM.
  * Se elimina mediante `clear_hot_cache()` con la señal `stop` y al iniciar una nueva conversación.

---

## 3b. Puerta de Recuperación de Memoria (Recall Gate)

* **Archivo**: `src/jarvis/memory/recall_gate.py` — `should_recall()`.
* **Disparador**: una vez por respuesta, antes de ejecutar el enriquecimiento del diario/grafo/digest, después de que el planificador haya determinado que potencialmente se necesita memoria.
* **Modelo / control**: **NO utiliza LLM**. Es una heurística determinista de cobertura de palabras clave.
* **Entradas**:

  * Consulta.
  * Diálogo reciente, incluyendo filas de herencia de herramientas.
* **Salida**:

  * Devuelve `False` únicamente si la ventana caliente contiene un resultado reciente de herramienta **y** al menos el 50 % de las palabras de contenido de la consulta aparecen en la transcripción de dicha ventana.
  * En ese caso se omiten el diario, el grafo y el digest de memoria para esa respuesta.
  * En cualquier otro caso devuelve `True`.
  * Ante cualquier excepción utiliza fail-open.
* La extracción de palabras de contenido usa `\w{3,}` con `re.UNICODE`, por lo que funciona con latín, cirílico, CJK, árabe, hebreo, etc.
* Las palabras coincidentes se pasan por `redact()` antes de escribirse en los logs de depuración.
* **Prioridad del planificador**:

  * Cuando el planificador genera explícitamente un paso `searchMemory`, la puerta se ignora.
  * El planificador tiene más información semántica que la cobertura de palabras.
  * La puerta únicamente puede saltarse el flujo fail-open de plan vacío.
* **Motivo**: evita repetir búsquedas en diario/grafo cuando la ventana caliente ya contiene el contexto necesario para responder a una continuación, por ejemplo:

  * "¿Cuál es su canción más famosa?"
  * después de una búsqueda web sobre Bieber.

---

## 4. Digest de Memoria (opcional, modelos SMALL)

* **Archivo**: `src/jarvis/reply/enrichment.py` — `digest_memory_for_query()` + `_distil_batch()`.
* **Disparador**: una vez por respuesta cuando el enriquecimiento devuelve resultados y `memory_digest_enabled` está activo.
* Por defecto está desactivado; `null` significa:

  * **ON automáticamente** para modelos SMALL ≤7B.
  * **OFF automáticamente** para modelos LARGE.
* Se omite si el contenido sin procesar tiene menos de `_DIGEST_MIN_CHARS` (400 caracteres).
* Si supera `_DIGEST_BATCH_MAX_CHARS` (2000), se divide en lotes.
* **Modelo / control**: `ollama_chat_model`.
* **Entradas**:

  * Consulta del usuario.
  * Entradas sin procesar del diario.
  * Nodos sin procesar del grafo.
* **Prompt de sistema**: `_DIGEST_SYSTEM_PROMPT` en `enrichment.py:122`.
* Enseña filtrado por relevancia, detección de preferencias, conservación de atribución, sentinel `NONE` y consultas de identidad.
* **Salida**: máximo 400 caracteres por lote (`_DIGEST_MAX_CHARS`), inyectados como contexto de memoria únicamente de referencia en el mensaje de sistema del bucle principal.
* **Límites**: `llm_digest_timeout_sec` (8 s, compartido).

---

## 5. Digest de Resultados de Herramientas (opcional, activable)

* **Archivo**: `src/jarvis/reply/enrichment.py` — `digest_tool_result_for_query()` + `_distil_tool_batch()`.
* **Disparador**: después de cada resultado de herramienta en el bucle, si `tool_result_digest_enabled` está activo.
* Por defecto `null` significa:

  * **ON automáticamente** para SMALL ≤7B.
  * **OFF automáticamente** para LARGE.
* Su principal objetivo en modelos pequeños es evitar que los resultados de `fetch_web_page` de hasta 50.000 caracteres llenen la ventana `num_ctx=8192`.
* Se omite si el resultado tiene menos de 400 caracteres (`_TOOL_DIGEST_MIN_CHARS`).
* Se divide en lotes si supera 2500 caracteres (`_TOOL_DIGEST_BATCH_MAX_CHARS`).
* **Modelo / control**: `ollama_chat_model`.
* **Entradas**:

  * Consulta del usuario.
  * Nombre de la herramienta.
  * Resultado bruto de la herramienta, por ejemplo el contenido de `webSearch` dentro del fence `UNTRUSTED WEB EXTRACT`.
* **Prompt de sistema**: `_TOOL_DIGEST_SYSTEM_PROMPT`.
* Enseña extracción de hechos atribuidos, sentinel `NONE` y prohibición de inferencias.
* **Salida**: máximo 600 caracteres por lote (`_TOOL_DIGEST_MAX_CHARS`), reemplazando el contenido bruto en el flujo de mensajes.
* Si devuelve `NONE`, se utiliza el resultado bruto.
* **Límites**: `llm_digest_timeout_sec` (8 s, compartido).

---

## 6. Digest del Bucle al Alcanzar el Máximo de Turnos

* **Archivo**: `src/jarvis/reply/enrichment.py` — `digest_loop_for_max_turns()` (~línea 847).
* **Disparador**: cuando el bucle agéntico consume `agentic_max_turns` sin producir una respuesta en lenguaje natural, por ejemplo un bucle compuesto únicamente por llamadas a herramientas.
* El evaluador ya no controla este comportamiento; la terminación ocurre inmediatamente cuando existe contenido.
* **Modelo / control**: `_resolve_loop_digest_model(cfg)`:

  * Prefiere `intent_judge_model`.
  * Si no existe, utiliza `ollama_chat_model`.
* **Entradas**:

  * Consulta del usuario.
  * Actividad del bucle: llamadas a herramientas, resúmenes de resultados y cualquier texto generado.
* **Prompt de sistema**: `_LOOP_DIGEST_SYSTEM_PROMPT`, con advertencia inicial, idioma del usuario y respuesta concisa.
* **Salida**: respuesta final con advertencia.
* Ante un error utiliza como fallback el último candidato bruto o un error genérico.
* **Límites**: `llm_digest_timeout_sec` (8 s, compartido).

---

## 7. Router de Herramientas (selección previa al bucle)

* **Archivo**: `src/jarvis/tools/selection.py` — `select_tools_with_llm()` (~línea 331).
* **Disparador**: una vez por respuesta, **al principio del flujo y antes del planificador (#12)**.
* Siempre se ejecuta: el router es el selector autorizado de herramientas y su catálogo reducido es el que recibe el planificador.
* Cuando el planificador posteriormente referencia herramientas, sus nombres se combinan con la lista permitida del router, pero no la reemplazan.
* Esto es importante porque los modelos pequeños tienden a elegir `webSearch` por defecto cuando una herramienta específica como `getWeather` sería más adecuada.
* `tool_selection_strategy == "llm"` es la estrategia predeterminada. También existen:

  * `all`
  * `keyword`
  * `embedding`
* **Modelo / control**:

  * `tool_router_model`
  * `intent_judge_model`
  * `ollama_chat_model`
* **Entradas**:

  * Consulta del usuario.
  * Catálogo de herramientas, tanto integradas como MCP, con descripciones.
  * Sugerencia opcional para limitar el catálogo.
* **Prompt de sistema**: integrado aproximadamente en las líneas 260-315.
* Enseña a seleccionar hasta 5 herramientas o `none`.
* **Salida**:

  * Nombres de herramientas separados por comas.
  * `none` cuando no necesita herramientas.
  * Límite `_LLM_MAX_SELECTED` = 5.
  * Las herramientas siempre incluidas (`stop`, `toolSearchTool`) se agregan independientemente del resultado.
* **Límites**:

  * `llm_timeout_sec`.
  * Ante fallo → todas las herramientas.
* **Caché**:

  * `routed_tools` se almacena en `DialogueMemory._hot_cache`.
  * Clave:
    `router:{redacted_query}|{strategy}|{builtin-names}|{mcp-names}`
  * Permanece durante la conversación activa.
  * La firma del catálogo permite invalidar el caché cuando se actualiza MCP durante una conversación.
  * `context_hint` se excluye deliberadamente para que los cambios de hora/ubicación no invaliden el caché.
  * Se limpia mediante `clear_hot_cache()` al ejecutar `stop` y al iniciar una nueva conversación.
* **Protección de herencia de herramientas**:

  * Después de consultar/escribir el caché, el motor inspecciona las llamadas a herramientas del turno anterior.
  * Si una herramienta anterior terminó con `success=False` en `ToolExecutionResult`, el nombre de esa herramienta vuelve a agregarse a `routed_tools` únicamente para el turno actual.
  * Esto compensa errores de routers pequeños cuando el usuario proporciona información faltante.
  * Ejemplo:

    * Un `getWeather` falla.
    * Usuario: "Estoy en Londres".
    * El router podría elegir incorrectamente `webSearch`.
    * La protección permite volver a utilizar `getWeather`.
  * Las cadenas exitosas no se heredan.
  * La modificación no afecta al caché.
  * Las repeticiones futuras de la misma consulta reciben la salida original del router.
* Contrato completo: `src/jarvis/reply/reply.spec.md`, sección §6.

---

## 8. Buscador de Herramientas (válvula de escape durante el bucle)

* **Archivo**: `src/jarvis/tools/builtin/tool_search.py` — `toolSearchTool`.
* **Disparador**: cuando el modelo invoca explícitamente `toolSearchTool` durante el bucle.
* Máximo `tool_search_max_calls` = 3 por respuesta.
* **Modelo**: reutiliza el router de herramientas (#7), por lo que no realiza una llamada LLM independiente.
* **Entrada**: consulta autocontenida proporcionada por el modelo.
* **Salida**: nombres de herramientas separados por líneas + descripción de una línea.
* Los resultados se combinan con la lista permitida para el siguiente turno.

---

## 9. Resumidor de Conversaciones

* **Archivo**: `src/jarvis/memory/conversation.py` — `generate_conversation_summary()` (~líneas 350/355).
* **Disparador**: proceso periódico en segundo plano, cuando el diálogo no guardado alcanza `dialogue_memory_timeout`.
* Uno por día y por `source_app`.
* **Modelo / control**: `ollama_chat_model`.
* Respeta `llm_thinking_enabled`.
* Utiliza streaming cuando existe un callback de tokens; de lo contrario, ejecución directa.
* **Entradas**:

  * Fragmentos recientes de conversación.
  * Resumen previo del mismo día para realizar una actualización incremental.
* **Prompt de sistema**: integrado aproximadamente en las líneas 310-320.
* Reglas de higiene definidas en `src/jarvis/memory/summariser.spec.md`:

  * Sin narración de desviaciones.
  * Conservación de atribuciones.
  * Separación de temas.
* La regla 6 sobre desviaciones incluye ejemplos BAD/GOOD en inglés y pares equivalentes en turco y español para que los modelos pequeños no interpreten que la regla depende del idioma.
* **Salida**:

  * `(summary_text, topics_text)`
  * Se almacena en `conversation_summaries`.
  * Se utiliza para búsqueda vectorial.
  * Alimenta el enriquecimiento (#3) y la extracción del grafo (#10).
* No existe limpieza posterior: el prompt es la fuente única de verdad, independiente del idioma y mejora automáticamente cuando se actualiza el modelo de chat.
* **Límite**: `timeout_sec` = 30 s por defecto.

### Reescritura de desviaciones

Operación masiva independiente:

`rewrite_all_diary_summaries()`

Endpoint:

`POST /api/diary/scrub-deflections`

Se utiliza para limpiar filas históricas creadas antes de endurecer el prompt.

* Una llamada a `ollama_chat_model` por fila.
* Utiliza `_REWRITE_DEFLECTION_SYSTEM_PROMPT`.
* Solicita eliminar frases que narren los fallos del propio asistente manteniendo el resto sin cambios.
* El texto del diario se encapsula como datos no confiables utilizando el mismo fence de la herramienta web.
* Conserva `ts_utc`.
* Vuelve a generar embeddings cuando es posible.
* Si el modelo devuelve un texto vacío, se conserva el original.
* Todas las capas utilizan fail-open.

### Optimización de temas

Operación masiva independiente:

`optimise_diary_topics()`

Endpoint:

`POST /api/diary/optimise-topics`

* Recopila todas las etiquetas únicas de `conversation_summaries`.
* Realiza una llamada a `ollama_chat_model` con `_TOPIC_OPTIMISE_SYSTEM_PROMPT`.
* Propone una taxonomía normalizada:

  * fusionar sinónimos;
  * dividir etiquetas compuestas.
* Aplica el mapeo a las filas que necesitan actualización.
* Conserva `ts_utc`.
* Regenera embeddings cuando es posible.
* Se ejecuta manualmente desde la sección Maintenance del panel lateral del diario.

---

## 10. Extracción de Hechos del Grafo de Conocimiento + Clasificación de Rama

* **Archivo**: `src/jarvis/memory/graph_ops.py` — `extract_graph_memories()`.
* **Disparador**: después de cada resumen diario (#9).
* Se ejecuta en segundo plano.
* **Modelo**: `ollama_chat_model`.
* **Entradas**:

  * Texto del resumen.
  * Fecha opcional.
* **Prompt de sistema**:

  * Solicita un array JSON con objetos:
    `{"branch": "USER|DIRECTIVES|WORLD", "fact": "..."}`
  * Utiliza la siguiente heurística:

    * El usuario indica cómo debe comportarse el asistente → `DIRECTIVES`.
    * El usuario proporciona información sobre sí mismo → `USER`.
    * Hechos externos → `WORLD`.
  * Ramas desconocidas → `USER`.
* El bloque `DO-NOT-EXTRACT` evita dos problemas recurrentes:

  * Recomendaciones generadas por el asistente.
  * Datos temporales como el clima o la hora actual.
* Estos últimos se consideran "momentos, no hechos", evitando confundir información temporal con conocimiento persistente.
* **Salida**:

  * Lista de tuplas `(branch_id, fact_text)`.
  * Se almacena en la rama correspondiente mediante descenso con rama fija, evitando contaminación entre ramas.
* **Límites**:

  * `timeout_sec`.
  * Los fallos devuelven una lista vacía.

---

## 11. Selector del Mejor Hijo del Grafo de Conocimiento

* **Archivo**: `src/jarvis/memory/graph_ops.py` — `_llm_pick_best_child()` (~línea 167).
* **Disparador**: durante la inserción de cada hecho, para colocarlo bajo la mejor categoría existente.
* Se ejecuta en segundo plano.
* **Modelo**:

  * Utiliza `picker_model` cuando es proporcionado por `update_graph_from_dialogue`.
  * El daemon resuelve este modelo mediante `resolve_tool_router_model(cfg)`.
  * Utiliza un modelo pequeño cuando está disponible.
  * Si no existe un modelo pequeño, utiliza `ollama_chat_model`.
* **Entradas**:

  * Texto del hecho.
  * Lista numerada de nodos hijos candidatos, con nombre + descripción.
* **Prompt de sistema**: aproximadamente líneas 156-161.
* Debe responder con un número o `NONE`.
* **Salida**:

  * ID del nodo hijo.
  * `None` si no existe una categoría adecuada.
  * El hecho se inserta igualmente, aunque no quede bajo el padre óptimo.

---

## 11b. Fusión de Nodos del Grafo

### Consolidación mediante reescritura al guardar

* **Archivo**: `src/jarvis/memory/graph_ops.py` — `merge_node_data()`.
* Prompt de sistema: `_MERGE_SYSTEM_PROMPT`.
* **Disparador**:

  * Una vez por `(node, flush)` durante `update_graph_from_dialogue`.
  * Primero se aplica deduplicación mediante coincidencia exacta.
  * Después se agrupan los hechos restantes por su `node_id`.
  * Si un flush de 5 hechos afecta al nodo User, se realiza una única reescritura en lugar de cinco.
* Las escrituras de arranque en frío, cuando el nodo está vacío, pasan directamente a un append normal.
* También se invoca con `new_facts=[]` mediante `consolidate_all_populated_nodes`, utilizado por la operación de mantenimiento del visor de memoria mediante el botón 🧹.

### Modelo

Utiliza la misma cadena `picker_model` que #11:

`small router model → ollama_chat_model`

Temperatura `0`, ya que se trata de una tarea de clasificación basada en reglas.

### Entradas

* Datos existentes del nodo.
* Lote de nuevos hechos dirigidos a ese nodo en ese flush.

### Reglas del prompt

Define un conjunto ordenado de reglas:

1. Las contradicciones o reversiones eliminan la versión anterior.
2. Las frases casi duplicadas se combinan.
3. Las actividades diarias repetidas se consolidan como patrones.
4. Los atributos independientes pueden coexistir.
5. Las contradicciones visibles **no** se eliminan silenciosamente.
6. Los hechos de conocimiento común se eliminan.

Exige un objeto JSON simple:

`{"facts": [...]}`

El parser intenta primero `json.loads` directo y posteriormente un regex acotado, sin utilizar `\{.*\}` de forma codiciosa.

### Salida

`MergeResult(success: bool, incorporated_indices: list[int])`

La lista de hechos revisada se escribe como el `data` completo del nodo.

`incorporated_indices` indica qué entradas sobrevivieron como nuevas líneas mediante comparación con `NFKC + casefold`.

Esto evita reportar como "recién almacenados" los hechos que fueron absorbidos por consolidación.

La operación engloba:

* supersesión por flush;
* deduplicación de frases similares;
* consolidación continua.

Como el prompt actual reescribe todo el nodo, las nuevas convenciones se propagan a los datos antiguos sin necesidad de una migración independiente.

### Límites y protección contra alucinaciones

* Timeout: 20 s.
* Si la reescritura produce más de:

`len(existing) + len(new) + 2`

líneas, se rechaza como salida descontrolada.

Ante:

* error;
* fallo de parseo;
* reescritura demasiado grande;
* reescritura vacía;

se utiliza `append_to_node` para cada hecho nuevo.

Esto garantiza que una contradicción sea recuperable, mientras que una eliminación silenciosa o una expansión alucinada no lo sea.

---

## 12. Planificador de Tareas

### Descomposición previa al vuelo que controla todo el turno

* **Archivo**: `src/jarvis/reply/planner.py` — `plan_query()`.
* **Disparador**:

  * Una vez por respuesta.
  * Después del router de herramientas.
  * Antes de la búsqueda de memoria.
* Se omite cuando:

  * `cfg.planner_enabled = False`;
  * la consulta tiene menos de `MIN_QUERY_CHARS` (4);
  * no existe modelo o URL base.
* **Modelo / control**:

  * `planner_model` (override).
  * `ollama_chat_model`.
* El planificador sigue al modelo de chat, de modo que actualizar el modelo mediante el asistente de configuración también actualiza automáticamente la calidad del planificador.

### Entradas

* Consulta del usuario.
* Contexto del diálogo.
* Catálogo de herramientas reducido por el router:

  * nombres;
  * descripciones de una línea.

No recibe la lista completa de más de 30 herramientas.

Cuando la protección de herencia de #7 se activa, la herramienta fallida del turno anterior se agrega al catálogo antes de que el planificador lo vea.

No recibe contexto de memoria: el planificador decide si la memoria es necesaria.

### Prompt

`_PROMPT_TEMPLATE` en `planner.py`.

Enseña:

* directiva `searchMemory topic='...'`;
* pasos breves de herramientas;
* placeholders de entidades mediante `<...>`;
* paso final de síntesis;
* respuesta en el mismo idioma del usuario;
* ausencia de numeración.

### Salida

Lista de pasos con un máximo de:

`MAX_STEPS = 5`

Controla:

* enriquecimiento de memoria (#3/#4);
* selección de herramientas (#7).

Las herramientas seleccionadas por el planificador se agregan a las del router, no las sustituyen.

Un plan de un único paso:

`["Reply to the user."]`

es una señal positiva de que no se necesita memoria ni herramientas.

Un plan vacío es fail-open: el motor vuelve a ejecutar #3 de manera incondicional.

El resultado también se utiliza para construir el bloque de sistema:

`ACTION PLAN:`

y controlar el bucle de ejecución directa (#13) para modelos pequeños.

### Límite

`planner_timeout_sec` = 6 s.

En caso de fallo → `[]`.

---

## 13. Resolutor de Pasos del Plan

### Por turno de ejecución directa para modelos pequeños

* **Archivo**: `src/jarvis/reply/planner.py` — `resolve_next_tool_call()`.
* **Disparador**:

  * Al principio de cada iteración del bucle agéntico.
  * Cuando `use_text_tools` es `True`.
  * Cuando el plan de #12 todavía tiene pasos de herramientas sin ejecutar.
* En ese turno se ejecuta en lugar del modelo de chat.
* **Fast path**:

  * Si el paso ya contiene completamente:

    * nombre de herramienta;
    * argumentos `key='value'`;
    * ningún `<placeholder>`;
  * se omite completamente el LLM.
* El LLM solo se utiliza cuando hace falta sustitución de entidades o reasignación de claves.

### Modelo

La misma cadena utilizada por #12:

`planner_model → ollama_chat_model`

### Entradas

* Texto del siguiente paso.
* Llamadas a herramientas anteriores:

  * nombre;
  * argumentos;
  * fragmento del resultado.
* Esquema de herramientas disponible para ese turno.

### Prompt

`_STEP_RESOLVER_SYSTEM` en `planner.py:300`.

Enseña:

* salida como un único objeto JSON;
* sustitución de placeholders utilizando resultados anteriores;
* `null` para pasos de síntesis.

### Salida

Tupla:

`(tool_name, arguments)`

o:

`None`

Los nombres desconocidos se rechazan mediante la protección de lista permitida.

### Límites

`planner_timeout_sec`.

En caso de fallo → `None`.

El motor vuelve a ejecutar el turno del modelo de chat.

---

## 14. Llamadas LLM Específicas de Herramientas

### Clima

`src/jarvis/tools/builtin/weather.py`

* Aproximadamente línea 60.
* Modelo: `ollama_chat_model`.
* Extrae de la consulta:

  * ubicación;
  * hora;
  * unidad.

### Registro nutricional

`src/jarvis/tools/builtin/nutrition/log_meal.py`

* Líneas 48 y 136.
* Modelo: `ollama_chat_model`.
* Extrae nutrientes.
* Confirma el registro.

---

# Resumen de Frecuencia / Tamaño

| #   | Contexto                                 |            Por respuesta | ¿Opcional?                                | Nivel del modelo                         |
| --- | ---------------------------------------- | -----------------------: | ----------------------------------------- | ---------------------------------------- |
| 1   | Bucle principal de chat                  |                      1-8 | No                                        | GRANDE                                   |
| 2   | Evaluador de intención                   |             1 (solo voz) | fallback disponible                       | PEQUEÑO                                  |
| 3   | Extracción de enriquecimiento de memoria |                      0-1 | controlado por planner                    | PEQUEÑO (cadena del router)              |
| 4   | Digest de memoria                        |                      0-N | automático según tamaño                   | PEQUEÑO (usa modelo de chat)             |
| 5   | Digest de resultados de herramientas     |                      0-N | automático según tamaño                   | PEQUEÑO (usa modelo de chat)             |
| 6   | Digest al alcanzar máximo de turnos      |                      0-1 | No                                        | PEQUEÑO                                  |
| 7   | Router de herramientas                   |                        1 | siempre; los pasos del planner se agregan | PEQUEÑO                                  |
| 8   | Buscador de herramientas                 |                      0-3 | iniciado por el modelo                    | PEQUEÑO (reutiliza #7)                   |
| 9   | Resumidor                                |                ~1/sesión | No (segundo plano)                        | GRANDE                                   |
| 10  | Extracción del grafo                     |                ~1/sesión | No (segundo plano)                        | GRANDE                                   |
| 11  | Mejor hijo del grafo                     |                      0-N | No (segundo plano)                        | PEQUEÑO (cadena del router)              |
| 11b | Fusión de nodos del grafo                | 0-N (por nodo, agrupado) | No (segundo plano)                        | PEQUEÑO (cadena del router)              |
| 12  | Planificador (`plan_query`)              |                        1 | sí (`planner_enabled`)                    | GRANDE/PEQUEÑO (sigue al modelo de chat) |
| 13  | Resolutor de pasos                       |         0-N (solo SMALL) | automático según tamaño + plan            | PEQUEÑO (cadena del router)              |
| 14  | Específicas de herramientas              |          por herramienta | n/a                                       | GRANDE                                   |

---

# Cambios Automáticos Según Tamaño

Se determinan mediante:

`detect_model_size(model_name) → SMALL (≤7B) | LARGE (8B+)`

| Característica                         | SMALL    | LARGE                |
| -------------------------------------- | -------- | -------------------- |
| Digest de memoria                      | ACTIVADO | DESACTIVADO          |
| Digest de resultados de herramientas   | ACTIVADO | DESACTIVADO          |
| Llamadas a herramientas mediante texto | ACTIVADO | DESACTIVADO (nativo) |
| Ejecución directa mediante planner     | ACTIVADO | DESACTIVADO          |

---

# Claves de Configuración

### Modelos

* `ollama_chat_model`
* `intent_judge_model`
* `tool_router_model`

### Flags

* `memory_digest_enabled`
* `tool_result_digest_enabled`
* `llm_thinking_enabled`
* `intent_judge_thinking_enabled`
* `tool_selection_strategy`

### Timeouts

* `llm_chat_timeout_sec` — 45 s
* `llm_digest_timeout_sec` — 8 s, compartido entre #4/#5/#6
* `llm_tools_timeout_sec`
* `intent_judge_timeout_sec` — 15 s

### Límites

* `agentic_max_turns` — 8
* `tool_search_max_calls` — 3
* `_LLM_MAX_SELECTED` — 5
* `_DIGEST_MAX_CHARS` — 400
* `_TOOL_DIGEST_MAX_CHARS` — 600

---

# Flujo

```text
entrada del usuario
  └─▶ [2] Evaluador de intención
        (solo voz, SMALL)
              │
              ▼
        [7] Router de herramientas
        (reduce el catálogo para el planner)
              │
              ▼
        [12] Planner
        (controla memoria; asesor para la lista permitida del router)
              │
              ├─ plan solicita searchMemory
              │      └─▶ [3] Extractor de enriquecimiento
              │              └─▶ [4] Digest de memoria (opcional)
              │
              ├─ plan vacío (fail-open)
              │      └─▶ [3] Extractor de enriquecimiento
              │              └─▶ [4] Digest de memoria
              │
              └─ plan solo de respuesta
                     └─▶ omitir #3 y #4
              
              └─▶ BUCLE AGÉNTICO (≤ agentic_max_turns)
                       │
                       ├─ [13] Resolutor de pasos
                       │      (SMALL, ejecución directa)
                       │
                       ├─ [1] Turno principal del chat
                       │
                       ├─ ejecución de herramienta
                       │      │
                       │      ├─ [5] Digest de resultado
                       │      │      (opcional)
                       │      │
                       │      └─ [8] Buscador de herramientas
                       │             (iniciado por el modelo)
                       │
                       ├─ contenido
                       │      └─▶ entregar inmediatamente
                       │
                       └─ si alcanza el máximo
                              └─▶ [6] Digest de máximo de turnos

              └─▶ TTS / salida
              
              └─▶ segundo plano:
                    [9] Resumidor
                         │
                         ▼
                    [10] Extracción del grafo
                         │
                         ▼
                    [11] Mejor hijo
```

---

# Ideas de Optimización

Lista inicial:

1. Agrupar varios bloques del digest de memoria (#4) en una única llamada con marcadores explícitos.
2. Paralelizar múltiples digests de resultados de herramientas (#5) cuando lleguen varios resultados simultáneamente.
3. Precargar el modelo del evaluador de intención antes de que termine el TTS.
4. Almacenar en caché la salida del router de herramientas (#7) mediante hash de la consulta.
5. Dar a cada digest su propio presupuesto de timeout en lugar de compartir `llm_digest_timeout_sec`. Actualmente un digest de memoria lento puede dejar sin tiempo al digest de máximo de turnos.
6. Considerar despliegues con un solo modelo: router + planner deberían preferir `intent_judge_model`; cargar un segundo modelo aumenta la latencia de arranque en hardware pequeño.
7. Limitar `llm_thinking_enabled` al router/planner en lugar de activarlo para todos los contextos.
8. Reducir `intent_judge_timeout_sec` de 15 s o ejecutarlo en paralelo con la detección de wake word basada en texto para evitar bloquear el ciclo de audio.

---

# Medición

`tests/performance/test_pipeline_timings.py` mide cada contexto de este grafo utilizando una instancia real de Ollama.

Ejecutar:

```bash
pytest tests/performance/ -v -m performance -s
```

El sistema registra las latencias p50/p95 de cada contexto mediante un recorder basado en monkey-patching que identifica el contexto a partir del `__qualname__` del llamador.

El mapeo se encuentra en:

```text
_CALLER_TO_CONTEXT
```

en:

```text
tests/performance/timing_recorder.py
```

El informe JSON se guarda en:

```text
tests/performance/reports/
```

También se ejecuta un microbenchmark con un prompt pequeño y fijo para obtener un límite inferior por llamada.

Si ese límite cambia, todos los contextos pueden cambiar proporcionalmente, por lo que permite detectar cambios relacionados con hardware o modelo.

### Línea base

Con un `gemma4:e2b` local, a fecha del 22 de abril de 2026:

* 3 consultas × 3 ejecuciones.
* Turno principal de chat:

  * p50 ≈ 4.5 s.
* Extracción de enriquecimiento:

  * p50 ≈ 0.9 s.
* Piso del micro-prompt:

  * ≈ 0.15 s.
* Muestras:

  * chat principal: 25 llamadas.
  * enriquecimiento: 9 llamadas.

Estos valores son únicamente referencias aproximadas.

Las aserciones de los tests comprueban principalmente la forma relativa, por ejemplo:

```text
router ≤ 1.5 × turno principal de chat
```

y no valores absolutos.

---

# Mantener este documento sincronizado

Este grafo es la referencia principal para la optimización de la latencia de los LLM.

Debe considerarse **autoritativo**.

Cada vez que un cambio de código afecte una llamada a un LLM, actualiza este documento en el mismo PR.

Esto incluye:

* agregar un nuevo contexto;
* eliminar un contexto;
* modificar el modelo;
* cambiar un timeout;
* cambiar un límite;
* modificar una condición de ejecución;
* cambiar el origen de un prompt;
* agregar o modificar una conexión en el flujo de datos.

Si la actualización requiere más que un cambio de una sola línea, también debe reflejarse en el archivo `*.spec.md` correspondiente.
