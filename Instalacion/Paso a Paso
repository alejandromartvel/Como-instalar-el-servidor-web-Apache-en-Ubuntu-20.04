# 🌐 Guía de Instalación y Configuración de Apache en Ubuntu

Esta guía detalla los pasos necesarios para instalar el servidor web Apache, ajustar el firewall, administrar los servicios de red y configurar hosts virtuales para alojar múltiples dominios.

---

## 📌 Paso 1: Instalar Apache

Apache está disponible en los repositorios de software predeterminados de Ubuntu, lo que permite instalarlo con las herramientas convencionales de administración de paquetes.

1. Actualice el índice de paquetes locales para reflejar los últimos cambios:
   ```cmd
   sudo apt update
   ```

2. Instale el paquete `apache2`:
   ```cmd
   sudo apt install apache2
   ```

> ℹ️ Una vez confirmada la instalación, `apt` instalará Apache y todas las dependencias necesarias de forma automática.

---

## 🛡️ Paso 2: Ajustar el Firewall

Antes de probar Apache, es necesario modificar los ajustes del firewall para permitir el acceso externo a los puertos web predeterminados mediante `UFW`.

Durante la instalación, Apache se registra con UFW para proporcionar algunos perfiles de aplicación:

1. Enumere los perfiles de aplicación de `ufw` disponibles:
   ```cmd
   sudo ufw app list
   ```

   **Resultado esperado (Output):**
   ```text
   Available applications:
     Apache
     Apache Full
     Apache Secure
     OpenSSH
   ```

### Perfiles disponibles:
* **Apache:** Abre solo el puerto `80` (tráfico web normal no cifrado).
* **Apache Full:** Abre el puerto `80` y el puerto `443` (tráfico TLS/SSL cifrado).
* **Apache Secure:** Abre solo el puerto `443` (tráfico TLS/SSL cifrado).

Se recomienda habilitar el perfil más restrictivo. Como en esta guía aún no configuramos SSL, solo permitiremos el tráfico en el puerto `80`:

```cmd
sudo ufw allow 'Apache'
```

2. Verifique el cambio en el estado del firewall:
   ```cmd
   sudo ufw status
   ```

   **Resultado esperado (Output):**
   ```text
   Status: active

   To                         Action      From
   --                         ------      ----
   OpenSSH                    ALLOW       Anywhere                  
   Apache                     ALLOW       Anywhere                
   OpenSSH (v6)               ALLOW       Anywhere (v6)             
   Apache (v6)                ALLOW       Anywhere (v6)
   ```

---

## 🖥️ Paso 3: Comprobar su Servidor Web

Al final del proceso de instalación, Ubuntu inicia Apache automáticamente. El servidor web ya debería estar activo.

1. Realice una verificación con el sistema init `systemd` para saber si se encuentra en ejecución:
   ```cmd
   sudo systemctl status apache2
   ```

   **Resultado esperado (Output):**
   ```text
   ● apache2.service - The Apache HTTP Server
        Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
        Active: active (running) since Thu 2020-04-23 22:36:30 UTC; 20h ago
          Docs: https://apache.org
      Main PID: 29435 (apache2)
         Tasks: 55 (limit: 1137)
        Memory: 8.0M
        CGroup: /system.slice/apache2.service
                ├─29435 /usr/sbin/apache2 -k start
                ├─29437 /usr/sbin/apache2 -k start
                └─29438 /usr/sbin/apache2 -k start
   ```

2. Si no conoce la dirección IP pública de su servidor para probarlo en el navegador, puede obtenerla mediante comandos:
   ```cmd
   hostname -I
   ```
   O usando la herramienta `Icanhazip`:
   ```cmd
   curl -4 icanhazip.com
   ```

3. Introdúzcala en la barra de direcciones de su navegador web:
   ```text
   http://your_server_ip
   ```

> 📄 **Página predeterminada de Apache:** Debería ver la página de bienvenida de Apache en Ubuntu. Esta pantalla indica que funciona correctamente e incluye información básica sobre archivos y directorios importantes.

---

## ⚙️ Paso 4: Administrar el Proceso de Apache

Comandos básicos de administración con `systemctl`:

* **Detener** el servidor web:
  ```cmd
  sudo systemctl stop apache2
  ```
* **Iniciar** el servidor web:
  ```cmd
  sudo systemctl start apache2
  ```
* **Reiniciar** el servicio completo:
  ```cmd
  sudo systemctl restart apache2
  ```
* **Recargar** la configuración (sin cerrar conexiones activas):
  ```cmd
  sudo systemctl reload apache2
  ```
* **Deshabilitar** el inicio automático con el sistema:
  ```cmd
  sudo systemctl disable apache2
  ```
* **Habilitar** el inicio automático con el sistema:
  ```cmd
  sudo systemctl enable apache2
  ```

---

## 📂 Paso 5: Configurar Hosts Virtuales (Recomendado)

Los hosts virtuales permiten encapsular detalles de configuración y alojar más de un dominio desde un único servidor. Configuraremos el dominio ficticio `your_domain` (cámbielo por el suyo).

En vez de modificar el directorio por defecto `/var/www/html`, crearemos una estructura limpia:

1. Cree el directorio para `your_domain`:
   ```cmd
   sudo mkdir /var/www/your_domain
   ```

2. Asigne la propiedad del directorio a su usuario actual del sistema:
   ```cmd
   sudo chown -R USER:USER /var/www/your_domain
   ```

3. Asegúrese de que los permisos sean correctos (lectura, escritura y ejecución para el propietario; lectura y ejecución para otros):
   ```cmd
   sudo chmod -R 755 /var/www/your_domain
   ```

4. Cree una página web de ejemplo `index.html` utilizando el editor `nano`:
   ```cmd
   sudo nano /var/www/your_domain/index.html
   ```

5. Pegue el siguiente código HTML de prueba dentro del archivo, luego guarde y cierre:
   ```html
   <!-- Archivo: /var/www/your_domain/index.html -->
   <html>
       <head>
           <title>Welcome to Your_domain!</title>
       </head>
       <body>
           <h1>Success! The your_domain virtual host is working!</h1>
       </body>
   </html>
   ```

6. Cree el archivo de configuración para el nuevo Host Virtual:
   ```cmd
   sudo nano /etc/apache2/sites-available/your_domain.conf
   ```

7. Añada el siguiente bloque de configuración base adaptado al nuevo dominio y directorio:
   ```apache
   # Archivo: /etc/apache2/sites-available/your_domain.conf
   <VirtualHost *:80>
       ServerAdmin webmaster@localhost
       ServerName your_domain
       ServerAlias www.your_domain
       DocumentRoot /var/www/your_domain
       ErrorLog \${APACHE_LOG_DIR}/error.log
       CustomLog \${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```
