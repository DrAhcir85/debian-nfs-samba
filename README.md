# 🖥️ Proyecto: Servidor de Archivos en Linux (NFS y Samba)

Este repositorio contiene la documentación técnica y el despliegue paso a paso de una infraestructura cliente-servidor basada en Debian. El objetivo principal es la provisión de almacenamiento en red mediante NFS y Samba, asegurado mediante Listas de Control de Acceso (ACLs) y montajes automatizados.

## 🏗️ Arquitectura del Laboratorio

La práctica se desarrolla sobre dos máquinas virtuales en VirtualBox conectadas mediante una red NAT privada:

| Rol | Hostname | Sistema Operativo | IP Estática | Disco Adicional |
| :--- | :--- | :--- | :--- | :--- |
| **Servidor** | `eq-srv` | Debian (Sin entorno gráfico) | 10.100.0.10/16 | 4 GB (sdb) |
| **Cliente** | `eq-cliente`| Debian (Con entorno gráfico) | 10.100.0.20/16 | - |

*Red NAT: 10.100.0.0/16 | DNS: 8.8.8.8 y 1.1.1.1*

## 📚 Índice de Documentación

El despliegue está dividido en fases operativas. Haz clic en cada enlace para acceder a los comandos ejecutados, configuraciones y explicaciones detalladas de cada tarea:

*   [1. Configuración Base y Acceso Remoto SSH](01-ssh-y-red.md) *(Tareas 1 a 8)*
*   [2. Gestión de Discos, Particiones y FSTAB](02-discos-y-particiones.md) *(Tarea 9)*
*   [3. Despliegue de NFS y Permisos ACL](03-nfs-y-acls.md) *(Tareas 10 a 21)*
*   [4. Despliegue de Samba y Montajes con libpam-mount](04-samba-y-pam.md) *(Tareas 22 a 34)*

## 🛠️ Directorio de Configuraciones y Soluciones
*   📁 `/config-backups`: Contiene las copias de seguridad limpias de los archivos de configuración (`/etc/fstab`, `/etc/exports`, `/etc/samba/smb.conf`).
*   📄 [Registro de Errores (Troubleshooting)](registro-errores.md): Bitácora de problemas encontrados durante el despliegue y sus respectivas soluciones.












--------------------------------------------------------
NUEVO

# 🖥️ debian-servicios: servidor de archivos con NFS y Samba en Debian

Laboratorio de administración de sistemas con dos máquinas virtuales Debian en VirtualBox. El servidor (`eq-srv`) comparte carpetas por **NFS** y por **Samba**, protegidas con **ACL**. El cliente (`eq-cliente`) las monta de forma manual, automática (`/etc/fstab`) y al iniciar sesión (`libpam-mount`).

> ⚠️ **Laboratorio de pruebas.** Algunas configuraciones (acceso root por SSH temporal, `no_root_squash`, invitados en Samba, `chmod 777`) no son adecuadas para producción.

## 🗺️ Escenario

| Rol | Hostname | Sistema | IP / máscara | Gateway | DNS | Disco extra | Servicios |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **[SERVER]** | `eq-srv` | Debian sin entorno gráfico | `10.100.0.10/16` | `10.100.0.1` | `8.8.8.8`, `1.1.1.1` | 4 GB (`/dev/sdb`, dos particiones de 2 GB) | SSH, NFS, Samba |
| **[CLIENTE]** | `eq-cliente` | Debian con entorno gráfico | `10.100.0.20/16` | `10.100.0.1` | `8.8.8.8`, `1.1.1.1` | - | Cliente SSH, NFS, CIFS, `libpam-mount` |

Red: **Red NAT de VirtualBox `10.100.0.0/16`** (máscara `255.255.0.0`).

| Usuario / grupo | UID / GID | Dónde | Función |
| :--- | :--- | :--- | :--- |
| `jefe` | - | cliente | Administra con `sudo` y entra por SSH como root al servidor |
| `profesores` (grupo) | GID 1501 | servidor y cliente | Escritura en `profesores` y `alumnos` |
| `alumnos` (grupo) | GID 1502 | servidor y cliente | Escritura solo en `alumnos` |
| `julian` | UID 1501, grupo `profesores` | servidor y cliente | Usuario de pruebas |
| `alumno` | UID 1502, grupo `alumnos` | servidor y cliente | Usuario de pruebas |

| Recurso | Partición | Servidor | Cliente |
| :--- | :--- | :--- | :--- |
| NFS | `/dev/sdb1` | `/mnt/nfs` | `/mnt/comunNFS` y `~/NFS` |
| Samba (`comunSMB`) | `/dev/sdb2` | `/mnt/smb` | `/mnt/comunSMB` y `~/SMB` |

## 🔌 Topología

```text
                    Internet
                       │
          ┌────────────┴────────────┐
          │  Red NAT 10.100.0.0/16  │  gateway 10.100.0.1
          └─────┬──────────────┬────┘
                │              │
       ┌────────┴───────┐ ┌────┴─────────────┐
       │ [SERVER]       │ │ [CLIENTE]        │
       │ eq-srv         │ │ eq-cliente       │
       │ 10.100.0.10/16 │ │ 10.100.0.20/16   │
       │ SSH · NFS ·    │ │ ssh · nfs-common │
       │ Samba          │ │ cifs · pam_mount │
       │ sdb1 → /mnt/nfs│ └──────────────────┘
       │ sdb2 → /mnt/smb│
       └────────────────┘
```

## 🌳 Estructura del repositorio

```text
debian-servicios/
├── README.md
├── 01-ssh-y-red.md
├── 02-discos-y-particiones.md
├── 03-nfs-y-acls.md
├── 04-samba-y-pam.md
├── registro-errores.md
└── config-backups/
    ├── server/
    │   ├── Interfaces.md
    │   ├── smb.conf.md
    │   ├── exports.md
    │   ├── fstab.md
    │   └── sshd_config.md
    └── cliente/
        ├── Interfaces.md
        ├── fstab.md
        └── pam_mount.conf.xml.md
```

## 📚 Índice de navegación

### Guías paso a paso

| Nº | Guía | Contenido |
| :---: | :--- | :--- |
| 1 | [01-ssh-y-red.md](01-ssh-y-red.md) | VirtualBox, nombres, red NAT, repositorios, usuario `jefe`, SSH |
| 2 | [02-discos-y-particiones.md](02-discos-y-particiones.md) | Particiones, ext4, montaje manual y automático por UUID |
| 3 | [03-nfs-y-acls.md](03-nfs-y-acls.md) | Usuarios, exportación NFS, ACL, `fstab` y `libpam-mount` |
| 4 | [04-samba-y-pam.md](04-samba-y-pam.md) | Recurso `comunSMB`, usuarios Samba, ACL y `libpam-mount` |
| ⚠️ | [registro-errores.md](registro-errores.md) | Errores habituales, diagnóstico y solución |

### Copias de configuración (`config-backups/`)

| Máquina | Archivo | Ruta real en Debian |
| :--- | :--- | :--- |
| Servidor | [Interfaces.md](config-backups/server/Interfaces.md) | `/etc/network/interfaces` |
| Servidor | [sshd_config.md](config-backups/server/sshd_config.md) | `/etc/ssh/sshd_config` |
| Servidor | [fstab.md](config-backups/server/fstab.md) | `/etc/fstab` |
| Servidor | [exports.md](config-backups/server/exports.md) | `/etc/exports` |
| Servidor | [smb.conf.md](config-backups/server/smb.conf.md) | `/etc/samba/smb.conf` |
| Cliente | [Interfaces.md](config-backups/cliente/Interfaces.md) | `/etc/network/interfaces` (red gestionada por NetworkManager) |
| Cliente | [fstab.md](config-backups/cliente/fstab.md) | `/etc/fstab` |
| Cliente | [pam_mount.conf.xml.md](config-backups/cliente/pam_mount.conf.xml.md) | `/etc/security/pam_mount.conf.xml` |

## 🚀 Cómo usar este repositorio

1. Sigue las guías en orden, de la `01` a la `04`.
2. Cada comando indica si va en `[SERVER]` o en `[CLIENTE]`.
3. Si algo falla, busca el síntoma en [registro-errores.md](registro-errores.md).
4. Las contraseñas y la *passphrase* de la práctica **no** están escritas: usa las que indica el enunciado.

---
🏠 Estás en el índice | [Empezar: 01-ssh-y-red.md](01-ssh-y-red.md) ➡️