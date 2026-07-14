# claudetoolkit-office

Visualizador en tiempo real del pipeline de agentes de **unmassk** (Claude Code), renderizado como una **oficina isométrica pixel art** de fondo fijo donde cada agente es un personaje con su propio puesto.

Herramienta interna + demo de agencia. **Está acoplada a unmassk**: no es un producto genérico y no funciona sin él. Será un plugin más del toolkit.

## Estado

En **diseño**. Todavía no hay código del proyecto: por regla, no se escribe una línea hasta congelar y aprobar la especificación.

- [x] **Fase 0 — Investigación** — cómo observar el trabajo de Claude Code (formato de datos)
- [ ] **Fase 1 — Grill** — cerrar la spec: alcance de v1, eventos, roles reales, motor de render…
- [ ] Spec congelada
- [ ] Código

## Cómo va a funcionar (alto nivel)

Doble fuente de eventos:

- **JSONL de Claude Code** (`~/.claude/projects/`) → el movimiento: quién trabaja y qué herramienta usa en cada momento.
- **Hooks propios de unmassk** → la semántica de dominio: gates de Cerberus/Argus, score de Yoda, compresión de memoria de Gitto, ciclo del loop… Es la fuente prioritaria y estable; si Claude Code cambia su formato se pierde animación fina, no el estado del pipeline.

## Documentación

Ver [`docs/`](docs/) para el índice completo. Destacado:

- [`docs/jsonl-schema-reference.md`](docs/jsonl-schema-reference.md) — diccionario del formato JSONL de Claude Code (entregable de la Fase 0).
