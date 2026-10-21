[README.md](https://github.com/user-attachments/files/33791865/README.md)
# NEXUS — Colegio Balbino García

Aplicación web responsive para registrar y hacer seguimiento a fallas institucionales, con Firebase Authentication, Cloud Firestore y Cloud Storage.

## Contenido
- `index.html`: interfaz y vistas.
- `styles.css`: diseño adaptable y personalizable.
- `app.js`: autenticación, reportes, conversación, estados y perfil.
- `firebase-config.js`: configuración del proyecto Firebase.
- `firestore.rules`: reglas de seguridad de Firestore.
- `storage.rules`: reglas de seguridad para evidencias y fotos.
- `firebase.json`: configuración para Firebase CLI y Hosting.

## Antes de publicar
1. En Firebase Console, abre el proyecto `nexus-6ecd6`.
2. En **Authentication → Sign-in method**, habilita **Email/Password**.
3. En **Firestore Database**, crea la base de datos si todavía no existe.
4. En **Storage**, habilita el bucket.
5. Publica `firestore.rules` y `storage.rules` (ver abajo).
6. Sirve la web mediante un servidor HTTP; no abras `index.html` con `file://`, porque usa módulos JavaScript.

## Probar localmente
Con Python instalado, desde la carpeta del proyecto:
```bash
python -m http.server 8000
```
Luego abre `http://localhost:8000`.

## Publicar en GitHub Pages
1. Sube todos los archivos a la raíz de un repositorio GitHub.
2. Ve a **Settings → Pages** y selecciona la rama y carpeta raíz.
3. Espera la publicación y abre la URL de GitHub Pages.
4. En Firebase Console → Authentication → Settings → Authorized domains, agrega el dominio de tu web publicado (y `localhost` para pruebas).

GitHub Pages solo aloja los archivos estáticos; Firebase gestiona autenticación, base de datos y archivos. Para publicar las reglas de seguridad se necesita Firebase Console o Firebase CLI.

## Publicar reglas con Firebase CLI
Instala Node.js y Firebase CLI, inicia sesión y selecciona el proyecto:
```bash
npm install -g firebase-tools
firebase login
firebase use nexus-6ecd6
firebase deploy --only firestore:rules,storage
```
Para desplegar también Hosting:
```bash
firebase deploy --only hosting
```
El archivo `firebase.json` configura Hosting para servir la raíz del proyecto.

## Roles
Las cuentas nuevas se crean como `estudiante`. No existe un botón público para elevar permisos. Para otorgar un rol, un administrador institucional debe editar el documento `users/{UID}` en Firestore Console y asignar `docente`, `funcionario`, `tecnico`, `directivo` o `administrador`. Copia el UID desde Firebase Authentication. Protege el acceso a Firebase Console y asigna el rol `administrador` únicamente a personas de confianza.

**Importante:** las reglas de Firestore son la barrera de seguridad real; ocultar controles en el navegador no concede ni revoca permisos. Revisa y prueba las reglas en Firebase Rules Playground antes de usar datos reales. Las reglas iniciales de este repositorio son una base funcional que debe validarse y ajustarse a las políticas institucionales. Para organizaciones con datos sensibles, conviene mover cambios de roles y asignaciones privilegiadas a un backend confiable/Cloud Functions y usar notificaciones push/email mediante un servicio backend.

## Sobre el identificador
El ID del documento Firestore es único. El código visible `NX-AAAA-XXXXXX` se construye a partir del año y una parte del ID. Para una garantía formal de secuencia y unicidad visible, implementar una Cloud Function que asigne consecutivos transaccionales.

## Seguridad de credenciales
No guardes contraseñas en el repositorio, HTML, JavaScript ni archivos de configuración. La contraseña compartida en el mensaje no se incluye en estos archivos. Si es real, cámbiala de inmediato desde Firebase Authentication / la cuenta correspondiente y activa MFA en las cuentas administrativas. La configuración web de Firebase es pública por diseño; las reglas de acceso y Authentication deben proteger los datos.

## Limitaciones de esta versión
- El envío de correo/avisos automáticos, SLA programados y escalamiento por vencimiento requieren Cloud Functions o un servicio backend.
- La prioridad crítica queda sujeta a revisión institucional; los incidentes con peligro inmediato deben seguir el protocolo de emergencia del colegio.
- Los mensajes se almacenan en el documento del reporte para simplificar el MVP. Para conversaciones extensas, conviene migrarlos a una subcolección `reports/{id}/messages`.
- El panel directivo y técnico usa la misma lista de casos autorizados; una futura versión puede agregar filtros, indicadores avanzados y asignación con auditoría backend.
