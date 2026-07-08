# AGENTS.md

Contrato de convenciones para agentes que trabajan en **claude-statusline**.
Las convenciones transversales (Git Flow, Conventional Commits, SemVer, releases,
documentación docreview, seguridad, graphify) siguen el estándar del usuario en
`~/dev/proyectos/rules/dev_rules.md`. Este archivo solo agrega lo **específico**
del proyecto. Ante conflicto: (1) instrucción explícita del usuario, (2) este
archivo, (3) `dev_rules.md`, (4) patrones existentes en el código.

## Visión

Statusline de una sola línea para Claude Code, en **Bash** (`statusline.sh`) y
**PowerShell** (`statusline.ps1`), con **paridad de funcionalidad** entre ambas.

## Stack y estructura

- Scripts standalone, sin build ni dependencias empaquetadas.
- `statusline.sh` — Bash (Linux/macOS). Requiere `jq`, `curl`, `git`, coreutils.
- `statusline.ps1` — PowerShell 5.1+ / Core 7+. Requiere `git` en el `PATH`.
- `README.md` · `CHANGELOG.md` — documentación de uso e historial.
- La versión vive en la cabecera de cada script (`# statusline vX.Y.Z`) y en el tag git.

## Comandos clave

```bash
# Probar el .sh con un JSON de entrada simulado (salida sin ANSI):
echo '{"model":{"display_name":"Opus 4.8"},"context_window":{"context_window_size":200000,"current_usage":{"input_tokens":50000}}}' \
  | bash statusline.sh | sed 's/\x1b\[[0-9;]*m//g'

bash -n statusline.sh   # Chequeo de sintaxis (no ejecuta)
```

No hay `pwsh` garantizado en el entorno de desarrollo: cuando se toca el `.ps1`,
validar por **paridad línea a línea** contra el `.sh` y, si hay acceso a
PowerShell, correr ambos con el mismo JSON y comparar la salida.

## Cómo obtiene los datos

1. **stdin** — JSON de Claude Code (modelo, contexto, effort, `rate_limits` 5h/7d). Fuente primaria, sin token.
2. **Endpoint OAuth** `/api/oauth/usage` — para lo que no viene por stdin (límites por modelo, `extra_usage`). Token resuelto multiplataforma.
3. **Caché compartida** en el temporal, con throttling (60 s) y lock anti-estampida.

## Convenciones de código

- Paleta de colores ANSI compartida (alineada al tema oh-my-posh); mismos nombres de color en ambos scripts.
- Segmentos separados por `|` atenuado; orden fijo: `modelo | dir@rama | tokens | effort | 5h | 7d | modelos | extra | vCLI`.
- Formato de reset estandarizado: `@HH:MM` (5h) y `@DD/MM HH:MM` (7d y por modelo), 24 h.
- Color por severidad en límites (`critical`→rojo, `warning`→amarillo, `high`→naranja, resto por %).
- Respetar `CLAUDE_CONFIG_DIR` para rutas de config y credenciales.

## Reglas de negocio críticas

- **Paridad `.sh`/`.ps1` es invariante:** todo cambio funcional en uno se porta al otro en el mismo commit. Un cambio que rompe la paridad es un commit incompleto.
- Nunca mostrar 0% engañoso: si los builtin vienen en cero sin reset válido, caer a la caché (probable fallo de API).
- No commitear secretos: el token OAuth se lee en runtime, nunca se persiste en el repo.

## Qué NO hacer

- No hardcodear rutas absolutas de credenciales ni tokens.
- No agregar dependencias pesadas: el statusline debe correr rápido en cada render.
- No introducir un segmento en un script sin portarlo al otro.
- No reintroducir indicadores obsoletos (p. ej. PEAK/OFF-PEAK, removido tras la eliminación de límites por hora pico en mayo 2026).
