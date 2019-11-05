# Analítica de Grafos Sobre Productos de Mercado Libre
Se suben los artefactos para la tesis de analitica de datos con neo4j

# Esquema del grafo
La información de compras y ventas en la categoría de tecnología de mercadolibre para la ciudad de Bogotá. Se puede esquematizar en un modelo relacional de la siguiente forma:

![MER productos de mercadolibre](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/0_1_MER_productos.png)
 
La misma información se puede expresar en un esquema de grafos de la siguiente manera:

![Grafo productos de mercadolibre](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/0_1_Grafo.jpg)
 
# Importar datos en Neo4j

La información recolectada con técnicas de webscraping se plasma en archivos CSV. Neo4j con su lenguaje Cypher proporciona una opción de importación de datos, usando el comando LOAD CSV para transformar el contenido de los archivos CSV a una estructura de grafos.
Se deben seguir los siguientes pasos para cargar los datos a Neo4j:

•	Los datos se pueden descargar en la URL (Carrillo. Gerson, 2019, https://github.com/gersongelvez/TESIS_MAESTRIA/tree/master/DATOS) 

•	Se debe tener un motor de bases de datos Neo4j instalado y en correcto funcionamiento. Los archivos descargados en el punto anterior, se deben poner en la carpeta import del servidor donde se encuentra instalado Neo4j. Este punto es necesario para usar la herramienta de importación de archivos CSV.

•	Ejecutar los comandos de importación uno a uno en la consola de Neo4j:

```cypher
match(d:departamento) return d.id as id, d.codigo as codigo ,d.nombre as nombre

match(m:municipio)-[rmd:municipio_en]-(d:departamento) return m.id as id, m.codigo as codigo ,m.nombre as nombre, d.id as departamento_id

match(u:usuario) optional match(u)-[rum:ubicado_en]-(m:municipio) return u.id as id, u.nombre as nombre, u.es_vendedor as es_vendedor, u.es_comprador as es_comprador, u.url as url, m.id as municipio_id order by  es_vendedor desc, id

match (u:usuario)-[rup:vende]-(p:producto) return p.id as id, p.nombre as nombre, p.valor_con_descuento as valor, p.es_nuevo as es_nuevo, p.id as url, u.id as usuario_id order by usuario_id, nombre

match (c:categoria) optional match(c)-[rcc:clasificado_en]-(c1:categoria) return c.id as id, c.nombre as nombre, c1.id as categoria_padre_id order by nombre

match (p:producto)-[rpc:clasificado_en]-(c:categoria) return p.id as producto_id, c.id as categoria_id, rpc.nivel as nivel order by producto_id, nivel

match(uv:usuario)-[ro:opina]-(uc:usuario) return ro.tipo_opinion as tipo, ro.fecha as fecha, ro.opinion as opinion, uv.id as usuario_vendedor_id, uc.id as usuario_comprador_id

```


# Aplicar algoritmos Neo4j sobre el conjunto de datos de Mercadolibre

Los algoritmos se utilizan para calcular métricas de grafos, nodos o relaciones.
Pueden proporcionar información sobre entidades relevantes en el grafo (centralidades, clasificación) o estructuras inherentes como las comunidades (detección de comunidades, partición de grafos, agrupación).
Muchos algoritmos de grafos son enfoques iterativos que frecuentemente atraviesan el grafo para el cálculo utilizando caminos aleatorios, búsquedas de amplitud o de profundidad o coincidencia de patrones.

***•	Resumen estadístico***

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

***•	Grado de centralidad***

Este algoritmo mide el número de relaciones entrantes y salientes de un nodo. Con esto se puede identificar los nodos más populares en el grafo. Con el siguiente comando se puede aplicar este algoritmo:

```cypher
CALL algo.degree.stream("USUARIO", "OPINA", {
  direction: "BOTH"
})
YIELD nodeId, score
RETURN algo.asNode(nodeId).CODIGO AS USUARIO, score
ORDER BY score DESC
LIMIT 10
```
||
|:---:|
|![5 grado de centralidad](https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/5_DEGREE_CENTRALITY.png)|


->Text<-
```Text```
<p align="center">
<img src="https://github.com/gersongelvez/TESIS_MAESTRIA/blob/master/IMAGENES/5_DEGREE_CENTRALITY.png">
</p>

Se confirman los mismos resultados obtenidos en el analisis de los vendedores vs sus clientes. Debido a que este algoritmo identifica los números de relaciones entrantes y salientes del nodo.
