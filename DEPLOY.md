# UNIFORMASC · instalación en servidores propios

UNIFORMASC es una aplicación web instalable (PWA) hecha con React + Vite y un
backend Postgres/Supabase. No usa servicios propietarios de Enter: puedes
exportar el código, montar tu propia base de datos y publicarla en cualquier
hosting estático con HTTPS.

## Qué necesitas

- Node.js 20 o superior y `pnpm`.
- Un proyecto de Supabase (cloud o autoalojado) con Postgres, Auth, REST,
  Realtime y Edge Functions.
- Un hosting estático con HTTPS (Nginx, Apache, Netlify, Vercel, Cloudflare
  Pages, etc.). El HTTPS es obligatorio para instalar la app en el teléfono.

## 1. Base de datos

El esquema completo está en
`supabase/migrations/20261117090000_uniformasc.sql` y se puede ejecutar varias
veces sin romper nada.

Con la CLI de Supabase:

```bash
supabase link --project-ref TU_PROYECTO
supabase db push
```

O pegando el archivo en el editor SQL de tu proyecto. Crea las tablas
`uniformasc_clientes`, `uniformasc_productos`, `uniformasc_ventas`,
`uniformasc_venta_items` y `uniformasc_abonos`, sus reglas de negocio y las
políticas RLS.

## 2. Autenticación

En tu proyecto de Supabase:

- Habilita el proveedor **Email** (usuario y contraseña).
- Deshabilita el registro público de usuarios (`disable_signup`). La cuenta del
  negocio se crea desde la propia app con la función del punto 3.
- Deshabilita usuarios anónimos.

La app usa un usuario interno con correo `usuario@uniformasc.app`; el usuario
final solo escribe su nombre de usuario y contraseña.

## 3. Función de acceso

```bash
supabase functions deploy uniformasc-acceso
```

La función usa la llave de servicio que Supabase entrega automáticamente a las
Edge Functions. Solo permite crear la cuenta cuando todavía no existe ningún
usuario, por lo que debes crear tu cuenta en cuanto publiques la app.

## 4. Configurar el backend del frontend

Edita `public/config.js`:

```js
window.UNIFORMASC_CONFIG = {
  supabaseUrl: "https://TU-PROYECTO.supabase.co",
  supabaseAnonKey: "TU_LLAVE_PUBLICA_ANON",
};
```

La llave `anon` es pública por diseño: la seguridad de los datos vive en las
políticas RLS de la base de datos. Si dejas los valores vacíos, la app usa el
backend conectado en Enter.

## 5. Compilar y publicar

```bash
pnpm install
pnpm run build:prod
```

Sube el contenido de `dist/` a tu hosting. El archivo `public/config.js` se
copia tal cual, así que puedes cambiar de backend sin recompilar.

## 6. Primer uso

1. Abre la app en el navegador.
2. Pulsa **Crear la cuenta** y define usuario y contraseña del negocio.
3. Todos los dispositivos entran con esa misma cuenta.
4. Instala la app desde Chrome (Android) o Safari (iPhone) con la guía incluida
   en la sección **Instalar en tu teléfono**.

## Respaldos

Los datos viven en tu Postgres. Programa respaldos con `pg_dump` o el respaldo
automático de tu proveedor:

```bash
pg_dump "postgresql://USUARIO:CLAVE@HOST:5432/postgres" > uniformasc.sql
```

## Notas

- **No** se publica en App Store ni Google Play: se instala desde el navegador.
- La sincronización en vivo requiere Realtime habilitado en tu proyecto; si no
  está activo, la app sigue funcionando al recargar.
- `src/analytics.ts` es un no-op: no se envía telemetría a Enter.
- `vite-plugin-enter-dev` solo sirve para la vista previa de Enter. Para una
  compilación totalmente independiente puedes quitar esa dependencia de
  `package.json` y el bloque de plugins en `vite.config.ts`; Vite compila igual.
- Para exportar el código usa **Download Code** en el menú del proyecto.
