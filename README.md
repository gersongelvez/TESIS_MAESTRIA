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

