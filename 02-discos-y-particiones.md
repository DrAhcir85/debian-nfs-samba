# 2. Gestión de Discos y Particiones (Tarea 9)

## Particionado y Formateo en el Servidor
*   Se particiona el segundo disco (`/dev/sdb`) en dos particiones primarias de 2GB usando `cfdisk`.
*   Se formatean en ext4: `mkfs.ext4 /dev/sdb1` y `mkfs.ext4 /dev/sdb2`.
*   Se crean puntos de montaje: `mkdir -p /mnt/nfs /mnt/smb`.

## Montaje Automático (Resistente a cambios de puerto)
Para evitar fallos si cambian los discos SATA, obtenemos los UUID con el comando `blkid` y configuramos el montaje persistente en `/etc/fstab`:
``text
UUID=xxxx-xxxx-xxxx /mnt/nfs ext4 defaults 0 2
UUID=yyyy-yyyy-yyyy /mnt/smb ext4 defaults 0 2


------------------------------------------------------------------------------------------------------------------------------------

## Paso a paso

Instalar fdisk - apt install fdisk

## Crear particiones:
    fdisk /dev/sdb
    nueva - primaria/extendida - Valores (OJO con extendidas en numeración) - guardar W

## Formatear particiones:
    mkfs.ext4 /dev/sdbX

## Montar/desmontar particiones
    mount /dev/sdb1 /mnt/X
    df -h   -   muestra particiones para verificar
    umount /mnt/X /mnt/X

## Montaje automático por si los discos cambian de puerto:
    volcado - blkid /deb/sdbx /dev/sdbX >> /etc/fstab
    Eliminar antes de UUID y despues de comillas
    UUID=XXXXX  /mnt/X ext4 defaults 0 0


--------------------------------------------------------------
NUEVO


# 2. Discos y particiones

**Objetivo:** dividir el segundo disco del servidor en dos particiones primarias ext4 de unos 2 GB, montarlas en `/mnt/nfs` y `/mnt/smb` y conseguir que se monten solas al arrancar aunque cambien los discos de puerto.

**Configuración de referencia:** [server/fstab.md](config-backups/server/fstab.md)

> Todo este bloque se hace en **[SERVER]** por SSH (`ssh root@10.100.0.10`).

---

## 2.1 Crear las particiones (Tarea 9)

**[SERVER]**

```bash
lsblk                            # confirma que el disco nuevo es sdb y está vacío
fdisk /dev/sdb
```

Dentro de `fdisk` (pulsa Enter tras cada línea):

| Partición 1 | Partición 2 |
| :--- | :--- |
| `n` (nueva) | `n` (nueva) |
| `p` (primaria) | `p` (primaria) |
| `1` | `2` |
| Enter (primer sector) | Enter (primer sector) |
| `+2G` | Enter (último sector: el resto, ≈ 2 GB) |

Después `p` para revisar y `w` para guardar y salir.

> Si falta la herramienta: `apt install fdisk`. No elijas `e` (extendida): el enunciado pide dos **primarias**.

## 2.2 Formatear en ext4

**[SERVER]**

```bash
mkfs.ext4 /dev/sdb1
mkfs.ext4 /dev/sdb2
```

## 2.3 Directorios y prueba de montaje manual

**[SERVER]**

```bash
mkdir -p /mnt/nfs /mnt/smb
mount /dev/sdb1 /mnt/nfs
mount /dev/sdb2 /mnt/smb
df -h                            # las dos deben aparecer montadas
umount /mnt/nfs /mnt/smb
```

## 2.4 Montaje automático con UUID

Se usa el **UUID** y no `/dev/sdb1` para que el montaje siga funcionando aunque los discos cambien de puerto o de nombre.

**[SERVER]**

```bash
blkid /dev/sdb1 /dev/sdb2        # copia los dos UUID
nano /etc/fstab
```

Añade al final (con **tus** UUID, sin comillas):

```text
UUID=xxxx-xxxx-xxxx  /mnt/nfs  ext4  defaults  0 2
UUID=yyyy-yyyy-yyyy  /mnt/smb  ext4  defaults  0 2
```

**Antes de reiniciar**, valida el archivo:

```bash
mount -a
findmnt --verify
df -h
```

> 💡 Haz una instantánea de la máquina antes de reiniciar. Un `fstab` mal escrito puede impedir el arranque normal.

## 2.5 Reiniciar

**[SERVER]**

```bash
reboot
```

El enunciado dice "reinicia el equipo cliente", pero el `fstab` modificado es el del servidor. Reinicia el servidor y, si tienes dudas, también el cliente. Espera 1 o 2 minutos antes de reconectar por SSH.

---

## ✅ Comprobación / Verificación

**[SERVER]**

```bash
lsblk -f                         # sdb1 y sdb2 en ext4, con sus UUID
df -h                            # /mnt/nfs y /mnt/smb montados automáticamente
findmnt /mnt/nfs
findmnt /mnt/smb
```

| Comprobación | Resultado esperado |
| :--- | :--- |
| `df -h` tras reiniciar | `/dev/sdb1` en `/mnt/nfs` y `/dev/sdb2` en `/mnt/smb` |
| `mount -a` | Sin mensajes de error |
| `findmnt --verify` | Sin errores en `fstab` |

---
⬅️ [Anterior](01-ssh-y-red.md) | 🏠 [Volver al Índice](README.md) | [Siguiente](03-nfs-y-acls.md) ➡️