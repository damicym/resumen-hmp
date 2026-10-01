# Resumen ampliado de Backend y Bases de Datos

Este apunte reúne los conceptos fundamentales para diseñar una base de datos relacional, consultarla con SQL y conectarla con un servidor básico desarrollado con Node.js y Express.

## 1. DER (Diagrama Entidad-Relación)

El **DER** es un modelo conceptual: representa qué información necesita almacenar un sistema y cómo se relaciona, sin depender todavía de SQL Server, PostgreSQL u otro motor.

### Elementos principales

- **Entidad:** objeto o concepto sobre el cual se guardan datos, como `Cliente`, `Producto` o `Curso`. Se representa con un rectángulo.
- **Atributo:** característica de una entidad, como `nombre`, `precio` o `email`. En la notación clásica se representa con un óvalo.
- **Atributo clave:** identifica de forma única cada instancia, como `id_cliente` o `dni`. Suele aparecer subrayado.
- **Relación:** asociación entre entidades. Por ejemplo, un cliente **realiza** pedidos. Se representa con un rombo.
- **Cardinalidad:** indica cuántas instancias pueden participar en una relación.

### Cardinalidades

- **1:1:** una instancia de A se relaciona, como máximo, con una de B. Ejemplo: una persona y su pasaporte vigente.
- **1:N:** una instancia de A se relaciona con muchas de B, pero cada B pertenece a una sola A. Ejemplo: un departamento tiene muchos empleados.
- **N:M:** muchas instancias de A se relacionan con muchas de B. Ejemplo: muchos estudiantes cursan muchas materias.

La participación también puede ser **obligatoria** (debe existir la relación) u **opcional** (puede no existir). Por ejemplo, todo pedido debe pertenecer a un cliente, pero un cliente podría no haber realizado pedidos.

## 2. DLR (Diagrama Lógico Relacional)

El **DLR** transforma el modelo conceptual en una estructura cercana a la implementación. Las entidades pasan a ser tablas, los atributos se convierten en columnas y las relaciones se expresan mediante claves.

- **Tabla:** conjunto de datos sobre un mismo tipo de objeto.
- **Fila o registro:** una instancia concreta.
- **Columna o campo:** una característica del registro.
- **Primary Key (PK):** identifica cada fila de manera única. No debe repetirse ni contener `NULL`.
- **Foreign Key (FK):** referencia una clave de otra tabla y mantiene la integridad de la relación.
- **Clave compuesta:** clave formada por dos o más columnas.
- **`UNIQUE`:** impide valores repetidos.
- **`NOT NULL`:** obliga a ingresar un valor.

En la notación **Pata de Gallo**, `|` representa uno, `O` representa cero u opcional y la pata de gallo representa muchos. Así, `O<` significa cero a muchos, `|<` uno a muchos y `||` exactamente uno.

## 3. Transformación de DER a DLR

### Entidades fuertes

Cada entidad se convierte en una tabla; sus atributos pasan a ser columnas y su identificador se transforma en PK.

```sql
CREATE TABLE clientes (
    id_cliente INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE
);
```

### Relación 1:N

La PK del lado **uno** se incorpora como FK en la tabla del lado **muchos**.

```sql
CREATE TABLE departamentos (
    id_departamento INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE empleados (
    id_empleado INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    id_departamento INT NOT NULL,
    FOREIGN KEY (id_departamento)
        REFERENCES departamentos(id_departamento)
);
```

### Relación N:M

Se crea una tabla intermedia que contiene las FK de ambas entidades. También puede guardar atributos propios de la relación.

```sql
CREATE TABLE inscripciones (
    id_estudiante INT,
    id_curso INT,
    fecha_inscripcion DATE NOT NULL,
    nota_final DECIMAL(4,2),
    PRIMARY KEY (id_estudiante, id_curso),
    FOREIGN KEY (id_estudiante) REFERENCES estudiantes(id_estudiante),
    FOREIGN KEY (id_curso) REFERENCES cursos(id_curso)
);
```

### Relación 1:1

La PK de una tabla se coloca como FK en la otra. Para garantizar que siga siendo 1:1, esa FK debe ser `UNIQUE`. Normalmente se coloca en la tabla dependiente o cuya participación sea obligatoria.

### Entidad débil

No puede identificarse sin otra entidad. Su PK suele combinar la PK de la entidad propietaria con un identificador parcial. Por ejemplo, una línea de pedido puede identificarse mediante `id_pedido` y `numero_linea`.

## 4. Normalización

La normalización reduce duplicaciones y evita anomalías al insertar, actualizar o eliminar datos.

- **1FN:** cada celda contiene un único valor y no existen grupos repetidos.
- **2FN:** cumple 1FN y cada atributo no clave depende de la clave completa.
- **3FN:** cumple 2FN y los atributos no clave no dependen de otros atributos no clave.

Por ejemplo, no conviene repetir el nombre y domicilio del cliente en cada pedido. La tabla `pedidos` guarda `id_cliente` y los datos personales permanecen en `clientes`.

## 5. Operaciones básicas de SQL

```sql
SELECT nombre, precio
FROM productos
WHERE precio > 1000
ORDER BY precio DESC;
```

Orden lógico simplificado de una consulta:

1. `FROM`: determina el origen.
2. `WHERE`: filtra filas.
3. `GROUP BY`: forma grupos.
4. `HAVING`: filtra grupos.
5. `SELECT`: elige las columnas del resultado.
6. `ORDER BY`: ordena.
7. `LIMIT` o `TOP`: limita la cantidad de filas.

### Insertar, actualizar y eliminar

```sql
INSERT INTO productos (id_producto, nombre, precio)
VALUES (1, 'Teclado', 25000);

UPDATE productos
SET precio = 28000
WHERE id_producto = 1;

DELETE FROM productos
WHERE id_producto = 1;
```

En `UPDATE` y `DELETE`, omitir el `WHERE` afecta todas las filas de la tabla.

## 6. SQL JOIN

Los `JOIN` combinan datos de tablas relacionadas, normalmente mediante una PK y una FK.

- **`INNER JOIN`:** solo devuelve filas con coincidencia en ambas tablas.
- **`LEFT JOIN`:** devuelve todas las filas de la izquierda; si no existe coincidencia, las columnas derechas contienen `NULL`.
- **`RIGHT JOIN`:** devuelve todas las filas de la derecha. Puede reescribirse como `LEFT JOIN` invirtiendo las tablas.
- **`FULL OUTER JOIN`:** devuelve coincidencias y filas sin pareja de ambos lados.

```sql
SELECT c.nombre, p.id_pedido
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

Esta consulta incluye incluso a los clientes que todavía no realizaron pedidos.

## 7. Agregación y agrupamiento

- `COUNT(*)`: cuenta filas.
- `COUNT(columna)`: cuenta valores no nulos.
- `SUM(columna)`: suma valores.
- `AVG(columna)`: calcula el promedio.
- `MAX(columna)` y `MIN(columna)`: obtienen el mayor y menor valor.

`WHERE` filtra antes de agrupar; `HAVING` filtra después de formar y calcular los grupos.

```sql
SELECT producto, SUM(cantidad) AS cantidad_total
FROM ventas
WHERE fecha >= '2026-01-01'
GROUP BY producto
HAVING SUM(cantidad) > 10
ORDER BY cantidad_total DESC;
```

Las columnas del `SELECT` que no estén dentro de una función de agregación deben aparecer normalmente en el `GROUP BY`.

## 8. Subconsultas

Una subconsulta es una consulta incluida dentro de otra.

```sql
SELECT nombre, salario
FROM empleados
WHERE salario > (
    SELECT AVG(salario)
    FROM empleados
);
```

La consulta interna calcula el promedio y la externa devuelve quienes lo superan. Si la subconsulta puede devolver varias filas, suelen utilizarse `IN`, `EXISTS`, `ANY` o `ALL` en lugar de `=`.

## 9. Transacciones

Una transacción agrupa operaciones que deben completarse como una unidad, por ejemplo una transferencia bancaria.

```sql
BEGIN TRANSACTION;

UPDATE cuentas SET saldo = saldo - 1000 WHERE id_cuenta = 1;
UPDATE cuentas SET saldo = saldo + 1000 WHERE id_cuenta = 2;

COMMIT;
```

`COMMIT` confirma los cambios y `ROLLBACK` los deshace ante un error. Las propiedades **ACID** son atomicidad, consistencia, aislamiento y durabilidad.

## 10. Procedimientos almacenados

Un **stored procedure** es un conjunto de instrucciones guardado en la base de datos. Puede recibir parámetros y devolver resultados. Ejemplo en SQL Server:

```sql
CREATE PROCEDURE GetCustomersByCity
    @City NVARCHAR(50)
AS
BEGIN
    SELECT CustomerID, CustomerName, City
    FROM Customers
    WHERE City = @City;
END;
GO
```

```sql
EXEC GetCustomersByCity @City = 'Madrid';
```

Pueden centralizar lógica y permisos, aunque no siempre conviene colocar toda la lógica de negocio dentro de la base de datos.

## 11. SQL Server y PostgreSQL

### Diferencias frecuentes

- **Concatenar texto:** SQL Server usa `'Hola ' + 'Mundo'`; PostgreSQL usa `'Hola ' || 'Mundo'`.
- **Limitar resultados:** SQL Server usa `SELECT TOP 10 ...`; PostgreSQL usa `... LIMIT 10`.
- **Identificadores especiales:** SQL Server usa `[Mi Columna]`; PostgreSQL usa `"Mi Columna"`.
- **Fecha actual:** SQL Server usa `GETDATE()`; PostgreSQL usa `NOW()` o `CURRENT_TIMESTAMP`.
- **Nulos:** SQL Server ofrece `ISNULL`; ambos admiten `COALESCE`.
- **Conversión:** ambos admiten `CAST`; SQL Server también tiene `CONVERT` y PostgreSQL el atajo `valor::tipo`.
- **Variables:** SQL Server utiliza `@variable`; PostgreSQL las declara dentro de funciones o bloques PL/pgSQL.
- **Condicionales:** SQL Server permite `IF ... BEGIN ... END`; PostgreSQL requiere una función o un bloque como `DO
BEGIN...END
;`.

Los nulos no se comparan con `= NULL`; se utiliza `IS NULL` o `IS NOT NULL`.

### Columnas autoincrementales

```sql
-- SQL Server
id INT IDENTITY(1,1) PRIMARY KEY

-- PostgreSQL, sintaxis moderna
id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

PostgreSQL también admite `SERIAL`, aunque `IDENTITY` es la alternativa moderna.

## 12. Servidor básico con Node.js y Express

**Node.js** ejecuta JavaScript fuera del navegador. **Express** simplifica la creación de servidores HTTP y APIs.

### Preparación

```bash
npm init -y
npm install express
```

Archivo `server.js`:

```js
const express = require('express');

const app = express();
const port = 3000;

app.use(express.json());

app.get('/', (req, res) => {
  res.send('¡Hola desde mi servidor con Node.js y Express!');
});

app.get('/api/productos', (req, res) => {
  res.json([
    { id: 1, nombre: 'Teclado' },
    { id: 2, nombre: 'Mouse' }
  ]);
});

app.post('/api/productos', (req, res) => {
  const producto = req.body;
  res.status(201).json(producto);
});

app.listen(port, () => {
  console.log(`Servidor disponible en http://localhost:${port}`);
});
```

Ejecutar con `node server.js`. La ruta `/` responde texto y `/api/productos` responde JSON.

### Métodos HTTP y CRUD

- `GET`: consultar (**Read**).
- `POST`: crear (**Create**).
- `PUT` o `PATCH`: actualizar (**Update**).
- `DELETE`: eliminar (**Delete**).

En una aplicación real, la ruta valida los datos, llama a la lógica de negocio y accede a la base mediante consultas parametrizadas.

## 13. Organización de un backend

```text
src/
├── routes/         Rutas y métodos HTTP
├── controllers/    Peticiones y respuestas
├── services/       Lógica de negocio
├── repositories/   Acceso a la base de datos
├── middlewares/    Validación, autenticación y errores
└── app.js           Configuración de la aplicación
server.js            Inicio del servidor
```

Separar responsabilidades facilita las pruebas, el mantenimiento y la evolución del proyecto.

## 14. Seguridad y buenas prácticas

- Usar **consultas parametrizadas** para evitar inyección SQL.
- No guardar contraseñas en texto plano; aplicar un hash adecuado.
- Guardar credenciales en variables de entorno, no en el código.
- Validar los datos recibidos.
- Devolver códigos HTTP apropiados: `200`, `201`, `400`, `404` y `500`.
- Manejar errores sin exponer información interna.
- Aplicar permisos mínimos al usuario de la base.
- Crear índices en columnas consultadas con frecuencia, especialmente PK, FK y campos de búsqueda.
- Usar transacciones cuando varias modificaciones deban confirmarse o deshacerse juntas.

## 15. Flujo completo

1. Analizar el problema y reconocer entidades, atributos y reglas.
2. Crear el DER con relaciones, cardinalidades y participaciones.
3. Transformarlo en un DLR con tablas, PK y FK.
4. Normalizar y elegir tipos de datos adecuados.
5. Crear la estructura con SQL y restricciones.
6. Probar consultas, joins, agregaciones y transacciones.
7. Crear el servidor y sus rutas HTTP.
8. Conectar el backend con la base de datos.
9. Validar entradas, manejar errores y proteger datos sensibles.
10. Probar cada capa antes de publicar.

En síntesis, el DER explica **qué existe y cómo se relaciona**, el DLR define **cómo se organiza en tablas**, SQL permite **crear y manipular los d