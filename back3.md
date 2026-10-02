# Guía de Estudio Teórica (Backend & APIs)

Esta guía está diseñada específicamente para rendir un **examen escrito (en papel)**. En lugar de enfocarse en cómo ejecutar o inicializar código, se centra puramente en **definiciones claras, conceptos clave y cómo explicar cada tema con palabras**.

## 1. Node.js
* **¿Qué es?** Es un entorno de ejecución (*runtime*) que permite compilar y ejecutar código JavaScript fuera de un navegador web, del lado del servidor. Está construido sobre el motor V8 de Google Chrome.
* **Características principales:** Es asíncrono, está orientado a eventos y utiliza un modelo de un solo hilo de ejecución (*single-thread*).
* **El Event Loop (Bucle de Eventos):** Es el mecanismo central de Node.js. Permite que el servidor sea "no bloqueante". Cuando llega una tarea pesada o lenta (ej. buscar datos en la base de datos o leer un archivo), el Event Loop la delega al sistema operativo en el fondo y continúa escuchando y atendiendo nuevas peticiones de los usuarios. Cuando la tarea pesada termina, emite un evento para avisar y devuelve el resultado.
* **Módulos:** Son bloques de código independientes y reutilizables. En Node.js, el código se divide en diferentes archivos (módulos) para mantener el orden. Para compartir un módulo entre archivos se debe "exportar" (ej. `module.exports`) en el archivo origen y luego "importar" (ej. `require()`) en el destino.
  * *Ejemplo para escribir en papel:*
    ```js
    // Exportar
    module.exports = { sumar: (a,b) => a + b };
    // Importar
    const math = require('./math');
    ```

## 2. Arquitectura MVC / Folder Based
* **¿Qué es?** Es un patrón de diseño arquitectónico y una convención de carpetas para separar las responsabilidades del código (Separation of Concerns).
* **Modelo (Model):** Es la capa encargada de interactuar exclusivamente con la Base de Datos. Maneja la forma en que los datos se guardan, se buscan o se borran.
* **Vista (View):** Es la interfaz gráfica que ve el usuario final. En el desarrollo de APIs REST puras, esta capa no suele existir en el servidor, ya que el backend solo devuelve texto en formato JSON y el encargado de la vista pasa a ser el Frontend (React, HTML/CSS).
* **Controlador (Controller):** Es el "cerebro" (la lógica de negocio). Su función es recibir la petición del usuario, llamar al Modelo para pedirle datos, procesarlos y decidir qué responderle al usuario (ej. un mensaje de éxito o un error).
* **Rutas (Routes):** Son archivos que funcionan como "recepcionistas". Su única tarea teórica es escuchar una URL específica (ej. `/api/usuarios`) y derivarla al Controlador adecuado para que la atienda. No deben tener lógica de programación.

## 3. Express, Rutas y Middlewares
* **Express.js:** Es un framework (marco de trabajo) minimalista para Node.js. Oculta la complejidad de crear servidores web nativos y facilita enormemente el enrutamiento y el manejo de peticiones HTTP.
* **Rutas (Endpoints):** Son las direcciones únicas a las que un cliente puede llamar dentro del servidor. Se componen de una URL y un verbo HTTP.
* **Middlewares:** Son funciones "intermediarias". Se ejecutan **en el medio** del ciclo de vida de una petición (después de que el servidor recibe la solicitud, pero antes de que llegue al controlador final). 
  * *Para qué sirven:* Validar datos, verificar si un usuario está logueado (autenticación), registrar logs de visitas, etc.
  * *Concepto fundamental:* Todo middleware recibe 3 parámetros: Petición (req), Respuesta (res), y la función `next()`. Si el middleware no finaliza la respuesta (ej. tirando un error), **debe** llamar obligatoriamente a `next()` para cederle el paso al siguiente middleware o controlador, de lo contrario la petición queda "colgada" para siempre.
  * *Ejemplo de sintaxis para papel:*
    ```js
    const logger = (req, res, next) => {
        console.log("Petición recibida");
        next(); // ¡Obligatorio!
    };
    ```

## 4. APIs, Arquitectura REST y CRUD
* **API (Application Programming Interface):** Es un puente de comunicación estandarizado. Permite que dos sistemas de software distintos se hablen entre sí (ej. una app móvil comunicándose con los servidores de un banco).
* **Arquitectura REST:** Es un conjunto de buenas prácticas y reglas para diseñar APIs utilizando el protocolo de internet (HTTP). Sus principios teóricos son:
  1. **Sin estado (Stateless):** El servidor no guarda memoria de peticiones anteriores. Cada petición debe traer toda la info necesaria para ser atendida.
  2. **URLs basadas en recursos:** Las URLs deben ser sustantivos en plural (ej. `/productos` y no `/conseguirProductos`).
  3. La acción a realizar se define estrictamente a través del **verbo HTTP**.
* **CRUD:** Es el acrónimo de las 4 operaciones básicas del manejo de datos, mapeadas a verbos HTTP:
  * **C**reate (Crear un recurso nuevo) -> `POST`
  * **R**ead (Leer o consultar) -> `GET`
  * **U**pdate (Actualizar) -> `PUT` (reemplazo total) o `PATCH` (modificación parcial)
  * **D**elete (Eliminar) -> `DELETE`

## 5. JSON y Postman
* **JSON (JavaScript Object Notation):** Es un formato de texto ultraligero utilizado para el intercambio de datos entre sistemas (Frontend y Backend). Aunque su sintaxis nace de JavaScript, es universal (lo entienden todos los lenguajes). Conceptualmente es una colección de pares clave-valor donde **las claves obligatoriamente van entre comillas dobles** (ej. `{"nombre": "Juan"}`).
* **Postman:** Es una herramienta de software que actúa como un cliente HTTP artificial. Teóricamente, sirve para probar y simular peticiones hacia nuestra propia API (permitiendo configurar manualmente los verbos, las URLs y el cuerpo JSON) sin necesidad de tener programada una interfaz gráfica o web real.

## 6. Middlewares de Validación y Manejo de Errores
* **Validación:** Conceptualmente, implica colocar un middleware específico para interceptar los datos sucios que envía el cliente. Si un dato es inválido (ej. falta la contraseña o un número es negativo), el middleware aborta la operación respondiendo con un código de error HTTP (ej. 400 Bad Request). Si los datos son válidos, invoca a `next()`.
  * *Ejemplo de validación rápida:*
    ```js
    const validarEdad = (req, res, next) => {
        if(req.body.edad < 18) return res.status(400).send("Error");
        next();
    };
    ```
* **Manejo de Errores (Error Handling):** Consiste en centralizar las caídas del sistema. Se logra creando un middleware especial al final de toda la aplicación (que recibe 4 parámetros en lugar de 3: `err, req, res, next`) cuyo único objetivo es "atrapar" cualquier falla de código imprevista y devolverle al usuario un mensaje de error elegante (ej. "Error 500: Fallo interno del servidor") en lugar de que el programa de Node.js se rompa y se apague por completo.
  * *Ejemplo de middleware de error para papel:*
    ```js
    const errorHandler = (err, req, res, next) => {
        console.error(err.message);
        res.status(500).send("Algo salió mal en el servidor");
    };
    ```

---

## 7. Conceptos de Bases de Datos y SQL (Resumen Teórico)
*(Si en la prueba escrita te piden definiciones de BD, estos son los conceptos pulidos)*

* **Estructura Relacional:** Todo se organiza en **Tablas** (que representan entidades como 'Usuarios'), compuestas por **Campos** (las columnas o atributos de esa tabla, ej. 'nombre') y **Registros** (cada una de las filas con la información concreta de un elemento).
* **DER (Diagrama Entidad-Relación):** Es el modelo conceptual y abstracto. Las entidades se grafican como rectángulos, los atributos como óvalos, y las relaciones como rombos. El punto crítico es definir la **Cardinalidad** (1:1, 1:N, N:M).
* **DLR (Diagrama Lógico Relacional):** Es la traducción del DER al diseño de bases de datos reales. Aquí aparecen las **Claves Primarias (PK)** para identificar registros de forma única, y las **Claves Foráneas (FK)** para vincular tablas. *Regla de oro escrita: Una relación Muchos a Muchos (N:M) se resuelve sí o sí creando una tercera tabla intermedia.*
* **Normalización:** Es el proceso de diseño para evitar la redundancia (datos repetidos) y anomalías. Implica que cada dato debe ser atómico (indivisible) y depender única y exclusivamente de la Clave Primaria de su tabla.
* **Lenguajes SQL (DDL vs DML):**
  * **DDL (Data Definition Language):** Comandos que modifican la *estructura* o el esqueleto de la base (ej. `CREATE TABLE`, `ALTER TABLE`).
  * **DML (Data Manipulation Language):** Comandos que interactúan con la *información* adentro de las tablas.
    * *Ejemplo de DML:* `SELECT nombre FROM usuarios WHERE edad > 18;`
* **JOINs y Agrupación:**
  * **INNER JOIN:** Operación de intersección. Trae únicamente los registros que tienen pareja/coincidencia en ambas tablas.
    * *Ejemplo:* `SELECT * FROM A INNER JOIN B ON A.id = B.a_id;`
  * **LEFT JOIN:** Operación de prioridad. Trae todos los registros de la tabla principal (izquierda) obligatoriamente, rellenando con `NULL` si no encontraron coincidencia en la segunda tabla.
  * **GROUP BY / HAVING:** `GROUP BY` condensa filas con valores idénticos en grupos de resumen (ideal para aplicar sumatorias o promedios). `HAVING` actúa como el filtro `WHERE`, pero diseñado exclusivamente para filtrar a esos grupos ya formados.
    * *Ejemplo:* `SELECT pais, SUM(ventas) FROM usuarios GROUP BY pais HAVING SUM(ventas) > 100;`
* **PostgreSQL (Teoría de diferencias con SQL Server):** Es un motor de BD relacional open-source. Se distingue teóricamente de alternativas comerciales por su soporte nativo avanzado para datos semi-estructurados (JSONB con indexación rápida), su capacidad de instalar extensiones de terceros, ser sensible a mayúsculas por defecto en las búsquedas, y usar su sistema de bloqueos MVCC (garantizando que las consultas de lectura jamás bloqueen a las de escritura).
