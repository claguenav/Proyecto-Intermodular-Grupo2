# Proyecto Intermodular – (Ansible)

Automatización de la gestión de **aulas informáticas** con **Ansible**: configuración de equipos, creación y eliminación de usuarios, instalación de paquetes, mantenimiento, y apagado/reinicio remoto, tanto en equipos **Linux** como **Windows**.
 Proyecto intermodular realizado en **2º del Ciclo Formativo de Grado Medio de Sistemas Microinformáticos y Redes (SMR)**.

## Descripción

Este proyecto permite administrar de forma centralizada y reproducible los equipos de varias aulas desde un único nodo de control:

- **Aula 1 y Aula 2:** equipos Linux.
- **Aula 3:** equipos Windows.

**Qué se puede hacer:**

- Configurar cada aula (configuración del sistema, paquetes, usuarios y seguridad).
- Crear usuarios de forma masiva a partir de ficheros de variables.
- Eliminar usuarios de cada aula.
- Instalar paquetes comunes.
- Ejecutar tareas de mantenimiento.
- Apagar y reiniciar equipos de forma remota.

**Tecnologías:** Ansible · YAML · Bash · PowerShell ·


## Estructura del repositorio

```
Proyecto-Intermodular-Grupo2/
├── ansible.cfg
├── inventario/
├── playbooks/
│   ├── apagar_equipos_linux.yml
│   ├── apagar_equipos_windows.yml
│   ├── configurar_aula1.yml
│   ├── configurar_aula2.yml
│   ├── configurar_aula3_win.yml
│   ├── eliminar_usuarios_aula1.yml
│   ├── eliminar_usuarios_aula2.yml
│   ├── eliminar_usuarios_aula3_win.yml
│   ├── mantenimiento.yml
│   ├── mantenimiento_windows.yml
│   ├── reiniciar_equipos.yml
│   └── reiniciar_equipos_windows.yml
├── roles/
│   ├── configuracion/
│   ├── eliminar_usuarios/
│   ├── mantenimiento/
│   ├── paquetes/
│   ├── seguridad/
│   ├── usuarios/
│   ├── windows_configuracion/
│   ├── windows_eliminar_usuarios/
│   ├── windows_mantenimiento/
│   ├── windows_paquetes/
│   ├── windows_seguridad/
│   └── windows_usuarios/
├── scripts/
│   ├── boostrap_ansible.sh
│   └── configuracion-inicial-win.ps1
├── vars/
│   ├── eliminar_usuarios_aula1.yml
│   ├── eliminar_usuarios_aula2.yml
│   ├── paquetes_comunes.yml
│   ├── usuarios_aula1.yml
│   ├── usuarios_aula2.yml
│   ├── usuarios_aula3_win.yml
│   └── usuarios_eliminar_aula3_win.yml
└── README.md
```

| Carpeta / archivo | Descripción |
|-------------------|-------------|
| `ansible.cfg` | Configuración general de Ansible. |
| `inventario/` | Hosts y grupos (equipos de cada aula). |
| `playbooks/` | Playbooks que orquestan cada tarea. |
| `roles/` | Lógica reutilizable, separada para Linux y Windows (`windows_*`). |
| `scripts/` | Scripts para preparar los nodos antes de usar Ansible. |
| `vars/` | Listas de usuarios y paquetes por aula. |



## Requisitos previos

**Nodo de control**
- Linux (o WSL) con Python 3 y **Ansible** instalado.
- Cliente SSH.
- Para gestionar Windows: la librería `pywinrm` y la colección `ansible.windows` (y `community.windows` si se usa).

**Equipos Linux (aulas 1 y 2)**
- Acceso por SSH y Python 3.
- Usuario con permisos `sudo`.

**Equipos Windows (aula 3)**
- **WinRM** habilitado (se prepara con `configuracion-inicial-win.ps1`).
- Cuenta de administrador.



## Instalación y preparación

1. **Clona el repositorio:**

   ```bash
   git clone https://github.com/claguenav/Proyecto-Intermodular-Grupo2.git
   cd Proyecto-Intermodular-Grupo2
   ```

2. **Prepara el nodo de control** con el script incluido:

   ```bash
   chmod +x scripts/boostrap_ansible.sh
   sudo ./scripts/boostrap_ansible.sh
   ```

3. **Prepara los equipos Windows** ejecutando en cada uno, desde PowerShell como administrador:

   ```powershell
   .\scripts\configuracion-inicial-win.ps1
   ```

4. **Instala las dependencias para Windows** (si no las instaló el bootstrap):

   ```bash
   pip install pywinrm
   ansible-galaxy collection install ansible.windows community.windows
   ```



## Configuración

1. **Inventario:** añade las IPs o nombres de los equipos de cada aula en `inventario/`.

   ```ini
   # Ejemplo
   [aula1]
   pc-aula1-01 ansible_host=192.168.1.11

   [aula3_win]
   pc-aula3-01 ansible_host=192.168.3.11

   [aula3_win:vars]
   ansible_connection=winrm
   ```

2. **Variables:** edita los ficheros de `vars/` con los usuarios y paquetes de cada aula.

3. **Credenciales:** configura el acceso SSH (claves) para Linux y las credenciales WinRM para Windows. Si guardáis contraseñas en ficheros, protegedlas con **Ansible Vault**:

   ```bash
   ansible-vault encrypt vars/usuarios_aula3_win.yml
   ```

4. **Comprueba la conectividad:**

   ```bash
   ansible aula1 -m ping
   ansible aula3_win -m win_ping
   ```



## Uso

Los playbooks se lanzan desde la raíz del proyecto:

```bash
ansible-playbook playbooks/configurar_aula1.yml
```

Opciones útiles:

```bash
# Simular sin aplicar cambios
ansible-playbook playbooks/configurar_aula1.yml --check

# Pedir contraseña de sudo
ansible-playbook playbooks/configurar_aula1.yml --ask-become-pass

# Si usáis Ansible Vault
ansible-playbook playbooks/configurar_aula1.yml --ask-vault-pass

# Limitar a un equipo
ansible-playbook playbooks/configurar_aula1.yml --limit pc-aula1-01

# Salida detallada
ansible-playbook playbooks/configurar_aula1.yml -vv
```



## Playbooks

| Playbook | Sistema | Descripción |
|----------|---------|-------------|
| `configurar_aula1.yml` | Linux | Configura los equipos del aula 1. |
| `configurar_aula2.yml` | Linux | Configura los equipos del aula 2. |
| `configurar_aula3_win.yml` | Windows | Configura los equipos del aula 3. |
| `eliminar_usuarios_aula1.yml` | Linux | Elimina los usuarios definidos para el aula 1. |
| `eliminar_usuarios_aula2.yml` | Linux | Elimina los usuarios definidos para el aula 2. |
| `eliminar_usuarios_aula3_win.yml` | Windows | Elimina los usuarios definidos para el aula 3. |
| `mantenimiento.yml` | Linux | Tareas de mantenimiento. |
| `mantenimiento_windows.yml` | Windows | Tareas de mantenimiento. |
| `apagar_equipos_linux.yml` | Linux | Apaga los equipos de forma remota. |
| `apagar_equipos_windows.yml` | Windows | Apaga los equipos de forma remota. |
| `reiniciar_equipos.yml` | Linux | Reinicia los equipos. |
| `reiniciar_equipos_windows.yml` | Windows | Reinicia los equipos. |



## Roles

| Rol Linux | Rol Windows | Función |
|-----------|-------------|---------|
| `configuracion` | `windows_configuracion` | Configuración general del sistema. |
| `paquetes` | `windows_paquetes` | Instalación de paquetes y software. |
| `usuarios` | `windows_usuarios` | Creación y gestión de usuarios. |
| `eliminar_usuarios` | `windows_eliminar_usuarios` | Eliminación de usuarios. |
| `seguridad` | `windows_seguridad` | Medidas de seguridad. |
| `mantenimiento` | `windows_mantenimiento` | Tareas de mantenimiento. |



## Variables

| Fichero | Contenido |
|---------|-----------|
| `usuarios_aula1.yml` / `usuarios_aula2.yml` | Usuarios a crear en cada aula Linux. |
| `usuarios_aula3_win.yml` | Usuarios a crear en el aula Windows. |
| `eliminar_usuarios_aula1.yml` / `eliminar_usuarios_aula2.yml` | Usuarios a eliminar en cada aula Linux. |
| `usuarios_eliminar_aula3_win.yml` | Usuarios a eliminar en el aula Windows. |
| `paquetes_comunes.yml` | Paquetes comunes a instalar. |



## Scripts

| Script | Descripción |
|--------|-------------|
| `boostrap_ansible.sh` | Prepara el nodo de control / entorno para usar Ansible. |
| `configuracion-inicial-win.ps1` | Configuración inicial de los equipos Windows para poder gestionarlos con Ansible. |



## Licencia

https://creativecommons.org/licenses/by-nc-sa/4.0/deed.es
