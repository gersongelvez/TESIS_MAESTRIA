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


# 4.4 Aplicar algoritmos Neo4j sobre el conjunto de datos de Mercadolibre

Los algoritmos se utilizan para calcular métricas de grafos, nodos o relaciones.
Pueden proporcionar información sobre entidades relevantes en el grafo (centralidades, clasificación) o estructuras inherentes como las comunidades (detección de comunidades, partición de grafos, agrupación).
Muchos algoritmos de grafos son enfoques iterativos que frecuentemente atraviesan el grafo para el cálculo utilizando caminos aleatorios, búsquedas de amplitud o de profundidad o coincidencia de patrones.

***•	Resumen estadístico tradicional***

Se hace una exploración del conjunto de datos antes de ejecutar algoritmos más complejos. 

La siguiente consulta da un detalle de los vendedores y las de categorias de los productos que venden.

```cypher
match 
	(U:USUARIO)-[RUP:VENDE]-(P:PRODUCTO) 
	optional match (P)-[RPC1:CLASIFICADO_EN {NIVEL:'0'} ]-(C1:CATEGORIA) 
	optional match (P)-[RPC2:CLASIFICADO_EN {NIVEL:'1'} ]-(C2:CATEGORIA) 
	optional match (P)-[RPC3:CLASIFICADO_EN {NIVEL:'2'} ]-(C3:CATEGORIA)
	optional match (P)-[RPC4:CLASIFICADO_EN {NIVEL:'3'} ]-(C4:CATEGORIA)
	optional match (P)-[RPC5:CLASIFICADO_EN {NIVEL:'4'} ]-(C5:CATEGORIA)
RETURN
	U.CODIGO, 'VEN '+ID(U) AS ALIAS1, U.ID AS ALIAS2, COALESCE(C3.ID,'') as CATEGORIA_GENERAL, COALESCE(C3.ID,'')+' - '+COALESCE(C4.ID,'') as CATEGORIA, COUNT(P.NOMBRE) AS OFERTADOS, SUM(toInt(COALESCE(P.VENDIDOS,0))) AS VENDIDOS
ORDER BY VENDIDOS DESC
```

Analizando los datos que se generan en la consulta anterior:

![Grafo productos de mercadolibre](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/0_2_PRODUCTOS_VENDIDOS.png)

Se identifica que los productos mas vendidos son los XIAOMI seguidos por los HUAWEI. Sin embargo los productos mas ofertados son los HUAWEI seguido por APPLE y SAMSUMG.

![Grafo productos de mercadolibre](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/0_3_VENDEDORES_VS_PRODUCTOS.png)

Analizando la información de productos ofertados y vendidos por los vendedores. Se identifica que el vendedor que tiene más productos vendidos es _MEGATIENDAVIRTUAL77_ con 716.964 productos sin embargo de estos solo se han ofertado 1.607. Se identifica que el vendedor que tiene más productos ofertados es _EVERPRINTBOGOTA_ con 3.309 productos sin embargo de estos no se registra la venta de ninguno.

![Vendedores vs cantidad de clientes](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/0_4_VENDEDORES_VS_CANTIDAD_DE_CLIENTES.png)

Haciendo una revisión de los vendedores y la cantidad de clientes, se identifica que los vendedores _WILLINTON_MX_, _GAB_AFN_, _SANDERS712_ Y _CELLUPARTESCELLUPARTES_ son los que tienen mas clientes con un valor de 3.100.


***•	Resumen estadístico mediante algoritmos de grafos***
Se puede identificar el comportamiento de la información mediante unos algoritmos proporcionados por NEO4J. Se pueden responder unas preguntas.


***•	PageRank***

PageRank es el más conocido de los algoritmos de centralidad. Mide la transitiva influencia de los nodos. PageRank considera la influencia de un nodo vecino y sus vecinos. Por ejemplo, tener unos pocos nodos vecinos muy poderosos
pueden hacer el nodo más influyente que tener muchos vecinos menos poderosos. 

Con lo anterior, podemos responder a la pregunta ¿Cuales son los usuarios mas poderosos del grafo?

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

***•	Grado de centralidad***

Cuál es el usuario mas popular en el grafo?

```cypher
CALL algo.degree.stream("USUARIO", "OPINA", {
  direction: "BOTH"
})
YIELD nodeId, score
RETURN algo.asNode(nodeId).CODIGO AS USUARIO, score
ORDER BY score DESC
LIMIT 10
```

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/5_DEGREE_CENTRALITY.png">
</p>

El algoritmo "Degree Centrality" ayuda a identificar el nodo mas popular en el grafo, dando un score según el número de relaciones entrantes y salientes. Para el grafo que se está analizando, se identifica que los USUARIOS mas populares son "GAB_AFN", "WILLINTONMX" y "CELLUPARTESCELLUPARTES".

***•	Camino mas corto***
Se busca el diametro del grafo para los vendedores y clientes, tomando el camino mas corto. Para eso se usa el siguiente código:

```cypher
MATCH (U1:USUARIO)-[:OPINA]-(U2:USUARIO) WHERE id(U1) > id(U2)
MATCH path = shortestPath((U1)-[:OPINA]-(U2))
RETURN path, length(path) AS len
ORDER BY len DESC
LIMIT 100
```

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/6_SHORTEST_PATH.png">
</p>

Tomando el camino mas corto, se identifica que hay nodos separados de la red y un grupo de nodos unidos.

***•	Betweenness Centrality***

Con este algoritmo se puede identificar el vendedor mas influyente en el grafo.
```cypher
CALL algo.betweenness.stream("USUARIO", "OPINA", {
  direction: "BOTH"
})
YIELD nodeId, centrality
RETURN algo.asNode(nodeId).CODIGO, centrality
ORDER BY centrality DESC
LIMIT 10
```

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/9_Betweenness_Centrality.png">
</p>

Se identifica que el vendedor mas influyente es _GAB_AFN_.


***•	Almacenando el score del algoritmo Betweenness Centrality***

Debido a que normalmente los grafos son demasiado grandes, una manera óptima es almacenar el valor de retornado por el algoritmo en cada nodo, para así posteriormente se puedan hacer consultas subconjuntos del grafo.

El siguiente código fuente almancena el resultado del algoritmo en la propiedad BETWEENNESS_CENTRALITY del nodo USUARIO: 

```cypher
CALL algo.betweenness("USUARIO", "OPINA", {direction: "BOTH", writeProperty: "BETWEENNESS_CENTRALITY"})
```

```cypher
Con el valor almacenado, se puede proceder a consultar:
MATCH (U:USUARIO)
RETURN U.CODIGO, U.BETWEENNESS_CENTRALITY AS centrality
ORDER BY centrality DESC
LIMIT 10
```
<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/12_Querying_Betweenness_Centrality.png">
</p>




***•	Closeness Centrality***

La centralidad de proximidad es una forma de detectar nodos que pueden difundir información de manera muy eficiente a través de un gráfico. La centralidad de proximidad de un nodo mide su lejanía promedio (distancia inversa) a todos los demás nodos. Los nodos con un alto puntaje de cercanía tienen las distancias más cortas a todos los demás nodos.

Podemos ejecutar este algoritmo sobre los usuarios de mercadolibre:

```cypher
CALL algo.closeness.stream("USUARIO", "OPINA", {
  direction: "BOTH"
})
YIELD nodeId, centrality
RETURN algo.asNode(nodeId).CODIGO, centrality
ORDER BY centrality DESC
LIMIT 10
```
<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/18_Closeness_Centrality.png">
</p>


***•	Usuarios bien conectados***

¿Por qué Daenerys está tan bien conectado?

De manera predeterminada, el algoritmo de Centralidad de cercanía calcula la conexión de un nodo con todos los nodos a los que puede llegar. Podemos ejecutar el algoritmo de componentes conectados para encontrar conjuntos de nodos que tengan rutas entre sí.

```cypher
CALL algo.unionFind.stream("USUARIO", "OPINA", {
  direction: "BOTH"
})
YIELD nodeId, setId
WITH setId, collect(algo.asNode(nodeId).CODIGO) AS USUARIO
RETURN setId, USUARIO, size(USUARIO) AS size
ORDER BY size(USUARIO) DESC
LIMIT 10
```
<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/19_USUARIO_BIEN_CONECTADA.png">
</p>

***•	Closeness Centrality: Wasserman and Faust / Harmonic***


Por lo tanto, el algoritmo de centralidad de proximidad realmente mide la lejanía de un nodo a todos los demás nodos en el mismo componente conectado. Si queremos encontrar la lejanía para todos los demás nodos en el gráfico, podemos usar las variantes de Wasserman y Faust o Harmonic del algoritmo.

***Wasserman and Faust***

```cypher
CALL algo.closeness.stream("USUARIO", "OPINA", {
  direction: "BOTH", improved: true
})
YIELD nodeId, centrality
RETURN algo.asNode(nodeId).CODIGO, centrality
ORDER BY centrality DESC
LIMIT 10
```

***Harmonic***

```cypher
CALL algo.closeness.harmonic.stream("USUARIO", "OPINA", {
  direction: "BOTH"
})
YIELD nodeId, centrality
RETURN algo.asNode(nodeId).CODIGO, centrality
ORDER BY centrality DESC
LIMIT 10
```




***Analizando las comunidades***
  
```cypher
MATCH (U:USUARIO)
WHERE exists(U.community)
RETURN U.community, count(*) AS count
ORDER BY count DESC
```
<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/25_Querying_Communities_USUARIO.png">
</p>

```cypher
CALL algo.pageRank(
  'MATCH (U:USUARIO) RETURN id(U) as id',
  'MATCH (U:USUARIO)-[rel:OPINA]-(U2) RETURN id(U) as source,id(U2) as target, SUM(rel.weight) as weight',
  {graph:'cypher', writeProperty: 'pageRank'})  
```

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/25_Querying_Communities_USUARIO.png">
</p>

***Visualizando las comunidades***


Podemos escribir el siguiente código para ver la interaccion entre las personas de mercadolibre:

```cypher
MATCH (U:USUARIO) WHERE exists(U.community)
WITH U.community AS community, COUNT(*) AS count
ORDER BY count DESC
SKIP 1 LIMIT 1
MATCH path = (U:USUARIO {community: community})--(U2:USUARIO {community: community})
RETURN path
```

El grafo se ve desde la siguiente manera:

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/27 Visualising Communities.png">
</p>
