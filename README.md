# ConexionArregladaSpring
Arreglo de conexión mediante configuración de CORS en Spring Boot

---

# Configuración de CORS en Spring Boot

Este repositorio demuestra cómo habilitar **Cross-Origin Resource Sharing (CORS)** en una aplicación **Spring Boot** que expone un endpoint `/greeting`. Además, muestra un ejemplo de frontend en HTML/JavaScript consumiendo dicho endpoint desde un dominio o puerto distinto.

## Descripción General

En aplicaciones web, **CORS** es el mecanismo que permite (o bloquea) que un recurso sea solicitado desde un origen distinto al que sirvió la página. El navegador, por seguridad, restringe tales peticiones a menos que el servidor especifique los encabezados de CORS apropiados.

Al intentar consumir el endpoint `http://localhost:8080/greeting` desde un archivo HTML en otro dominio/puerto (por ejemplo, `http://localhost:3000` o incluso `file://`), el navegador lanzará un error de tipo `CORS` si no están habilitadas las cabeceras adecuadas en el servidor.

## Estructura del Proyecto

```
├─ src
│  ├─ main
│  │  ├─ java
│  │  │  └─ com.example.restservice
│  │  │     ├─ GreetingController.java
│  │  │     ├─ Greeting.java
│  │  │     └─ [más clases si existen...]
│  │  └─ resources
│  │     └─ static
│  │        └─ index.html  <-- Ejemplo de frontend que consume /greeting
├─ pom.xml or build.gradle  <-- Configuración de Maven/Gradle
└─ README.md
```

## Pasos para Configurar CORS

1. **Habilitar CORS en el controlador**  
   En el archivo `GreetingController.java`, o donde se maneje la ruta `/greeting`, añadir la anotación `@CrossOrigin`. Por ejemplo:
   ```java
   @RestController
   @CrossOrigin(origins = "*") // o un dominio específico
   public class GreetingController {

       private static final String template = "Hello, %s!";
       private final AtomicLong counter = new AtomicLong();

       @GetMapping("/greeting")
       public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
           return new Greeting(counter.incrementAndGet(), String.format(template, name));
       }
   }
   ```
   Esto permite peticiones desde cualquier origen (`*`). Para entornos de producción, se recomienda especificar los dominios confiables, por ejemplo:
   ```java
   @CrossOrigin(origins = "https://mi-dominio.com")
   ```
2. **Ejecutar la aplicación**  
   - Si usas Maven:
     ```bash
     mvn spring-boot:run
     ```
   Esto levantará el servicio en `http://localhost:8080`.

3. **Consumir el endpoint desde un frontend**  
   Se puede colocar un archivo HTML con la siguiente lógica (ejemplo simplificado):
   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <meta charset="UTF-8" />
     <title>Greeting App</title>
   </head>
   <body>
     <h1>Ingrese su nombre</h1>
     <input type="text" id="nameInput" />
     <button onclick="getGreeting()">Saludar</button>
     <div id="displayContent"></div>

     <script>
       async function getGreeting() {
         const name = document.getElementById('nameInput').value;
         try {
           const response = await fetch(`http://localhost:8080/greeting?name=${encodeURIComponent(name)}`);
           const data = await response.json();
           document.getElementById('displayContent').textContent = data.content;
         } catch (error) {
           console.error('Error de CORS u otro:', error);
         }
       }
     </script>
   </body>
   </html>
   ```

4. **Verificación**  
   Abrir el HTML en tu navegador (ya sea con un server local o directamente `file://`). Tras introducir un nombre y dar clic en el botón, debería ver en la consola o en el `div` el saludo devuelto por el backend. Si hay un problema de CORS, asegúrese de que el servidor responda con los encabezados `Access-Control-Allow-Origin`.

## Solución de Problemas

- **Error: `No 'Access-Control-Allow-Origin' header is present`**  
  Esto significa que el navegador bloqueó la petición. Asegurarse de:
  1. Haber añadido `@CrossOrigin` en tu controlador (o configurado CORS globalmente).
  2. Ejecutar la aplicación y acceder a la URL correcta (`http://localhost:8080`).


## Conclusiones

CORS es un mecanismo de seguridad fundamental en el desarrollo web moderno. Al configurar el servidor para permitir únicamente los orígenes requeridos, se evita el acceso no autorizado desde otros sitios, asegurando la integridad de la aplicación. Con **Spring Boot**, habilitar CORS es bastante sencillo mediante la anotación `@CrossOrigin` o la configuración global con `WebMvcConfigurer`.
