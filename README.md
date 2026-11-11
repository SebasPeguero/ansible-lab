# Práctica 2do Parcial: Laboratorio de Ansible con Docker

**Estudiante:** Sebastián Peguero  
**Matrícula:** 2025-1172  
**Materia:** Electiva II (DevOps)  
**Institución:** Instituto Tecnológico de Las Américas (ITLA)  
**Repositorio:** [ansible-lab](https://github.com/SebasPeguero/ansible-lab)

---

## 🎯 Objetivo de la Práctica
Crear un laboratorio con dos servidores virtuales Linux (Ubuntu 24.04) utilizando **Docker Compose** y administrarlos de manera automatizada mediante **Ansible** utilizando el conector nativo de Docker (`ansible_connection=docker`).

---

## 📁 Estructura del Laboratorio
```text
ansible-lab/
├── docker-compose.yml   # Definición de los servidores Linux (server1 y server2)
├── inventory.ini        # Inventario de Ansible con conector Docker
├── playbook.yml         # Playbook con las tareas de automatización y condicionales
├── .gitignore           # Archivos ignorados por Git
└── README.md            # Documentación completa y evidencia de ejecución
```

---

## ⚙️ Configuración del Entorno

### 1. Servidores en Docker (`docker-compose.yml`)
```yaml
services:
  server1:
    image: ubuntu:24.04
    container_name: server1
    command: sleep infinity

  server2:
    image: ubuntu:24.04
    container_name: server2
    command: sleep infinity
```

### 2. Inventario de Ansible (`inventory.ini`)
```ini
[docker]
server1 ansible_connection=docker
server2 ansible_connection=docker
```

---

## 🚀 Comandos de Ejecución Paso a Paso

### Paso 1: Iniciar los contenedores
```bash
docker compose up -d
```

Verificar que ambos estén ejecutándose:
```bash
docker ps
```

### Paso 2: Instalar Python en cada contenedor
Ansible requiere Python en los nodos administrados para ejecutar sus módulos:
```bash
# En server1
docker exec -it server1 bash -c "apt update && apt install -y python3"

# En server2
docker exec -it server2 bash -c "apt update && apt install -y python3"
```

### Paso 3: Comprobar la conexión con Ansible
```bash
ansible docker -i inventory.ini -m ping
```

### Paso 4: Ejecutar el Playbook
```bash
ansible-playbook -i inventory.ini playbook.yml
```

---

## 📋 Tareas Realizadas por el Playbook (`playbook.yml`)
1. **Mostrar el nombre del host:** Muestra el nombre (`server1` / `server2`) mediante el módulo `debug`.
2. **Mostrar el sistema operativo:** Muestra la distribución y versión (`Ubuntu 24.04`).
3. **Crear el directorio `/tmp/ansible-demo`:** Asegura la existencia del directorio con permisos `0755`.
4. **Crear el archivo `info.txt`:** Escribe en `/tmp/ansible-demo/info.txt` el nombre del servidor, sistema operativo, kernel, arquitectura y datos del estudiante.
5. **Crear directorios con loop:** Crea mediante un bucle los subdirectorios:
   * `logs`
   * `backup`
   * `config`
6. **Mensaje condicional de Ubuntu:** Evalúa `when: ansible_distribution == "Ubuntu"` y muestra un mensaje confirmando la distribución.

---

## 📸 Evidencia de Ejecución Exitosa

### 1. Salida del Ping de Ansible:
```json
server1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
server2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

### 2. Ejecución del Playbook (`ansible-playbook -i inventory.ini playbook.yml`):
```text
PLAY [Practica 2do Parcial Ansible - Gestion de Servidores Docker] ********************

TASK [Gathering Facts] *****************************************************************
ok: [server1]
ok: [server2]

TASK [1. Mostrar el nombre del host] ***************************************************
ok: [server1] => {
    "msg": "El nombre del host es: server1"
}
ok: [server2] => {
    "msg": "El nombre del host es: server2"
}

TASK [2. Mostrar el sistema operativo] *************************************************
ok: [server1] => {
    "msg": "El sistema operativo es: Ubuntu 24.04 (Familia: Debian)"
}
ok: [server2] => {
    "msg": "El sistema operativo es: Ubuntu 24.04 (Familia: Debian)"
}

TASK [3. Crear el directorio /tmp/ansible-demo] ****************************************
changed: [server1]
changed: [server2]

TASK [4. Crear archivo info.txt y escribir el nombre del servidor y el sistema operativo] *
changed: [server1]
changed: [server2]

TASK [5. Crear mediante un loop los directorios (logs, backup, config)] ****************
changed: [server1] => (item=logs)
changed: [server1] => (item=backup)
changed: [server1] => (item=config)
changed: [server2] => (item=logs)
changed: [server2] => (item=backup)
changed: [server2] => (item=config)

TASK [6. Mostrar un mensaje únicamente cuando el sistema operativo sea Ubuntu] **********
ok: [server1] => {
    "msg": "¡Confirmado! Este servidor (server1) está ejecutando el sistema operativo Ubuntu 24.04."
}
ok: [server2] => {
    "msg": "¡Confirmado! Este servidor (server2) está ejecutando el sistema operativo Ubuntu 24.04."
}

PLAY RECAP *****************************************************************************
server1                    : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
server2                    : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 3. Verificación de archivos generados dentro de los contenedores:
```text
$ docker exec server1 cat /tmp/ansible-demo/info.txt
==============================================
Laboratorio 2do Parcial - Ansible & Docker
==============================================
Nombre del Servidor: server1
Sistema Operativo  : Ubuntu 24.04
Kernel             : 6.6.137+
Arquitectura       : x86_64
Estudiante         : Sebastián Peguero (2025-1172)
Materia            : Electiva II (DevOps)
==============================================

$ docker exec server1 ls -la /tmp/ansible-demo
total 12
drwxr-xr-x 5 root root 4096 Nov 10 20:49 .
drwxrwxrwt 1 root root 4096 Nov 10 20:49 ..
drwxr-xr-x 2 root root 4096 Nov 10 20:49 backup
drwxr-xr-x 2 root root 4096 Nov 10 20:49 config
-rw-r--r-- 1 root root  384 Nov 10 20:49 info.txt
drwxr-xr-x 2 root root 4096 Nov 10 20:49 logs
```
