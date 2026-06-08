# kata-customers-bd

Guia de versionado de scripts SQL y promocion de cambios por ambientes con Flyway.

## 1. Como se versionan los scripts

Todos los cambios de base de datos se registran en la carpeta `migrations/` con archivos SQL versionados.

### Regla de nombre

- Formato obligatorio: `V<numero>__<descripcion>.sql`
- Ejemplos:
  - `V1__initial_schema.sql`
  - `V2__add_index_products_customer_id.sql`
  - `V3__add_revoked_default.sql`

### Reglas de versionado

- Nunca modificar un script ya aplicado en un ambiente.
- Cada cambio nuevo va en una nueva version (V2, V3, V4...).
- Los scripts deben ser idempotentes cuando aplique (`IF NOT EXISTS`) para reducir riesgo.
- Un script debe representar un cambio pequeno, claro y auditable.

## 2. Flujo por ambientes hasta despliegue

### Paso 1: Desarrollo del cambio

1. Crear un nuevo script en `migrations/`.
2. Commit en rama de trabajo.
3. Abrir Pull Request.

### Paso 2: Validacion en CI

1. El workflow `DB Validate` se ejecuta en el PR.
2. Si falla, corregir el script y actualizar el PR.
3. Si pasa, el cambio esta listo para promocion.

### Paso 3: Promocion a DEV

1. Aprobado el PR, ejecutar workflow `DB Migrate Manual` con `target=dev`.
2. Confirmar en Neon DEV que la migracion quedo registrada en `flyway_schema_history`.

### Paso 4: Promocion a PROD

1. Solo despues de validar en DEV, ejecutar `DB Migrate Manual` con `target=prod`.
2. Confirmar en Neon PROD `flyway_schema_history` con estado exitoso.

### Paso 5: Despliegue de aplicacion

1. Primero migrar BD.
2. Despues desplegar backend.
3. Mantener esta secuencia para evitar incompatibilidades.
