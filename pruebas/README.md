# Pruebas estáticas — Caso Cuidarte+

Evidencia del análisis estático de seguridad y calidad realizado sobre el código de Cuidarte+
(ISY1102 – Seguridad y Calidad en el Desarrollo de Software, Duoc UC, Evaluación Parcial N°2).

Todas las pruebas son **estáticas**: se analizan el código fuente, la configuración y las
dependencias **sin ejecutar** la aplicación.

## Resumen de resultados

| Herramienta | Tipo | Resultado | Reporte |
|---|---|---|---|
| Semgrep (13 reglas propias) | SAST | 69 hallazgos (18 ERROR, 50 WARNING, 1 INFO) | `semgrep-reglas-propias.txt` / `.json` |
| Semgrep (reglas públicas) | SAST | 1 hallazgo | `semgrep-reglas-publicas.txt` |
| npm audit — backend | SCA | 10 vulnerabilidades (1 crítica, 7 altas, 2 moderadas) | `npm-audit-backend.txt` |
| npm audit — frontend | SCA | 24 vulnerabilidades (17 altas, 6 moderadas, 1 baja) | `npm-audit-frontend.txt` |
| gitleaks | Secretos | 3 secretos (JWT en `.env`, `BACKEND/.env` y `README.md`) | `gitleaks.txt` / `.json` |
| ESLint | Calidad | 16 problemas (13 errores, 3 advertencias) | `eslint-frontend.txt` |

Los resultados de npm audit pueden variar en el tiempo, porque su base de datos de
vulnerabilidades se actualiza continuamente.

## Reglas propias de Semgrep

Las reglas públicas de Semgrep solo encontraron 1 hallazgo, por lo que se diseñaron 13 reglas
a partir de los requerimientos de seguridad del ERS. Están en
[`../semgrep-reglas/cuidarte.yml`](../semgrep-reglas/cuidarte.yml).

| Regla | Severidad | Requerimiento ERS |
|---|---|---|
| plaintext-password-comparison | ERROR | NFR-SEG-2 |
| plaintext-password-insert | ERROR | NFR-SEG-2 |
| jwt-secret-fallback-hardcoded | ERROR | NFR-SEG-9 |
| db-credentials-hardcoded | ERROR | NFR-SEG-8 |
| route-without-auth-check | ERROR | NFR-SEG-4 |
| error-message-leak | WARNING | NFR-SEG-6, NFR-USAB-3 |
| artificial-delay | WARNING | NFR-PERF-1 |
| audit-username-as-id | WARNING | RF-5.1, NFR-SEG-6 |
| token-in-localstorage | WARNING | NFR-SEG-9 |
| cors-wildcard | WARNING | NFR-SEG-5 |
| upload-trusts-client-mimetype | WARNING | RF-4.3, NFR-SEG-5 |
| validation-result-discarded | WARNING | NFR-SEG-4, NFR-SEG-5 |
| missing-helmet | INFO | NFR-SEG-5 |

## Cómo reproducir

Requisitos: Node.js, Python 3 (`pip install semgrep`) y gitleaks (`winget install gitleaks`).
Desde la raíz del repositorio, en PowerShell:

```powershell
$env:PYTHONUTF8=1

# SAST con reglas propias
semgrep scan --config semgrep-reglas/cuidarte.yml BACKEND/src FRONTEND/src

# SAST con reglas públicas (referencia)
semgrep scan --config p/javascript --config p/nodejs --config p/secrets BACKEND/src FRONTEND/src

# Detección de secretos (los reportes de pruebas/ también contienen el secreto y aparecerán como hallazgos extra)
gitleaks dir . -v

# Dependencias
cd BACKEND;  npm install; npm audit
cd ..\FRONTEND; npm install; npm audit

# Calidad de código
npm run lint
```

## Nota

El sistema es **intencionalmente vulnerable** con fines académicos. Las credenciales y el
secreto JWT incluidos en el repositorio son de prueba y no deben usarse en ningún entorno real.
