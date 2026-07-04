# Kiro Skill: Linear

Skill para [Kiro IDE](https://kiro.dev) que permite gestionar issues, ciclos y proyectos de [Linear.app](https://linear.app) directamente desde el agente AI usando la API GraphQL.

## Qué incluye

```
├── SKILL.md                    # Instrucciones del skill para el agente
├── assets/
│   ├── queries.graphql         # Todas las queries de lectura
│   └── mutations.graphql       # Todas las mutaciones (con preview obligatorio)
└── references/
    └── api-notes.md            # Referencia rápida: auth, filtros, paginación, rate limits
```

## Instalación paso a paso

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

### 3. Instalar el skill en Kiro

Cloná este repo dentro de tu directorio de skills de Kiro:

```bash
# Skills globales (disponible en todos los proyectos)
git clone https://github.com/GPnoir/kiro-skill-linear.git ~/.kiro/skills/linear

# O skills de un proyecto específico
git clone https://github.com/GPnoir/kiro-skill-linear.git .kiro/skills/linear
```

### 4. Personalizar para tu equipo

Editá `SKILL.md` y cambiá la referencia al equipo:

```markdown
# Línea actual:
- Team: "Equipo AppStride" — resolve team ID on first use and cache it.

# Cambialo por tu equipo:
- Team: "Tu Equipo" — resolve team ID on first use and cache it.
```

También en `references/api-notes.md`, actualizá la sección "Team Context" al final del archivo.

### 5. Verificar que funciona

En Kiro, pedile al agente:

```
listame los issues del equipo
```

Debería resolver el team ID automáticamente y mostrarte los issues.

## Uso

Una vez instalado, podés pedirle al agente cosas como:

| Pedido | Qué hace |
|--------|----------|
| "listame los issues" | Muestra issues del equipo con estado, prioridad y asignado |
| "creá un issue: Fix login bug" | Crea un issue (siempre pide confirmación antes) |
| "asigname el APP-42" | Asigna el issue al usuario actual |
| "comentá en APP-42: listo, deployado" | Agrega un comentario al issue |
| "qué hay en el sprint actual?" | Muestra el ciclo activo con sus issues |
| "issues de alta prioridad" | Filtra por prioridad 1 (Urgent) y 2 (High) |
| "buscá issues de auth" | Busca por texto en títulos y descripciones |

## Seguridad

- La API key se lee del entorno (`$LINEAR_API_KEY`), nunca se hardcodea
- Toda mutación (crear, editar, borrar) muestra un preview y pide confirmación antes de ejecutar
- No se almacenan tokens en archivos del skill

## Requisitos

- [Kiro IDE](https://kiro.dev) instalado
- `curl` y `jq` disponibles en el PATH
- Una API key personal de Linear

## Licencia

Apache-2.0
