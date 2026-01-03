# Documentación de Workflows

## `build-checklists.yml`

Para construir checklists y Crear un PR con cambios hechos en el master.

- Trigger: Push, Solo cuando archivos dentro del directorio document son cambiados. Manual (`workflow_dispatch`), GitHub web UI.
- Ver: `/.github/xlsx/` en la raíz del repositorio para construcción XLSX.

## `build-ebooks.yml`

Para construir PDF y EPUB e-Books en release.

- Trigger: Tag aplicado al repositorio. Manual (`workflow_dispatch`), GitHub web UI.
- Ver: `/.github/pdf/` en la raíz del repositorio para configuraciones específicas de construcción PDF.
- Ver: `/.github/epub/` en la raíz del repositorio para configuraciones específicas de construcción EPUB.

## `comment.yml`

Activado por la finalización de otros workflows para comentar resultados de lint u otros en PRs.
Los workflows que lo aprovechan deberían crear un archivo de texto `pr_number` y `artifact.txt` con el contenido a ser comentado, que son adjuntados a sus ejecuciones de workflow como `artifact`.

- Trigger: Otros workflows `workflow_run`.

## `dummy.yml`

Acción de utilidad para que PRs sin archivos Markdown puedan pasar las reglas de protección de rama. `lint` es requerido por las reglas de protección de rama. Si un PR no contiene archivos Markdown (ej. solo una imagen o YAML que no es linted) el dummy corre y pasa el requerimiento de protección de rama.

- Trigger: Pull Requests.

## `md-link-check.yml`

Verifica Pull Requests por enlaces rotos.

- Trigger: Pull Requests.
- Archivo de Config: `markdown-link-check-config.json`

## `md-lint-check.yml`

Verifica archivos Markdown y marca problemas de estilo o sintaxis.

- Trigger: Pull Requests.
- Archivo de Config: `.markdownlint.json`

## `md-textlint-check.yml`

Verifica archivos Markdown por problemas de ortografía, estilo y typos.

- Trigger: Pull Requests.
- Archivo de Config: `.textlintrc`

## `www_latest_update.yml`

Publica el contenido web más reciente usando la cuenta @wstgbot a `OWASP/www-project-web-security-testing-guide`.

- Trigger: Push.
- Ver: `/.github/www/latest/` en la raíz del repositorio.

## `www_stable_update.yml`

Publica contenido web estable y versionado usando la cuenta @wstgbot a `OWASP/www-project-web-security-testing-guide`.

- Trigger: Tag aplicado al repositorio (formato `v*`).
- Ver: `/.github/www/` en la raíz del repositorio.