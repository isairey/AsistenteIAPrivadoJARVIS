# Directrices de Desarrollo de Jarvis

La **privacidad de los datos es lo primero, siempre**.

Toda salida de línea de comandos visible para el usuario debe utilizar emojis. En especial, debe incluirse un emoji inicial para comenzar las líneas que indiquen de qué trata esa línea. La salida debe utilizar espacios de indentación para establecer una jerarquía visual y procurar que sea lo más fácil posible de revisar rápidamente.

**Excepción:** los scripts `.bat` de Windows no pueden utilizar emojis, ya que `cmd.exe` no representa Unicode correctamente.

---

Cualquier punto importante de nuestros flujos lógicos debe tener logs de depuración mediante el método `debug_log` de:

```text
src/jarvis/debug.py
```

Evita registrar información excesiva para mantener los logs fáciles de leer y útiles para actuar sobre ellos.

---

## Cambios de código y archivos de especificación

Cualquier cambio de código debe cumplir perfectamente con nuestros archivos de especificación. Si no es posible hacerlo, debes solicitar al usuario que confirme los cambios. Esa confirmación también debe propagarse a los propios archivos de especificación.

Los archivos de especificación utilizan el formato:

```text
*.spec.md
```

y se encuentran junto al código que implementan.

**Siempre busca los archivos de especificación relacionados antes de comenzar cualquier trabajo.**

Cuando se corrija cómo debería funcionar algo, comprueba si existe una especificación para ese comportamiento y determina si necesita actualizarse.

---

# Registro de Archivos de Especificación

| Archivo de especificación                             | Cubre                                                                                                                                                     | Principios clave                                                                                                                                                                                             |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `src/desktop_app/desktop_app.spec.md`                 | Aplicación de bandeja del sistema, flujo de inicio, integración con daemon, ventanas, tema y actualizaciones                                              | Desktop está separado del núcleo; jarvis no tiene conocimiento de `desktop_app`                                                                                                                              |
| `src/desktop_app/settings_window.spec.md`             | UI de configuración generada automáticamente a partir de metadatos de configuración                                                                       | Basada en metadatos; solo se escriben valores diferentes a los predeterminados; conserva claves desconocidas                                                                                                 |
| `src/desktop_app/setup_wizard.spec.md`                | Asistente de primera ejecución, Ollama, modelos, Whisper y ubicación                                                                                      | Fricción mínima; solo aparece cuando requiere una acción del usuario; no configura todo                                                                                                                      |
| `src/jarvis/dictation/dictation.spec.md`              | Motor de dictado mediante pulsación prolongada, hotkey y pegado desde el portapapeles                                                                     | Independiente del pipeline del asistente; comparte el modelo Whisper; utiliza una bandera de pausa en el listener                                                                                            |
| `src/jarvis/listening/listening.spec.md`              | Listener de voz, detección de palabra de activación y pipeline de audio                                                                                   | -                                                                                                                                                                                                            |
| `src/jarvis/reply/reply.spec.md`                      | Generación de respuestas LLM, uso de herramientas y perfiles                                                                                              | Las herramientas devuelven datos sin procesar; los perfiles gestionan el formato                                                                                                                             |
| `src/jarvis/reply/evaluator.spec.md`                  | **Obsoleto**. El evaluador ya no se ejecuta en el motor de respuestas; se conserva como referencia                                                        | Sustituido por el planner; consultar `planner.spec.md`                                                                                                                                                       |
| `src/jarvis/reply/planner.spec.md`                    | Planner de tareas: descomposición de consultas antes del bucle + resolutor de ejecución directa para modelos pequeños                                     | Fail-open; utiliza la cadena de modelos pequeños en caliente; asesor para modelos grandes y ejecución directa para modelos pequeños                                                                          |
| `src/jarvis/tools/builtin/tool_search.spec.md`        | `toolSearchTool` como mecanismo de escape para el enrutamiento de herramientas durante el bucle                                                           | Reejecuta el mismo router; nunca elimina `stop/self`; limitado por respuesta                                                                                                                                 |
| `src/jarvis/tools/external/mcp_runtime.spec.md`       | Runtime MCP persistente: sesión stdio de larga duración por servidor, despacho basado en cola y reintento ante pérdida temporal de sesión                 | Un worker por servidor basado en configuración; las llamadas al mismo servidor se serializan; `MCPServerSessionError` para errores a nivel de sesión; `idle_timeout_sec` opcional para servidores sin estado |
| `src/jarvis/reply/prompts/prompts.spec.md`            | Plantillas de prompts de sistema/usuario                                                                                                                  | -                                                                                                                                                                                                            |
| `src/jarvis/tools/builtin/web_search.spec.md`         | Herramienta `webSearch`: cascade fetch, protección SSRF, fence contra prompt injection y envelope de solo enlaces                                         | El contenido web no confiable se encapsula como datos, no como instrucciones; preferencia por relevancia sobre velocidad; un fallo honesto es preferible a una confabulación                                 |
| `src/jarvis/tools/builtin/nutrition/log_meal.spec.md` | Herramienta `logMeal`: esquema de propiedad única para fast-path del planner, extracción nutricional interna, fence de datos no confiables y seguimientos | El esquema público es una única cadena opcional `meal`; los campos nutricionales son internos; el texto del usuario se encapsula como datos                                                                  |
| `src/jarvis/utils/location.spec.md`                   | Detección de ubicación mediante GeoIP                                                                                                                     | Privacidad primero; únicamente base de datos GeoLite2 local                                                                                                                                                  |
| `src/jarvis/memory/graph.spec.md`                     | Memoria mediante grafo de nodos (v2), árbol autoorganizado y explorador UI                                                                                | Estructura dinámica; consciente del acceso; auto-división/fusión (futuro)                                                                                                                                    |
| `src/jarvis/memory/summariser.spec.md`                | Contrato del prompt del resumidor del diario, reglas de higiene, atribución, separación de temas, limpieza posterior y botón de limpieza masiva           | Defensa en dos capas: prompt + limpieza determinista; los resúmenes corruptos contaminan todos los consumidores posteriores                                                                                  |
| `src/jarvis/memory/recall_gate.spec.md`               | Heurística determinista para omitir enriquecimiento cuando la ventana caliente ya cubre una continuación                                                  | Fail-open; independiente del idioma mediante `\w{3,}` + `re.UNICODE`; la intención del planner siempre tiene prioridad                                                                                       |

---

# Grafo de Contextos LLM

El grafo de contextos LLM ubicado en:

```text
docs/llm_contexts.md
```

mapea cada llamada al LLM de la aplicación, incluyendo:

* modelo;
* condiciones de ejecución;
* entradas;
* salidas;
* límites;
* flujo.

Debe mantenerse actualizado en todo momento.

Cualquier cambio que:

* agregue un contexto LLM;
* elimine un contexto;
* modifique un contexto;
* cambie la resolución del modelo;
* cambie un timeout;
* cambie un límite;
* cambie el origen de un prompt;
* cambie una bandera de control;
* agregue o modifique una conexión del flujo de datos;

debe actualizar también:

```text
docs/llm_contexts.md
```

en el mismo PR.

---

# Soporte Multilingüe

Evita patrones de lenguaje codificados manualmente.

Este asistente debe ser capaz de soportar una cantidad arbitraria de idiomas.

No diseñes lógica que dependa de palabras o patrones específicos de un idioma cuando pueda utilizarse una solución independiente del idioma.

---

# Herramientas

Las herramientas definen **cuándo y cómo deben utilizarse** y devuelven datos sin procesar, sin procesamiento mediante LLM.

El prompt de sistema unificado en:

```text
src/jarvis/system_prompt.py
```

gestiona:

* formato de las respuestas;
* personalidad;
* presentación del resultado;

mediante el bucle LLM del daemon.

---

# Flujo de Git

La rama predeterminada es:

```text
develop
```

Todos los PR y las ramas de funcionalidades deben dirigirse a:

```text
develop
```

y no a:

```text
main
```

---

## Conventional Commits

Utiliza **Conventional Commits** para todos los mensajes de commit y títulos de PR.

Ejemplos:

```text
fix:
feat:
refactor:
docs:
test:
chore:
```

Por ejemplo:

```text
feat: add weather tool routing
```

---

## Actualización de PR

Cuando se agreguen commits a un PR, actualiza siempre:

* el título del PR;
* el cuerpo del PR;

para que representen el conjunto completo de cambios.

---

## Commits fusionados mediante squash

Los commits fusionados mediante squash en `develop` deben contener **únicamente el número del PR en el título**.

Ejemplo:

```text
(#171)
```

Nunca deben utilizar el número del issue original en el título.

Las referencias a issues deben aparecer en el cuerpo del commit:

```text
Closes #NNN
```

Esto permite que el issue se cierre automáticamente cuando el commit llegue a `main` durante una release.

---

# Gestión de Issues

Utiliza el skill:

```text
/triage
```

para clasificar y gestionar issues y discusiones abiertas.

Este skill controla:

* el flujo completo;
* patrones de diagnóstico;
* convenciones de etiquetas;
* tono de las respuestas.

---

# Releases

"Release" significa avanzar `main` mediante fast-forward hasta el commit actual de `develop` y hacer push.

Primero sincroniza el `develop` local con:

```text
origin/develop
```

Después utiliza exactamente:

```bash
git checkout main
git merge --ff-only develop
git push origin main
```

No utilices:

* merge commits;
* force push.

Este procedimiento activa el workflow de release y el cierre automático de los issues referenciados mediante:

```text
Closes #NNN
```

en los commits de `develop`.

---

# Entorno de Desarrollo

El proyecto utiliza un entorno micromamba ubicado en:

```text
.mamba_env/
```

Siempre actívalo antes de ejecutar:

* builds;
* tests;
* la aplicación.

Comando:

```bash
eval "$(micromamba.exe shell hook --shell bash)" && micromamba activate "C:/Users/baris/projects/jarvis/.mamba_env"
```

---

# Mantenimiento del README

Mantén `README.md` actualizado cuando realices cambios que afecten a funcionalidades visibles para el usuario.

Actualiza el README cuando:

* agregues o elimines herramientas integradas;

  * actualiza `Features → Built-in Tools`.
* cambies opciones de configuración;

  * actualiza la sección `Configuration`.
* agregues nuevos ejemplos de integración MCP.
* cambies los requisitos del sistema.
* cambies los pasos de instalación.
* corrijas o introduzcas limitaciones conocidas.

---

## Prioridades del README

Las prioridades son, en este orden:

### 1. Privacidad primero

El carácter local/offline es una de las principales características del proyecto.

### 2. Instalación rápida

El usuario debería poder poner el sistema en funcionamiento en cuestión de minutos.

### 3. Lista de funcionalidades

Mostrar las capacidades principales de un vistazo.

### 4. Limitaciones conocidas

Ser transparente sobre aquello que todavía no funciona correctamente.

### 5. Configuración

Documentar únicamente las opciones que realmente necesitan los usuarios.

### 6. Integraciones MCP

Incluir ejemplos para herramientas populares.

### 7. Solución de problemas

Documentar problemas frecuentes y sus soluciones.

---

Mantén las secciones concisas.

Utiliza `<details>` para contenido extenso.

Evita documentar detalles internos de implementación.

El README está dirigido a **usuarios finales**, no a desarrolladores.

---

# Memoria

Cuando el usuario diga:

```text
remember
```

agrega esa información a:

```text
CLAUDE.md
```

en la sección correspondiente:

* información específica del proyecto → antes de `---`;
* información portable → después de `---`.

---

# Desarrollo y Pruebas

Ejecuta los cambios y pruébalos manualmente.

Itera hasta que todo funcione correctamente.

---

## TDD obligatorio

Utiliza siempre **TDD**:

1. Escribe primero los tests que fallen.
2. Implementa la corrección.
3. Ejecuta los tests.
4. Corrige los problemas encontrados.
5. Repite hasta que todo funcione.

Los tests deben verificar **comportamientos**, no detalles de implementación.

Comprueba:

> qué hace el sistema

y no:

> cómo lo hace internamente.

---

# Cobertura de Tests

Todos los cambios deben estar cubiertos por las formas de pruebas automatizadas apropiadas.

Dependiendo del cambio, esto puede incluir:

* tests unitarios;
* tests de integración;
* regresión visual;
* evals;
* otros tipos de pruebas relevantes.

---

## Tests de mecanismos

Los tests deben verificar mecanismos y comportamientos, no valores actuales específicos.

Utiliza referencias:

* derivadas de configuración;
* calculadas dinámicamente;
* dependientes del entorno;

en lugar de valores codificados manualmente que puedan cambiar durante futuras migraciones.

---

# Evals

Ejecuta los evals después de finalizar cualquier cambio que pueda afectar a la precisión del agente.

---

## Cambios en Prompts

Cualquier modificación de prompts LLM, incluyendo:

* prompts de sistema;
* prompts de herramientas;
* incentivos;
* restricciones;
* instrucciones;

debe verificarse mediante un caso de eval relevante.

Si no existe un eval para el comportamiento que se está modificando:

1. crea primero el eval;
2. comprueba que falla o produce peores resultados antes del cambio;
3. aplica el cambio del prompt;
4. comprueba que el eval pasa o mejora posteriormente.

El eval debe demostrar realmente la mejora.

---

# Commits

Haz commit de los cambios cuando termines una corrección o funcionalidad antes de pasar a la siguiente tarea.

---

## `git commit --amend`

Antes de ejecutar:

```bash
git commit --amend
```

comprueba siempre:

```bash
git log --oneline -3
```

Esto garantiza que estás modificando el commit correcto.

---

# Inglés Británico

Utiliza siempre **inglés británico**.

Por ejemplo:

```text
colour
behaviour
initialise
```

y no:

```text
color
behavior
initialize
```

---

# Uso de Em Dash

No utilices em dash (`—`) en:

* respuestas de GitHub issues;
* respuestas de PR;
* respuestas de discusiones;
* textos dirigidos al usuario.

En su lugar utiliza:

* una coma;
* un punto;
* dos puntos;
* paréntesis;

según corresponda a la oración.

Esta regla también se aplica a las respuestas que publiques en nombre del usuario y al texto generado para él.

---

# Ingeniería de Prompts: Reflejo de Plantillas de Negación

Cuando un modelo pequeño continúa generando una negación canónica como:

> "I only have access to the information you have shared in our current conversation"

o:

> "I don't have any personal information about you"

no intentes discutir esa negación dentro del prompt de sistema.

Esto rara vez funciona frente a priors fuertes del modelo.

En su lugar, formula el contexto inyectado de manera que ocupe literalmente el espacio semántico al que se refiere la negación.

Por ejemplo, si el modelo afirma que no tiene:

> "information the user has shared in prior conversations"

entonces etiqueta explícitamente el bloque con exactamente esa idea y coloca allí la información correspondiente.

El objetivo es que la negación deje de activarse porque aquello que el modelo afirma no tener ahora aparece visiblemente en el prompt.

**Discutir contra los priors del modelo es costoso; proporcionar directamente los datos en el espacio semántico esperado es más barato.**
