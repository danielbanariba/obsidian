 
## Cosas que necesito aprender

-Datahouse (el que mire yo en bases de datos 2 xd)
-powerBI
-PowerAutomed
-Orquestaciones


Activar la VPN

Activamos la vpn 
```
sudo openvpn --config client.ovpn
```

---
Para poder acceder a la carpeta compartida "Data Repo, Ventas" es con el siguiente comando:
```shell
sudo mount -t cifs "//10.0.4.157/data_repo" /mnt/data_repo \
          -o username=dbarrientos,domain=farinternet,vers=3.0
```
