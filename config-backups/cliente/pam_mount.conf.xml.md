# Montaje al iniciar sesión con libpam-mount

**Ruta real:** `/etc/security/pam_mount.conf.xml` · **Máquina:** `[CLIENTE]`

Las líneas `<volume>` se añaden **al final del archivo, justo encima de `</pam_mount>`**.

## NFS (Tarea 21)

```xml
<!-- Usuarios con gid 1501 (profesores) -->
<volume gid="1501" fstype="nfs" server="10.100.0.10" path="/mnt/nfs" mountpoint="~/NFS" />
<!-- Usuarios con gid 1502 (alumnos) -->
<volume gid="1502" fstype="nfs" server="10.100.0.10" path="/mnt/nfs" mountpoint="~/NFS" />
```

## Samba (Tarea 34)

El enunciado dice "gid 2001 y 2001" (errata): se usan los GID correctos.

```xml
<volume gid="1501" fstype="cifs" server="10.100.0.10" path="comunSMB" mountpoint="~/SMB" />
<volume gid="1502" fstype="cifs" server="10.100.0.10" path="comunSMB" mountpoint="~/SMB" />
```

| Atributo | Significado |
| :--- | :--- |
| `gid` | Se monta solo para los usuarios cuyo grupo principal tiene ese número |
| `fstype` | `nfs` o `cifs` |
| `server` | IP del servidor |
| `path` | Carpeta exportada (NFS) o nombre del recurso (Samba) |
| `mountpoint` | Dónde se monta; `~` es la carpeta del usuario, así aparece como `NFS` o `SMB` en "Archivos" |

## Requisitos

- Que exista (sin comentar) `<mkmountpoint enable="1" remove="true" />`, que crea la carpeta al iniciar sesión.
- Para Samba, la contraseña de Samba del usuario debe ser igual a la de su inicio de sesión.
- Iniciar sesión **desde la pantalla gráfica**; con `su` o por SSH no se aplica `pam_mount`.
- Si no se monta: `pam-auth-update` (marca `libpam-mount`) y `journalctl -b | grep -i pam_mount`.

---
🏠 [Volver al README](../../README.md) | 📘 [Módulo 03: NFS y ACL](../../03-nfs-y-acls.md) | 📘 [Módulo 04: Samba y PAM](../../04-samba-y-pam.md)