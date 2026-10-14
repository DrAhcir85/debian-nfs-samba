# Red del servidor

**Ruta real:** `/etc/network/interfaces` · **Máquina:** `[SERVER]` (`eq-srv`)

```ini
# Interfaz de loopback
auto lo
iface lo inet loopback

# Interfaz conectada a la Red NAT (cambia enp0s3 si en tu máquina se llama distinto: ip -br link)
auto enp0s3                          # se activa al arrancar
allow-hotplug enp0s3                 # se activa también si aparece después
iface enp0s3 inet static             # IP fija
    address 10.100.0.10/16           # IP del servidor y máscara (255.255.0.0)
    gateway 10.100.0.1               # puerta de enlace de la Red NAT
    dns-nameservers 8.8.8.8 1.1.1.1  # DNS (en plural); requiere el paquete resolvconf
```

Aplicar los cambios: `systemctl restart networking`

---
🏠 [Volver al README](../../README.md) | 📘 [Módulo 01: SSH y red](../../01-ssh-y-red.md)