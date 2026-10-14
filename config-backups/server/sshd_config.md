# Configuración de SSH del servidor

**Ruta real:** `/etc/ssh/sshd_config` · **Máquina:** `[SERVER]`

Solo se muestran las directivas que se tocan. El resto del archivo se deja como está.

## Fase 1: copiar la clave a root (temporal)

```ini
# Permite a root entrar por SSH (también con contraseña). SOLO mientras se copia la clave.
PermitRootLogin yes

# Ya vienen así por defecto en Debian:
PubkeyAuthentication yes             # permite entrar con clave pública
PasswordAuthentication yes           # permite la contraseña (necesaria para ssh-copy-id)
```

## Fase 2: estado final (clave sí, contraseña de root no)

```ini
# Valor por defecto de Debian: root puede entrar con clave pero NO con contraseña.
# Equivale a dejar la línea original comentada (#PermitRootLogin prohibit-password).
PermitRootLogin prohibit-password
```

Aplicar los cambios:

```bash
systemctl restart ssh
sshd -t                              # comprueba la sintaxis antes de reiniciar
```

> Este laboratorio no usa PAM para SSH: no hay cambios en `/etc/pam.d/`. `libpam-mount` se configura en el cliente ([pam_mount.conf.xml.md](../cliente/pam_mount.conf.xml.md)).

---
🏠 [Volver al README](../../README.md) | 📘 [Módulo 01: SSH y red](../../01-ssh-y-red.md)