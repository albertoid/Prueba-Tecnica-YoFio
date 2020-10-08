# Prueba Técnica YoFio

Las bibliotecas que tienen dependencia en la aplicación deberán instalarse según las carencias del sistema en el que se ejecuta.

En el archivo de [Instación de Bibliotecas](https://github.com/albertoid/Prueba-Tecnica-YoFio/blob/main/Instalaci%C3%B3n%20de%20bibliotecas.ipynb).
Una vez instaladas todas las dependencias la aplicación puede ser ejecutada de forma local.

## Punto 1. Limpieza y transformación de los datos

Utilizaré la técnica de tokenización manual como primer proceso de limpieza. Utilizo la distancia de Levenshtein para determinar la similaridad de las palabras en el contexto de ser ocupaciones siendo y en la función `Clasificador_Empleo` se define como argumento la variable `precision` para asignar la diferencia aceptable al realizar el cálculo de las distancias para la identificación del empleo ingresado. En esta versión al identificar una ocupación con una distancia de Levenshtein menor a la establecida en `precision` asigna el nombre del grupo a ese elemento de la lista. En futuras implementaciones se puede modificar para que evalue todas las palabras y asigne la de menor distancia de Levenshtein.

Para más información acerca de la distancia de Levenshtein ver la siguiente liga: [Distancia de Levenshtein](https://en.wikipedia.org/wiki/Levenshtein_distance#:~:text=Informally%2C%20the%20Levenshtein%20distance%20between,considered%20this%20distance%20in%201965.). Y para conocer de la biblioteca en Python que la calcula, ver la siguiente liga: [python-Levenshtein](https://github.com/ztane/python-Levenshtein/).

El flujo del programa tiene los siguientes 8 pasos:

1. Carga de datos 
2. Revisión de la consistencia de los datos y transformación a string
3. Normalización de mayúsculas y minúsculas
4. Eliminación de puntuación y caracteres especiales
5. Separar por espacios
6. Eliminación de palabras con poco significado.
7. Definición de grupos y reclasificación
8. Guardado de base de datos en un archivo

Puede ejecurse paso a paso o al final de todos se encuentra concentrado todo el código para la ejecución en un solo paso.

# Punto 2. Modelo de recolección, limpieza, transformación, almacenamiento y uso de los datos


# Punto 3. Métricas a partir de los datos

