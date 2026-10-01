# 1. Qué es Expo?  
_Expo es un ecosistema y framework de código abierto para React Native que simplifica radicalmente el desarrollo de aplicaciones móviles nativas para iOS, Android y la web, permitiendo a los desarrolladores escribir un único código en JavaScript o TypeScript sin necesidad de configurar herramientas nativas complejas como Xcode o Android Studio_

# 2. Cómo usar reduce()?  
```js
const resultado = array.reduce((acumulador, elementoActual) => {
  return nuevoAcumulador;
}, valorInicial);
```

# 3. Cómo usar Axios?  
**api.js:**
```js
import axios from 'axios';
const api = axios.create({
  baseURL: 'https://api.tudominio.com/v1', 
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
    // 'Authorization': 'Bearer TU_TOKEN' (puedes agregarlo aquí si es estático)
  }
});
export default api;
```
**index.js:**
```js
import api from './api.js';

export const crearUsuario = async (datosUsuario) => {
  try {
    const response = await api.post('/users', datosUsuario);
    console.log('Usuario creado exitosamente:', response.data);
    return response.data;
  } catch (error) {
    // Axios guarda los detalles del error HTTP en error.response
    console.error('Error en el servidor:', error.response?.data);
    throw error;
  }
};
```

# 4. Comando para crear React Native  
**Con Expo:**
```bash
npx create-expo-app@latest NombreDeTuProyecto
```  
**CLI Nativa:**
```bash
npx react-native@latest init NombreDeTuProyecto
```

# 5. StyleSheet (diferencias con CSS)  
El motor de renderizado móvil (`Yoga Layout`) no es un navegador web. Escribir estilos como en la web romperá la aplicación. Las diferencias principales son:

* **Dirección Flex por defecto:** En `StyleSheet` es `flexDirection: 'column'` (en web es `row`). Todos los componentes son contenedores Flex por defecto.
* **Sin Unidades de Texto (`px`, `em`, `rem`):** Todos los tamaños numéricos deben ser **números puros** (puntos lógicos) o strings si usás porcentajes (`'50%'`).
* **Valores como Strings:** Todos los valores de texto deben ir obligatoriamente entre comillas (ej: `position: 'absolute'`).
* **Sin Shorthands de Bordes ni Fondos:** No podés usar `border: 1px solid red` ni `background`. Deben desglosarse en `borderWidth: 1`, `borderColor: 'red'`, `borderStyle: 'solid'` y `backgroundColor`.
* **Propiedades Inexistentes:** No existen `display: grid` / `block` (solo `'flex'` o `'none'`), `visibility: hidden`, `white-space`, ni `box-shadow`.
* **Sin Selectores ni Pseudo-clases:** No existen `:hover`, `:focus`, `.clase` ni `#id`. Toda la interacción se maneja mediante componentes como `<Pressable>` y estados de JavaScript.  
```jsx
<Pressable
      // style recibe una función con el estado del componente
      style={({ pressed }) => [
        styles.botonBase, // primero se aplica este (menos prioridad)
        pressed && styles.botonPresionado // (como :active)
      ]}
      onPress={() => console.log('¡Botón presionado!')}
    >/* (...) */</Pressable>
```

# 6. React Hook Form  
_Biblioteca para crear y gestionar formularios en React de forma rápida, eficiente y sin renderizados innecesarios_  
```jsx
import React from "react";
import { useForm } from "react-hook-form";

export default function App() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const onSubmit = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <label>Nombre</label>
      <input {...register("nombre", { required: "El nombre es obligatorio" })} />
      {errors.nombre && <p>{errors.nombre.message}</p>}

      <button type="submit">Enviar</button>
    </form>
  );
}
```

# 7. Context API  
_Herramienta nativa de React para compartir datos globales en toda la aplicación sin tener que pasar props manualmente por cada nivel (evita el prop drilling)._

**1. Crear el Contexto:**
```javascript
import { createContext } from 'react';

export const ThemeContext = createContext();
```

**2. Usar el Provider:**
```jsx
import { ThemeContext } from './ThemeContext';
import Boton from './Boton';

function App() {
  const temaActual = "oscuro";

  return (
    <ThemeContext.Provider value={temaActual}>
      <Boton />
    </ThemeContext.Provider>
  );
}
```

**3. Consumir con useContext:**
```jsx
import { useContext } from 'react';
import { ThemeContext } from './ThemeContext';

function Boton() {
  const tema = useContext(ThemeContext); 

  return (
    <button className={`btn-${tema}`}>
      El tema actual es: {tema}
    </button>
  );
}
```

# 8. Promesas  
```js
const miPromesa = new Promise((resolve, reject) => {
  let exito = true; // Simulación de una condición

  if (exito) {
    resolve("¡Operación exitosa!"); // Se cumple
  } else {
    reject("Hubo un error."); // Se rechaza
  }
});
```