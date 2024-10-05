---
lab:
  title: Uso de la API de REST para ejecutar consultas de vector de búsqueda
---

# Uso de la API de REST para ejecutar consultas de vector de búsqueda

En este ejercicio configurará el proyecto, creará un índice, cargará los documentos y ejecutará consultas.

Necesitará lo siguiente para realizar correctamente este ejercicio:

- La aplicación [Postman](https://www.postman.com/downloads/)
- Una suscripción de Azure
- Servicio Azure AI Search
- La colección de muestras de Postman ubicadas en este repositorio: *Vector-Search-Quickstart.postman_collection v1.0 json*.

> **Nota** Puede encontrar más información sobre la aplicación Postman [aquí](https://learn.microsoft.com/en-us/azure/search/search-get-started-rest) si es necesario.

## Configuración del proyecto

En primer lugar, configure el proyecto realizando los pasos siguientes:

1. Anote la **dirección URL** y la **clave** del servicio de Búsqueda de Azure AI.

    ![Ilustración de la ubicación del nombre del servicio y las claves.](../media/vector-search/search keys.png)

1. Descargue la [colección de muestras de Postman](https://github.com/MicrosoftLearning/mslearn-knowledge-mining/blob/main/Labfiles/10-vector-search/Vector%20Search.postman_collection%20v1.0.json).
1. Abra Postman e importe la colección seleccionando el botón **Importar**; y arrástrela y coloque la carpeta de recopilación en el cuadro.

    ![Imagen del cuadro de diálogo Importar](../media/vector-search/import.png)

1. Seleccione el botón **Bifurcar** para crear una bifurcación de la colección y agregar un nombre único.
1. Haga clic con el botón derecho en el nombre de la colección y seleccione **Editar**.
1. Seleccione la pestaña **Variables** y escriba los valores siguientes mediante el servicio de búsqueda y los nombres de índice del servicio de Búsqueda de Azure AI:

    ![Diagrama que muestra un ejemplo de configuración de variables](../media/vector-search/variables.png)

1. Para guardar los cambios, seleccione el botón **Guardar**.

Está listo para enviar sus solicitudes al servicio de Búsqueda de Azure AI.

## Crear un índice

A continuación, cree el índice en Postman:

1. Seleccione **PUT Crear/actualizar índice** en el menú lateral.
1. Actualice la dirección URL con los valores de **search-service-name**, **index-name** y **api-version** (nombre del servicio de búsqueda, nombre del índice y versión de la API) que anotó anteriormente.
1. Seleccione la pestaña **Cuerpo** para ver la respuesta.
1. Establezca **index-name** con el valor de nombre de índice de la dirección URL y seleccione **Enviar**.

Debería ver un código de estado de tipo **200** que indica una solicitud correcta.

## Cargar documentos

Hay 108 documentos incluidos en la solicitud Cargar documentos; cada uno tiene un conjunto completo de incrustaciones para los campos **titleVector** y **contentVector**.

1. Seleccione **POST Cargar documentos** en el menú lateral.
1. Actualice la dirección URL con **search-service-name**, **index-name** y **api-version** como lo hizo anteriormente.
1. Seleccione la pestaña **Cuerpo** para ver la respuesta y seleccione **Enviar**.

Debería ver un código de estado de tipo **200** que muestra que la solicitud se realizó correctamente.

## Ejecutar consultas

1. Ahora intente ejecutar las siguientes consultas en el menú lateral. Para ello, asegúrese de actualizar la dirección URL cada vez como lo hizo anteriormente y envíe una solicitud seleccionando **Enviar**:

    - Búsqueda de un solo vector
    - Búsqueda de un solo vector con filtro
    - Búsqueda híbrida simple
    - Búsqueda híbrida simple con filtro
    - Búsqueda entre campos
    - Búsqueda de varias consultas

1. Seleccione la pestaña **Cuerpo** para ver la respuesta y ver los resultados.

Debería ver un código de estado de tipo **200** si se obtiene una solicitud correcta.

### Limpieza

Ahora que ha completado el ejercicio, elimine todos los recursos que ya no necesita. Comience con el código clonado en la máquina. A continuación, elimine los recursos de Azure.
