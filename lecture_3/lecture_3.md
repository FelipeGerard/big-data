Big data :: Lecture 3
========================================================
author: Adolfo De Unánue T.
date: 4 de marzo, 2015
font-import: http://fonts.googleapis.com/css?family=Risque
font-family: 'Risque'
css: ../css/itam_big_data.css



========================================================
type: exclaim

## Bases de datos




Bases de datos
=========================================================
* Un RDBMS es la solución para datos no tan grandes...
    - Pero mas grandes que MS Excel.
* Adecuada para todas las tareas
    - Pero no es excelente en ninguna.
* Fácil de usar
    - Pocos requerimientos de HW.
    - Soporte en casi todos los lenguajes.
    - Todo el mundo la conoce.
* En realidad, no puede hacer big, big data.

Bases de datos Relacionales
=========================================================

- **Edward F. Codd**
  - Héroes verdaderos, no como *Steve Jobs*.

- Contienen *relaciones* (`tablas`), las cuales tienen un conjunto de *tuplas* (`renglones`), las cuales
mapean *atributos* a valores *atómicos*, los cuales quedan definidos por una *tupla header* mapeado a un *dominio* (`columnas`).

- Originalmente `booleanas`, la implementación actual es `logica trivaluada` (`TRUE`, `FALSE`, `NULL`).


Bases de datos Relacionales
=========================================================

- No son **relacionales** por que las tablas estén relacionadas por campos y restricciones.

- Son **relacionales**  por que están construidas sobre la rama matemática de *álgebra relacional*.

- Aunque en la implementación es simplificada a la rama de *cálculo relacional de tuplas* el cuál se puede convertir en *álgebra relacional*.

========================================================
type: exclaim

## PostgreSQL

¿Por qué PostgreSQL?
========================================================

* Open-source
* Es una base de datos orientada a objetos.
* Varios tipos de [índices](http://www.postgresql.org/docs/9.3/static/indexes.html)
* `JOIN`s optimizados
    - 5 diferentes tipos de joins
    - Soporta optimizaciones (vía GA) de más de 20 tablas.
* Subqqueries en cualquier cláusula.
* Subqueries anidados.
* Windowing functions
* Geoespacial

¿Por qué PostgreSQL?
========================================================

* Recursive queries
* Soporta `XML`, `JSON`, arreglos...
* Extensibilidad:
    - Bibliotecas externas
    - Lenguajes externos
    - Puedes crear tipos de datos, funciones, agregadores, operadores, etc.
* Foreign Data Wrappers (FDW)
* Creación concurrente de índices
* Listen/Notify
* NOSQL dentro de SQL


¿Dónde está PostgreSQL según el CAP?
======================================================
![cap-theorem](images/scalability-cap-theorem1.png)

- Tomado de una página de **IBM**
  - perdí el `URL` :( ...

========================================================
type: exclaim

## PostgreSQL
### Instalación


Instalación
=======================================================

- En distros basadas en `Debian`:

```
> sudo apt-get update
> sudo apt-get -y install python-software-properties
> sudo apt-get install postgresql-9.4 libpq-dev postgresql-contrib
```

- Accesando a la base

```
> sudo su postgres
postgres$ psql
```
- A partir de aquí habrá que crear usuarios

Instalación
=======================================================
- Por omisión, guardará todo en `/`, detenemos y destruimos:

```
> sudo pg_dropcluster --stop 9.4 main
```

- Creamos uno nuevo

```
> sudo pg_createcluster -d ~/data 9.4 main
```

Instalación
=======================================================

- Ahora PostGIS `:)`

```
> sudo apt-get update
> sudo apt-get install postgis
```

- Dentro de `psql` (un poco más adelante)

```
# \dx+
# create extension postgis;
# create extension postgis_topology;
# create extension fuzzystrmatch;
# create extension postgis_tiger_geocoder;
# \dx+
# select postgis_version();
```


========================================================
type: exclaim

## PostgreSQL
## `psql`


psql
=======================================================

- Es el cliente *par excellence* de PostgreSQL.
  - Algunos prefieren `pgadmin`, no lo veremos en el curso...

- Como he dicho en otras ocasiones, las interfaces de línea de comandos (`CLI`) son las más eficientes
  - Y las mejores para **big data**. Punto.

psql
=======================================================

### Modo interactivo

- `\?` les indica que comandos de `psql` existen
  - Algunos comandos útiles: `\l`, `\connect`, `\d`, `\dt`, `\a`,  `\x`, `\i`, `\o`, `\g`, `\!`, `\pset pager`, `\timing on/off`
- `\help` adelante de una sentencia `SQL` les muestra la ayuda de la sentencia
  - **Ejercicio** Intenten `\help select` y ejecútenlo cada vez que veamos un comando de `SQL` que desconozcan.

psql
=======================================================

### Modo interactivo

- Definir un *alias*
  - `\set eav 'EXPLAIN ANALYZE VERBOSE'`

- Otro alias interesante:

```
\set show_slow_queries
'SELECT
  (total_time / 1000 / 60) as total_minutes,
  (total_time/calls) as average_time, query
FROM pg_stat_statements
ORDER BY 1 DESC
LIMIT 100;'
```

psql
=======================================================

### Modo no interactivo

- Para evitar que pregunte la contraseña, creen un archivo `.pgpass` en su `$HOME`
con la siguiente sintaxis:

```
host:port:*:username:password
```

Una línea por cada conexión.


psql
=======================================================

### Modo no interactivo

- Conexión

```
psql -h host -U user -d base_de_datos
```

- Ejecutar un archivo `.sql`

```
psql -f script.sql
```

- Ejecutar un comando `SQL`

```
psql -d base_de_datos -c "SELECT * from pg_tables limit 1;"

## Aquí es un caso donde el infierno de las comillas (quotes) puede aparecer.
```

psql
=======================================================

- **Ejercicio**: Averigua que hace la famosa bandera **axe cutie**: `-Ax -qt` ¿En que circunstancia lo usarías? ¿Y la bandera `-e`?

psql
=======================================================

### Modo interactivo

- Algunas mejoras (ejecuta un `select * from pg_tables` entre cada uno de los siguientes.)

```
\pset linestyle unicode  -- Mejora la línea de presentación

\pset border 2  -- Borde a 2 px

\pset pager off  -- Deshabilita el pager

\pset null '[NULL]' -- Qué imprimir cuando NULL
```

psql
=======================================================

### Modo interactivo

- Mejorar el **[`PROMPT1`](http://www.postgresql.org/docs/9.3/static/app-psql.html#APP-PSQL-PROMPTING)**

```
\set PROMPT1 '%[%033[33;1m%]%x%[%033[0m%]%[%033[1m%]%/%[%033[0m%]%R%# '
-- PROMPT2 es lo que imprime cuando está esperando más input
\set PROMPT2 '[más] %R > '
```

-  Obviamente pueden agregar todo  a su **`.psqlrc`**, sólo recuerden desactivar cuando ejecuten un comando de forma no interactiva.
    - con `-X`, por si les dió flojera leer el `man`.


========================================================
type: exclaim

## PostgreSQL
### Tipos de datos

Tipos de datos
========================================================
- `smallint`, `integer`, `bigint`, `numeric`, `float`
  - Nunca usen `money`

- `uuid`
  - Ver [esta extensión](http://www.postgresql.org/docs/9.4/static/uuid-ossp.html).

- `boolean`, `char`, `varchar`, `text` ...

- `inet`, `cidr`, `macaddr`

- `date`, `timestamp`, `time`

- Más los que agregan las extensiones


Arreglos
=======================================================

- [Documentación](http://www.postgresql.org/docs/9.4/static/arrays.html)

- ¿Cuándo usarlos?
  - En cualquier caso donde tus relaciones acaben siendo

  ```

  | ... | campo1 | campo2 | campo3 | ... |

  ```

Arreglos
=======================================================

- Es importante que no son estructuras de datos como en otros lenguajes
    - PostgreSQL  accesa los elementos del arreglo de manera secuencial, ya que los guarda como un arreglo de valores, no de apuntadores.
    - Esto ocurre con arreglos variables como `hstore`, `json`, `varchar`, `text`, etc.

- Para un ejemplo ver esta [página](http://blog.heapanalytics.com/dont-iterate-over-a-postgres-array-with-a-loop/).

- Una manera de darle la vuelta es usando `unnest`.

Arreglos
=======================================================

```
create table cosas (
  id serial not null,
  nombre varchar,
  etiquetas varchar [],
  created_at timestamp,
  updated_at timestamp
);
```

* Insertamos de la siguiente manera:

```
insert into cosas
values (1, 'Cosa 1', '{"Algo", "Otra cosa"}', now(), now());
insert into cosas
values (2, 'Cosa 2', '{"Algo más", "cosa"}', now(), now());
insert into cosas
values (3, 'Cosa 3', '{"Algo", "cosa"}', now(), now());
```

Arreglos
=======================================================

* Seleccionamos usando el operador de contención `@>`
  - Más operadores [aquí](http://www.postgresql.org/docs/9.4/static/functions-array.html).

```
select nombre from cosas
where etiquetas @> '{Algo}';
```



* Seleccionamos usando unnest

```
select unnest(etiquetas) from cosas;
```


Rangos
=======================================================

* [Documentación](http://www.postgresql.org/docs/9.4/static/rangetypes.html)

* Tipos
  - `int4range`
  - `int8range`
  - `numrange`
  - `tsrange`
  - `tstzrange`
  - `daterange`


Rangos
=======================================================

```
create table clases (
  salon varchar,
  periodo tsrange
);
```

* Insertamos de la siguiente manera:

```
insert into clases
values ('Sala de video 2', '[2015-03-04 08:00, 2015-03-04 11:00]');
```

Tipos compuestos
=======================================================

- El ejemplo paradigmático es el de los números complejos $\mathbf{C}$

```
create type complejo as (
  real double precision,
  img double precision
  );

create table test (
  num_complejo complejo,
  num_real double precision
);

```


Tipos compuestos
=======================================================

- ¿Cómo se usa?

```
-- Insertar
insert into test values (row(1.0, 32), 1000);
insert into test values ((1.234, 2), 0);

-- Es preferible usar row para no escapar quotes :(

-- Seleccionar por atributo
select (num_complejo).real from test;

-- Actualizar
update test  set num_complejo = (2, 64)
  where num_real = 1000;

update test  set num_complejo.real = 4
  where num_real = 1000;
-- Nota la asimetría con el select
```


========================================================
type: exclaim

## PostgreSQL
## Configuración


Configuración
=======================================================

- **`postgres.conf`** Configuración general.
- **`pg_hba.conf`** Controla la seguridad.
- **`pg_ident.conf`** Mapea los usuarios del sistema a usuarios de postgresql.

```
select name, setting
from pg_settings
where category = 'File Locations';
```

```
select name, context, unit, setting, boot_val, reset_val
from pg_settings
where name in (
'listen_addresses', 'max_connections', 'shared_buffers',
'effective_cache_size', 'work_mem', 'maintenance_work_mem'
)
order by context, name;
```

Configuración
=======================================================

- **`pg_hba.conf`**:

  - Método de autenticación: Usen `md5` para ss usuarios remotos, `ident` para los locales
  - En `connections` la notación es `IPv4 / IPv6` si quieren conexiones remotas, pueden empezar con `0.0.0.0/0` (esto abre todas!).
  - Limiten que el usuario `postgres` y su usuario `dba` sólo se conecten localmente (creen un usuario de `SO` para el `dba`).

Configuración
=======================================================

- **`pg_hba.conf`**:

```
local     all    postgres     peer
local     all      all        md5

...

host      all      all     0.0.0.0/0  md5
```


Administración básica
=======================================================

- Crear usuarios:

```
create role leonidas login password 'king0fSpart4'
createdb valid until 'infinity';
```

```
create role dario login password 'persianRuler'
superuser valid until '2020-10-20 03:00:00';
```

- Grupo

```
create role persians inherit;
```

- Agregando alguien al grupo

```
grant persians to some_guy_about_to_die;
```

Administración básica
=======================================================

- Crear una base de datos:

```
create database ufo;

-- Pueden agregar OWNER dueño y TABLESPACE table_space
```

- Esquemas (para organizar la base de datos)

```
create schema dirty;

-- Podemos agregar un dueño con AUTHORIZATION dueño
```

Administración básica
=======================================================

   **Ejercicio:** Crea los esquemas:
      `"$user", dirty, clean, shameful, playground, output, mining, ml`

- Modifica el `path` de búsqueda

```
alter database ufo set search_path="$user", dirty, clean, shameful, playground, output, mining, ml;
```

Administración básica
=======================================================

- Permisos en el esquema

```
grant usage on schema dirty to public;

alter default privileges in schema dirty
grant select, references on tables
  to public;

alter default privileges in schema dirty
grant select, update on sequences
  to public;

alter default privileges in schema dirty
grant execute on functions
  to public;

alter default privileges in schema dirty
grant usage on types
  to public;
```

Administración básica
=======================================================


- Si el esquema ya está poblado:

```
grant usage on schema clean to public;

grant select, references, trigger
  on all tables in schema clean
    to public;

grant execute on all functions in schema clean to public;
grant select, update on all sequences in schema clean to public;
```

Extensiones
======================================================

- Listar las disponibles

```
select *
from pg_available_extensions
where comment like '%string%' or installed_version is not null
order by name
```

- Es equivalente a `\dx` en `psql`.

- En `psql`: `\dx+ fuzzystrmatch` para obtener más información.


Extensiones
======================================================


- Instalar extensiones

```
create extension fuzzystrmatch schema mis_extensiones;
```

- Las extensiones se deben de instalar *por base de datos*.

**Ejercicio**: Instalar las siguientes extensiones: `dblink`, `file_fdw`, `fuzzystrmatch`, `hstore`
, `pgcrypto`, `postgres_fdw`, `tablefunc`, `cube`, `dict_xsyn`, `pg_trgm`, `uuid-ossp`.


========================================================
type: exclaim

## PostgreSQL
### Operaciones básicas


Operaciones básicas
=======================================================

* `SELECT`

* `INSERT`

* `UPDATE`

* `DELETE`

SELECT
======================================================

- En teoría es sencillo, pero ejecuta lo que sigue
    - En pantalla completa

```

postgres=# \help select

Command:     SELECT
Description: retrieve rows from a table or view
Syntax:

...

```

SELECT
=======================================================

Existen además:

- `VALUES` regresa una tabla luego de evaluar las expresiones.

- `TABLE` es un `SELECT * FROM`.

- **Ejercicio** Usa los `\help` para ver como son.


Modificadores básicos
=======================================================

* `GROUP BY`

* `ORDER BY`

* `HAVING`

Joins
=======================================================
Imagen tomada de [aquí](http://giannopoulos.net/wp-content/uploads/2013/05/B)

<div class="midcenter" style="margin-left:-300px; margin-top:-240px;">
  <img src="images/sql_joins.jpg"></img>
</div>


Joins
=======================================================

- `CROSS JOIN`
  - `select * from tabla1 CROSS JOIN tabla2;`
  - Producto cartesiano
  - Si lo están usando, quizá sea por error:
      - `select * from tabla1, tabla2;`


Joins
=======================================================

- `INNER JOIN`

```
select ... from tabla1 INNER JOIN tabla2 ON condicion;

select ... from tabla1 INNER JOIN tabla2 USING (lista columnas);

select ... from tabla1 NATURAL INNER JOIN tabla2;
```

Joins
=======================================================

- `OUTER JOIN`
    - Agrega renglones sin _match_ llenándolas con valores `NULL`.

```
select ... from tabla1
  LEFT/RIGHT/FULL  OUTER JOIN tabla2 ON condicion;

select ... from tabla1
  LEFT/RIGHT/FULL  OUTER JOIN tabla2
    USING (lista columnas);

select ... from tabla1 NATURAL
  LEFT/RIGHT/FULL  OUTER  JOIN tabla2;
```

Operaciones de conjuntos
======================================================

- Va enmedio de dos `selects`

  - `union`
  - `union all`
  - `intersect`
  - `except`


Subqueries
=======================================================

- Sin correlacionar
  - Calculan un resultado constante para el `query` superior
  - Se ejecutan una sola vez

- Correlacionado
  - Referencia variables del `query` superior.
  - Se repite por cada renglón del `query` superior.
  - Se puede reescribir como un `join`.


Agregaciones
======================================================

- Reciben `n` valores regresan `1`.

- **Básicos:** `avg()`, `count(*)`,  `count(expression)`, `max()`, `min()`, `sum()`

- **Estadísticos:** `corr()`, `stdenv()`, `variance()`.
  - También en versión población `_pop` y muestra `_samp`.
  - Los de arriba son alias para `_samp`

- **Lógicos:** `bool_and()`, `bool_or()`.

- `array_agg()`, `json_agg()`, `xmlagg()` <- `:(`






========================================================
type: exclaim

## Algunos trucos...

Generales
======================================================

* *Randomize*

`select * from foo order by random();`

(ver más adelante)


* `::` para *castear*: `select '2013-02-14'::date;`

* La división es entera por omisión: `select 1/10 as wrong, 1::float8/10 as correct;`


Generar queries
=======================================================

- Unir varias tablas con **bulk inserts**

```
select 'insert into myschema.mynewtable select * from ' || table_schema || '.' || table_name || ';'
from information_schema.tables
where table_schema='myschema' and table_name like 'old%';
```

- Transformar una tabla de *wide* a *long*

```
select
'insert into output_table (pais, entidad, variable, valor) select pais, entidad, entidad, modelo, "' || column_name::varchar || '", ' || column_name::varchar || ';' as query
from information_schema.columns
where table_name = 'table_name' and table_schema='table_schema';
```

Generar queries
=======================================================

- Generar *queries* para `MongoDB`

```
select
    'db.foo.findOne({_id: ObjectId("' || _id || '")})'
from foo;
```
- Generar una tabla con muchas columnas de un tipo
  - por ejemplo vas a importar un archivo MSExcel para un **LET**.

```
SELECT 'CREATE TABLE data_import('
|| string_agg('field' || i::text || ' varchar(255)', ',') || ');'
FROM generate_series(1,100) As i;
```

Fechas
=======================================================

```
select
date_trunc('year', '2014-02-25'::date) as year,
date_trunc('month', '2014-02-25'::date) as month,
date_trunc('day', '2014-02-25'::date) as day;
```

```
select
to_char('2013-02-25'::date, 'YYYY') as year,
to_char('2013-02-25'::date, 'MM-YYYY') as month,
to_char('2013-02-25'::date, 'YYYYMMDD') as day;
```

```
select
date_part('day', '2013-02-25'::date);
```

Generar secuencias
======================================================

* Ejemplo básico

```
select * from generate_series(0,100,5);
```

* Usándola con funciones

```
select avg(val)
from generate_series(0,100,5) as val;
```

* Una serie de fechas

```
select
current_date + step.i as date_series
from
generate_series(0,14,2) as step(i);
```


"Pivotear" tablas
========================================================

- Como las tablas dinámicas de Excel.

- Usaremos `tablefunc`, en particular la función `crosstab(source_sql, category_sql)`

- Recuerda activar la extensión

```
create extension tablefunc;
create extension "uuid-ossp"; -- También la usaremos
```

"Pivotear" tablas
========================================================

- Generemos una tabla transaccional

```
select
generate_series as fecha,
cus.tarjeta as tarjeta,
(ARRAY['ATM', 'COMERCIO', 'INTERNET'])[trunc(random()*3)+1] as tipo_comercio,
(random() * 10000 + 1)::int AS monto
into transacciones
from generate_series((now() - '100 days'::interval)::date, now()::date, '1 day'::interval),
(select uuid_generate_v4() as tarjeta from generate_series(1,15)) cus;
```

"Pivotear" tablas
========================================================

```
select *
from crosstab(
'select                      -- source sql
date_part(''year'', fecha),
date_part(''month'', fecha),
tipo_comercio,
monto from transacciones
order by 1',
'select                      -- category sql
distinct tipo_comercio
from transacciones'
)
as                           -- tabla de salida
( year int,
  month int,
  ATM int,
  COMERCIO int,
  INTERNET int
);
```

Ejercicio
=======================================================

- Modifica el generador de datos para incluir la columna `colonia` la cual pueda tener 10 valores.
- Modifica el generador de datos para incluir horas en las fechas.
- Modifica el generador para que **no** transaccionen las tarjetas todos los días.


Estadísticas muy loca
========================================================
- Como **feature engineering**

- Promedio y desviación estandar...

```
SELECT
    tarjeta,
    tipo_comercio,
    avg(monto) AS avg,
    stddev(monto) AS stddev
FROM

GROUP BY
    tarjeta,
    tipo_comercio
```

Feature engineering
========================================================

- Top 5 colonias por gasto

```
SELECT
    ...
    (array_agg( colonia ORDER BY monto DESC))[1:5] AS top5_colonias
FROM
    ...
```

- ¿Si los queremos ordenados?

```
(SELECT
    (array_agg(colonia ORDER BY cnt DESC))[1:5]
FROM
    (SELECT colonia, count(*) FROM transacciones AS t2
     WHERE t2.tarjeta = transacciones.tarjeta AND t2.tipo_comercio = transacciones.tipo_comercio
     GROUP BY colonia) AS s (colonia, cnt)
) AS top5_colonias
```


Feature engineering
========================================================

- ¿Y un histograma de horas en los lugares?


```
(SELECT
    array_agg(cnt ORDER BY hour DESC)
FROM
    (SELECT extract(hour FROM fecha), count(*) FROM transacciones AS t2
     WHERE t2.tarjeta = transacciones.tarjeta AND t2.tipo_comercio = transacciones.tipo_comercio
     GROUP BY 1) AS s (hour, cnt)
) AS hourly_histogram
```


Feature engineering
========================================================


**Ejercicio:** El último query no trae los datos correctos, es necesario usar un `generate_series`
¿Se les ocurre como?

**Ejercicio:** Más adelante veremos que `PostgreSQL` es muy bueno haciendo mucho en un sólo `query` ¿Cómo unirían estos en uno solo?

**Ejercicio** ¿Se te ocurre como definir distancias entre estos registros?


Tamaños...
============================================================

- Tamaño de la base de datos

```
SELECT pg_database_size(current_database());
```

- En formato bonito

```
select pg_size_pretty(pg_database_size(current_database()));
```

- Tamaño de una tabla

```
select pg_relation_size('libros');
```

Eliminar duplicados
===========================================================

```
SELECT * FROM libros WHERE ctid NOT IN
(SELECT max(ctid) FROM libros GROUP BY libro) -- ver cuales están duplicadas
```

```
DELETE FROM libros WHERE ctid NOT IN
(SELECT max(ctid) FROM libros GROUP BY libro);  -- Sólo un campo
```

```
DELETE FROM libros WHERE ctid NOT IN
(SELECT max(ctid) FROM libros GROUP BY libros.*) ;  -- Línea completa
```

- **ctid** es una columna oculta en todas las tablas de `Postgresql`, y es único para cada renglón.


Un poco de Text Mining
========================================================

- Crea una base de datos llamada `movies`.

- Observa el *script* `movies_data.sql`:
    - Siempre parte de cero. Esto es una buena práctica.
    - Partir de cero: _dropear_, crear, etc.


- Ejecuta el script `movies_data.sql`.

**NOTA**: Obtenido del libro *Seven databases in seven weeks*.


Un poco de Text Mining
========================================================

El _script_ hace lo siguiente:

- Crea tres tablas:
  - `genres` (`name` (text UNIQUE), `position` (integer)),
  - `movies` (`movie_id` (serial primary key), `title` (text), `genre` (cube))
  - `actors` (`actor_id` (serial primary key), `name` (text))


Un poco de Text Mining
========================================================

- Y una tabla **`habtm`**:

```
  create table movies_actors (
    movie_id integer references movies not null,
    actor_id integer references actors not null,
    unique (movie_id, actor_id)
  );
```

Un poco de Text Mining
========================================================

- Índices

```
create index movies_actors_movie_id on movies_actors (movie_id);
create index movies_actors_actor_id on movies_actors (actor_id);
create index movies_genres_cube on movies using gist (genre);
```

- **Ejercicio**: Modifica el _script_ agregando los índices y ejecuta de nuevo.

Un poco de Text Mining
========================================================

- ¿Qué hace lo siguiente?

```
select title from movies where title ilike 'stardust%';
```

```
-- Regex
-- ~ operador de matching
-- ! No matching
-- * case insensitive
select count(*) from movies where title !~* '^the.*';
```
- Índice para `regex`

```
create index movies_title_pattern on movies (lower(title) text_pattern_ops);
-- Alternativas: varchar_pattern_ops, bpchar_pattern_ops, tern_ops, name_pattern_ops
```

Un poco de Text Mining
========================================================

- Levenshtein

```
select movie_id title from movies
where levenshtein(lower(title), lower('a hard day nght')) <= 3;
```



-Trigramas

```
select show_trgm('Avatar');

-- Buscar parecidos simplemente se reduce a contar el número de trigramas que coinciden.

create index movie_title_trigram on movies
using gist (title gist_trgm_ops);

select title
from movies
where title % 'Avatre';

-- Excelentes para user input...
```

Un poco de Text Mining
========================================================

-- Metafonemas

```
select title
from movies NATURAL join movies_actors NATURAL join actors
where metaphone(name, 6) = metaphone('Broos Wils', 6);

-- ¡Me sorprende cada vez que corre!
-- Natural join es un inner join que hace el on en automático.

```

Un poco de Text Mining
========================================================

- El campo `genre` es un vector multidimensional...

```
select name,
cube_ur_coord('(0,7,0,0,0,0,0,0,0,7,0,0,0,0,10,0,0,0)', position) as score
FROM genres g
WHERE cube_ur_coord('(0,7,0,0,0,0,0,0,0,7,0,0,0,0,10,0,0,0)', position) > 0;
```

- Cercanas

```
SELECT *,
cube_distance(genre, '(0,7,0,0,0,0,0,0,0,7,0,0,0,0,10,0,0,0)') dist
FROM movies
ORDER BY dist;
```

Un poco de Text Mining
========================================================

- Dentro del cubo

```
SELECT title, cube_distance(genre, '(0,7,0,0,0,0,0,0,0,7,0,0,0,0,10,0,0,0)') dist
FROM movies
WHERE cube_enlarge('(0,7,0,0,0,0,0,0,0,7,0,0,0,0,10,0,0,0)'::cube, 5, 18) @> genre
ORDER BY dist;

-- El operador @> es "contiene"
-- cube_enlarge(cubo, radio, dimensiones): select cube_enlarge('(1,1)', 1, 2)
```


Ejercicio
========================================================

**Ejercicio**: Trae todas las películas cercanas a *Apocalypse Now* para recomendárselas a un amigo...

**Ejercicio**: ¿Cómo modificas el código, si tu amigo escribe mal el nombre de la película?

**Ejercicio**: Usando el truco del `cube` y las estadísticas por tarjeta, ve que tarjetas se parecen.

Agregaciones
====================================================

- Películas por actor

```
select
actors.name,
array_to_string(array_agg(movies.title), ',') as movies
from
actors
natural join movies_actors
natural join movies
group by
actors.name
```

Agregaciones: JSON
====================================================

- Películas por actor (en JSON)

```
select row_to_json(tabla)
from (
     select actor_id, name,
     (
        select array_to_json(array_agg(row_to_json(jd)))
        from (
             select movie_id, title, genre
             from movies
             natural join movies_actors ma
             where  ma.actor_id = j.actor_id;
        ) jd
     ) as peliculas
     from actors j
) as tabla;
```

========================================================
type: exclaim
## PostgreSQL
## Untrusted Languages



Ejecutar Python dentro de PostgreSQL
=======================================================

- Instalar la librería

```
sudo apt-get install postgresql-plpython-9.4
```

- En la base de datos

```
create language plpythonu;
```

Ejecutar Python dentro de PostgreSQL
=======================================================


- Crear una función

```
create function hola(name text)
  returns text as $$
    return 'hola %s!' % name
$$ language plpythonu;
```

- Usar la función

```
select hola('Adolfo');
```

Ejecutar Python dentro de PostgreSQL
=======================================================

- ¿Por qué?
  - Es más fácil programar en `python` que en `PL/pgSQL`

  - Se puede escribir todo: `queries`, `caching`, `triggers`, `exceptions`

- ¿Peligros?
  - Por algo se llaman `untrusted languages`.
  - `SQL injection`
  - Corromper la base



Tarea / Ejercicio
=======================================================

- En `python` dentro del `postgresql` obviamente.
- Escribe una función en python que cree una tabla mejorada de `transacciones`.
- Usa 20 tarjetas, con un uso distribuido normalmente, gasto distribuido normalmente, etc, etc.
- Usa la [**documentación**](http://www.postgresql.org/docs/9.4/static/plpython.html)


JSON
========================================================

- Siempre quisieramos que los datos fueran perfectos y se mantuviera la relación y todo eso
- La realidad es que los datos:

    - son imperfectos
    - se guardan de manera imperfecta
    - se tienen que mover entre sistemas
      - como al web...

- Y a veces, pues no quieres ir con `SQL`


JSONB
========================================================

- PostgreSQL soporta dos tipos de `JSON`: estándar (`JSON`) y binario (`JSONB`).
- Es importante recalcar que `JSONB` **NO** es igual a `BSON`.
  - `JSONB` es interno a PostgreSQL, se transmiten en `JSON`, se guardan en `JSONB`.
  - `JSONB` soporta más tipos que `BSON`
      - e.g. `BSON` no soporta enteros o flotantes con más de 64 bits de precisión.


JSONB
========================================================
- PostgreSQL soporta codificación en `javascript` para poder garantizar los resultados.
    - Tiene un `V8` engine dentro!
    - Cuidado con los `XSS` y `SQL injections` !!
        - Más adelante un ejemplo

JSONB
=======================================================

- Nos da más operadores:

```
-- Contención derecha
select '{"a":1, "b": 2}'::jsonb @> '{"a": 1}'::jsonb;

-- Contención izquierda
select '{"a":1}'::jsonb <@ '{"a":1, "b": 2}'::jsonb;

-- Existencia
select '{"a":1, "b": 2}'::jsonb ? 'a';

-- ¿Existe alguno?
select '{"a":1, "b": 2}'::jsonb ?| ARRAY['b', 'd'];

-- ¿Existen todos?
select '{"a":1, "b": 2}'::jsonb ?& ARRAY['a', 'b'];
```

JSON
=======================================================

```
create table json_test (
  data JSONB
);
```

```
insert into json_test (data)
  values
    ('
      {
      "nombre": "Adolfo",
      "apellido": "De Unánue",
      "clase": "big data"
      }
    ');
```

JSON
========================================================

```
-- Observa el operador
select distinct data->>'nombre' as nombres from json_test;

-- Se puede usar en where
select distinct data->>'nombre' as nombres from json_test where data->>'clase' = 'big data';

-- E inclusive con JOINs
-- (Este código no va a correr)
select cargo,
    data ->> 'nombre' as nombre,
    data ->> 'apellido' as apellido
    from json_test
join nomina on
(
  nomina.nombre = json_test.data ->> 'nombre'
);
```

JSON
========================================================
- Operadores

```
-- existencia
select * from json_test where data ? 'nombre';

-- ?| ¿Existe alguno?
-- ?& ¿Todos existen?

-- contención
select * from json_test where
  data @> '{"nombre": "Daniela"}';
select * from json_test where
  data @> '{"nombre": "Adolfo"}';
```

JSON
=========================================================

- Como todo en la vida, hay que crearle índices:

```
create index idx_nombre_gin
  on json_test using gin(data);
```

- Nota que es de tipo `gin` y no `btree`.
    - Los índices `GIN` son para "ver dentro" de los objetos.

- Soporta los operadores `@>, ?, ?&`

JSON
========================================================

- Se puede extraer la información de una tabla relacional como JSON
    - para mandar a un servicio, por ejemplo

```
select
  row_to_json(transacciones)
  from transacciones
  limit 10;
```

JSON: Últimas cosas
========================================================

- `JSON` **NO SIGNIFICA** `schema-less`.
    - Tiene que haber alguna estructura, convención y validación
    - Aquí puede entrar el `V8`

    ```
    create extension plv8;

    create or replace function tiene_llave(doc json)
    return boolean as
    $$
    if ....
    // Código JS
    return true;
    $$ language plv8 immutable;
    ```

JSON: Últimas cosas
========================================================

- `JSON` **NO SIGNIFICA** _escalamiento_.
    - No hay nada en el formato que garantice que correrá en compus normales.
    - No hay nada en el formato que garantice escalabilidad horizontal.
    - No hay nada en el formato que garantice que no hay un punto único de fallo.

  - Me acabo de enterar que existe:  `pl/proxy`


JSON: Últimas cosas
========================================================

- Ver la carpeta [`docs`](./docs) para ejemplos y presentaciones.



========================================================
type: exclaim
## PostgreSQL
## Performance

¿Por qué?
========================================================

- Más usuarios.
- Mejor rendimiento para los usuarios existentes.
- Guardar más información.
- Mejorar la disponibilidad del sistema.
- Dispersión geográfica.

Capas que afectan el performance
========================================================

![capas](images/capas_performance.png)


Diferentes tipos, diferentes problemas
========================================================
* Webapp

* OLTP

* Dataware-house (DW)
    - Importaciones de datos masivas, pero pocas
    - Años de datos
    - Queries duran horas (o días)
    - 5x - 2000x el RAM disponible

========================================================
type: exclaim
# PostgreSQL Performance
## Hardware



Prioridades
=======================================================
Regularmente para un servidor genérico, las prioridades son las siguientes:

1. `CPU`
2. `RAM`
3. `I/O`


Prioridades
=======================================================
Pero para una RDBMS son

3. `I/O`
2. `RAM`
1. `CPU`

Necesidades
========================================================

- Las necesidades son diferentes por que:

  - Hacemos recorridos secuenciales gigantescos
  - Si usamos índices accesamos aleatoriamente al I/O
  - Peticiones de `queries` nunca antes vistos
  - Reportes

- Todo esto no requiere `CPU`.


Hardware
========================================================
## CPU,  RAM, I/O, Network

- **DW**: I/O, CPU, RAM
- **OLTP**: Todos
- **Web**: CPU, Network, RAM, .....,.... I/O


CPU
========================================================
type:alert


# **Un core, un query**

- `=>`  Por lo menos dos cores...

- `PostgreSQL` **aún** no soporta `queries` en paralelo...

    - Pero en `9.5` quizá esté...
    - Actualmente `pg_pool` y `pl/proxy` dan soporte básico.

CPU
========================================================

* ¿Rápidos o más?

  - Muchos `queries` enormes `=>` Cores rápidos.
  - Muchos `queries` pequeños `=>` Más cores.


RAM
========================================================
* WebApp
    - BD en `shared_buffers` -> 6x BD

* OLTP
    - Toda la BD a caché  -> 2x-3x el tamaño de la BD

* DW
    - ¿Cuál es el tamaño de los `sorts` y los `aggregates`?

RAM
========================================================

- ¿La base de datos cabe en RAM?

  - Si sí, ¡compra más cores!

  - Si no, compra mejores discos.

I/O
========================================================

* Son principalmente el cuello de botella.

* ¿Habrá problemas de I/O?
    - ¿Muchos inserts, updates? -> Limitado por el log de transacciones
    - ¿DB es más grande que 3x el RAM? -> Cada query sufrirá

* Solución: Discos


* ¿Cargas masivas? -> Discos


I/O
========================================================

- `RAID`: *Redundant Array of Independent Disks*
  - Técnica de virtualización de almacenamiento de datos.
  - Combinación de varios discos en una sóla unidad lógica de almacenamiento.
  - [Wikipedia](http://en.wikipedia.org/wiki/RAID)

I/O
========================================================

- SO
  - I/O Aleatorio, poco uso
  - `RAID 1`

- WAL
  - Acceso secuencial
  - Uso constante
  - `RAID 1`

I/O
========================================================

- Datos
  - Acceso aleatorio
  - Uso elevado
  - `RAID 10`

- **NO USAR**:
  - `RAID 0` -> No hay redundancia
  - `RAID 5` -> Malo para escribir


Árbol de decisión
========================================================
![io](images/decision.png)




Tips finales
========================================================
* La calidad importa.

* Prueben, prueben y vuelvan a probar.

* Nunca le crean a su vendedor o a su sysadmin.

* Prueben... (en serio)


========================================================
type: exclaim
# PostgreSQL: Performance
## Sistema operativo



Divide et impera
========================================================
* Poner en su propia partición o disco el log de transacciones
    - `pg_xlog`
    - 10-50% más rápido.

    ```
    # /<mountpoint1>/.../pg_xlog donde están los WAL
    # /<mountpoint2>/.../pg_xlog donde quieres que están (en otro disco)

    > /etc/init.d/postgresql stop
    > mkdir -p /<mountpoint2>/.../pg_xlog
    > mv /<mountpoint1>/.../pg_xlog/* /<mountpoint2>/.../pg_xlog/
    > rmdir /<mountpoint1>/.../pg_xlog
    > ln -s /<mountpoint2>/.../pg_xlog/ /<mountpoint1>/.../pg_xlog
    > /etc/init.d/postgresql start
    ```





Divide et impera
========================================================
* `Tablespaces` en sus propios discos
    - Tablas grandes
    - Tablas e índices más usados

    ```
     create tablespace bigdata-space owner su_nombre_de_usuario location 'path_a_/bigdata-disk';
    ```
* ¿Y si ahora quiero cambiar de lugar una tabla ya existente?

    ```
    alter table tablota set tablespace = 'newstorage';
    ```

    - **Cuidado**: La BD se bloquea durante esta transacción.


- Tablas temporales a `SSDs`.


GNU/Linux: Tipos de `filesystem`
========================================================
* OLTP -> `XFS` ó `JFS`

* DW -> `ext4` sin "loggeo"  (a la hora de crear)

    ```
    -data=writeback, noatime, nodiratime
    ```
* Ver
  - [Improving MetaData Performance of the Ext4 Journaling Device](http://www.linux-mag.com/id/7642/).
  - [SSDOptimization](https://wiki.debian.org/SSDOptimization)
  - [Hadoop Operations](http://my.safaribooksonline.com/book/databases/hadoop/9781449327279/4dot-planning-a-hadoop-cluster/id2548524)

GNU/Linux: Tamaño de las páginas
========================================================

* Incrementar (súper importante) `shmmax`,  `shmall`  en el kernel

    ```
    #!/bin/bash

    page_size=`getconf PAGE_SIZE`
    phys_pages=`getconf _PHYS_PAGES`
    shmall=`expr $phys_pages / 2`
    shmmax=`expr $shmall \* $page_size`
    echo kernel.shmmax=$shmmax
    echo kernel.shmall=$shmall
    ```

* Modificarlos permanentemente depende de su versión de kernel y distibución...

* Un `kernel` nuevo (algo menor a `2.6.9` es malísimo...)




Windows OS
========================================================
type:alert

# ¿Están bromeando verdad?





Por último...
========================================================
* ¡Monitoreen todo!
    - CPU
    - I/O
    - Network
    - Espacio en disco
    - Swap
    - RAM

========================================================
type: exclaim
# PostgreSQL: Performance
## postgres.conf



Modificar, modificar
========================================================
* Modificar: `/etc/postgresql/9.x/postgres.conf`
  * Puedes hacerlo con

  ```
  \e /etc/postgresql/9.4/postgres.conf
  ```

  desde `psql`.
Modificar, modificar
========================================================
  * `shared_buffers`
      - ¿Cuánto cache podemos usar para mantener datos?
      - Por lo menos un 1/4 de RAM
      - Si tienes más de 32 Gb `->` 8Gb


Modificar, modificar
========================================================
  * `work_mem`
      - ¿Cuánta memoria para operaciones de ordenamiento y tablas `hash` antes de usar discos?
      - Se aplica por operación.
      - Por lo menos 1Gb

Modificar, modificar
========================================================
- [Mitigar el costo de escritura a disco](http://www.postgresql.org/docs/9.2/static/runtime-config-query.html)
  - `fsync`, `synchronous_commit`
    - Determina si todas las páginas `WAL` deben grabarse a disco antes de que se considere terminada la transacción.
    - Ponerlo en `off` puede causar corrupción de datos si se cae el servidor
    - En un `dwh` puede apagarse, y aumentar el rendimiento...

Modificar, modificar
========================================================
  * `wal_buffers=16Mb`
  * `maintenance_work_mem` por lo menos 1Gb
  * `temp_buffers=16Mb`

Modificar, modificar
========================================================
* [Planeador](http://www.postgresql.org/docs/9.4/static/runtime-config-query.html)

  * `effective_cache_size`
      - 3/4 de la RAM disponible...
      - ¿Cuánta memoria hay disponible para mantener en caché las consultas?

  * `default_statistics_target` 1000
      - Estadísticas por columnas es aún más rápido,
      - Es decir, se puede establecer por columnas.


Modificar, modificar
========================================================
  - `autovacuum=off`,  `vacuum_cost_delay=off`
      - Hacerlo a mano e incluir `analyze`

- `random_page_cost` Determina la forma en que el planeador considera o no usar índices
  - Acceder secuencialmente es más rápido que el acceso con índices
  - Un valor bajo favorece el uso de índices, uno alto las lecturas secuenciales.
Modificar, modificar
========================================================

- Checkpoints

```
checkpoint_completion_target = 0.9
checkpoint_timeout = 10m-30m
checkpoint_segments = 32 # Para iniciar
```


Modificar, modificar
========================================================

- `random_page_cost` 3.0 para un arreglo `RAID10`, 2.0 para un `SAN` 1.1 para `Amazon EBS`.

  * [Docs](http://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server)

- `listen_address`
  - ¡Muy pocas! cada conexión cuesta muchísimo.
  - Mejor filtren a nivel del `firewall`.
  - Usen mucho el `pg_hba.conf`.

Modificar, modificar
========================================================

- `max_connections`
  - De nuevo, las conexiones toman memoria y procesamiento.
  - Mejor usen un `pool` de conexiones.
  - `pgbouncer` o `pgpool` son opciones interesantes.

Por último...
=======================================================

- Sea científicos: midan, prueben, configuren.

- La [**documentación**](http://www.pgconfig.org/) es su amiga.

- Una guía rápida está [aquí](http://www.pgconfig.org/)

========================================================
type: exclaim
# PostgreSQL: Performance
## Application layer



Estadísticas
========================================================
* Al igual que en el código, no optimicen al inicio...

* Todo empieza con las estadísticas: `pg_stat_statements`.

* Registra las estadísticas de todos los SQL ejecutados en el servidor.

```
create extension pg_stat_statements;
```

Estadísticas
========================================================

Hay que mover el archivo `postgresql.conf` de nuevo

```
# postgresql.conf
shared_preload_libraries = 'pg_stat_statements'

pg_stat_statements.max = 10000
pg_stat_statements.track = all
```

- [Ver](http://www.postgresql.org/docs/9.4/static/pgstatstatements.html) la documentación

Estadísticas
========================================================

```
SELECT
  -- Tiempo total respecto al sistema (minutos)
  (total_time / 1000 / 60) as total_minutes,
  --Tiempo promedio en ejecutar en milisegundos
  (total_time/calls) as average_time,
  -- El query
  query
FROM pg_stat_statements
ORDER BY 1 DESC
LIMIT 100;
```

- ¿Qué optimizar?

Cache
=======================================================

- 80/20
- `PostgreSQL` mantendrá los datos que se accesan mucho en `cache`.
- Queremos que el cache esté usándose cerca del 99%.

```
SELECT
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit)  as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM
  pg_statio_user_tables;
```

- Ahorita va a marcar un error (división por cero).

Queries
========================================================
* Los problemas principales de un `query` son:
  - No hay memoria para el _sorting_.
  - No hay estadísticas.
  - Está escrito con las patas.

* Hagan mucho en cada query
    - PostgreSQL es muy bueno con queries grandes
    - (y no tan bueno con muchos queries pequeños).

* Asegurar que cuando usen una llave o índice los tipos coincidan (o no se usará el índice).

* Eviten búsquedas de texto como `LIKE %Hola%`


Queries: EXPLAIN ANALYZE
========================================================

```
explain [analyze]
select * from ...
```

* Es un  árbol invertido -> Hay que buscar en el nivel más profundo para ver donde ocurre el problema.

* Leerlo es una arte...

Queries: EXPLAIN ANALYZE
========================================================

```
postgres=# explain select * from transacciones;
                 QUERY PLAN
--------------------------------------------------
 Seq Scan on transacciones
 (cost=0.00..28.15 rows=1515 width=35)
(1 row)

```

```
postgres=# explain analyze select * from transacciones;
                 QUERY PLAN
--------------------------------------------------
 Seq Scan on transacciones
 (cost=0.00..28.15 rows=1515 width=35)
 (actual time=0.007..0.105 rows=1515 loops=1)
 Planning time: 0.024 ms
 Execution time: 0.158 ms
(3 rows)

```

Queries: EXPLAIN ANALYZE
========================================================


```
postgres=# explain analyze select * from transacciones where monto > 5000;
                 QUERY PLAN
--------------------------------------------------
 Seq Scan on transacciones
 (cost=0.00..31.94 rows=742 width=35)
 (actual time=0.007..0.170 rows=746 loops=1)
   Filter: (monto > 5000)
   Rows Removed by Filter: 769
 Planning time: 0.039 ms
 Execution time: 0.207 ms
(5 rows)
```


Queries: EXPLAIN ANALYZE
========================================================


* Buscar malos conteos, escaneos secuenciales, loops enormes, etc.

- Ve la carpeta `docs` para una mayor profundización al tema.

Indexing
========================================================

- `B-tree`
  - Es el valor por omisión. Regularmente quieres usar este.

- `Gin`
  - _generalized inverted index_
  - Varios valores en una columna
  - `hstore / array / json`

Indexing
========================================================


- `Gist`
  - _generalized search tree_
  - Full text search
  - Shapes
  - GIS



- `SP-Gist`
  - _spatial-partitioned Gist_

- `hash`

- Entre otros

Indexing
========================================================


* Indexing
    - foreign keys
    - `WHERE` comunes
    - Agregados de columnas comunes
    - expresiones
    - `full text`
    - `partial`
* Los índices afectan muchísimo los `updates`, `deletes`
    - No tiene caso hacerlo para tablas pequeñas


Indexing
========================================================

```
select amname from pg_am;

 amname
--------
 btree
 hash
 gist
 gin
 spgist
(5 rows)


```

Indexing
========================================================



* Ya se pueden generar concurrentemente:

```
create index concurrently...
```

* Con filtros

```
create index where var = valor
```

* Rule of thumb: Índices en más de 10,000 filas...

* Recuerda que hay otro costo relacionado: su tamaño en disco.

```
select pg_size_pretty(pg_total_relation_size('idx'));
```

Indexing
========================================================

- Índices compuestos
  - Debe de incluir el `query` todas las columnas

```
create index idx_nombre on tabla (col1, col2, col3);
```

- Es diferente a un índice combinado...

```
create index idx_nombre1 on tabla (col1);
create index idx_nombre2 on tabla (col2);
create index idx_nombre3 on tabla (col3);
```


Indexing
========================================================
*  `pg_stat_user_indexes`
    - Índices no usados (ya que muestra el uso de los índices)

*  `pg_stat_user_tables`
    - Escaneos secuenciales -> candidatos para índices
    - Muchos Updates/Deletes  -> Borrar índices (quizá)

* [Docs](http://www.postgresql.org/docs/9.4/static/monitoring-stats.html)

Indexing
========================================================

- Por ejemplo, para ver una lista de todas las tablas con las más grandes primero y el porcentaje de las veces que se accesan por índice:

```
SELECT
  relname,
  100 * idx_scan / (seq_scan + idx_scan)
    as percent_of_times_index_used,
  n_live_tup
    as rows_in_table
FROM
  pg_stat_user_tables
WHERE
    seq_scan + idx_scan > 0
ORDER BY
  n_live_tup DESC;
```


Divide et impera
========================================================

## (Reloaded)

- **Particionado vertical**
  - Tablas con muchas columnas en varias tablas
  - ¿Estás mezclando atributos que se accesan mucho con aquellos que lo hacen poco?
  - ¿Mezclando atributos estáticos con aquellos que se modifican mucho?

Divide et impera
========================================================

## (Reloaded)

- **Particionado horizontal**
  - Tablas con muchas filas en varias tablas



Partitioning
========================================================
* Excelente para:
    - Datos históricos
    - Tablas muy grandes
    - Performance de `inserts`
    - Menos de 300 particiones

Partitioning
========================================================
* Está basado en `table inheritance` y  `constraint exclusion`:
    - `triggers` o `RULES` se encargan de las inserciones y actualizaciones.
    - Las constricciones definen los rangos para la tabla.

* Consideraciones:
    - Un delete puede matar al servirdor (literal)
    - Cada query debe usar la llave de particionado.
    - Hay que precrearlas



Partitioning
========================================================
```
CREATE TABLE ufo_2001 (
CONSTRAINT partition_date_range
CHECK (report_date >=   '2001-01-01'
AND report_date <= '2001-12-31 23:59:59')
) INHERITS ( ufo );
```


Partitioning
========================================================
```
CREATE FUNCTION ufo_insert ()
RETURNS trigger
LANGUAGE plpgsql AS $f$
BEGIN
CASE WHEN NEW.report_date < '1989-01-01'
THEN INSERT INTO ufo_1988 VALUES (NEW.*);
WHEN ufo_date < '1990-01-01'
THEN INSERT INTO ufo_1989 VALUES (NEW.*);
...
...
ELSE
INSERT INTO ufo_overflow VALUES (NEW.*);
END CASE;
RETURN NULL;
END;$f$;
```



Partitioning
========================================================
```
CREATE TRIGGER ufo_insert BEFORE INSERT ON ufo
FOR EACH ROW EXECUTE PROCEDURE ufo_insert();
```



Tablespaces
========================================================
* Paraleliza el acceso.
    - Tu  tabla más grande a un tablespace.
    - Sus índices a otro.
* Tablas temporales a un tablespace temporal.
* Los joins muévelos a un SSD (si se puede),
* Y al migrar, una tabla a la vez.

========================================================
type: exclaim

# PostgreSQL: Performance
## ETL


ETL
========================================================


- Los datos están guardados en:
  - Diferentes lugares
  - Diferentes sistemas
  - Diferentes formatos

- Los datos debe de ser extraído de diferentes fuentes.

- Los datos deben de ser filtrados.

ETL
========================================================

- Seleccionar
- Filtrado
- Ordenado
- Convertido
- Integrado
- Analizado
- Nuevas variables
- ...

ETL
========================================================
* Extracción, transformación y carga

* Extracción, carga y transformación (ELT)
    - PostgreSQL es muy bueno en ELT.

* Incluye:
    - Limpiar la basura
    - Partir atributos
    - Normalizar dimensiones
    - Vistas materializadas, índices.

* Cargar en masa, en paralelo y olvídense de los `update`s.

ETL
========================================================

- PostgreSQL permite conectarse a diferentes fuentes externas:

  - `INSERT...SELECTS`
  - `dblink`
  - `FDW`
  - ...

- O usar herramientas externas como
  - `bash`
  - `python` con `SQL Alchemy` (por ejemplo)
  - `R` con `dplyr` (por ejemplo)
  - `ODBC`/`JDBC`

INSERTs
========================================================
* _Bulk inserts_

```
UPDATE  mytable
SET     s.s_start = 1
FROM    (
    VALUES
    (value1),
    (value2),
    …
) q
WHERE   …
```


```
INSERT INTO mytable (mykey, mytext, myint)
VALUES
(1, 'text1', 11),
(2, 'text2', 22),
```



INSERTs
========================================================
```
INSERT INTO bar (a, b, c)
SELECT d, e, f FROM foo
WHERE  foo.d not in (SELECT a FROM bar);
```

```
UPDATE bar
SET    b = foo.e, c = foo.f
FROM   (SELECT d, e, f FROM foo) AS foo
WHERE  a = foo.d;
```



INSERTs
========================================================
* Crea y carga las tablas en una sola transacción (`SELECT INTO`, `CREATE TABLE AS`)

* Agrega los índices al final

* Inserciones en paralelo
      -pero no más que los CPUs que dispones




COPY
========================================================
* Dos versiones en  `psql` y en `SQL`:
    - `COPY`  en el servidor
    - `\copy` local

* 3x-5x veces más rápido que los `insert`s.

* Tienes que conocer la estructura con antelación (si no truena a la mitad y `rollback` :(  ).

* Trata de no hacer que la base "pagine".
    - Divide los archivos según el tamaño de la página del SO.
    - Usa `split` y sus amigos.




COPY
========================================================
* En `SQL`

```
COPY airports
FROM
'airports.csv'
with
delimiter ',' nulls as 'NA' header csv;
```

* Usando `psql`

```
\copy airports
FROM
'airports.csv' with
delimiter ',' header csv;
```


COPY
========================================================

* **Versión segura**:

```
\copy
airports(iata, airport, city, state, country, lat, long)
from 'airports.csv' with delimiter ',' header csv
```

COPY: Archivos gigantes
===============================================================

- Si tengo un archivo gigante y quiero aprovechar la memoria de manera eficiente y mis CPUs

```
cat archivote.psv | \
  parallel --pipe --block 500M ./carga_postgres.sh

## Si no lee de stdin y usa archivos...
cat archivote.psv | \
  parallel --pipe --block 500M "cat > {#}; ./carga_postgres.sh {#}"

## Salvando el avance...
cat archivote.psv | \
  parallel --pipe --block 500M \
    --joblog my_log --resume-failed \
      "cat > {#}; ./carga_postgres.sh {#}"
```


COPY: Archivos gigantes
===============================================================

- Es  mejor usar la herramienta [**`GNU sql`**](http://www.gnu.org/software/parallel/sql.html)
  - Viene incluida dentro de `GNU parallel`

```
cat archivote.psv | \
  parallel --pipe --block 500M sql pg://user:pass@host/db
```

Esta versión no usa archivos temporales , todo pasa por los *pipes* (memoria).

FDW: Foreign Data Wrappers
========================================================
* Soporta: Oracle, MySQL, ODBC, NoSQL, Archivos, twitter, ldap, www, etc.
    - [Docs](http://wiki.postgresql.org/wiki/Foreign_data_wrappers)

```
CREATE EXTENSION file_fdw;  -- Habilitamos la extensión
CREATE SERVER file_server FOREIGN DATA WRAPPER file_fdw;  -- Una formalidad
```

```
CREATE FOREIGN TABLE carriers
( Code varchar, Description varchar )
SERVER file_server
OPTIONS \
(format 'csv', delimiter ',', filename 'ufo.csv', header 'true', null '');
```



FDW
========================================================
```
ALTER FOREIGN TABLE carriers OPTIONS ( SET filename '../ufo.csv' );
-- Cambiamos la localización del archivo
```

* ¡Y es una tabla cualquiera!
    - Ver ejemplo [aquí](http://www.postgresonline.com/journal/archives/250-File-FDW-Family-Part-1-file_fdw.html) para tratar fechas.

**Ejercicio**: Creen otra base de datos, y usen el `postgres_fdw` para accesarlo, ejecuten un query con ambas bases de datos. (Los pasos son muy similares).

Tablas temporales
========================================================
* Yo las uso principalmente dentro de funciones (como pasos intermedios)

```
CREATE TEMPORARY TABLE
ON COMMIT DROP  -- Comenten esto o desaparecerá
rita_rollup AS
SELECT tail_num, year, month, sum(distance) as distancia_recorrida,
array_agg(origin || '->' || dest) as itinerario
FROM rita
WHERE year BETWEEN '1988'
AND '2001'
GROUP BY tail_num, year, month;
```



Unlogged tables
========================================================
* Hasta 40% más rápidas (no usan el wal)

```
CREATE UNLOGGED TABLE
cleaned_ufo
AS SELECT ...
FROM dirty_ufo,
WHERE ...
```
* Mucho cuidado, se cae el servidor y desaparece la tabla
    - Ya me pasó :( varios Tb perdidos



Tips
========================================================
* Bulk insert a una nueva tabla en lugar de actualizar la existente.
    - Sin miedo, de verdad

* Actualiza todas la columnas de un sólo golpe, no una por una.

* Usa vistas y funciones para simplificar.

* ¡Insertar en la tabla final limpia es el último paso!
    - ¡No le hagas `updates`!


Tips
=======================================================

- *Loggea*, siempre, todo

- [**Docs**](http://www.postgresql.org/docs/9.4/static/runtime-config-logging.html)

- En **`postgres.conf`**

```
log_destination = 'csvlog'
log_directory = 'pg_log'
logging_collector = on
log_filename = 'postgres-%Y-%m-%d_%H%M%S'
log_rotation_age = 1d
log_rotation_size = 1GB
log_min_duration_statement = 250ms
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on
log_temp_files = 0
log_statements = all
```


Tips
=======================================================

- Sólo ejecuta la base de datos en el host.

- Desactiva el `OOM Killer` (GNU/Linux)
  - Olvidarme de esto fué lo que hizo que perdiera mis 6 Tb :(
  - Depende de su distro

- **Nunca** mezcles una base de datos operativa con el **DW**.

- Si tienes una tabla que tiene partes que se actualizan mucho y otras que se actualizan poco `->` divide la tabla en dos.

Tips
========================================================

- `vacuum` a mano, regularmente (¿Recuerdan que lo desactivamos?)

- Es posible correrlo en paralelo

```
sql -n --list-tables pg://user:pass@host/db  | parallel -j0 -r --colsep '\|' sql pg://user:pass@host/db vacuum full verbose {2}
```

- La misma idea sirve para varios *queries* en paralelo.

Tarea
========================================================
* Configurar su `postgres.conf`
* Cargar los archivos ufo y gdelt de manera sucia
* Crear un particionado para ufo y gdelt por año.
  - Por ejemplo, ciudades, países, aeropuertos, demografía, educación, etc.
  - Al menos 2.
* Cargar  todas las tablas auxiliares como fdw.
* Limpiar ufo y gdelt en la BD.
* Exportarlas a archivo CSV (con `copy`).
* Usen masivamente `parallel` y sus utilerías.

========================================================
type: exclaim

## Intermezzo: DWH


Esquemas
=======================================================

- Esquema estrella
- Esquema constalación
- Esquema copo de nieve

Tarea
========================================================

- Crear un esquema de estrella para `ufo`.
- Desarrollar los `LET`.



========================================================
type: exclaim

## PostgreSQL: Analítica


LATERAL JOIN
=======================================================
- Es el `for each` de `sql`.
- Se incluye desde `9.3` (aunque es `SQL 1999`).
- Permite referirse a columnas externas dentro del sub-select.

LATERAL JOIN
========================================================

- Top-N

```
select t.*
  from tabla  t
  join otra_tabla ot
  on (t.col = ot.col)
  where ot.campo = ...
  order by t.created_at desc
  limit 10
```

```
select t.*
  from otra_tabla ot
  join LATERAL (
    select *
      from tabla t
      where t.col = ot.col
      order by t.created_at desc
      limit 10
  ) top_algo on (true)
where ot.campo = ....
order by t.created_at desc
limit 10
```

Ejercicio
=======================================================

- Genera ambos `queries` que calcule el top 3 de avistamientos por año
en duración.
- Comparalos con el `explain analyze`.




Agregados regulares
========================================================
![capas](images/regular_aggregate.png)




Windowing functions
========================================================
* Provee acceso a un conjunto de renglones, desde el renglón actual.

![capas](images/windowing_function.png)

- Se realiza **sobre** (`over`) un conjunto de renglones.


OVER
=======================================================
- `over` puede seguir de cualquier función de agregación.
- `over` define que renglones son visibles en cada renglón.
    - `over()` hace visibles todos los renglones para cada renglón.

- `over(partition by algo)` segrega de manera parecida a un `group by`.


Componentes de la ventana
========================================================
* Una partición
    - Especificado por `PARTITION`.
    - Nunca cambia
    - Puede contener un `frame`.


Componentes de la ventana
========================================================
* Un marco (frame)
    - Especificado por `ORDER BY`
    - Se mueve en la partición.
    - Pero sólo dentro de una partición.


Componentes de la ventana
========================================================
* Función
    - Algunas toman valores en la partición, otras en el frame.




Partition
========================================================
![partition](images/partition.png)



Frame
========================================================
![frame1](images/frame.png)

Frame
=======================================================
![frame2](images/window_frame.png)

Window
========================================================
![window](images/window.png)


Todo junto
========================================================
```
  select ...
  window_function()
  over (
  partition by
  order by ...
  )
  from ...
  where ...
```




Windowing functions: Generalidades
========================================================
* Regresa un valor por cada renglón.

* El valor de retorno es calculado en los renglones de la ventana.

*  `OVER()` convierte funciones normales en `window function`. Esto significa que cuando PostgreSQL ve una `window function` revisa todos los registros que satisfacen el `WHERE`, hace la agregación y devuelve la salida como parte del registro actual.

* `PARTITION BY` le indica a PostgreSQL que subdivida la ventana. La agregación se hace en la subventana.


Windowing functions: Generalidades
========================================================
* `ORDER BY` ordena las filas, las `window functions` trabajan entre el renglón actual y el ultimo de la ventana.

* Se pueden combinar `ORDER BY`  y  `PARTITION BY`. Reinicia el ordenamiento por cada ventana.

* A diferencia de ORACLE, PostgreSQL permite usar como `window function` los agregados que ustedes creen.

* Pueden tener varias particiones, no se preocupen...



Lista de funciones
========================================================
* `row_number()` -> Regresa el número de la fila actual.
* `rank()` -> Rango de la fila actual con gaps.
* `dense_rank()` -> Lo mismo pero sin gap.
* `percent_rank()` -> Rango relativo.
* `cume_dist()` -> Rango relativo.
* `ntile()` ->

Lista de funciones
========================================================
* `lag()` -> Regresa el valor de la fila anterior (partición).
* `lead()` -> Regresa el valor de la fila siguiente (partición).
* `first_value()` -> Regresa el primer valor del marco.
* `last_value()` -> Regresa el último valor del marco.
* `nth_value()` -> Regresa el n-ésimo valor del marco.


Otras funciones
=======================================================

- Son un tipo de funciones de agregación
  - muchos renglones `->` un renglón.
- La diferencia estriba en que el orden de los renglones es importante.

- Se agregaron varias funciones de este tipo y la cláusula `WITHIN GROUP` para especificar el ordenamiento.


- `mode()` es un buen ejemplo


Otras funciones
=======================================================

- Un subtipo especial de estas funciones son las **hypothetical set functions**.
    - `percent_rank`, `dense_rank`.

- Otro subconjunto: **inverse distribution functions**:
    - `percentile_cont`,  `percentile_disc`

    ```
    select percentile_disc(0.5) -- mediana
    within group(order by algo)
    from tabla
    ```

Otras funciones
=======================================================

```
select
  aggfnoid, aggkind
  from pg_aggregate
  where aggkind in ('o', 'h')
  order by aggkind;
```

- `n` -> normal aggregates
- `o` -> ordered-set aggregates
- `h` -> hypothetical-set aggregates

Agregados y acumuladores
========================================================
### Agregados
* Regresa el mismo valor  para todo el marco.
    - `sum`, `avg`, `max`, etc,
    - Se pueden checar desde `psql`, con   `\da[S+]`

```
select col, sum(col) over() from tabla;
```



Agregados y acumuladores
========================================================
### Acumulados

* Regresa diferentes valores a lo largo del marco.

```
select carrier, tail_num,
air_time,
sum(air_time) over (
  partition by carrier, tail_num
  order by flight_date
  rows between unbounded preceding
  and current row
  ) as acumulado,
sum(air_time) over (
  partition by carrier, tail_num
  ) as total
from rita
order by carrier
```

Agregar una función de agregación
========================================================

```
CREATE OR REPLACE FUNCTION _final_median(anyarray)
  RETURNS float8 as
  $$
    WITH q AS
      (
        SELECT val
        FROM unnest($1) val
        WHERE VAL is not null
        ORDER BY 1
      ),
    cnt AS
    (
      SELECT COUNT(*) AS c FROM q
    )
    SELECT AVG(val)::float8
      FROM
      (
        SELECT val FROM q
        LIMIT 2 - MOD((SELECT c FROM cnt), 2)
        OFFSET GREATEST(CEIL((SELECT c FROM cnt) / 2.0) - 1, 0)
      ) q2;
  $$
LANGUAGE sql IMMUTABLE;
)
```

Agregar una función de agregación
========================================================

```
CREATE AGGREGATE median(anyelement)
(
SFUNC = array_append,
STYPE = anyarray,
FINALFUNC = _final_median,
INITCOND= '{}'
);
```

```
select median(score), avg(score)
from game_results;
```

CTEs
========================================================
* `CTE` = **Common Table Expression**

* Es como un método privado.

* Permite encadenado en lugar de anidado.

* Es un `query` se puede usar en un `query` más grande.

* Hay tres tipos:
    - Estándar, no recursivo, no-escribible CTE.
    - Escribible CTE, puede incluir `INSERT`s, `UPDATE`s.
    - Recursivos.




CTE estándar
========================================================
```
with  ciudad_forma as (
  select
  city as ciudad, state as estado, shape as forma, count(*) as conteo
  where shape  is not null
from ufo
group by city, state, shape
)


select
estado, max(conteo)
over(partition by ciudad)
from ciudad_forma;
```

- De esta manera es como una _vista materializada_
    - ver más adelante...

**Ejercicio** Reescríbelo como un subquery y compara con `explain analyze`.


CTEs escribibles
========================================================
* Se pueden usar para reparticionar las tablas, por ejemplo extraer un carrier

```
-- ¡NO EJECUTAR!
with
deleted_countries as (
  delete from only ufo
  where country is not  'US'
  returning *
  ),

insert into foreign_ufos
select * from deleted_countries;
```



CTEs recursivos
========================================================

- Es el `while` de `SQL`.

* Secuencias:

```
with recursive seq(n)
as (
  -- término no recursivo
  select 1 as n -- 1. se ejecuta primero
  union all

  -- término recursivo
  select n+1
  from seq
  where n < 100
)

select n from seq
```

CTEs recursivos
========================================================

- Es el `while` de `SQL`.

* Secuencias:

```
with recursive seq(n)  -- 2. Se envía el resultado aquí
as (
  -- término no recursivo
  select 1 as n
  union all

  -- término recursivo
  select n+1
  from seq
  where n < 100
)

select n from seq
```

CTEs recursivos
========================================================

- Es el `while` de `SQL`.

* Secuencias:

```
with recursive seq(n)
as (
  -- término no recursivo
  select 1 as n
  union all

  -- término recursivo
  select n+1
  from seq -- 3. Se ve aquí
  where n < 100
)

select n from seq -- 3. Se ve aquí
```

CTEs recursivos
========================================================

- Es el `while` de `SQL`.

* Secuencias:

```
with recursive seq(n)
as (
  -- término no recursivo
  select 1 as n
  union all

  -- término recursivo
  select n+1 -- 4. Se ejecuta el union
  from seq
  where n < 100
)

select n from seq
```

CTEs recursivos
========================================================

- Es el `while` de `SQL`.

* Secuencias:

```
with recursive seq(n)   -- 5. se vuelve a mandar

as (
  -- término no recursivo
  select 1 as n
  union all

  -- término recursivo
  select n+1
  from seq
  where n < 100
)

select n from seq
```

**Es como un bucle**

CTEs recursivos
========================================================

- Este ejemplo recién mostrado sirve como sustituto del `generate_series`
    - Ya que sólo funciona en `PostgreSQL`.


CTEs recursivos
========================================================
## **Árboles**

- Simple

```
with recursive t as (
select *
from arbol
where parent_id is null  /* No recursivo */
union all
select e1.*   /*Recursivo*/
from arbol as e1
join t as e2
on e1.parent_id = e2.id
)

select *
from t
order by id
```


CTEs recursivos
========================================================
## **Árboles**

```
with recursive t(node, path) as (
-- Inicialización
select id, array[id] from arbol where parent_id is null

union all
    select e1.id, e2.path || array[e1.id]    /* Recursión */
    from arbol as e1 join t as e2 on (e1.parent_id = e2.node)
    where array[id] not in (e2.path)  /* Condición de fin */
)

select
  case when array_upper(path,1) > 1 then '+-' else '' end ||
  repeat('--', array_upper(path, 1)-2) ||
  node as "Branch" /* Despliegue */
from t
order by path;
```


CTEs recursivos
========================================================
## **Árboles**

- Con `path`

```
with recursive t as (
  select *,
  1 as lvl,
  cast(id as text) as tree,
  id || '.' as path
  from arbol
  where parent_id is null

  union all

  select e1.*, e2.lvl + 1 as lvl,
  lpad(e1.id::text, lvl*2, ' ') as tree,
  e2.path || e1.id || '.' as path
  from arbol as e1
  join t as e2
  on e1.parent_id = e2.id
)

select * from t
order by path;
```

CTEs recursivos
========================================================
## **Transitive Closure**

```
-- Suponiendo que existe una tabla nodes y edges...
with recursive transitive_closure (a, b, distance, path_string) as
(
select a, b, 1 as distance,
'.' || a || '.' || b || '.' as path_string
from edges
union all
select tc.a, e.b, tc.distance+1, tc.path_string || e.b || '.' as path_string
from edges as e
join transitive_closure as tc
on e.a = tc.b
where tc.path_string not like '%.' || e.b || '.%'
)
select * from transitive_closure
order by a,b
;
```

FILTER
=========================================================

- sustituto del `CASE WHEN... THEN ... ELSE... END`

```
select year,
sum(duration) filter (where month = 1) as enero,
sum(duration) filter (where month = 1) as febrero,
...
from ufo
group by year;
```

La alternativa es

```
...sum(case when month = 1 then duration else 0 end ) as enero...
```

Vistas materializadas
========================================================
- Existen a partir de 9.3
- El `query` es ejecutado y usado para popular la vista materializada.
- Puede ser actualizada (a diferencia de una tabla normal)

```
create materialized view vista_mat_nombre
as query;
```

Vistas materializadas
========================================================

- Para refrescar la vista:

```
refresh materialized view vista_mat_nombre;
```

- Esto causa un `lock` en la tabla.
  - `> 9.4`:

    ```
    refresh materialized view concurrently ...
    ```

- También se puede alterar, "dropear", etc.


Muestreo
========================================================
* Esta función asume que los id de transacciones son densos y ordenamos en el tiempo
* Obtenido de [Best way to select random rows PostgreSQL](http://stackoverflow.com/questions/8674718/best-way-to-select-random-rows-postgresql)


```
-- Obtener estimados:
-- SELECT count(*) AS check_count,min(id)  AS min_id,max(id)
-- AS max_id,max(id) - min(id) AS min_id_span FROM bigtbl
WITH params AS (
    SELECT 1       AS min_id        -- minimum id <= currently min id
          ,5100000 AS id_span       -- beyond max id (max_id - min_id + buffer)
    )

    --continua
```



Muestreo
========================================================
```
-- continua

SELECT *
FROM  (
    SELECT p.min_id + floor(random() * p.id_span)::integer AS id
    FROM   params p
          ,generate_series(1, 1100) g
    GROUP  BY 1                     -- trim duplicates
    ) r
JOIN   bigtbl USING (id)
LIMIT  1000;
```

Tarea
=======================================================

* Rescribir con una CTE
  - Ejercicio de **Estadística muy loca**
  - Muestreo


========================================================
type: exclaim

## PostgreSQL
# **`R`**

PostgreSQL y R
========================================================

## **`RPostgreSQL`**

- La forma más directa es `RPostgreSQL`.

- Hereda del paquete `DBI`.

- No soporta conexiones con `SSL`.

- Modo de empleo: Ejecutas *queries*, obtienes `data.frames`, los manipulas en `R`, escribes las tablas de resultado.

- Hay que tener cuidado en los tipos de retorno.

PostgreSQL y R
========================================================

## **`sqldf`**

- Por default tratará de usar la base de datos `test`, con el usuario `postgres` en `localhost`.


```r
options(sqldf.RPostgreSQL.user = "postgres",
  sqldf.RPostgreSQL.password = "postgres",
  sqldf.RPostgreSQL.dbname = "test",
  sqldf.RPostgreSQL.host = "localhost",
  sqldf.RPostgreSQL.port = 5432)

ds <- # Crea un data.frame

library(RPostgreSQL)
library(sqldf) # Automáticamente usará PostgreSQL

sqldf("select count(*) from ds")

# El driver de PostgreSQL permite funciones analíticas
sqldf('select *, rank() over (partition by ... order by ...) from ds order by ...')
```


PostgreSQL y R
========================================================
## **`dplyr`**


```r
tabla_postgres <- tbl(src_postgres(dbname=..., host=..., port=..., user=..., password=...))

# PostgreSQL soporta todas las funciones analíticas
# row_number(), min_rank(), dense_rank(), cume_dist(), percent_rank(). ntile()
# lead(), lag()
# cumsum(), cummin(), cummax(), cumall(), cumany(), cummean()
op1 <- group_by(tabla_postgres, var1, var2)

op2 <- filter(op1, var4 == min(var4), var4 == max(var4))

op3 <- mutate(op1, rank = rank(desc(var4)))

# etc, etc
```

========================================================
type: exclaim

## PostgreSQL
# **Proyecto 2**

Proyecto 2
========================================================
- Para UFO y GDELT
    - Utiliza las plantillas de la clase de Minería de datos para describirlas.

Proyecto 2
========================================================
* En la base de datos: UFO
    - ¿Primer avistamiento en cada estado? ¿Primer avistamiento de cada forma?
    - ¿Promedio de entre avistamientos, por mes, por año? ¿Por estado?
    - ¿Cuál estado tiene mayor varianza?
    - ¿Existen olas temporales? ¿Existen olas espacio-temporales?
    - ¿Narrativas parecidas?
    - ¿Cómo está relacionado con la geografía?
    - ¿Con características sociales?
    - Desarrolla un modelo predictivo.

Proyecto 2
========================================================
* En la base de dato: GDELT
    - Responde preguntas parecidas a las anteriores.
    - Señala el caso de México.
    - ¿Existen clústers?
    - ¿Qué países son parecidos a México?
    - ¿Puedes desarrollar un modelo predictivo?
    - ¿Qué más bases de datos puedes traer al análisis?



========================================================
type:exclaim

# Bibliografía



Bibliografía
========================================================
* You Can Do That Without Perl?, PGCon2009, 21-22 May 2009 Ottawa
* Really Big Elephants Data Warehousing with PostgreSQL, MySQL User Conference 2011
* Data Warehousing 101 Everything you never wanted to know about big databases but were forced to find out anyway, Open Source Bridge 2011
* PostgreSQL: Up and Running, Regina Obe and Leo Hsu, O'Reilly, 2012.
* [Trees and more with postgreSQL](http://www.slideshare.net/PerconaPerformance/trees-and-more-with-postgre-s-q-l)
* [Graphs in database: RDBMs in the social network age](http://www.slideshare.net/quipo/rdbms-in-the-social-networks-age)





========================================================
type: exclaim

# **Addendum**


Grafos
========================================================


- En una `RDBMS` los grafos se guardan como *listas de adyacencia*

- **Nodos**

```
create table nodes (
id integer primary key,
name varchar,
feature_1 varchar,
feature_2 varchar,
...
)
```

Grafos
========================================================

- **Edges**

```
create table edges (
a integer not null references nodes(id)
on update cascade
on delete cascade,
b integer not null references nodes(id)
on update cascade
on delete cascade,
primary key(a,b)
);
```

- **Índices**

```
create index a_idx on edges(a);
create index b_idx on edges(b);
```

- Otros

```
-- - Si el grafo no es direccionado
create unique index pair_unique_idx
on edges (least(a,b), greatest(a,b));

-- Sin auto bucles
alter table edges add constraint
no_auto_bucles_chk check (a <> b);

```

Tarea
========================================================

## **Ejercicio**

- Modificar el código:
    - Nodos: ciudades; atributos del nodo: ¿aerolíneas? / ¿principal aerolínea?
    - Edges: conexiones

- Crear estas tablas


Árboles
========================================================

```

create table arbol (
  id integer not null,
  parent_id integer references tree(id),
--  feature_1 varchar,
--  feature_2 integer,
--  ...
  unique(id, parent_id)
);

-- Insertamos

insert into arbol(id, parent_id)
values(1, NULL), /* Root */
(2,1), (3,1), (4,1),
(5,2), (6,2), (7,2), (8,3),
(9,3), (10,4), (11.5), (12,5), (13, 6),
(14,7), (15, 8);
```
