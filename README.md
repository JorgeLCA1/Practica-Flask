# Practica-Flask
Práctica: Servidor Flask con Dashboard del Clima usando Tailscale (VPN personal)

# Jorge Luis Castro Alvarado

-----

# Mini Dashboard del Clima con Flask

Este proyecto consiste en una aplicación web simple desarrollada con Python y el microframework Flask. La aplicación consulta en tiempo real la API de OpenWeatherMap para mostrar la temperatura actual, las condiciones climáticas y otros detalles de una ciudad específica en una interfaz web limpia y moderna.

El objetivo es crear un servicio ligero que pueda ser ejecutado en cualquier dispositivo (como una Raspberry Pi) y ser accesible a través de la red local o mediante una red privada como Tailscale.

-----

### Demostración en la Terminal (Asciinema)

A continuación, se muestra cómo se inicia el servidor de Flask desde la terminal. La grabación muestra el comando de ejecución y la salida que indica que el servidor está corriendo y listo para recibir conexiones.


[![asciicast](https://asciinema.org/a/A60uKf3Xvm31A68g57yYsSqeE.svg)](https://asciinema.org/a/A60uKf3Xvm31A68g57yYsSqeE)

-----

### Demostración Web (Loom)

En el siguiente video se puede ver el resultado final: cómo se ve la página web desde un navegador. La demostración muestra la interfaz, el ícono dinámico del clima, y cómo la información se presenta de manera clara en la tarjeta de clima.

https://www.loom.com/share/5e3cff5ffe95450aabed436c130c02de?sid=86591fec-b613-4549-b9be-14be369bde69

-----

### Código Fuente (`app.py`)

Todo el proyecto está contenido en un único archivo de Python. Incluye el servidor web, la lógica para consultar el API y el HTML con CSS integrado para generar la página.

```python
# ╔═══════════════════════════════════════════════════════════════╗
# ║         🌤 Mini Dashboard del Clima con Flask + Tailscale         ║
# ║         Lenguajes de Interfaz - TECNM / ITT - 2025           ║
# ║         Autor: [Jorge Luis Castro Alvarado]           ║
# ║         Descripción: Servidor Flask consultando OpenWeatherMap ║
# ╚═══════════════════════════════════════════════════════════════╝

from flask import Flask
import requests
import datetime # Para obtener la fecha actual

app = Flask(_name_)

# --- CONFIGURACIÓN ---
API_KEY = "8c9a24c4e3ee31a9ba26600b2b272659"
CITY = "Tijuana" # Puedes cambiar la ciudad aquí

@app.route("/")
def clima():
    # URL del API de OpenWeatherMap para obtener el clima actual
    url = f"http://api.openweathermap.org/data/2.5/weather?q={CITY}&appid={API_KEY}&units=metric&lang=es"
    
    try:
        data = requests.get(url).json()

        # Verificamos si la solicitud fue exitosa
        if data.get("cod") != 200:
            return f"""
                <html>
                    <head><title>Error</title></head>
                    <body style='font-family: sans-serif; text-align: center; margin-top: 50px;'>
                        <h1>Error al consultar el clima</h1>
                        <p>No se pudo encontrar la ciudad '{CITY}' o la API Key es incorrecta.</p>
                        <p>Mensaje: {data.get("message", "Error desconocido")}</p>
                    </body>
                </html>
            """

        # --- Extracción de datos del JSON ---
        temp = round(data["main"]["temp"])
        feels_like = round(data["main"]["feels_like"])
        humidity = data["main"]["humidity"]
        weather_desc = data["weather"][0]["description"].capitalize()
        weather_icon = data["weather"][0]["icon"]
        icon_url = f"http://openweathermap.org/img/wn/{weather_icon}@4x.png"
        
        # --- Lógica para el color de fondo ---
        # Cambia el color del fondo según la temperatura
        if temp < 10:
            color = "#a7c7e7"  # Azul pastel
        elif temp < 25:
            color = "#a8e6cf"  # Verde menta
        else:
            color = "#ffb347"  # Naranja pastel

        # --- Fecha y hora actual ---
        now = datetime.datetime.now()
        fecha = now.strftime("%A, %d de %B de %Y")


        # --- Estructura HTML y CSS ---
        # Se inyecta todo el estilo y la estructura en un solo string
        return f"""
        <!DOCTYPE html>
        <html lang="es">
        <head>
            <meta charset="UTF-g">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Clima en {CITY}</title>
            <style>
                /* Estilos generales del cuerpo de la página */
                body {{
                    background-color: {color};
                    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    height: 100vh;
                    margin: 0;
                    transition: background-color 0.5s ease;
                }}
                /* Estilo de la "tarjeta" que contiene la información */
                .weather-card {{
                    background: rgba(255, 255, 255, 0.7);
                    padding: 30px 40px;
                    border-radius: 20px;
                    box-shadow: 0 10px 25px rgba(0,0,0,0.15);
                    text-align: center;
                    backdrop-filter: blur(10px); /* Efecto de vidrio esmerilado */
                    border: 1px solid rgba(255, 255, 255, 0.2);
                }}
                /* Título de la ciudad */
                h1 {{
                    margin: 0;
                    font-size: 2.5em;
                    color: #333;
                }}
                /* Fecha */
                .date {{
                    color: #777;
                    margin-bottom: 20px;
                    font-size: 1em;
                }}
                /* Contenedor para el ícono y la temperatura principal */
                .temp-container {{
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    margin: 10px 0;
                }}
                /* Ícono del clima */
                .weather-icon {{
                    width: 150px;
                    height: 150px;
                    margin-right: -15px; /* Ajuste para acercar al texto */
                }}
                /* Temperatura principal */
                .temperature {{
                    font-size: 5em;
                    font-weight: bold;
                    color: #333;
                    margin: 0;
                }}
                /* Descripción del clima (ej. "Nubes dispersas") */
                .description {{
                    font-size: 1.5em;
                    color: #555;
                    margin-top: -10px;
                }}
                /* Contenedor para detalles adicionales */
                .details {{
                    margin-top: 25px;
                    font-size: 1.1em;
                    color: #666;
                }}
            </style>
        </head>
        <body>
            <div class="weather-card">
                <h1>{CITY}</h1>
                <p class="date">{fecha}</p>
                <div class="temp-container">
                    <img src="{icon_url}" alt="Icono del clima" class="weather-icon">
                    <p class="temperature">{temp}°</p>
                </div>
                <p class="description">{weather_desc}</p>
                <div class="details">
                    <span>Sensación: {feels_like}°C</span> | <span>Humedad: {humidity}%</span>
                </div>
            </div>
        </body>
        </html>
        """
    except requests.exceptions.RequestException as e:
        return f"<p>Error de conexión: {e}</p>"
    except KeyError:
        return "<p>Error: No se pudieron procesar los datos recibidos del API. ¿Es correcta la ciudad?</p>"


if _name_ == "_main_":
    app.run(host="0.0.0.0", port=5000, debug=True)
```
