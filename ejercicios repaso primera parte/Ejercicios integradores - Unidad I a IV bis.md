# Ejercicios integradores — Unidad I a IV bis

Diseño y Administración de Base de Datos — UTN

Los ejercicios 3 a 10 se resuelven sobre las tablas que salen de los tres DER del ejercicio 2. Conviene resolver primero ese punto.

---

## Ejercicio 1 — Multiple choice (teoría)

Seleccione la respuesta correcta.

**1.** La independencia de los datos es:

- a) Que cada aplicación tenga sus propios archivos de datos.
- b) La inmunidad de las aplicaciones a cambios en la representación física y en las técnicas de acceso a los datos.
- c) Que los datos no dependan del motor de base de datos que se utilice.
- d) Que los datos se puedan guardar sin metadatos.

**2.** Dentro del procesador de consultas, el módulo que traduce las instrucciones DML a un plan de evaluación es:

- a) El gestor de archivos.
- b) El intérprete de DDL.
- c) El compilador de DML.
- d) El gestor de memoria.

**3.** La mejora principal del modelo de red respecto del modelo jerárquico es que:

- a) Permite que un nodo hijo tenga más de un nodo padre.
- b) Ya no hace falta conocer toda la estructura para acceder a un dato.
- c) Implementa claves primarias y foráneas.
- d) Las relaciones se definen a nivel lógico y no físico.

**4.** Un sistema OLAP se caracteriza por:

- a) Muchas operaciones cortas de alta, baja y modificación de registros.
- b) Datos normalizados y backups muy frecuentes.
- c) Datos que provienen directamente de las aplicaciones de usuario.
- d) Consultas complejas sobre datos históricos consolidados, generalmente desnormalizados.

**5.** En la arquitectura ANSI/SPARC, el nivel conceptual:

- a) Es el más cercano al almacenamiento físico.
- b) Es único y describe qué datos se almacenan y cómo se relacionan.
- c) Existe uno por cada usuario o aplicación.
- d) Define el tamaño de las páginas de memoria y de los cilindros del disco.

**6.** Un campo definido como clave foránea:

- a) Puede quedar vacío (NULL) o, si tiene un valor, ese valor debe existir en el campo referenciado de la tabla padre.
- b) Nunca puede ser NULL.
- c) Sólo puede definirse una por tabla.
- d) Debe tener además una restricción de unicidad.

**7.** Una relación muchos a muchos entre `alumnos` y `materias` se resuelve:

- a) Agregando en `alumnos` una clave foránea hacia `materias`.
- b) Agregando una restricción de unicidad en ambas tablas.
- c) Creando una tabla intermedia y dos relaciones 1:N.
- d) No se puede representar en el modelo relacional.

**8.** Según la teoría, ¿cuál de estas sentencias pertenece a DML?

- a) `create table`
- b) `alter table`
- c) `drop table`
- d) `select`

**9.** La cláusula `having`:

- a) Filtra sobre el resultado ya agrupado y se usa junto a `group by`.
- b) Filtra las filas antes de agruparlas.
- c) Reemplaza a `where` cuando la consulta tiene joins.
- d) Ordena los grupos del resultado.

**10.** La propiedad que garantiza que, una vez confirmada una transacción (`commit`), sus cambios persisten aunque el sistema falle después, es:

- a) Atomicidad
- b) Aislamiento
- c) Durabilidad
- d) Consistencia

---

## Ejercicio 2 — DER

Para cada caso, obtener el diagrama entidad-relación indicando entidades, atributos, claves primarias, claves foráneas y cardinalidades.

### 2.a — Academia de idiomas

Una academia de idiomas quiere informatizar la gestión de sus cursos.

- De cada **idioma** que enseña se conoce su nombre, que no se repite.
- Cada **curso** corresponde a un único idioma y a un nivel (A1, A2, B1, B2, C1 o C2). Tiene un precio mensual y una cantidad de cupos disponibles. Lo dicta un único profesor, y un profesor puede dictar varios cursos.
- De los **profesores** se registra nombre, apellido, email (único) y fecha de ingreso.
- De los **alumnos** se registra DNI (único), nombre, apellido, email y fecha de nacimiento.
- Un alumno puede inscribirse en varios cursos y un curso tiene varios alumnos. Al inscribirse se registra la fecha de inscripción y, cuando el curso termina, la nota final. Mientras el alumno cursa, la nota queda vacía.

> Pregunta guía: ¿en qué tabla va la nota final? ¿Es un dato del alumno, del curso o de otra cosa?

### 2.b — Plataforma de música

Una plataforma de streaming quiere registrar su catálogo y lo que escuchan sus usuarios.

- De cada **artista** se guarda nombre y país.
- Cada **álbum** pertenece a un único artista, y tiene título y año de lanzamiento.
- Cada álbum tiene varias **canciones**. Una canción se identifica por su número de pista *dentro del álbum*: la pista 1 de un álbum no tiene nada que ver con la pista 1 de otro. De cada canción se guarda título y duración en segundos.
- De los **usuarios** se guarda email (único), nombre, fecha de alta y plan (gratis o premium).
- Se quiere registrar cada **reproducción** con su fecha y hora, sabiendo que un usuario puede escuchar la misma canción muchas veces.

> Pregunta guía: ¿qué tipo de entidad es la canción? ¿Cuál es su clave primaria?

### 2.c — Empresa de envíos

Una empresa de logística quiere ordenar la información de sus envíos.

- La empresa tiene **sucursales**, de las que se conoce nombre y ciudad.
- Los **empleados** se identifican por un número de legajo que asigna Recursos Humanos. De cada uno se registra nombre, apellido, la sucursal en la que trabaja y quién es su jefe directo, que también es un empleado de la empresa. El gerente general no tiene jefe.
- Los **clientes** son empresas: CUIT (único), razón social y email.
- Cada **envío** pertenece a un cliente, sale de una sucursal de origen y lo lleva un repartidor (un empleado; puede no estar asignado todavía). Además tiene ciudad de destino, fecha de despacho, fecha de entrega (vacía mientras no se entregó), peso en kg y costo.

> Pregunta guía: ¿cómo se modela que el jefe también sea un empleado?

---

## Ejercicio 3 — DDL: CREATE, DROP y ALTER

### 3.a — Script de creación (DER 2.a, Academia de idiomas)

Escribir un script que se pueda ejecutar varias veces sin error: primero debe eliminar las tablas si existen y después crearlas, con sus claves primarias y foráneas. Además:

- El nombre del idioma y el email del profesor no se pueden repetir.
- El DNI del alumno es obligatorio y no se puede repetir.
- El nivel del curso sólo puede ser `'A1'`, `'A2'`, `'B1'`, `'B2'`, `'C1'` o `'C2'`.
- Los cupos disponibles nunca pueden ser negativos.
- La nota final, si está cargada, debe estar entre 1 y 10.

> Pista: pensá en qué orden hay que borrar las tablas para que las claves foráneas no lo impidan.

### 3.b — Modificar la estructura (DER 2.b, Plataforma de música)

Con las tablas ya creadas, escribir las sentencias para:

1. Agregar a `usuario` una columna `pais` de tipo texto (hasta 50 caracteres).
2. Renombrar la columna `plan` de `usuario` a `tipo_plan`.
3. Hacer obligatorio el año del álbum.
4. *(Extra)* Agregar a `cancion` una restricción que obligue a que la duración sea mayor a 0.

---

## Ejercicio 4 — Consultas simples

**4.a** (Academia) Listar nombre, apellido y email de los alumnos nacidos antes del año 2000, ordenados por apellido y nombre.

**4.b** (Envíos) Listar número de envío, ciudad de destino, peso y fecha de despacho de los envíos de más de 20 kg que todavía no fueron entregados. Ordenar del despacho más reciente al más antiguo.

---

## Ejercicio 5 — INNER JOIN y LEFT JOIN

**5.a** (Academia) Listar apellido y nombre del alumno, idioma, nivel del curso y nota final de todas las inscripciones del año 2025 con nota final 7 o más. Ordenar por apellido y nombre del alumno.

**5.b** (Envíos) Listar legajo, apellido y nombre de todos los empleados que trabajan en sucursales de la ciudad de Rosario, junto con el apellido de su jefe. Los empleados que no tienen jefe también deben aparecer.

---

## Ejercicio 6 — Condiciones con IN y LIKE

**6.a** (Música) Listar el título de la canción, el título del álbum y el nombre del artista de todas las canciones cuyo título contenga la palabra "amor", sin importar mayúsculas o minúsculas.

**6.b** (Academia) Listar número de curso, idioma, nivel y precio mensual de los cursos de Inglés, Portugués o Italiano que sean de nivel B1 o B2.

---

## Ejercicio 7 — Funciones de agregado

**7.a** (Academia) Listar, para cada curso, su número, idioma, nivel y la cantidad de alumnos inscriptos. Los cursos sin inscriptos deben aparecer con 0. Ordenar de mayor a menor cantidad de alumnos.

**7.b** (Envíos) Listar, para cada sucursal de origen, el peso promedio y el costo promedio de los envíos despachados en 2025, redondeados a 2 decimales.

---

## Ejercicio 8 — HAVING

**8.a** (Envíos) Listar las sucursales cuyo envío más pesado supere los 100 kg, mostrando el nombre de la sucursal y ese peso máximo.

**8.b** (Música) Listar los artistas cuyo primer álbum sea anterior a 1990, mostrando el nombre del artista y el año de ese primer álbum.

---

## Ejercicio 9 — Subconsultas

**9.a** (Música) Listar título y duración de las canciones que duran más que el promedio de duración de todas las canciones. Ordenar de la más larga a la más corta.

**9.b** (Envíos) Listar legajo, apellido, nombre y cantidad de envíos del repartidor (o los repartidores, si hay empate) que realizó la mayor cantidad de envíos.

---

## Ejercicio 10 — Transacciones

(Academia) Inscribir al alumno 5 en el curso 3 y, en la misma operación, descontar un cupo disponible de ese curso. Las dos cosas tienen que pasar juntas o no pasar ninguna.

Responder además: si el curso ya no tenía cupos disponibles, ¿qué pasa con la inscripción?
