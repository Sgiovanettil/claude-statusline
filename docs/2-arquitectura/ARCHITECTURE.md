# Arquitectura

## Visión general

`claude-statusline` es un script standalone que Claude Code invoca en cada render
de la statusline. Recibe un JSON por `stdin`, consulta fuentes de datos
complementarias y emite una sola línea con secuencias ANSI. Existe en dos
implementaciones paralelas con **paridad de funcionalidad**: `statusline.sh`
(Bash) y `statusline.ps1` (PowerShell).

## Componentes

| Componente | Responsabilidad |
|------------|-----------------|
| `statusline.sh` | Implementación Bash (Linux/macOS). Parseo con `jq`, red con `curl`. |
| `statusline.ps1` | Implementación PowerShell (Windows + Core). Parseo con `ConvertFrom-Json`, red con `Invoke-RestMethod`. |
| Caché de uso | Archivo JSON en el temporal, compartido entre instancias, con throttling y lock anti-estampida. |
| Caché de versión CLI | Archivo con la versión de `claude --version`, TTL 1 h. |

Ambos scripts replican la misma secuencia de segmentos:
`modelo | dir@rama (+/-) | tokens (%) | effort | 5h @reset | 7d @reset | <modelos> | extra | vCLI`.

## Fuentes de datos y flujo

1. **stdin (primaria).** Claude Code entrega un JSON con `model`, `context_window`,
   `effort` y `rate_limits` (`five_hour` / `seven_day` con `used_percentage` y
   `resets_at` en epoch). No requiere token. Es la fuente de los límites 5h/7d.
2. **Endpoint OAuth `/api/oauth/usage` (complementaria).** Aporta lo que no viene
   por stdin: límites por modelo (`.limits[]`, p. ej. Fable/Opus/Sonnet) y
   `extra_usage`. Requiere token OAuth.
3. **Caché compartida.** Guarda la última respuesta de la API. Se usa como fuente
   de los segmentos por modelo/extra en la ruta builtin, y como *fallback* cuando
   la API falla o los builtin llegan en cero sin `resets_at` válido.

Selección de ruta (`effective_builtin`): se confía en `rate_limits` de stdin si
algún porcentaje es distinto de cero **o** si hay `resets_at` válido (un cero
genuino tras reset trae timestamp). Si no, se cae a la caché para no mostrar 0%
engañoso.

## Dependencias externas

- **Bash:** `jq`, `curl`, `git`, coreutils (`date`, `awk`, `sed`, `shasum`/`sha256sum`).
- **PowerShell:** `git` en el `PATH`; `Invoke-RestMethod` (nativo).
- **API:** `https://api.anthropic.com/api/oauth/usage` (header `anthropic-beta: oauth-2025-04-20`).

## Resolución de token OAuth (multiplataforma)

Orden de intento hasta encontrar `claudeAiOauth.accessToken`:

1. Variable de entorno `CLAUDE_CODE_OAUTH_TOKEN`.
2. macOS Keychain (`security`, servicio `Claude Code-credentials[-<hash>]`) — solo `.sh`.
3. Windows Credential Manager / `%LOCALAPPDATA%\Claude Code\credentials.json` — solo `.ps1`.
4. Archivo `${CLAUDE_CONFIG_DIR}/.credentials.json`.
5. GNOME Keyring vía `secret-tool` — solo `.sh`.

Las diferencias entre scripts en este punto son **específicas de plataforma** y no
rompen la paridad funcional.

## Límites del sistema

- Sin estado persistente propio salvo las cachés en el directorio temporal.
- El token nunca se persiste en el repo: se lee en runtime desde el sistema.
- Respeta `CLAUDE_CONFIG_DIR` para resolver config y credenciales.
