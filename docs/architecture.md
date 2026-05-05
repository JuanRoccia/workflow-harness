# Arquitectura — Qué significa "hacer un buen trabajo"

> Este documento define el estándar de calidad del workflow harness.
> Los agentes evalúan su trabajo contra este archivo.

## Principios

1. **Capas claras.** El harness tiene capas definidas:
   - `agents/` — definiciones de roles y comportamiento de agentes.
   - `tasks/` — tareas específicas asignadas a cada agente.
   - `skills/` — capacidades especializadas reutilizables.
   - `progress/` — seguimiento de sesiones y bitácora.
   No introducir capas adicionales sin documentar la razón en `feature_list.json`.

2. **Sin dependencias externas innecesarias.** Bash y Python stdlib son suficientes.
   Si se requiere una herramienta externa, primero documentarse en `feature_list.json` (estado `blocked`).

3. **Errores explícitos.** Los scripts y archivos de configuración deben validar
   entradas y lanzar errores claros con mensajes descriptivos.

4. **Atomicidad en archivos críticos.** Operaciones sobre `feature_list.json`,
   `progress/current.md` y archivos de configuración deben ser atómicas.
   Escribir a archivo temporal y luego `mv`/`rename`.

5. **Inmutabilidad de registros históricos.** `progress/history.md` es append-only.
   Nunca editar entradas anteriores.

## Flujo del workflow

```
usuario ─→ init.sh (verificación)
           │
           ├─→ AGENTS.md (lee instrucciones)
           │
           └─→ agents/[rol].md (asume rol)
                │
                ├─→ tasks/task-[rol].md (lee tarea)
                │
                └─→ progress/current.md (registra trabajo)
                     │
                     └─→ feature_list.json (actualiza estado)
```

## Estructura de datos

### feature_list.json
- Formato JSON estructurado con array de `features`.
- Cada feature tiene: `id`, `name`, `title`, `description`, `priority`, `acceptance`, `status`.
- Estados válidos: `pending`, `in_progress`, `done`, `blocked`.

### progress/current.md
- Plantilla de sesión actual.
- Se vacía al cerrar sesión y se mueve a `history.md`.

### agents/*.md
- Definición de rol, responsabilidades, instrucciones específicas.
- Un archivo por agente siguiendo patrón: `NN-nombre.md`.

## Qué NO hacer

- No usar rutas hardcodeadas a proyectos específicos (ej: `/home/runner/workspace/`).
- No mezclar múltiples tareas en una sesión.
- No marcar `done` sin ejecutar `./init.sh` y ver tests verdes.
- No editar `progress/history.md` (solo append).
- No dejar archivos temporales ni TODOs sin contexto.
