# Guía de Migración de Supabase con pg_dump y psql

Esta guía te ayudará a migrar completamente una base de datos de un cluster de Supabase a otro usando las herramientas nativas de PostgreSQL.

## 📋 Prerrequisitos

- **PostgreSQL client tools** instalados en tu máquina
- **Credenciales de conexión** de ambos clusters (origen y destino)
- **Permisos de lectura** en el cluster origen
- **Permisos de escritura** en el cluster destino

### Instalación de PostgreSQL client tools

```bash
# Ubuntu/Debian
sudo apt-get install postgresql-client

# macOS
brew install postgresql

# Windows
# Descargar desde https://www.postgresql.org/download/windows/
```

## 🔗 Obtener credenciales de conexión

En ambos proyectos de Supabase:
1. Ve a **Settings** → **Database** → **Connection string**
2. Copia la URI de conexión que tiene este formato:
   ```
   postgresql://postgres:[PASSWORD]@[HOST]:[PORT]/postgres
   ```

## 📤 Paso 1: Crear backup del cluster origen

### Comando de backup (recomendado)

```bash
pg_dump "postgresql://postgres:[PASSWORD_ORIGEN]@[HOST_ORIGEN]:[PORT]/postgres" \
  --clean \
  --if-exists \
  --create \
  --verbose \
  --no-owner \
  --no-privileges \
  > supabase_backup.sql
```

### Explicación de parámetros

- `--clean`: Elimina objetos antes de recrearlos
- `--if-exists`: Usa IF EXISTS al eliminar objetos (evita errores)
- `--create`: Incluye comandos para crear la base de datos
- `--verbose`: Muestra progreso detallado
- `--no-owner`: No incluye comandos de ownership
- `--no-privileges`: No incluye comandos de permisos
- `> archivo.sql`: Redirecciona la salida al archivo

### Verificar el backup creado

```bash
# Verificar que el archivo existe y su tamaño
ls -lh supabase_backup.sql

# Ver primeras líneas
head -10 supabase_backup.sql

# Ver últimas líneas para confirmar que está completo
tail -10 supabase_backup.sql

# Contar líneas (debe tener miles)
wc -l supabase_backup.sql
```

## 📥 Paso 2: Restaurar en cluster destino

### Comando de restauración

```bash
psql "postgresql://postgres:[PASSWORD_DESTINO]@[HOST_DESTINO]:[PORT]/postgres" \
  --file=supabase_backup.sql \
  --verbose
```

### Alternativa con redirección (si hay problemas con --file)

```bash
psql "postgresql://postgres:[PASSWORD_DESTINO]@[HOST_DESTINO]:[PORT]/postgres" \
  --verbose < supabase_backup.sql
```

## ✅ Paso 3: Verificación post-migración

### Verificar estructura y datos

```bash
# Conectarse al cluster destino para verificar
psql "postgresql://postgres:[PASSWORD_DESTINO]@[HOST_DESTINO]:[PORT]/postgres"
```

```sql
-- Verificar tablas migradas
\dt public.*;

-- Verificar conteo de registros en tablas principales
SELECT schemaname, tablename, n_tup_ins as rows 
FROM pg_stat_user_tables 
WHERE schemaname = 'public';

-- Verificar funciones personalizadas
\df public.*;

-- Verificar secuencias
\ds;

-- Ejemplo: verificar productos
SELECT COUNT(*) FROM products;
SELECT COUNT(*) FROM categories;
SELECT COUNT(*) FROM orders;
```

### Verificar funcionalidad específica de Supabase

```sql
-- Verificar autenticación
SELECT COUNT(*) FROM auth.users;

-- Verificar storage
SELECT COUNT(*) FROM storage.objects;

-- Verificar políticas RLS
SELECT schemaname, tablename, policyname 
FROM pg_policies 
WHERE schemaname = 'public';
```

## 🛠️ Troubleshooting

### Error: "no se encontró la orden"

Si ves errores como `--file=archivo.sql: no se encontró la orden`, usa redirección:

```bash
# En lugar de --file=archivo.sql
# Usar:
> archivo.sql  # para backup
< archivo.sql  # para restauración
```

### Error: "permission denied"

```bash
# Verificar permisos del directorio
ls -la

# Cambiar a directorio con permisos de escritura
cd ~
```

### Error de conexión

```bash
# Verificar conectividad
ping [HOST]

# Probar conexión simple
psql "postgresql://postgres:[PASSWORD]@[HOST]:[PORT]/postgres" -c "SELECT version();"
```

### Base de datos muy grande

Para bases de datos grandes (>1GB), considera:

```bash
# Backup comprimido
pg_dump "postgresql://..." --format=custom --compress=9 --file=backup.dump

# Restauración desde backup comprimido
pg_restore "postgresql://..." backup.dump
```

### Migración selectiva

Si solo necesitas ciertas tablas:

```bash
# Backup solo esquema público
pg_dump "postgresql://..." --schema=public > backup_public.sql

# Backup solo datos (sin estructura)
pg_dump "postgresql://..." --data-only > backup_data.sql

# Backup solo estructura (sin datos)
pg_dump "postgresql://..." --schema-only > backup_schema.sql
```

## 📝 Checklist de migración

- [ ] Credenciales de origen obtenidas
- [ ] Credenciales de destino obtenidas
- [ ] PostgreSQL client tools instalados
- [ ] Backup creado exitosamente
- [ ] Archivo de backup verificado
- [ ] Cluster destino preparado
- [ ] Restauración ejecutada
- [ ] Estructura verificada
- [ ] Datos verificados
- [ ] Funcionalidad probada
- [ ] Aplicación actualizada con nuevas credenciales

## ⚠️ Consideraciones importantes

1. **Tiempo de migración**: Para bases grandes, el proceso puede tomar horas
2. **Downtime**: Planifica una ventana de mantenimiento
3. **Credenciales**: Actualiza tu aplicación con las nuevas credenciales
4. **Storage**: Los archivos en storage deben migrarse por separado
5. **Extensiones**: Verifica que el cluster destino soporte las mismas extensiones
6. **Versiones**: Asegúrate de que las versiones de PostgreSQL sean compatibles

## 🔄 Migración de Storage (opcional)

Los archivos en Supabase Storage requieren migración separada:

```javascript
// Script para migrar archivos de storage
const { createClient } = require('@supabase/supabase-js')

const supabaseOrigen = createClient(URL_ORIGEN, KEY_ORIGEN)
const supabaseDestino = createClient(URL_DESTINO, KEY_DESTINO)

// Implementar lógica de migración de archivos
```

## 🎯 Script automatizado

Para automatizar el proceso completo:

```bash
#!/bin/bash

# Variables
SOURCE_DB="postgresql://postgres:[PASS_ORIGEN]@[HOST_ORIGEN]:5432/postgres"
TARGET_DB="postgresql://postgres:[PASS_DESTINO]@[HOST_DESTINO]:5432/postgres"
BACKUP_FILE="supabase_migration_$(date +%Y%m%d_%H%M%S).sql"

echo "🚀 Iniciando migración de Supabase..."

# Crear backup
echo "📤 Creando backup..."
pg_dump "$SOURCE_DB" \
  --clean \
  --if-exists \
  --create \
  --verbose \
  --no-owner \
  --no-privileges \
  > "$BACKUP_FILE"

if [ $? -eq 0 ]; then
  echo "✅ Backup creado: $BACKUP_FILE"
else
  echo "❌ Error al crear backup"
  exit 1
fi

# Restaurar
echo "📥 Restaurando en destino..."
psql "$TARGET_DB" \
  --file="$BACKUP_FILE" \
  --verbose

if [ $? -eq 0 ]; then
  echo "🎉 Migración completada exitosamente!"
else
  echo "❌ Error en la restauración"
  exit 1
fi
```

---

## 📞 Soporte

Si encuentras problemas:
1. Verifica las credenciales de conexión
2. Confirma la conectividad de red
3. Revisa los logs de error detalladamente
4. Considera migración por partes si es una base muy grande

**¡Buena suerte con tu migración!** 🚀