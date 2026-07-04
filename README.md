# Kiro Skill: Linear

Skill para agentes AI que permite gestionar issues, ciclos y proyectos de [Linear.app](https://linear.app) directamente desde conversación usando la API GraphQL.

Compatible con: **Kiro IDE**, **Claude Desktop**, **ChatGPT (Custom GPTs)**, **Cursor**, **Windsurf**, **Cline/Continue (VS Code)**.

## Qué incluye

```
├── SKILL.md                    # Instrucciones del skill para el agente
├── assets/
│   ├── queries.graphql         # Todas las queries de lectura
│   └── mutations.graphql       # Todas las mutaciones (con preview obligatorio)
└── references/
    └── api-notes.md            # Referencia rápida: auth, filtros, paginación, rate limits
```

## Requisitos previos (todos los setups)

### 1. Obtener tu API Key de Linear

1. Abrí [Linear](https://linear.app) → Settings → Account → **Security & Access**
2. En la sección **Personal API Keys**, creá una nueva clave
3. Copiá la clave generada

### 2. Configurar la variable de entorno

Agregá la clave a tu shell (`~/.zshrc`, `~/.bashrc`, o equivalente):

```bash
export LINEAR_API_KEY="lin_api_xxxxxxxxxxxxxxxxxxxxxxxxxx"
```

Recargá tu shell:

```bash
source ~/.zshrc
```

### 3. Personalizar para tu equipo

Después de instalar, editá `SKILL.md` y cambiá la referencia al equipo:

```markdown
# Línea actual:
- Team: "Equipo AppStride" — resolve team ID on first use and cache it.

# Cambialo por tu equipo:
- Team: "Tu Equipo" — resolve team ID on first use and cache it.
```

También actualizá la sección "Team Context" al final de `references/api-notes.md`.

---

## Instalación por plataforma

### Kiro IDE

```bash
git clone https://github.com/GPnoir/kiro-skill-linear.git ~/.kiro/skills/linear
```

El agente lo detecta automáticamente cuando mencionás Linear, issues, o sprints.

---

### Claude Desktop (Projects)

1. Abrí Claude Desktop → **Projects** → creá o abrí un proyecto
2. En **Project Knowledge**, subí estos archivos:
   - `SKILL.md`
   - `assets/queries.graphql`
   - `assets/mutations.graphql`
   - `references/api-notes.md`
3. En **Project Instructions** (Custom Instructions), pegá:

```
You have access to a Linear skill for managing issues via GraphQL API.
Use the queries and mutations from the uploaded files.
Execute curl commands when the user asks to interact with Linear.
Always preview mutations before executing them.
The API key is available as $LINEAR_API_KEY in the environment.
```

> **Nota**: Claude Desktop no ejecuta comandos directamente. Necesitás tener un MCP server que permita ejecutar shell commands (como `mcp-shell` o similar), o copiar los comandos que Claude genera y ejecutarlos manualmente.

---

### Claude Desktop (MCP)

Si tenés un MCP server con capacidad de ejecutar shell (ej: `@anthropic/mcp-shell`):

1. Agregá el skill como context en tu `claude_desktop_config.json` o project files
2. Claude podrá ejecutar los `curl` directamente contra la API de Linear

---

### ChatGPT — Custom GPT

1. Creá un nuevo GPT en [chat.openai.com/gpts](https://chat.openai.com/gpts/editor)
2. En **Instructions**, pegá el contenido completo de `SKILL.md`
3. En **Knowledge**, subí:
   - `assets/queries.graphql`
   - `assets/mutations.graphql`
   - `references/api-notes.md`
4. En **Actions**, creá una acción con este schema OpenAPI:

```yaml
openapi: 3.1.0
info:
  title: Linear API
  version: "1.0"
servers:
  - url: https://api.linear.app
paths:
  /graphql:
    post:
      operationId: linearGraphQL
      summary: Execute a GraphQL query against Linear
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                query:
                  type: string
                  description: GraphQL query or mutation
                variables:
                  type: object
                  description: Query variables
              required:
                - query
      responses:
        "200":
          description: GraphQL response
```

5. En **Authentication** de la acción, configurá:
   - Type: **API Key**
   - Auth Type: **Custom Header**
   - Header Name: `Authorization`
   - Value: tu `lin_api_...` key

---

### Cursor

1. Cloná el repo en tu proyecto o en `~/.cursor/skills/`:

```bash
git clone https://github.com/GPnoir/kiro-skill-linear.git ~/.cursor/skills/linear
```

2. Creá o editá `.cursor/rules/linear.mdc` en tu proyecto:

```markdown
---
description: Linear issue management via GraphQL API
globs: []
alwaysApply: false
---

@file ~/.cursor/skills/linear/SKILL.md
@file ~/.cursor/skills/linear/assets/queries.graphql
@file ~/.cursor/skills/linear/assets/mutations.graphql
@file ~/.cursor/skills/linear/references/api-notes.md
```

3. Cursor cargará el skill cuando pidas interactuar con Linear.

---

### Windsurf (Cascade)

1. Cloná el repo:

```bash
git clone https://github.com/GPnoir/kiro-skill-linear.git ~/.windsurf/skills/linear
```

2. Creá `.windsurfrules` en tu proyecto o `~/.windsurf/global_rules.md`:

```markdown
## Linear Skill

When the user asks about Linear issues, sprints, or project management,
read and follow the instructions in ~/.windsurf/skills/linear/SKILL.md
Use queries from ~/.windsurf/skills/linear/assets/queries.graphql
Use mutations from ~/.windsurf/skills/linear/assets/mutations.graphql
Reference: ~/.windsurf/skills/linear/references/api-notes.md

Always preview mutations before executing. Use $LINEAR_API_KEY for auth.
```

---

### Cline / Continue (VS Code)

1. Cloná el repo en tu proyecto:

```bash
git clone https://github.com/GPnoir/kiro-skill-linear.git .vscode/skills/linear
```

2. Para **Cline**, agregá en `.clinerules`:

```markdown
## Linear API Skill

When asked about Linear issues, follow instructions in .vscode/skills/linear/SKILL.md
Queries: .vscode/skills/linear/assets/queries.graphql
Mutations: .vscode/skills/linear/assets/mutations.graphql
Reference: .vscode/skills/linear/references/api-notes.md
```

3. Para **Continue**, agregá en `.continuerules`:

```markdown
When managing Linear issues, use the skill files in .vscode/skills/linear/
Always preview mutations before executing.
```

---

## Uso

Una vez instalado, pedile al agente:

| Pedido | Qué hace |
|--------|----------|
| "listame los issues" | Muestra issues del equipo |
| "creá un issue: Fix login bug" | Crea un issue (pide confirmación) |
| "asigname el APP-42" | Asigna el issue al usuario actual |
| "comentá en APP-42: listo, deployado" | Agrega un comentario |
| "qué hay en el sprint actual?" | Muestra el ciclo activo |
| "issues de alta prioridad" | Filtra por urgente y alto |
| "buscá issues de auth" | Busca por texto |

---

## Estimación de costos (tokens)

### Tamaño del skill en contexto

| Archivo | Tokens aprox |
|---------|-------------|
| SKILL.md | ~1,200 |
| queries.graphql | ~3,200 |
| mutations.graphql | ~3,400 |
| api-notes.md | ~1,400 |
| **Total cargado** | **~9,200 tokens** |

### Costo por interacción (input del skill + respuesta típica)

Una sesión típica (cargar skill + 5 operaciones) gasta ~15K-25K tokens totales.

#### Claude (Anthropic)

| Modelo | Costo por sesión (5 ops) | Mejor para |
|--------|--------------------------|------------|
| Haiku 3.5 | ~$0.01 - $0.02 | Ejecución rápida y barata |
| Sonnet 4 | ~$0.04 - $0.08 | Balance calidad/costo |
| Opus 4 | ~$0.15 - $0.30 | Overkill para esto |

#### OpenAI (ChatGPT)

| Modelo | Costo por sesión (5 ops) | Mejor para |
|--------|--------------------------|------------|
| GPT-4.1 mini | ~$0.01 - $0.02 | El más económico |
| GPT-4.1 | ~$0.03 - $0.05 | Buen balance |
| GPT-4o | ~$0.03 - $0.06 | Similar a 4.1 |
| o3 | ~$0.10 - $0.20 | Innecesario |

#### Notas

- **Custom GPT (ChatGPT Plus/Pro)**: el costo está incluido en la suscripción. No pagás tokens extra.
- **Claude Desktop (Pro)**: incluido en la suscripción de $20/mes.
- **Cursor/Windsurf**: incluido en el plan del IDE (usan tus tokens de Anthropic/OpenAI o los del plan).
- **API directa**: los costos de arriba aplican por sesión.

---

## Seguridad

- La API key se lee del entorno (`$LINEAR_API_KEY`), nunca se hardcodea
- Toda mutación muestra un preview y pide confirmación antes de ejecutar
- No se almacenan tokens en archivos del skill
- En Custom GPTs, la key se configura como auth de la Action (nunca en las instructions)

## Licencia

Apache-2.0
