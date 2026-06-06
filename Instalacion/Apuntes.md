# 📚 Apuntes Completos: Examen de HTTP / Apache

Este repositorio contiene la guía paso a paso y la documentación necesaria para la configuración de DNS (Bind9) y Servidores Web (Apache2) basada en escenarios de exámenes prácticos.

---

## 🔹 Datos de Configuración Asignados
* **Estudiante:** Alejandro Martinez Velasco
* **Número asignado:** 15


| Concepto | Valor |
| :--- | :--- |
| **Dominio principal** | `tesla15.cat` |
| **Subdominio IT** | `it.tesla15.cat` |
| **Subdominio Vendes** | `vendes.tesla15.cat` |
| **Subdominio Taller** | `taller.tesla15.cat` |
| **IP del servidor (ServerRA3)** | *Asignada en examen* (Ej: `10.14.15.2`) |
| **Cliente Vendes** | `vendes15` |
| **Cliente Taller** | `taller15` |

---

## 🎯 Resumen de Requisitos del Examen


| Apartado | Descripción de la Tarea | Puntuación |
| :---: | :--- | :---: |
| **1** | Configurar DNS para los 3 subdominios | 0.25 pts |
| **2** | Crear carpetas, mover archivos HTML y personalizar contenidos | 0.25 pts |
| **3** | Configurar Apache con 3 VirtualHosts independientes | 3.50 pts |
| **4** | Validar el funcionamiento de las 3 páginas web | 1.00 pts |
| **5** | Implementar el control de acceso por IP | 3.00 pts |
| **6** | Pruebas de conectividad y acceso desde el cliente Vendes | 1.00 pts |
| **7** | Pruebas de conectividad y acceso desde el cliente Taller | 1.00 pts |

---

## 📝 Procedimiento Paso a Paso

### 🔹 Parte 1: Configuración DNS (Bind9)
**Objetivo:** Añadir 3 registros tipo `A` en la zona directa para que los subdominios apunten a la IP del servidor web.

* **Archivo a editar:** `/etc/bind/zonas/db.tesla15.cat`

Añade las siguientes líneas al final del archivo (respetando la estructura existente):

```text
; Registros para los subdominios web
it      IN      A       IP_DEL_SERVIDOR
vendes  IN      A       IP_DEL_SERVIDOR
taller  IN      A       IP_DEL_SERVIDOR
```

#### 📖 Explicación de los parámetros:

| Línea / Parámetro | Significado | Acción requerida |
| :--- | :--- | :--- |
| `; Registros...` | Comentario informativo | Opcional, no afecta a la sintaxis |
| `it` / `vendes` / `taller` | Nombre del subdominio a crear | Modificar si el enunciado cambia |
| `IN` | Clase Internet | **No cambiar** |
| `A` | Tipo de registro (Mapeo Nombre $\rightarrow$ IP) | **No cambiar** |
| `IP_DEL_SERVIDOR` | Dirección IPv4 del servidor destino | **Reemplazar por la IP asignada** |

#### 🔴 Ejemplo de configuración real (Caso IP: `10.14.15.2`):
```text
; Registros para los subdominios web
it      IN      A       10.14.15.2
vendes  IN      A       10.14.15.2
taller  IN      A       10.14.15.2
```

#### Comandos de validación:
```bash
# Reiniciar el servicio DNS para aplicar cambios
sudo systemctl restart bind9

# Probar la resolución de nombres localmente
nslookup it.tesla15.cat
nslookup vendes.tesla15.cat
nslookup taller.tesla15.cat
```

> 📸 **Capturas requeridas para entrega:**
> * `cat /etc/bind/zonas/db.tesla15.cat` (Verificando los 3 registros A).
> * `nslookup it.tesla15.cat` (Verificando la resolución correcta a la IP).

---

### 🔹 Parte 2: Estructura de Directorios y Archivos
**Objetivo:** Crear el entorno de almacenamiento para cada departamento, organizar los recursos web y personalizar el sitio.

```bash
# PASO 1: Crear la estructura de directorios
sudo mkdir -p /var/www/html/it
sudo mkdir -p /var/www/html/vendes
sudo mkdir -p /var/www/html/taller

# PASO 2: Organizar los archivos HTML en sus respectivas rutas
sudo mv /var/www/html/it.html /var/www/html/it/
sudo mv /var/www/html/vendes.html /var/www/html/vendes/
sudo mv /var/www/html/taller.html /var/www/html/taller/

# PASO 3: Desplazar imágenes asociadas (silenciando errores si no existen)
sudo mv /var/www/html/it.jpg /var/www/html/it/ 2>/dev/null
sudo mv /var/www/html/vendes.jpg /var/www/html/vendes/ 2>/dev/null
sudo mv /var/www/html/taller.jpg /var/www/html/taller/ 2>/dev/null

# PASO 4: Modificar el contenido de las páginas para la autoría
sudo nano /var/www/html/it/it.html
sudo nano /var/www/html/vendes/vendes.html
sudo nano /var/www/html/taller/taller.html
```

#### 📖 Explicación de los comandos utilizados:
* `mkdir -p`: Crea directorios. La bandera `-p` genera las carpetas padre automáticamente de ser necesario.
* `mv [origen] [destino]`: Mueve o renombra archivos y directorios.
* `2>/dev/null`: Redirige el canal de errores (*stderr*) al dispositivo nulo para evitar mensajes si el archivo no existe.
* `nano`: Editor de texto en terminal.

#### Ejemplo de personalización HTML:
Dentro de cada archivo `.html`, asegúrate de incluir tus datos personales en una sección visible:
```html
<h1>Departament IT</h1>
<p>Administrador: Alejandro Martinez Velasco</p>
```

> 📸 **Captura requerida para entrega:**
> * `ls -la /var/www/html/it/` (Validando los permisos y archivos contenidos por carpeta).

---

### 🔹 Parte 3: Configuración de Servidor Web (Apache2 VirtualHosts)
**Objetivo:** Definir bloques de configuración independientes para que Apache asocie cada subdominio con su respectiva ruta de archivos.

#### Paso 3.1: VirtualHost para IT
* **Comando:** `sudo nano /etc/apache2/sites-available/it.tesla15.cat.conf`
* **Contenido:**
```apache
<VirtualHost *:80>
    ServerName it.tesla15.cat
    DocumentRoot /var/www/html/it
    
    <Directory /var/www/html/it>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/error_it.log
    CustomLog ${APACHE_LOG_DIR}/access_it.log combined
</VirtualHost>
```

#### Paso 3.2: VirtualHost para VENDES
* **Comando:** `sudo nano /etc/apache2/sites-available/vendes.tesla15.cat.conf`
* **Contenido:**
```apache
<VirtualHost *:80>
    ServerName vendes.tesla15.cat
    DocumentRoot /var/www/html/vendes
    
    <Directory /var/www/html/vendes>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/error_vendes.log
    CustomLog ${APACHE_LOG_DIR}/access_vendes.log combined
</VirtualHost>
```

#### Paso 3.3: VirtualHost para TALLER
* **Comando:** `sudo nano /etc/apache2/sites-available/taller.tesla15.cat.conf`
* **Contenido:**
```apache
<VirtualHost *:80>
    ServerName taller.tesla15.cat
    DocumentRoot /var/www/html/taller
    
    <Directory /var/www/html/taller>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/error_taller.log
    CustomLog ${APACHE_LOG_DIR}/access_taller.log combined
</VirtualHost>
```

#### 📖 Desglose de directivas del VirtualHost:

| Directiva | Propósito | ¿Se debe modificar? |
| :--- | :--- | :--- |
| `<VirtualHost *:80>` | Escucha peticiones en cualquier IP de la máquina por el puerto 80 (HTTP). | No cambiar |
| `ServerName` | Dominio o subdominio asignado al sitio. | Ajustar por cada VirtualHost |
| `DocumentRoot` | Ruta absoluta del sistema de archivos donde vive la web. | Ajustar según el directorio |
| `<Directory>` | Abre el contenedor de directivas de seguridad para la ruta específica. | Debe coincidir con `DocumentRoot` |
| `Options Indexes ...` | Permite el listado de archivos si no hay un `index.html`. | No cambiar |
| `AllowOverride None` | Ignora las directivas de archivos locales `.htaccess`. | No cambiar |
| `Require all granted` | Otorga acceso sin restricciones a cualquier petición de origen. | *Se modificará en la Parte 5* |
| `ErrorLog` / `CustomLog` | Rutas personalizadas de auditoría y registros de errores. | No cambiar |

#### Paso 3.4: Activación de sitios y reinicio del servicio
```bash
# Activar las nuevas configuraciones de sitio
sudo a2ensite it.tesla15.cat.conf
sudo a2ensite vendes.tesla15.cat.conf
sudo a2ensite taller.tesla15.cat.conf

# Desactivar el sitio predeterminado de Apache para evitar conflictos
sudo a2dissite 000-default.conf

# Validar de forma integral la sintaxis del archivo de configuración
sudo apache2ctl configtest

# Aplicar los cambios reiniciando el demonio Web
sudo systemctl restart apache2
```

> 💡 **Nota de resolución de problemas:**
> * Si `configtest` devuelve `Syntax OK`, la configuración estructural es correcta.
> * Si muestra el error `could not resolve name`, revisa la configuración del archivo DNS o la falta de conectividad.

> 📸 **Captura requerida para entrega:**
> * `cat /etc/apache2/sites-available/it.tesla15.cat.conf` (O muestra de los ficheros de configuración creados).

---

### 🔹 Parte 4: Verificación de Disponibilidad Web
**Objetivo:** Comprobar que los tres sitios responden de manera autónoma y correcta.

#### Comprobación local mediante consola:
```bash
curl http://it.tesla15.cat
curl http://vendes.tesla15.cat
curl http://taller.tesla15.cat
```

#### Comprobación desde entorno gráfico (Navegador):
Introduce individualmente los subdominios en la barra de direcciones:
* `http://it.tesla15.cat`
* `http://vendes.tesla15.cat`
* `http://taller.tesla15.cat`

> 📸 **Capturas requeridas para entrega:**
> * Tres capturas de pantalla del navegador web donde se visualice claramente la URL en la barra de direcciones y tu nombre impreso en el cuerpo del HTML.

---

### 🔹 Parte 5: Control de Acceso por Direcciones IP

#### 📖 Requisitos de Mensajes de Error (Personalizados):
Cuando un equipo no tenga privilegios para acceder al sitio, se debe mostrar un aviso descriptivo:
* **IT:** `"Lloc disponible nomes per equips de IT"`
* **Vendes:** `"Lloc disponible per equips de VENDES i IT"`
