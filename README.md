# ⚖️ DEMO — Asistente Legal con Vector Store en Memoria

Agente conversacional de IA especializado en consultas legales sobre documentos PDF. Permite subir contratos, reglamentos o cualquier documento legal y hacer preguntas en lenguaje natural sobre su contenido, usando RAG con vector store en memoria y GPT-4.1-mini.

> **Nota:** Este proyecto es una demostración técnica de RAG con vector store en memoria. Para un entorno de producción con sincronización de Google Drive y persistencia en base de datos, consulta [Asistente-Legal-RAG-Sync-Google-Drive-PGVector](https://github.com/Jairh1294/Asistente-Legal-RAG-Sync-Google-Drive-PGVector).

---

## 🧩 El problema que resuelve

Revisar un contrato o reglamento extenso para encontrar una cláusula específica es lento y tedioso. Un abogado, asistente legal o estudiante de derecho puede tardar horas buscando información puntual en documentos de 50, 100 o más páginas.

**Este asistente permite subir el PDF y preguntar directamente:** "¿Qué dice la cláusula de rescisión?", "¿Cuál es el plazo de entrega?", "¿Hay penalizaciones por incumplimiento?" — y obtener una respuesta precisa en segundos, citando el documento.

---

## 🏗️ Arquitectura

```
┌──────────────────────────────────────────────────────────────┐
│            ASISTENTE LEGAL — VECTOR STORE EN MEMORIA         │
│                                                              │
│  FLUJO 1: INGESTIÓN DE DOCUMENTO                             │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Form Trigger (subida de PDF)                          │  │
│  │           │                                            │  │
│  │           ▼                                            │  │
│  │  Extraer texto del PDF                                 │  │
│  │           │                                            │  │
│  │           ▼                                            │  │
│  │  Recursive Character Text Splitter                     │  │
│  │  (chunkSize configurable, overlap: 250 tokens)         │  │
│  │           │                                            │  │
│  │           ▼                                            │  │
│  │  Generar embeddings (text-embedding-3-large)           │  │
│  │           │                                            │  │
│  │           ▼                                            │  │
│  │  Almacenar en Vector Store en Memoria                  │  │
│  │  "¡Documento listo! Ahora puedes hacer preguntas."     │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  FLUJO 2: CONSULTA AL AGENTE                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Chat Trigger (pregunta del usuario)                   │  │
│  │           │                                            │  │
│  │           ▼                                            │  │
│  │  AI Agent (GPT-4.1-mini)                              │  │
│  │  + Herramienta: Vector Store retrieval                 │  │
│  │  + Memory: Window Buffer (5 turnos)                    │  │
│  │  + System Prompt: abogado especialista                 │  │
│  │           │                                            │  │
│  │           ▼                                            │  │
│  │  Respuesta fundamentada en el documento                │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Stack tecnológico

| Componente | Tecnología |
|---|---|
| Orquestación | n8n |
| Modelo de lenguaje | OpenAI GPT-4.1-mini |
| Embeddings | OpenAI text-embedding-3-large |
| Vector Store | In-Memory Vector Store (n8n nativo) |
| Text Splitter | Recursive Character Text Splitter |
| Entrada de documentos | Form Trigger n8n (subida de PDF) |
| Interfaz de consulta | Chat Trigger n8n |
| Memoria de conversación | Window Buffer Memory (5 turnos) |

---

## 🤖 Comportamiento del agente

El agente tiene un system prompt especializado que lo instruye a:

- Actuar como **abogado especialista** en el área del documento subido
- Responder **únicamente con base en el documento** cargado
- Indicar explícitamente cuando la información solicitada **no está en el documento**
- Usar **lenguaje claro y accesible**, evitando jerga innecesaria
- Mantener el **contexto de la conversación** para preguntas de seguimiento

---

## 🔄 Flujo detallado

### 1. Ingestión del documento

El usuario sube un PDF a través del formulario de n8n. El flujo:

1. **Extrae el texto** completo del PDF
2. **Divide el contenido** en fragmentos manejables usando el Recursive Character Text Splitter con superposición de 250 tokens para no perder contexto entre chunks
3. **Genera embeddings** vectoriales para cada fragmento con `text-embedding-3-large` (3072 dimensiones, máxima precisión semántica)
4. **Almacena los vectores** en el In-Memory Vector Store de n8n
5. **Confirma al usuario** que el documento está listo para consultas

### 2. Consulta conversacional

El usuario escribe su pregunta en el chat. El agente:

1. **Recupera los fragmentos más relevantes** del vector store mediante búsqueda de similitud semántica
2. **Construye el contexto** con esos fragmentos + historial de conversación (últimos 5 turnos)
3. **Genera la respuesta** con GPT-4.1-mini, fundamentada en el contenido del documento
4. **Mantiene la conversación** para preguntas de seguimiento

---

## ⚙️ Configuración

### Credenciales requeridas en n8n

| Credencial | Descripción |
|---|---|
| `OpenAI API` | API Key de OpenAI para embeddings y GPT-4.1-mini |

### Parámetros configurables

| Parámetro | Valor por defecto | Descripción |
|---|---|---|
| Chunk Overlap | 250 tokens | Superposición entre fragmentos para mantener contexto |
| Memory Window | 5 turnos | Historial de conversación que el agente recuerda |
| Embedding Model | text-embedding-3-large | Modelo para generar vectores |
| Chat Model | GPT-4.1-mini | Modelo para generar respuestas |

---

## 🚀 Instalación

1. **Clona el repositorio**
   ```bash
   git clone https://github.com/Jairh1294/-DEMO-Asistente-Legal-Vector-Store-en-memoria.git
   ```

2. **Importa el workflow en n8n**
   - Ve a **Workflows → Import from File**
   - Selecciona el archivo `.json` del repositorio

3. **Configura las credenciales**
   - En n8n: **Settings → Credentials → New**
   - Agrega tu **OpenAI API Key**

4. **Activa el workflow**
   - Haz clic en **Activate**
   - Copia la URL del Form Trigger para subir documentos
   - Copia la URL del Chat Trigger para hacer consultas

---

## 💬 Ejemplo de uso

```
📄 [Usuario sube: contrato_arrendamiento.pdf]

✅ Bot: ¡Documento cargado correctamente! El contrato tiene 24 páginas
       y ya está indexado. Puedes hacerme preguntas sobre su contenido.

👤 Usuario: ¿Cuánto tiempo dura el contrato?

🤖 Agente: Según la Cláusula 3 del contrato, la duración es de
           12 meses, iniciando el 1 de enero de 2025 y
           concluyendo el 31 de diciembre de 2025, con opción
           de renovación automática por períodos iguales.

👤 Usuario: ¿Hay penalización por terminar antes?

🤖 Agente: Sí. De acuerdo con la Cláusula 8 (Terminación
           Anticipada), si el arrendatario rescinde antes de
           los 6 meses, deberá pagar el equivalente a 2 meses
           de renta como penalización.
```

---

## ⚠️ Limitaciones de la versión demo

| Limitación | Descripción |
|---|---|
| **Persistencia** | Los vectores se almacenan en memoria RAM; se pierden al reiniciar n8n |
| **Un documento a la vez** | El vector store en memoria no separa namespaces por documento |
| **Sin sincronización** | Los documentos deben subirse manualmente cada vez |
| **Escalabilidad** | No recomendado para equipos con múltiples usuarios simultáneos |

Para superar estas limitaciones, consulta la versión con **PostgreSQL PGVector y sincronización con Google Drive**.

---

## 📄 Licencia

MIT License — libre para uso comercial y personal.

---

*Construido con n8n · OpenAI GPT-4.1-mini · text-embedding-3-large*
