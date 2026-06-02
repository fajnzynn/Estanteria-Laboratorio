# Estantería automatizada para cultivo de hongos en investigación
> Scripts de pyhton ejecutables en Rpi que permiten controlar la temperatura, humedad y ventilación de una estantería de cultivo de hongos. Un servidor que corre en un Rpi3B+ y Rpis Zero W como mini servidores
> que se comunican con el 3B+ a través de HTTP.

---

## Arquitectura del Sistema

El sistema se divide en dos componentes principales:

1.  **Servidor (Raspberry Pi 3B+):** Centraliza la recepción de datos, los procesa y (opcionalmente) los almacena o muestra.
2.  **Cliente / Nodo (Raspberry Pi Zero W):** Se conecta físicamente a los sensores, realiza las lecturas y las envía al servidor a través de la red local.

```
[Pi Zero W]                          [Pi 3B+]
  DHT22 sensor                         Flask server
  Relé → ventiladores   ──HTTP──►      Dashboard web
  ACH + override logic                 CSV export
                                        Alertas Telegram
```

## Estructura del Repositorio

Para mantener la lógica ordenada y entender la evolución del proyecto, el repositorio está estructurado de la siguiente manera:

* `servidor/`: Código y scripts que se ejecutan en la Raspberry Pi 3B+.
* `cliente/`: Código destinado a la Raspberry Pi Zero W para la lectura de sensores.
* `precursores/`: Primeras pruebas de concepto, scripts primitivos de lectura de sensores en local y versiones iniciales antes de separar la lógica cliente-servidor.
* `requirements.txt`: Librerías de Python necesarias para correr el proyecto.

---
## Instalación

### En ambas Pis — copiar config.py
```bash
# Editá la IP del 3B+ en config.py antes de cualquier cosa
nano config.py
```

### Pi Zero W

```bash
sudo apt-get update
sudo apt-get install -y python3-pip libgpiod2
pip3 install adafruit-circuitpython-dht RPi.GPIO requests
```

Ejecutar:
```bash
python3 zero_w.py
```

Ejecutar como servicio (arranque automático):
```bash
sudo nano /etc/systemd/system/cultivo-zero.service
```
```ini
[Unit]
Description=Cultivo Zero W
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/pi/cultivo/zero_w.py
WorkingDirectory=/home/pi/cultivo
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl enable cultivo-zero
sudo systemctl start cultivo-zero
```

---

### Pi 3B+

```bash
pip3 install flask requests
```

Ejecutar:
```bash
python3 server_3b.py
```

Dashboard disponible en: `http://<IP_3B+>:5000`

Ejecutar como servicio:
```bash
sudo nano /etc/systemd/system/cultivo-server.service
```
```ini
[Unit]
Description=Cultivo Server 3B+
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/pi/cultivo/server_3b.py
WorkingDirectory=/home/pi/cultivo
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl enable cultivo-server
sudo systemctl start cultivo-server
```

---
## Endpoints del servidor (Pi 3B+)

| URL | Descripción |
|-----|-------------|
| `/` | Dashboard web en tiempo real |
| `/status` | Última lectura (JSON) |
| `/historico?n=120` | Últimas N lecturas en RAM (JSON) |
| `/hoy` | Promedios del día (JSON) |
| `/csv` | Descargar CSV completo |
| `/health` | Estado del sistema |

---

1. Clonar el repositorio:
```bash
   git clone [https://github.com/tu-usuario/tu-repositorio.git](https://github.com/tu-usuario/tu-repositorio.git)
