# claude-statusline

Statusline personalizada para [Claude Code](https://claude.com/claude-code), disponible en **Bash** (`statusline.sh`) y **PowerShell** (`statusline.ps1`) con **paridad de funcionalidad** entre ambas.

Renderiza, en una sola línea, el estado de la sesión actual:

```
Opus 4.8 | statusline@develop (+47 -34) | 60k/200k (30%) | effort: high | 5h 12% @16:00 | 7d 40% @12/07 16:00 | Fable 88% @09/07 11:00 | Opus 25% @12/07 16:00 | v2.1.204
```

## Segmentos

| Segmento | Descripción |
|----------|-------------|
| Modelo | Nombre del modelo activo (normaliza `(1M context)` → `1M`). |
| `dir@rama (+N -M)` | Directorio de trabajo, rama git y líneas agregadas/eliminadas sin commitear. |
| `tokens (%)` | Tokens usados / tamaño de ventana de contexto y porcentaje ocupado. |
| `effort` | Nivel de esfuerzo del modelo, con color por nivel. |
| `5h` / `7d` | Límites de uso de 5 horas y 7 días con porcentaje, color por severidad y hora de reset (`@HH:MM` y `@DD/MM HH:MM`). |
| `<modelo>` | Límites semanales por modelo (Fable/Opus/Sonnet), mostrados solo cuando están activos. |
| `extra` | Uso extra (créditos usados / límite mensual) cuando está habilitado. |
| `vX.Y.Z` | Versión del CLI de Claude Code (cacheada, TTL 1h). |

## Instalación

Referencia el script desde tu `settings.json` de Claude Code (`~/.claude/settings.json`):

**Linux / macOS (Bash):**

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash ~/.claude/statusline/statusline.sh"
  }
}
```

**Windows (PowerShell):**

```json
{
  "statusLine": {
    "type": "command",
    "command": "pwsh -NoProfile -File ~/.claude/statusline/statusline.ps1"
  }
}
```

## Requisitos

- **Bash**: `jq`, `curl`, `git` y coreutils (`date`, `awk`, `sed`).
- **PowerShell**: PowerShell 5.1+ o PowerShell Core 7+; `git` en el `PATH`.

## Cómo obtiene los datos

1. **Entrada por stdin**: Claude Code entrega un JSON con modelo, contexto, effort y `rate_limits` (5h/7d). Es la fuente primaria y no requiere token.
2. **Endpoint de uso OAuth** (`/api/oauth/usage`): se consulta para datos que no vienen por stdin (límites por modelo y `extra_usage`). El token se resuelve de forma multiplataforma (variable de entorno → macOS Keychain / Windows Credential Manager → archivo de credenciales → GNOME Keyring).
3. **Caché compartida** en el directorio temporal, con throttling (60 s) y lock anti-estampida para no gatillar rate limits cuando hay varios paneles abiertos.

Respeta `CLAUDE_CONFIG_DIR` para resolver rutas de configuración y credenciales.
