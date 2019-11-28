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

CREATE CONSTRAINT ON (n:DEPARTAMENTO) ASSERT n.ID IS UNIQUE

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///MUNICIPIO.csv' AS row
CREATE (:MUNICIPIO { ID : row.ID, NOMBRE : row.NOMBRE})

CREATE CONSTRAINT ON (n:MUNICIPIO) ASSERT n.ID IS UNIQUE

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///MUNICIPIO.csv' AS row
MATCH (d:DEPARTAMENTO),(m:MUNICIPIO) WHERE d.ID=row.DEPARTAMENTO_ID AND m.ID=row.ID CREATE (m)-[:MUNICIPIO_EN]->(d)

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///USUARIO.csv' AS row
CREATE (:USUARIO { ID : row.ID, NOMBRE : row.NOMBRE, ES_VENDEDOR : row.ES_VENDEDOR, ES_COMPRADOR : row.ES_COMPRADOR, URL : row.URL})

CREATE CONSTRAINT ON (n:USUARIO) ASSERT n.ID IS UNIQUE

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///USUARIO.csv' AS row
MATCH (m:MUNICIPIO),(u:USUARIO) WHERE m.ID=row.MUNICIPIO_ID AND u.ID=row.ID CREATE (u)-[:UBICADO_EN]->(m)

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///OPINION.csv' AS row
MATCH (uc:USUARIO),(uv:USUARIO) WHERE uc.ID=row.USUARIO_COMPRADOR_ID AND uv.ID=row.USUARIO_VENDEDOR_ID CREATE (uc)-[:OPINA {TIPO: row.TIPO, FECHA:row.FECHA, OPINION: row.OPINION} ]->(uv)

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///PRODUCTO.csv' AS row
CREATE (:PRODUCTO { ID : row.ID, NOMBRE : row.NOMBRE, VALOR : row.VALOR, ES_NUEVO : row.ES_NUEVO, URL : row.URL, USUARIO_ID : row.USUARIO_ID})

CREATE CONSTRAINT ON (n:PRODUCTO) ASSERT n.ID IS UNIQUE

USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///PRODUCTO.csv' AS row
MATCH (p:PRODUCTO),(u:USUARIO) WHERE p.ID=row.ID AND u.ID=row.USUARIO_ID CREATE (u)-[:VENDE]->(p)

/*CATEGORIA*/
USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///CATEGORIA.csv' AS row
CREATE (:CATEGORIA { ID : row.ID, NOMBRE : row.NOMBRE})

CREATE CONSTRAINT ON (n:CATEGORIA) ASSERT n.ID IS UNIQUE

/*RELACION CATEGORIA - CATEGORIA */
USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///CATEGORIA.csv' AS row
MATCH (ch:CATEGORIA),(cp:CATEGORIA) WHERE ch.ID=row.ID AND cp.ID=row.CATEGORIA_PADRE_ID CREATE (ch)-[:CLASIFICADO_EN]->(cp)

/*RELACION PRODUCTO - CATEGORIA */
USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'file:///PRODUCTO_CATEGORIA.csv' AS row
MATCH (p:PRODUCTO),(c:CATEGORIA) WHERE p.ID=row.PRODUCTO_ID AND c.ID=row.CATEGORIA_ID CREATE (p)-[:CLASIFICADO_EN {NIVEL: row.NIVEL} ]->(c)

CREATE CONSTRAINT ON (n:PRODUCTO) ASSERT n.ID IS UNIQUE

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


# 4.5 Laboratorio de querys cypher sobre el conjunto de datos 

Se realiza un laboratorio con el fin de usar el set de datos suministrado. Para esto se debe responder al siguiente cuestionario.

•	Listar los departamentos
```cypher
MATCH (d:DEPARTAMENTO) RETURN d
```

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/40_respuestas_1_departamentos.png">
</p>

•	Listar los municipios del departamento "BOGOTÁDC" en orden alfabetico
```cypher
MATCH (m:MUNICIPIO)-[:MUNICIPIO_EN]-(d:DEPARTAMENTO) WHERE d.NOMBRE='BOGOTÁDC' RETURN m.NOMBRE as nombre_municipio order by nombre_municipio
```

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/41_respuestas_2_departamentos_mun.png">
</p>


•	Listar los 5 primeros municipios y su departamento, que tengan mas usuarios vendedores
```cypher
MATCH (u:USUARIO {ES_VENDEDOR:1})-[:UBICADO_EN]-(m:MUNICIPIO)-[:MUNICIPIO_EN]-(d:DEPARTAMENTO) RETURN d.NOMBRE as departamento, m.NOMBRE as municipio,  count(u) as cantidad order by cantidad desc limit 5
```

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/42_respuestas_3_departamentos_mun_mas_usu_vendedo.png">
</p>

•	Cual es el vendedor con el máximo número de opiniones malas y cuales son sus caracteristicas. Que lo diferencia de los usuarios que venden productos similares?

```cypher
XXX
```

•	Cuales son los 5 usuarios que mas han dado opiniones malas ?
```cypher
MATCH(uc:USUARIO)-[:OPINA {TIPO_OPINION:'Mala'}]-(uv:USUARIO) WITH count(1) as cantidad,uc.NOMBRE as nombre WHERE cantidad>1 RETURN nombre, cantidad order by cantidad desc limit 5
```

•	Cuales son los 'IPHONE 8 PLUS' mas caros que se venden?
```cypher
MATCH (p:PRODUCTO)-[:CLASIFICADO_EN]-(c:CATEGORIA {NOMBRE:'IPHONE 8 PLUS'}) RETURN c.NOMBRE, p.NOMBRE, toInt(REPLACE(p.VALOR_CON_DESCUENTO,'.','')) as valor order by valor desc limit 5
```

NOTA: Los valores de producto vienen en formato texto, se debe copnsiderar hacer limpieza de este campo para poder tratarlo como numérico.


# 4.6 Algoritmos de analítica de grafos 

Los algoritmos se utilizan para calcular métricas de grafos, nodos o relaciones.
Pueden proporcionar información sobre entidades relevantes en el grafo (centralidades, clasificación) o estructuras inherentes como las comunidades (detección de comunidades, partición de grafos, agrupación).
Muchos algoritmos de grafos son enfoques iterativos que frecuentemente atraviesan el grafo para el cálculo utilizando caminos aleatorios, búsquedas de amplitud o de profundidad o coincidencia de patrones.


Se realiza un laboratorio con el fin de usar el set de datos suministrado. Para esto se debe responder al siguiente cuestionario.

•	Cuales son los usuarios mas influyentes en el grafo?

NOTA: Para identificar nodos influyentes, se pueden aplicar algotirmos de centralidad como Page Rank.


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

Con lo anterior se puede identificar que los usuarios "CELLUPARTESCELLUPARTES", "WILLINTONMX", "GAB_AFN" Y "SANDERS712" son los mas influyentes del grafo.


•	Identificar las comunidades de usuarios que existen en el grafo.

NOTA: Se pueden identificar comunidades de un grafo, aplicando algoritmos de detección de comunidades como es Label propagation.


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


# 4.6 Visualización 

Con las comunidades detectadas, visualizar las comunidades que tengan 65 usuarios.

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
          size: "PAGE_RANK_F1",
          community: "COMUNIDAD_F"
        }
      },
      relationships: {
        OPINA: {
          caption: false,
          thickness: "CANTIDAD"
        }
      },
      initial_cypher:
        "MATCH (U1:USUARIO)-[R:OPINA]->(U2:USUARIO) WHERE U1.community2=1716 OR U1.community2=1999 OR U1.community2=2503 OR U2.community2=1716 OR U2.community2=1999 OR U2.community2=2503 RETURN U1, R, U2"
    };
```

Al ejecutar la visualización se muestra de la siguiente manera:

<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/31_3_VISUALIZACION_USUARIOS_65_personas.png">
</p>
