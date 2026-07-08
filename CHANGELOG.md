# Changelog

Todos los cambios notables de este proyecto se documentan en este archivo.

El formato se basa en [Keep a Changelog](https://keepachangelog.com/es/1.1.0/),
y este proyecto adhiere a [Versionado Semántico](https://semver.org/lang/es/).

## [1.0.0] - 2026-07-08

### Added

- Statusline para Claude Code en Bash (`statusline.sh`) y PowerShell (`statusline.ps1`) con paridad de funcionalidad entre ambas.
- Segmentos de una sola línea: modelo, directorio + rama git + diff stats, uso de tokens y % de contexto, y nivel de effort con color por nivel.
- Límites de uso 5h y 7d con porcentaje, color por severidad y hora de reset en formato estandarizado (`@HH:MM` y `@DD/MM HH:MM`, 24 h).
- Límites semanales por modelo (Fable/Opus/Sonnet) desde el endpoint de uso OAuth, mostrados solo cuando están activos, con color por severidad.
- Segmento `extra_usage` (créditos usados / límite mensual) cuando está habilitado.
- Versión del CLI de Claude Code cacheada con TTL de 1 hora.
- Resolución de token OAuth multiplataforma (variable de entorno, macOS Keychain, archivo de credenciales, GNOME Keyring; Windows Credential Manager en PowerShell).
- Caché de uso compartida entre instancias con throttling (60 s) y lock anti-estampida.
- `README.md` con instalación, segmentos y requisitos.

[1.0.0]: https://github.com/Sgiovanettil/claude-statusline/releases/tag/v1.0.0
