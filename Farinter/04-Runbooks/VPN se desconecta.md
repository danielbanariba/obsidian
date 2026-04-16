---
tags: [runbook, vpn, infraestructura]
sistema: vpn
severidad: media
ultima_ocurrencia: 2026-04-05
---

# VPN se desconecta

## Sintomas
- El watchdog no puede conectar a Dagster (172.16.2.220)
- El bot de Telegram no responde consultas de datos
- `systemctl status openvpn-client@farinter` muestra `inactive (dead)`

## Resolución
```bash
# Reiniciar servicio
sudo systemctl restart openvpn-client@farinter

# Si falla, conectar manual
cd ~/Documents/VPN && sudo openvpn --config client.ovpn
```

## Configuración del servicio automático
```bash
# Archivos en /etc/openvpn/client/
# farinter.conf, update-dns.sh, credentials.txt, client.crt, client.pem

# Habilitar auto-start
sudo systemctl enable openvpn-client@farinter
```

## Historial
| Fecha | Qué pasó | Resolución |
|-------|----------|------------|
| 2026-04-05 | VPN caída 5h, watchdog sin acceso | Reinicio manual |
| 2026-04-04 | Configurado como systemd service | Auto-start habilitado |
