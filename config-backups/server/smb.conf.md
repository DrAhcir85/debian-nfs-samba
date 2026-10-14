# Configuración de Samba

**Ruta real:** `/etc/samba/smb.conf` · **Máquina:** `[SERVER]`

La sección `[global]` del archivo se deja como viene en Debian (incluye `map to guest = bad user`, que permite el acceso de invitado). El recurso se añade **al final** y evoluciona en tres fases.

## Fase 1: acceso libre para cualquier usuario (Tarea 23)

```ini
[comunSMB]                           # nombre del recurso que verá el cliente
   path = /mnt/smb                   # carpeta real del servidor
   browseable = yes                  # aparece en la lista de recursos
   writable = yes                    # permite escribir
   guest ok = yes                    # permite el acceso sin usuario ni contraseña
   create mask = 0777                # permisos de los archivos nuevos
   directory mask = 0777             # permisos de las carpetas nuevas
```

Requiere además `chmod 777 /mnt/smb`.

## Fase 2: solo el usuario `samba` (Tarea 28)

```ini
[comunSMB]
   path = /mnt/smb
   browseable = yes
   writable = yes
   guest ok = no                     # se retira el acceso de invitado
   valid users = samba               # solo este usuario puede entrar
   create mask = 0777
   directory mask = 0777
```

## Fase 3: estado final con `julian` y `alumno` (Tarea 31)

```ini
[comunSMB]
   path = /mnt/smb
   browseable = yes
   writable = yes
   guest ok = no
   valid users = julian, alumno      # usuarios dados de alta con smbpasswd -a
   create mask = 0777
   directory mask = 0777
```

## Variante de teoría (Tarea 33)

```ini
   writeable = no                    # solo lectura para todos...
   write list = @profesores          # ...salvo los miembros del grupo profesores
```

Aplicar y comprobar:

```bash
testparm -s
systemctl restart smbd
```

---
🏠 [Volver al README](../../README.md) | 📘 [Módulo 04: Samba y PAM](../../04-samba-y-pam.md)