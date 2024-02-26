---
lab:
  title: Configuración del clasificador semántico
---

# Configuración del clasificador semántico

> **Nota** Para completar este laboratorio, necesitará una [suscripción de Azure](https://azure.microsoft.com/free?azure-portal=true) en la que tenga acceso de administrador. Para este ejercicio también es necesario el servicio **Búsqueda de Azure AI** con un nivel facturable.

En este ejercicio, agregará el clasificador semántico a un índice y lo usará en una consulta.

## Habilitación del clasificador semántico

1. Abra Azure Portal e inicie sesión.
1. Seleccione **Todos los recursos** y seleccione el servicio de búsqueda.
1. En el panel de navegación, seleccione **Clasificador semántico (versión preliminar)**.
1. En **Disponibilidad**, en la opción **Gratis**, seleccione **Seleccionar plan**.

![Captura de pantalla del cuadro de diálogo del clasificador semántico.](../media/semantic-search/semanticsearch.png)

## Importación de un índice de ejemplo

1. Vuelva a la página **Información general** del servicio de búsqueda.
1. Seleccione **Importar datos**.

    ![Captura de pantalla del botón Importar datos.](../media/semantic-search/importdata.png)

1. En **Origen de datos**, seleccione **Ejemplos**.
1. Seleccione **hotels-sample** y, posteriormente, **Siguiente: Agregar aptitudes cognitivas (opcional)**.
1. Seleccione **Ir a: Personalizar índice de destino**.
1. Seleccione **Siguiente: Crear indizador**.
1. Seleccione **Submit** (Enviar).

## Configuración de la clasificación semántica

Una vez que tenga habilitados un índice de búsqueda y el clasificador semántico, puede configurar la clasificación semántica. Necesita un cliente de búsqueda que admita API en versión preliminar en la solicitud de consulta. Puede usar el Explorador de búsqueda en Azure Portal, la aplicación Postman, el SDK de Azure para .NET o el SDK de Azure para Python. En este ejercicio, usará el Explorador de búsqueda en Azure Portal.

Para configurar la clasificación semántica, siga estos pasos:

1. En la barra de navegación, en **Administración de búsquedas**, seleccione **Índices**.

    ![Captura de pantalla del botón Índices.](../media/semantic-search/indexes.png)

1. Seleccione el índice.
1. Seleccione **Configuraciones semánticas** y **Agregar configuración semántica**.
1. En **Nombre**, escriba **hotels-conf**.
1. En **Campo de título**, seleccione **HotelName**.
1. En **Campos de contenido**, en **Nombre del campo**, seleccione **Descripción**.
1. Repita el paso anterior para los siguientes campos:
    - **Categoría**
    - **Dirección/Ciudad**
1. En **Campos de palabra clave**, en **Nombre de campo**, seleccione **Etiquetas**.
1. Seleccione **Guardar**.
1. En la página de índice, seleccione **Guardar**.
1. Seleccione **Explorador de búsqueda**.
1. Seleccione **Ver** y, posteriormente, **Vista JSON**.
1. En el editor de consultas JSON, escriba el texto siguiente:

    ```json
        {
         "queryType": "semantic",
         "queryLanguage" : "en-us",
         "search": "all hotels near the water" , 
         "semanticConfiguration": "hotels-conf" , 
         "searchFields": "",
         "speller": "lexicon" , 
         "answers": "extractive|count-3",
         "count": true
        }
    ```

1. Seleccione **Buscar**.
1. Revise los resultados de la consulta.

## Limpieza

Si ya no necesita el servicio Búsqueda de Azure AI, debe eliminar el recurso de la suscripción de Azure, para reducir costes.

>**Nota:** Al eliminar el servicio Búsqueda de Azure AI, garantiza que no se cobre por recursos en la suscripción. Sin embargo, se le cobrará un importe reducido por el almacenamiento de datos, siempre que el almacenamiento exista en la suscripción. Si ha terminado de explorar el servicio Cognitive Search, puede eliminar el servicio Cognitive Search y los recursos asociados. Sin embargo, si planea completar cualquier otro laboratorio de esta serie, tendrá que volver a crearla.
> Para eliminar los recursos:
> 1. En [Azure Portal](https://portal.azure.com?azure-portal=true ), en la página **Grupos de recursos**, abra el grupo de recursos que haya especificado al crear el servicio Cognitive Search.
> 1. Haga clic en **Eliminar grupo de recursos**, escriba el nombre del grupo de recursos para confirmar que quiere eliminarlo y seleccione **Eliminar**.
