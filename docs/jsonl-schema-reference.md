# Referencia del formato JSONL de Claude Code

> **Versión observada:** Claude Code 2.1.209 (Windows)
> **Verificado contra:** archivos `.jsonl` reales de 3 proyectos de esta máquina (nuestra sesión + `C--Users-bextia` + `poe2-claude`), más un agente "todo-terreno" que ejercitó el máximo de herramientas.
> **Propósito:** referencia canónica de qué escribe Claude Code y dónde, para diseñar el visualizador. Todo es **VERIFICADO** contra archivo real salvo lo marcado como *NO CONFIRMADO*.
> **Nota:** este documento se duplica a propósito con la memoria de git (memo `stack`). Si cambia el formato en una versión futura, actualizar ambos.

---

## 1. Dónde viven los datos

- **Raíz:** `~/.claude/projects/<proyecto-codificado>/`
  - El nombre del proyecto se codifica reemplazando todo carácter **no alfanumérico por `-`**. Ej.: `C:\Users\bextia\Projects\claudetoolkit-office` → `C--Users-bextia-Projects-claudetoolkit-office`. Lookup case-insensitive en Windows.
- **Sesión principal:** `<sessionId>.jsonl` (un archivo por sesión).
- **Subagentes:** `<sessionId>/subagents/agent-<agentId>.jsonl` + sidecar `agent-<agentId>.meta.json`.
  - El anidamiento es **plano**: aunque un agente llame a otro (profundidad 2), todos los `agent-*.jsonl` viven en el **mismo** directorio `subagents/`, sin subcarpetas por profundidad.
- **Workflows:** `<sessionId>/subagents/workflows/wf_<hash>/` agrupa N subagentes hermanos + un `journal.jsonl`.
- **`sessionId` es idéntico** en todos los niveles (principal, subagente, nieto). El anidamiento no crea sesión nueva.
- **`isSidechain`**: `false` en el archivo principal, `true` en todas las líneas de cualquier archivo de subagente. Campo canónico para distinguir principal vs subagente.

---

## 2. Tipos de línea (`type` top-level)

| `type` | Significado |
|---|---|
| `user` | Turno de usuario **o** retorno de un `tool_result` |
| `assistant` | Turno del modelo (texto / thinking / tool_use) |
| `attachment` | Adjunto asociado a un turno. Forma vista: `attachment.type = "deferred_tools_delta"` con `addedNames[]` (herramientas diferidas descubribles, incluidas MCP) |
| `system` | Evento de sistema — ver subtypes abajo |
| `mode` | Cambio de modo de sesión: `{mode:"normal", ...}` |
| `permission-mode` | Cambio de modo de permisos: `{permissionMode:"bypassPermissions", ...}` |
| `last-prompt` | `{lastPrompt, leafUuid, sessionId}` |
| `ai-title` | Título auto-generado de la sesión: `{aiTitle, sessionId}` |
| `queue-operation` | `{operation, timestamp, sessionId}` — `operation` ∈ `enqueue / dequeue / remove / popAll` |
| `file-history-snapshot` | `{messageId, snapshot:{trackedFileBackups, timestamp}, isSnapshotUpdate}` |
| `file-history-delta` | `{messageId, snapshotMessageId, trackingPath, backup:{backupFileName, version, backupTime}, timestamp}` |

**Subtypes de `system`:** `local_command`, `stop_hook_summary`, `turn_duration`, `away_summary`, `compact_boundary`.

- `stop_hook_summary` → `{hookCount, hookInfos:[{command, durationMs}], hookErrors, preventedContinuation, stopReason, hasOutput, toolUseID}`. **Registra los hooks externos con su duración** (ej. visto `node "C:\Users\bextia\.pixel-agents\hooks\claude-hook.js"`).
- `turn_duration` → `{durationMs, messageCount, timestamp}`.
- `away_summary` → `{content}` (recap de ausencia).
- `compact_boundary` → `{content:"Conversation compacted", compactMetadata:{trigger, preTokens, durationMs, preCompactDiscoveredTools}, logicalParentUuid}`. **`parentUuid` viene `null`**; usar **`logicalParentUuid`** como enlace alternativo tras compactar.

> **`type:"progress"` NO existe en esta versión.** No aparece ni con subagentes en background ni en foreground (agent-flow lo implementa para otras versiones). **El parser propio debe ser fail-open** ante tipos desconocidos: el formato evoluciona entre versiones.

---

## 3. Bloques dentro de `message.content[]`

| Bloque | Forma | Notas |
|---|---|---|
| `text` | `{type, text}` | Texto plano |
| `thinking` | `{type, thinking, signature}` | Si `thinking` está vacío y `signature` es un string largo (~6000 chars) = **thinking redactado**. Hay que manejarlo para no pintar una burbuja vacía |
| `tool_use` | `{type, id, name, input, caller}` | `caller` siempre `{type:"direct"}` en lo observado |
| `tool_result` | `{type, tool_use_id, content, is_error}` | `content` = string **o** array `[{type:"text", text}]` según la herramienta |

**Error real** (`is_error: true`): `content` es un **string** envuelto en la pseudo-etiqueta `<tool_use_error>...</tool_use_error>`. Ej.: `<tool_use_error>File has not been read yet...</tool_use_error>`.

---

## 4. Catálogo por herramienta

Formato: `input` (campos) → forma de `toolUseResult` (el campo hermano en la línea `user`, con la metadata rica).

| Herramienta (`tool_use.name`) | `input` | `toolUseResult` |
|---|---|---|
| `Bash` / `PowerShell` | `command, description` | dict: `stdout, stderr, interrupted, isImage, ...` |
| `Glob` | `pattern, path` | dict: `filenames, numFiles, totalMatches, truncated, durationMs` |
| `Grep` | `pattern, path, output_mode` | dict: `content, filenames, mode, numFiles, numMatches` |
| `Read` | `file_path` | dict: `file, type` (content = string con nº de línea, o array para notebook) |
| `Write` | `content, file_path` | dict: `filePath, content, structuredPatch, userModified, ...` |
| `Edit` | `file_path, old_string, new_string, replace_all` | dict: `filePath, oldString, newString, structuredPatch, replaceAll, ...` |
| `NotebookEdit` | `cell_id, new_source, notebook_path` | dict: `cell_type, edit_mode, language, old_source, new_source, ...` |
| `WebSearch` | `query` | dict: `query, results, searchCount, durationSeconds` |
| `WebFetch` | `url, prompt` | dict: `url, bytes, code, result, durationMs` |
| `ToolSearch` | `query, max_results` | dict: `matches, query, total_deferred_tools` (content = array de `tool_reference`) |
| `Monitor` | `command, description, persistent, timeout_ms` | dict: `taskId, persistent, timeoutMs` |
| `Agent` | `subagent_type, description, prompt, run_in_background` | dict rico: `agentId, agentType, status, resolvedModel, totalDurationMs, totalTokens, totalToolUseCount, usage.iterations[]` |
| `SendMessage` | `to, message, summary, recipient, content, type` | dict: `success, message, pin, resumedAgentId` |
| `Skill` | `skill` | dict: `allowedTools, commandName, success` |
| Tools MCP (`mcp__<server>__<tool>`) | según la tool | **array** de bloques (sin metadata extra) — asimetría con las nativas |

**Notas importantes:**
- **`TodoWrite` NO existe** en este entorno (ni activa ni diferida).
- **Nativas** → `toolUseResult` es un **dict** con metadata rica. **MCP** → `toolUseResult` es directamente el **array** de contenido, sin metadata. Si el visualizador quiere mostrar metadata, las MCP no la traen.

---

## 5. Identidad y atribución de agentes

Para **nombrar cada personaje** en pantalla, por orden de fiabilidad:

| Campo | Dónde vive | Ejemplo | Fiabilidad |
|---|---|---|---|
| `tool_use.input.subagent_type` | en el archivo del **padre**, en el `tool_use` que lanza al agente | `"general-purpose"`, `"unmassk-toolkit:bilbo"` | Identidad declarada al invocar |
| `attributionAgent` | top-level, en las líneas `assistant` del **propio** archivo del subagente | `"unmassk-toolkit:bilbo"` | Identidad real atribuida por Claude Code |
| `attributionPlugin` | ídem | `"unmassk-toolkit"` | Solo en agentes de plugin |
| `agentType` | en el `.meta.json` | `"general-purpose"` | Coincide con `subagent_type` |
| `agentId` | top-level + nombre de archivo | `af629c9dafadcfb5a` | Estable |

- **Tools MCP:** derivar servidor/tool del propio `name` (`mcp__<server>__<tool>`), que es 100 % consistente. **NO fiarse** de `attributionMcpServer` / `attributionMcpTool`: se observaron **desalineados** con el `tool_use` de su propia línea.
- ⚠️ **El crew de unmassk (Ultron, Cerberus, Dante, Argus, House, Yoda, Alexandria, Gitto) NO aparece en ningún dato real existente.** El único slug confirmado es `unmassk-toolkit:bilbo` (por usarlo hoy). El formato `unmassk-toolkit:<agente>` para el resto es *NO CONFIRMADO* hasta verlos correr en vivo.

---

## 6. Anidamiento (spawnDepth)

- El `.meta.json` de cada subagente: `{agentType, description, toolUseId, spawnDepth [, parentAgentId]}`.
- **`spawnDepth`** = contador de profundidad (1 = lanzado desde la sesión principal, 2 = lanzado por otro subagente…).
- **`parentAgentId`** aparece **solo si `spawnDepth >= 2`** y apunta al `agentId` del padre → **campo autoritativo del enlace jerárquico**.
- **`toolUseId`** del meta enlaza con el `tool_use.id` del bloque `Agent` que lo lanzó (en el archivo del padre).
- Al reanudar un subagente con `SendMessage`, los nuevos turnos **se anexan al mismo archivo** del subagente (no se crea uno nuevo).

---

## 7. Workflows (fan-out paralelo)

- `subagents/workflows/wf_<hash>/` agrupa **N subagentes hermanos** homogéneos.
- Su `.meta.json` es **mínimo**: `{agentType:"workflow-subagent"}` — sin `description` ni `spawnDepth`. La identidad real de la tarea solo está en el **primer mensaje `user`** (el prompt) de cada archivo.
- **`journal.jsonl`** en el directorio del workflow: líneas `{type:"started"|"result", key, agentId, result}`, una `started` + una `result` por cada agente.
- Es un **fan-out paralelo** (N agentes iguales resolviendo en paralelo y desapareciendo), no una cadena de roles distintos. El directorio `wf_*` es el **contenedor visual natural del "loop"** de un workflow.

---

## 8. Campos útiles para animar

- **`message.usage`** (en cada línea `assistant`): `input_tokens, output_tokens, cache_creation_input_tokens, cache_read_input_tokens, service_tier`. Disponible **turno a turno** → animar el "esfuerzo" de cada agente en tiempo real.
- **`toolUseResult.usage.iterations[]`** del `Agent`: desglose de tokens por iteración del hijo + `totalDurationMs`, `totalToolUseCount` → se puede animar la actividad del hijo **desde el resultado en el padre**, sin tailar el archivo del hijo.
- **`timestamp`** (ISO 8601, cada línea): duración real de cada tool = timestamp del `tool_result` − timestamp del `tool_use`. Más preciso que tiempos relativos.
- **`gitBranch`** (cada línea): contexto de rama si se quiere anotar.

---

## 9. Gotchas y NO CONFIRMADOS

- `type:"progress"` / `parentToolUseID` (camino de subagente inline): **NO CONFIRMADO** en 2.1.209 (no se activa en background ni foreground). Puede existir en otras versiones → parser fail-open.
- `attributionMcpServer` / `attributionMcpTool`: **poco fiables** (desalineados). Usar `tool_use.name`.
- `sourceToolUseID` / `sourceToolAssistantUUID` (en algunas líneas `user`): correlacionan con un `tool_use`, propósito exacto *NO CONFIRMADO*.
- `isMeta: true` (visto una vez en línea `user` tras un `Skill`): posible marcador de contenido inyectado por el sistema, forma exacta *NO CONFIRMADO*.
- Slugs del crew unmassk distintos de Bilbo: *NO CONFIRMADO* (§5).

---

## 10. Implicaciones para nuestro visualizador vs agent-flow

- **agent-flow** (proyecto de referencia, Apache 2.0, en `_study/`) usa un patrón inferior: nombra a los agentes por `description` (frase libre de la tarea) en vez de por su identidad real (`subagent_type` / `attributionAgent`), e ignora `isSidechain`, `parentAgentId`, `spawnDepth`, `usage`, `attributionAgent`. **Nuestro parser debe apoyarse en los campos de identidad reales, no copiar su heurística.**
- Doble fuente de eventos (decisión ya tomada): (a) este JSONL para el movimiento, (b) hooks propios de unmassk para la semántica de dominio.
