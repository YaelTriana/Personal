
# Simulador de Vibración de Motor con Prometheus
## Autor: Yael Kristoph Triana Sanchez 

### 🧠 Justificación

Este proyecto simula la vibración de múltiples motores eléctricos en tiempo real, con el objetivo de monitorear su comportamiento mediante métricas expuestas a Prometheus. La solución está diseñada para entornos académicos y de laboratorio donde no se dispone de sensores físicos, pero se requiere validar flujos de monitoreo, visualización y exportación de datos.

La métrica `vibracion_motor` permite observar variaciones simuladas por motor, facilitando el análisis de patrones, la detección de anomalías y la integración con herramientas como Grafana o alertas automáticas. Esta simulación también se publica vía MQTT, lo que permite extender el sistema hacia dashboards distribuidos o brokers externos.

---

### ⚙️ Instalación de dependencias

Puedes instalar todos los paquetes necesarios ejecutando el siguiente script:

```bash
#!/bin/bash

echo "Instalando dependencias para el simulador..."
pip install prometheus_client paho-mqtt
echo "Listo. Puedes ejecutar el simulador con: python simulador_prometheus.py"
```

Guarda este script como `instalar_dependencias.sh` y ejecútalo con:

```bash
bash instalar_dependencias.sh
```

---

### 🚀 Ejecución del simulador

```bash
python simulador_prometheus.py
```

Al iniciar, el script te preguntará cuántos motores deseas simular. Por ejemplo:

```
¿Cuántos motores quieres simular? 4
```

Esto generará métricas para `M1`, `M2`, `M3`, `M4`, actualizadas cada segundo.

---

### 📊 Resultados obtenidos

Una vez que Prometheus está corriendo y scrapeando `localhost:8000`, puedes consultar la métrica en:

```
http://localhost:9090
```

Busca:

```
vibracion_motor
```

Verás series como:

```
vibracion_motor{motor_id="M1"} 0.87
vibracion_motor{motor_id="M2"} 1.12
...
```
<img width="1874" height="1039" alt="image" src="https://github.com/user-attachments/assets/e249d75b-a62d-4cee-941c-811a77156ca6" />

Cada serie representa la vibración simulada de un motor en tiempo real. Esto permite:

- Visualizar tendencias por motor
- Exportar datos para análisis externo
- Integrar con Grafana para dashboards personalizados

<img width="601" height="52" alt="image" src="https://github.com/user-attachments/assets/e3a6d80e-4f58-4839-a5e6-8c4ee35ef18e" />

## Script 
    import time
    import random
    import json
    import paho.mqtt.client as mqtt
    from prometheus_client import start_http_server, Gauge

    # --- Configuración MQTT ---
    MQTT_BROKER = "localhost"
    MQTT_PORT = 1883
    MQTT_TOPIC = "motores/vibracion"

    # --- Preguntar cuántos motores simular ---
    while True:
        try:
            cantidad = int(input("¿Cuántos motores quieres simular? "))
            if cantidad < 1:
                print("Por favor, ingresa un número mayor que cero.")
                continue
            break
        except ValueError:
            print("Entrada no válida. Ingresa un número entero.")

    # --- Generar lista de motores dinámicamente ---
    MOTORES = [f"M{i+1}" for i in range(cantidad)]

    # --- Métrica Prometheus con etiqueta por motor ---
    vibracion_metric = Gauge("vibracion_motor", "Nivel de vibración del motor", ["motor_id"])

    # --- Inicializar MQTT ---
    mqtt_client = mqtt.Client()
    mqtt_client.connect(MQTT_BROKER, MQTT_PORT, 60)

    # --- Servidor Prometheus ---
    start_http_server(8000)

    # --- Función para generar vibración aleatoria ---
    def generar_vibracion():
        return round(random.uniform(0.2, 1.5), 2)

    # --- Loop principal ---
    try:
        while True:
            timestamp = int(time.time())

            for motor_id in MOTORES:
                vibracion = generar_vibracion()

                # Publicar en MQTT
                payload = {
                    "motor_id": motor_id,
                    "timestamp": timestamp,
                    "vibracion": vibracion
                }
                mqtt_client.publish(MQTT_TOPIC, json.dumps(payload))

                # Actualizar métrica Prometheus
                vibracion_metric.labels(motor_id=motor_id).set(vibracion)

                print(f"[{timestamp}] {motor_id} → {vibracion} g")

            time.sleep(1)

    except KeyboardInterrupt:
        print("Simulación detenida.")
        mqtt_client.disconnect()
<img width="1232" height="190" alt="image" src="https://github.com/user-attachments/assets/f7940b79-abff-44fe-a96c-4abbd252cf11" />

## 🧪 Mini guía: Entornos virtuales en Bash

### 🔧 Crear un entorno virtual

```bash
python3 -m venv venv
```

Esto crea una carpeta `venv` con una instalación aislada de Python.

---

### 🚀 Activar el entorno

```bash
source venv/bin/activate
```

Verás que tu terminal cambia, indicando que estás dentro del entorno virtual.

---

### 📦 Instalar dependencias

```bash
pip install -r requirements.txt
```

O directamente:

```bash
pip install prometheus_client paho-mqtt
```

---

### 📁 Guardar dependencias

```bash
pip freeze > requirements.txt
```

Esto registra las versiones exactas de los paquetes instalados.

---

### ❌ Desactivar el entorno

```bash
deactivate
```

Vuelves al entorno global de tu sistema.
