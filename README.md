# Aqua Assistant AI · Base de datos

Repositorio dueño del esquema de la base de datos de **Aqua Assistant AI**, la aplicación para medir y cuidar el pH del agua. La base de datos es PostgreSQL y está alojada en **Neon**. Este repositorio guarda las migraciones de Flyway y las aplica a Neon con GitHub Actions; ningún servicio crea ni cambia tablas por su cuenta.

| Repositorio | Qué es | Dónde corre |
|---|---|---|
| [FrontendPatSft](https://github.com/WillC372/FrontendPatSft) | La aplicación web | Vercel |
| [BackendPatSft](https://github.com/WillC372/BackendPatSft) | API principal: cuentas, mediciones, lugares, cuidado, alertas, pagos, datos oficiales | Render |
| [AIPatSft](https://github.com/WillC372/AIPatSft) | Servicio de IA: Aqua, diagnóstico, predicción, modelos, informes, fotos, traducciones | Render |
| **DBPatSft** (este) | Esquema y migraciones | Neon (PostgreSQL) |

## Contenido

- `migrations/`: las migraciones en orden (`V1__CreateSchema.sql` … `V26__AlertKind.sql`). V1 a V24 son las mismas, byte por byte, que el backend aplicaba antes de separar la base de datos, así que la base de Neon ya las tiene registradas y no se vuelven a ejecutar; desde V25 se aplican desde este repositorio.
- `docs/Esquema.md`: las 27 tablas, qué servicio escribe cada una y el diagrama entidad-relación.
- `.github/workflows/Migrations.yml`: prueba y despliegue (abajo).
- `docker-compose.yml`: una base local con todo el esquema.

## Despliegue

Cada cambio en `migrations/` pasa por dos pasos:

1. **Probar en una base vacía.** Las migraciones se aplican dos veces a un PostgreSQL 16 nuevo: la primera debe crear todo sin errores y la segunda no debe hacer nada. En un pull request solo corre este paso.
2. **Aplicar en Neon.** En `main`, y solo si la prueba pasó, Flyway valida que las migraciones ya aplicadas no hayan cambiado y aplica las pendientes.

La conexión es el secreto del repositorio **`DATABASE_URL`** (Settings → Secrets and variables → Actions). Su valor es la misma cadena de Neon que usan los servicios en Render: `postgresql://usuario:contraseña@host/neondb?sslmode=require`. El flujo la convierte a la forma JDBC y la oculta en el registro.

También se puede correr a mano desde la pestaña Actions → Migraciones → Run workflow.

## Agregar una migración

1. Crea `migrations/V25__QueCambia.sql`. El número sigue al último y el nombre dice qué cambia, en inglés y en PascalCase, como los anteriores.
2. **Nunca edites una migración que ya está en `main`**: Neon ya la aplicó y Flyway rechazará el despliegue porque cambió su suma de verificación. Para corregir algo, agrega otra migración.
3. Si la migración agrega columnas que usa el código, despliega primero la base de datos y después el servicio.
4. Copia la migración a las pruebas de los servicios, que construyen su base H2 con estas migraciones: `./scripts/sync-migrations.sh ../DBPatSft` en BackendPatSft y en AIPatSft.

## Base local

```bash
docker compose up
```

PostgreSQL queda en `localhost:5432` (base, usuario y contraseña `aquasense`) con todas las migraciones aplicadas. Los servicios se conectan con `DB_URL=jdbc:postgresql://localhost:5432/aquasense`.
