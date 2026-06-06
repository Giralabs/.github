# Guía de Contribución y Flujo de Trabajo con Git/GitHub

Documento de referencia para todos los repositorios de la organización. Define cómo crear ramas, escribir commits, abrir Pull Requests y mantener un historial limpio y profesional. El objetivo es que cualquier persona pueda leer el historial del proyecto y entender qué cambió, por qué y cuándo, sin necesidad de preguntar.

## Índice

- [Principios generales](#principios-generales)
- [Estrategia de ramas](#estrategia-de-ramas)
- [Nomenclatura de ramas](#nomenclatura-de-ramas)
- [Flujo de trabajo diario](#flujo-de-trabajo-diario)
- [Convención de commits](#convención-de-commits)
- [Pull Requests](#pull-requests)
- [Revisión de código](#revisión-de-código)
- [Resolución de conflictos](#resolución-de-conflictos)
- [Versionado y etiquetas](#versionado-y-etiquetas)
- [Buenas prácticas](#buenas-prácticas)
- [Qué evitar](#qué-evitar)

## Principios generales

- **Nunca trabajes directamente sobre `main`.** Todo cambio entra a través de una rama y una Pull Request.
- **Commits pequeños y atómicos.** Cada commit representa un cambio coherente y completo. Si un commit necesita la conjunción "y" para describirse, probablemente deberían ser dos.
- **El historial cuenta una historia.** Debe poder leerse de arriba a abajo y entenderse la evolución del proyecto.
- **Nada de secretos en el repositorio.** Claves, tokens, contraseñas y archivos `.env` jamás se suben. Para eso está `.gitignore`.
- **Sincroniza antes de empezar.** Cada jornada se arranca actualizando la rama base para reducir conflictos.

## Estrategia de ramas

Usamos un modelo basado en `main` con ramas de corta duración. Es simple, escalable y suficiente para el ritmo de la organización.

### Ramas permanentes

- **`main`** — Rama estable y desplegable en todo momento. Refleja lo que está (o puede estar) en producción. Protegida: no admite push directo.
- **`develop`** *(opcional, solo en repos con releases agrupadas)* — Rama de integración donde se acumulan funcionalidades antes de un release. Si un repositorio despliega de forma continua, puede prescindirse de ella y trabajar directamente contra `main`.

### Ramas temporales

Se crean a partir de la rama base, viven mientras dura el trabajo y se eliminan tras fusionarse.

- **`feature/`** — Nueva funcionalidad.
- **`fix/`** — Corrección de un error en desarrollo.
- **`hotfix/`** — Corrección urgente que sale directamente desde `main` a producción.
- **`refactor/`** — Reestructuración de código sin cambiar comportamiento.
- **`docs/`** — Cambios exclusivamente de documentación.
- **`chore/`** — Tareas de mantenimiento (dependencias, configuración, tooling).

## Nomenclatura de ramas

Formato: `tipo/descripcion-corta-en-kebab-case`. Opcionalmente con referencia al issue.

```
feature/login-con-google
fix/error-validacion-formulario
hotfix/caida-conexion-mongodb
refactor/servicio-usuarios
docs/actualizar-readme-instalacion
chore/actualizar-dependencias
```

Con número de issue:

```
feature/142-carrusel-colaboradores
fix/87-token-expirado
```

Reglas: todo en minúsculas, palabras separadas por guiones, sin tildes ni caracteres especiales, descripción breve pero significativa.

## Flujo de trabajo diario

1. **Sitúate en la rama base y actualízala.**
   ```bash
   git checkout main
   git pull origin main
   ```

2. **Crea tu rama de trabajo.**
   ```bash
   git checkout -b feature/nombre-descriptivo
   ```

3. **Trabaja y haz commits frecuentes y atómicos.**
   ```bash
   git add archivo-concreto.ts
   git commit -m "feat: añade validación de email en el registro"
   ```
   Prefiere `git add` por archivo o por bloques (`git add -p`) frente a `git add .`, para no incluir cambios accidentales.

4. **Mantén la rama al día con la base** (especialmente en ramas que viven varios días).
   ```bash
   git fetch origin
   git rebase origin/main
   ```

5. **Sube tu rama al remoto.**
   ```bash
   git push -u origin feature/nombre-descriptivo
   ```

6. **Abre una Pull Request** hacia la rama base.

7. **Tras la aprobación y el merge, limpia.**
   ```bash
   git checkout main
   git pull origin main
   git branch -d feature/nombre-descriptivo
   ```

## Convención de commits

Seguimos [Conventional Commits](https://www.conventionalcommits.org/). Es un estándar que permite generar changelogs automáticos, deducir el versionado y leer el historial con claridad.

### Estructura

```
<tipo>(<ámbito opcional>): <descripción>

[cuerpo opcional]

[footer opcional]
```

### Tipos

| Tipo | Uso |
|------|-----|
| `feat` | Nueva funcionalidad para el usuario |
| `fix` | Corrección de un error |
| `docs` | Cambios solo en documentación |
| `style` | Formato, espacios, comas; sin cambios de lógica |
| `refactor` | Reestructuración sin cambiar comportamiento ni corregir errores |
| `perf` | Mejora de rendimiento |
| `test` | Añadir o corregir tests |
| `build` | Cambios en el sistema de build o dependencias |
| `ci` | Cambios en la configuración de integración continua |
| `chore` | Tareas de mantenimiento que no afectan al código de producción |
| `revert` | Revierte un commit anterior |

### Ámbito (scope)

Opcional, indica la zona del código afectada. Útil en monorepos o proyectos con módulos claros.

```
feat(auth): añade refresh token
fix(api-v2): corrige paginación de usuarios
```

### Reglas de escritura

- La descripción va en **minúscula**, en **modo imperativo** y **sin punto final**: "añade", no "añadido" ni "añade.".
- Máximo recomendado de 50 caracteres en la primera línea; el detalle va en el cuerpo.
- El cuerpo explica el **qué** y el **por qué**, no el cómo (el cómo ya está en el código).
- Los cambios que rompen compatibilidad se marcan con `!` y se explican en el footer con `BREAKING CHANGE:`.

### Ejemplos

Commit simple:

```
feat: añade carrusel infinito de colaboradores
```

Commit con cuerpo:

```
fix: corrige expiración prematura del token JWT

El cálculo de expiración usaba segundos en lugar de milisegundos,
provocando cierres de sesión a los pocos minutos. Se unifica la
unidad de tiempo en la configuración.

Closes #87
```

Cambio que rompe compatibilidad:

```
feat(api)!: elimina el endpoint /api/v1/users

BREAKING CHANGE: los clientes deben migrar a /api/v2/users.
La versión v1 queda retirada tras el periodo de deprecación.
```

## Pull Requests

La Pull Request (PR) es el punto de control de calidad antes de integrar código. No es un trámite: es donde se revisa, se discute y se deja constancia del porqué de los cambios.

### Antes de abrir una PR

- La rama está actualizada con la base (`rebase` o `merge` reciente).
- El código compila y pasa el linter.
- Los tests pasan en local.
- No quedan `console.log`, código comentado ni archivos temporales.
- No se han colado secretos ni archivos que deberían estar en `.gitignore`.

### Cómo redactar la PR

- **Título** claro siguiendo el mismo estilo que los commits: `feat: integra pasarela de pago con Stripe`.
- **Descripción** que responda: qué se hace, por qué, y cómo probarlo.
- **Referencia al issue** correspondiente (`Closes #142`) para que se cierre automáticamente al fusionar.
- **PRs pequeñas.** Una PR enorme es difícil de revisar bien. Divide el trabajo cuando sea posible.

### Plantilla sugerida

```markdown
## Descripción
Breve resumen de qué resuelve esta PR.

## Tipo de cambio
- [ ] Nueva funcionalidad (feat)
- [ ] Corrección de error (fix)
- [ ] Refactor
- [ ] Documentación

## Cómo probarlo
Pasos para verificar el cambio.

## Checklist
- [ ] El código pasa el linter y compila
- [ ] Se han añadido o actualizado tests
- [ ] Se ha actualizado la documentación si procede
- [ ] No se incluyen secretos ni archivos sensibles

Closes #
```

## Revisión de código

- Toda PR hacia `main` requiere al menos **una aprobación** antes de fusionarse (configurable en las reglas de protección de rama).
- **Quien revisa** comenta con respeto y concreción, sugiere en lugar de imponer, y aprueba solo lo que entiende.
- **Quien recibe** la revisión no la toma como algo personal: el objetivo es la calidad del producto, no juzgar a la persona.
- Resuelve todos los comentarios antes de fusionar. Si discrepas, discútelo en el hilo en vez de ignorarlo.

### Estrategia de merge

- **Squash and merge** (recomendado por defecto): condensa todos los commits de la rama en uno solo sobre `main`. Mantiene el historial de `main` limpio y legible.
- **Rebase and merge**: cuando se quiere conservar los commits individuales bien formados.
- **Evita el merge commit estándar** salvo en integraciones de ramas largas (por ejemplo, `develop` hacia `main`).

Tras fusionar, **elimina la rama** desde GitHub.

## Resolución de conflictos

Los conflictos son normales; lo importante es resolverlos con cuidado.

1. Actualiza tu rama con la base:
   ```bash
   git fetch origin
   git rebase origin/main
   ```
2. Git señalará los archivos en conflicto. Edítalos resolviendo manualmente, conservando lo correcto de ambos lados.
3. Marca como resueltos y continúa:
   ```bash
   git add archivo-resuelto.ts
   git rebase --continue
   ```
4. Si te equivocas o quieres abortar:
   ```bash
   git rebase --abort
   ```
5. Tras un rebase ya publicado, sube con `--force-with-lease` (más seguro que `--force`):
   ```bash
   git push --force-with-lease
   ```

Nunca uses `--force` a secas sobre ramas compartidas: puedes borrar el trabajo de otra persona.

## Versionado y etiquetas

Usamos [Versionado Semántico](https://semver.org/lang/es/): `MAJOR.MINOR.PATCH`.

- **MAJOR** — Cambios incompatibles con versiones anteriores (`BREAKING CHANGE`).
- **MINOR** — Nueva funcionalidad compatible hacia atrás (`feat`).
- **PATCH** — Correcciones compatibles (`fix`).

Las versiones se marcan con tags anotados sobre `main`:

```bash
git tag -a v1.4.0 -m "Release 1.4.0: integración de pagos"
git push origin v1.4.0
```

Cada release debe tener su entrada en el changelog o en las Releases de GitHub.

## Buenas prácticas

- **Commits atómicos y frecuentes.** Es más fácil revertir o entender cambios pequeños.
- **Mensajes en imperativo y en el mismo idioma en todo el repositorio** (define uno y mantenlo).
- **Sincroniza a menudo** para evitar conflictos grandes y dolorosos.
- **`.gitignore` desde el día uno.** Incluye `node_modules/`, `dist/`, `.env`, logs y archivos de IDE.
- **No subas archivos generados** (compilados, dependencias, builds). Se reconstruyen, no se versionan.
- **Un issue, una rama, una PR.** Mantiene la trazabilidad clara.
- **Borra las ramas fusionadas.** Un listado de ramas limpio refleja un proyecto cuidado.
- **Escribe en la PR el contexto que no cabe en el código.** Las decisiones futuras se agradecen.

## Qué evitar

- Hacer push directo a `main` o `develop`.
- Commits del tipo `cambios`, `arreglo`, `wip`, `.` o `asdf`.
- Acumular semanas de trabajo en un único commit gigante.
- Subir `.env`, claves, tokens o credenciales de cualquier tipo.
- Mezclar varios cambios sin relación en una sola PR.
- Forzar el historial (`--force`) sobre ramas que otros están usando.
- Fusionar tu propia PR sin revisión cuando el repositorio exige aprobación.
- Dejar comentarios de revisión sin resolver antes del merge.
