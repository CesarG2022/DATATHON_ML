# Proyecto individual 2 - Datathon Machine Learning

## Objetivos.

Implementar un modelo de clasificación que permita clasificar el precio de las propiedades en venta en las categorías barato(por debajo de la media) o caro(por encima de la media), utilizando datatasets de ventas de propiedades inmobiliarias de Colombia del año 2020.

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

- Carga de datos
- Formateo de datos

    - fechas: 
            - En columna end_date se cambia instancias con año 9999(11974 datos) por None; se dejan instancias con año 2020 y 2021(esta última porque es fecha de baja de aviso).
            
- Manejo de inconsistencias

    - Currency: se eliminan 8 datos con USD.
    - longitud, latitud: se eliminan valores fuera de rango (dos filas)
    - tratamiento de nulos:
        - end_date: se dejan 11974 (tienen valor: 1 despues de aplicar formato to_ordinal())
        - currency y price: se elimnan 67 filas
    - identificacion y eliminación de columnas que no dan información:
        - invariantes, estas son columnas con variable categórica con un solo valor(categoría) para parte de o todos los datos:
            - operation_type: todos 'Venta'
            - ad_type: todos 'Propiedad'
            - price_period: solo tiene nulos o 'Mensual' y mensual no da información ya que no hay   arriendos, solo hay ventas.
            - l1: todos son 'colombia'
            - currency: despues de eliminar las 8 filas con USD y nulo, todos los valores son COP
        - nulas, estas son columnas con mayoría nulos: !!!!!!!!!!!!!!!!PENDIENTE
            - 
        - no relacionadas con el objetivo, estas son columnas no relacionadas con el precio: 
            - id
            - unnamed:0
        - duplicadas: 
            - created_on y start-date: se elimina created_on
            - geometry y lat, lon: se elimina geometry
    - eliminación de duplicados: !!!!!!!!!!!!!!!!!!!PENDIENTE
    - imputacion de valores: !!!!!!!!!!!!!!!!!!!!!PENDIENTE
            -rooms: todos los nulos cambian por 0
            -bedrooms: todos los nulos cambian por 0
            -bath

- EDA(selección de features):
    - creación de target
    - detectar outliers
    - verificar escala
    - ver correlación: pairplot
    - verificar balanceo

- Preprocesamiento
    - StandardScaler
    - LabelEncoder
    - Reducción de dimensionalidad.

- Selección de hiperparametros
- Predicción (instanciación, fitting, y predicción)
- Evaluación: 




