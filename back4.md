# Guía Técnica y Práctica de Backend (Node.js & Express)

A diferencia de la guía teórica, este resumen está enfocado netamente en el **código, la implementación y la sintaxis**. Aquí vas a encontrar *cómo* se utilizan técnicamente las herramientas y cómo interactúan los archivos en un proyecto real.

## 1. Inicialización y Módulos en Node.js
Para arrancar un proyecto en Node.js, siempre se debe crear un archivo `package.json` que es el que administra las librerías y scripts del proyecto.
* **Comando inicial:** Ejecutar `npm init -y` en la terminal (crea el package.json por defecto).
* **Instalación de paquetes:** `npm install express` (lo agrega a las dependencias) o `npm install nodemon -D` (dependencia de desarrollo para que el server se reinicie solo).
* **Scripts:** En el `package.json` se configuran comandos rápidos:
  ```json
  "scripts": {
    "start": "node src/app.js",
    "dev": "nodemon src/app.js"
  }
  ```
  *(Luego se corren en la terminal con `npm run dev`)*.

### Importaciones (CommonJS)
La forma clásica de requerir dependencias locales en Node es usando `require` y `module.exports`.
```js
// En archivo: src/helpers.js
const saludar = (nombre) => `Hola ${nombre}`;
const despedir = () => `Chau`;

module.exports = { saludar, despedir };

// En archivo: src/app.js
const helpers = require('./helpers'); 
console.log(helpers.saludar("Dami")); // Imprime "Hola Dami"
```

## 2. Express y los objetos Req/Res
Toda ruta en Express recibe dos objetos vitales: `req` (Request / Petición del cliente) y `res` (Response / Respuesta del servidor).

### Desglosando el `req` (Lo que nos llega del Frontend/Postman)
* **`req.body`:** Contiene la información enviada en el "cuerpo" de la petición (típico en formularios y métodos POST/PUT). *Aclaración crucial: Express no entiende JSON por defecto, para que `req.body` no sea `undefined`, se debe poner `app.use(express.json())` al principio del proyecto.*
* **`req.params`:** Lee variables dinámicas directamente en la URL, definidas con dos puntos `:`. Ej: Si la ruta es `/usuarios/:id`, y llaman a `/usuarios/5`, lo leemos con `req.params.id` (vale "5").
* **`req.query`:** Lee los parámetros extra que van después del `?` en la URL. Ej: `/usuarios?rol=admin&edad=20`. Se leen con `req.query.rol` y `req.query.edad`.

### Desglosando el `res` (Lo que devolvemos)
* **`res.send()`:** Envía texto plano o HTML.
* **`res.json()`:** Envía la respuesta formateada como un objeto JSON (es el estándar absoluto al crear APIs REST).
* **`res.status(codigo)`:** Modifica el código de estado HTTP. Por defecto, Express devuelve 200 (OK). Ej: `res.status(404).json({error: "Usuario no encontrado"})`.

---

## 3. Implementación de Arquitectura MVC (Folder Based)
Un proyecto escalable nunca tiene todo suelto en `app.js`. Se debe modularizar. Así se conectan los archivos técnicamente:

**1. Archivo de Rutas (`src/routes/userRoutes.js`)**: Sólo redirigen al controlador (no tienen lógica).
```js
const express = require('express');
const router = express.Router(); // Módulo Router de Express
const userController = require('../controllers/userController');

router.get('/', userController.getAllUsers); // Importante: Va SIN los paréntesis finales
router.post('/', userController.createUser);

module.exports = router;
```

**2. Controlador (`src/controllers/userController.js`)**: Contiene la inteligencia de la app.
```js
const getAllUsers = (req, res) => {
    // Acá iría la query SQL al Modelo para traer los datos
    res.status(200).json({ mensaje: "Lista de usuarios enviada" });
};

const createUser = (req, res) => {
    const { nombre, edad } = req.body; // Leemos lo que nos manda el front
    res.status(201).json({ mensaje: `Usuario ${nombre} insertado en BD` });
};

module.exports = { getAllUsers, createUser };
```

**3. Archivo Principal (`src/app.js`)**: Une el rompecabezas.
```js
const express = require('express');
const userRoutes = require('./routes/userRoutes');

const app = express();
app.use(express.json()); // Clave para leer JSON

// Le decimos a la app que todas las peticiones a /api/users vayan al router de usuarios
app.use('/api/users', userRoutes); 

app.listen(3000, () => console.log('Servidor corriendo en puerto 3000'));
```

---

## 4. Uso Técnico de Postman
Al crear un endpoint `POST /api/users`, no podemos probarlo escribiendo la URL en el navegador (el buscador del navegador solo dispara peticiones `GET`). Por eso usamos Postman:
1. Cambiamos el select del verbo a **POST**.
2. Ponemos la URL (`http://localhost:3000/api/users`).
3. Vamos a la pestaña **Body** (debajo de la URL).
4. Seleccionamos el botón **raw** y cambiamos el dropdown final a **JSON**.
5. Escribimos el JSON a mano con comillas dobles:
   ```json
   {
       "nombre": "Damián",
       "edad": 25
   }
   ```
6. Al hacer click en *Send*, Express recibe esto dentro de `req.body`.

---

## 5. Implementación Práctica de Middlewares
Los middlewares no solo atajan errores, a veces se usan para **inyectar información** a la petición antes de que llegue al controlador:

```js
// Archivo: src/middlewares/auth.js
const verificarAuth = (req, res, next) => {
    const token = req.headers.authorization;
    if (!token) {
        return res.status(401).json({ error: "No tenés permiso" }); // Corta la cadena acá
    }
    
    // Si está todo bien, le "INYECTAMOS" un dato al req para que viaje hasta el controlador
    req.usuarioLogueado = "Damián"; 
    next(); // Pasa el control
};
```
Se utiliza inyectándolo literalmente en la ruta:
```js
router.get('/perfil', verificarAuth, (req, res) => {
    // El controlador ahora puede leer la info que le pasó el middleware en el req!
    res.json({ mensaje: `Bienvenido al sistema, ${req.usuarioLogueado}` }); 
});
```

---

## 6. Middlewares de Validación
Para no llenar el controlador de bloques `if/else`, se valida estrictamente en un middleware previo. 

**Ejemplo de validación manual de datos de un formulario:**
```js
const validarUsuario = (req, res, next) => {
    const { email, password } = req.body;
    
    if (!email || !email.includes('@')) {
        return res.status(400).json({ error: "El email provisto es inválido o falta" });
    }
    if (!password || password.length < 6) {
        return res.status(400).json({ error: "La contraseña debe tener al menos 6 caracteres" });
    }
    
    next(); // Solo si sobrevive a los filtros, ejecutamos next()
};

// Se enchufa en la ruta antes del controlador:
router.post('/registro', validarUsuario, userController.crearUsuario);
```

---

## 7. Manejo Centralizado de Errores (Error Handling)
Cuando un controlador asíncrono (por ejemplo, una conexión a BD) falla o tira una excepción, usamos un bloque `try/catch`. En el `catch`, **le pasamos el error a la función `next(error)`**.

```js
// src/controllers/userController.js
const buscarUsuario = async (req, res, next) => {
    try {
        const usuario = await BaseDeDatos.buscar(req.params.id); // Si esto falla...
        res.json(usuario);
    } catch (error) {
        // En vez de responder acá mismo, se lo "pateamos" al Error Handler
        next(error); 
    }
};
```

Para que esto funcione, el middleware de errores se debe colocar **SIEMPRE al final de todo el archivo `app.js`**, debajo de todas las rutas:

```js
// src/app.js
app.use('/api/users', userRoutes);
app.use('/api/products', productRoutes);

// Middleware Final de Manejo de Errores (Se identifica porque recibe 4 parámetros)
app.use((err, req, res, next) => {
    console.error("Hubo un error fatal:", err.message); // Queda en la consola del server
    
    // Se devuelve un JSON amigable para que el Frontend sepa qué falló
    res.status(500).json({
        estado: "Error crítico del servidor",
        detalle: err.message
    });
});
```
