---
lab:
  title: Agregación a un índice mediante la API de inserción
---

# Agregación a un índice mediante la API de inserción

Quiere explorar cómo crear un índice de Búsqueda de Azure AI y cargar documentos en ese índice mediante código de C#.

En este ejercicio, clonará una solución de C# existente y la ejecutará para resolver el tamaño de lote óptimo para cargar documentos. A continuación, usará este tamaño de lote y cargará documentos de forma eficaz mediante un enfoque de subprocesos.

> **Nota**: Para completar este ejercicio, necesitará una suscripción a Microsoft Azure. Si aún no tiene una, puede suscribirse para solicitar una prueba gratuita en [https://azure.com/free](https://azure.com/free?azure-portal=true).

## Configuración de los recursos de Azure

Para ahorrar tiempo, seleccione esta plantilla de Azure Resource Manager para crear los recursos que necesitará más adelante en el ejercicio:

1. [Implementación de recursos en Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F07-exercise-add-to-index-use-push-api%2Fazuredeploy.json): seleccione este vínculo para crear los recursos de Azure AI.
    ![Captura de pantalla de las opciones que se muestran al implementar recursos en Azure](../media/07-media/deploy-azure-resources.png)
1. En **Grupo de recursos**, seleccione **Crear nuevo** y asígnele el nombre **cog-search-language-exe**.
1. En **Región**, seleccione una [región admitida](/azure/ai-services/language-service/custom-text-classification/service-limits#regional-availability) cercana a usted.
1. El **Prefijo de recurso** debe ser único a nivel global; escriba un prefijo aleatorio con números y caracteres en minúsculas como, por ejemplo, **acs118245**.
1. En **Ubicación**, seleccione la misma región que ha elegido anteriormente.
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.
1. Una vez finalizada la implementación, seleccione **Ir al grupo de recursos** para ver todos los recursos que ha creado.

    ![Captura de pantalla que muestra todos los recursos de Azure implementados](../media/07-media/azure-resources-created.png)

## Copia de la información de la API de REST del servicio de Búsqueda de Azure AI

1. En la lista de recursos, seleccione el servicio de búsqueda que ha creado. En el ejemplo anterior, **acs118245-search-service**.
1. Copie el nombre del servicio de búsqueda en un archivo de texto.

    ![Captura de pantalla de la sección de claves de un servicio de búsqueda](../media/07-media/search-api-keys-exercise-version.png)
1. A la izquierda, seleccione **Claves** y copie la **Clave de administrador principal** en el mismo archivo de texto.

## Clonación del repositorio en Cloud Shell

Vas a desarrollar el código mediante Cloud Shell desde Azure Portal. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: si ya has clonado el repositorio **mslearn-knowledge-mining** recientemente, puedes omitir esta tarea. De lo contrario, sigue estos pasos para clonarlo en el entorno de desarrollo.

1. En Azure Portal, usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. En el panel de PowerShell, escribe los siguientes comandos para clonar el repo de GitHub para este ejercicio:

    ```
    rm -r mslearn-knowledge-mining -f
    git clone https://github.com/microsoftlearning/mslearn-knowledge-mining mslearn-knowledge-mining
    ```

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    ```
   cd mslearn-knowledge-mining/Labfiles/07-exercise-add-to-index-use-push-api/OptimizeDataIndexing
    ```

## Configuración de la aplicación

1. Con el comando `ls`, puedesver el contenido de la carpeta **OptimizeDataIndexing**. Ten en cuenta que contiene un archivo `appsettings.json` para las opciones de configuración.

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code appsettings.json
    ```

    El archivo se abre en un editor de código.

    ![Captura de pantalla que muestra el contenido del archivo appsettings.json](../media/07-media/update-app-settings.png)

1. Pegue el nombre del servicio de búsqueda y la clave de administrador principal.

    ```json
    {
      "SearchServiceUri": "https://acs118245-search-service.search.windows.net",
      "SearchServiceAdminApiKey": "YOUR_SEARCH_SERVICE_KEY",
      "SearchIndexName": "optimize-indexing"
    }
    ```

    El archivo de configuración debe tener un aspecto similar a lo que se muestra más arriba.
   
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.
1. En el terminal, escribe `dotnet run` y presiona **Entrar**.

    ![Captura de pantalla que muestra la aplicación que se ejecuta en VS Code con una excepción](../media/07-media/debug-application.png)

    La salida muestra que, en este caso, el tamaño de lote con mejor rendimiento es de 900 documentos con la velocidad de transferencia más alta (MB / segundo).
   
    >**Nota**: los valores de velocidad de transferencia pueden ser diferentes de los que se muestran en la captura de pantalla. Sin embargo, el tamaño del lote con mejor rendimiento debe ser el mismo. 

## Edición del código para implementar subprocesos y una estrategia de retroceso y reintento

Hay código marcado que está listo para cambiar la aplicación para que use subprocesos para cargar documentos en el índice de búsqueda.

1. Escribe el siguiente comando para abrir el archivo de código para la aplicación cliente:

    ```
   code Program.cs
    ```

1. Comente las líneas 38 y 39 de esta manera:

    ```csharp
    //Console.WriteLine("{0}", "Finding optimal batch size...\n");
    //await TestBatchSizesAsync(searchClient, numTries: 3);
    ```

1. Quite la marca de comentario de las líneas de 41 a 49.

    ```csharp
    long numDocuments = 100000;
    DataGenerator dg = new DataGenerator();
    List<Hotel> hotels = dg.GetHotels(numDocuments, "large");

    Console.WriteLine("{0}", "Uploading using exponential backoff...\n");
    await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8);

    Console.WriteLine("{0}", "Validating all data was indexed...\n");
    await ValidateIndexAsync(indexClient, indexName, numDocuments);
    ```

    El código que controla el tamaño del lote y el número de subprocesos es `await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8)`. El tamaño del lote es 1000, y hay ocho subprocesos.

    ![Captura de pantalla que muestra todo el código editado](../media/07-media/thread-code-ready.png)

    El código debe tener un aspecto parecido a lo que se muestra más arriba.

1. Guarda los cambios.
1. Seleccione el terminal y presione cualquier tecla para finalizar el proceso en ejecución si aún no lo ha hecho.
1. Ejecute `dotnet run` en el terminal.

    La aplicación iniciará ocho subprocesos y, a continuación, a medida que cada subproceso termine de escribir un mensaje nuevo en la consola:

    ```powershell
    Finished a thread, kicking off another...
    Sending a batch of 1000 docs starting with doc 57000...
    ```

    Después de cargar 100 000 documentos, la aplicación escribe un resumen (esto puede tardar en completarse):

    ```powershell
    Ended at: 9/1/2023 3:25:36 PM
    
    Upload time total: 00:01:18:0220862
    Upload time per batch: 780.2209 ms
    Upload time per document: 0.7802 ms
    
    Validating all data was indexed...
    
    Waiting for service statistics to update...
    
    Document Count is 100000
    
    Waiting for service statistics to update...
    
    Index Statistics: Document Count is 100000
    Index Statistics: Storage Size is 71453102
    
    ``````

Explore el código del procedimiento `TestBatchSizesAsync` para ver cómo el código prueba el rendimiento del tamaño del lote.

Explore el código del procedimiento `IndexDataAsync` para ver cómo el código administra los subprocesos.

Explore el código de `ExponentialBackoffAsync` para ver cómo el código implementa una estrategia de retroceso exponencial y reintento.

Puede buscar y comprobar que los documentos se han agregado al índice en Azure Portal.

![Captura de pantalla que muestra el índice de búsqueda con 100 000 documentos](../media/07-media/check-search-service-index.png)

## Limpieza

Ahora que ha completado el ejercicio, elimine todos los recursos que ya no necesita. Comience con el código clonado en la máquina. A continuación, elimine los recursos de Azure.

1. En **Azure Portal**, seleccione Grupos de recursos.
1. Seleccione el grupo de recursos que ha creado para este ejercicio.
1. Seleccione **Eliminar grupo de recursos**. 
1. Confirme la eliminación y seleccione **Eliminar**.
1. Seleccione los recursos que no necesita y, a continuación, seleccione **Eliminar**.
