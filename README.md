# API SENA

API REST sencilla en PHP y MySQL para registrar usuarios, iniciar sesión y administrar tareas.

## Requisitos

- XAMPP (Apache, PHP y MySQL/MariaDB).
- PHP con las extensiones PDO y `pdo_mysql` habilitadas.

## Instalación local

1. Copia o clona este proyecto en `C:\xampp\htdocs\api-sena`.
2. Inicia Apache y MySQL desde el panel de control de XAMPP.
3. En phpMyAdmin (`http://localhost/phpmyadmin`), crea la base de datos `api_sena`.
4. Ejecuta este SQL para crear las tablas que utiliza la API:

   ```sql
   CREATE DATABASE IF NOT EXISTS api_sena
     CHARACTER SET utf8mb4
     COLLATE utf8mb4_unicode_ci;

   USE api_sena;

   CREATE TABLE usuarios (
     id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
     nombre VARCHAR(100) NOT NULL,
     email VARCHAR(255) NOT NULL UNIQUE,
     password VARCHAR(255) NOT NULL
   );

   CREATE TABLE tareas (
     id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
     usuario_id INT UNSIGNED NOT NULL,
     titulo VARCHAR(255) NOT NULL,
     completado TINYINT(1) NOT NULL DEFAULT 0,
     CONSTRAINT fk_tareas_usuario
       FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
       ON DELETE CASCADE
   );
   ```

5. Si tus credenciales de MySQL son distintas a las predeterminadas de XAMPP, actualiza host, nombre de base de datos, usuario y contraseña en [`config/database.php`](config/database.php).
6. La API estará disponible en `http://localhost/api-sena`.

## Endpoints

Todas las solicitudes con datos deben enviar JSON con `Content-Type: application/json`.

### Registrar usuario

`POST /auth/register.php`

```json
{
  "nombre": "Ana",
  "email": "ana@example.com",
  "password": "mi-clave"
}
```

La contraseña se almacena como hash. Responde `201` cuando el registro es exitoso.

### Iniciar sesión

`POST /auth/login.php`

```json
{
  "email": "ana@example.com",
  "password": "mi-clave"
}
```

Responde con los datos públicos del usuario si las credenciales son válidas. Este endpoint no genera un token ni mantiene una sesión.

### Consultar tareas

`GET /api/tareas.php`

Devuelve la lista de tareas en formato JSON.

### Crear tarea

`POST /api/tareas.php`

```json
{
  "usuario_id": 1,
  "titulo": "Completar la actividad"
}
```

### Actualizar tarea

`PUT /api/tareas.php`

```json
{
  "id": 1,
  "titulo": "Completar la actividad",
  "completado": 1
}
```

Usa `completado: 1` para marcarla como terminada y `completado: 0` para dejarla pendiente.

### Eliminar tarea

`DELETE /api/tareas.php`

```json
{
  "id": 1
}
```

## Ejemplo con cURL

```bash
curl -X POST http://localhost/api-sena/auth/register.php \
  -H "Content-Type: application/json" \
  -d "{\"nombre\":\"Ana\",\"email\":\"ana@example.com\",\"password\":\"mi-clave\"}"
```

## Alcance actual

El inicio de sesión verifica las credenciales, pero no emite tokens. Las rutas de tareas no requieren autenticación y las consultas actuales no limitan los resultados por usuario. No expongas esta API públicamente sin implementar autenticación y autorización para las operaciones sobre tareas.
