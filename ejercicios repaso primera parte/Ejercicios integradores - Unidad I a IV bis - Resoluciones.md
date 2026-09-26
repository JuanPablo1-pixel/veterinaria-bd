# Ejercicios integradores — Unidad I a IV bis: resoluciones

Diseño y Administración de Base de Datos — UTN

Resoluciones de `Ejercicios integradores - Unidad I a IV bis.md`.

---

## Ejercicio 1

| Pregunta | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|
| Respuesta | b | c | a | d | b | a | c | d | a | c |

Algunas aclaraciones:

- **2.** El gestor de archivos y el gestor de memoria son parte del gestor de almacenamiento, no del procesador de consultas.
- **3.** La opción b es falsa: el modelo de red mantiene la desventaja del jerárquico de tener que conocer toda la estructura para navegar los datos. Las relaciones siguen siendo físicas (opción d).
- **8.** En el material de la cátedra, `select` está dentro de DML ("consultar datos"), junto con `insert`, `update` y `delete`. `create`, `alter` y `drop` son DDL.

## Ejercicio 2

Los diagramas están escritos en DBML para verlos en [dbdiagram.io](https://dbdiagram.io/d): abrí el sitio, borrá el ejemplo que aparece en el panel izquierdo y pegá el código de cada caso. El diagrama se dibuja solo a la derecha.

En DBML, `ref: >` marca una FK ("muchos a uno"), `[pk]` la clave primaria e `indexes { (a, b) [pk] }` una PK compuesta.

### 2.a — Academia de idiomas

La nota final no es un dato del alumno (tiene una nota por cada curso) ni del curso (cada alumno tiene la suya): es un atributo de la relación N:M, así que va en la tabla intermedia `inscripcion`. Es el mismo caso que la dosis en la receta de la veterinaria.

```dbml
Table idioma {
  id_idioma serial [pk]
  nombre varchar(40) [not null, unique]
}

Table profesor {
  id_profesor serial [pk]
  nombre varchar(60) [not null]
  apellido varchar(60) [not null]
  email varchar(100) [not null, unique]
  fecha_ingreso date [not null]
}

Table curso {
  id_curso serial [pk]
  id_idioma int [not null, ref: > idioma.id_idioma]
  id_profesor int [not null, ref: > profesor.id_profesor]
  nivel char(2) [not null, note: 'A1, A2, B1, B2, C1 o C2']
  cupo_disponible int [not null, note: 'nunca negativo']
  precio_mensual "numeric(10,2)" [not null]
}

Table alumno {
  id_alumno serial [pk]
  dni varchar(10) [not null, unique]
  nombre varchar(60) [not null]
  apellido varchar(60) [not null]
  email varchar(100) [unique]
  fecha_nac date
}

Table inscripcion {
  id_alumno int [not null, ref: > alumno.id_alumno]
  id_curso int [not null, ref: > curso.id_curso]
  fecha_inscripcion date [not null]
  nota_final "numeric(4,2)" [note: 'NULL mientras cursa; entre 1 y 10']

  indexes {
    (id_alumno, id_curso) [pk]
  }
}
```

Tablas:

- `idioma (id_idioma PK, nombre UNIQUE)`
- `profesor (id_profesor PK, nombre, apellido, email UNIQUE, fecha_ingreso)`
- `curso (id_curso PK, id_idioma FK → idioma, id_profesor FK → profesor, nivel, cupo_disponible, precio_mensual)`
- `alumno (id_alumno PK, dni UNIQUE, nombre, apellido, email UNIQUE, fecha_nac)`
- `inscripcion (id_alumno FK → alumno, id_curso FK → curso, fecha_inscripcion, nota_final)` — PK compuesta `(id_alumno, id_curso)`

### 2.b — Plataforma de música

`cancion` es una **entidad débil**: no existe sin su álbum, y su número de pista sólo la identifica dentro del álbum. Por eso su PK es compuesta, `(id_album, nro_pista)`, igual que el ítem de una factura.

La reproducción resuelve la N:M entre usuario y canción, pero lleva **PK propia** (`id_reproduccion`): como un usuario puede escuchar la misma canción muchas veces, `(id_usuario, id_album, nro_pista)` se repetiría. La FK hacia `cancion` también es compuesta.

```dbml
Table artista {
  id_artista serial [pk]
  nombre varchar(100) [not null]
  pais varchar(50)
}

Table album {
  id_album serial [pk]
  id_artista int [not null, ref: > artista.id_artista]
  titulo varchar(150) [not null]
  anio int
}

Table cancion {
  id_album int [not null, ref: > album.id_album]
  nro_pista int [not null]
  titulo varchar(150) [not null]
  duracion_seg int [not null]

  indexes {
    (id_album, nro_pista) [pk]
  }
}

Table usuario {
  id_usuario serial [pk]
  email varchar(100) [not null, unique]
  nombre varchar(60)
  fecha_alta date [not null]
  plan varchar(10) [not null, note: 'gratis o premium']
}

Table reproduccion {
  id_reproduccion serial [pk]
  id_usuario int [not null, ref: > usuario.id_usuario]
  id_album int [not null]
  nro_pista int [not null]
  fecha_hora timestamp [not null]
}

Ref: reproduccion.(id_album, nro_pista) > cancion.(id_album, nro_pista)
```

Tablas:

- `artista (id_artista PK, nombre, pais)`
- `album (id_album PK, id_artista FK → artista, titulo, anio)`
- `cancion (id_album FK → album, nro_pista, titulo, duracion_seg)` — PK compuesta `(id_album, nro_pista)`
- `usuario (id_usuario PK, email UNIQUE, nombre, fecha_alta, plan)`
- `reproduccion (id_reproduccion PK, id_usuario FK → usuario, (id_album, nro_pista) FK → cancion, fecha_hora)`

### 2.c — Empresa de envíos

"El jefe también es un empleado" es una **relación recursiva**: `empleado` tiene una FK hacia sí misma (`legajo_jefe → empleado.legajo`). Tiene que admitir NULL, porque el gerente general no tiene jefe. Lo mismo pasa con `legajo_repartidor` en `envio`: puede ser NULL mientras no haya repartidor asignado.

El legajo no se autogenera: lo asigna RRHH, así que la PK viene dada (como la matrícula del veterinario).

```dbml
Table sucursal {
  id_sucursal serial [pk]
  nombre varchar(60) [not null]
  ciudad varchar(60) [not null]
}

Table empleado {
  legajo int [pk, note: 'lo asigna RRHH, no se autogenera']
  nombre varchar(60) [not null]
  apellido varchar(60) [not null]
  id_sucursal int [not null, ref: > sucursal.id_sucursal]
  legajo_jefe int [ref: > empleado.legajo, note: 'NULL: el gerente general no tiene jefe']
}

Table cliente {
  id_cliente serial [pk]
  cuit varchar(13) [not null, unique]
  razon_social varchar(100) [not null]
  email varchar(100)
}

Table envio {
  id_envio serial [pk]
  id_cliente int [not null, ref: > cliente.id_cliente]
  id_sucursal_origen int [not null, ref: > sucursal.id_sucursal]
  legajo_repartidor int [ref: > empleado.legajo, note: 'NULL si todavia no tiene repartidor']
  ciudad_destino varchar(60) [not null]
  fecha_despacho date [not null]
  fecha_entrega date [note: 'NULL mientras no se entrego']
  peso_kg "numeric(8,2)" [not null]
  costo "numeric(12,2)" [not null]
}
```

Tablas:

- `sucursal (id_sucursal PK, nombre, ciudad)`
- `empleado (legajo PK, nombre, apellido, id_sucursal FK → sucursal, legajo_jefe FK → empleado, admite NULL)`
- `cliente (id_cliente PK, cuit UNIQUE, razon_social, email)`
- `envio (id_envio PK, id_cliente FK → cliente, id_sucursal_origen FK → sucursal, legajo_repartidor FK → empleado, admite NULL, ciudad_destino, fecha_despacho, fecha_entrega, peso_kg, costo)`

## Ejercicio 3

### 3.a

Las tablas se borran en orden inverso a las dependencias: primero las que tienen FK hacia otras (`inscripcion`, después `curso`) y al final las que no dependen de nadie.

```sql
drop table if exists inscripcion;
drop table if exists curso;
drop table if exists alumno;
drop table if exists profesor;
drop table if exists idioma;

create table idioma (
  id_idioma serial primary key,
  nombre    varchar(40) not null unique
);

create table profesor (
  id_profesor   serial primary key,
  nombre        varchar(60) not null,
  apellido      varchar(60) not null,
  email         varchar(100) not null unique,
  fecha_ingreso date not null
);

create table curso (
  id_curso        serial primary key,
  id_idioma       int not null references idioma (id_idioma),
  id_profesor     int not null references profesor (id_profesor),
  nivel           char(2) not null check (nivel in ('A1', 'A2', 'B1', 'B2', 'C1', 'C2')),
  cupo_disponible int not null check (cupo_disponible >= 0),
  precio_mensual  numeric(10, 2) not null
);

create table alumno (
  id_alumno serial primary key,
  dni       varchar(10) not null unique,
  nombre    varchar(60) not null,
  apellido  varchar(60) not null,
  email     varchar(100) unique,
  fecha_nac date
);

create table inscripcion (
  id_alumno         int not null references alumno (id_alumno),
  id_curso          int not null references curso (id_curso),
  fecha_inscripcion date not null,
  nota_final        numeric(4, 2) check (nota_final between 1 and 10),
  primary key (id_alumno, id_curso)
);
```

La nota final no lleva `not null` porque queda vacía mientras el alumno cursa. Un `check` sobre un valor NULL no falla, así que la restricción sólo se controla cuando la nota está cargada.

Alternativa para el borrado: `drop table if exists inscripcion, curso, alumno, profesor, idioma;` en una sola sentencia, o agregando `cascade` a cada `drop`.

### 3.b

```sql
alter table usuario add column pais varchar(50);

alter table usuario rename column plan to tipo_plan;

alter table album alter column anio set not null;

-- extra
alter table cancion add constraint chk_duracion_positiva check (duracion_seg > 0);
```

Ojo con el 3: si ya hay álbumes con `anio` vacío, el `set not null` falla. Primero hay que completar esos datos con un `update`.

## Ejercicio 4

**4.a**

```sql
select nombre, apellido, email
from alumno
where fecha_nac < '2000-01-01'
order by apellido, nombre;
```

**4.b**

```sql
select id_envio, ciudad_destino, peso_kg, fecha_despacho
from envio
where peso_kg > 20
  and fecha_entrega is null
order by fecha_despacho desc;
```

"Todavía no fue entregado" se pregunta con `is null`, no con `= null`.

## Ejercicio 5

**5.a**

```sql
select a.apellido, a.nombre, idi.nombre as idioma, c.nivel, ins.nota_final
from inscripcion ins
inner join alumno a   on a.id_alumno = ins.id_alumno
inner join curso c    on c.id_curso = ins.id_curso
inner join idioma idi on idi.id_idioma = c.id_idioma
where ins.nota_final >= 7
  and ins.fecha_inscripcion >= '2025-01-01'
  and ins.fecha_inscripcion <  '2026-01-01'
order by a.apellido, a.nombre;
```

Las inscripciones sin nota (NULL) quedan afuera solas: `null >= 7` no es verdadero.

**5.b**

```sql
select e.legajo, e.apellido, e.nombre, j.apellido as apellido_jefe
from empleado e
inner join sucursal s on s.id_sucursal = e.id_sucursal
left join empleado j  on j.legajo = e.legajo_jefe
where s.ciudad = 'Rosario'
order by e.apellido;
```

La tabla `empleado` aparece dos veces con distinto alias: `e` es el empleado y `j` es su jefe. Tiene que ser `left join` para que el gerente general (sin jefe) aparezca con `apellido_jefe` en NULL; con `inner join` desaparecería.

El `where` filtra por una columna de `sucursal`, que está del lado del `inner join`, así que no rompe el `left join`. Si en el `where` se pusiera una condición sobre `j`, las filas sin jefe se perderían.

## Ejercicio 6

**6.a**

```sql
select c.titulo as cancion, al.titulo as album, ar.nombre as artista
from cancion c
inner join album al   on al.id_album = c.id_album
inner join artista ar on ar.id_artista = al.id_artista
where c.titulo ilike '%amor%'
order by ar.nombre, c.titulo;
```

`ilike` no distingue mayúsculas de minúsculas: encuentra "Amor", "AMOR" y "amor". Con `like` habría que escribir cada variante.

**6.b**

```sql
select c.id_curso, idi.nombre as idioma, c.nivel, c.precio_mensual
from curso c
inner join idioma idi on idi.id_idioma = c.id_idioma
where idi.nombre in ('Inglés', 'Portugués', 'Italiano')
  and c.nivel in ('B1', 'B2')
order by idi.nombre, c.nivel;
```

Equivale a encadenar `or`, pero hay que tener cuidado con los paréntesis si se escribe así: `(idi.nombre = 'Inglés' or idi.nombre = 'Portugués' or ...) and (c.nivel = 'B1' or c.nivel = 'B2')`.

## Ejercicio 7

**7.a**

```sql
select c.id_curso, idi.nombre as idioma, c.nivel,
       count(ins.id_alumno) as cantidad_alumnos
from curso c
inner join idioma idi     on idi.id_idioma = c.id_idioma
left join inscripcion ins on ins.id_curso = c.id_curso
group by c.id_curso, idi.nombre, c.nivel
order by cantidad_alumnos desc;
```

Dos detalles para que los cursos sin inscriptos salgan con 0: el `left join` hacia `inscripcion`, y contar `count(ins.id_alumno)` en vez de `count(*)`. `count(*)` cuenta la fila del curso aunque no tenga inscripción y devolvería 1.

**7.b**

```sql
select s.nombre as sucursal,
       round(avg(e.peso_kg), 2) as peso_promedio,
       round(avg(e.costo), 2)   as costo_promedio
from envio e
inner join sucursal s on s.id_sucursal = e.id_sucursal_origen
where extract(year from e.fecha_despacho) = 2025
group by s.id_sucursal, s.nombre
order by s.nombre;
```

## Ejercicio 8

**8.a**

```sql
select s.nombre as sucursal, max(e.peso_kg) as envio_mas_pesado
from envio e
inner join sucursal s on s.id_sucursal = e.id_sucursal_origen
group by s.id_sucursal, s.nombre
having max(e.peso_kg) > 100;
```

**8.b**

```sql
select ar.nombre as artista, min(al.anio) as primer_album
from artista ar
inner join album al on al.id_artista = ar.id_artista
group by ar.id_artista, ar.nombre
having min(al.anio) < 1990;
```

En los dos casos la condición es sobre una función de agregado (`max`, `min`), por eso va en `having` y no en `where`: el `where` se evalúa fila por fila, antes de agrupar.

## Ejercicio 9

**9.a**

```sql
select c.titulo, c.duracion_seg
from cancion c
where c.duracion_seg > (select avg(duracion_seg) from cancion)
order by c.duracion_seg desc;
```

No se puede resolver con un join: hay que comparar cada canción contra un valor calculado sobre toda la tabla. `where c.duracion_seg > avg(duracion_seg)` directo da error, porque `where` no admite funciones de agregado.

**9.b**

```sql
select e.legajo, e.apellido, e.nombre, count(*) as cantidad_envios
from empleado e
inner join envio en on en.legajo_repartidor = e.legajo
group by e.legajo, e.apellido, e.nombre
having count(*) = (
  select max(cnt) from (
    select count(*) as cnt
    from envio
    where legajo_repartidor is not null
    group by legajo_repartidor
  ) sub
);
```

La subconsulta interna cuenta los envíos de cada repartidor y la de afuera se queda con el máximo. Comparar contra ese máximo devuelve a todos los empatados. Con `order by cantidad_envios desc limit 1` saldría uno solo, elegido al azar, aunque haya empate.

## Ejercicio 10

```sql
begin;

insert into inscripcion (id_alumno, id_curso, fecha_inscripcion)
values (5, 3, current_date);

update curso
set cupo_disponible = cupo_disponible - 1
where id_curso = 3;

commit;
```

La nota final no se inserta: queda NULL porque el alumno recién empieza a cursar.

**Si el curso no tenía cupos disponibles:** el `update` intenta dejar `cupo_disponible` en -1 y viola el `check (cupo_disponible >= 0)` del ejercicio 3.a, así que falla. Como está dentro de la transacción, PostgreSQL la marca como abortada y el `commit` se comporta como un `rollback`: la inscripción que ya se había insertado también se descarta. Es la **atomicidad**: o se hacen las dos operaciones o no se hace ninguna. Sin la transacción, el alumno habría quedado inscripto en un curso sin cupo.
