# Proyecto individual 2 - Datathon Machine Learning

## Objetivos.

Implementar un modelo de clasificación que permita clasificar el precio de las propiedades en venta en las categorías barato(por debajo de la media) o caro(por encima de la media), utilizando datatasets de ventas de propiedades inmobiliarias de Colombia del año 2020.

## Archivos: 
- DATATHON_ML.ipynb : notebook con la carga de datos, preprocesamiento y entrenamiento de los modelos
- CesarG2022.csv: git statusvalores predichos(0:barato, 1:caro) para el dataset properties_colombia_test.csv por el mejor modelo  
- properties_colombia.zip: archivo comprimido que contine 1 dataset de entrenamiento y 1 de testeo.

## Métricas para evaluar.

Como método de evaluación del desempeño del modelo se utiliza la métrica de Exhaustividad (Recall) para las propiedades caras, adicionalmente se incluye la Accuracy como métrica de control.

## Descripción de las dimensiones de datatsets.

- id - Identificador del aviso. No es único: si el aviso es actualizado por la inmobiliaria (nueva versión del aviso) se crea un nuevo registro con la misma id pero distintas fechas: de alta y de baja.
- ad_type - Tipo de aviso (Propiedad, Desarrollo/Proyecto).
- start_date - Fecha de alta del aviso.
- end_date - Fecha de baja del aviso.
- created_on - Fecha de alta de la primera versión del aviso.
- lat - Latitud.
- lon - Longitud.
- l1 - Nivel administrativo 1: país.
- l2 - Nivel administrativo 2: usualmente provincia.
- l3 - Nivel administrativo 3: usualmente ciudad.
- l4 - Nivel administrativo 4: usualmente barrio.
- l5 - Nivel administrativo 5.
- l6 - Nivel administrativo 6.
- rooms - Cantidad de ambientes.
- bedrooms - Cantidad de dormitorios (útil en el resto de los países).
- bathrooms - Cantidad de baños.
- surface_total - Superficie total en m².
- surface_covered - Superficie cubierta en m².
- price - Precio publicado en el anuncio.
- currency - Moneda del precio publicado.
- price_period - Periodo del precio (Diario, Semanal, Mensual)
- title - Título del anuncio.
- description - Descripción del anuncio.
- property_type - Tipo de propiedad (Casa, Departamento, PH).
- operation_type - Tipo de operación (Venta).
- geometry - Puntos geométricos formados por las coordenadas latitud y longitud.

## Flujo de trabajo

- Carga de datos: lectura de archivo properties_colombia_train.csv
- Dar formato a datos
    - fechas: 
            - En columna end_date se cambian instancias con año 9999(11974 datos) por None; se dejan instancias con año 2020 y 2021(esta última porque es fecha de baja de aviso).
            
- Manejo inconsistencias
    - Currency: se eliminan 8 datos con USD porque representan menos del 0.01% de los datos.
    - longitud, latitud: se eliminan valores(dos filas) fuera del rango de valores posibles para colombia.
    - tratamiento de nulos:
        - end_date: se dejan 11974 nulos (se rellenan automáticamente con 1 después de aplicar formato to_ordinal()).
        - currency y price: se elimnan 67 filas con nulos, representan menos de 0.1% de los datos.

    - identificación y eliminación de columnas que no dan información:

        - columnas invariantes, estas son columnas con variable categórica con un solo valor(categoría) para parte o todos los datos:
            - operation_type: todos son 'Venta'.
            - ad_type: todos son 'Propiedad'.
            - price_period: solo tiene nulos o 'Mensual' y mensual no da información ya que no hay arriendos, solo hay ventas.
            - l1: todos son 'colombia'.
            - currency: despues de eliminar las 8 filas con USD y nulo, todos los valores son COP.

        - columnas no relacionadas con el objetivo, estas son columnas no relacionadas con el precio: 
            - id
            - unnamed:0.

        - columnas duplicadas: 
            - created_on y start-date: se elimina created_on
            - geometry y lat, lon: se elimina geometry

    - Eliminación de dúplicados: se realizan dos tipos de eliminacion de duplicados:
         - sin fechas: se aplica dropduplicate() sobre un nuevo dataframe llamado data_train_sf que se crea elimando las columnas start_date y end_date del dataframe(data_train) del flujo de trabajo principal. las columnas de fechas se eliminan considerando que ninguna de ellas tiene correlación con el precio. En este caso se eliminan aprox. 38% de filas.
        - con fechas: simplemente se aplica dropduplicate() sobre el dataframe(data_train) del flujo de trabajo principal. en este caso eliminan aprox. 2% de filas. Este dataframe se útiliza en adelante como dataframe de referencia y se trabaja principalmente con el dataframe sin fechas porque en el análisis exploratorio no se encontró relación entre fechas y target.
     
    
- EDA(selección de features):
    - creación de target: nueva columna con dos valores, 0:precio < media , 1:precio>media. la media se obtiene del dataframe con fechas porque tiene mas datos.
    - verificación de balanceo: se considera que la relacion de aprox. de 24/76 es aceptable(>5/95)
    - búsqueda de correlación: usando corr en un mapa de calor y pairplot se logran identificar tres variables correlacionadas con el target, estas son bathrooms, surface_total y rooms 
    - imputación de valores: se prefirió el KNNimputer sobre la media, debido a la alta cantidad de datos faltantes en las columnas de interes.
        - lat y lon: se imputan con el método de KNNimputer utilizando también las columnas l2, l3, l4 porque están relacionadas con ubicación y property_type porque de las columnas que no tinen nulos es la que podría estar mas relacionada con ubicación. para poder utilizar l2, l3, l4 y property_type se realizo una codificación con getdummies y debido a la gran cantidad de columnas que se obtuvieron(<400) se utiliza PCA para reducirlas a 20 columnas. no se utilizaron l5 y l6 porque son muy pocos datos.
        - rooms, bedrooms, bathrooms, surface_total, surface covered: se imputan también con el método KNNimputer utilizando ademas de estas mismas la columna property_type, de nuevo codificada con getdummies pero sin reducción de dimesiones debido a la baja cantidad de categorías. se eligen estas columnas porque todas estan relacionadas con tamaño.

    - codificación de variable property_type, para incluirla en selección de features con Kbest(), la nueva columna codificada se denomina 'property_type_le'.
    - Selección de features con KBest: se introducen las columnas 'lat', 'lon', 'rooms', 'bedrooms', 'bathrooms', 'surface_total', 'surface_covered','property_type_le'. se obtienen mayores scores para columnas: lat, lon, bathrooms, surface_total.
    
- Aplicación de modelos:
    - HistGradientBoostingClassifier: se utilizo en primer ya que acepta columnas con NAN, da como resultado recall muy bajos.
    - RandomForestClassifier: ofrece tanto accuracy como recall bastante altos.
    - XGBClassifier(boosting)
    - 

- Preparación de dataset de testeo: se realiza todo el preprocesamiento que se realizó al dataframe de entrenamiento.
- Predicciones sobre dataset de testeo con modelos previamente entrenados y exportación a csv.




