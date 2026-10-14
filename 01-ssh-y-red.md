# 1. Configuración Base y Acceso SSH (Tareas 1 a 8)

## Configuración de Red y Nombres
*   Servidor renombrado a `eq-srv` y cliente a `eq-cliente`.
*   Configurada red NAT (`10.100.0.0/16`) con DNS 8.8.8.8 y 1.1.1.1 en ambos equipos.

## Creación de Usuario Cliente
Se crea el usuario `jefe` con acceso sudo en el cliente:
`sudo useradd -m -s /bin/bash jefe`
`sudo usermod -aG sudo jefe`

## Configuración SSH
En el servidor instalamos `openssh-server` y permitimos el acceso root editando `/etc/ssh/sshd_config` (`PermitRootLogin yes`).
En el cliente, desde el usuario jefe, generamos claves y las enviamos al servidor:
`ssh-keygen` (Passphrase: "Trump always ChickenOut")
`ssh-copy-id root@IP_SRV`
A partir de este punto, la administración del servidor se realiza remotamente.


------------------------------------------------------------------------------------------------------------------------------------
## Paso a paso

## RED:
* Cambiar nombre a los equipos - host/hostname
* Descargar resolvconf
## Cambiar configuración de red:
*    Repositorios /etc/apt/sources.list
*    srv - 10.100.0.10/16
*    cliente - 10.100.0.20/16
*    Configuración en config-backups
*    systemctl restart networking
*    Migrar a sources.list.d - apt modernize-sources
*    OJO! con GUI cambiar siempre graficamente la IP

## Usuarios:
* sudo useradd -m -s /bin/bash jefe  -  -m carpeta en /home -s bash
* passwd usuario
* usermod -aG sudo usuario   -   añade al grupo sudo sin borrar de otros

## SSH:
* server: apt install openssh-server
* cliente: apt install openssh-client
## Generar clave pub/priv - ssh-keygen -t rsa -b 2048
Enviar clave - ssh-copy-id usuario@server
Si se quiere logear por ssh como root y que root no pueda logearse en servidor:
*    nano /etc/ssh/sshd.conf - permitrootlogin yes (copiar la linea comentada para no eliminarla)
*    generar passwd a root - restart ssh
*    cuando la clave esté copiada borrar la linea y dejarlo como estaba



-------------------------------------------------------------
NUEVo


# 1. Configuración base y acceso SSH

**Objetivo:** dejar las dos máquinas con nombre, red NAT, repositorios oficiales, un usuario administrador en el cliente y acceso SSH por clave al usuario `root` del servidor.

**Configuraciones de referencia:** [server/Interfaces.md](config-backups/server/Interfaces.md) · [cliente/Interfaces.md](config-backups/cliente/Interfaces.md) · [server/sshd_config.md](config-backups/server/sshd_config.md)

> 💡 Para ser administrador desde un usuario normal usa **`su -`** (con guion). Con `su` a secas faltan las rutas de administración y salen errores como `useradd: command not found`.

---

## 1.1 VirtualBox (Tareas 1, 2 y 4)

**[VIRTUALBOX]**
1. `Archivo > Importar servicio virtualizado`: importa el `.ova` del servidor (sin entorno gráfico) y el del cliente (con entorno gráfico).
2. Con el servidor **apagado**: `Configuración > Almacenamiento` > añadir disco duro > **VDI, 4 GB**.
3. Crea la red NAT: `Archivo > Herramientas > Administrador de red > Redes NAT` > nueva red con prefijo **`10.100.0.0/16`**.
4. En **cada máquina**: `Configuración > Red > Adaptador 1` > **Red NAT** > selecciona esa red.

**[SERVER]** comprueba el disco nuevo:

```bash
lsblk        # debe aparecer sdb de 4G
```

## 1.2 Nombre de los equipos (Tarea 3)

**[SERVER]**

```bash
su -
hostnamectl set-hostname eq-srv
nano /etc/hosts          # en la línea 127.0.1.1 pon: 127.0.1.1   eq-srv
```

**[CLIENTE]**

```bash
su -
hostnamectl set-hostname eq-cliente
nano /etc/hosts          # en la línea 127.0.1.1 pon: 127.0.1.1   eq-cliente
```

Cierra sesión y vuelve a entrar para ver el nombre nuevo en el prompt.

## 1.3 Red con IP fija y DNS (Tarea 4)

### [SERVER] (sin entorno gráfico)

1. Instala `resolvconf` **antes** de pasar a IP fija (necesita internet, que aún tienes por DHCP). Es el paquete que aplica la línea `dns-nameservers`:

```bash
   apt install resolvconf
```
2. Averigua el nombre de la interfaz y edita la red:

```bash
   ip -br link
   nano /etc/network/interfaces
```
3. Contenido (copia limpia en [server/Interfaces.md](config-backups/server/Interfaces.md)):

```ini
   auto enp0s3
   allow-hotplug enp0s3
   iface enp0s3 inet static
       address 10.100.0.10/16
       gateway 10.100.0.1
       dns-nameservers 8.8.8.8 1.1.1.1
```
4. Aplica:

```bash
   systemctl restart networking
```

### [CLIENTE] (con entorno gráfico)

> ⚠️ En el cliente la IP se configura **solo desde la herramienta gráfica** (NetworkManager). No edites `/etc/network/interfaces`: provoca pérdida de red al cabo de un minuto (ver [registro-errores.md](registro-errores.md)).

`Configuración > Red` > engranaje de la conexión cableada > pestaña **IPv4** > método **Manual**:

| Campo | Valor |
| :--- | :--- |
| Dirección | `10.100.0.20` |
| Máscara | `255.255.0.0` |
| Puerta de enlace | `10.100.0.1` |
| DNS | `8.8.8.8, 1.1.1.1` (desactiva el DNS automático) |

Apaga y enciende la conexión. Alternativa por terminal: ver [cliente/Interfaces.md](config-backups/cliente/Interfaces.md).

## 1.4 Repositorios oficiales (Tarea 5)

**[SERVER]** y **[CLIENTE]**

```bash
cat /etc/os-release              # anota VERSION_CODENAME
nano /etc/apt/sources.list
```

Deja solo los repositorios oficiales (sustituye `CODENAME`) y comenta con `#` cualquier línea `deb cdrom:`:

```ini
deb http://deb.debian.org/debian CODENAME main
deb http://deb.debian.org/debian CODENAME-updates main
deb http://security.debian.org/debian-security CODENAME-security main
```

```bash
apt modernize-sources            # opcional: migra al formato en sources.list.d
apt update
```

## 1.5 Usuario `jefe` con sudo (Tarea 6)

**[CLIENTE]**, como root:

```bash
useradd -m -s /bin/bash jefe
passwd jefe                      # contraseña indicada en el enunciado
usermod -aG sudo jefe            # si "sudo" no existe: apt install sudo
```

## 1.6 SSH (Tarea 7)

> ⚠️ Haz este bloque **antes** de la Tarea 8: después no puedes usar la consola de `eq-srv`.

**[SERVER]**, como root:

```bash
apt install openssh-server
nano /etc/ssh/sshd_config        # copia la línea PermitRootLogin comentada y deja: PermitRootLogin yes
passwd root                      # contraseña temporal de root
systemctl restart ssh
```

**[CLIENTE]**, como `jefe`:

```bash
ssh-keygen -t rsa -b 2048        # pide la passphrase indicada en el enunciado
ssh-copy-id root@10.100.0.10     # pide la contraseña de root del servidor
ssh root@10.100.0.10             # ahora debe pedir la passphrase de la clave
exit
```

**[SERVER]** cierra el acceso por contraseña: vuelve a dejar `PermitRootLogin` como estaba (por defecto root solo entra con clave) y reinicia:

```bash
nano /etc/ssh/sshd_config
systemctl restart ssh
```

## 1.7 Cerrar sesión (Tarea 8)

**[CLIENTE]:** cierra sesión e inicia con `jefe`. Desde ahora el servidor se administra por SSH:

```bash
ssh root@10.100.0.10             # prompt: root@eq-srv:~#
```

---

## ✅ Comprobación / Verificación

**[SERVER]** y **[CLIENTE]**

```bash
hostname                         # eq-srv / eq-cliente
ip -br addr                      # 10.100.0.10/16 / 10.100.0.20/16
ip route                         # default via 10.100.0.1
ping -c 2 8.8.8.8                # salida a internet
ping -c 2 google.com             # resolución DNS
apt update                       # sin líneas E: ni Err:
```

**[CLIENTE]**

```bash
id jefe                          # incluye el grupo sudo
sudo whoami                      # root
ssh root@10.100.0.10             # pide la passphrase, no la contraseña de root
```

| Comprobación | Resultado esperado |
| :--- | :--- |
| `ping 8.8.8.8` falla | Revisar red NAT y gateway |
| `ping 8.8.8.8` funciona y `google.com` no | Problema de DNS: ver [registro-errores.md](registro-errores.md) |
| SSH pide contraseña de root | La clave no se copió: repetir `ssh-copy-id` |

---
⬅️ [Anterior](README.md) | 🏠 [Volver al Índice](README.md) | [Siguiente](02-discos-y-particiones.md) ➡️