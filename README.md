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

