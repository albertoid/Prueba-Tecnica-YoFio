# Prueba Técnica YoFio

Las bibliotecas que tienen dependencia en la aplicación deberán instalarse según las carencias del sistema en el que se ejecuta.

En el archivo de [Instación de Bibliotecas](https://github.com/albertoid/Prueba-Tecnica-YoFio/blob/main/Instalaci%C3%B3n%20de%20bibliotecas.ipynb).
Una vez instaladas todas las dependencias la aplicación puede ser ejecutada de forma local.

## Punto 1. Limpieza y transformación de los datos

Utilizaré la técnica de tokenización manual como primer proceso de limpieza. Utilizo la distancia de Levenshtein para determinar la similaridad de las palabras en el contexto de ser ocupaciones siendo y en la función `Clasificador_Empleo` se define como argumento la variable `precision` para asignar la diferencia aceptable al realizar el cálculo de las distancias para la identificación del empleo ingresado. En esta versión al identificar una ocupación con una distancia de Levenshtein menor a la establecida en `precision` asigna el nombre del grupo a ese elemento de la lista. En futuras implementaciones se puede modificar para que evalué todas las palabras y asigne la de menor distancia de Levenshtein.

Para más información acerca de la distancia de Levenshtein ver la siguiente liga: [Distancia de Levenshtein](https://en.wikipedia.org/wiki/Levenshtein_distance#:~:text=Informally%2C%20the%20Levenshtein%20distance%20between,considered%20this%20distance%20in%201965.). Y para conocer de la biblioteca en Python que la calcula, ver la siguiente liga: [python-Levenshtein](https://github.com/ztane/python-Levenshtein/).

El flujo del programa tiene los siguientes 8 pasos:

1. Extracción de datos 
2. Revisión de la consistencia de los datos y transformación a string
3. Normalización de mayúsculas y minúsculas
4. Eliminación de puntuación y caracteres especiales
5. Separar por espacios
6. Eliminación de palabras con poco significado.
7. Definición de grupos y reclasificación
8. Guardado de base de datos en un archivo

Puede ejecurse paso a paso o al final de todos se encuentra concentrado todo el código para la ejecución en un solo paso.

A manera de visualización genere una nube de palabras con la reducción de los empleos identificados mostrada a continuación. Se redujo a 16 clasificaciones y una adicional de "otro" a aquellas que no se pudieron clasificar. La clasificación fue en cierta medida manual, después de la priera clasificación utilizando la similitud de las palabras.

<p align="center">
  <img src="https://github.com/albertoid/Prueba-Tecnica-YoFio/blob/main/img/nubepalabras.png">
</p>

## Punto 2. Modelo de recolección, limpieza, transformación, almacenamiento y uso de los datos

Para el diseño del diagrama de componentes se consideraron las siguientes tecnologías y herramientas:

1. Python
    1. Pandas
    2. psyconpg2
    3. Flask / Django 
2. PostgreSQL
3.AWS
    1. EC2
    2. RDS
4. HTML

El Origen 1 al ser alimentado directamente por el usuario al registrarse, se considera que proviene de un componente Web Browser que se conecta por medio de HTML y este mediante un API alimenta una base de datos en la nube (podría ser en un serivicio local pero en este caso se piensa en un servicio en la nube). El Origen 2 no hace referencia a la alimentación de los datos, pero estos se consideran previemente existentes en una base de datos relacional que se aloja en un servidor en la nube, por lo que sus procesos son interconectados mediante peticiones (<em>requests</em>) a la base de datos. Otro compomente deberá ser el encargado de hacer el ingreso de la información, ya que el caso esta enfocado a la solución particular de la recolección de datos dada la naturaleza del proceso. Para el Origen 3 al igual que el Origen 2 la base  de datos se alimenta mediante otro componente, y en este caso funciona de manera similar que el componente anterior.

A continuación se muestra el diagrama de componentes descrito en el punto 2

<p align="center">
  <img src="https://github.com/albertoid/Prueba-Tecnica-YoFio/blob/main/img/DiagramaComponentes.png">
</p>

Para el caso de la consulta de datos y la interconexión posterior a un motor de Machine Learning (ML), mediante la implementación de una API corriendo en un servicio EC2 o S3 de AWS, este hará los requerimientos a las base de datos de los 3 origenes y hará los procesos de transformación (preparación para hacer los datos compatibles con las bibliotecas de ML) mediante la biblioteca de pandas y posteriormente se ejecutaran los algoritomos de ML definidos mediante la libreria de Scikit Learn (Podría ser cualquier otra considerando los procedimientoe de estas con funciones que hagan compatibles los datos transformados) y estos podrían regresar en mediante el mismo servicio de la API que recibe los datos en un despliegue instantáneo o bien alimentando una bitácora de consultas, integrando un informe automático, que inclusive emita alguna recomendación dependiendo del resultado.

A continuación se muestra el diagrama de este módulo de consulta y aplicación de un motor de ML.

<p align="center">
  <img src="https://github.com/albertoid/Prueba-Tecnica-YoFio/blob/main/img/DiagramaComponeteConsulta.png">
</p>

Como conclusción todos los componentes descritos pueden encapsularse en un solo para realizar la tarea descrita. De la misma manera un <em>Query</em> de SQL puede tener la potencia extraer la información requerida de los procesos definidos, lo que a nivel de implementación simplificaría la tarea, por supuesto si las bases de datos compartieran un entorno en donde pudieran relacionarse entre si con el motor de SQL. El API de consulta podría complementarse con el de limpieza para tomar los datos de las bases como están sin procesarse y hacer la tarea de consulta.

## Punto 3. Métricas a partir de los datos

Para el cálculo de la métrica first payment rate el con la información de ls 4 procesos fue la siguiente:

1. Extracción de bases de datos
2. Limpieza y transformación
3. Cálculo del first Payment Default Rate por dia

### 3.1. Extracción de bases datos

Esta la realice utilizando la relación de los nombres de los arhivos de datos agrupandolos en una lista de data frames de Pandas

### 3.2. Limpieza y transformación

Los filtrados correspondientes a las restricciones de las reglas de cálculo fueron realizadas mediante la funcionalidad de pandas de extracción de parámetros que cumplen ciertas carácterísticas, para el caso del proceso 2, tener todos los campos completos, en especial el de `address` ya que todos los demás campos son válidos por defecto y en el caso del proceso 3, contar con una primera compra mayor a 40 pesos. Una vez teniendo las dos listas de `user_id` la combinación de las validaciones consiste en verificar si los `user_id` están en ambas listas, por lo que las que cumplan con esta condición serán los usuarios que cuentan con datos completos y una compra inicial en `tx1_amount` mayor a 40 pesos.

Algo importante en la limpieza y transformación es el tratamiento de las fechas, las cuales al estar en `string` deben ser pasadas a un formato de fecha funcional con el cual se puedan hacer operaciones, tal como lo es el tipo de datos `datetime.date`.   Una vez tienendo una lista de `user_id` que cumplen con la condiciones para ser tomados en cuenta en el cálculo del first payment default rate es posible proceder al mismo, calculando las fechas inicial y final, así como cálcular su diferencia para utilziarlo como un parámetro de control en el cálculo.

### 3.3. Cálculo del first Payment Default Rate por dia

Para hacer el cálculo genero una lista de usuarios por cada dia, para después hacer el cálculo dia por dia extrayendo directamente de la información de los procesos 3 y 4. Después de contabilizar los procesos en impago, la cantidad de elementos de la lista de usuarios por dia nos da el núemero de cuentas, lo que al sacar la razón entre estos nos dá el porcentaje de cuentas en impago de la primera compra con relación al fia o  el first payment default rate, resultados que se van inegrando a una lista, para posteriormente ser transformados a un data frame de pandas y pasado un archivo csv o cualquier otro proceso de carga.


|    |      Fecha | Total_de_cuentas | Cuentas_en_impago | Porcentaje_cuentas_impago |
|---:|-----------:|-----------------:|------------------:|---------------------------|
|  0 | 2020-07-07 |                6 |                 1 |                     16.67 |
|  1 | 2020-07-08 |               12 |                 1 |                      8.33 |
|  2 | 2020-07-09 |               13 |                 1 |                      7.69 |
|  3 | 2020-07-10 |               11 |                 1 |                      9.09 |
|  4 | 2020-07-11 |                5 |                 1 |                     20.00 |
|  5 | 2020-07-12 |                9 |                 1 |                     11.11 |
|  6 | 2020-07-13 |               10 |                 1 |                     10.00 |
|  7 | 2020-07-14 |                8 |                 0 |                      0.00 |
|  8 | 2020-07-15 |                3 |                 1 |                     33.33 |
|  9 | 2020-07-16 |               15 |                 1 |                      6.67 |
| 10 | 2020-07-17 |               12 |                 1 |                      8.33 |
| 11 | 2020-07-18 |               11 |                 0 |                      0.00 |
| 12 | 2020-07-19 |               15 |                 1 |                      6.67 |
| 13 | 2020-07-20 |               11 |                 1 |                      9.09 |
| 14 | 2020-07-21 |               13 |                 1 |                      7.69 |
| 15 | 2020-07-22 |               11 |                 1 |                      9.09 |
| 16 | 2020-07-23 |               11 |                 1 |                      9.09 |
| 17 | 2020-07-24 |               16 |                 1 |                      6.25 |
| 18 | 2020-07-25 |               10 |                 1 |                     10.00 |
| 19 | 2020-07-26 |                6 |                 0 |                      0.00 |
| 20 | 2020-07-27 |               15 |                 1 |                      6.67 |
| 21 | 2020-07-28 |               13 |                 1 |                      7.69 |
| 22 | 2020-07-29 |               11 |                 1 |                      9.09 |
| 23 | 2020-07-30 |               12 |                 1 |                      8.33 |
| 24 | 2020-07-31 |               12 |                 1 |                      8.33 |
| 25 | 2020-08-01 |               12 |                 1 |                      8.33 |
| 26 | 2020-08-02 |               25 |                 1 |                      4.00 |
| 27 | 2020-08-03 |               18 |                 1 |                      5.56 |
| 28 | 2020-08-04 |               15 |                 1 |                      6.67 |
| 29 | 2020-08-05 |               18 |                 1 |                      5.56 |
| 30 | 2020-08-06 |                9 |                 1 |                     11.11 |
| 31 | 2020-08-07 |               23 |                 1 |                      4.35 |
| 32 | 2020-08-08 |                9 |                 1 |                     11.11 |
| 33 | 2020-08-09 |               14 |                 1 |                      7.14 |
| 34 | 2020-08-10 |               11 |                 1 |                      9.09 |
| 35 | 2020-08-11 |                9 |                 1 |                     11.11 |
| 36 | 2020-08-12 |                5 |                 1 |                     20.00 |
| 37 | 2020-08-13 |                8 |                 1 |                     12.50 |
| 38 | 2020-08-14 |               11 |                 1 |                      9.09 |
| 39 | 2020-08-15 |               13 |                 1 |                      7.69 |
| 40 | 2020-08-16 |               11 |                 1 |                      9.09 |
| 41 | 2020-08-17 |               13 |                 1 |                      7.69 |
| 42 | 2020-08-18 |               15 |                 1 |                      6.67 |
| 43 | 2020-08-19 |               10 |                 1 |                     10.00 |
| 44 | 2020-08-20 |               14 |                 1 |                      7.14 |
| 45 | 2020-08-21 |               11 |                 1 |                      9.09 |
| 46 | 2020-08-22 |               13 |                 1 |                      7.69 |
| 47 | 2020-08-23 |               16 |                 1 |                      6.25 |
| 48 | 2020-08-24 |                7 |                 1 |                     14.29 |
| 49 | 2020-08-25 |               19 |                 1 |                      5.26 |
| 50 | 2020-08-26 |               13 |                 1 |                      7.69 |
| 51 | 2020-08-27 |               11 |                 1 |                      9.09 |
| 52 | 2020-08-28 |               17 |                 1 |                      5.88 |
| 53 | 2020-08-29 |               10 |                 1 |                     10.00 |
| 54 | 2020-08-30 |                8 |                 1 |                     12.50 |
| 55 | 2020-08-31 |               13 |                 1 |                      7.69 |
| 56 | 2020-09-01 |                9 |                 1 |                     11.11 |
| 57 | 2020-09-02 |               10 |                 1 |                     10.00 |
| 58 | 2020-09-03 |                9 |                 1 |                     11.11 |
| 59 | 2020-09-04 |               10 |                 1 |                     10.00 |
| 60 | 2020-09-05 |               13 |                 1 |                      7.69 |
| 61 | 2020-09-06 |               10 |                 1 |                     10.00 |
| 62 | 2020-09-07 |               13 |                 1 |                      7.69 |
| 63 | 2020-09-08 |               13 |                 1 |                      7.69 |
| 64 | 2020-09-09 |               10 |                 1 |                     10.00 |
| 65 | 2020-09-10 |               11 |                 1 |                      9.09 |
| 66 | 2020-09-11 |               17 |                 1 |                      5.88 |
| 67 | 2020-09-12 |               12 |                 1 |                      8.33 |
| 68 | 2020-09-13 |               12 |                 1 |                      8.33 |
| 69 | 2020-09-14 |               11 |                 1 |                      9.09 |
| 70 | 2020-09-15 |               12 |                 1 |                      8.33 |
| 71 | 2020-09-16 |                9 |                 1 |                     11.11 |
| 72 | 2020-09-17 |               12 |                 1 |                      8.33 |
| 73 | 2020-09-18 |               11 |                 1 |                      9.09 |
| 74 | 2020-09-19 |                8 |                 1 |                     12.50 |
| 75 | 2020-09-20 |                8 |                 1 |                     12.50 |
| 76 | 2020-09-21 |               14 |                 1 |                      7.14 |
| 77 | 2020-09-22 |                9 |                 1 |                     11.11 |
| 78 | 2020-09-23 |               18 |                 1 |                      5.56 |
| 79 | 2020-09-24 |               17 |                 1 |                      5.88 |
| 80 | 2020-09-25 |               13 |                 1 |                      7.69 |
| 81 | 2020-09-26 |               11 |                 1 |                      9.09 |
| 82 | 2020-09-27 |                9 |                 1 |                     11.11 |
| 83 | 2020-09-28 |                4 |                 1 |                     25.00 |
| 84 | 2020-09-29 |                9 |                 1 |                     11.11 |

Para una mejor visualización de estos datos las siguientes gráficas describen el comportamiento de los usuarios y su cumplimiento de pago en la gráfica 1

<p align="center">
  <img src="https://github.com/albertoid/Prueba-Tecnica-YoFio/blob/main/img/Grafica1.png">
</p>


En la gráfica 2 se observa que el máximo registrado de usuarios impagos en su primera compra es de 1 (un) usuario al dia según su fecha de registro. En esta gráfica observamos con la distribución de la cantidad de usuarios registrados por dia se comporta de forma normal o gaussiana con una media en 11.71

<p align="center">
  <img src="https://github.com/albertoid/Prueba-Tecnica-YoFio/blob/main/img/Grafica2.png">
</p>

Aqui en la gráfica 3 podemos observar que la distribución entre aquellos dias con niguna cuanta impaga y las que están al corriente es una distribución J

<p align="center">
  <img src="https://github.com/albertoid/Prueba-Tecnica-YoFio/blob/main/img/Grafica3.png">
</p>

Por último en la gráfica 4 la distribución de los first Payment Default Rate muestra un desplazamiento a la izquierda al tener outlayers a la derecha muy grandes según el margen observado, por aquellos dias que existen pocos registros.

<p align="center">
  <img src="https://github.com/albertoid/Prueba-Tecnica-YoFio/blob/main/img/Grafica4.png">
</p>

 
