[README.md](https://github.com/user-attachments/files/33793039/README.md)
# NEXUS — Colegio Balbino García

Aplicación web para reportar daños y darles seguimiento. Mantiene Firebase Authentication y Firestore para cuentas, perfiles, reportes y mensajes. Las imágenes usan Supabase Storage a través de una Edge Function que valida el token de Firebase antes de permitir cada operación.

## Archivos principales

- `index.html`, `styles.css`: interfaz web.
- `app.js`: autenticación, perfiles, reportes y llamadas al almacenamiento de imágenes.
- `firebase-config.js`: configuración web pública de Firebase.
- `firestore.rules`: permisos de Firestore.
- `image-storage-config.js`: URL pública de la Edge Function (debes configurarla después del despliegue).
- `supabase/functions/nexus-images/index.ts`: backend que valida Firebase ID tokens, sube fotos y genera enlaces temporales de evidencias privadas.
- `storage.rules`: deniega el acceso a Firebase Storage; NEXUS ya no lo usa.

## Arquitectura de imágenes

- **Fotos de perfil:** bucket público `nexus-profile-photos`. Solo se sube contenido desde la función que valida la sesión Firebase; el contenido del bucket es públicamente visible por URL, así que no subas documentos ni evidencias aquí.
- **Evidencias de reportes:** bucket privado `nexus-evidence`. En Firestore se guarda la ruta del objeto, no una URL pública. La función solo genera una URL firmada de 5 minutos después de comprobar los permisos del reporte en Firestore.
- **Autorización:** Firebase ID token verificado contra las claves públicas oficiales de Firebase. El propietario puede subir/ver sus evidencias; `nexusadmin@gmail.com` puede verlas al iniciar sesión con su cuenta; un técnico puede verlas si el reporte está asignado a su UID. La función usa una clave de servicio solo en el servidor Supabase, nunca en los archivos del navegador.

Los límites gratuitos y las condiciones de almacenamiento de Supabase pueden cambiar. Revisa el uso y las cuotas de tu proyecto; este método evita tener que habilitar Firebase Storage, pero no promete almacenamiento ilimitado ni costo cero para siempre.

## Configuración necesaria (una sola vez)

### 1. Crear el proyecto y los buckets de Supabase

1. Crea un proyecto en Supabase y guarda su **Project URL** y el **Project Reference ID**.
2. En **Storage → New bucket**, crea `nexus-profile-photos` y marca **Public bucket**. Limita el tamaño máximo a 3 MB y los tipos MIME a `image/jpeg`, `image/png`, `image/webp`.
3. Crea otro bucket llamado `nexus-evidence`, dejando **Public bucket desactivado**. Limita el tamaño máximo a 5 MB y los mismos tipos MIME.
4. No compartas ni pegues en GitHub la clave `service_role` de Supabase.

### 2. Instalar y vincular Supabase CLI

Instala Node.js y Supabase CLI siguiendo la documentación oficial de Supabase. Desde la carpeta raíz de NEXUS:

```bash
supabase login
supabase link --project-ref TU_PROJECT_REF
```

### 3. Configurar el origen web permitido y desplegar la función

Usa el dominio exacto de tu web. Para GitHub Pages será algo parecido a `https://TU_USUARIO.github.io` o `https://TU_USUARIO.github.io/TU_REPOSITORIO`; usa el origen (sin la ruta del repositorio, sin barra final). Para pruebas locales, agrega `http://localhost:8000`. Separa los orígenes con coma.

```bash
supabase secrets set ALLOWED_ORIGINS="https://TU_USUARIO.github.io,http://localhost:8000"
supabase functions deploy nexus-images --no-verify-jwt
```

**Por qué `--no-verify-jwt`:** el gateway de Supabase no debe intentar validar el token de Supabase, porque el navegador envía un ID token de Firebase. La propia función verifica criptográficamente ese token de Firebase y valida permisos con Firestore antes de usar la clave de servicio. No elimines esa verificación del código ni publiques la clave de servicio.

La función usa `SUPABASE_URL` y `SUPABASE_SERVICE_ROLE_KEY` como variables de entorno de Supabase disponibles en el entorno de ejecución. Nunca copies `SUPABASE_SERVICE_ROLE_KEY` a `app.js`, HTML, `image-storage-config.js` ni al repositorio.

### 4. Configurar el endpoint en NEXUS

Después de desplegar la función, Supabase mostrará una URL parecida a:

`https://TU_PROJECT_REF.supabase.co/functions/v1/nexus-images`

Abre `image-storage-config.js` y reemplaza `PEGA_AQUI_LA_URL_DE_LA_FUNCION_SUPABASE` por la URL real. Esa URL no es una clave secreta. Luego publica todos los archivos actualizados en tu sitio.

### 5. Mantener Firebase

1. En Firebase Authentication, conserva habilitado el proveedor Email/Password y agrega el dominio publicado en **Authorized domains**.
2. Conserva Firestore con tus usuarios y reportes actuales. Publica `firestore.rules` desde Firebase Console → Firestore Database → Rules, o usa:

```bash
firebase login
firebase use nexus-6ecd6
firebase deploy --only firestore:rules
```

3. **No necesitas habilitar ni desplegar Firebase Storage para esta solución.** `storage.rules` lo deja cerrado. No uses `firebase deploy --only storage` como parte de esta configuración.

### 6. Pruebas antes de usar datos reales

- Inicia sesión con una cuenta estudiante y sube una foto de perfil.
- Crea un reporte con una evidencia y abre el detalle: la imagen debe abrirse mediante un enlace firmado temporal.
- Inicia sesión con otro estudiante e intenta abrir la ruta del primer reporte: debe denegarse.
- Prueba con `nexusadmin@gmail.com` iniciando sesión normalmente: debe poder ver las evidencias de los reportes que tiene permiso de leer en Firestore.
- Prueba con un técnico asignado al reporte; un técnico no asignado no debe obtener acceso.
- Revisa los logs de Edge Functions en Supabase si una operación falla.

## Notas de seguridad

- No guardes contraseñas, claves de servicio ni tokens en el repositorio.
- Las fotos de perfil son públicas por diseño. Si también quieres que sean privadas, habrá que cambiar el flujo para firmar sus URLs igual que las evidencias.
- Los enlaces firmados de evidencias expiran a los 5 minutos; vuelve a abrir el detalle para generar otro.
- La función comprueba el acceso al documento de reporte mediante el token del usuario y las reglas de Firestore antes de permitir la descarga. La autorización del frontend no sustituye las reglas.
- Antes de usar datos institucionales, prueba las reglas con cuentas de estudiante, técnico y administrador.

## Panel de administración

`nexusadmin@gmail.com` obtiene acceso administrativo al iniciar sesión correctamente; no se exige verificación por correo. El panel debe estar protegido también por las reglas de Firestore; ocultar botones en la interfaz no es una medida de seguridad.
