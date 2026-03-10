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

**4. Controlador Web (`app/Http/Controllers/RestaurantController.php`)**
Toma los datos del modelo y los envía a una vista HTML.
```php
namespace App\Http\Controllers;

use App\Models\Restaurant;
use Illuminate\Http\Request;

class RestaurantController extends Controller
{
    public function index()
    {
        $restaurantes = Restaurant::all();
        // Devuelve una vista de Blade (HTML)
        return view('restaurants.index', compact('restaurantes')); 
    }
}
```

**5. Ruta Web (`routes/web.php`)**
Define la URL para acceder a la página web.
```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\RestaurantController;

Route::get('/restaurantes', [RestaurantController::class, 'index']);
```

---

## 3. Transformación a un Flujo de API (Usando Endpoints)

En las aplicaciones modernas (apps móviles o frontends separados en React/Vue), el backend no devuelve HTML, sino datos puros. La Base de Datos, la Migración y el Modelo **se mantienen exactamente igual**. Solo cambiamos el Controlador y las Rutas.



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
Ya no hay vista Blade. El frontend (React, Vue, App Móvil o JS puro) consume el endpoint haciendo una petición HTTP por debajo:

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
