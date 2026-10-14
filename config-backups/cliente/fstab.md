# Montajes permanentes del cliente

**Ruta real:** `/etc/fstab` · **Máquina:** `[CLIENTE]`

Las líneas se añaden **al final** del archivo. Cambian según la fase de la práctica.

## NFS (Tarea 18)

```text
# <servidor:carpeta>       <punto montaje>  <tipo>  <opciones>  <dump> <pass>
10.100.0.10:/mnt/nfs       /mnt/comunNFS    nfs     defaults    0      0
```

> En la **Tarea 21** esta línea se **borra** y el montaje pasa a `libpam-mount` ([pam_mount.conf.xml.md](pam_mount.conf.xml.md)).

## Samba con acceso de invitado (Tarea 26)

```text
//10.100.0.10/comunSMB     /mnt/comunSMB    cifs    guest,uid=1000,file_mode=0777,dir_mode=0777,iocharset=utf8  0  0
```

| Opción | Significado |
| :--- | :--- |
| `guest` | Entra sin usuario ni contraseña |
| `uid=1000` | Los archivos aparecen como propiedad del usuario 1000 |
| `file_mode`, `dir_mode` | Permisos que ve el cliente; con `0777` pueden escribir otros usuarios |
| `iocharset=utf8` | Juego de caracteres de los nombres |

## Samba con usuario `samba` (Tareas 28 y 29)

La línea de invitado se **comenta** con `#`. Si se quisiera montar con `fstab`, hay que dar las credenciales:

```text
#//10.100.0.10/comunSMB    /mnt/comunSMB    cifs    guest,uid=1000,file_mode=0777,dir_mode=0777,iocharset=utf8  0  0
//10.100.0.10/comunSMB     /mnt/comunSMB    cifs    credentials=/root/.smbcredentials,file_mode=0777,dir_mode=0777,iocharset=utf8  0  0
```

Archivo `/root/.smbcredentials` (permisos `chmod 600`):

```text
username=samba
password=CONTRASEÑA_DEL_ENUNCIADO
```

> Es más seguro usar `credentials=` que escribir `username=` y `password=` directamente en `fstab`, porque ese archivo lo puede leer cualquier usuario.

## Estado final (Tarea 34)

Ninguna línea de NFS ni de Samba queda activa en `fstab`: los dos montajes los hace `libpam-mount` al iniciar sesión.

Después de cada cambio: `systemctl daemon-reload` y `mount -a`.

---
🏠 [Volver al README](../../README.md) | 📘 [Módulo 03: NFS y ACL](../../03-nfs-y-acls.md) | 📘 [Módulo 04: Samba y PAM](../../04-samba-y-pam.md)