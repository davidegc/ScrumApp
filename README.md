# Scrum App con Google Apps Script y Google Sheets

Esta aplicación web, construida con Google Apps Script, sirve como un dashboard para visualizar el progreso de un equipo Scrum, utilizando Google Sheets como backend para almacenar los datos.

## Características

* Visualización de Features, Historias de Usuario y Dependencias.
* **Tabla de Historias de Usuario Pendientes:** Muestra las historias no desplegadas con columnas específicas (Nombre, Seguimiento, ID Feature, ID Historia, Sprint, Estatus, Prioridad), ancho de columnas uniforme y scroll vertical.
* **Filtros globales por Q, Sprint, Actividad (Feature, Historia, Dependencia), Seguimiento (Responsable) y Estatus.**
* Barras de progreso generales para la sección de Features, **mostrando conteo de Features, Historias de Usuario y Dependencias consideradas en el progreso**, basadas en el estado "Deployed".
* Podios de progreso para Features, Historias de Usuario y Dependencias por responsable, **mostrando conteo de ítems desplegados sobre el total**.
* Sección de "Anuncios" configurable desde la hoja `Announcements`, con animación y opción de cierre.
* Sección de "Comunicados del Equipo" dinámica, alimentada desde la hoja `Comunicados`, mostrando fecha, mensaje y enlace opcional con icono. Incluye buscador para filtrar comunicados. **Con opción de enlace de edición externo.**
* Registro de "Mood del Usuario" y visualización del "Mood del Equipo (Hoy)" (mood más frecuente del día).
* Sección de "Enlaces Útiles" con buscador y opción de marcar favoritos. **Con opción de enlace de edición externo.**
* Registro de "Impedimentos" con notificación opcional por email y asociación a Features/Historias de Usuario.
* Sección de "Próximos Eventos" cargados desde una hoja de cálculo. **Con opción de enlace de edición externo.**
* Contador de Sprint con fecha de finalización configurable.
* Botón de "Ayuda" configurable para enlazar a un chat.
* Título y subtítulo de la aplicación configurables.
* **El pie de página muestra "Documento Fuente" como un enlace a la hoja de cálculo si está configurado.**

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
    * `Impediments`
    * `QuickLinks`
    * `UpcomingEvents`
    * `Comunicados`

### 2. Configurar las Columnas en Cada Hoja

Asegúrate de que la primera fila de cada hoja contenga los siguientes encabezados de columna (los nombres son importantes, aunque el script intenta normalizar algunos):

* **Hoja `Config`:**
    * `Key`
    * `Value`
    * *Ejemplos de Keys (ver sección "Configuración Avanzada" más abajo)*

* **Hoja `Features`:**
    * `ID` (Identificador único para la Feature)
    * `Name` (Nombre de la Feature)
    * `Status` (Ej: New, Deployed, Blocked, Discarded, In Progress, QA, u otros)
    * `Seguimiento` (Nombre de la persona responsable)
    * `Q` (Opcional, ej: Q1, Q2)
    * `Priority` (Opcional, ej: Alta, Media, Baja)
    * `URL` (Opcional, enlace a más detalles de la Feature)

* **Hoja `UserStories`:**
    * `ID` (ID de la Historia de Usuario, ej: JIRA-123)
    * `Name` (Descripción de la Historia de Usuario)
    * `Status`
    * `Seguimiento`
    * `FeatureID` (ID de la Feature a la que pertenece, debe coincidir con un `ID` de la hoja `Features`)
    * `Sprint` (Opcional, número o nombre del Sprint)
    * `Priority` (Opcional)
    * `JiraLink` (Opcional, enlace al ticket en Jira u otro sistema)

* **Hoja `Dependencies`:**
    * `Name` (Descripción de la Dependencia)
    * `Status`
    * `Seguimiento`
    * `FeatureID` (Opcional, ID de la Feature a la que está ligada)
    * `Sprint` (Opcional)
    * `Priority` (Opcional)
    * `URL` (Opcional, enlace a más detalles de la dependencia)

* **Hoja `Announcements`:**
    * `Text` (Texto del anuncio)
    * `Link` (Opcional, URL para el anuncio)
    * `Order` (Opcional, número para ordenar los anuncios)
    * `Show` (`TRUE` o `FALSE`. Si hay alguno con `TRUE`, se mostrarán esos. Si no, se mostrará uno aleatorio de los que no estén explícitamente en `FALSE`)

* **Hoja `Moods`:** (El script la llenará automáticamente)
    * Columna A: Timestamp
    * Columna B: Email del Usuario
    * Columna C: Emoji del Mood

* **Hoja `Impediments`:** (El script la llenará automáticamente al registrar impedimentos)
    * `Timestamp`
    * `UserEmail`
    * `Title` (Título del impedimento, autogenerado o basado en Feature/HU)
    * `Description`
    * `Status` (Por defecto "Nuevo")
    * `AssociatedUserStoryID` (Opcional, ID de la HU asociada)
    * `AssociatedFeatureID` (Opcional, ID de la Feature asociada)

* **Hoja `QuickLinks`:**
    * `Text` (Texto visible del enlace)
    * `URL` (Dirección URL del enlace)
    * `Favorite` (Opcional, `TRUE` o `FALSE` para marcar como favorito)
    * `Order` (Opcional, número para ordenar los enlaces)

* **Hoja `UpcomingEvents`:** (Usada si `CALENDAR_ID_SHEET` en `Config` apunta a esta hoja)
    * `Event Title` (o `Title`)
    * `Start Date` (o `StartDate`) - Formato de fecha reconocido por Google Sheets.
    * `End Date` (o `EndDate`) - Opcional. Formato de fecha.
    * `All Day Event` (o `AllDayEvent`) - Opcional (`TRUE` o `FALSE`).

* **Hoja `Comunicados`:**
    * `Fecha` (Fecha del comunicado, ej: `YYYY-MM-DD`, `DD/MM/YYYY`)
    * `Mensaje` (o `Comunicado`) (Texto del comunicado)
    * `URL` (Opcional, si el comunicado debe enlazar a una URL)

### 3. Configurar Google Apps Script

1.  Abre tu Hoja de Cálculo de Google.
2.  Ve a `Extensiones` > `Apps Script`. Se abrirá el editor de Apps Script.
3.  **Archivo `Code.gs`:** (Si ya existe, puedes actualizarlo. Si no, créalo)
    * Borra cualquier contenido existente.
    * Copia **todo** el contenido del archivo `Code.gs` proporcionado y pégalo en el editor.
    * **IMPORTANTE:** Reemplaza el valor de la constante `SPREADSHEET_ID` al principio del archivo con el ID real de tu Hoja de Cálculo.
        ```javascript
        const SPREADSHEET_ID = 'TU_ID_DE_HOJA_DE_CALCULO_AQUI';
        ```
4.  **Archivo `Index.html`:**
    * En el editor de Apps Script, haz clic en el `+` junto a "Archivos" y selecciona `HTML`.
    * Nombra el archivo `Index.html` (respetando mayúsculas y minúsculas) y presiona Enter. (Si ya existe, actualízalo).
    * Borra cualquier contenido existente.
    * Copia **todo** el contenido del archivo `Index.html` proporcionado y pégalo.
5.  Guarda los cambios en ambos archivos (icono de disquete o `Ctrl+S` / `Cmd+S`).

### 4. Desplegar la Aplicación Web

1.  En el editor de Apps Script, haz clic en el botón `Desplegar` (arriba a la derecha).
2.  Selecciona `Nuevo despliegue`.
3.  Junto a "Seleccionar tipo", haz clic en el icono del engranaje y elige `Aplicación web`.
4.  En el cuadro de diálogo:
    * **Descripción:** Puedes poner algo como "Scrum App vX.Y".
    * **Ejecutar como:** Selecciona `Yo ([tu dirección de correo])`.
    * **Quién tiene acceso:**
        * `Cualquier usuario con una cuenta de Google` (recomendado si es para un equipo dentro de una organización).
        * `Cualquier usuario` (si necesitas que sea accesible públicamente sin inicio de sesión de Google, aunque el script intenta obtener el email del usuario).
5.  Haz clic en `Desplegar`.
6.  **Autorización:** La primera vez que despliegues, o si el script requiere nuevos permisos, Google te pedirá que autorices los permisos que el script necesita (acceder a Google Sheets, enviar emails si configuraste la notificación de impedimentos, etc.). Sigue los pasos:
    * Haz clic en `Autorizar acceso`.
    * Elige tu cuenta de Google.
    * Puede que veas una advertencia de "Google no ha verificado esta aplicación". Haz clic en `Configuración avanzada` (o similar) y luego en `Ir a [Nombre de tu proyecto] (no seguro)`.
    * Revisa los permisos y haz clic en `Permitir`.
7.  Una vez desplegado, se te proporcionará una **URL de la aplicación web**. Esta es la URL que usarás para acceder a tu Scrum App. Cópiala. Si estás actualizando un despliegue existente, asegúrate de usar la URL de la versión más reciente.

## Configuración Avanzada (Hoja `Config`)

La hoja `Config` te permite personalizar varios aspectos de la aplicación sin modificar el código:

| Key                             | Descripción                                                                                                | Ejemplo de Value                             |
| :------------------------------ | :--------------------------------------------------------------------------------------------------------- | :------------------------------------------- |
| `APP_TITLE`                     | El título que se muestra en la aplicación y en la pestaña del navegador.                                    | `Dashboard del Equipo Alfa`                  |
| `APP_SUBTITLE`                  | El subtítulo que se muestra debajo del título principal.                                                     | `Seguimiento Ágil Q3`                        |
| `HELP_CHAT_URL`                 | La URL a la que enlazará el botón flotante de "Ayuda".                                                      | `https://chat.google.com/room/ABCXYZ`        |
| `TEAM_MESSAGE_CARD_TITLE`       | El título de la tarjeta de "Comunicados del Equipo".                                                        | `Últimos Comunicados`                        |
| `IMPEDIMENT_NOTIFICATION_EMAIL` | La dirección de correo a la que se enviarán notificaciones de nuevos impedimentos. Dejar en blanco para no enviar. | `jefe.de.proyecto@ejemplo.com`             |
| `SPRINT_END_DATE`               | Fecha de finalización del sprint actual para el contador. Formato: `YYYY-MM-DDTHH:MM:SS` o `YYYY-MM-DD`.      | `2025-05-30`                                 |
| `SPRINT_INFO_YEAR`              | Año del sprint (ej: `2024`).                                                                               | `2024`                                       |
| `SPRINT_INFO_QUARTER`           | Trimestre del sprint (ej: `Q2`).                                                                           | `Q2`                                         |
| `SPRINT_INFO_NUMBER_IN_QUARTER` | Número del sprint dentro del trimestre (ej: `3`).                                                          | `3`                                          |
| `CALENDAR_ID_SHEET`             | Nombre de la hoja de cálculo que contiene los eventos próximos (ej: `UpcomingEvents`).                       | `UpcomingEvents`                             |
| `MAX_CALENDAR_EVENTS`           | Número máximo de eventos próximos a mostrar.                                                                 | `5`                                          |
| `TEAM_MOOD_DEFAULT_EMOJI`       | Emoji por defecto para el mood del equipo si no hay datos.                                                   | `🚀`                                         |
| `APP_DATA_SOURCE_URL`           | **URL de la hoja de cálculo principal para el enlace en el pie de página.** | `https://docs.google.com/spreadsheets/d/ABC...` |
| `UPCOMING_EVENTS_EDIT_URL`      | **URL para editar directamente los datos de Próximos Eventos (ej. enlace a la hoja).** | `https://docs.google.com/spreadsheets/d/ABC...#gid=123` |
| `COMMUNICATIONS_EDIT_URL`       | **URL para editar directamente los datos de Comunicados.** | `https://docs.google.com/spreadsheets/d/ABC...#gid=456` |
| `QUICKLINKS_EDIT_URL`           | **URL para editar directamente los datos de Enlaces Útiles.** | `https://docs.google.com/spreadsheets/d/ABC...#gid=789` |

## Uso de la Aplicación

* Abre la URL de la aplicación web obtenida durante el despliegue.
* Los datos se cargarán desde tu Google Sheet.
* Usa los filtros para refinar la vista de Features e Historias Pendientes, incluyendo el filtro de "Actividad".
* Interactúa con las tarjetas de Mood, Impedimentos, Comunicados, etc.
* Usa los buscadores en "Enlaces Útiles" y "Comunicados del Equipo" para encontrar información rápidamente.

## Solución de Problemas

* **La aplicación muestra un error al cargar:**
    * Verifica que el `SPREADSHEET_ID` en `Code.gs` sea correcto.
    * Asegúrate de que los nombres de todas las hojas en tu Google Sheet coincidan exactamente con los definidos en `Code.gs` (sensible a mayúsculas y minúsculas).
    * Revisa que los encabezados de las columnas en cada hoja sean los correctos.
    * Abre el editor de Apps Script y ve a "Ejecuciones" para ver los logs del servidor. Cualquier error en `getAllSheetData` o funciones relacionadas aparecerá ahí.
    * Abre la consola de desarrollador de tu navegador (`Ctrl+Shift+J` o `Cmd+Opt+J`) para ver errores del lado del cliente.
* **Los datos no se actualizan:** Asegúrate de que los datos en tu Google Sheet estén guardados. La aplicación carga los datos cada vez que se abre o actualiza la página.
* **Permisos:** Si realizas cambios significativos en el script que requieran nuevos permisos (ej: usar un nuevo servicio de Google), puede que necesites volver a autorizar la aplicación desplegándola de nuevo.

## Changelog

### [Fecha de la Actualización - ej: 2024-05-22]
* **Nueva Característica: Tabla de Historias de Usuario Pendientes**
    * Se ha añadido una nueva tabla debajo de los filtros globales para mostrar específicamente las Historias de Usuario que no están en estado "Deployed".
    * **Columnas mostradas:** Nombre (texto plano, sin URL), Seguimiento, ID Feature (con enlace a la URL de la Feature si está disponible), ID Historia (con enlace a JiraLink si está disponible), Sprint, Estatus (con badge de color) y Prioridad (con tag).
    * Las columnas tienen un ancho distribuido uniformemente.
    * La tabla tiene un alto máximo predefinido (aproximadamente para 4-5 filas visibles) con una barra de desplazamiento vertical si el contenido excede este alto, permitiendo ver todas las historias pendientes sin alargar excesivamente la página.
    * Esta tabla se actualiza dinámicamente con los filtros globales aplicados (Q, Sprint, Seguimiento, Estatus, y término de búsqueda general). El filtro de "Actividad" en "Feature" o "Dependency" ocultará esta tabla ya que se enfoca en Historias.
* **Mejoras en `Code.gs`**:
    * Se ha optimizado la obtención de datos para `nonDeployedUserStories` para incluir `FeatureURL` (URL de la Feature padre) para poder enlazar el ID de la Feature en la nueva tabla.
    * Se ha revisado el mapeo de cabeceras en `getSheetDataAsObjects` para asegurar la correcta asignación de la columna "URL" de la hoja "Features".
* **Actualizaciones en `Index.html`**:
    * Se ha añadido la estructura HTML y los estilos CSS para la nueva tabla de "Historias de Usuario Pendientes", incluyendo el `max-height` y `overflow-y: auto` para el scroll.
    * Se ha modificado la función `renderPendingUserStories` para construir las filas de la tabla con las nuevas columnas, el orden especificado y los enlaces correspondientes.
    * Se han ajustado los anchos de las columnas en el `<thead>` de la nueva tabla para una distribución uniforme.
    * Se han actualizado los encabezados de columna en la tabla de Features y sus tablas anidadas para usar "Seguimiento" en lugar de "Responsable" para consistencia con la terminología del usuario.
