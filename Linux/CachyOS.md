![[Pasted image 20251117113049.png]]

---
## Comandos

Actualizar el sistema
```shell
sudo pacman -Syu
```

```shell
yay -Syu
```

Instalar un programa 
```shell
sudo pacman -U [nombre-del-archivo.pacman]
```

Instalar las extenciones de gnanome
```
sudo pacman -S gnome-browser-connector
```


Instalar todo con paro
Notion Calendar
```
paru -S notion-calendar-widget
```


RECOMENDACION!
nunca instalar .rpm o .deb, ya que el .rpm en fedora y el .deb en debian


---

desistalar cualquier programa
```
sudo pacman -Rns [nombre-del-programa]
```


Consultar como se llama el programa 
```
pacman -Qs [nombre-del-programa]
```

---

## Porgramas para instalar y volver a configurar si algo sale mal

qBitTorrent
```
sudo pacman -S qbittorrent
```

Obsidean:
```
sudo pacman -S obsidian
```

Postgresql
```
sudo pacman -S postgresql
```

Dbeader
```
sudo pacman -S dbeaver
```

jDownloader
```
sudo pacman -S jdownloader2
```



Comando universal
```
sudo pacman -S dbeaver postgresql obsidian qbittorrent 
```



---
# Python

Activar el entorno virtual
```shell
source env/bin/activate.fish
```

