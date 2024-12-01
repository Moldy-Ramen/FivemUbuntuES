
# Guía de Instalación de FiveM Debian
Una guía personalizada para configurar un servidor FiveM en una máquina virtual basada en Debian.

## Comenzando
### Requisitos previos
1. **Archivo ISO de Ubuntu Server**
   - Descarga el archivo ISO [aquí](https://ubuntu.com/download/server).

2. **Software de Virtualización**
   - Elige una de las siguientes opciones:
     - [Guía de instalación de VMware](https://www.youtube.com/watch?v=PoNPBdKLZdk)
     - [Guía de instalación de Hyper-V](https://www.youtube.com/watch?v=FCIA4YQHx9U)
     - [Guía de instalación de VirtualBox](https://www.youtube.com/watch?v=8mns5yqMfZk)

3. **Instancia de VM**
   - Usa las guías proporcionadas para crear una instancia de VM con la ISO de Ubuntu Linux.

---

## Instalación de Ubuntu
1. **Selección de idioma**  
   Usa las teclas de flecha para seleccionar tu idioma y presiona **Enter**.
2. **Actualizar el instalador**  
   Selecciona "Actualizar el instalador" y espera a que el proceso se complete.
3. **Diseño del teclado**  
   Elige el diseño de teclado que prefieras.
4. **Configuración de red**  
   Toma nota de la dirección IP de la máquina; la necesitarás más adelante.
5. **Configuración del proxy**  
   Omite este paso seleccionando **Hecho**.
6. **Espejo de Ubuntu**  
   - Si estás en los EE. UU., omite este paso y selecciona **Hecho**.  
   - Si estás en otro país, encuentra tu espejo local [aquí](https://launchpad.net/ubuntu/+archivemirrors).
7. **Configuración de almacenamiento**  
   - Para usar todo el disco, selecciona **Hecho**.
   - Para particiones personalizadas, configúralas según sea necesario.
8. **Resumen de particiones**  
   Confirma seleccionando **Hecho**.
9. **Confirmación de partición**  
   Selecciona **Continuar** para particionar el disco e instalar Ubuntu.
10. **Configuración de perfil**  
    Completa el formulario con tu nombre, nombre del servidor y credenciales de usuario.
11. **Configuración de SSH**  
    - Selecciona OpenSSH para instalarlo.  
    - Para servidores de producción, considera importar una clave SSH.
12. **Paquetes adicionales del servidor**  
    Omite seleccionando **Hecho**.
13. **Finalizar instalación**  
    Después de la instalación, selecciona **Reiniciar**.

---

## Configuración de FiveM
1. **Inicia sesión en tu servidor**  
   Usa un cliente SSH (por ejemplo, [Putty](https://www.putty.org/)) para conectarte:
   - Ingresa la dirección IP de tu servidor.
   - Inicia sesión con las credenciales creadas anteriormente.
2. **Actualizar servidor**  
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
3. **Instalar paquetes requeridos**  
   ```bash
   sudo apt install git nano xz-utils -y
   ```
4. **Instalación y configuración de MariaDB**  
   - Instala MariaDB:
     ```bash
     sudo apt install mariadb-server -y
     ```
   - Inicia el servicio MariaDB (solo es necesario una vez, ya que está habilitado por defecto en el inicio):
     ```bash
     sudo systemctl start mariadb.service
     ```
   - Asegura MariaDB ejecutando:
     ```bash
     sudo mysql_secure_installation
     ```
     Sigue estos pasos:
     1. Ingresa la contraseña de root de MariaDB.
     2. En el mensaje **¿Cambiar a la autenticación de Unix Socket?**, escribe `n` y presiona **Enter**.
     3. En el mensaje **¿Cambiar la contraseña de root?**, escribe `n` y presiona **Enter**.
     4. En el mensaje **¿Eliminar usuarios anónimos?**, escribe `Y` y presiona **Enter**.
     5. En el mensaje **¿Deshabilitar el inicio de sesión remoto de root?**, escribe `Y` y presiona **Enter**.  
        *(Esto es crucial para la seguridad.)*
     6. En el mensaje **¿Eliminar la base de datos de prueba?**, escribe `Y` y presiona **Enter** (opcional, pero recomendado).
     7. En el mensaje **¿Recargar las tablas de privilegios?**, escribe `Y` y presiona **Enter**.
   - Crea un usuario y una base de datos para el servidor FiveM:
     1. Abre la línea de comandos de MySQL:
        ```bash
        sudo mysql
        ```
     2. Crea un usuario con una contraseña segura (reemplaza `password123!` con tu propia contraseña segura):
        ```bash
        create user 'fivem'@'%' identified by 'password123!';
        ```
     3. Otorga todos los privilegios al usuario `fivem`:
        ```bash
        grant all privileges on *.* to 'fivem'@'%';
        ```
     4. Actualiza los cambios de privilegios:
        ```bash
        flush privileges;
        ```
     5. Sal de la línea de comandos de MySQL:
        ```bash
        exit
        ```
     6. Reinicia MariaDB para aplicar los cambios:
        ```bash
        sudo systemctl restart mariadb
        ```

5. **Configuración de FXServer**  
   - Crea y entra al directorio del servidor:
     ```bash
     mkdir -p ~/FXServer/server
     cd ~/FXServer/server
     ```
   - Descarga la última versión del servidor:
     ```bash
     wget <server_download_url>
     tar xf fx.tar.xz
     rm fx.tar.xz
     ```
   - Obtén una clave de licencia del servidor [aquí](https://keymaster.fivem.net).

---

## Iniciar tu servidor
### Usando `screen`:
1. Instala `screen`:
   ```bash
   sudo apt install screen -y
   ```
2. Ejecuta el servidor:
   ```bash
   screen -S fivem
   cd ~/FXServer/server
   bash run.sh
   ```

### Configuración de txAdmin:
1. Abre `http://<your_vm_ip>:40120` en tu navegador.
2. Vincula tu cuenta de CFX.re usando el PIN proporcionado.
3. Completa la configuración:
   - Contraseña de respaldo.
   - Nombre del servidor.
   - Selección de receta y plantilla.
   - Ubicación del servidor:
     ```bash
     /home/<your_username>/FXServer/server-data
     ```
   - Agrega tu clave de licencia del servidor y credenciales de base de datos.

---

## Opcional: Configurar un servicio de Linux
1. Crea un archivo de servicio:
   ```bash
   sudo nano /etc/systemd/system/fivem.service
   ```
2. Copia y pega lo siguiente:
   ```ini
   [Unit]
   Description=FiveM Server Service

   [Service]
   Type=simple
   ExecStart=/home/<your_username>/FXServer/server/run.sh
   Restart=always

   [Install]
   WantedBy=default.target
   ```
3. Habilita e inicia el servicio:
   ```bash
   sudo systemctl enable fivem.service
   sudo systemctl start fivem.service
   ```

---

## ¡Felicidades!
Tu servidor está ahora en línea y listo para usarse.
