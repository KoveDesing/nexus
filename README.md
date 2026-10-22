[README.md](https://github.com/user-attachments/files/33793989/README.md)
# NEXUS — Colegio Balbino García

Aplicación web responsive para registrar y hacer seguimiento a fallas institucionales, con tema oscuro, acento configurable, perfil básico y panel privado de administración. Esta versión usa Firebase Authentication y Firestore. La foto de perfil se guarda localmente en el navegador; no utiliza servicios externos de imágenes.

## Contenido
- `index.html`: interfaz y vistas.
- `styles.css`: diseño adaptable y personalizable.
- `app.js`: autenticación, reportes, conversación, estados y perfil.
- `firebase-config.js`: configuración del proyecto Firebase.
- `firestore.rules`: reglas de seguridad de Firestore.
- `firebase.json`: configuración para Firebase CLI y Hosting.

## Antes de publicar
1. En Firebase Console, abre el proyecto `nexus-6ecd6`.
2. En **Authentication → Sign-in method**, habilita **Email/Password**.
3. En **Firestore Database**, crea la base de datos si todavía no existe.
4. Publica `firestore.rules` (ver abajo).
5. Sirve la web mediante un servidor HTTP; no abras `index.html` con `file://`, porque usa módulos JavaScript.

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

GitHub Pages solo aloja los archivos estáticos; Firebase gestiona la autenticación y la base de datos. Para publicar las reglas de seguridad se necesita Firebase Console o Firebase CLI.

## Publicar reglas con Firebase CLI
Instala Node.js y Firebase CLI, inicia sesión y selecciona el proyecto:
```bash
npm install -g firebase-tools
firebase login
firebase use nexus-6ecd6
firebase deploy --only firestore:rules
```
Para desplegar también Hosting:
```bash
firebase deploy --only hosting
```
El archivo `firebase.json` configura Hosting para servir la raíz del proyecto. La carga de imágenes está desactivada temporalmente para que el inicio de sesión, el registro y los reportes no dependan de un servicio de almacenamiento de archivos.

## Panel privado de administración
La cuenta `nexusadmin@gmail.com` recibe el rol de Administrador según el correo autenticado por Firebase. Inicia sesión con esa cuenta para abrir directamente el panel privado. Las reglas de Firestore también comprueban el correo autenticado; ocultar botones en la interfaz no es la protección principal.

Las cuentas nuevas se crean como `estudiante`; no hay botón público ni contraseña secreta para convertirse en administrador. Los usuarios ven sus propios reportes; los técnicos ven los casos asignados; la cuenta administradora ve todas las solicitudes. Publica `firestore.rules` antes de usar datos reales.

**Importante:** las reglas de Firestore son la barrera de seguridad real; ocultar controles en el navegador no concede ni revoca permisos. Revisa y prueba las reglas en Firebase Rules Playground antes de usar datos reales. Las reglas iniciales de este repositorio son una base funcional que debe validarse y ajustarse a las políticas institucionales. Para organizaciones con datos sensibles, conviene mover cambios de roles y asignaciones privilegiadas a un backend confiable/Cloud Functions y usar notificaciones push/email mediante un servicio backend.

## Sobre el identificador
El ID del documento Firestore es único. El código visible `NX-AAAA-XXXXXX` se construye a partir del año y una parte del ID. Para una garantía formal de secuencia y unicidad visible, implementar una Cloud Function que asigne consecutivos transaccionales.

## Seguridad de credenciales
No guardes contraseñas en el repositorio, HTML, JavaScript ni archivos de configuración. La contraseña compartida en el mensaje no se incluye en estos archivos. Si es real, cámbiala de inmediato desde Firebase Authentication / la cuenta correspondiente y activa MFA en las cuentas administrativas. La configuración web de Firebase es pública por diseño; las reglas de acceso y Authentication deben proteger los datos.

## Personalización visual
La interfaz usa tema oscuro con fondos carbón/gris, tarjetas un poco más claras e icono NEXUS geométrico. Cada usuario puede abrir el engranaje junto a su perfil o tocar su avatar arriba para cambiar nombre, elegir cualquier color de acento. La foto de perfil se comprime y se guarda localmente en el navegador actual, sin subirla a ningún servicio externo.

## Limitaciones de esta versión
- El envío de correo/avisos automáticos, SLA programados y escalamiento por vencimiento requieren Cloud Functions o un servicio backend.
- La prioridad crítica queda sujeta a revisión institucional; los incidentes con peligro inmediato deben seguir el protocolo de emergencia del colegio.
- Los mensajes se almacenan en el documento del reporte para simplificar el MVP. Para conversaciones extensas, conviene migrarlos a una subcolección `reports/{id}/messages`.
- La asignación de técnicos se realiza desde el panel privado de administración. El técnico ve únicamente los casos asignados a su cuenta.


## Personalización y administración añadidas
- Tema claro/oscuro y color de acento personalizable.
- Foto de perfil comprimida y guardada en el almacenamiento local del navegador (no se sube a Firebase ni a servicios externos; no se sincroniza entre dispositivos).
- Panel de administración con conteo de cuentas, alumnos y estados de suspensión.
- La suspensión se aplica en reglas de Firestore. Para deshabilitar también la autenticación en Firebase se necesita una función de servidor/Admin SDK, que no se incluye en una web estática.
- Modal informativo de privacidad y seguridad.

**Importante al desplegar:** publica también `firestore.rules` desde Firebase Console → Firestore Database → Rules. El acceso administrativo se determina por el correo `nexusadmin@gmail.com`; no otorgues roles privilegiados a cuentas no autorizadas.
