## Registro de incidencia: errores de red y repositorios

1. Verificación inicial de red para instalación de paquetes
Antes de instalar resolvconf u otros componentes necesarios, es obligatorio comprobar que la máquina está conectada mediante red NAT.
Solo con conectividad activa se deben descargar los paquetes y continuar con la configuración de red.

2. Pérdida de conexión tras la configuración
Después de configurar la red, actualizar los repositorios y confirmar que existe conexión, se produce una pérdida total de red aproximadamente un minuto después.
Este comportamiento indica que la configuración aplicada no se mantiene o entra en conflicto con otro método de gestión de red.
En equipos que disponen de interfaz gráfica para gestionar la red, la configuración de la IP debe realizarse exclusivamente desde la herramienta gráfica, evitando modificar manualmente los ficheros del sistema.
Si se han realizado cambios manuales previamente, el procedimiento correcto es:
    Vaciar el contenido del fichero de configuración de interfaces.
    Reiniciar el servicio de red con:
    systemctl restart networking
    Configurar la red nuevamente desde la interfaz gráfica.
    Si el problema persiste, realizar un reinicio completo del sistema.

Error en cliente al montar carpeta compartida comunSMB, me decía que estaba en uso y me da error por fstab
Solucion - systemctl daemon-reload / umount /mnt/comunSMB


--------------------------------------------------------------
NUEVO

# Registro de errores (troubleshooting)

Fallos habituales en los cuatro bloques, con su causa, el comando de diagnóstico y la solución.

**Índice:** [Red y SSH](#-red-y-ssh) · [fstab y discos](#-fstab-y-discos) · [NFS y ACL](#-nfs-y-acl) · [Samba y PAM](#-samba-y-pam)

> 💡 **Orden de diagnóstico:** red (`ip -br addr`, `ping`) → servicio (`systemctl status`) → configuración (`exportfs -v`, `testparm -s`) → cliente (`showmount`, `smbclient`) → montaje (`df -h`) → permisos (`id`, `getfacl`) → logs (`journalctl -xe`).

---

## 🌐 Red y SSH

### 1. No hay ping al gateway ni a internet
- **Síntoma:** `ping 10.100.0.1` o `ping 8.8.8.8` sin respuesta.
- **Causa probable:** la máquina no está en la Red NAT correcta, o la IP/gateway son erróneos.
- **Diagnóstico:** `ip -br addr` · `ip route`
- **Solución:**
  1. En VirtualBox, comprueba `Adaptador 1` = **Red NAT** y la misma red en ambas máquinas.
  2. Revisa IP `/16` y gateway `10.100.0.1`.
  3. Servidor: `systemctl restart networking`. Cliente: apaga y enciende la conexión.

### 2. La red se pierde a los 60 segundos (cliente)
- **Síntoma:** funciona un minuto y luego se cae.
- **Causa probable:** conflicto entre `/etc/network/interfaces` y NetworkManager.
- **Diagnóstico:** `cat /etc/network/interfaces` · `nmcli dev status`
- **Solución:**
  1. Deja `/etc/network/interfaces` solo con la interfaz `lo` (ver [cliente/Interfaces.md](config-backups/cliente/Interfaces.md)).
  2. `systemctl restart networking`.
  3. Configura la IP desde la interfaz gráfica.
  4. Si persiste, `reboot`.

### 3. Responde `8.8.8.8` pero no `google.com`
- **Síntoma:** `Temporary failure in name resolution`.
- **Causa probable:** DNS no aplicado (falta `resolvconf` o `dns-nameservers` mal escrito).
- **Diagnóstico:** `cat /etc/resolv.conf`
- **Solución:**
  1. `apt install resolvconf` (necesita red).
  2. Comprueba que la línea es `dns-nameservers 8.8.8.8 1.1.1.1` (en plural).
  3. `systemctl restart networking` y repite `ping google.com`.

### 4. `apt update` da errores
- **Síntoma:** `repository 'cdrom://...' does not have a Release file` o `404`.
- **Causa probable:** queda la línea del CD o el nombre en clave es incorrecto.
- **Diagnóstico:** `cat /etc/os-release` · `cat /etc/apt/sources.list`
- **Solución:**
  1. Comenta con `#` las líneas `deb cdrom:`.
  2. Usa el `VERSION_CODENAME` de `/etc/os-release`.
  3. `apt update`.

### 5. SSH: `Permission denied (publickey)`
- **Síntoma:** no deja entrar como root con la clave.
- **Causa probable:** la clave no se copió, o se cerró `PermitRootLogin` antes de copiarla.
- **Diagnóstico:** `ssh -v root@10.100.0.10` · en el servidor `journalctl -u ssh -n 30`
- **Solución:**
  1. En el servidor: `PermitRootLogin yes` y `systemctl restart ssh`.
  2. En el cliente: `ssh-copy-id root@10.100.0.10`.
  3. Vuelve a dejar `PermitRootLogin` como estaba (ver [server/sshd_config.md](config-backups/server/sshd_config.md)).

### 6. SSH: `REMOTE HOST IDENTIFICATION HAS CHANGED`
- **Síntoma:** advertencia al conectar.
- **Causa probable:** el servidor se reinstaló o clonó y cambió su huella.
- **Diagnóstico:** el mensaje indica la línea de `~/.ssh/known_hosts`.
- **Solución:** `ssh-keygen -R 10.100.0.10` y vuelve a conectar.

### 7. SSH: `Connection refused` o `Connection timed out`
- **Causa probable:** `ssh` parado (refused) o sin red entre máquinas (timed out).
- **Diagnóstico:** `systemctl status ssh` · `ping 10.100.0.10` · `ss -tlnp | grep ':22 '`
- **Solución:**
  1. Sin red: corrige el caso 1.
  2. Con red: `systemctl enable --now ssh`.
  3. Tras un reinicio, espera 1 o 2 minutos.

---

## 💽 fstab y discos

### 8. La máquina no arranca bien tras editar `/etc/fstab`
- **Síntoma:** modo emergencia o espera larga al arrancar.
- **Causa probable:** línea mal escrita o UUID incorrecto.
- **Diagnóstico:** `journalctl -xb | grep -i mount` · `findmnt --verify`
- **Solución:**
  1. Entra como root en el modo emergencia (o restaura la instantánea).
  2. Comenta con `#` la línea problemática.
  3. `mount -a` hasta que no dé errores y `reboot`.
  4. Para evitarlo: ejecuta siempre `mount -a` antes de reiniciar.

### 9. `special device UUID=... does not exist`
- **Causa probable:** UUID mal copiado.
- **Diagnóstico:** `blkid /dev/sdb1 /dev/sdb2` · `lsblk -f`
- **Solución:** copia el UUID exacto, sin comillas, en la línea de `fstab` y ejecuta `mount -a`.

### 10. Las particiones no se montan tras reiniciar
- **Causa probable:** líneas comentadas, campos incompletos o punto de montaje inexistente.
- **Diagnóstico:** `mount -a` · `df -h` · `ls -ld /mnt/nfs /mnt/smb`
- **Solución:** comprueba que cada línea tiene 6 campos, que `/mnt/nfs` y `/mnt/smb` existen (`mkdir -p`) y ejecuta `mount -a`.

### 11. `fdisk` o `mkfs.ext4: command not found`
- **Causa probable:** entraste con `su` sin guion, o falta el paquete.
- **Solución:** usa `su -`, o `apt install fdisk`.

---

## 📂 NFS y ACL

### 12. `showmount -e` no lista nada o da error RPC
- **Síntoma:** `RPC: Program not registered`, `Port mapper failure` o lista vacía.
- **Causa probable:** NFS parado en el servidor o exportación sin aplicar.
- **Diagnóstico:** `systemctl status nfs-kernel-server` · `exportfs -v`
- **Solución:**
  1. Servidor: `exportfs -ra` y `systemctl restart nfs-kernel-server`.
  2. Comprueba la red con `ping 10.100.0.10`.
  3. Instala `nfs-common` en el cliente.

### 13. `access denied by server while mounting`
- **Causa probable:** el cliente no está dentro de `10.100.0.0/16`, o hay un espacio antes del paréntesis en `/etc/exports`.
- **Diagnóstico:** `cat /etc/exports` · `exportfs -v`
- **Solución:** deja la línea `/mnt/nfs 10.100.0.0/16(opciones)` sin espacios y ejecuta `exportfs -ra`.

### 14. `Read-only file system` al escribir
- **Causa probable:** la exportación sigue en `ro` (es lo correcto en las Tareas 13 a 16).
- **Diagnóstico:** `exportfs -v` en el servidor.
- **Solución:** para la Tarea 17 cambia `ro` por `rw` en `/etc/exports`, `exportfs -ra` y vuelve a montar en el cliente.

### 15. `Permission denied` a pesar de las ACL
- **Causa probable:** UID/GID distintos entre cliente y servidor, o ACL sin aplicar.
- **Diagnóstico:** `id julian` en ambas máquinas · `getfacl /mnt/nfs/profesores`
- **Solución:**
  1. Los números deben ser iguales (1501 y 1502). Si no, corrige con `usermod`/`groupmod` o recrea los usuarios.
  2. Reaplica el `setfacl` que corresponda y comprueba con `getfacl`.

### 16. `Stale file handle` o `target is busy`
- **Causa probable:** se cambió la exportación con la carpeta montada, o estás dentro de la carpeta.
- **Diagnóstico:** `mount | grep nfs` · `pwd`
- **Solución:** `cd /`, `umount -f /mnt/comunNFS` y vuelve a montar.

### 17. `libpam-mount` no monta `~/NFS`
- **Síntoma:** no aparece `NFS` al iniciar sesión.
- **Causa probable:** líneas fuera de `<pam_mount>`, `pam_mount` no activo, inicio de sesión sin pasar por PAM (`su`, SSH), o GID incorrecto.
- **Diagnóstico:** `journalctl -b | grep -i pam_mount` · `id julian` · `mount | grep NFS`
- **Solución:**
  1. Pon las líneas `<volume>` **antes** de `</pam_mount>`.
  2. Activa el módulo con `pam-auth-update` (marca `libpam-mount`).
  3. Comprueba que `<mkmountpoint enable="1" remove="true" />` no está comentada.
  4. Inicia sesión desde la pantalla gráfica.

---

## 🗄️ Samba y PAM

### 18. `mount error(13): Permission denied`
- **Causa probable:** invitados desactivados, usuario no válido o contraseña errónea.
- **Diagnóstico:** `testparm -s` · `pdbedit -L` · `journalctl -u smbd -n 30`
- **Solución:**
  1. Con invitados: `guest ok = yes` y `-o guest`.
  2. Con usuarios: revisa `valid users` y la contraseña de `smbpasswd`.
  3. `systemctl restart smbd`.

### 19. `mount: unknown filesystem type 'cifs'`
- **Causa probable:** falta `cifs-utils` en el cliente.
- **Solución:** `apt install cifs-utils`.

### 20. `NT_STATUS_HOST_UNREACHABLE` o `CONNECTION_REFUSED`
- **Causa probable:** `smbd` parado o sin red.
- **Diagnóstico:** `systemctl status smbd` · `ping 10.100.0.10`
- **Solución:** `systemctl restart smbd` y comprueba la red.

### 21. El invitado no puede escribir
- **Síntoma:** `touch: Permission denied` en `comunSMB`.
- **Causa probable:** `/mnt/smb` sin permiso de escritura (`create mask` solo afecta a lo que se crea dentro).
- **Diagnóstico:** `ls -ld /mnt/smb` en el servidor.
- **Solución:** `chmod 777 /mnt/smb`.

### 22. Otros usuarios no pueden escribir en el cliente
- **Causa probable:** con `uid=1000`, el cliente solo permite escribir a ese usuario.
- **Diagnóstico:** `ls -l /mnt/comunSMB` · `mount | grep cifs`
- **Solución:**
  1. Añade `file_mode=0777,dir_mode=0777` a las opciones de montaje.
  2. `umount /mnt/comunSMB` y `mount -a`.

### 23. `Failed to add entry for user samba` (o `julian`)
- **Causa probable:** el usuario no existe en Linux.
- **Diagnóstico:** `getent passwd samba`
- **Solución:** `useradd -M -s /usr/sbin/nologin samba` (o crea el usuario normal) y repite `smbpasswd -a`.

### 24. Error "en uso" al montar `comunSMB`
- **Causa probable:** ya está montada o `systemd` conserva la unidad antigua del `fstab`.
- **Diagnóstico:** `mount | grep comunSMB`
- **Solución:**
  1. `systemctl daemon-reload`
  2. `umount /mnt/comunSMB`
  3. `mount -a`

### 25. `libpam-mount` no monta `~/SMB`
- **Causa probable:** la contraseña de Samba no coincide con la de inicio de sesión, el usuario no está en la base de datos de Samba, o líneas fuera de `<pam_mount>`.
- **Diagnóstico:** `journalctl -b | grep -i pam_mount` · `pdbedit -L`
- **Solución:**
  1. `smbpasswd julian` con la misma contraseña que el inicio de sesión.
  2. Comprueba las líneas `<volume>` y los GID 1501 y 1502.
  3. Inicia sesión desde la pantalla gráfica.

---
⬅️ [Anterior](04-samba-y-pam.md) | 🏠 [Volver al Índice](README.md)