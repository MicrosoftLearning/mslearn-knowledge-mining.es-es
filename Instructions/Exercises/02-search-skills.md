---
lab:
  title: "Creación de una aptitud personalizada para Búsqueda de Azure\_AI"
  module: Module 12 - Creating a Knowledge Mining Solution
---

# Creación de una aptitud personalizada para Búsqueda de Azure AI

Búsqueda de Azure AI usa una canalización de enriquecimiento de aptitudes de IA para extraer campos generados mediante inteligencia artificial de documentos e incluirlos en un índice de búsqueda. Hay un conjunto completo de aptitudes integradas que puede usar, pero si tiene un requisito específico que no cumplen estas aptitudes, puede crear una aptitud personalizada.

En este ejercicio, creará una aptitud personalizada que tabula la frecuencia de las palabras individuales de un documento para generar una lista de las cinco palabras más usadas y agregarla a una solución de búsqueda para Margie's Travel, una agencia de viajes ficticia.

## Preparación para desarrollar una aplicación en Visual Studio Code

Desarrollará la aplicación de búsqueda con Visual Studio Code. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-knowledge-mining**, ábralo en Visual Studio Code. De lo contrario, siga estos pasos para clonarlo en el entorno de desarrollo.

1. Inicie Visual Studio Code.
1. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` en una carpeta local (no importa qué carpeta).
1. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
1. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## Creación de recursos de Azure

> **Nota**: Si ha completado previamente el ejercicio **[Creación de una solución de Búsqueda de Azure AI](01-azure-search.md)** y todavía tiene esos recursos de Azure en su suscripción, puede omitir esta sección y empezar en la sección **Creación de una solución de búsqueda**. De lo contrario, siga los pasos que se indican a continuación para aprovisionar los recursos de Azure necesarios.

1. En un explorador web, abra Azure Portal en `https://portal.azure.com` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
2. En la barra de búsqueda superior, busca *Servicios de Azure AI*, selecciona **Cuenta de varios servicios de Azure AI** y crea un recurso de cuenta de varios servicios de Azure AI con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *Elegir entre regiones disponibles geográficamente cercanas*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0
1. Una vez implementado, vaya al recurso y, en la página **información general**, anote el **identificador de suscripción** y la **ubicación**. Los necesitará, junto con el nombre del grupo de recursos, en los pasos posteriores. 
1. En Visual Studio Code, expanda la carpeta **Labfiles/02-search-skill** y seleccione **setup.cmd**. Usará este script por lotes para ejecutar los comandos de la interfaz de la línea de comandos (CLI) de Azure necesarios para crear los recursos de Azure que necesita.
1. Haga clic con el botón derecho en la carpeta **02-search-skill** y seleccione **Abrir en terminal integrado**.
1. En el panel de terminal, escriba el siguiente comando para establecer una conexión autenticada con su suscripción de Azure.

    ```powershell
    az login --output none
    ```

8. Cuando se le solicite, seleccione o inicie sesión en su suscripción de Azure. Después, vuelva a Visual Studio Code y espere a que se complete el proceso de inicio de sesión.
9. Ejecute el siguiente comando para enumerar las ubicaciones de Azure.

    ```powershell
    az account list-locations -o table
    ```

10. En la salida, busque el valor de **Nombre** que corresponde a la ubicación del grupo de recursos (por ejemplo, para *Este de EE. UU.* , el nombre correspondiente es *eastus*).
11. En el script **setup.cmd**, modifique las declaraciones de las variables **subscription_id**, **resource_group** y **location** con los valores adecuados del id. de suscripción, el nombre del grupo de recursos y el nombre de la ubicación. A continuación, guarde los cambios.
12. En el terminal de la carpeta **02-search-skill**, escriba el siguiente comando para ejecutar el script:

    ```powershell
    ./setup
    ```

    > **Nota**: Si se produce un error en el script, asegúrese de guardarlo con los nombres de variable correctos e inténtelo de nuevo.

13. Cuando se complete el script, revise la salida que se muestra y anote la siguiente información de los recursos de Azure (necesitará estos valores más adelante):
    - Nombre de la cuenta de almacenamiento
    - Cadena de conexión de almacenamiento
    - Punto de conexión del servicio de búsqueda
    - Clave de administración del servicio de búsqueda
    - Clave de consulta del servicio de búsqueda

14. En Azure Portal, actualice el grupo de recursos y compruebe que contiene la cuenta de Azure Storage, el recurso de Servicios de Azure AI y el recurso de Búsqueda de Azure AI.

## Creación de una solución de búsqueda

Ahora que tiene los recursos de Azure necesarios, puede crear una solución de búsqueda que conste de los siguientes componentes:

- Un **origen de datos** que haga referencia a los documentos del contenedor de almacenamiento de Azure.
- Un **conjunto de aptitudes** que defina una canalización de enriquecimiento de aptitudes para extraer campos generados mediante inteligencia artificial de los documentos.
- Un **índice** que defina un conjunto de registros de documentos en los que se pueden realizar búsquedas.
- Un **indexador** que extraiga los documentos del origen de datos, aplique el conjunto de aptitudes y rellene el índice.

En este ejercicio, usarás la interfaz de REST de Búsqueda de Azure AI para crear estos componentes mediante el envío de solicitudes JSON.

1. En Visual Studio Code, en la carpeta **02-search-skill**, expande la carpeta **create-search** y selecciona **data_source.json**. Este archivo contiene una definición JSON para un origen de datos denominado **margies-custom-data**.
2. Reemplaza el marcador de posición **YOUR_CONNECTION_STRING** por la cadena de conexión de la cuenta de almacenamiento de Azure, que debe ser similar a la siguiente:

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *Puedes encontrar la cadena de conexión en la página **Claves de acceso** de tu cuenta de almacenamiento en Azure Portal.*

3. Guarda y cierra el archivo JSON actualizado.
4. En la carpeta **create-search,**, abre **skillset.json**. Este archivo contiene una definición JSON para un conjunto de aptitudes denominado **margies-custom-skillset**.
5. En la parte superior de la definición del conjunto de aptitudes, en el elemento **cognitiveServices**, reemplaza el marcador de posición **YOUR_AI_SERVICES_KEY** por cualquiera de las claves de los recursos de Servicios de Azure AI.

    *Puede encontrar las claves en la página **Keys and Endpoint** (Claves y punto de conexión) del recurso de Servicios de Azure AI en Azure Portal.*

6. Guarda y cierra el archivo JSON actualizado.
7. En la carpeta **create-search,**, abre **index.json**. Este archivo contiene una definición JSON para un índice denominado **margies-custom-index**.
8. Revisa el archivo JSON del índice y ciérralo sin realizar ningún cambio.
9. En la carpeta **create-search,**, abre **indexer.json**. Este archivo contiene una definición JSON para un indexador denominado **margies-custom-indexer**.
10. Revisa el archivo JSON del indexador y ciérralo sin realizar ningún cambio.
11. En la carpeta **create-search,**, abre **create-search.cmd**. Este script por lotes usa la utilidad cURL para enviar las definiciones JSON a la interfaz de REST del recurso de Búsqueda de Azure AI.
12. Reemplaza los marcadores de posición de variables **YOUR_SEARCH_URL** y **YOUR_ADMIN_KEY** por la dirección **URL** y una de las **claves de administración** del recurso de Búsqueda de Azure AI.

    *Puedes encontrar estos valores en las páginas **Información general** y **Claves** del recurso de Búsqueda de Azure AI en Azure Portal.*

13. Guarda el archivo por lotes actualizado.
14. Haz clic con el botón derecho en la carpeta **create-search** y selecciona **Open in Integrated Terminal**.
15. En el panel del terminal de la carpeta **create-search**, escribe el siguiente comando para ejecutar el script por lotes.

    ```powershell
    ./create-search
    ```

16. Cuando se complete el script, en Azure Portal, en la página del recurso de Búsqueda de Azure AI, selecciona la página **Indizadores** y espera a que se complete el proceso de indexación.

    *Puedes seleccionar **Actualizar** para supervisar el progreso de la operación de indexación. Puede tardar alrededor de un minuto en completarse.*

## Búsqueda en el índice

Ahora que tienes un índice, puedes realizar búsquedas en él.

1. En la parte superior del panel del recurso de Búsqueda de Azure AI, selecciona **Explorador de búsqueda**.
2. En el Explorador de búsqueda, en el cuadro **Cadena de consulta**, escribe la siguiente cadena de consulta y, a continuación, selecciona **Buscar**.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    Esta consulta recupera los valores de **url**, **sentiment** y **keyphrases** de todos los documentos que mencionan *London*, cuyo autor es *Reviewer* y que tienen una etiqueta de **opinión** positiva (es decir, reseñas positivas que mencionan London).

## Creación de una función de Azure para una aptitud personalizada

La solución de búsqueda incluye una serie de aptitudes de IA integradas que enriquecen el índice con información de los documentos, como las puntuaciones de opinión y las listas de frases clave que se han visto en la tarea anterior.

Puedes mejorar aún más el índice mediante la creación de aptitudes personalizadas. Por ejemplo, puede ser útil identificar las palabras que se usan con más frecuencia en cada documento, pero ninguna aptitud integrada ofrece esta funcionalidad.

Para implementar la funcionalidad de recuento de palabras como una aptitud personalizada, deberás crear una función de Azure en el lenguaje que prefiera.

> **Nota**: en este ejercicio, crearás una función Node.JS sencilla mediante las funcionalidades de edición de código de Azure Portal. En una solución de producción, lo normal es usar un entorno de desarrollo como Visual Studio Code para crear una aplicación de funciones en el lenguaje preferido (por ejemplo, C#, Python, Node.JS o Java) y publicarla en Azure como parte de un proceso de DevOps.

1. En Azure Portal, en la página **Inicio**, crea un nuevo recurso **Aplicación de funciones** con la siguiente configuración:
    - **Plan de hospedaje**: consumo
    - **Suscripción**: *tu suscripción*
    - **Grupo de recursos**: *el mismo grupo de recursos que el recurso de Búsqueda de Azure AI*
    - **Nombre de la aplicación de funciones**: *un nombre único*
    - **Pila en tiempo de ejecución**: Node.js.
    - **Versión**: 20 LTS
    - **Región**: *la misma región que el recurso de Búsqueda de Azure AI*
    - **Sistema operativo**: Windows

>**Nota**: Si la región en la que creaste el recurso de Búsqueda de Azure AI no tiene cuota disponible para implementar el recurso de aplicación de funciones, puedes elegir otra región en la que la suscripción tenga cuota disponible para la implementación.

2. Espera a que se complete la implementación y, a continuación, ve al recurso de aplicación de funciones implementado.
3. En la página **Información general**, selecciona **Crear función** de la parte inferior de la página para crear una nueva función con la siguiente configuración:
    - **Seleccione una plantilla**
        - **Plantilla**: Desencadenador HTTP    
    - **Detalles de plantilla**:
        - **Nombre de la función**: wordcount
        - **Nivel de autorización**: Función

    > **Nota**: si recibes un error de función de creación, actualiza la página y el recurso debería crearse según lo previsto.

4. Espere a que se cree la función *recuento de palabras*. A continuación, en su página, seleccione la pestaña **Código y prueba**.
5. Reemplace el código de la función predeterminada por el siguiente:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. Guarde la función y, a continuación, abra el panel **Probar/ejecutar**.
7. En el panel **Probar/ejecutar**, reemplace el **cuerpo** existente por el siguiente código JSON, que refleja el esquema esperado por una aptitud de Búsqueda de Azure AI en la que los registros que contienen datos de uno o varios documentos se envían para su procesamiento:

    ```json
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```

8. Haga clic en **Ejecutar** y analice el contenido de la respuesta HTTP que devuelve la función. El contenido refleja el esquema que espera Búsqueda de Azure AI al consumir una aptitud, en el que se devuelve una respuesta para cada documento. En este caso, la respuesta está formada por hasta 10 términos en cada documento en orden descendente según la frecuencia con la que aparecen:

    ```json
    {
        "values": [
        {
            "recordId": "a1",
            "data": {
                "text": [
                "tiger",
                "burning",
                "bright",
                "darkness",
                "night"
                ]
            }
        },
        {
            "recordId": "a2",
            "data": {
                "text": [
                    "rain",
                    "spain",
                    "stays",
                    "mainly",
                    "plains",
                    "thats",
                    "youll",
                    "find"
                ]
            }
        }
        ]
    }
    ```

9. Cierre el panel **Probar/ejecutar** y, en el panel de la función **wordcount**, haga clic en **Obtener la dirección URL de la función**. Después, copie la dirección URL de la clave predeterminada en el portapapeles, la necesitará en el siguiente procedimiento.

## Adición de la aptitud personalizada a la solución de búsqueda

Ahora tiene que incluir la función como una aptitud personalizada en el conjunto de aptitudes de la solución de búsqueda y asignar los resultados que genera a un campo del índice. 

1. En Visual Studio Code, en la carpeta **02-search-skill/update-search**, abra el archivo **update-skillset.json**. Este archivo contiene la definición JSON de un conjunto de aptitudes.
2. Revise la definición del conjunto de aptitudes. Incluye las mismas aptitudes que antes, así como una nueva aptitud de API web (**WebApiSkill**) denominada **get-top-words**.
3. Edite la definición de la aptitud **get-top-words** para establecer el valor de **uri** en la dirección URL de la función de Azure (la cual ha copiado en el Portapapeles en el procedimiento anterior); para ello, reemplace **YOUR-FUNCTION-APP-URL**.
4. En la parte superior de la definición del conjunto de aptitudes, en el elemento **cognitiveServices**, reemplace el marcador de posición **YOUR_AI_SERVICES_KEY** por cualquiera de las claves de los recursos de Servicios de Azure AI.

    *Puede encontrar las claves en la página **Keys and Endpoint** (Claves y punto de conexión) del recurso de Servicios de Azure AI en Azure Portal.*

5. Guarde y cierre el archivo JSON actualizado.
6. En la carpeta **update-search**, abra **update-index.json**. Este archivo contiene la definición JSON para el índice **margies-custom-index**, con un campo adicional denominado **top_words** en la parte inferior de la definición del índice.
7. Revise el archivo JSON del índice y ciérrelo sin realizar ningún cambio.
8. En la carpeta **update-search**, abra **update-indexer.json**. Este archivo contiene una definición JSON para **margies-custom-indexer**, con una asignación adicional para el campo **top_words**.
9. Revise el archivo JSON del indexador y ciérrelo sin realizar ningún cambio.
10. En la carpeta **update-search**, abra **update-search.cmd**. Este script por lotes usa la utilidad cURL para enviar las definiciones JSON actualizadas a la interfaz de REST del recurso de Búsqueda de Azure AI.
11. Reemplace los marcadores de posición de variables **YOUR_SEARCH_URL** y **YOUR_ADMIN_KEY** por la dirección **URL** y una de las **claves de administración** del recurso de Búsqueda de Azure AI.

    *Puedes encontrar estos valores en las páginas **Información general** y **Claves** del recurso de Búsqueda de Azure AI en Azure Portal.*

12. Guarde el archivo por lotes actualizado.
13. Haga clic con el botón derecho en la carpeta **update-search** y seleccione **Abrir en terminal integrado**.
14. En el panel del terminal de la carpeta **update-search**, escriba el siguiente comando para ejecutar el script por lotes.

    ```powershell
    ./update-search
    ```

15. Cuando se complete el script, en Azure Portal, en la página del recurso de Búsqueda de Azure AI, seleccione la página **Indizadores** y espere a que se complete el proceso de indexación.

    *Puedes seleccionar **Actualizar** para supervisar el progreso de la operación de indexación. Puede tardar alrededor de un minuto en completarse.*

## Búsqueda en el índice

Ahora que tienes un índice, puedes realizar búsquedas en él.

1. En la parte superior del panel del recurso de Búsqueda de Azure AI, seleccione **Explorador de búsqueda**.
2. En el Explorador de búsqueda, cambie la vista a **la vista JSON** y, a continuación, envíe la siguiente consulta de búsqueda:

    ```json
    {
      "search": "Las Vegas",
      "select": "url,top_words"
    }
    ```

    Esta consulta recupera los campos **url** y **top_words** de todos los documentos que mencionan *Las Vegas*.

## Limpieza

Ahora que ha completado el ejercicio, elimine todos los recursos que ya no necesita. Eliminación de los recursos de Azure:

1. En **Azure Portal**, seleccione Grupos de recursos.
1. Seleccione el grupo de recursos que no necesite y luego **Eliminar grupo de recursos**.

## Más información

Para obtener más información sobre cómo crear aptitudes personalizadas para Búsqueda de Azure AI, consulte la [documentación de Búsqueda de Azure AI](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface).
