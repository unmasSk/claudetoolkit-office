# Documentación — claudetoolkit-office

Mapa de la documentación del proyecto. No es solo lo que ya existe: es también **dónde va a ir cada cosa** conforme cerremos la spec, para que nada quede suelto.

Leyenda: ✅ hecho · 🚧 en curso · ⬜ pendiente · 🔒 bloqueado

## Investigación (Fase 0)

| Doc | Estado | Qué es |
|---|---|---|
| [jsonl-schema-reference.md](jsonl-schema-reference.md) | ✅ | Diccionario del formato JSONL de Claude Code: tipos de línea, herramientas, identidad de agentes, anidamiento, workflows y campos útiles para animar. |

## Arquitectura y eventos (se cierra en el grill — Fase 1)

| Doc | Estado | Qué contendrá |
|---|---|---|
| `architecture.md` | ⬜ | Arquitectura del visualizador: doble fuente de eventos, parser del JSONL, receptor local, canal hacia el navegador, motor de render. Alternativas consideradas y por qué se descartan. |
| `event-schema.md` | ⬜ | Nuestros **eventos tipados de dominio** emitidos por los hooks de unmassk (gates de Cerberus/Argus, score de Yoda, compresión de Gitto, ciclo del loop…). El contrato estable del pipeline. |
| `v1-scope.md` | ⬜ | Qué entra en la v1 y qué queda explícitamente fuera. Solo lectura vs control bidireccional. |

## Diseño visual (Fase C — parte bloqueada por el PNG ancla)

| Doc | Estado | Qué contendrá |
|---|---|---|
| `agent-roles.md` | ⬜ | Cada agente de unmassk, su **función real** y su **metáfora visual** en la oficina (puesto, estados, qué hace en cada evento). Incluye validar los roles marcados como borrador. |
| `visual-spec.md` | 🔒 | Paleta, resolución, ángulo isométrico, estilo de sprites y props. **Bloqueado** hasta localizar el PNG de la oficina (ancla de estilo). |
| `render.md` | ⬜ | Motor de render: fondo único, capa de oclusores, personajes con y-sorting, props animables (2–4 frames). |

## Ingeniería (Fase D–E)

| Doc | Estado | Qué contendrá |
|---|---|---|
| `standards.md` | ⬜ | Estándares de calidad del plugin (hereda los de unmassk). |
| `testing.md` | ⬜ | Estrategia de test: replay de JSONL grabados como fixtures deterministas. |

---

> Los nombres de archivo son tentativos. Este mapa se actualiza conforme el grill (Fase 1) vaya cerrando decisiones — cada bloque anticipa una parte de la spec, no es documentación ya escrita.
