# AgroConecta — Sistema de Gestión de Pedidos e Inventario (Avance 3)

Proyecto Integrador · Curso: Gestión de Proyectos de Tecnologías de Información.
Solución en **Java 17** con arquitectura **MVC + DAO**, principios **SOLID**, pruebas **TDD (JUnit 5)**, seguridad aplicada
y las librerías **Google Guava, Apache Commons, Logback y Apache POI**.

## Requisitos

- JDK 17 o superior
- Maven 3.9 o superior
- Visual Studio Code con la extensión *Extension Pack for Java* (se recomienda al abrir la carpeta)

## Cómo ejecutarlo

```bash
mvn test                  # ejecuta las pruebas (unitarias, de base de datos y de la API)
mvn exec:java             # inicia el sistema en http://127.0.0.1:8080/
mvn javadoc:javadoc       # regenera la documentación del código
```

En VS Code también puede abrir `App.java` y pulsar **Run** (o usar la configuración *AgroConecta (App)* de `.vscode/launch.json`).

### Cuentas de demostración

| Usuario | Rol | Contraseña inicial |
|---|---|---|
| `admin` | Personal AgroConecta | `Admin#2026` |
| `jquispe` | Agricultor | `Agro#2026` |
| `mrojas` | Agricultor | `Agro#2026` |

Son contraseñas **de demostración**. Para cambiarlas antes de iniciar, defina las variables de entorno
`AGRO_ADMIN_PASSWORD` y `AGRO_DEMO_PASSWORD`. Otras variables opcionales: `AGRO_PORT` (8080), `AGRO_HOST` (127.0.0.1),
`AGRO_JDBC_URL` (por defecto H2 en memoria; use `jdbc:h2:file:./data/agroconecta` para conservar los datos).

## Arquitectura

```
Vista (public/index.html, css, js)  ──HTTP/JSON──►  Controlador (controller/*)  ──►  Servicio (service/*)  ──►  DAO (dao/*) ──► H2
                                                         │                              │
                                                  seguridad (security/*)       modelo (model/*) + Guava/Commons/Logback/POI
```

| Paquete | Responsabilidad |
|---|---|
| `model` | Entidades del dominio (records inmutables) |
| `dao` | Interfaces de acceso a datos (`ProductoDao`, `StockDao`, `PedidoDao`...) y `Transaccion` |
| `dao.jdbc` | Implementación JDBC con consultas parametrizadas |
| `service` | Reglas de negocio (pedidos, inventario, catálogo, reportes, autenticación) |
| `service.export` | Exportadores de reportes (CSV y Excel con Apache POI) |
| `security` | Hash de contraseñas PBKDF2 y control de acceso por roles |
| `controller` | Endpoints HTTP, cabeceras de seguridad y manejo de errores |
| `config` | Raíz de composición (inyección de dependencias manual) y datos iniciales |

## API

| Método y ruta | Rol | Descripción |
|---|---|---|
| `POST /api/auth/login` | público | Inicia sesión y devuelve un token |
| `POST /api/auth/logout` | sesión | Cierra la sesión |
| `GET /api/productos`, `GET /api/almacenes` | sesión | Catálogo con stock por almacén |
| `GET /api/pedidos` | sesión | Admin: todos · Agricultor: solo los suyos |
| `POST /api/pedidos` | agricultor | Registra un pedido y descuenta stock |
| `PATCH /api/pedidos/{id}/estado` | admin | Cambia el estado (no retrocede) |
| `PATCH /api/stock/{producto}/{almacen}` | admin | Ajusta el inventario |
| `GET /api/reportes/resumen` | admin | Indicadores gerenciales |
| `GET /api/reportes/exportar?formato=csv\|xlsx` | admin | Descarga el reporte |

## Seguridad aplicada

- Contraseñas con **PBKDF2-HMAC-SHA256**, sal aleatoria e iteraciones altas; comparación en tiempo constante.
- Sesiones con **token aleatorio de 256 bits** que expira por inactividad; bloqueo temporal tras 5 intentos fallidos.
- **Control de acceso por roles** en cada endpoint y mensajes de error que no revelan datos internos.
- **SQL parametrizado** (sin concatenar texto) y restricciones `CHECK`/`FOREIGN KEY` en la base de datos.
- **Validación de entradas** (rangos, tamaños, límite de 64 KB por solicitud).
- Cabeceras `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Cache-Control: no-store`.
- Interfaz con **escape de HTML** en todo texto dinámico (anti-XSS) y token en `sessionStorage`.
- Servidor de archivos estáticos con **lista blanca** (anti recorrido de directorios).
- Exportaciones protegidas contra **inyección de fórmulas** en CSV/Excel.
- **Auditoría** en `logs/auditoria.log` (accesos, pedidos, cambios de stock y de estado).

## Limitaciones conocidas

- La notificación por WhatsApp (RF04) está **simulada** en el log (`NotificadorLog`); falta integrar la API real.
- El servidor escucha en HTTP; en producción debe ir detrás de un proxy con HTTPS.
- La base de datos H2 en memoria se reinicia en cada arranque (use `AGRO_JDBC_URL` para persistir).
- Queda pendiente: pagos en línea, facturación electrónica y la app móvil para agentes de campo (fuera del alcance de este avance).

## Control de versiones

- Detalle completo (responsable, roles, ramas, commits y reglas): `docs/CONTROL_DE_VERSIONES.md`.
- Guía de comandos y publicación en GitHub: `docs/GITHUB.md`.
- Roles del equipo: `docs/EQUIPO.md`.
