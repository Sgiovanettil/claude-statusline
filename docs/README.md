# Documentación — claude-statusline

Índice navegable de la documentación del proyecto. Puerta de entrada única.

**Nivel de madurez:** N1 · **Última auditoría:** 2026-07-08

| # | Documento | Descripción | Estado |
|---|-----------|-------------|--------|
| — | [`../README.md`](../README.md) | Qué es, instalación, segmentos, requisitos | ✅ al día |
| — | [`../CHANGELOG.md`](../CHANGELOG.md) | Historial de cambios (Keep a Changelog) | ✅ al día |
| — | [`../AGENTS.md`](../AGENTS.md) | Contrato de convenciones para agentes | ✅ al día |
| 2 | [`2-arquitectura/ARCHITECTURE.md`](2-arquitectura/ARCHITECTURE.md) | Componentes, fuentes de datos y flujo | ✅ al día |
| 2 | [`2-arquitectura/DECISIONS.md`](2-arquitectura/DECISIONS.md) | Decisiones técnicas (ADRs) | ✅ al día |
| 2 | [`2-arquitectura/DIAGRAMAS.md`](2-arquitectura/DIAGRAMAS.md) | Diagrama del flujo de datos (Mermaid) | ✅ al día |
| 3 | [`3-desarrollo/SETUP_DEV.md`](3-desarrollo/SETUP_DEV.md) | Requisitos, instalación, variables, pruebas | ✅ al día |

## Archivos que NO aplican (sin evidencia)

- `PRD.md` — utilidad personal sin lógica de negocio de producto documentable.
- `ROADMAP.md` — sin plan futuro ni hitos declarados.
- `API.md` / `BD_DICCIONARIO.md` — no expone API propia ni usa base de datos.
- `5-operacion/*` — sin despliegue ni operación en producción.

## Evidencia débil (no creados; reevaluar si crecen)

- `DEFINICIONES.md` — glosario de términos (`effort`, `rate_limits`, `extra_usage`, `builtin`); crear si se busca onboarding.
- `KNOWN_ISSUES.md` — limitación registrable: el `.ps1` se valida por paridad estructural cuando no hay `pwsh` en el entorno.
