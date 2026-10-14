# Exportaciones NFS

**Ruta real:** `/etc/exports` · **Máquina:** `[SERVER]`

## Estado final (a partir de la Tarea 17)

```text
# <carpeta>  <red>(opciones)   ← sin espacio entre la red y el paréntesis
/mnt/nfs 10.100.0.0/16(rw,sync,no_root_squash,no_subtree_check)
```

| Directiva | Significado |
| :--- | :--- |
| `/mnt/nfs` | Carpeta que se comparte |
| `10.100.0.0/16` | Todos los equipos de la red del laboratorio |
| `rw` | Lectura y escritura (necesario para que las ACL permitan escribir) |
| `sync` | El servidor confirma una escritura cuando ya está en disco |
| `no_root_squash` | El root del cliente actúa como root en el servidor |
| `no_subtree_check` | Desactiva la comprobación de subárboles (más fiable) |

## Estado inicial (Tareas 13 a 16)

```text
/mnt/nfs 10.100.0.0/16(ro,sync,no_root_squash,no_subtree_check)    # ro = solo lectura
```

Aplicar los cambios y comprobar:

```bash
exportfs -ra
exportfs -v
systemctl restart nfs-kernel-server
```

---
🏠 [Volver al README](../../README.md) | 📘 [Módulo 03: NFS y ACL](../../03-nfs-y-acls.md)