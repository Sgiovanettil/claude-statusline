# Decisiones de arquitectura (ADRs)

Registro de decisiones técnicas no obvias. Cada entrada: contexto · decisión ·
alternativas · consecuencias.

## 0001 — Paridad estricta entre `statusline.sh` y `statusline.ps1`

- **Contexto:** el statusline debe funcionar tanto en Linux/macOS (Bash) como en
  Windows (PowerShell).
- **Decisión:** mantener dos implementaciones con **paridad de funcionalidad**;
  todo cambio funcional en una se porta a la otra en el mismo commit.
- **Alternativas:** un solo script multiplataforma (descartado: `jq`/`curl` no son
  nativos en Windows y PowerShell no corre en el flujo Bash); reescribir en un
  lenguaje compilado (descartado: sobreingeniería para un statusline).
- **Consecuencias:** un cambio que rompe la paridad es un commit incompleto. Las
  únicas diferencias legítimas son específicas de plataforma (resolución de token,
  zona horaria, formato de fecha).

## 0002 — `rate_limits` de stdin como fuente primaria de límites 5h/7d

- **Contexto:** los límites de uso pueden obtenerse del JSON de stdin o del
  endpoint OAuth.
- **Decisión:** usar `rate_limits` de stdin como fuente primaria (sin token ni red)
  y recurrir al endpoint OAuth solo para lo que stdin no expone (límites por modelo
  y `extra_usage`).
- **Alternativas:** consultar siempre la API (descartado: más lento, requiere token,
  gatilla rate limits).
- **Consecuencias:** el render base funciona sin credenciales. Se agrega la heurística
  `effective_builtin` para no mostrar 0% engañoso ante un fallo de API (se exige
  porcentaje ≠ 0 o `resets_at` válido; si no, se cae a la caché).

## 0003 — Caché compartida con throttling y lock anti-estampida

- **Contexto:** varios paneles de Claude Code pueden renderizar en paralelo y
  gatillar múltiples llamadas simultáneas al endpoint de uso.
- **Decisión:** caché JSON compartida en el temporal, clave por hash de
  `CLAUDE_CONFIG_DIR`, con throttling de 60 s y *lock* anti-estampida (`touch` del
  archivo antes de la llamada).
- **Alternativas:** sin caché (descartado: rate limits); caché por proceso
  (descartado: no comparte entre paneles).
- **Consecuencias:** si el fetch falla y deja el archivo vacío, se elimina el
  centinela para no suprimir reintentos durante toda la ventana.

## 0004 — Remoción del indicador PEAK/OFF-PEAK

- **Contexto:** el statusline mostraba un indicador de ventana de hora pico
  (weekdays 05:00–11:00 PT).
- **Decisión:** eliminar el indicador de ambos scripts.
- **Alternativas:** mantenerlo (descartado).
- **Consecuencias:** los límites por hora pico de Claude Code Pro/Max fueron
  removidos en mayo 2026 (fuente: claude-peak-time.pages.dev), por lo que la ventana
  ya no reflejaba ningún efecto real. No debe reintroducirse.

## 0005 — Versión en la cabecera del script como manifiesto ligero

- **Contexto:** el proyecto no tiene manifiesto (`package.json`, etc.) donde fijar
  la versión que exige SemVer.
- **Decisión:** declarar la versión en la cabecera de cada script
  (`# statusline vX.Y.Z`), alineada con el tag git y el `CHANGELOG.md`.
- **Alternativas:** archivo `VERSION` (descartado: innecesario para dos scripts).
- **Consecuencias:** el bump de versión toca la cabecera de ambos scripts y debe
  mantenerse en paridad.
