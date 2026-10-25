# Bersz Records — auditoría e integración parcial

Fecha: 2026-10-24  
Base: `bersz-records-portal-v2.9.1.zip`, extraído a una copia de trabajo. El ZIP original no se modificó.

## Resultado

Esta entrega no es una conversión completa ni debe considerarse lista para producción. Conecta dos operaciones que ya existen en el Web App: la validación de acceso con Google y el envío público de una solicitud de ingreso. Las demás pantallas no tienen endpoints disponibles suficientes para operar con seguridad. El acceso a la página heredada de demo se redirige a un aviso para no presentar sus datos ficticios como información real.

## Inventario

### Frontend recibido

- `index.html`: acceso, flujo Google previamente simulado y formulario «Solicitar acceso». La solicitud antes se guardaba en `localStorage`.
- `portal-demo.html`: aplicación React compilada en una cadena Babel, con cuentas y registros de ejemplo, cambios en `localStorage` y un vault de audio en IndexedDB. Contiene paneles para artistas, administración y marketing, pero la lógica sigue siendo local.
- `brand/` y `favicon.png`: recursos de marca conservados.
- El ZIP no contiene `.git`, `CNAME`, `Código.gs` ni `BackendFase1.gs`; por eso no se pudo crear una rama del historial original ni verificar sus rutas de publicación.

### Backend encontrado localmente

Se encontraron copias en Descargas llamadas `Codigo_Bersz_corregido_completo.gs`, `Codigo_Bersz_corregido.gs`, `BackendFase1_corregido.gs` y `BERSZ_RECORDS_Backend_Fase1.gs`. Son copias locales, no se pudo comprobar que sean idénticas al código activo del despliegue.

El snapshot de backend más completo (`Codigo_Bersz_corregido_completo.gs`) define:

- `GET`: `health`, `me` y listado de lanzamientos autenticado.
- `POST`: `health`, `login`, `submitAccessRequest`, `submit`, `listReleases` y `updateStatus`.
- Pestañas requeridas por ese código: `Users`, `Artists`, `Releases` y `ArtistAccessRequests`.
- Login: valida el ID token con `tokeninfo`, audiencia, emisor, correo verificado, `sub` y expiración; busca la cuenta en `Users`; y emite una sesión HMAC con duración de seis horas.
- Roles admitidos por el código: artista y administración. `Marketing Staff` no puede iniciar sesión por este flujo.
- Subida de WAV y portada: existe código para crear archivos en Drive durante `submit`, pero no hay endpoint de lectura/descarga privada. El código inspeccionado no valida un tamaño máximo ni una lista estricta de tipos MIME.
- El archivo `BackendFase1_corregido.gs` enumera además `Roles`, `RolePermissions`, `Tracks`, `Files` y `AuditLog`, y declara un dispatcher interno de solicitudes/arranque de propietario. No expone por sí solo una API completa para las funciones del portal.

El adjunto enumera pestañas adicionales como `Roles`, `RolePermissions`, `Tracks` y `Files`; la respuesta pública de salud solo confirma conexión y las cuatro pestañas requeridas indicadas arriba. No se leyó ni modificó la hoja de cálculo.

### Origen público revisado (solo lectura)

- `https://portal.berszrecords.top/` respondió HTTP 200 y todavía publica el botón de modo demo junto al código GIS del estado anterior ([sitio publicado](https://portal.berszrecords.top/)).
- `https://portal.berszrecords.top/portal-demo.html` respondió HTTP 200 y entrega el bundle de la demo (aprox. 596 KB), que contiene `DEMO_SEED_STATE` y `BERSZ_PORTAL_V2`.
- El árbol de accesibilidad del navegador del sitio público también mostró «Entrar en modo demo». Se intentó una prueba CORS desde la consola de navegador disponible, pero su entorno de evaluación no expone `fetch`; por eso la respuesta CORS solo quedó comprobada con peticiones HTTP desde consola, no en un navegador real.
- El ZIP fuente no trae `CNAME` y usa recursos `brand/`; la página pública observada referencia recursos en la raíz. No se modificó el origen ni se hizo despliegue.

## Cambios de esta copia

- `index.html`: el botón usa Google Identity Services y envía `action=login`; no persiste el ID token. Solo reconoce los roles `artist` y `admin` que admite el backend inspeccionado. Si login es válido, informa el rol, pero no abre `portal-demo.html` como sesión real porque esa página no consume los endpoints ni los datos de la sesión.
- `index.html`: «Solicitar acceso» envía los campos validados mediante `action=submitAccessRequest`. Solo confirma éxito ante `status: "success"`; errores del servidor o tiempo agotado se muestran como error/resultado incierto, sin crear una entrada local.
- `index.html`: se retiró el botón «Entrar en modo demo» y su flujo de transición.
- `portal-demo.html`: la ruta redirige al inicio con un aviso de que el panel no está disponible. Su código heredado permanece dentro del archivo, pero no llega a ejecutar el panel.
- `LEEME.txt`: documenta el estado y los límites de la integración.
- No se modificó ningún `.gs`, hoja, permiso, archivo de Drive ni despliegue.

## Endpoints conectados y pendientes

Conectados desde el navegador:

- `POST login` — ID token de Google para validación.
- `POST submitAccessRequest` — solicitud de ingreso pública.

Definidos en el snapshot local, pero no conectados a pantallas en esta copia:

- `GET me`, `GET/POST listReleases`, `POST submit`, `POST updateStatus`, `health`.
- No hay contratos implementados para cuentas/perfiles, `RolePermissions`, marketing, estadísticas, promociones, retiradas, notificaciones, soporte, adelantos, historiales completos ni acceso privado a archivos de Drive.

## Revisiones de seguridad y limitaciones

- `appendMappedRow` usa `appendRow` y las solicitudes públicas aceptan texto de usuario. No se encontró neutralización explícita de cadenas que comiencen con `=`; se requiere protección contra inyección de fórmulas.
- La comprobación de duplicados de solicitudes no usa `LockService` y no se encontró limitación de tasa.
- La sesión firmada expira y el servidor vuelve a consultar si el usuario está activo en operaciones autenticadas. No se encontró acción de cierre/revocación ni lista de revocación de sesiones.
- Las autorizaciones del snapshot se basan principalmente en el rol (`admin`/`artist`), no en una aplicación completa de `RolePermissions` para cada recurso.
- Aunque `Users` incluye el campo `is_demo` en el alta del propietario, el flujo de login inspeccionado no lo consulta al autenticar una fila existente. Si ese snapshot corresponde al despliegue, se debe impedir en servidor que cuentas demo entren en producción.
- El endpoint de subida no tiene validación estricta de tamaño/tipo en el código inspeccionado, y no existe un endpoint de lectura privado para Drive.
- El frontend usa una petición CORS simple `text/plain` con JSON en el cuerpo para evitar preflight; el parser de Apps Script procesa `postData.contents` como JSON. No se usa `no-cors`.
- Apps Script devolvió `Access-Control-Allow-Origin: *` en las respuestas probadas y redirigió respuestas de ContentService a `script.googleusercontent.com`. La implementación usa `redirect: "follow"` y espera JSON. La guía de Google documenta esta redirección.
- No se probó el flujo en el navegador real de `portal.berszrecords.top` ni con una cuenta autorizada. Antes de publicar, el origen HTTPS debe estar permitido en el cliente OAuth y el navegador/host deben permitir el flujo GIS de popup.
- La página pública sigue siendo la versión anterior hasta que alguien publique y verifique este ZIP; esta entrega no cambia producción.

## Pruebas ejecutadas

- Sintaxis JavaScript del bloque de `index.html`: correcta con `node --check`.
- Sintaxis del guard y del módulo de `portal-demo.html`: correcta con `node --check`.
- Estructura HTML: se confirmó la presencia de los controles de Google, solicitud, aviso y puntos de entrada previstos.
- `GET .../exec?action=health`: HTTP 200; estado correcto, hoja `BERSZ_RECORDS_DB_PORTAL_v2.9.1_REGENERADO` conectada y pestañas requeridas reportadas.
- `POST` `action=login` con credencial deliberadamente inválida: backend devolvió error de token inválido/expirado; no creó sesión.
- `POST` `action=submitAccessRequest` sin campos: backend rechazó la petición por campos obligatorios; no se insertó una solicitud.
- No se realizó un login positivo, una escritura válida, una subida ni una prueba con roles reales para evitar modificar datos reales o usar cuentas de producción.

## Pendiente para completar la conversión

1. Auditar el código efectivamente desplegado y los encabezados reales de todas las hojas con acceso de solo lectura autorizado.
2. Diseñar e implementar en el backend contratos autenticados por recurso y rol; agregar Marketing Staff y usar `RolePermissions` si corresponde a la estructura real.
3. Implementar revisión/aprobación de solicitudes, usuarios/artistas, lanzamientos completos, tracks, marketing, soporte, notificaciones, historiales y demás operaciones que hoy solo simula el cliente.
4. Añadir sesiones revocables, protección de concurrencia y tasa, saneamiento para Sheets, validación de archivos y lectura/descarga privada de Drive.
5. Conectar el frontend a esos contratos, retirar estado local simulado y probar acceso/aislamiento/error/persistencia en un entorno de pruebas autorizado antes de publicar.
6. Verificar el repositorio Git y `CNAME` del origen de producción; no venían en el ZIP de entrada.

