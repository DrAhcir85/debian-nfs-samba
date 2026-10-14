# Red del cliente

**Ruta real:** `/etc/network/interfaces` · **Máquina:** `[CLIENTE]` (`eq-cliente`)

El cliente tiene entorno gráfico, así que **NetworkManager** gestiona la red. El archivo `interfaces` debe quedarse solo con el loopback; si se añade la interfaz de red, ambos sistemas entran en conflicto y la conexión se pierde al cabo de un minuto.

```ini
# Contenido mínimo: solo loopback
auto lo
iface lo inet loopback
```

## Configuración de red (herramienta gráfica de NetworkManager)

| Campo | Valor |
| :--- | :--- |
| Método IPv4 | Manual |
| Dirección | `10.100.0.20` |
| Máscara | `255.255.0.0` (`/16`) |
| Puerta de enlace | `10.100.0.1` |
| DNS | `8.8.8.8, 1.1.1.1` (con el DNS automático desactivado) |

## Alternativa por terminal

```bash
nmcli con show                       # anota el nombre de la conexión cableada
nmcli con mod "Wired connection 1" ipv4.method manual \
  ipv4.addresses 10.100.0.20/16 ipv4.gateway 10.100.0.1 \
  ipv4.dns "8.8.8.8 1.1.1.1" ipv4.ignore-auto-dns yes
nmcli con up "Wired connection 1"
```

> Sustituye `Wired connection 1` por el nombre real de tu conexión.

---
🏠 [Volver al README](../../README.md) | 📘 [Módulo 01: SSH y red](../../01-ssh-y-red.md)