<div align="center">

<img width="220" src="https://cdn-icons-png.flaticon.com/512/4712/4712109.png" />

# 🤖 JARVIS AI Assistant

### Asistente de Inteligencia Artificial Privado, Local y Conversacional 🚀

<p align="center">
  <b>JARVIS AI Assistant</b> es un asistente inteligente completamente privado que funciona localmente en tu computadora, capaz de mantener conversaciones naturales, recordar información, controlar herramientas externas, realizar búsquedas web, automatizar tareas y ofrecer una experiencia similar a un asistente personal avanzado.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/AI-Assistant-blueviolet?style=for-the-badge">
  <img src="https://img.shields.io/badge/Ollama-LocalAI-black?style=for-the-badge">
  <img src="https://img.shields.io/badge/Whisper-SpeechRecognition-ff9800?style=for-the-badge">
  <img src="https://img.shields.io/badge/MCP-Integrations-success?style=for-the-badge">
</p>

<p align="center">
  <a href="#-acerca-del-proyecto">Acerca</a> •
  <a href="#-características">Características</a> •
  <a href="#-arquitectura">Arquitectura</a> •
  <a href="#-instalación">Instalación</a> •
  <a href="#-configuración">Configuración</a>
</p>

</div>

---

# 🌌 Acerca del proyecto

**JARVIS AI Assistant** es un asistente virtual impulsado por modelos de lenguaje ejecutados localmente, diseñado para ofrecer privacidad total, memoria persistente y automatización avanzada.

A diferencia de asistentes tradicionales, Jarvis mantiene contexto conversacional, aprende preferencias del usuario y puede interactuar con herramientas externas mediante MCP (Model Context Protocol).

La plataforma permite:

* 🤖 Conversaciones inteligentes
* 🎙️ Control por voz
* 🧠 Memoria persistente
* 🌐 Búsquedas web
* 📂 Acceso a archivos
* 📍 Conocimiento de ubicación y hora
* 🛠️ Automatización de tareas
* 🔒 Procesamiento completamente local

---

# ✨ Características

## 🔒 Privacidad Total

* Procesamiento local
* Sin servidores externos
* Sin suscripciones
* Sin recopilación de datos
* Redacción automática de información sensible

---

## 🧠 Memoria Inteligente

* Recuerda conversaciones
* Almacena preferencias
* Grafo de conocimiento
* Historial permanente
* Recuperación contextual

---

## 🎙️ Interacción por Voz

* Activación mediante palabra clave
* Reconocimiento de voz con Whisper
* Conversaciones naturales
* Dictado offline
* Respuestas habladas

---

## ⚡ Automatización

* Control de navegador
* Integración con aplicaciones
* Automatización de tareas
* Acciones contextuales
* Herramientas personalizadas

---

# 🚀 Funcionalidades principales

## 🗣️ Conversación Natural

Jarvis entiende conversaciones completas y puede participar como si fuera una tercera persona dentro del diálogo.

### Ejemplos

* Consultas generales
* Conversaciones técnicas
* Ayuda en programación
* Asistencia diaria
* Resolución de problemas

---

## 🌐 Búsqueda Inteligente

* Consultas web
* Noticias
* Información actualizada
* Resultados contextuales
* Respuestas enriquecidas

---

## 📂 Gestión de Archivos

* Lectura de documentos
* Acceso a carpetas
* Búsqueda local
* Organización de información

---

## 🍎 Seguimiento Personal

* Registro nutricional
* Objetivos de salud
* Seguimiento de hábitos
* Recomendaciones personalizadas

---

# 🧩 Arquitectura del sistema

## 🤖 Motor de IA

Encargado de comprender, procesar y responder solicitudes.

### Componentes

* Modelos LLM
* Clasificación de intención
* Memoria contextual
* Generación de respuestas

---

## 🎤 Reconocimiento de voz

Sistema basado en Whisper.

### Funcionalidades

* Conversión voz a texto
* Detección de comandos
* Filtrado de ruido
* Procesamiento local

---

## 🔊 Síntesis de voz

Generación de respuestas habladas.

### Funcionalidades

* Text-to-Speech
* Voces naturales
* Respuestas en tiempo real
* Soporte multilenguaje

---

## 🔌 MCP Integration

Sistema de conexión con herramientas externas.

### Integraciones

* GitHub
* Google Workspace
* Home Assistant
* Slack
* Discord
* Bases de datos
* APIs personalizadas

---

# 🛠️ Tecnologías utilizadas

## 🤖 Inteligencia Artificial

<p>
  <img src="https://skillicons.dev/icons?i=python" />
</p>

* Ollama
* LLMs Locales
* Whisper
* Embeddings
* Knowledge Graph

---

## ⚙️ Backend

<p>
  <img src="https://skillicons.dev/icons?i=python" />
</p>

* Python
* MCP Protocol
* REST APIs
* Automatización

---

## 💾 Almacenamiento

* Memoria persistente
* Knowledge Graph
* Archivos locales
* Configuración JSON

---

## 🧰 Herramientas

<p>
  <img src="https://skillicons.dev/icons?i=git,github,vscode" />
</p>

* Git
* GitHub
* VS Code
* Ollama

---

# 📂 Estructura del proyecto

```bash
AsistenteIAPrivadoJARVIS/
│
├── src/
│   ├── agent/
│   ├── memory/
│   ├── speech/
│   ├── tools/
│   ├── planner/
│   ├── integrations/
│   └── ui/
│
├── config/
├── models/
├── logs/
├── docs/
├── scripts/
├── requirements.txt
├── README.md
└── LICENSE
```

---

# 💻 Requisitos del sistema

## Hardware recomendado

| Configuración | VRAM  | Modelo      |
| ------------- | ----- | ----------- |
| Básico        | 8GB   | Gemma       |
| Intermedio    | 16GB  | Gemma 4B    |
| Avanzado      | 24GB+ | GPT-OSS 20B |

---

## Software requerido

* Ollama
* Python 3.10+
* Git
* Micrófono
* Conexión opcional para búsquedas web

---

# ⚡ Instalación

## 1️⃣ Clonar repositorio

```bash
git clone https://github.com/isairey/AsistenteIAPrivadoJARVIS.git
```

---

## 2️⃣ Entrar al proyecto

```bash
cd AsistenteIAPrivadoJARVIS
```

---

## 3️⃣ Instalar dependencias

```bash
pip install -r requirements.txt
```

---

## 4️⃣ Instalar Ollama

```bash
https://ollama.com/download
```

---

## 5️⃣ Ejecutar Jarvis

### Windows

```bash
scripts/run_windows.ps1
```

### Linux

```bash
bash scripts/run_linux.sh
```

### macOS

```bash
bash scripts/run_macos.sh
```

---

# ⚙️ Configuración

## Archivo principal

```json
{
  "assistant_name": "Jarvis",
  "voice_enabled": true,
  "memory_enabled": true,
  "web_search_enabled": true,
  "planner_enabled": true
}
```

---

# 🎙️ Dictado Inteligente

Jarvis incluye un sistema de dictado completamente offline.

### Características

* Conversión voz a texto
* Funciona en cualquier aplicación
* Sin conexión
* Sin costo
* Sin límites de uso

---

# 🌍 Integraciones MCP

## Servicios compatibles

* 🏠 Home Assistant
* 📧 Gmail
* 📅 Google Calendar
* ☁️ Google Drive
* 💻 GitHub
* 💬 Slack
* 🎮 Discord
* 🗄️ MySQL
* 🐘 PostgreSQL
* 🍃 MongoDB

---

# 📊 Funcionalidades destacadas

## 🧠 Memoria Persistente

* Recuerda conversaciones
* Aprende preferencias
* Recupera contexto automáticamente

---

## 🔍 Búsqueda Inteligente

* DuckDuckGo
* Brave Search
* Wikipedia
* Información contextual

---

## 🛡️ Seguridad

* Datos locales
* Redacción automática
* Sin rastreo
* Sin recopilación de información

---

# 🚧 Roadmap

## 🔮 Próximas mejoras

* 📱 Aplicación móvil
* 💬 Interfaz de chat
* 🎨 Avatar 3D
* 🏠 Automatización avanzada
* 🤖 Agentes especializados
* 🌎 Traducción en tiempo real
* 🧠 Mejoras en memoria semántica

---

# 🤝 Contribuciones

Las contribuciones son bienvenidas ❤️

## Cómo contribuir

1. Fork del proyecto

```bash
git checkout -b feature/nueva-funcionalidad
```

2. Commit

```bash
git commit -m "✨ Nueva funcionalidad"
```

3. Push

```bash
git push origin feature/nueva-funcionalidad
```

4. Crear Pull Request 🚀

---

# 👨‍💻 Desarrollador

<div align="center">

## Isai Reyes — AI & Full Stack Developer

Desarrollador apasionado por la inteligencia artificial, automatización, asistentes virtuales y arquitecturas modernas de software 🚀

</div>

---

# 🌟 Apoya el proyecto

⭐ Dale una estrella

🍴 Haz Fork

📢 Comparte el proyecto

🤖 Ayuda a construir el futuro de los asistentes inteligentes

---

# 📜 Licencia

Proyecto Open Source orientado al desarrollo de asistentes virtuales privados, inteligencia artificial local y automatización avanzada.

---

<div align="center">

### 🤖 JARVIS AI Assistant — Tu asistente inteligente, privado y siempre disponible 🚀

</div>
