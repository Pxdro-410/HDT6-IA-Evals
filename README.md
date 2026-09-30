# Evaluación de Sistemas Multi-Agente (AI Evals) - Parachute S.A.
### Implementación de Pruebas Automatizadas con Promptfoo

**Autores:** Pedro Caso y Hugo Méndez  
**Curso:** AI Engineering Creativity  
**Hoja de Trabajo 6 (HDT6):** AI Evals  

---

## 📖 Descripción del Proyecto

Este proyecto se centra en la **evaluación automatizada y sistemática** de un sistema multi-agente desarrollado para **Parachute S.A.** (empresa de paracaidismo tándem en Guatemala). Utilizando el framework **Promptfoo**, se validaron las capacidades del agente conversacional centralizado que opera sobre dos capacidades de negocio:

1. **Resolución de Preguntas Frecuentes (FAQs):** Consultas sobre requerimientos respaldadas por una base de datos vectorial en PostgreSQL con `pgvector` y embeddings locales (`all-MiniLM-L6-v2`).
2. **Agendamiento y Factibilidad de Citas:** Evaluación de fechas de salto consultando la API meteorológica de **Open-Meteo** para verificar estrictamente la velocidad del viento, ráfagas, precipitación y nubosidad.

---

## 🔬 Metodología de Evaluación

Para garantizar el comportamiento robusto del sistema, se diseñó una suite de evaluación (`promptfooconfig.yaml`) que cubre 4 dimensiones críticas exigidas:

1. **Factuality (`model-graded-closedqa`):** Verifica que el agente no alucine información y responda con precisión sobre restricciones médicas, de edad, peso y condiciones meteorológicas reales.
2. **Determinísticos (`regex` / `icontains`):** Valida la presencia de datos críticos y formatos exactos en la respuesta final (ej. `\d{4}-\d{2}-\d{2}` para fechas ISO).
3. **Latencia (`latency`):** Asegura que el tiempo total de respuesta (incluyendo el ruteo del LLM, consultas a base de datos vectorial y peticiones HTTP a APIs externas) se mantenga bajo los umbrales operativos.
4. **Tool Execution (`javascript`):** Inspecciona los metadatos de las llamadas al agente para comprobar que el Supervisor Central delegue correctamente a los Workers correspondientes (FAQ o Weather) y con los argumentos estructurados de manera adecuada.

### Casos de Prueba Implementados

| ID Test | Dominio | Tipo de Evaluación | Objetivo de la Prueba | Aserción |
| :--- | :--- | :--- | :--- | :--- |
| `FAQ-01` | RAG FAQs | Factuality | Validación de regla de peso máximo. | `model-graded-closedqa` |
| `FAQ-02` | RAG FAQs | Factuality | Validación de regla de edad mínima. | `model-graded-closedqa` |
| `FAQ-03` | RAG FAQs | Determinístico | Verificar espera obligatoria tras buceo. | `icontains: 24 horas` |
| `FAQ-04` | RAG FAQs | Determinístico | Verificar altura correcta de salto. | `regex: 3,000\|10,000` |
| `FAQ-05` | RAG FAQs | Latencia | Garantizar RAG rápido. | `latency < 25000 ms` |
| `FAQ-06` | RAG FAQs | Tool Execution | Verificar ruteo exclusivo a FAQ Worker. | `javascript` (inspect tools) |
| `CLIMA-01` | Clima | Factuality | Validación de clima vs recomendación. | `model-graded-closedqa` |
| `CLIMA-02` | Clima | Determinístico | Verificación de fecha ISO y conclusión. | `regex: \d{4}-\d{2}-\d{2}` |
| `CLIMA-03` | Clima | Latencia | Rendimiento de LLM + Open-Meteo. | `latency < 60000 ms` |
| `CLIMA-04` | Clima | Tool Execution | Ruteo y validación ISO a Weather Worker. | `javascript` (inspect args) |

---

## 🚀 Tecnologías y Herramientas

* **Python 3:** Lenguaje principal de orquestación de los agentes.
* **Promptfoo:** Framework de evaluación de LLMs e Inteligencia Artificial generativa.
* **PostgreSQL + pgvector:** Motor relacional con almacenamiento vectorial.
* **Sentence Transformers (`all-MiniLM-L6-v2`):** Embeddings locales.
* **Open-Meteo API:** Evaluaciones climáticas en tiempo real.
* **OpenAI Python SDK & NVIDIA NIM:** Plataforma proveedora del modelo `meta/llama-3.2-11b-vision-instruct`.

---

## 📂 Estructura del Proyecto (Evals)

```text
HDT6-IA-Evals/
├── .env                              # Credenciales (NVIDIA_API_KEY, OPENAI_API_KEY)
├── promptfooconfig.yaml              # Configuración y suite de 10 casos de prueba
├── agent_provider.py                 # Custom Provider de Python para conectar Promptfoo con el agente
├── package.json                      # Scripts de ejecución de NPM
├── eval_report.html                  # Reporte estático de resultados autogenerado
├── services.py                       # Herramientas de dominio (RAG y Clima)
├── centralized.py                    # Agente principal a evaluar (Supervisor -> Workers)
├── load_faqs.py                      # Script de ingesta de FAQs
├── Corpus_FAQs_Parachute_SA_2026.txt # Base de conocimiento oficial
└── README.md                         # Este documento
```

---

## ⚙️ Requisitos Previos

1. **Docker Desktop** (o Docker Engine + Compose) instalado y activo.
2. **Python 3.10+** y **Node.js** instalados en el sistema.
3. **API Key de NVIDIA Build** y **OpenAI API Key** (para el modelo juez evaluador).

---

## 💻 Instrucciones de Ejecución

Sigue estos pasos en orden para inicializar el entorno y correr las evaluaciones:

### 1. Inicializar la infraestructura (Docker)
Levanta el contenedor de PostgreSQL con la extensión `pgvector`:
```bash
docker compose up -d
```

### 2. Entorno virtual y dependencias
```bash
# Crear entorno virtual
python -m venv .venv

# Activar en macOS / Linux:
source .venv/bin/activate
# En Windows:
.venv\Scripts\activate

# Instalar dependencias Python
pip install -r requirements.txt

# Instalar dependencias de JS (Promptfoo)
npm install
```

### 3. Cargar la base de conocimientos
Configura las variables de entorno basándote en el archivo `.example_env` y luego ejecuta el script de ingesta:
```bash
python load_faqs.py
```

### 4. Ejecutar la suite de pruebas
Se utiliza el script configurado en `package.json`, el cual inyecta automáticamente el archivo `.env`:
```bash
npm run eval
```

### 5. Analizar Resultados
Promptfoo incluye un visor web interactivo con filtros, análisis de latencia y detalle de aserciones. Inícialo con:
```bash
npm run view
```
*(También puedes consultar el archivo estático `eval_report.html` autogenerado en la raíz del proyecto).*
