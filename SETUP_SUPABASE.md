# Configuración de Supabase para KMZ Studio

## Paso 1: Crear cuenta en Supabase (GRATIS)

1. Ve a https://supabase.com
2. Click en "Start your project" o "Sign Up"
3. Registrate con tu email o GitHub
4. Crea un nuevo proyecto:
   - Nombre: `kmz-turnos`
   - Contraseña de base de datos: (guárdala en un lugar seguro)
   - Región: South America (São Paulo) - la más cercana

## Paso 2: Crear la tabla de turnos

1. En el dashboard de Supabase, ve a **SQL Editor**
2. Copia y pega este código SQL:

```sql
-- Crear tabla de turnos
CREATE TABLE turnos (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  fecha DATE NOT NULL,
  hora VARCHAR(5) NOT NULL,
  servicio VARCHAR(100) NOT NULL,
  nombre_cliente VARCHAR(100),
  telefono VARCHAR(20),
  estado VARCHAR(20) DEFAULT 'confirmado',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

  -- Evitar duplicados: misma fecha y hora
  UNIQUE(fecha, hora)
);

-- Habilitar Row Level Security (RLS)
ALTER TABLE turnos ENABLE ROW LEVEL SECURITY;

-- Política para que todos puedan LEER los turnos
CREATE POLICY "Todos pueden ver turnos" ON turnos
  FOR SELECT USING (true);

-- Política para que todos puedan INSERTAR turnos
CREATE POLICY "Todos pueden crear turnos" ON turnos
  FOR INSERT WITH CHECK (true);

-- Política para que solo admins puedan ELIMINAR
-- (por ahora permitimos a todos para el panel admin)
CREATE POLICY "Todos pueden eliminar turnos" ON turnos
  FOR DELETE USING (true);

-- Habilitar realtime para la tabla
ALTER PUBLICATION supabase_realtime ADD TABLE turnos;
```

3. Click en "Run" para ejecutar

## Paso 3: Obtener las credenciales

1. Ve a **Project Settings** (ícono de engranaje)
2. Click en **API**
3. Copia estos valores:
   - **Project URL**: `https://xxxx.supabase.co`
   - **anon public key**: `eyJhbGciOiJI...` (la clave pública)

## Paso 4: Configurar en el proyecto

1. Crea un archivo `.env` en la raíz del proyecto con:

```env
PUBLIC_SUPABASE_URL=https://tu-proyecto.supabase.co
PUBLIC_SUPABASE_ANON_KEY=tu-clave-anon-publica
```

2. Reemplaza los valores con los que copiaste de Supabase

## Paso 5: Ejecutar el proyecto

```bash
npm install
npm run dev
```

## Cómo funciona

- Cuando un usuario confirma un turno por WhatsApp, el turno se guarda en Supabase
- TODOS los usuarios ven los mismos turnos disponibles/ocupados
- Los cambios se sincronizan en TIEMPO REAL (si alguien reserva, los demás lo ven al instante)
- El panel de admin permite eliminar turnos

## Troubleshooting

### Error: "relation turnos does not exist"
- Asegúrate de haber ejecutado el SQL del Paso 2

### Los turnos no se sincronizan
- Verifica que las credenciales en `.env` sean correctas
- Revisa la consola del navegador por errores

### Error de CORS
- En Supabase Dashboard > Settings > API
- Agrega tu dominio de Netlify a los "Allowed origins"
