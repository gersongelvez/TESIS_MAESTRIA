<p align="center">
<h1>Analítica de Grafos Sobre Productos de Mercado Libre</h1>
</p>


# 1. Introducción
Este artículo es una guía con enfoque académico para aplicar analítica de grafos y computar métricas de grafos, nodos y relaciones sobre grandes volúmenes de datos.
Incrementar la documentación y sets de datos disponibles a la comunidad interesada en aprender o profundizar en temas relacionados con analítica de grafos.
Este trabajo se realiza en la ciudad de Bogotá, tomando como información base, los datos públicos alojados en la página web Mercadolibre.com, en la categoría de tecnología.
Se implementa con las tecnologías Java, Python y Neo4j.

# 2. Objetivos

## 2.1. Objetivo general

Este artículo se ejecuta  con el fin de hacer un aporte académico a la comunidad interesada en aprender o profundizar en analítica de grafos. 


## 2.2. Objetivos específicos

* Compartir a la comunidad un set de datos para que se pueda hacer analítica de datos con finalidades académicas.

* Hacer el análisis de la información mediante analítica descriptiva tradicional.

* Hacer el análisis de la información mediante analítica descriptiva usando algoritmos de grafos.

* Visualizar los resultados 

# 3. Problema
No hay un set de datos con información de Bogotá, Colombia, sobre una tienda online y su respectiva guía para hacer analítica de grafos. Con este trabajo se consolida la información pública, presente en la página www.mercadolibre.com.co sobre vendedores, compradores y productos de tecnología en la ciudad de Bogotá. Se crea una guía o laboratorio donde se explica cómo aplicar analítica de grafos sobre el set de datos consolidado.

# 4. Método
Se describe el paso a paso la ejecución de este trabajo y los resultados

# 4.1. Extracción de la información
* Tecnologías: La información se extrae mediante técnicas de webscraping con tecnologías de JAVA y PYTHON. Se usa NEO4J para almacenar la información en un esquema de grafos y usar sus algoritmos. Se usa la libreria JavaScript NEOVIS.JS para la visualización de información.
* Dominio de la información extraida: Para asegurar que la información extraída de la página www.mercadolibre.com.co tenga un alcance definido. Se toman los siguientes criterios: Información de Bogotá, Colombia. Información de la categoría de tecnología. Se ejecutan procesos paralelos de extracción de información que filtran los datos por País (Colombia), Ciudad (Bogotá), 		   Sector de bogotá, Categoría.

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/4_0_PROCESO_EXTRACCION_DATOS.jpg">
</p>

# 4.2 Esquema del grafo
La información de compras y ventas en la categoría de tecnología de mercadolibre para la ciudad de Bogotá. Se puede esquematizar en un modelo relacional de la siguiente forma:

![MER productos de mercadolibre](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/4_2_MER.png)
 
La misma información se puede expresar en un esquema de grafos de la siguiente manera:

![Grafo productos de mercadolibre](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/4_3_GRAFO.png)

El grafo anteriormente presentado se describe mediante el diccionario de datos:

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/4_3_DICCIONARIO_DATOS_2.jpg">
</p>
 
# 4.3 Importar datos en Neo4j

La información recolectada con técnicas de webscraping se plasma en archivos CSV. Neo4j con su lenguaje Cypher proporciona una opción de importación de datos, usando el comando LOAD CSV para transformar el contenido de los archivos CSV a una estructura de grafos.
Se deben seguir los siguientes pasos para cargar los datos a Neo4j:

•	Los datos se pueden descargar en la URL (Carrillo. Gerson, 2019, https://github.com/gersongelvez/TESIS_MAESTRIA/tree/master/DATOS) 

•	Se debe tener un motor de bases de datos Neo4j instalado y en correcto funcionamiento. Los archivos descargados en el punto anterior, se deben poner en la carpeta import del servidor donde se encuentra instalado Neo4j. Este punto es necesario para usar la herramienta de importación de archivos CSV.

•	Ejecutar los comandos de importación uno a uno en la consola de Neo4j:

```cypher
USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///DEPARTAMENTO.csv' AS row
CREATE (:DEPARTAMENTO { ID : row.ID, NOMBRE : row.NOMBRE})

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///MUNICIPIO.csv' AS row
CREATE (:MUNICIPIO { ID : row.ID, NOMBRE : row.NOMBRE})

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///MUNICIPIO.csv' AS row
MATCH (D:DEPARTAMENTO),(M:MUNICIPIO) WHERE D.ID=row.DEPARTAMENTO_ID AND M.ID=row.ID CREATE (M)-[:MUNICIPIO_EN]->(D)

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///USUARIO.csv' AS row
CREATE (:USUARIO { ID : row.ID, NOMBRE : row.NOMBRE, ES_VENDEDOR : row.ES_VENDEDOR, ES_COMPRADOR : row.ES_COMPRADOR, URL : row.URL})

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///USUARIO.csv' AS row
MATCH (M:MUNICIPIO),(U:USUARIO) WHERE M.ID=row.MUNICIPIO_ID AND U.ID=row.ID CREATE (U)-[:UBICADO_EN]->(M)

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///OPINION.csv' AS row
MATCH (UC:USUARIO),(UV:USUARIO) WHERE UC.ID=row.USUARIO_COMPRADOR_ID AND UV.ID=row.USUARIO_VENDEDOR_ID CREATE (UC)-[:OPINA {TIPO: row.TIPO, FECHA:row.FECHA, OPINION: row.OPINION} ]->(UV)


USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///PRODUCTO.csv' AS row
CREATE (:PRODUCTO { ID : row.ID, NOMBRE : row.NOMBRE, VALOR : row.VALOR, ES_NUEVO : row.ES_NUEVO, URL : row.URL, USUARIO_ID : row.USUARIO_ID})

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///PRODUCTO.csv' AS row
MATCH (P:PRODUCTO),(U:USUARIO) WHERE P.ID=row.ID AND U.ID=row.USUARIO_ID CREATE (U)-[:VENDE]->(P)

/*CATEGORIA*/
USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///CATEGORIA.csv' AS row
CREATE (:CATEGORIA { ID : row.ID, NOMBRE : row.NOMBRE})

/*RELACION CATEGORIA - CATEGORIA */
USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///CATEGORIA.csv' AS row
MATCH (CH:CATEGORIA),(CP:CATEGORIA) WHERE CH.ID=row.ID AND CP.ID=row.CATEGORIA_PADRE_ID CREATE (CH)-[:CLASIFICADO_EN]->(CP)

/*RELACION PRODUCTO - CATEGORIA */
USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///PRODUCTO_CATEGORIA.csv' AS row
MATCH (P:PRODUCTO),(C:CATEGORIA) WHERE P.ID=row.PRODUCTO_ID AND C.ID=row.CATEGORIA_ID CREATE (P)-[:CLASIFICADO_EN {NIVEL: row.NIVEL} ]->(C)
```


# 4.4 Resumen estadístico tradicional

Se hace una exploración del conjunto de datos para entender el comportamiento de esta: 

Analizando los datos que se generan en la consulta anterior:

![Grafo productos de mercadolibre](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/0_2_PRODUCTOS_VENDIDOS.png)

Se identifica que los productos mas vendidos son los XIAOMI seguidos por los HUAWEI. Sin embargo los productos mas ofertados son los HUAWEI seguido por APPLE y SAMSUMG.

![Grafo productos de mercadolibre](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/0_3_VENDEDORES_VS_PRODUCTOS.png)

Analizando la información de productos ofertados y vendidos por los vendedores. Se identifica que el vendedor que tiene más productos vendidos es _MEGATIENDAVIRTUAL77_ con 716.964 productos sin embargo de estos solo se han ofertado 1.607. Se identifica que el vendedor que tiene más productos ofertados es _EVERPRINTBOGOTA_ con 3.309 productos sin embargo de estos no se registra la venta de ninguno.

![Vendedores vs cantidad de clientes](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/0_4_VENDEDORES_VS_CANTIDAD_DE_CLIENTES.png)

Haciendo una revisión de los vendedores y la cantidad de clientes, se identifica que los vendedores _WILLINTON_MX_, _GAB_AFN_, _SANDERS712_ Y _CELLUPARTESCELLUPARTES_ son los que tienen mas clientes con un valor de 3.100.


# 4.5 Laboratorio

Se realiza un laboratorio con el fin de usar el set de datos suministrado

***•	Querys cypher***

Realizar los querys que respondan las siguientes preguntas:

Cuantos productos existen por categoria?

```cypher
MATCH 
	(p:PRODUCTO)-[r:CLASIFICADO_EN]-(c:CATEGORIA)
RETURN
	 COALESCE(c.NOMBRE,'') as NOMBRE_CATEGORIA, COUNT(p) AS cantidad
ORDER BY cantidad DESC
```

Cuantos vendedores existen y cual es el número de productos que ofertan por categoria?

```cypher
MATCH 
	(u:USUARIO)-[:VENDE]-(p:PRODUCTO)-[:CLASIFICADO_EN]-(c:CATEGORIA)
RETURN
	 u.CODIGO, COALESCE(c.NOMBRE,'') as NOMBRE_CATEGORIA, COUNT(p) AS CANTIDAD
ORDER BY CANTIDAD DESC
```

Cuales son los vendedores que tienen mas opiniones malas?

```cypher
MATCH 
	(uc:USUARIO)-[r:OPINA {TIPO_OPINION:'Mala'}]-(uv:USUARIO) 
return 
	uv.CODIGO,  COUNT(r) AS cantidad 
ORDER BY 
	cantidad DESC
```

***•	Algoritmos***

Los algoritmos se utilizan para calcular métricas de grafos, nodos o relaciones.
Pueden proporcionar información sobre entidades relevantes en el grafo (centralidades, clasificación) o estructuras inherentes como las comunidades (detección de comunidades, partición de grafos, agrupación).
Muchos algoritmos de grafos son enfoques iterativos que frecuentemente atraviesan el grafo para el cálculo utilizando caminos aleatorios, búsquedas de amplitud o de profundidad o coincidencia de patrones.



***•	PageRank***

PageRank es el más conocido de los algoritmos de centralidad. Mide la transitiva influencia de los nodos. PageRank considera la influencia de un nodo vecino y sus vecinos. Por ejemplo, tener unos pocos nodos vecinos muy poderosos
pueden hacer el nodo más influyente que tener muchos vecinos menos poderosos. 

Con lo anterior, podemos responder a la pregunta ¿Cuales son las categorias mas poderosas del grafo?

Para dar respuesta a esa pregunta, se ejecuta el siguiente algoritmo:

```cypher
CALL algo.pageRank("USUARIO", "OPINA", {direction: "BOTH", writeProperty:'PAGE_RANK'})
```

```cypher
CALL algo.pageRank.stream('USUARIO', 'OPINA', {iterations:20, dampingFactor:0.85})
YIELD nodeId, score
RETURN algo.getNodeById(nodeId).CODIGO AS USUARIO, score
ORDER BY score DESC
```

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/22_2_QUERYING_Calculating_PageRank.png">
</p>

Con lo anterior se puede identificar que los usuarios "CELLUPARTESCELLUPARTES", "WILLINTONMX", "GAB_AFN" Y "SANDERS712" son los mas poderosos del grafo.



***•	Community Detection - Label propagation***

El algoritmo de propagación de etiquetas (LPA) es un algoritmo rápido para encontrar comunidades
en un grafo, los nodos seleccionan su grupo en función de sus vecinos directos. Este proceso
es adecuado para redes donde las agrupaciones son menos claras y se pueden usar pesos
para ayudar a un nodo a determinar en qué comunidad ubicarse.

Se puede detectar comunidades en los datos ejecutando un algoritmo que atraviesa la estructura del grafo para encontrar subgrafos altamente conectados con menos conexiones que otros.

Con lo anterior se puede dar respuesta a la pregunta ¿Cuantas comunidades existen y tamaño tiene cada una?

```cypher
CALL algo.labelPropagation('USUARIO','OPINA','OUTGOING', {write:true, partitionProperty: "community", weightProperty:"count"})
```

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/2_0_LABEL_PROPAGATION_CANTIDAD_COMUNIDADES.png">
</p>

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/2_1_LABEL_PROPAGATION_COMUNIDADES.png">
</p>

Con lo anterior se puede identificar que hay 11.640 comunidades, donde la comunidad mas grande tiene 2.840 usuarios.


***•	Visualización***

Para poder visualizar las estadísticas que proporcionan los algoritmos de neo4j. Con la librería de NEOVIS.JS se puede mostrar la información de grafos de manera eficiente.

Tomando como base el procedimiento para visualización de información de https://github.com/jackdbd/react-neovis-example.

Configurando las propiedades y el query de visualización de información:

```cypher
    const config = {
      container_id: this.visRef.current.id,
      server_url: neo4jUri,
      server_user: neo4jUser,
      server_password: neo4jPassword,
      labels: {
        USUARIO: {
          caption: "CODIGO",
          size: "PAGE_RANK_C2",
          community: "COMMUNITY_C2"
        },
        TIPO_PRODUCTO_CATEGORIA2: {
          caption: "NOMBRE",
          size: "CANTIDAD",
          community: "NOMBRE"
        }
      },
      relationships: {
        OFERTA_CATEGORIA2: {
          caption: false,
          thickness: "CANTIDAD"
        }
      },
      initial_cypher:
        "MATCH (U:USUARIO)-[R:OFERTA_CATEGORIA2]->(TP:TIPO_PRODUCTO_CATEGORIA2) WHERE toInt(R.CANTIDAD) >= 500 RETURN U, R, TP"
    };
```

Al ejecutar la visualización se muestra de la siguiente manera:

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/30_VISUALIZACION_USUARIO_OFERTADOS_CATEGORIA_2_MAYOR_100.png">
</p>

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/30_VISUALIZACION_USUARIO_OFERTADOS_CATEGORIA_2.png">
</p>

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/30_VISUALIZACION_USUARIO_OFERTADOS_CATEGORIA_3.png">
</p>

Como conslusión, se puede observar que la categoria de APPLE y GOOGLE son las mas importantes para la venta de productos.



Actividad...
