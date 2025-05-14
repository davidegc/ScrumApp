# Scrum App con Google Apps Script y Google Sheets

Esta aplicación web, construida con Google Apps Script, sirve como un dashboard para visualizar el progreso de un equipo Scrum, utilizando Google Sheets como backend para almacenar los datos.

## Características

* Visualización de Features, Historias de Usuario y Dependencias.
* Filtros globales por Seguimiento (Responsable) y Estatus.
* Barras de progreso generales para cada sección (Features, HU, Dependencias) basadas en el estado "Deployed".
* "Podio del Q" para gamificar el cierre de Features por responsable.
* Sección de "Anuncios" configurable, con opción de mostrar mensajes específicos siempre.
* Tarjeta de "Mensajes del Equipo" configurable desde la hoja `Config`.
* Registro de "Mood del Usuario" y visualización del "Mood del Equipo (Hoy)" (mood más frecuente del día).
* Sección de "Enlaces Útiles".
* Funcionalidad de "Comentarios Rápidos" con notificación opcional por email.
* Contador de Sprint.
* Botón de "Ayuda" configurable para enlazar a un chat.
* Título de la aplicación configurable.

## Prerrequisitos

* Una cuenta de Google.
* Acceso a Google Drive, Google Sheets y Google Apps Script.

## Configuración

### 1. Crear la Hoja de Cálculo de Google (Google Sheet)

1.  Crea una nueva Hoja de Cálculo en Google Drive.
2.  **Obtén el ID de la Hoja de Cálculo:** El ID se encuentra en la URL de tu hoja. Por ejemplo, si la URL es `https://docs.google.com/spreadsheets/d/ABC123XYZ789/edit`, el ID es `ABC123XYZ789`.
3.  **Crea las siguientes hojas (pestañas) dentro del archivo, con los nombres exactos:**
    * `Features`
    * `UserStories`
    * `Dependencies`
    * `Announcements`
    * `Config`
    * `Moods`
    * `Comments`
    * `QuickLinks`

### 2. Configurar las Columnas en Cada Hoja

Asegúrate de que la primera fila de cada hoja contenga los siguientes encabezados de columna (los nombres son importantes, aunque el script intenta normalizar algunos):

* **Hoja `Config`:**
    * `Key`
    * `Value`
    * *Ejemplos de Keys (ver sección "Configuración Avanzada" más abajo)*

* **Hoja `Features`:**
    * `ID` (Identificador único para la Feature, usado para enlazar HU y Dependencias)
    * `Name` (Nombre de la Feature)
    * `Status` (Ej: New, Deployed, Blocked, Discarded, u otros)
    * `Seguimiento` (Nombre de la persona responsable)
    * `StatusOrder` (Opcional, número para ordenar por estatus)
    * `URL` (Opcional, enlace a más detalles)
    * `FeatureID` (Este campo no es necesario aquí, el `ID` de esta hoja es el FeatureID para otras)

* **Hoja `UserStories`:**
    * `ID` (ID de la Historia de Usuario, ej: JIRA-123)
    * `Name` (Descripción de la Historia de Usuario)
    * `Status`
    * `Seguimiento`
    * `FeatureID` (ID de la Feature a la que pertenece, debe coincidir con un `ID` de la hoja `Features`)
    * `JiraLink` (Opcional, enlace al ticket en Jira u otro sistema)
    * `StatusOrder` (Opcional)

* **Hoja `Dependencies`:**
    * `Name` (Descripción de la Dependencia)
    * `Status`
    * `Seguimiento`
    * `FeatureID` (Opcional, si la dependencia está ligada a una Feature específica)
    * `URL` (Opcional)
    * `StatusOrder` (Opcional)

* **Hoja `Announcements`:**
    * `Text` (Texto del anuncio)
    * `Link` (Opcional, URL para el anuncio)
    * `Tag` (Opcional, texto corto para una etiqueta, ej: "Importante")
    * `TagClass` (Opcional, clase CSS para el color del tag, ej: `badge-red`, `badge-green`)
    * `Show` (Opcional, poner `TRUE` si quieres que este anuncio se muestre siempre que haya alguno marcado así. Si no hay ninguno con `TRUE`, se mostrará uno aleatorio.)

* **Hoja `Moods`:** (El script la llenará automáticamente)
    * Columna A: Timestamp
    * Columna B: Email del Usuario
    * Columna C: Emoji del Mood

* **Hoja `Comments`:** (El script la llenará automáticamente)
    * Columna A: Timestamp
    * Columna B: Email del Usuario
    * Columna C: Texto del Comentario

* **Hoja `QuickLinks`:**
    * `Text` (Texto visible del enlace)
    * `URL` (Dirección URL del enlace)
    * `Icon` (Opcional, clase de FontAwesome, ej: `fas fa-file-alt`)
    * `Order` (Opcional, número para ordenar los enlaces)

### 3. Configurar Google Apps Script

1.  Abre tu Hoja de Cálculo de Google.
2.  Ve a `Extensiones` > `Apps Script`. Se abrirá el editor de Apps Script.
3.  **Archivo `Codigo.gs`:**
    * Borra cualquier contenido existente en el archivo `Codigo.gs`.
    * Copia **todo** el contenido del archivo `Codigo.gs` proporcionado y pégalo en el editor.
    * **IMPORTANTE:** Reemplaza el valor de la constante `SPREADSHEET_ID` al principio del archivo con el ID real de tu Hoja de Cálculo.
        ```javascript
        const SPREADSHEET_ID = 'TU_ID_DE_HOJA_DE_CALCULO_AQUI';
        ```
4.  **Archivo `Index.html`:**
    * En el editor de Apps Script, haz clic en el `+` junto a "Archivos" y selecciona `HTML`.
    * Nombra el archivo `Index.html` (respetando mayúsculas y minúsculas) y presiona Enter.
    * Borra cualquier contenido existente en `Index.html`.
    * Copia **todo** el contenido del archivo `Index.html` proporcionado y pégalo.
5.  Guarda los cambios en ambos archivos (icono de disquete o `Ctrl+S` / `Cmd+S`).

### 4. Desplegar la Aplicación Web

1.  En el editor de Apps Script, haz clic en el botón `Desplegar` (arriba a la derecha).
2.  Selecciona `Nuevo despliegue`.
3.  Junto a "Seleccionar tipo", haz clic en el icono del engranaje y elige `Aplicación web`.
4.  En el cuadro de diálogo:
    * **Descripción:** Puedes poner algo como "Scrum App v1".
    * **Ejecutar como:** Selecciona `Yo ([tu dirección de correo])`.
    * **Quién tiene acceso:**
        * `Cualquier usuario con una cuenta de Google` (recomendado si es para un equipo dentro de una organización).
        * `Cualquier usuario` (si necesitas que sea accesible públicamente sin inicio de sesión de Google, aunque el script intenta obtener el email del usuario).
5.  Haz clic en `Desplegar`.
6.  **Autorización:** La primera vez que despliegues, Google te pedirá que autorices los permisos que el script necesita (acceder a Google Sheets, enviar emails si configuraste la notificación de comentarios, etc.). Sigue los pasos:
    * Haz clic en `Autorizar acceso`.
    * Elige tu cuenta de Google.
    * Puede que veas una advertencia de "Google no ha verificado esta aplicación". Haz clic en `Configuración avanzada` (o similar) y luego en `Ir a [Nombre de tu proyecto] (no seguro)`.
    * Revisa los permisos y haz clic en `Permitir`.
7.  Una vez desplegado, se te proporcionará una **URL de la aplicación web**. Esta es la URL que usarás para acceder a tu Scrum App. Cópiala.

## Configuración Avanzada (Hoja `Config`)

La hoja `Config` te permite personalizar varios aspectos de la aplicación sin modificar el código:

| Key                         | Descripción                                                                 | Ejemplo de Value                                        |
| :-------------------------- | :-------------------------------------------------------------------------- | :------------------------------------------------------ |
| `APP_TITLE`                 | El título que se muestra en la aplicación y en la pestaña del navegador.     | `Dashboard del Equipo Alfa`                             |
| `HELP_CHAT_URL`             | La URL a la que enlazará el botón flotante de "Ayuda".                      | `https://chat.google.com/room/ABCXYZ`                   |
| `TEAM_MESSAGE_CARD_TITLE`   | El título de la tarjeta de mensajes/anuncios del equipo.                    | `Avisos Importantes`                                    |
| `TEAM_MESSAGE_CONTENT`      | El contenido del mensaje que se mostrará en la tarjeta de mensajes.         | `Recordatorio: Daily a las 9 AM. ¡No olvidar el café! ☕` |
| `COMMENT_NOTIFICATION_EMAIL`| La dirección de correo a la que se enviarán notificaciones de nuevos comentarios. | `jefe.de.proyecto@ejemplo.com`                        |
| `SPRINT_END_DATE`           | Fecha de finalización del sprint actual para el contador. Formato: `YYYY-MM-DDTHH:MM:SS` (ej: `2024-05-30T17:00:00`) o simplemente `YYYY-MM-DD`. | `2025-05-30`                                            |
| `SPRINT_INFO_YEAR`          | Año del sprint (ej: `2024`).                                                | `2024`                                                  |
| `SPRINT_INFO_QUARTER`       | Trimestre del sprint (ej: `Q2`).                                            | `Q2`                                                    |
| `SPRINT_INFO_NUMBER_IN_QUARTER`| Número del sprint dentro del trimestre (ej: `3`).                          | `3`                                                     |

## Uso de la Aplicación

* Abre la URL de la aplicación web obtenida durante el despliegue.
* Los datos se cargarán desde tu Google Sheet.
* Usa los filtros para refinar la vista.
* Interactúa con las tarjetas de Mood, Comentarios, etc.

## Solución de Problemas

* **La aplicación muestra un error al cargar:**
    * Verifica que el `SPREADSHEET_ID` en `Codigo.gs` sea correcto.
    * Asegúrate de que los nombres de todas las hojas en tu Google Sheet coincidan exactamente con los definidos en `Codigo.gs`.
    * Revisa que los encabezados de las columnas en cada hoja sean los correctos.
    * Abre el editor de Apps Script y ve a "Ejecuciones" para ver los logs del servidor. Cualquier error en `getAllSheetData` o funciones relacionadas aparecerá ahí.
    * Abre la consola de desarrollador de tu navegador (`Ctrl+Shift+J` o `Cmd+Opt+J`) para ver errores del lado del cliente.
* **Los datos no se actualizan:** Asegúrate de que los datos en tu Google Sheet estén guardados. La aplicación carga los datos cada vez que se abre o actualiza la página.
* **Permisos:** Si realizas cambios significativos en el script que requieran nuevos permisos (ej: usar un nuevo servicio de Google), puede que necesites volver a autorizar la aplicación desplegándola de nuevo.

