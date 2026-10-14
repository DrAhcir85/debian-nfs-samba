# Montaje de las particiones (servidor)

**Ruta real:** `/etc/fstab` · **Máquina:** `[SERVER]`

Se añaden **al final** del archivo. El resto de líneas no se toca.

```text
# <dispositivo>        <punto montaje>  <tipo>  <opciones>  <dump> <pass>
UUID=xxxx-xxxx-xxxx    /mnt/nfs         ext4    defaults    0      2    # sdb1: recurso NFS
UUID=yyyy-yyyy-yyyy    /mnt/smb         ext4    defaults    0      2    # sdb2: recurso Samba
```

| Campo | Significado |
| :--- | :--- |
| `UUID=...` | Identifica la partición aunque el disco cambie de puerto o de nombre |
| `/mnt/nfs`, `/mnt/smb` | Directorios donde se monta (deben existir: `mkdir -p`) |
| `ext4` | Sistema de archivos |
| `defaults` | Opciones estándar (`rw`, `suid`, `dev`, `exec`, `auto`...) |
| `0` | No se usa `dump` |
| `2` | `fsck` se ejecuta al arrancar, después del sistema raíz |

> Los UUID son de ejemplo. Los reales se obtienen con `blkid /dev/sdb1 /dev/sdb2` o `lsblk -f`.

Validar antes de reiniciar:

```bash
mount -a
findmnt --verify
```

---
🏠 [Volver al README](../../README.md) | 📘 [Módulo 02: discos y particiones](../../02-discos-y-particiones.md)