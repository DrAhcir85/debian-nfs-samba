# 4. Servicio Samba (Tareas 22 a 34)

## Instalación y Configuración Básica
En el servidor se instala `samba`. Se edita `/etc/samba/smb.conf` para compartir `/mnt/smb` (comunSMB) con `guest ok = yes` y `writeable = yes`.
En el cliente instalamos `cifs-utils` y `smbclient`. 

## Seguridad en Samba
Se retira el acceso invitado y se añaden usuarios locales a la base de datos de Samba (`smbpasswd -a julian`).
**Teoría (Tarea 33):** Si se usa `writeable = no` junto con `write list = @profesores`, significa que el recurso es de solo lectura para todos, excepto para los miembros del grupo profesores, que tendrán permisos de escritura.

## ACLs en Samba (Aviso de errata Tarea 30)
Se aplican las mismas reglas ACL que en NFS, pero sobre los directorios creados en `/mnt/smb`.

## Montaje Samba con libpam-mount (Tarea 34)
Se añaden las líneas CIFS en `/etc/security/pam_mount.conf.xml` del cliente (corrigiendo la errata del GID 2001 por los correctos 1501 y 1502):
``xml
<volume sgrp="profesores" fstype="cifs" server="IP_SRV" path="comunSMB" mountpoint="~/SMB" />
<volume sgrp="alumnos" fstype="cifs" server="IP_SRV" path="comunSMB" mountpoint="~/SMB" />



------------------------------------------------------------------------------------------------------------------------------------

## Paso a paso

Instalación de paquetes:
    server: apt install samba
    cliente: apt install cifs-utils smbclient

## Server:
## Server:
Crear recurso compartido - nano /etc/samba/smb.conf
En el final del archivo añadir [config smb.conf](config-backups/server/smb.conf.md)
Reiniciar servicio - systemctl restart smbd




## Cliente:
Comprobar si servidor comparte: smbclient -L //10.100.0.10 -N
Montar carpeta compartida: sudo mount -t cifs //10.100.0.10/comunSMB /mnt/comunSMB -o guest
Desmontar carpeta - umount /mnt/comunSMB
Configuración automática en fstab
    nano /etc/fstab
    //10.100.0.10/comunSMB    /mnt/comunSMB    cifs    guest,uid=1000,iocharset=utf8    0 0
    Verificar que no haya errores -- mount -a
    

----------------------------------------------------------

NUEVO

# 4. Samba y libpam-mount

**Objetivo:** compartir `/mnt/smb` como recurso `comunSMB` (primero con invitados, después con usuarios), controlar el acceso con ACL y montarlo en el cliente de forma manual, con `fstab` y con `libpam-mount`.

**Configuraciones de referencia:** [server/smb.conf.md](config-backups/server/smb.conf.md) · [cliente/fstab.md](config-backups/cliente/fstab.md) · [cliente/pam_mount.conf.xml.md](config-backups/cliente/pam_mount.conf.xml.md)

---

## 4.1 Instalar Samba (Tareas 22 y 24)

**[SERVER]**

```bash
apt install samba
systemctl status smbd                # active (running)
```

**[CLIENTE]**

```bash
apt install cifs-utils smbclient
```

## 4.2 Recurso `comunSMB` con acceso de invitado (Tarea 23)

**[SERVER]**

```bash
nano /etc/samba/smb.conf
```

Añade **al final**:

```ini
[comunSMB]
   path = /mnt/smb
   browseable = yes
   writable = yes
   guest ok = yes
   create mask = 0777
   directory mask = 0777
```

```bash
chmod 777 /mnt/smb                   # el invitado también necesita permiso sobre la carpeta
systemctl restart smbd
testparm -s
smbclient -L //localhost -N          # debe aparecer comunSMB
```

## 4.3 Ver y montar a mano (Tareas 24 y 25)

**[CLIENTE]**, como root:

```bash
smbclient -L //10.100.0.10 -N
mkdir /mnt/comunSMB
mount -t cifs //10.100.0.10/comunSMB /mnt/comunSMB -o guest
touch /mnt/comunSMB/prueba
umount /mnt/comunSMB
```

## 4.4 Montaje automático con `fstab` (Tarea 26)

**[CLIENTE]**

```bash
nano /etc/fstab
```

```text
//10.100.0.10/comunSMB  /mnt/comunSMB  cifs  guest,uid=1000,file_mode=0777,dir_mode=0777,iocharset=utf8  0 0
```

> `file_mode` y `dir_mode` evitan que el cliente impida escribir a usuarios distintos del `uid=1000` (Tarea 27).

```bash
mount -a
df -h
reboot
```

## 4.5 Escribir con dos usuarios (Tarea 27)

**[CLIENTE]:** inicia sesión con dos usuarios (por ejemplo `julian` y `alumno`):

```bash
touch /mnt/comunSMB/prueba-$USER
ls -l /mnt/comunSMB
```

## 4.6 Acceso con el usuario `samba` (Tareas 28 y 29)

**[SERVER]**

```bash
useradd -M -s /usr/sbin/nologin samba    # Samba necesita un usuario de sistema
smbpasswd -a samba                       # contraseña indicada en el enunciado
nano /etc/samba/smb.conf
```

```ini
[comunSMB]
   path = /mnt/smb
   browseable = yes
   writable = yes
   guest ok = no
   valid users = samba
   create mask = 0777
   directory mask = 0777
```

```bash
systemctl restart smbd
```

**[CLIENTE]**

```bash
umount /mnt/comunSMB
nano /etc/fstab                          # comenta con # la línea de Samba
systemctl daemon-reload
mount -t cifs //10.100.0.10/comunSMB /mnt/comunSMB -o guest      # debe fallar: error(13)
```

**Respuesta a la Tarea 29:** sí, solicita usuario y clave. En `fstab` hay que dar las credenciales con un archivo protegido:

```bash
nano /root/.smbcredentials
```

```text
username=samba
password=CONTRASEÑA_DEL_ENUNCIADO
```

```bash
chmod 600 /root/.smbcredentials
```

```text
//10.100.0.10/comunSMB  /mnt/comunSMB  cifs  credentials=/root/.smbcredentials,file_mode=0777,dir_mode=0777,iocharset=utf8  0 0
```

## 4.7 ACL en `/mnt/smb` (Tarea 30)

> **Errata del enunciado:** habla de `/mnt/nfs/...`, pero las carpetas de esta tarea son las de `/mnt/smb`.

**[SERVER]**

```bash
mkdir /mnt/smb/profesores /mnt/smb/alumnos
setfacl -m g:profesores:rwx /mnt/smb/profesores
setfacl -m g:profesores:rwx /mnt/smb/alumnos
setfacl -m g:alumnos:rwx   /mnt/smb/alumnos
setfacl -m g:alumnos:---   /mnt/smb/profesores
```

## 4.8 Usuarios `julian` y `alumno` en Samba (Tareas 31 y 32)

**[SERVER]**

```bash
smbpasswd -x samba                       # borra samba de la base de datos de Samba
smbpasswd -a julian
smbpasswd -a alumno
pdbedit -L                               # deben salir julian y alumno
nano /etc/samba/smb.conf                 # valid users = julian, alumno
systemctl restart smbd
```

**[CLIENTE]**, como root (repite con `alumno`):

```bash
mount -t cifs //10.100.0.10/comunSMB /mnt/comunSMB -o username=julian,file_mode=0777,dir_mode=0777
touch /mnt/comunSMB/profesores/prueba-julian
touch /mnt/comunSMB/alumnos/prueba-julian
umount /mnt/comunSMB
```

Pide la contraseña de Samba de cada usuario. Resultado esperado: `julian` escribe en las dos carpetas; `alumno` solo en `alumnos`.

## 4.9 Teoría (Tarea 33)

Con `writeable = no` y `write list = @profesores`, el recurso es de **solo lectura para todos**, salvo para los miembros del grupo `profesores` (la `@` indica grupo), que sí pueden escribir.

## 4.10 Montaje con `libpam-mount` (Tarea 34)

> **Errata del enunciado:** dice "gid 2001 y 2001". Los GID correctos son **1501 y 1502**.

**[CLIENTE]**, como root:

```bash
nano /etc/fstab                          # la línea de Samba debe estar comentada o borrada
nano /etc/security/pam_mount.conf.xml
```

Añade **encima de `</pam_mount>`**:

```xml
<volume gid="1501" fstype="cifs" server="10.100.0.10" path="comunSMB" mountpoint="~/SMB" />
<volume gid="1502" fstype="cifs" server="10.100.0.10" path="comunSMB" mountpoint="~/SMB" />
```

> `pam_mount` usa la contraseña con la que inicias sesión: la contraseña de Samba (`smbpasswd`) del usuario debe ser **igual** a la de su inicio de sesión en el cliente.

Cierra sesión e inicia con `julian` **desde la pantalla gráfica**.

---

## ✅ Comprobación / Verificación

**[SERVER]**

```bash
testparm -s                              # sin errores; muestra [comunSMB]
systemctl status smbd
pdbedit -L                               # usuarios de Samba
getfacl /mnt/smb/profesores
```

**[CLIENTE]**

```bash
smbclient -L //10.100.0.10 -U julian     # lista el recurso (pide contraseña)
df -h | grep comunSMB                    # con montaje manual o fstab
mount | grep SMB                         # con libpam-mount, tras iniciar sesión gráfica
```

| Usuario | `profesores` | `alumnos` |
| :--- | :---: | :---: |
| `julian` | ✅ escribe | ✅ escribe |
| `alumno` | ❌ sin acceso | ✅ escribe |

Si algo no coincide, consulta [registro-errores.md](registro-errores.md#-samba-y-pam).

---
⬅️ [Anterior](03-nfs-y-acls.md) | 🏠 [Volver al Índice](README.md) | [Siguiente](registro-errores.md) ➡️