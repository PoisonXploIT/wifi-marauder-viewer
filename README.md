# wifi-marauder-viewer
# Visualizador WiFi M5StickC + Marauder

Este proyecto permite visualizar en tiempo real, desde una interfaz web, los resultados del escaneo WiFi realizado por un dispositivo **M5StickC** con el firmware **WiFi Marauder**.  
Los datos se recogen mediante **conexión serial** (PuTTY), se guardan en un archivo `.log`, y se muestran a través de un **servidor web en Flask**.

---

## Requisitos

- M5StickC flasheado con [WiFi Marauder](https://github.com/justcallmekoko/ESP32Marauder)
- PuTTY (o similar) para leer el puerto COM y generar el log
- Python 3 instalado
- Biblioteca Flask (`pip install flask`)

---

Configuración y uso

1. **Conecta el M5StickC por USB**  
   Asegúrate de tener PuTTY configurado con:
   - Puerto COM correcto (`CH9102`, `CP2102`, etc.) COM3 COM4 etc...
   - Baud rate: **115200**
   - Flow control: **XON/XOFF**

2. **Abre PuTTY y activa el log:**
   - En la sección *Logging*, selecciona:
     - `All session output`
     - Ruta de archivo: por ejemplo `E:\LOGS WIFI\wifi_scan_marauder.log`
   - Pulsa **Open** y reinicia el M5StickC para que empiece a emitir datos.

3. **Ejecuta el servidor Flask**

   Asegúrate de editar en el script `wifi_web.py` la ruta al archivo `.log`:

   Ejemplo de Ruta: LOG_FILE = r"E:\LOGS WIFI\wifi_scan_marauder.log"

4. Luego abre PowerShell o CMD en la carpeta del proyecto y ejecuta:
python wifi_web.py
Abre tu navegador en http://127.0.0.1:5000

Verás la interfaz web con una tabla que se actualiza cada 5 segundos mostrando:

Canal (Ch) RSSI BSSID ESSID

--------------------------CODIGO PYTHON------------------------------------

from flask import Flask, render_template_string
import re

# RUTA al archivo de log generado por PuTTY
LOG_FILE = r"E:\LOGS WIFI\wifi_scan_marauder.log"

app = Flask(__name__)

TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>WiFi Scan Log</title>
    <meta http-equiv="refresh" content="5">
    <style>
        body { font-family: monospace; background: #111; color: #0f0; padding: 1em; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #0f0; padding: 4px; text-align: left; }
        th { background-color: #0f0; color: #111; }
    </style>
</head>
<body>
    <h1>📡 Escaneo WiFi (M5StickC + Marauder)</h1>
    <table>
        <tr><th>Canal</th><th>RSSI</th><th>BSSID</th><th>ESSID</th></tr>
        {% for ch, rssi, bssid, essid in data %}
        <tr><td>{{ ch }}</td><td>{{ rssi }}</td><td>{{ bssid }}</td><td>{{ essid }}</td></tr>
        {% endfor %}
    </table>
    <p>Actualiza cada 5 segundos.</p>
</body>
</html>
"""

@app.route("/")
def index():
    data = []
    try:
        with open(LOG_FILE, encoding="utf-8") as f:
            for line in f:
                # Regex que busca las partes del log correctamente
                match = re.search(
                    r"RSSI:\s*(-?\d+)\s+Ch:\s*(\d+)\s+BSSID:\s*([0-9a-f:]+)\s+.*ESSID:\s*(.+)", 
                    line.strip(), 
                    re.IGNORECASE
                )
                if match:
                    rssi, ch, bssid, essid = match.groups()
                    data.append((ch, rssi, bssid, essid))
    except FileNotFoundError:
        data.append(("?", "?", "Archivo no encontrado", "?"))
    return render_template_string(TEMPLATE, data=data)

if __name__ == "__main__":
    app.run(debug=False)
