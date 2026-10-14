# 3. Servicio NFS y Permisos ACL (Tareas 10 a 21)

## Usuarios y Grupos (Servidor y Cliente)
Se replica la misma estructura en ambos equipos para mantener coherencia de IDs:
`groupadd -g 1501 profesores` | `groupadd -g 1502 alumnos`
`useradd -u 1501 -g profesores -s /bin/bash -m julian`
`useradd -u 1502 -g alumnos -s /bin/bash -m alumno`

## Configuración NFS y Exportación
Se instala `nfs-kernel-server` en el servidor y se edita `/etc/exports`:
`/mnt/nfs 10.100.0.0/16(rw,sync,no_root_squash,no_subtree_check)`
*(Nota: Se configura como rw para poder cumplir la Tarea 17 de permisos ACL).*

En el cliente, `root` no puede crear archivos por defecto si la exportación está en modo `ro` (Read-Only), aunque tenga `no_root_squash`.

## Seguridad mediante ACLs
Se crean directorios en `/mnt/nfs` y se aplican ACLs en el servidor:
* `setfacl -m g:profesores:rwx /mnt/nfs/profesores` (y en alumnos).
* `setfacl -m g:alumnos:rwx /mnt/nfs/alumnos`.
* `setfacl -m g:alumnos:--- /mnt/nfs/profesores` (Acceso denegado).

## Montaje con libpam-mount (Tarea 21)
Se instala `libpam-mount` en el cliente y se configuran los volúmenes en `/etc/security/pam_mount.conf.xml` para montar NFS al iniciar sesión gráfica:
``xml
<volume sgrp="profesores" fstype="nfs" server="IP_SRV" path="/mnt/nfs" mountpoint="~/NFS" />
<volume sgrp="alumnos" fstype="nfs" server="IP_SRV" path="/mnt/nfs" mountpoint="~/NFS" />



------------------------------------------------------------------------------------------------------------------------------------

## Paso a paso


## Grupo y usuarios (srv y cliente):
*   groupadd -g 1500 profesores
*   useradd -u 1501 -g profesores -s /bin/bash -m julian
*       -u para el identificador de usuario (UID).
        -g (en minúscula) para asignar el grupo principal.
        -s para definir la consola bash.
        -m para forzar la creación de la carpeta en /home.
*   id julian   -  asegurarse

## NFS y ACL:
srv: apt install nfs-kernel-server acl
Cliente: apt install nfs-common libpam-mount
## Server (exportar carpeta a la red):
*   nano /etc/exports
*   /mnt/nfs 10.100.0.0/16(rw,syc,no_root_squash,no_subtree_check)
*       root_squash (por defecto) → El root del cliente NO es root en el
        servidor. Se convierte en nobody.
        no_root_squash → El root del cliente actúa como root en el servidor.
        all_squash → Todos los usuarios del cliente se convierten en nobody. 
* Aplicar cambios (systemctl restart nfs-kernel-server)
----------------------------------------------------------
* setfacl -m g:profesores:rwx /mnt/nfs/profesores
* setfacl -m g:profesores:rwx /mnt/nfs/alumnos
* setfacl -m g:alumnos:rwx /mnt/nfs/alumnos
* setfacl -m g:alumnos:--- /mnt/nfs/profesores
* Comprobación: getfacl /mnt/nfs/profesores
## Cliente (montaje gráfico automático):
* Comprobación de si hay carpetas compartidas: showmount -e 10.100.0.10
* mount -t nfs 10.100.0.10:/mnt/comunNFS
* umount /mnt/comunNFS
* nano /etc/fstab  -  10.100.0.10:/mnt/nfs /mnt/comun nfs defaults 0 0
* desmontar carpeta - umount /mnt/comunNFS
* nano /etc/nfstab - borrar línea
* nano /etc/security/pam_mount.conf.xml
* Añadir en el final del archivo, encima de última línea que dice </pam_mount>
    <volume gid="1501" fstype="nfs" server="IP_SERVIDOR" path="/mnt/nfs" mountpoint="~/NFS" />
    <volume sgrp="1502" fstype="nfs" server="IP_SERVIDOR" path="/mnt/nfs" mountpoint="~/NFS" />



    ---------------------------------------------------------

    NUEVO

    # 3. NFS y ACL

**Objetivo:** compartir `/mnt/nfs` por NFS, controlar el acceso por grupos con ACL y montarlo en el cliente de forma manual, automática (`fstab`) y al iniciar sesión (`libpam-mount`).

**Configuraciones de referencia:** [server/exports.md](config-backups/server/exports.md) · [cliente/fstab.md](config-backups/cliente/fstab.md) · [cliente/pam_mount.conf.xml.md](config-backups/cliente/pam_mount.conf.xml.md)

> NFS identifica a los usuarios por **número** (UID/GID). Por eso grupos y usuarios deben tener **los mismos números** en servidor y cliente.

---

## 3.1 Grupos y usuarios (Tareas 10 y 11)

**[SERVER]** y **[CLIENTE]**, como root (mismos comandos en ambos):

```bash
groupadd -g 1501 profesores
groupadd -g 1502 alumnos
useradd -u 1501 -g profesores -s /bin/bash -m julian
useradd -u 1502 -g alumnos -s /bin/bash -m alumno
passwd julian
passwd alumno
```

## 3.2 Instalar paquetes (Tareas 12 y 14)

**[SERVER]**

```bash
apt install nfs-kernel-server acl
```

**[CLIENTE]**

```bash
apt install nfs-common libpam-mount
```

## 3.3 Exportar `/mnt/nfs` en solo lectura (Tarea 13)

**[SERVER]**

```bash
nano /etc/exports
```

```text
/mnt/nfs 10.100.0.0/16(ro,sync,no_root_squash,no_subtree_check)
```

> Sin espacio entre `10.100.0.0/16` y el paréntesis.

```bash
exportfs -ra
exportfs -v
systemctl restart nfs-kernel-server
```

## 3.4 Ver y montar a mano desde el cliente (Tareas 14, 15 y 16)

**[CLIENTE]**, como root:

```bash
showmount -e 10.100.0.10                         # debe mostrar /mnt/nfs 10.100.0.0/16
mkdir /mnt/comunNFS
mount -t nfs 10.100.0.10:/mnt/nfs /mnt/comunNFS
df -h
touch /mnt/comunNFS/prueba                       # Tarea 16: debe fallar
umount /mnt/comunNFS
```

**Respuesta a la Tarea 16:** root del cliente **no puede** crear archivos (`Read-only file system`) porque la exportación es de solo lectura (`ro`). `no_root_squash` solo hace que root del cliente sea root también en el servidor; no anula el modo de solo lectura.

## 3.5 Carpetas y ACL (Tarea 17)

**[SERVER]**

1. Pasa la exportación a lectura y escritura. Sin `rw` nadie escribiría aunque las ACL lo permitan:

```bash
   nano /etc/exports
```

```text
   /mnt/nfs 10.100.0.0/16(rw,sync,no_root_squash,no_subtree_check)
```

```bash
   exportfs -ra
```
2. Crea las carpetas y aplica las ACL:

```bash
   mkdir /mnt/nfs/profesores /mnt/nfs/alumnos
   setfacl -m g:profesores:rwx /mnt/nfs/profesores
   setfacl -m g:profesores:rwx /mnt/nfs/alumnos
   setfacl -m g:alumnos:rwx   /mnt/nfs/alumnos
   setfacl -m g:alumnos:---   /mnt/nfs/profesores
```

| Grupo | `/mnt/nfs/profesores` | `/mnt/nfs/alumnos` |
| :--- | :---: | :---: |
| `profesores` | escribe | escribe |
| `alumnos` | sin acceso | escribe |

## 3.6 Montaje automático con `fstab` (Tarea 18)

**[CLIENTE]**

```bash
mkdir -p /mnt/comunNFS
nano /etc/fstab
```

```text
10.100.0.10:/mnt/nfs  /mnt/comunNFS  nfs  defaults  0 0
```

```bash
mount -a
df -h
reboot
```

## 3.7 Pruebas con `julian` y `alumno` (Tareas 19 y 20)

**[CLIENTE]:** inicia sesión con cada usuario y ejecuta:

```bash
# como julian
touch /mnt/comunNFS/profesores/prueba-julian
touch /mnt/comunNFS/alumnos/prueba-julian

# como alumno
touch /mnt/comunNFS/alumnos/prueba-alumno
cd /mnt/comunNFS/profesores                     # debe dar "Permission denied"
```

Cierra la sesión al terminar cada usuario.

## 3.8 Montaje con `libpam-mount` (Tarea 21)

**[CLIENTE]**, como root:

1. Quita el montaje de `fstab`:

```bash
   umount /mnt/comunNFS
   nano /etc/fstab                  # borra la línea de /mnt/comunNFS
   systemctl daemon-reload
```
2. Configura `libpam-mount`:

```bash
   nano /etc/security/pam_mount.conf.xml
```

   Añade **encima de `</pam_mount>`**:

```xml
   <volume gid="1501" fstype="nfs" server="10.100.0.10" path="/mnt/nfs" mountpoint="~/NFS" />
   <volume gid="1502" fstype="nfs" server="10.100.0.10" path="/mnt/nfs" mountpoint="~/NFS" />
```

   > Debe existir (sin comentar) `<mkmountpoint enable="1" remove="true" />`, que crea `~/NFS` al iniciar sesión.
3. Cierra sesión e inicia con `julian` **desde la pantalla gráfica** (con `su` o por SSH no se ejecuta `pam_mount`). Repite con `alumno`.

---

## ✅ Comprobación / Verificación

**[SERVER]**

```bash
exportfs -v                                      # /mnt/nfs 10.100.0.0/16(rw,...,no_root_squash)
getfacl /mnt/nfs/profesores                      # group:profesores:rwx y group:alumnos:---
getfacl /mnt/nfs/alumnos
id julian && id alumno                           # uid/gid 1501 y 1502
```

**[CLIENTE]**

```bash
showmount -e 10.100.0.10
id julian && id alumno                           # mismos números que en el servidor
mount | grep NFS                                 # tras iniciar sesión gráfica con julian o alumno
```

| Usuario | Carpeta | Entrar | Crear archivo |
| :--- | :--- | :---: | :---: |
| `julian` | `profesores` | ✅ | ✅ |
| `julian` | `alumnos` | ✅ | ✅ |
| `alumno` | `alumnos` | ✅ | ✅ |
| `alumno` | `profesores` | ❌ | ❌ |

Si algo no coincide, consulta [registro-errores.md](registro-errores.md#-nfs-y-acl).

---
⬅️ [Anterior](02-discos-y-particiones.md) | 🏠 [Volver al Índice](README.md) | [Siguiente](04-samba-y-pam.md) ➡️