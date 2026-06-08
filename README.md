# kata-customers-bd

Repositorio de migraciones de base de datos para kata-customers, usando Flyway + Neon.

## Estructura

- `migrations/`: scripts SQL versionados.
- `.github/workflows/db-validate.yml`: valida migraciones en PR.
- `.github/workflows/db-migrate-manual.yml`: aplica migraciones por ejecucion manual.

## Objetivo operativo

- No romper lo que ya esta desplegado.
- Registrar cambios de esquema por PR y version.
- Aplicar cambios a Neon de forma controlada.

## Convencion de migraciones

- Formato: `V<version>__<descripcion>.sql`
- Ejemplo: `V2__add_index_to_refresh_tokens.sql`

## Secrets requeridos en GitHub

Configurar en `Settings > Secrets and variables > Actions`:

- `NEON_DEV_JDBC_URL`
- `NEON_DEV_DB_USER`
- `NEON_DEV_DB_PASSWORD`
- `NEON_PROD_JDBC_URL`
- `NEON_PROD_DB_USER`
- `NEON_PROD_DB_PASSWORD`

Notas:

- Para migraciones usar Neon **direct host**.
- Usar SSL (`sslmode=require`) en la URL JDBC.

## Flujo recomendado

1. Crear migracion nueva en `migrations/`.
2. Abrir PR.
3. Workflow `DB Validate` valida orden y consistencia.
4. Al aprobar PR, ejecutar `DB Migrate Manual` y elegir `dev` o `prod`.

## Seguridad para entornos existentes

Los workflows ejecutan Flyway con `baselineOnMigrate=true` para no romper bases ya existentes sin tabla de historial.

## Comandos locales opcionales

Validar:

```bash
docker run --rm \
	-v "$PWD/migrations:/flyway/sql" \
	flyway/flyway:10 \
	-url="<NEON_DEV_JDBC_URL>" \
	-user="<NEON_DEV_DB_USER>" \
	-password="<NEON_DEV_DB_PASSWORD>" \
	-locations="filesystem:/flyway/sql" \
	-baselineOnMigrate=true \
	validate
```

Migrar:

```bash
docker run --rm \
	-v "$PWD/migrations:/flyway/sql" \
	flyway/flyway:10 \
	-url="<NEON_PROD_JDBC_URL>" \
	-user="<NEON_PROD_DB_USER>" \
	-password="<NEON_PROD_DB_PASSWORD>" \
	-locations="filesystem:/flyway/sql" \
	-baselineOnMigrate=true \
	migrate
```
