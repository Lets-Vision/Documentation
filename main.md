# Como crear tus propias aplicaciones para LetsUI
Como primera instancia se debe coniciderar que Lets Vision funciona en un entorno React, esto significa que sus aplicaciones deben estar estrictamente en Javascript/Typescript
## Cómo preparar el entorno para la creacion de aplicaciones para LetsUI

Para hacer aplicaciones compatibles con LetsUI, se deben colocar los archivos en una carpeta con el nombre de nuestra aplicacion en la siguiente ruta: `./src/pkg/[APP-NAME]`

## Preparar el archivo data.json para el reconocimiento de la aplicación
Para que LetsUI reconozca las aplicaciones creadas por los usuarios, se debe incluir un archivo `data.json` para que este reconzca el nombre y descripcion de la misma, para que posteriormente sea enlistada en la interfaz


```json
{
  "name": "Speakers", 
  "description": "Herramienta que proporciona voz asistida a través de gestos oculares.",
  "cat": "Aplicaciones"
}
```
`"name:" ` `Nombre de la Aplicación`  

`"description:" ` `Descripcion de la Aplicación`  

`"cat:" ` `Categoria de la Aplicacion [No usado]` 

`"creator":` `Desarrollador de la Aplicación [No usado]`

?> **Recomendación:** Los parametros catalogados como "No usados", seran implementados en proximas actualizaciones.

!> **Advertencia:** El uso de la etiqueta "Creator" es un requisito **obligatorio** para la publicación de tu aplicacion en la Lets Store  


## Creación del archivo main.jsx
El codigo de tus aplicaciones debe encontrarse en un archivo llamado `main.jsx`. La estructura del mismo debe ser asi
``` js
import React, { useState } from 'react';

const MyApp= () => {
  return (
    <div>
      <h1 style={{textAlign: 'center'}}>Hello World!</h1>
    </div>
  );
};

export default MyApp;
```
?> *El archivo a ejecutar como primera instancia de tu aplicación debe llamarse `main.jsx`, sin embargo este mismo puede importar otros componentes react*

### ¡Ahora tu aplicacion se está ejecutando!
![](helloworld.PNG)
