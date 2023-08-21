# Manual para renovar certificados SSL

Enlistar los estatus de los timers del sistema con el siguiente comando:

````systemctl list-timers --all````

En el listado debemos buscar un servicio con el nombre de ```certbot.timer``` con status activo.

```Fri 2023-08-18 17:08:43 UTC 1h 5min left   Fri 2023-08-18 03:39:23 UTC 12h ago   certbot.timer    certbot.service```

En caso de que si exista el servicio en la lista de timers hacer caso omiso a los siguintes pasos, ya que en este momento se encuentra operando de forma automatica la renovación de certificados SSL.

Si por alguna razón no se encontro el servicio en la lista de timers debemos seguir los siguientes pasos. 

## Comprobación de archivos de configuración certbot.timer

El comando que lista el estatus de los timers del sistema toma la información de la siguinte ruta:

```
/etc/systemd/system/timers.target.wants
```
Aquí están todos los enlaces simbolicos que tiene la configuración de cada uno de los servicios que ejecutan una acción en un tiempo determinado.

En este folder podremos encontrar el enlace certbot.timer, en caso de que exista solo hay que revisar donde esta apuntando el enlace de certbot y pasar el siguinte comando para habilitar el servicio.

```sudo systemctl enable certbot.timer```

En caso de que no exista el enlace seguir los siguintes pasos:

## Crear y habilitar el servicio certbot.timer

Dentro de la siguiente ruta ```/lib/systemd/system```  buscaremos un par de archivos que sirven para habilitar el servicio de renovación de certbot

```ls /lib/systemd/system/certbot.*```

Output:

```certbot.service certbot.timer```


De no existir, crearemos manualmente el par de archivos con los siguinetes nombres:

```touch certbot.service certbot.timer```

El primer archivo que afectaremos sera ```certbot.service```  y dentro editamos con la siguiente configuración:

```
[Unit]
Description=Certbot
Documentation=file:///usr/share/doc/python-certbot-doc/html/index.html
Documentation=https://letsencrypt.readthedocs.io/en/latest/
[Service]
Type=oneshot
ExecStart=/usr/bin/certbot -q renew
PrivateTmp=true
```

*Nota*: Debemos asegurarnos que existe el archivo ```/usr/bin/certbot``` para que pueda realizar la renovación.

Ahora editamos el archivo certbot.timer con la siguiente configuración:

```
[Unit]
Description=Run certbot twice daily

[Timer]
OnCalendar=*-*-* 00,12:00:00
RandomizedDelaySec=43200
Persistent=true

[Install]
WantedBy=timers.target
```

Ahora dentro de la ruta ```/etc/systemd/system/timers.target.wants``` donde existe toda la configuración de todos los timers, crear el enlace simbolico que apunte ```certbot.timer``` de la ruta ```/lib/systemd/system/```.

```
ln -s /lib/systemd/system/certbot.timer /etc/systemd/system/timers.target.wants
```

Habilitamos el servicio con el siguiente comando:

```sudo systemctl enable certbot.timer```

Por ultmimo volver a listar los servicios de los timers con el comando:

````systemctl list-timers --all````

Y en este momento ya debe aparecer en la lista de timers.
