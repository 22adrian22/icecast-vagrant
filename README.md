# **Servidor de Streaming de Audio con Icecast2 e Ices2** 🎧

Este proyecto documenta la configuración de un servidor de streaming de audio utilizando **Icecast2** y **Ices2** en una máquina virtual Vagrant con Debian Bookworm. Además, se incluyen instrucciones detalladas para obtener y preparar archivos de audio en formato `.ogg` para la lista de reproducción. La configuración puede realizarse de dos formas: **manualmente** o **automáticamente** mediante la ejecución de un playbook de Ansible (`playbook.yml`).

---

## **Tabla de Contenidos** 📑

1. Requisitos Previos
2. Configuración de la Máquina Virtual Vagrant
3. Instalación y Configuración de Icecast2
4. Instalación y Configuración de Ices2
5. Preparación de Archivos de Audio
6. Automatización del Servicio Ices2
7. Prueba del Servidor de Streaming
8. Automatización con Playbook de Ansible

---

## **Requisitos Previos** ⚙️

Antes de comenzar, asegúrate de tener instalados los siguientes componentes en tu máquina real:

- **Vagrant**: [Descargar e instalar Vagrant](https://www.vagrantup.com/downloads).
- **VirtualBox**: [Descargar e instalar VirtualBox](https://www.virtualbox.org/wiki/Downloads).
- **Reproductor VLC**: Para probar el streaming. [Descargar VLC](https://www.videolan.org/vlc/).
- **Ansible**: Para ejecutar el playbook. [Instalar Ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html).

---

## **Configuración de la Máquina Virtual Vagrant** 🖥️

1. **Crear un directorio para el proyecto**:
   ```bash
   mkdir vagrant-icecast
   cd vagrant-icecast
   ```

2. **Inicializar el entorno Vagrant**:
   ```bash
   vagrant init debian/bookworm64
   ```

3. **Configurar el archivo `Vagrantfile`**:
   Edita el archivo `Vagrantfile` para configurar la red y asignar recursos a la máquina virtual:
   ```ruby
   Vagrant.configure("2") do |config|
     config.vm.box = "debian/bookworm64"
     config.vm.network "forwarded_port", guest: 8000, host: 8000
     config.vm.provider "virtualbox" do |vb|
       vb.memory = "1024"
     end
   end
   ```

4. **Iniciar la máquina virtual**:
   ```bash
   vagrant up
   ```

5. **Acceder a la máquina virtual**:
   ```bash
   vagrant ssh
   ```

---

## **Automatización con Playbook de Ansible** 🤖

El playbook de Ansible (`playbook.yml`) automatiza todas las tareas necesarias para configurar el servidor de streaming. A continuación, se describe cada tarea realizada por el playbook:

### **Tareas del Playbook**

1. **Actualizar el sistema**:
   - Actualiza los paquetes del sistema para asegurar que todo esté al día.
   ```yaml
   - name: Actualizar el sistema
     apt:
       update_cache: yes
       upgrade: dist
     tags: update
   ```

2. **Instalar Icecast2**:
   - Instala Icecast2 y maneja la pregunta de configuración de clave.
   ```yaml
   - name: Instalar Icecast2
     debconf:
       name: icecast2
       question: icecast2/configure
       value: "false"
       vtype: boolean
     tags: icecast2

   - name: Instalar Icecast2 (paquete)
     apt:
       name: icecast2
       state: present
     tags: icecast2
   ```

3. **Configurar Icecast2**:
   - Copia los archivos de configuración de Icecast2 (`icecast.xml` y `icecast2`) desde la máquina local a la máquina virtual.
   ```yaml
   - name: Copiar icecast.xml
     copy:
       src: /vagrant/icecast.xml
       dest: /etc/icecast2/icecast.xml
       owner: root
       group: root
       mode: '0644'
       force: yes
       remote_src: yes
     tags: icecast2

   - name: Copiar icecast2 (default)
     copy:
       src: /vagrant/icecast2
       dest: /etc/default/icecast2
       owner: root
       group: root
       mode: '0644'
       force: yes
       remote_src: yes
     tags: icecast2
   ```

4. **Iniciar y habilitar el servicio Icecast2**:
   - Inicia y habilita el servicio Icecast2 para que se ejecute automáticamente al iniciar el sistema.
   ```yaml
   - name: Iniciar servicio Icecast2
     systemd:
       name: icecast2
       state: started
       enabled: yes
     tags: icecast2
   ```

5. **Instalar Ices2 y Vorbis Tools**:
   - Instala Ices2 y las herramientas necesarias para convertir archivos de audio.
   ```yaml
   - name: Instalar Ices2 y Vorbis Tools
     apt:
       name:
         - ices2
         - vorbis-tools
       state: present
     tags: ices2
   ```

6. **Configurar Ices2**:
   - Crea el directorio de configuración de Ices2 y copia el archivo de configuración `ices-playlist.xml`.
   ```yaml
   - name: Crear directorio /etc/ices2
     file:
       path: /etc/ices2
       state: directory
       owner: root
       group: root
       mode: '0755'
     tags: ices2

   - name: Copiar ices-playlist.xml
     copy:
       src: /vagrant/ices-playlist.xml
       dest: /etc/ices2/ices-playlist.xml
       owner: root
       group: root
       mode: '0644'
       force: yes
       remote_src: yes
     tags: ices2
   ```

7. **Preparar archivos de audio**:
   - Crea el directorio de canciones y copia los archivos `.ogg` desde la máquina local a la máquina virtual.
   ```yaml
   - name: Crear directorio de canciones
     file:
       path: /home/vagrant/canciones
       state: directory
       owner: vagrant
       group: vagrant
       mode: '0755'
     tags: canciones

   - name: Copiar archivo1.ogg
     copy:
       src: /vagrant/archivo1.ogg
       dest: /home/vagrant/canciones/archivo1.ogg
       owner: vagrant
       group: vagrant
       mode: '0644'
       force: yes
       remote_src: yes
     tags: canciones

   - name: Copiar archivo2.ogg
     copy:
       src: /vagrant/archivo2.ogg
       dest: /home/vagrant/canciones/archivo2.ogg
       owner: vagrant
       group: vagrant
       mode: '0644'
       force: yes
       remote_src: yes
     tags: canciones

   - name: Copiar archivo3.ogg
     copy:
       src: /vagrant/archivo3.ogg
       dest: /home/vagrant/canciones/archivo3.ogg
       owner: vagrant
       group: vagrant
       mode: '0644'
       force: yes
       remote_src: yes
     tags: canciones

   - name: Copiar archivo4.ogg
     copy:
       src: /vagrant/archivo4.ogg
       dest: /home/vagrant/canciones/archivo4.ogg
       owner: vagrant
       group: vagrant
       mode: '0644'
       force: yes
       remote_src: yes
     tags: canciones
   ```

8. **Generar lista de reproducción**:
   - Genera la lista de reproducción a partir de los archivos `.ogg` en el directorio de canciones.
   ```yaml
   - name: Generar lista de reproducción
     shell: find /home/vagrant/canciones -name "*.ogg" > /home/vagrant/canciones/lista.txt
     args:
       creates: /home/vagrant/canciones/lista.txt
     tags: canciones
   ```

9. **Configurar y habilitar el servicio Ices2**:
   - Copia el archivo de servicio de Ices2, recarga systemd y habilita el servicio.
   ```yaml
   - name: Copiar ices2.service
     copy:
       src: /vagrant/ices2.service
       dest: /etc/systemd/system/ices2.service
       owner: root
       group: root
       mode: '0644'
       force: yes
       remote_src: yes
     tags: ices2

   - name: Recargar systemd
     systemd:
       daemon_reload: yes
     tags: ices2

   - name: Iniciar servicio Ices2
     systemd:
       name: ices2
       state: started
       enabled: yes
     tags: ices2
   ```

### **Ejecución del Playbook**

Para ejecutar el playbook, asegúrate de tener Ansible instalado y ejecuta el siguiente comando desde el directorio del proyecto:

```bash
ansible-playbook -i inventory playbook.yml
```

**Nota:** Si al ejecutar el playbook aparece el siguiente error:

```bash
user@user-pc:~/Desktop/icecast-vagrant$ ansible-playbook -i inventory.yml playbook.yml 

PLAY [Configurar servidor de streaming con Icecast2 e Ices2] *************************************************************ssh vagrant@192.168.57.101 -i .vagrant/machines/default/virtualbox/private_key****************

TASK [Gathering Facts] *******************************************************************************************************************
fatal: [192.168.57.101]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@\r\n@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @\r\n@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@\r\nIT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!\r\nSomeone could be eavesdropping on you right now (man-in-the-middle attack)!\r\nIt is also possible that a host key has just been changed.\r\nThe fingerprint for the ED25519 key sent by the remote host is\nSHA256:Aag6y/6TqR72tD1Rtw8CvJc/mDIQZ59Rjgp8FiBFulk.\r\nPlease contact your system administrator.\r\nAdd correct host key in /home/user/.ssh/known_hosts to get rid of this message.\r\nOffending ECDSA key in /home/user/.ssh/known_hosts:9\r\n  remove with:\r\n  ssh-keygen -f '/home/user/.ssh/known_hosts' -R '192.168.57.101'\r\nHost key for 192.168.57.101 has changed and you have requested strict checking.\r\nHost key verification failed.", "unreachable": true}

PLAY RECAP *******************************************************************************************************************************
192.168.57.101             : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored==0
```

Este error ocurre porque la clave SSH del host ha cambiado, lo que puede suceder si la máquina virtual ha sido recreada o si su configuración de red ha cambiado. Para solucionar este problema, debes eliminar la clave antigua del archivo `known_hosts` y volver a conectarte al host. Ejecuta los siguientes comandos:

```bash
ssh-keygen -f ~/.ssh/known_hosts -R 192.168.57.101
ssh vagrant@192.168.57.101 -i .vagrant/machines/default/virtualbox/private_key
```

El primer comando elimina la clave antigua del host, y el segundo comando te permite conectarte al host y agregar la nueva clave a `known_hosts`. Después de ejecutar estos comandos, podrás ejecutar el playbook sin problemas.

---

## **Instalación y Configuración Manual de Icecast2** 🎛️

1. **Instalar Icecast2**:
   ```bash
   sudo apt update
   sudo apt install icecast2
   ```
   - Durante la instalación, selecciona "No" cuando te pregunte si deseas establecer una clave de administración.

2. **Configurar Icecast2**:
   Edita el archivo de configuración:
   ```bash
   sudo nano /etc/icecast2/icecast.xml
   ```
   Modifica las siguientes secciones:
   ```xml
   <source-password>alumno</source-password>
   <admin-user>alumno</admin-user>
   <admin-password>alumno</admin-password>
   ```
   - **`<source-password>`**: Contraseña que el software fuente (Ices2) debe proporcionar para autenticarse con el servidor Icecast.
   - **`<admin-user>`** y **`<admin-password>`**: Credenciales para acceder a la interfaz de administración web de Icecast.

3. **Habilitar y iniciar Icecast2**:
   Edita el archivo `/etc/default/icecast2` y establece `ENABLE=true`:
   ```bash
   sudo nano /etc/default/icecast2
   ```
   Inicia y habilita el servicio:
   ```bash
   sudo systemctl start icecast2
   sudo systemctl enable icecast2
   ```

---

## **Instalación y Configuración Manual de Ices2** 🎚️

1. **Instalar Ices2 y Vorbis Tools**:
   ```bash
   sudo apt install ices2 vorbis-tools
   ```

2. **Configurar Ices2**:
   Crea un directorio para la configuración de Ices2:
   ```bash
   sudo mkdir /etc/ices2
   ```
   Copia el archivo de configuración de ejemplo:
   ```bash
   sudo cp /usr/share/doc/ices2/examples/ices-playlist.xml /etc/ices2/
   ```
   Edita el archivo de configuración:
   ```bash
   sudo nano /etc/ices2/ices-playlist.xml
   ```
   Modifica las siguientes secciones:
   ```xml
   <name>La radio trampa</name>
   <genre>Jazz-Pop</genre>
   <description>Recordando a Supertramp</description>
   <param name="type">basic</param>
   <param name="file">/home/vagrant/canciones/lista.txt</param>
   <password>alumno</password>
   <mount>/supertramp</mount>
   ```
   - **`<name>`**, **`<genre>`**, **`<description>`**: Metadatos descriptivos de la estación de radio.
   - **`<param name="type">basic</param>`**: Indica que la lista de reproducción se proporcionará a través de un archivo.
   - **`<param name="file">`**: Ruta del archivo que contiene la lista de reproducción.
   - **`<password>`**: Contraseña que Ices2 proporcionará a Icecast para autenticarse (debe coincidir con `<source-password>` en `icecast.xml`).
   - **`<mount>`**: Punto de montaje de la lista de reproducción en el servidor Icecast.

---

## **Preparación Manual de Archivos de Audio** 🎶

1. **Descargar archivos de audio**:
   - Descarga archivos `.mp3` desde fuentes legales como [Archive.org](https://archive.org/details/audio) o [Freesound](https://freesound.org/).

2. **Convertir archivos a formato `.ogg`**:
   - Convierte los archivos `.mp3` a `.ogg` usando `ffmpeg`:
     ```bash
     sudo apt install ffmpeg
     ffmpeg -i archivo1.mp3 -acodec libvorbis archivo1.ogg
     ffmpeg -i archivo2.mp3 -acodec libvorbis archivo2.ogg
     ffmpeg -i archivo3.mp3 -acodec libvorbis archivo3.ogg
     ffmpeg -i archivo4.mp3 -acodec libvorbis archivo4.ogg
     ```

3. **Copiar archivos a la máquina virtual**:
   - Copia los archivos `.ogg` al directorio de la máquina virtual:
     ```bash
     cp /vagrant/archivo1.ogg /home/vagrant/canciones/
     cp /vagrant/archivo2.ogg /home/vagrant/canciones/
     cp /vagrant/archivo3.ogg /home/vagrant/canciones/
     cp /vagrant/archivo4.ogg /home/vagrant/canciones/
     ```

4. **Generar la lista de reproducción**:
   - Usa el comando `find` para generar la lista de reproducción:
     ```bash
     find /home/vagrant/canciones -name "*.ogg" > /home/vagrant/canciones/lista.txt
     ```

---

## **Automatización Manual del Servicio Ices2** 🤖

1. **Crear un servicio systemd para Ices2**:
   Crea un archivo de servicio:
   ```bash
   sudo nano /etc/systemd/system/ices2.service
   ```
   Añade el siguiente contenido:
   ```ini
   [Unit]
   Description=ices2 daemon
   After=icecast2.service

   [Service]
   User=www-data
   Group=www-data
   WorkingDirectory=/etc/ices2
   ExecStart=ices2 "/etc/ices2/ices-playlist.xml"

   [Install]
   WantedBy=multi-user.target
   ```
   - **`[Unit]`**: Define la unidad y sus dependencias.
   - **`After=icecast2.service`**: Asegura que Ices2 se inicie después de Icecast2.
   - **`[Service]`**: Configura el servicio, incluyendo el usuario, grupo y comando de inicio.
   - **`[Install]`**: Define cómo se habilita el servicio.

2. **Habilitar y iniciar Ices2**:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start ices2.service
   sudo systemctl enable ices2.service
   ```

---

## **Prueba del Servidor de Streaming** 🎧

1. **Acceder a la interfaz web de Icecast**:
   - En tu máquina real, abre un navegador web y accede a `http://localhost:8000`.
   - Haz clic en "Administration" e ingresa con las credenciales (`alumno` / `alumno`).

2. **Reproducir la lista de reproducción**:
   - Usa **VLC Media Player** y abre la URL `http://localhost:8000/supertramp` para reproducir la lista de reproducción.

---
