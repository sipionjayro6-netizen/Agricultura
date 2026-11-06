# Cómo contribuir

1. Cree un *issue* con la tarea (plantilla en `.github/ISSUE_TEMPLATE`).
2. Parta de `develop`: `git switch develop && git pull && git switch -c feature/<tema>`.
3. Trabaje con commits pequeños siguiendo la convención `tipo(ámbito): descripción`
   (`feat`, `fix`, `test`, `docs`, `refactor`, `chore`).
4. En cambios de lógica, escriba primero la prueba (TDD) y verifique con `mvn test`.
5. Suba la rama y abra un *Pull Request* hacia `develop`; otro integrante lo revisa.
6. El responsable de control de versiones fusiona cuando la integración continua está en verde.
7. Las entregas se fusionan de `develop` a `main` y se marcan con una etiqueta (`v0.3.1`, `v0.4.0`...).

Guía completa: `docs/GITHUB.md`.
