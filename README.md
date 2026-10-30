# 🔎 Agente investigador con LangChain

Agente didáctico construido con `create_agent` de LangChain. Busca en la web, consulta una base de conocimiento propia (RAG), planifica con TODOs, carga *skills* y genera PDF a partir de LaTeX. La interfaz de Streamlit muestra en vivo su razonamiento y cada herramienta que usa.

## Requisitos

- Python 3.10 o superior y Git
- `GOOGLE_API_KEY` ([AI Studio](https://aistudio.google.com/apikey)): **obligatoria siempre**, también se usa para los embeddings del RAG
- `TAVILY_API_KEY` ([Tavily](https://app.tavily.com)): búsqueda web
- [Tectonic](https://tectonic-typesetting.github.io/book/latest/installation) (compilador de LaTeX para los PDF)
- Un modelo con **tool calling**: Gemini, LM Studio (local) u OpenRouter

## Instalación

**1. Clonar y entrar**

```bash
git clone https://github.com/<usuario>/<repositorio>.git
cd <repositorio>
```

**2. Entorno virtual e instalación de dependencias**

```bash
python -m venv .venv
.venv\Scripts\Activate.ps1        # Windows (macOS/Linux: source .venv/bin/activate)
pip install -r requirements.txt
```

**3. Instalar Tectonic**

- Windows: descargá `tectonic.exe` desde [releases](https://github.com/tectonic-typesetting/tectonic/releases) y guardalo en una carpeta fija.
- macOS: `brew install tectonic`. Con Conda: `conda install -c conda-forge tectonic`.

**4. Configurar el `.env`**

```bash
cp .env.example .env              # Windows: Copy-Item .env.example .env
```

Completá las claves y elegí el modelo:

| Modelo | Configuración |
|---|---|
| Gemini | `MODEL_PROVIDER=google_genai` y `MODEL_NAME=gemini-3.5-flash` |
| LM Studio | `MODEL_PROVIDER=lmstudio`, `MODEL_NAME=<id exacto del modelo cargado>` y `LMSTUDIO_BASE_URL=http://localhost:1234/v1` (servidor iniciado en la pestaña *Developer*) |
| OpenRouter | `MODEL_PROVIDER=openrouter`, `MODEL_NAME=<slug del modelo>` y `OPENROUTER_API_KEY` |

Si Tectonic no quedó en el `PATH`, agregá `TECTONIC_PATH=<ruta completa a tectonic.exe>`.

**5. Indexar los documentos del RAG** (una vez, y cada vez que los cambies)

```bash
python -m rag.ingest
```

**6. Ejecutar**

```bash
streamlit run app.py              # interfaz web
python agent.py                   # o por consola
```

## Primera prueba

1. *"Compará LangChain y LangGraph usando tu base de conocimiento."* → consulta al RAG.
2. *"¿Qué novedades hay de LangGraph este año?"* → búsqueda web.
3. *"Armame un PDF breve con esa comparación."* → carga la skill, escribe LaTeX y ofrece el PDF para descargar.

La primera compilación de PDF es lenta: Tectonic descarga paquetes solo esa vez.

## Estructura

```
├── app.py          # UI de Streamlit
├── agent.py        # Agente: prompt, create_agent, streaming, conversaciones
├── tools.py        # Estado y las 6 tools
├── model.py        # Fábrica de modelos de chat y de embeddings
├── chat_lmstudio.py
├── rag/            # store.py, ingest.py y docs/ (documentos de la base de conocimiento)
├── skills/         # Instrucciones que el modelo carga bajo demanda (latex.md)
└── guias/          # Material de clase (opcional)
```

Se generan al usar la app (no se versionan): `memoria.db`, `conversaciones.txt`, `rag/chroma_db/` y `output/`.

## Notas

- El razonamiento del modelo es opcional: si no lo devuelve, la UI no muestra esos bloques y todo lo demás funciona igual.
- La creación de PDF ejecuta Tectonic sin shell, con argumentos fijos y en modo `--untrusted`, y el cuerpo del LaTeX se filtra con una lista de comandos prohibidos. Es adecuado para uso local; no lo expongas a terceros sin un sandbox.
- Para reiniciar los datos, cerrá la app y borrá `memoria.db`, `conversaciones.txt` y `rag/chroma_db/`.

## Solución de problemas

| Síntoma | Causa probable |
|---|---|
| Al iniciar: "Falta ... API_KEY" | Clave ausente en el `.env` |
| "No se encontró Tectonic" | No instalado o fuera del `PATH`: definí `TECTONIC_PATH` |
| "Falta la base vectorial del RAG" | No se ejecutó `python -m rag.ingest` |
| Error 429 | Límite de peticiones del proveedor; esperá un minuto y repetí |
| El agente no usa las tools | El modelo no soporta tool calling o es demasiado chico |

## Créditos

Los documentos de `rag/docs/` provienen de los repositorios oficiales de [LangChain](https://github.com/langchain-ai/langchain) y [LangGraph](https://github.com/langchain-ai/langgraph) y pertenecen a sus autores; se rigen por la licencia de esos repositorios.

Autor: *(Facundo Maya)*
