# Ejercicio de subconsultas extras — Veterinaria (con resoluciones)

Diseño y Administración de Base de Datos — UTN

Todos los ejercicios usan la base de la veterinaria (`sql/01_esquema.sql` y `sql/02_datos.sql`). Ninguno sale con un `join` y un `group by` solos: en todos hace falta una subconsulta.

Una subconsulta puede ir en tres lugares, y en cada uno cumple un papel distinto:

- **En el `select`**: calcula un valor para cada fila del resultado. Tiene que devolver un único valor.
- **En el `from`**: se usa como si fuera una tabla. Necesita un alias.
- **En el `where`**: sirve para filtrar filas, comparando contra el valor o la lista que devuelve.

---

## Subconsultas en el SELECT

> Pista para los dos: antes de usar una subconsulta, probá resolverlo con un `join` a `consulta` y un `group by`, y compará el peso promedio con el que da sin el join. ¿Por qué cambia?

### 1. Ficha de cada dueño

**Enunciado.** Armar una ficha de cada dueño con su apellido, la cantidad de mascotas que tiene, el peso promedio de sus mascotas y la cantidad total de consultas que tuvieron sus mascotas. Los dueños sin mascotas también deben aparecer.

#### Resolución

```sql
select d.apellido,
       count(m.id_mascota)      as cantidad_mascotas,
       round(avg(m.peso_kg), 2) as peso_promedio,
       (select count(*)
          from consulta c
          join mascota m2 on m2.id_mascota = c.id_mascota
         where m2.id_dueno = d.id_dueno) as cantidad_consultas
from dueno d
left join mascota m on m.id_dueno = d.id_dueno
group by d.id_dueno, d.apellido
order by d.apellido;
```

Resultado esperado:

| apellido | cantidad_mascotas | peso_promedio | cantidad_consultas |
|---|---|---|---|
| Alvarez | 0 | NULL | 0 |
| Fernández | 3 | 13.27 | 4 |
| Gómez | 2 | 16.35 | 4 |
| Pérez | 2 | 4.90 | 4 |
| Sosa | 3 | 9.60 | 4 |

**Por qué hace falta la subconsulta.** La tentación es agregar un `left join consulta` y contar todo en el mismo `group by`. La cantidad de consultas y la de mascotas (con `count(distinct m.id_mascota)`) dan bien, pero el **peso promedio da mal**: el join repite cada mascota una vez por cada consulta que tuvo. En Gómez, Rocky (28,5 kg) tiene 3 consultas y Michi (4,2 kg) tiene 1, así que el promedio se calcula sobre 28,5 + 28,5 + 28,5 + 4,2 y da **22,43** en vez de **16,35**. Fernández da 18,70 en vez de 13,27.

`avg(distinct m.peso_kg)` tampoco sirve: con estos datos da bien de casualidad, pero si un dueño tiene dos mascotas con el mismo peso, las cuenta como una sola.

La subconsulta en el `select` cuenta las consultas por separado, así el join del `from` queda solo con `mascota` y no multiplica filas.

### 2. Resumen por especie

**Enunciado.** Armar un resumen por especie con la especie, la cantidad de mascotas, el peso promedio de las mascotas de esa especie y la cantidad total de consultas que tuvieron. Ordenar de la especie con más mascotas a la que tiene menos.

#### Resolución

```sql
select m.especie,
       count(*)                 as cantidad_mascotas,
       round(avg(m.peso_kg), 2) as peso_promedio,
       (select count(*)
          from consulta c
          join mascota m2 on m2.id_mascota = c.id_mascota
         where m2.especie = m.especie) as cantidad_consultas
from mascota m
group by m.especie
order by cantidad_mascotas desc, m.especie;
```

Resultado esperado:

| especie | cantidad_mascotas | peso_promedio | cantidad_consultas |
|---|---|---|---|
| Perro | 5 | 19.30 | 9 |
| Gato | 3 | 4.17 | 4 |
| Ave | 1 | 0.30 | 1 |
| Conejo | 1 | 1.80 | 2 |

**Por qué hace falta la subconsulta.** Es el mismo problema que el ejercicio 1, pero se nota todavía más: con un `left join consulta` y `count(*)`, el resultado dice que hay **10 perros** en vez de 5, porque cuenta consultas en lugar de mascotas. El peso promedio de los perros da 20,65 en vez de 19,30.

La novedad es que la subconsulta está correlacionada con una columna agrupada (`m.especie`), no con la clave de una tabla. Se puede, porque `especie` está en el `group by`.

---

## Subconsultas en el FROM

### 3. Promedio, máximo y mínimo de consultas por mascota

**Enunciado.** Calcular, en una sola fila, el promedio, el máximo y el mínimo de consultas por mascota. Las mascotas que nunca tuvieron una consulta cuentan con 0.

#### Resolución

```sql
select round(avg(cantidad), 2) as promedio,
       max(cantidad)           as maximo,
       min(cantidad)           as minimo
from (select m.id_mascota, count(c.id_consulta) as cantidad
        from mascota m
        left join consulta c on c.id_mascota = m.id_mascota
       group by m.id_mascota) as x;
```

Resultado esperado:

| promedio | maximo | minimo |
|---|---|---|
| 1.60 | 3 | 0 |

**Por qué hace falta la subconsulta.** Hay que agregar dos veces: primero contar las consultas de cada mascota, y después sacar promedio, máximo y mínimo de esas cantidades. `max(count(...))` da error, porque no se pueden anidar funciones de agregado en una misma consulta.

El promedio solo sí sale sin subconsulta, dividiendo el total de consultas por la cantidad de mascotas: `count(c.id_consulta) * 1.0 / count(distinct m.id_mascota)` da 1,6. Lo que no sale es el máximo y el mínimo; con `order by ... limit 1` se consigue uno solo, y no los dos en la misma fila.

El `left join` es necesario para que Bruno, que no tiene consultas, cuente con 0: si no, el mínimo daría 1.

### 4. La mascota más consultada de cada dueño

**Enunciado.** Para cada dueño, mostrar su apellido y la cantidad de consultas que tuvo su mascota más consultada. Ordenar de mayor a menor cantidad.

#### Resolución

```sql
select d.apellido, max(x.cantidad) as max_consultas
from dueno d
join (select m.id_dueno, m.id_mascota, count(c.id_consulta) as cantidad
        from mascota m
        left join consulta c on c.id_mascota = m.id_mascota
       group by m.id_dueno, m.id_mascota) as x on x.id_dueno = d.id_dueno
group by d.id_dueno, d.apellido
order by max_consultas desc, d.apellido;
```

Resultado esperado:

| apellido | max_consultas |
|---|---|
| Gómez | 3 |
| Fernández | 2 |
| Pérez | 2 |
| Sosa | 2 |

**Por qué hace falta la subconsulta.** Otra vez hay dos niveles: contar por mascota y después quedarse con el máximo por dueño. Si se agrupa directamente por dueño se obtiene el total de consultas de todas sus mascotas (4 para cada uno), no el de la más consultada. `order by ... limit 1` tampoco sirve, porque da un solo resultado para toda la tabla y no uno por dueño.

La tabla derivada `x` se usa como una tabla más: se le pone alias y se hace join con `dueno`. Alvarez no aparece porque no tiene mascotas; con `left join` aparecería con NULL.

---

## Subconsultas en el WHERE

> Pista para los dos: no vale mirar primero quién es el que más tiene y después escribir su matrícula o su id a mano. La consulta tiene que seguir funcionando si cambian los datos.


### 5. Consultas del veterinario que más atendió

**Enunciado.** Listar fecha, nombre de la mascota, apellido del veterinario y motivo de todas las consultas atendidas por el veterinario que más consultas atendió. Si hay empate, mostrar las consultas de todos los empatados.

#### Resolución

```sql
select c.fecha, m.nombre as mascota, v.apellido as veterinario, c.motivo
from consulta c
join mascota m     on m.id_mascota = c.id_mascota
join veterinario v on v.matricula = c.matricula
where c.matricula in (
    select matricula
    from consulta
    group by matricula
    having count(*) = (
        select max(cnt)
        from (select count(*) as cnt from consulta group by matricula) as sub
    )
)
order by c.fecha;
```

Resultado esperado: las 8 consultas de Ramírez.

| fecha | mascota | veterinario | motivo |
|---|---|---|---|
| 2023-03-10 | Rocky | Ramírez | Control anual |
| 2023-04-02 | Michi | Ramírez | Vacunación |
| 2023-06-18 | Luna | Ramírez | Chequeo general |
| 2023-08-25 | Pipo | Ramírez | Chequeo plumaje |
| 2024-01-15 | Rocky | Ramírez | Control post-operatorio |
| 2024-02-01 | Coco | Ramírez | Chequeo general |
| 2024-02-20 | Rocky | Ramírez | Vómitos |
| 2024-03-10 | Coco | Ramírez | Vacunación |

**Por qué hace falta la subconsulta.** El resultado es el detalle de las consultas, pero la condición depende de un dato agrupado (cuántas consultas atendió cada veterinario) comparado con otro agregado de ese agrupado (el máximo). Eso no entra en una sola consulta.

`order by count(*) desc limit 1` encuentra al veterinario, pero no devuelve sus consultas y, si hay empate, deja afuera a uno. Escribir `where c.matricula = 1001` después de mirar el resultado tampoco vale: deja de funcionar apenas cambian los datos.

Hay tres niveles de subconsulta: la de más adentro cuenta por veterinario, la del medio toma el máximo, y la del `in` devuelve las matrículas que llegan a ese máximo.

### 6. Mascotas del dueño que tiene más mascotas

**Enunciado.** Listar el apellido del dueño, el nombre y la especie de las mascotas del dueño que tiene más mascotas. Si hay empate, mostrar las mascotas de todos los empatados.

#### Resolución

```sql
select d.apellido as dueno, m.nombre, m.especie
from mascota m
join dueno d on d.id_dueno = m.id_dueno
where (select count(*) from mascota m2 where m2.id_dueno = m.id_dueno) = (
    select max(cnt)
    from (select count(*) as cnt from mascota group by id_dueno) as sub
)
order by d.apellido, m.nombre;
```

Resultado esperado: hay **empate** entre Fernández y Sosa, con 3 mascotas cada uno.

| dueno | nombre | especie |
|---|---|---|
| Fernández | Michi | Gato |
| Fernández | Pipo | Ave |
| Fernández | Rocky | Perro |
| Sosa | Bruno | Perro |
| Sosa | Coco | Conejo |
| Sosa | Nina | Perro |

**Por qué hace falta la subconsulta.** Este ejemplo muestra bien por qué no alcanza con `order by ... limit 1`: con estos datos, `select id_dueno, count(*) from mascota group by id_dueno order by count(*) desc limit 1` devuelve **solo a Sosa** y pierde a Fernández, que tiene la misma cantidad. Comparar contra el máximo devuelve a los dos.

A diferencia del ejercicio 5, acá el `where` usa una **subconsulta correlacionada**: `(select count(*) from mascota m2 where m2.id_dueno = m.id_dueno)` se calcula de nuevo para cada mascota de la consulta de afuera, con el dueño de esa mascota.
