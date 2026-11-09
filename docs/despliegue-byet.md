# Despliegue en Byet.host

Este documento explica cómo se publicó la demo en línea, disponible en https://enterrados.byethost6.com. La entrega evaluada es la del repositorio original (`Halloween-Atlantida`); este repositorio es una copia adaptada para portfolio.

## Por qué Byet.host

- Vercel no incluye PHP entre sus runtimes oficiales, así que no era una opción directa para esta web.
- Byet.host ofrece hosting gratuito con PHP, MySQL, FTP y SSL. En la cuenta usada, el servidor tiene PHP 8.4 con las extensiones `mysqli`, `mbstring` y `openssl`.

## Diferencias con el entorno local

| Aspecto | Local (XAMPP) | Byet.host |
|---|---|---|
| Raíz pública | `public/` como `DocumentRoot` | `htdocs/`; PHP solo puede leer dentro de esa carpeta (`open_basedir`) |
| Código del servidor | `app/`, `vendor/` y `recursos/` fuera de la raíz pública | Dentro de `htdocs/`, ocultos con una reescritura en `.htaccess` |
| Base de datos | `schema.sql` crea la base y `usuario-bd.sql` crea un usuario con permisos mínimos | La base se crea desde el panel y se usa el único usuario que da el hosting, con todos los permisos sobre ella |
| Dirección | `halloween-atlantida.test` | Subdominio gratuito con HTTPS |
| Correo | SMTP con Yahoo o Gmail | Gmail (`smtp.gmail.com`, puerto 587, STARTTLS) con contraseña de aplicación |

## Estructura en el servidor

```text
htdocs/
├── .htaccess       # Copia de despliegue/byet/.htaccess
├── public/         # Contenido de public/ de este repositorio
├── app/            # Incluye config.php (creado a mano, no está en Git)
├── vendor/
└── recursos/
```

No se suben `database/`, `docs/`, `.git` ni `README.md`.

## El archivo .htaccess

`despliegue/byet/.htaccess` se copia a `htdocs/.htaccess`. Hace dos cosas:

- Reescribe todas las peticiones hacia `public/`. Así, cualquier dirección que empiece por `/app/`, `/vendor/` o `/recursos/` acaba en `public/app/…`, que no existe, y devuelve 404.
- Define `ErrorDocument 404 /404.php`. En local esta directiva está en el Virtual Host, pero en Byet.host no hay acceso a él.

## Base de datos

1. Crear la base de datos en **MySQL Databases** del panel. El hosting añade al nombre un prefijo con el usuario de la cuenta.
2. Abrir phpMyAdmin, **seleccionar esa base** en el panel izquierdo y ejecutar en la pestaña SQL el contenido de `database/schema.sql` **sin** las dos primeras instrucciones (`CREATE DATABASE` y `USE`), ya que en hosting compartido la base no se crea por SQL:

```sql
CREATE TABLE IF NOT EXISTS usuarios (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    usuario VARCHAR(30) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    perfil ENUM('user', 'admin') NOT NULL DEFAULT 'user',
    pregunta VARCHAR(150) NOT NULL,
    respuesta_hash VARCHAR(255) NOT NULL,
    asesino VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE NOT NULL,
    fecha_alta TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

No se usa `usuario-bd.sql`: el plan gratuito no permite crear usuarios adicionales ni asignar permisos por usuario.

## Configuración

Crear `app/config.php` a partir de `app/config.ejemplo.php` con los datos que muestra el panel de MySQL (host, usuario, nombre de la base y contraseña). Este archivo está en `.gitignore` y no debe subirse a Git.

Para el correo se usa una cuenta de Gmail dedicada a la práctica:

- `smtp_host`: `smtp.gmail.com`, `smtp_puerto`: `587`.
- `smtp_usuario` y `correo_remitente`: la dirección completa de Gmail.
- `smtp_clave`: una **contraseña de aplicación** de 16 caracteres, sin espacios. Requiere tener activada la verificación en dos pasos.

## Subida de archivos

Se sube por FTP (por ejemplo con FileZilla) al servidor que indica el panel del hosting. Orden recomendado: `app/`, `vendor/` y `recursos/` primero, y `public/` al final por ser la más pesada. El `.htaccess` va en la raíz de `htdocs/`.

## Comprobaciones

1. La raíz carga la verja del cementerio.
2. Una dirección inexistente (por ejemplo `/contacta.php`) muestra la página 404 personalizada.
3. `/app/config.php` y `/vendor/autoload.php` devuelven 404.
4. Funcionan registro, inicio de sesión, recuperación de contraseña, recetario, ranking, Cluedo y envío de correo.

## Limitaciones

- Hay un único usuario de base de datos con todos los permisos sobre la base, en lugar del usuario con permisos mínimos del diseño original.
- Es un hosting gratuito sin garantías de disponibilidad; los hostings gratuitos pueden suspender cuentas inactivas.
- Según la información de Byet.host, el límite de subida por archivo es de unos 10 MB. El archivo más pesado del proyecto (`chucky-victoria.gif`) ocupa 8,88 MiB.
