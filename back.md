# 1. DER (Diagrama Entidad-Relación)

Es el modelo conceptual inicial. Define qué información se va a guardar y cómo se relaciona, sin importar el motor de base de datos.

* **Entidades:** Objetos del mundo real (ej. `Cliente`, `Producto`). Se dibujan como **rectángulos**.

* **Atributos:** Características de la entidad (ej. `Nombre`, `Precio`). Se dibujan como **óvalos**.

* **Atributo Clave (Primary Key):** Identificador único de la entidad (ej. `DNI`, `ID`). Se dibuja con el nombre **subrayado**.

* **Relaciones (Rombos):** Definen la interacción entre entidades (ej. un cliente "compra" productos). Se dibujan como **rombos** conectados por líneas a las entidades.
  * Sobre las líneas que unen el rombo con las entidades se escribe la **Cardinalidad**, que define cuántas instancias de una entidad se relacionan con la otra:
    * **1:1** (Uno a Uno).
    * **1:N** (Uno a Muchos).
    * **N:M** (Muchos a Muchos).

# 2. DLR (Diagrama Lógico Relacional)

Es el paso previo al código SQL. Traduce los conceptos abstractos del DER a estructuras (Tablas, Columnas, Claves).

* **PK (Primary Key / Clave Primaria):** Identificador único de un registro.

* **FK (Foreign Key / Clave Foránea):** Columna que hace referencia a la PK de otra tabla.

* **Líneas de Relación (Tipos de Flechas / Notación):** 
  En el DLR no hay rombos; las tablas se conectan directamente con líneas que van desde la PK hasta la FK. La notación visual más estándar es la **"Pata de Gallo" (Crow's Foot)** en los extremos de la línea:
  * **Uno (1 / Obligatorio):** Se dibuja con una (o dos) rayas pequeñas cruzando perpendicularmente la línea (`|` o `||`). Significa que debe existir exactamente un registro.
  * **Muchos (N):** Se dibuja dividiendo el final de la línea en tres ramas (como una pata de gallo o un tenedor `<`).
  * **Opcional (0):** Se dibuja con un pequeño círculo o "cero" (`O`) en la línea. Indica que puede haber cero registros asociados (relación no obligatoria).
  * *Ejemplo visual conjunto:* Si ves un círculo seguido de una pata de gallo (`-O-<`), se lee como "Cero a Muchos". Si ves dos rayas (`-||-`), se lee como "Exactamente Uno".

# 3. Reglas de Transformación (De DER a DLR)

* **Regla 1: Entidades Fuertes**
  * Cada entidad se convierte en una **Tabla**.
  * Los atributos se convierten en **Columnas**.
  * El atributo clave pasa a ser la **Primary Key (PK)**.

* **Regla 2: Relaciones 1 a N (Uno a Muchos)**
  * *Ejemplo:* 1 Departamento tiene N Empleados.
  * **Acción:** NO se crea tabla nueva. La PK de la tabla "Uno" (Departamento) viaja hacia la tabla "Muchos" (Empleados) y se convierte en una **Clave Foránea (FK)**.

* **Regla 3: Relaciones N a M (Muchos a Muchos)**
  * *Ejemplo:* N Estudiantes se anotan en M Cursos.
  * **Acción:** **SÍ se crea una tabla nueva** intermedia (ej. `Inscripciones`). Esta tabla heredará las PK de las dos entidades originales, que actuarán como **Claves Foráneas (FK)** y juntas formarán una PK compuesta.

* **Regla 4: Relaciones 1 a 1 (Uno a Uno)**
  * *Ejemplo:* 1 Ciudadano tiene 1 Pasaporte.
  * **Acción:** La PK de una de las tablas se envía a la otra como **FK**. Se suele colocar en la tabla con participación total/obligatoria.

# 4. SQL Joins  
* **INNER JOIN:** Devuelve solo los registros que tienen coincidencias en ambas tablas.
* **LEFT (OUTER) JOIN:** Devuelve todos los registros de la tabla izquierda, y las coincidencias de la tabla derecha. Si no hay coincidencia, devuelve NULL en el lado derecho.
* **RIGHT (OUTER) JOIN:** El inverso del LEFT JOIN. Devuelve todos los de la derecha y las coincidencias de la izquierda.
* **FULL (OUTER) JOIN:** Devuelve todos los registros cuando hay una coincidencia en cualquiera de las tablas.

# 5. SQL avanzado  
* **COUNT():** Cuenta filas.
* **SUM():** Suma valores.
* **AVG():** Calcula el promedio.
* **MAX() / MIN():** Encuentra el valor máximo o mínimo.
---
* **GROUP BY:** Agrupa filas que tienen los mismos valores en filas de resumen (ej. "encontrar el número de clientes en cada país"). Casi siempre se usa con funciones de agregación.
* **HAVING:** Es el equivalente al WHERE, pero se aplica a los grupos creados por el GROUP BY. (El WHERE filtra filas individuales antes de agrupar, HAVING filtra grupos después de agrupar).
```sql
SELECT producto, SUM(cantidad) AS cantidad_total
FROM ventas
GROUP BY producto
HAVING SUM(cantidad) > 10;
```
---
* **Subconsultas:**
```sql
SELECT nombre, salario
FROM empleados
WHERE salario > (
    SELECT AVG(salario)
    FROM empleados
);
```
---
* **Stored Procedures:**
```sql
-- 1. Crear el Stored Procedure
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
y se ejecuta con:
```sql
EXEC GetCustomersByCity @City = 'Madrid';
```

# 6. SQL vs PostgreSQL  
*   **Concatenación de Cadenas (Strings):**
    *   **SQL Server:** Usa el operador de suma `+` (ej. `'Hola ' + 'Mundo'`).
    *   **PostgreSQL:** Usa el operador pipe doble `||` (ej. `'Hola ' || 'Mundo'`).

*   **Límite de Resultados:**
    *   **SQL Server:** Usa la cláusula `TOP` al principio del SELECT (ej. `SELECT TOP 10 * FROM tabla`).
    *   **PostgreSQL:** Usa la cláusula `LIMIT` al final de la consulta (ej. `SELECT * FROM tabla LIMIT 10`).

*   **Identificadores (Nombres de tablas/columnas con espacios o palabras reservadas):**
    *   **SQL Server:** Los envuelve en corchetes `[Mi Columna]`.
    *   **PostgreSQL:** Los envuelve en comillas dobles `"Mi Columna"`. *(Nota: las comillas simples son estrictamente para valores de texto/cadenas)*.

*   **Fecha y Hora Actual:**
    *   **SQL Server:** Utiliza la función `GETDATE()`.
    *   **PostgreSQL:** Utiliza `NOW()` o `CURRENT_TIMESTAMP`.

*   **Manejo de Valores Nulos:**
    *   **SQL Server:** Suele usar la función `ISNULL(columna, 'valor_defecto')`.
    *   **PostgreSQL:** Usa la función estándar `COALESCE(columna, 'valor_defecto')`. *(COALESCE también funciona en SQL Server)*.

*   **Claves Primarias Autoincrementales:**
    *   **SQL Server:** Se define con `IDENTITY(1,1)` (ej. `ID INT IDENTITY(1,1)`).
    *   **PostgreSQL:** Se usa el seudotipo `SERIAL` o la sintaxis moderna `INT GENERATED ALWAYS AS IDENTITY`.

*   **Conversión de Tipos de Datos (Casting):**
    *   **SQL Server:** Usa `CAST(columna AS tipo)` o `CONVERT(tipo, columna)`.
    *   **PostgreSQL:** Soporta `CAST()`, pero tiene un atajo sintáctico muy útil usando doble dos puntos `::` (ej. `columna::varchar`).

*   **Declaración de Variables:**
    *   **SQL Server:** Se utiliza el símbolo arroba `@` (ej. `DECLARE @mi_variable INT`).
    *   **PostgreSQL:** No llevan arroba. Se declaran dentro de un bloque `DECLARE` en PL/pgSQL.

*   **Condicionales IF/ELSE en scripts:**
    *   **SQL Server:** Se pueden usar libremente en medio del código usando `IF ... BEGIN ... END`.
    *   **PostgreSQL:** Para usar lógica condicional procedimental, el código debe estar dentro de una función o de un bloque anónimo estructurado (ej. `DO $$ BEGIN ... END $$;`).

# 7. Crear server node
```js
const express = require('express');
const app = express();
const port = 3000;

// Ruta básica
app.get('/', (req, res) => {
  res.send('¡Hola desde mi servidor con Node.js y Express!');
});

// Iniciar servidor
app.listen(port, () => {
  console.log(`Servidor escuchando en http://localhost:${port}`);
});

```