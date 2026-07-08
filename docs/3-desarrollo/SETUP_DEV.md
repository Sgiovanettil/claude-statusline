# Setup de desarrollo

## Prerequisitos

| Script | Requisitos |
|--------|------------|
| `statusline.sh` | `jq`, `curl`, `git`, coreutils (`date`, `awk`, `sed`, `shasum`/`sha256sum`) |
| `statusline.ps1` | PowerShell 5.1+ o PowerShell Core 7+; `git` en el `PATH` |

No hay paso de build ni dependencias empaquetadas: los scripts se ejecutan tal cual.

## Instalación

Referenciar el script desde `settings.json` de Claude Code (`~/.claude/settings.json`):

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash ~/.claude/statusline/statusline.sh"
  }
}
```

En Windows, usar `pwsh -NoProfile -File ~/.claude/statusline/statusline.ps1`.

## Variables de entorno

Todas son opcionales; el script tiene defaults razonables.

| Nombre | Descripción | Ejemplo | Requerida |
|--------|-------------|---------|-----------|
| `CLAUDE_CONFIG_DIR` | Directorio de configuración/credenciales de Claude Code | `~/.claude` | No (default `~/.claude`) |
| `CLAUDE_CODE_OAUTH_TOKEN` | Fuerza el token OAuth (evita leer del Keychain/credenciales) | `sk-ant-oat...` | No |
| `CLAUDE_CODE_EFFORT_LEVEL` | Nivel de effort si no viene por stdin ni en `settings.json` | `high` | No |

El nivel de effort también se lee de `effortLevel` en `settings.json` cuando no
llega por stdin ni por variable de entorno.

## Comandos de desarrollo

```bash
# Probar el .sh con un JSON simulado (salida sin ANSI):
echo '{"model":{"display_name":"Opus 4.8"},"context_window":{"context_window_size":200000,"current_usage":{"input_tokens":50000}}}' \
  | bash statusline.sh | sed 's/\x1b\[[0-9;]*m//g'

# Chequeo de sintaxis (no ejecuta):
bash -n statusline.sh
```

## Pruebas

No hay suite de tests automatizada. La validación es:

1. **Sintaxis:** `bash -n statusline.sh`.
2. **Funcional:** alimentar un JSON representativo por stdin y revisar la salida.
   Para ejercitar límites por modelo o `extra_usage`, sembrar la caché de API en
   el temporal (`/tmp/claude/statusline-usage-cache-<hash>.json`) con un `.limits[]`
   de ejemplo antes de correr el script.
3. **Paridad `.sh`/`.ps1`:** cuando se toca el `.ps1` y no hay `pwsh` disponible,
   validar por comparación línea a línea contra el `.sh`; si hay PowerShell,
   correr ambos con el mismo JSON y comparar la salida.
