# Guía Práctica: Entendiendo los Endpoints en Laravel

Esta guía explica qué es un endpoint y cómo implementarlo en un proyecto de Laravel utilizando el ejemplo de una aplicación de reserva de restaurantes tipo "The Fork" (nuestro proyecto ficticio: "El Cucharón").



---

## 1. ¿Qué es un endpoint?

Un **endpoint** (punto final o punto de acceso) es una **URL específica de un servidor** a través de la cual una aplicación cliente (navegador web, app móvil, frontend en React/Vue) se comunica con el servidor (backend) a través de una API.

**Analogía del banco:**
Imagina que el servidor es un banco. El **endpoint** es la ventanilla específica a la que te acercas para hacer un trámite:
* Ventanilla A (`GET /api/saldo`): Exclusiva para consultar saldo.
* Ventanilla B (`POST /api/ingreso`): Exclusiva para ingresar dinero.

Cada endpoint requiere:
1.  **Una URL:** La dirección exacta (ej. `tusitio.com/api/restaurantes`).
2.  **Un método HTTP:** La acción (`GET` para leer, `POST` para crear, `PUT`/`PATCH` para actualizar, `DELETE` para borrar).

> **Aclaración importante:** El endpoint NO es el controlador. El endpoint es el "letrero de la ventanilla" (URL + Método HTTP). El **controlador** es el empleado del banco que está detrás de la ventanilla ejecutando la acción.

---

## 2. Flujo Tradicional Web (HTML/Blade) vs API (JSON)

Antes de crear un endpoint real de API, veamos cómo se haría en una **Aplicación Web Tradicional** usando el patrón MVC (Modelo-Vista-Controlador) en Laravel con WAMP y MySQL. 



### Archivos Clave (Flujo Web Tradicional)

**1. Base de Datos (`.env`)**
Aquí configuramos la conexión a nuestro MySQL local (WAMP).
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=elcucharon
DB_USERNAME=root
DB_PASSWORD=
```

**2. Migración (`database/migrations/..._create_restaurants_table.php`)**
Define la estructura de nuestra tabla en la base de datos.
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('restaurants', function (Blueprint $table) {
            $table->id();
            $table->string('nombre');
            $table->string('ciudad');
            $table->string('tipo_comida');
            $table->decimal('puntuacion', 2, 1);
            $table->string('imagen_url')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('restaurants');
    }
};
```

**3. Modelo (`app/Models/Restaurant.php`)**
La representación en código de nuestra tabla.
```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Restaurant extends Model
{
    use HasFactory;
    protected $fillable = ['nombre', 'ciudad', 'tipo_comida', 'puntuacion', 'imagen_url'];
}
```

**4. Ruta Web (`routes/web.php`)**
Define la URL para acceder a la página web. Es la puerta de entrada de la petición.
```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\RestaurantController;

// Cuando el usuario entra a la URL, Laravel lo envía al Controlador
Route::get('/restaurantes', [RestaurantController::class, 'index']);
```

**5. Controlador Web (`app/Http/Controllers/RestaurantController.php`)**
Recibe la petición de la ruta, toma los datos del modelo y los envía a la vista HTML final.
```php
namespace App\Http\Controllers;

use App\Models\Restaurant;
use Illuminate\Http\Request;

class RestaurantController extends Controller
{
    public function index()
    {
        $restaurantes = Restaurant::all();
        // Le pasamos la variable $restaurantes a la vista
        return view('restaurants.index', compact('restaurantes')); 
    }
}
```

**6. Vista Final (`resources/views/restaurants/index.blade.php`)**
La interfaz (HTML + Blade) que se pinta en el navegador del usuario utilizando los datos que le envió el controlador.
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>El Cucharón - Restaurantes</title>
</head>
<body>
    <h1>Restaurantes Disponibles</h1>

    @if($restaurantes->isEmpty())
        <p>Aún no hay restaurantes registrados.</p>
    @else
        <ul>
        @foreach($restaurantes as $restaurante)
            <li>
                <strong>{{ $restaurante->nombre }}</strong> ({{ $restaurante->ciudad }}) 
                - Especialidad: {{ $restaurante->tipo_comida }} 
                - Puntuación: {{ $restaurante->puntuacion }}/5
            </li>
        @endforeach
        </ul>
    @endif
</body>
</html>
```

---

## 3. Transformación a un Flujo de API (Usando Endpoints)

En las aplicaciones modernas (apps móviles o frontends separados en React/Vue), el backend no devuelve HTML, sino datos puros. La Base de Datos, la Migración y el Modelo **se mantienen exactamente igual**. Solo cambiamos el Controlador y las Rutas, y la Vista desaparece del backend.



### Paso 1: El Controlador Modificado (Devuelve JSON)
En lugar de llamar a una vista de Blade, empaquetamos los datos en formato JSON.

```php
// app/Http/Controllers/RestaurantController.php

namespace App\Http\Controllers;

use App\Models\Restaurant;
use Illuminate\Http\Request;

class RestaurantController extends Controller
{
    public function index()
    {
        // 1. Obtenemos los datos de MySQL
        $restaurantes = Restaurant::all();

        // 2. Devolvemos JSON (El idioma universal de las APIs)
        return response()->json([
            'success' => true,
            'mensaje' => 'Lista de restaurantes obtenida correctamente',
            'cantidad' => $restaurantes->count(),
            'data' => $restaurantes
        ], 200);
    }
}
```

### Paso 2: La Ruta en `api.php`
Movemos nuestra ruta del archivo `web.php` al archivo `api.php`. Laravel añadirá automáticamente el prefijo `/api` a la URL.

```php
// routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\RestaurantController;

// Endpoint: GET /api/restaurantes
Route::get('/restaurantes', [RestaurantController::class, 'index']);
```

### Paso 3: Consumo desde el Frontend (El cliente)
Ya no hay vista Blade en Laravel. El frontend (React, Vue, App Móvil o JS puro) consume el endpoint haciendo una petición HTTP por debajo:

```javascript
// Ejemplo de consumo en JavaScript (Frontend / Cliente)

fetch('http://localhost/tu-proyecto/public/api/restaurantes')
    .then(respuesta => respuesta.json())
    .then(datos => {
        if(datos.success) {
            console.log("¡Hemos recibido " + datos.cantidad + " restaurantes!");
            console.log(datos.data); 
            
            // Aquí iría la lógica en JS para dibujar las tarjetas en el DOM
        }
    })
    .catch(error => console.error("Error al conectar con el endpoint:", error));
```

---

## 4. Resumen del Flujo API
1. El cliente hace una petición al **Endpoint** (`GET /api/restaurantes`).
2. El enrutador de Laravel (`api.php`) le pasa el trabajo al **Controlador**.
3. El Controlador pide los datos a la Base de Datos a través del **Modelo**.
4. El Controlador devuelve una respuesta empaquetada en **JSON**.
5. El cliente recibe el JSON y pinta la interfaz para el usuario final.

## 5. ¿Por qué mover la ruta a `api.php` en lugar de dejarla en `web.php`?

Es una duda muy común: si dejas la ruta en `web.php` y tu controlador devuelve un `response()->json()`, al abrir la URL en el navegador verás el JSON perfectamente (en una petición `GET`). 

Sin embargo, a nivel técnico, dejar un endpoint de API en `web.php` es una mala práctica. La razón principal no es la URL en sí, sino los **Middleware** (los filtros de seguridad y configuración por los que pasa la petición antes de llegar al controlador).

Estas son las tres diferencias y afectaciones principales al no moverlo:

### 1. El problema del CSRF Token (El famoso Error 419)
El archivo `web.php` está diseñado para páginas web tradicionales y tiene activada por defecto la protección **CSRF** (Cross-Site Request Forgery). 
* Si intentas hacer una petición `POST`, `PUT` o `DELETE` (por ejemplo, enviar un formulario desde React para guardar una reserva) hacia una ruta en `web.php`, **Laravel bloqueará la petición y devolverá un Error 419 (Page Expired)** porque exigirá un token de seguridad oculto que las APIs no utilizan.
* En `api.php`, esta protección está desactivada porque las APIs emplean otros métodos de seguridad (como tokens de autenticación Bearer).

### 2. Memoria y Sesiones (Con Estado vs Sin Estado)
* **`web.php` tiene "Estado" (Stateful):** Cada vez que una petición entra por `web.php`, Laravel arranca el motor de sesiones, lee y escribe cookies, y guarda el estado del usuario en el servidor. Esto consume memoria y recursos.
* **`api.php` es "Sin Estado" (Stateless):** Las APIs no usan sesiones ni cookies. No recuerdan al usuario entre una petición y otra. Esto hace que tu servidor sea muchísimo más rápido, ligero y capaz de escalar para aguantar miles de peticiones de apps móviles sin colapsar.

### 3. Límite de Peticiones (Rate Limiting)
* Las rutas en `api.php` vienen protegidas por defecto con un middleware llamado `throttle:api`. Esto limita la cantidad de peticiones que una misma IP puede hacer por minuto (usualmente 60). 
* Esto es crucial en una API pública para evitar ataques de fuerza bruta (DDoS) o que saturen tu base de datos. `web.php` no aplica esta restricción tan estricta por defecto.

### Resumen comparativo: `web.php` vs `api.php`

| Característica | `web.php` | `api.php` |
| :--- | :--- | :--- |
| **Uso ideal** | Vistas Blade, páginas web tradicionales. | Apps móviles, React, Vue, sistemas externos. |
| **Formato de respuesta** | HTML (generalmente) | JSON (puros datos) |
| **Manejo de usuarios** | Sesiones y Cookies | Tokens (ej. Laravel Sanctum o JWT) |
| **Protección CSRF** | Activada (Bloquea POST sin token) | Desactivada |
| **Prefijo automático** | Ninguno (ej. `tusitio.com/ruta`) | `/api` (ej. `tusitio.com/api/ruta`) |

## 6. Arquitecturas: ¿Monolítico, Separado o Híbrido?

Una vez que entiendes la diferencia entre devolver vistas HTML (`web.php`) o datos JSON (`api.php`), surge la gran pregunta: ¿Debo usar solo endpoints y olvidar las vistas de Laravel? La respuesta no es absoluta, depende enteramente de las necesidades técnicas y comerciales de tu proyecto.

Existen tres enfoques principales a la hora de estructurar tu aplicación:

### 1. Modelo Tradicional "Monolítico" (Solo Blade)
Laravel se encarga de todo de principio a fin: conecta a la base de datos, procesa la lógica en el controlador y dibuja el HTML completo utilizando vistas Blade. No hay endpoints de API (el archivo `api.php` prácticamente no se usa).
* **¿Cuándo usarlo?** Es ideal para webs corporativas, blogs, o plataformas donde el SEO (posicionamiento en Google) es crítico y la interactividad en la pantalla es moderada. Es la forma más rápida, económica y sencilla de lanzar un proyecto en solitario.

### 2. Modelo 100% Separado (Backend API + Frontend Independiente)
Laravel funciona **exclusivamente como backend**. Su único trabajo es procesar datos y escupir JSON a través de endpoints en `api.php`. Las vistas Blade desaparecen del servidor. El frontend se construye en un repositorio totalmente independiente usando React, Next.js, Vue, o es una App Móvil nativa.
* **¿Cuándo usarlo?** Es la opción obligatoria si en tu hoja de ruta está lanzar una aplicación móvil (iOS/Android) además de la web. De esta forma, tanto la web moderna como la app consumen exactamente la misma API, evitando tener que programar la lógica de negocio dos veces.

### 3. El Modelo Híbrido (El punto intermedio estratégico)
En muchísimos proyectos reales, se mezclan ambos mundos para obtener las ventajas de los dos enfoques.
* **¿Cómo funciona?** Utilizas el enrutamiento web clásico (`web.php`) y vistas de **Blade** para la estructura principal y pública de la web (Landing page, Quiénes somos, Contacto) asegurando una carga inicial rapidísima y un SEO perfecto. Sin embargo, para las secciones interactivas (ej. un mapa en tiempo real, un buscador dinámico de restaurantes o un panel de usuario complejo), incrustas JavaScript o componentes de Vue/React que se comunican "por debajo" con tus propios endpoints internos (`api.php`).

> **💡 El estándar actual en Laravel:** Hoy en día, para evitar la complejidad de gestionar un frontend separado o programar endpoints manuales para un modelo híbrido, el ecosistema de Laravel utiliza herramientas oficiales como **Livewire** (escribes en PHP/Blade pero se siente como React) o **Inertia.js** (escribes en React/Vue pero te saltas la creación de la API).

## 7. Ejemplo Práctico: Modelo Híbrido (Buscador en Vivo)

En este ejemplo, combinaremos la carga tradicional de una vista Blade con el consumo dinámico de un endpoint interno mediante JavaScript. El objetivo es crear un buscador que muestre resultados instantáneos mientras el usuario escribe, sin recargar la página.

### Paso 1: El Endpoint de Búsqueda (`routes/api.php`)
Primero, creamos una nueva ruta en nuestra API que recibirá el texto a buscar.

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\RestaurantController;

// Endpoint para la búsqueda en vivo: GET /api/restaurantes/buscar?q=texto
Route::get('/restaurantes/buscar', [RestaurantController::class, 'search']);
```

### Paso 2: El Controlador (`app/Http/Controllers/RestaurantController.php`)
Añadimos el método `search` que filtrará la base de datos y devolverá un JSON.

```php
namespace App\Http\Controllers;

use App\Models\Restaurant;
use Illuminate\Http\Request;

class RestaurantController extends Controller
{
    public function search(Request $request)
    {
        // 1. Capturamos lo que el usuario ha escrito (ej. ?q=pizza)
        $query = $request->input('q');

        // 2. Buscamos en la base de datos coincidencias en el nombre o especialidad
        $restaurantes = Restaurant::where('nombre', 'LIKE', "%{$query}%")
                                  ->orWhere('tipo_comida', 'LIKE', "%{$query}%")
                                  ->get();

        // 3. Devolvemos los resultados en formato JSON
        return response()->json([
            'success' => true,
            'data' => $restaurantes
        ]);
    }
}
```

### Paso 3: La Vista Híbrida (`resources/views/restaurants/buscador.blade.php`)
Aquí está la magia. Tenemos una vista de Blade estándar con un campo de texto (`<input>`). Usaremos un pequeño script de JavaScript que "escuche" cada vez que el usuario teclea algo, llame al endpoint y redibuje los resultados.

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Buscador Híbrido - El Cucharón</title>
    <style>
        body { font-family: sans-serif; padding: 20px; }
        #resultados { margin-top: 20px; border-top: 2px solid #eee; padding-top: 10px; }
        .tarjeta { padding: 10px; border-bottom: 1px solid #ddd; }
    </style>
</head>
<body>

    <h1>Encuentra tu restaurante</h1>
    
    <input type="text" id="cajaBusqueda" placeholder="Ej. Sushi, Pizzería, Madrid..." style="width: 300px; padding: 10px;">

    <div id="resultados">
        <p>Escribe algo para empezar a buscar...</p>
    </div>

    <script>
        const inputBusqueda = document.getElementById('cajaBusqueda');
        const divResultados = document.getElementById('resultados');

        // Escuchamos cada vez que se levanta una tecla en el input
        inputBusqueda.addEventListener('keyup', function() {
            let texto = this.value;

            // Si el input está vacío, limpiamos la pantalla
            if (texto.length === 0) {
                divResultados.innerHTML = '<p>Escribe algo para empezar a buscar...</p>';
                return;
            }

            // Llamamos a nuestro propio endpoint de Laravel
            fetch(`/api/restaurantes/buscar?q=${texto}`)
                .then(respuesta => respuesta.json())
                .then(datos => {
                    // Limpiamos los resultados anteriores
                    divResultados.innerHTML = ''; 

                    if (datos.data.length === 0) {
                        divResultados.innerHTML = '<p>No se encontraron restaurantes.</p>';
                        return;
                    }

                    // Recorremos el JSON y creamos el HTML para cada restaurante
                    datos.data.forEach(restaurante => {
                        divResultados.innerHTML += `
                            <div class="tarjeta">
                                <h3>${restaurante.nombre}</h3>
                                <p>${restaurante.tipo_comida} - ${restaurante.ciudad} (⭐ ${restaurante.puntuacion})</p>
                            </div>
                        `;
                    });
                })
                .catch(error => console.error('Error:', error));
        });
    </script>
</body>
</html>
```

### ¿Por qué esto es poderoso?
* **SEO Intacto:** Puedes cargar los 10 mejores restaurantes directamente desde el Controlador Web (`web.php`) cuando la página carga por primera vez.
* **Experiencia de Usuario (UX):** El usuario no tiene que pulsar un botón de "Buscar" y esperar a que la página parpadee y se recargue por completo. La sensación es la de estar usando una App móvil nativa.
