# Aplicación PHP con Docker en local (Servidor Web + Servidor de Aplicaciones)

## 🧠 Idea del proyecto

El objetivo de este proyecto es crear una **aplicación web sencilla** utilizando:

- **HTML, CSS y JavaScript** para la parte cliente
- **PHP** para la lógica del servidor
- **Docker y Docker Compose** para separar correctamente los servicios

La aplicación se ha diseñado siguiendo una **arquitectura real de producción**, separando:

- Un **servidor web** (Nginx)
- Un **servidor de aplicaciones** (PHP-FPM)
- Un **sistema de archivos compartido** (equivalente a NFS / EFS en AWS)

Aunque el despliegue se realiza en local, el diseño simula cómo funcionaría en un entorno cloud como **AWS**.

---

## 🏗️ Arquitectura del proyecto

La aplicación está dividida en **dos contenedores Docker**:

### 1️⃣ Servidor Web (Nginx)

- Atiende las peticiones del navegador
- Sirve HTML, CSS y JavaScript
- Redirige las peticiones PHP al servidor de aplicaciones
- Escucha en el **puerto 80** dentro del contenedor (8080 en el host)

### 2️⃣ Servidor de Aplicaciones (PHP-FPM)

- Ejecuta el código PHP
- Escucha en el **puerto 9000**
- No es accesible directamente desde el navegador

### 📂 Sistema de archivos compartido (NFS / EFS simulado)

- Ambos contenedores comparten la carpeta `app/`
- Esto simula un **NFS** o un **EFS de AWS**, donde el código está centralizado

---

## 📁 Estructura del proyecto

```
mi-app-php/
│
├── docker-compose.yml
│
├── web/
│   ├── Dockerfile
│   └── default.conf
│
├── php/
│   ├── Dockerfile
│   └── php.ini
│
└── app/
    ├── index.php
    ├── procesar.php
    ├── css/
    │   └── style.css
    └── js/
        └── script.js
```

---

## ⚙️ Funcionamiento de la aplicación

1. El usuario accede a la aplicación desde el navegador
2. Nginx recibe la petición
3. Si es un archivo estático (HTML, CSS, JS), lo sirve directamente
4. Si es un archivo PHP:

   - Nginx envía la petición a PHP-FPM por el **puerto 9000**
   - PHP-FPM ejecuta el script
   - Devuelve el resultado a Nginx
   - Nginx responde al navegador

📌 El navegador **nunca se comunica directamente con PHP-FPM**.

---

## 🧪 Aplicación de ejemplo

La aplicación consiste en:

- Una página principal (`index.php`)
- Un formulario que pide el nombre del usuario
- Un script PHP (`procesar.php`) que procesa el formulario y muestra el resultado

Es una aplicación sencilla, pero suficiente para demostrar la arquitectura completa.

---

## 🚨 Problemas encontrados y soluciones

### ❌ Error 1: Puerto 80 ocupado

**Problema:**

```
failed to bind host port 0.0.0.0:80: address already in use
```

**Causa:**

- El puerto 80 del sistema ya estaba siendo utilizado por otro servicio

**Solución:**

- Se cambió el mapeo de puertos a:

```
8080:80
```

---

### ❌ Error 2: 403 Forbidden al acceder a index.php

**Mensaje de error:**

```
/var/www/html/index.php is forbidden (13: Permission denied)
```

**Causa inicial sospechada:**

- Permisos de archivos Linux

Se intentaron soluciones clásicas:

- `chmod`
- Cambiar el usuario del contenedor

❌ Ninguna funcionó.

---

### ❌ Causa real del problema: SELinux (Fedora)

El sistema operativo utilizado es **Fedora**, que tiene **SELinux activado por defecto**.

SELinux bloquea el acceso de los contenedores Docker a carpetas del host, **aunque los permisos Linux sean correctos**.

Este comportamiento es normal en sistemas basados en RHEL (Fedora, CentOS, Red Hat).

---

### ✅ Solución definitiva

Se añadió la opción `:z` al volumen compartido:

```
./app:/var/www/html:z
```

Esto permite a Docker:

- Ajustar automáticamente el contexto de seguridad SELinux
- Autorizar el acceso de los contenedores a la carpeta

📌 Esta solución es equivalente a configurar permisos y políticas de acceso en un **EFS de AWS**.

---

## ☁️ Relación con AWS (NFS / EFS)

Aunque no se ha usado AWS directamente, el proyecto simula su funcionamiento:

| AWS         | Proyecto                   |
| ----------- | -------------------------- |
| EC2 Web     | Contenedor Nginx           |
| EC2 PHP-FPM | Contenedor PHP             |
| EFS (NFS)   | Volumen Docker compartido  |
| IP privada  | Nombre del servicio Docker |
| Puerto 9000 | Puerto 9000                |

Esto permite entender cómo funcionaría la aplicación en un entorno real de cloud.

---

## ▶️ Ejecución del proyecto

Desde la carpeta raíz:

```
docker-compose up --build
```

Acceso desde el navegador:

```
http://localhost:8080
```

---

## ✍️ Cristina Moreno

Proyecto realizado como ejercicio práctico de despliegue de aplicaciones web con Docker y PHP.
