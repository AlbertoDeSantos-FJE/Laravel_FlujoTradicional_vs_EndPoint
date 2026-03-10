# Laravel_FlujoTradicional_vs_EndPoint

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
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=elcucharon
DB_USERNAME=root
DB_PASSWORD=
