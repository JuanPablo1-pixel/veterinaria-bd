# Diseño y Administración de Base de Datos — Unidad IV: la veterinaria

Material de práctica del ejercicio de la veterinaria (Unidad IV — DDL/DML/consultas).

## Contenido

- **`sql/01_esquema.sql`** — creación de las 6 tablas (`dueno`, `mascota`, `veterinario`, `consulta`, `medicamento`, `receta`) para Supabase/PostgreSQL. Es idempotente, se puede correr las veces que haga falta.
- **`sql/02_datos.sql`** — datos de ejemplo para practicar. Se corre después del esquema.
- **`ejercicios/veterinaria.md`** — guía de ejercicios de consultas sobre este modelo.
- **`ejercicios repaso primera parte/`** — ejercicios integradores de repaso (Unidad I a IV bis): teoría, DER, DDL, consultas, subconsultas y transacciones.
- **`practica/`** — espacio opcional para practicar el flujo de Pull Request (ver abajo).

## Cómo usar la base

1. Creá tu propio proyecto gratis en [supabase.com](https://supabase.com).
2. Abrí el **SQL Editor** del proyecto.
3. Pegá y corré `sql/01_esquema.sql`.
4. Pegá y corré `sql/02_datos.sql`.
5. Ya podés resolver los ejercicios de `ejercicios/veterinaria.md` contra tu propia base.

## Practicar el flujo de Fork + Pull Request

Esto **no es una entrega ni se corrige** — es una forma de practicar cómo se usa git y GitHub en un caso real, sin que nadie tenga que pedir acceso a nadie:

1. **Fork** este repositorio (botón arriba a la derecha en GitHub) → te queda una copia en tu cuenta.
2. En tu copia, agregá tu archivo (por ejemplo, alguna consulta resuelta) dentro de `practica/apellido-nombre/`.
3. Abrí un **Pull Request** desde tu copia hacia este repositorio.
4. Si querés, el profesor puede revisarlo y dejar comentarios directamente sobre el código.

### Ejemplo de estructura

```
practica/
  gomez-maria/
    consultas-tabla.sql
    consultas-multitabla.sql
```
