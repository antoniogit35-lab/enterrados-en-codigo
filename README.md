# Enterrados en código · Halloween Atlántida

Proyecto web de temática Halloween realizado por **Sergio** y **Antonio** (2.º DAW, Atlántida Formación Profesional) para el proyecto transversal DAWlloween. Es un cementerio que se recorre con el ratón: desde la verja de entrada se accede a un recetario, a un ranking de asesinos del cine, a un juego de deducción tipo Cluedo y a un formulario para escribirle a Jack.

## Demo en línea

Hay una versión alojada en [enterrados.byethost6.com](https://enterrados.byethost6.com). Es una adaptación de este proyecto para un hosting gratuito con PHP y MySQL; los pasos y las diferencias están en [Despliegue en Byet.host](docs/despliegue-byet.md).

## Qué incluye

| Página | Archivo | Descripción |
|---|---|---|
| Entrada | `index.php` | Verja del cementerio con animación al pulsar el cartel |
| Cementerio | `principal.php` | Portada con accesos a todas las secciones y estado de la sesión |
| Iniciar sesión | `login.php` | Lápida que muestra la foto, las fechas y un epitafio del usuario |
| Crear usuario | `registro.php` | Alta de usuario con elección de asesino |
| Cambiar contraseña | `recuperar.php` | Recuperación con pregunta de seguridad |
| Recetas malditas | `recetario.php` | Catálogo de 30 recetas con buscador y fichas |
| Top asesinos | `ranking.php` | Ranking por asesinatos; desde la ficha de cada asesino se puede llevar su arma junto al cursor |
| Asesinato en la mansión del código | `juego.php` | Juego de deducción estilo Cluedo contra el ordenador |
| Contacta con Jack | `contacto.php` | Envío de correo por SMTP (requiere haber iniciado sesión) |
| Error 404 | `404.php` | Página de error personalizada |

## Tecnologías

- PHP 8.1 o superior (se usa el tipo de retorno `never`) con la extensión `mysqli`.
- MySQL o MariaDB.
- HTML, CSS y JavaScript sin frameworks.
- [PHPMailer](https://github.com/PHPMailer/PHPMailer) para el correo. La carpeta `vendor/` ya viene incluida en el repositorio, así que **no hace falta Composer**.
- Apache mediante XAMPP para el entorno local.
- Se recomienda un navegador actualizado (Chrome, Edge, Firefox o Safari recientes).
- Extensiones de PHP necesarias: mysqli, mbstring y openssl (XAMPP las trae activadas).
- Tipografías Eater y Rubik Wet Paint (Google Fonts, licencia OFL), incluidas en `public/assets/fonts/`.

## Estructura del proyecto

```text
Halloween-Atlantida/
├── app/                        # Lógica de servidor, fuera de la raíz pública
│   ├── config.ejemplo.php      # Plantilla de configuración (se copia a config.php)
│   ├── conexion.php            # Conexión a la base de datos
│   ├── seguridad.php           # Escape, token CSRF y utilidades
│   ├── usuarios.php            # Alta, login, cambio de contraseña y datos de usuario
│   └── recetas.php             # Lectura del CSV de recetas
├── database/
│   ├── schema.sql              # Crea la base de datos y la tabla de usuarios
│   └── usuario-bd.sql          # Crea el usuario de aplicación con permisos mínimos
├── docs/                       # Documentación y material de diseño
├── public/                     # ÚNICA carpeta que Apache debe servir
│   ├── index.php, principal.php, login.php, registro.php, recuperar.php,
│   │   salir.php, ranking.php, recetario.php, juego.php, contacto.php, 404.php
│   ├── api/                    # Datos para la lápida del login
│   ├── data/                   # personajes.json, frases.json, cursores.json
│   └── assets/                 # css/, js/, img/, video/
├── recursos/datos/recetas.csv  # Fuente de datos del recetario
├── vendor/                     # PHPMailer (incluido)
└── composer.json
```

Solo `public/` se publica. `app/`, `database/`, `recursos/` y `vendor/` quedan fuera del alcance del servidor web.

## Instalación en local (XAMPP en Windows)

Los ejemplos suponen XAMPP en `C:\xampp`. Adapta las rutas si es otra.

### 1. Obtener el proyecto

```powershell
git clone https://github.com/antoniogit35-lab/Halloween-Atlantida.git
cd Halloween-Atlantida
```

### 2. Crear la base de datos

1. Arranca **Apache** y **MySQL** desde el panel de XAMPP (si MySQL no arranca, mira la sección *Problemas habituales*).
2. Abre `http://localhost/phpmyadmin`, entra en **Importar** y sube `database/schema.sql`. Se crean la base `halloween_atlantida` y la tabla `usuarios`.
3. Crea el usuario de aplicación: edita `database/usuario-bd.sql`, sustituye `CAMBIA_ESTA_CLAVE` por una contraseña y ejecútalo en la pestaña **SQL** de phpMyAdmin. Ese usuario solo puede hacer `SELECT`, `INSERT` y `UPDATE` sobre la tabla `usuarios`.

> Para una prueba rápida en un equipo propio también se puede usar `root` sin contraseña, que es el valor por defecto de XAMPP. No lo hagas en un servidor real.

### 3. Configurar la aplicación

Copia `app/config.ejemplo.php` como `app/config.php` y rellena los datos. Este archivo **no se sube al repositorio** (está en `.gitignore`).

```php
<?php
return [
    // Base de datos
    'host'    => '127.0.0.1',
    'usuario' => 'dawlloween',
    'clave'   => 'LA_CLAVE_QUE_HAS_ELEGIDO',
    'base'    => 'halloween_atlantida',

    // Correo de "Contacta con Jack" (opcional: sin esto el resto de la web funciona)
    'correo_destinatario'     => 'cuenta-que-recibe@ejemplo.com',
    'correo_remitente'        => 'cuenta-smtp@ejemplo.com',
    'correo_remitente_nombre' => 'Contacta con Jack',
    'smtp_host'               => 'smtp.mail.yahoo.com',
    'smtp_puerto'             => 587,
    'smtp_usuario'            => 'cuenta-smtp@ejemplo.com',
    'smtp_clave'              => 'CONTRASEÑA_DE_APLICACION',
];
```

Notas sobre el correo:

- El envío usa SMTP con STARTTLS. Con Yahoo o Gmail hay que crear una **contraseña de aplicación** en la cuenta y usarla en `smtp_clave`; la contraseña normal no funciona.
- El servidor SMTP debe corresponder al proveedor de la cuenta: `smtp.mail.yahoo.com` para Yahoo y `smtp.gmail.com` para Gmail.
- El formulario solo está disponible para usuarios con sesión iniciada y usa el email de su cuenta como remitente de respuesta.
- Si falta algún dato de correo, el formulario muestra un aviso de que no se ha podido enviar, y el resto de la web no se ve afectado.

### 4. Configurar Apache con un dominio local (`.test`)

Con el dominio local `halloween-atlantida.test` no hace falta HTTPS. El nombre puede cambiarse por cualquier otro que termine en `.test`, siempre que se use el mismo en los tres pasos siguientes.

**a) Virtual Host.** Comprueba que en `C:\xampp\apache\conf\httpd.conf` está activa, sin `#`, esta línea:

```apache
Include conf/extra/httpd-vhosts.conf
```

Después edita `C:\xampp\apache\conf\extra\httpd-vhosts.conf` y añade, sustituyendo `C:/ruta/Halloween-Atlantida` por la ubicación real del proyecto (con barras normales `/`, también en Windows):

```apache
# Conserva localhost apuntando a la página habitual de XAMPP
<VirtualHost *:80>
    ServerName localhost
    DocumentRoot "C:/xampp/htdocs"
    <Directory "C:/xampp/htdocs">
        Require local
    </Directory>
</VirtualHost>

<VirtualHost *:80>
    ServerName halloween-atlantida.test
    DocumentRoot "C:/ruta/Halloween-Atlantida/public"

    # Página 404 personalizada
    ErrorDocument 404 /404.php

    <Directory "C:/ruta/Halloween-Atlantida/public">
        Options -Indexes
        AllowOverride None
        Require local
    </Directory>
</VirtualHost>
```

**b) Archivo `hosts`.** Abre el Bloc de notas **como administrador**, abre `C:\Windows\System32\drivers\etc\hosts` (con el filtro «Todos los archivos») y añade al final:

```text
127.0.0.1 halloween-atlantida.test
```

Guarda y, en PowerShell, ejecuta `ipconfig /flushdns`.

**c) Validar y reiniciar Apache.**

```powershell
& "C:\xampp\apache\bin\httpd.exe" -t
```

Debe responder `Syntax OK`. Después pulsa **Stop** y **Start** en Apache.

### 5. Probar

Abre `http://halloween-atlantida.test/`. Debe aparecer la verja del cementerio; pulsa el cartel «Entrar al cementerio».

La configuración de Apache y el archivo `hosts` son locales a cada ordenador y no se suben al repositorio.

## Alternativa: dominio `halloween.dev` (requiere HTTPS)

Los dominios `.dev` están en la lista de precarga HSTS de los navegadores, que **obligan a usar HTTPS** al visitarlos. Por eso no basta con un Virtual Host en el puerto 80: hace falta un certificado local. La forma más sencilla es `mkcert`.

1. Instala `mkcert` (PowerShell): `winget install FiloSottile.mkcert`. Cierra y abre la terminal.
2. Crea la autoridad local de confianza: `mkcert -install` (acepta el aviso de Windows).
3. Genera el certificado dentro de la configuración de Apache:

```powershell
cd C:\xampp\apache\conf
mkcert -cert-file ssl.crt\halloween.dev.pem -key-file ssl.key\halloween.dev-key.pem halloween.dev
```

4. En el archivo `hosts`, añade `127.0.0.1 halloween.dev` y ejecuta `ipconfig /flushdns`.
5. En `httpd-vhosts.conf`, mantén el bloque de `localhost` de arriba y añade (con la ruta real del proyecto):

```apache
<VirtualHost *:80>
    ServerName halloween.dev
    Redirect permanent / https://halloween.dev/
</VirtualHost>

<VirtualHost *:443>
    ServerName halloween.dev
    DocumentRoot "C:/ruta/Halloween-Atlantida/public"

    SSLEngine on
    SSLCertificateFile "C:/xampp/apache/conf/ssl.crt/halloween.dev.pem"
    SSLCertificateKeyFile "C:/xampp/apache/conf/ssl.key/halloween.dev-key.pem"

    ErrorDocument 404 /404.php

    <Directory "C:/ruta/Halloween-Atlantida/public">
        Options -Indexes
        AllowOverride None
        Require local
    </Directory>
</VirtualHost>
```

6. Valida con `httpd.exe -t`, reinicia Apache y abre `https://halloween.dev/`.

Si Apache no arranca con SSL, comprueba que `mod_ssl` está cargado y que existe `Listen 443` en `httpd.conf` (XAMPP lo trae activado por defecto).

## Problemas habituales

**MySQL de XAMPP no arranca porque el puerto 3306 está ocupado** (por ejemplo, por otro MySQL instalado en el equipo). Hay dos soluciones:

- Usar ese otro MySQL: no arranques el de XAMPP, crea la base de datos en él y pon sus credenciales en `app/config.php`.
- Mover el MySQL de XAMPP al puerto 3307:
  1. En el panel de XAMPP: **Config** de MySQL → `my.ini`, y cambia `port=3306` a `port=3307` en las secciones `[client]` y `[mysqld]`.
  2. En `C:\xampp\php\php.ini`, cambia `mysqli.default_port=3306` a `3307`.
  3. En `C:\xampp\phpMyAdmin\config.inc.php`, añade `$cfg['Servers'][$i]['port'] = '3307';`.
  4. Pon `127.0.0.1` como `host` en `app/config.php` y reinicia Apache y MySQL.

**`Access denied for user` en la web.** La web está conectando a un MySQL distinto del que esperas, o las credenciales de `app/config.php` no coinciden. Si cambiaste el puerto de MySQL, revisa que `php.ini` se guardó y que se reinició Apache.

**Una dirección inexistente no muestra la página 404 personalizada.** Falta `ErrorDocument 404 /404.php` en el Virtual Host o no se reinició Apache. Con el servidor interno de PHP (`php -S`) esta directiva no se aplica; la página puede verse abriendo `/404.php` directamente.

**Las imágenes o estilos no cargan.** Comprueba que el `DocumentRoot` apunta a la carpeta `public` y no a la raíz del repositorio.

## Comprobaciones tras instalar

1. La entrada carga y el cartel inicia la animación hacia la portada.
2. Se puede crear un usuario, iniciar sesión y salir del cementerio con la confirmación.
3. En el login, al escribir un usuario existente aparecen su asesino, sus fechas y un epitafio.
4. El recetario muestra 30 recetas y el buscador filtra con y sin tildes.
5. En el ranking, la ficha de un asesino permite llevar su arma junto al cursor, y el cambio se mantiene al navegar.
6. El juego del Cluedo permite interrogar, tachar en el cuaderno y acusar, con pantalla final de victoria o derrota.
7. Escribir una dirección que no existe muestra la página del fantasma con el botón de vuelta.

## Datos editables

- **Recetas:** `recursos/datos/recetas.csv`. Columnas, en este orden: `nombre,foto,tiempo,dificultad,ingredientes,preparación,personas`. UTF-8. Las fotos van en `public/assets/img/recetas/`, y la columna `foto` lleva solo el nombre del archivo.
- **Asesinos del ranking, la lápida y el registro:** `public/data/personajes.json`.
- **Epitafios de la lápida:** `public/data/frases.json`.
- **Arma de cada asesino para el cursor:** `public/data/cursores.json`, con imágenes en `public/assets/img/cursores/`.
- **Datos del Cluedo (sospechosos, armas, habitaciones y reglas):** `public/assets/js/juego-datos.js`.

## Seguridad

- Las contraseñas y las respuestas de seguridad se guardan con `password_hash`.
- Los formularios usan token CSRF; cerrar sesión solo funciona con un envío POST válido.
- Las consultas a la base de datos son preparadas.
- El usuario de base de datos de la aplicación solo tiene permisos de lectura, inserción y actualización sobre `usuarios`.
- `app/config.php` contiene credenciales y está excluido del control de versiones.

## Control de versiones

Antes de cada commit, revisa los cambios y no subas credenciales ni configuraciones personales:

```powershell
git status
git diff
```

## Documentación

- [Guía de uso](docs/guia-de-uso.md): cómo usar la web, paso a paso.
- [Cómo funciona el código del Cluedo](docs/cluedo-codigo.md): explicación técnica del juego.
- [Despliegue en Byet.host](docs/despliegue-byet.md): cómo se publicó la demo en un hosting gratuito.

## Créditos

Sergio y Antonio · 2.º DAW · Atlántida Formación Profesional.
