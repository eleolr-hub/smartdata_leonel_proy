# Proyecto para SmartData - Pipeline de ingesta, transformación y carga

Este proyecto fue desarrollado para construir un flujo de procesamiento de datos sobre información de clientes, correos y SMS, con el objetivo de convertir fuentes crudas en tablas estructuradas y listas para análisis. El proceso está organizado en notebooks dentro de la carpeta proceso y sigue una lógica de arquitectura medallion: Bronze, Silver y Golden.

## ¿Qué se busca lograr?

El objetivo principal es tomar archivos fuente almacenados en el data lake, cargarlos en una capa inicial de datos crudos, transformarlos para dejar la información limpia y consistente, y finalmente generar agregados que permitan responder preguntas de negocio de forma más rápida y confiable.

En términos simples, el proyecto permite pasar de datos en bruto a información preparada para análisis, pasando por tres etapas claras:

1. Ingesta: lectura de los archivos fuente y carga inicial en tablas de Bronze.
2. Transformación: limpieza, estandarización y enriquecimiento de los datos para dejarlos listos en Silver.
3. Carga analítica: generación de tablas agregadas en Golden para consumo y visualización.

## Flujo del proceso

El flujo de notebooks se ejecuta en orden, desde la preparación del ambiente hasta la generación de tablas analíticas.

### 1. Preparación del ambiente

En el notebook proceso/0.- Preparacion_Ambiente.ipynb se definen los componentes base del entorno. Aquí se crean los catálogos, esquemas y ubicaciones externas necesarias para trabajar con Delta Tables en Databricks. También se crean las tablas base para las capas Bronze, Silver y Golden, preparándolas para recibir los datos del pipeline.

### 2. Ingesta de datos en Bronze

Esta fase se encarga de cargar los archivos fuente a la capa Bronze.

- proceso/1.01.Ingest_Catalogo.ipynb: lee el archivo de catálogo de cliente-producto y lo carga en la tabla bronze.catalogo_cliente_producto.
- proceso/1.02.Ingest_email.ipynb: ingesta los correos del archivo MessageTrace_2026.csv y los guarda en la tabla bronze.emails.
- proceso/1.03.Ingest_SMS_Antiguo.ipynb: carga la plataforma antigua de SMS en la tabla bronze.sms_antiguo.
- proceso/1.04.Ingest_SMS_Nuevo.ipynb: carga la plataforma nueva de SMS en la tabla bronze.sms_nuevo.

En esta parte del proceso se define el esquema de cada fuente, se leen los archivos desde el storage y se escriben los datos con una columna de ingestion_date para registrar el momento de carga.

### 3. Transformación a Silver

Una vez que los datos están en Bronze, se realizan transformaciones para limpiar y unificar la información.

- proceso/2.01.Transform_catalogo.ipynb: toma el catálogo de clientes y productos, construye una llave compuesta con cliente y producto, elimina registros vacíos o duplicados, y genera la tabla silver.catalogo_transformed.
- proceso/2.02.Transform_email.ipynb: aplica lógica de clasificación de productos a partir del asunto y del remitente del correo. Además, agrega información de año y mes, realiza el join con el catálogo y deja los correos listos para análisis en silver.emails_transformed.
- proceso/2.03.Transform_sms.ipynb: unifica los SMS de las plataformas antigua y nueva en una sola tabla de Silver. Se normalizan campos como número, usuario, periodo y origen, y se deja una vista consolidada en silver.sms_transformed.

### 4. Carga analítica en Golden

La última etapa del pipeline genera tablas agregadas listas para consumo analítico.

- proceso/3.01.Load.ipynb: crea los agregados finales en las tablas golden_sms y golden_email. Estas tablas resumen la información por año, mes, cliente, producto y otras dimensiones clave, facilitando la consulta posterior en dashboards o reportes.

## Resumen del proyecto

Este proyecto representa una implementación práctica de un pipeline de datos en Databricks, donde se combina ingesta, transformación y carga en una estructura ordenada y reproducible. El flujo fue diseñado para ser claro, modular y escalable, permitiendo que cada etapa tenga una responsabilidad específica dentro del proceso.

En resumen, el proyecto toma datos de origen, los ingesta en Bronze, los prepara y limpia en Silver, y produce métricas y resultados consolidados en Golden. Este enfoque facilita la trazabilidad, el mantenimiento y la posterior generación de valor analítico.

## Notas de entrega

La solución está pensada para entregarse como un proyecto funcional de referencia, con notebooks ejecutables en secuencia y tablas organizadas por capa. El flujo está preparado para trabajar con parámetros dinámicos como el nombre del storage, catálogo y esquema, lo que facilita su reutilización en distintos ambientes.