# Smart-City-Traffic-Management-System-Design
To regroup everything into a **single cohesive project**, I'll organize the files and folders into a well-structured project directory. Below is the complete structure and implementation of the **Smart City Traffic Management System** project.

---

## Project Structure
```
smart-city-traffic/
├── iot/
│   ├── sensor_network.py          # Simulates ESP32 and Raspberry Pi
│   ├── traffic_control_system.py  # Adaptive signal control logic
├── data_processing/
│   ├── data_processor.py          # Kafka and Node-RED integration
├── dashboard/
│   ├── app.py                     # Flask backend for the dashboard
│   ├── templates/
│   │   ├── index.html             # Frontend HTML for the dashboard
│   ├── static/
│   │   ├── styles.css             # CSS for the dashboard
├── config/
│   ├── main_config.py             # Configuration settings
├── requirements.txt               # Python dependencies
├── README.md                      # Project documentation
```

---

## Step-by-Step Implementation

### 1. IoT Sensor Network Simulation

#### `iot/sensor_network.py`
This script simulates ESP32 and Raspberry Pi devices collecting traffic data and sending it to the cloud via MQTT.

```python
import paho.mqtt.client as mqtt
import random
import time

# MQTT Configuration
MQTT_BROKER = "mqtt.eclipseprojects.io"
MQTT_TOPIC = "smart-city/traffic"

# Simulate sensor data
def simulate_sensor_data():
    return {
        "vehicle_count": random.randint(0, 100),
        "speed": random.randint(0, 60),
        "timestamp": time.time(),
    }

# MQTT Client
client = mqtt.Client()
client.connect(MQTT_BROKER)

# Publish data
while True:
    data = simulate_sensor_data()
    client.publish(MQTT_TOPIC, str(data))
    print(f"Published: {data}")
    time.sleep(5)  # Send data every 5 seconds
```

---

### 2. Data Processing with Apache Kafka and Node-RED

#### `data_processing/data_processor.py`
This script consumes data from Kafka and processes it.

```python
from kafka import KafkaConsumer
import json

# Kafka Configuration
KAFKA_TOPIC = "traffic-data"
KAFKA_BROKER = "localhost:9092"

# Kafka Consumer
consumer = KafkaConsumer(
    KAFKA_TOPIC,
    bootstrap_servers=KAFKA_BROKER,
    value_deserializer=lambda x: json.loads(x.decode("utf-8")),
)

# Process data
for message in consumer:
    data = message.value
    print(f"Processing: {data}")
    # Add your data processing logic here
```

---

### 3. Cloud Integration with AWS IoT Core

#### `iot/sensor_network.py` (Updated for AWS IoT Core)
Update the MQTT client to send data to AWS IoT Core.

```python
from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient
import json
import random
import time

# AWS IoT Core Configuration
ENDPOINT = "your-aws-iot-endpoint"
CLIENT_ID = "smart-city-traffic"
TOPIC = "smart-city/traffic"

# AWS IoT MQTT Client
client = AWSIoTMQTTClient(CLIENT_ID)
client.configureEndpoint(ENDPOINT, 8883)
client.configureCredentials("root-CA.pem", "private-key.pem", "certificate.pem")
client.connect()

# Simulate sensor data
def simulate_sensor_data():
    return {
        "vehicle_count": random.randint(0, 100),
        "speed": random.randint(0, 60),
        "timestamp": time.time(),
    }

# Publish data
while True:
    data = simulate_sensor_data()
    client.publish(TOPIC, json.dumps(data), 1)
    print(f"Published to AWS IoT Core: {data}")
    time.sleep(5)
```

---

### 4. Adaptive Traffic Signal Control

#### `iot/traffic_control_system.py`
This script implements adaptive traffic signal control logic.

```python
import random

def optimize_signal_timing(traffic_data):
    vehicle_count = traffic_data["vehicle_count"]
    if vehicle_count > 50:
        return "Extend green light"
    elif vehicle_count > 20:
        return "Normal timing"
    else:
        return "Reduce green light"

# Simulate traffic data
traffic_data = {"vehicle_count": random.randint(0, 100)}
signal_action = optimize_signal_timing(traffic_data)
print(f"Traffic Data: {traffic_data}, Action: {signal_action}")
```

---

### 5. Real-Time Monitoring Dashboard

#### `dashboard/app.py`
This script creates a Flask backend for the dashboard.

```python
from flask import Flask, render_template
import random
import time
import threading

app = Flask(__name__)

# Simulate real-time traffic data
traffic_data = {"vehicle_count": 0, "speed": 0}

def update_traffic_data():
    global traffic_data
    while True:
        traffic_data = {
            "vehicle_count": random.randint(0, 100),
            "speed": random.randint(0, 60),
        }
        time.sleep(5)

# Start background thread
threading.Thread(target=update_traffic_data, daemon=True).start()

@app.route("/")
def index():
    return render_template("index.html", data=traffic_data)

if __name__ == "__main__":
    app.run(debug=True)
```

#### `dashboard/templates/index.html`
This is the frontend HTML for the dashboard.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Traffic Dashboard</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='styles.css') }}">
</head>
<body>
    <h1>Smart City Traffic Dashboard</h1>
    <div id="traffic-data">
        <p>Vehicle Count: <span id="vehicle-count">{{ data.vehicle_count }}</span></p>
        <p>Average Speed: <span id="speed">{{ data.speed }}</span> km/h</p>
    </div>
    <script>
        // Update data every 5 seconds
        setInterval(async () => {
            const response = await fetch("/");
            const html = await response.text();
            const parser = new DOMParser();
            const doc = parser.parseFromString(html, "text/html");
            document.getElementById("vehicle-count").textContent = doc.getElementById("vehicle-count").textContent;
            document.getElementById("speed").textContent = doc.getElementById("speed").textContent;
        }, 5000);
    </script>
</body>
</html>
```

#### `dashboard/static/styles.css`
This is the CSS for the dashboard.

```css
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    color: #333;
    text-align: center;
    padding: 20px;
}

h1 {
    color: #007BFF;
}

#traffic-data {
    background-color: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    display: inline-block;
    margin-top: 20px;
}
```

---

### 6. Configuration File

#### `config/main_config.py`
This file contains configuration settings for the project.

```python
# MQTT Configuration
MQTT_BROKER = "mqtt.eclipseprojects.io"
MQTT_TOPIC = "smart-city/traffic"

# Kafka Configuration
KAFKA_TOPIC = "traffic-data"
KAFKA_BROKER = "localhost:9092"

# AWS IoT Core Configuration
AWS_IOT_ENDPOINT = "your-aws-iot-endpoint"
AWS_CLIENT_ID = "smart-city-traffic"
AWS_TOPIC = "smart-city/traffic"
```

---

### 7. Dependencies

#### `requirements.txt`
This file lists all Python dependencies.

```
paho-mqtt
kafka-python
AWSIoTPythonSDK
Flask
```

---

### 8. Run the Project

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Start the IoT sensor network:
   ```bash
   python iot/sensor_network.py
   ```

3. Start the data processor:
   ```bash
   python data_processing/data_processor.py
   ```

4. Run the dashboard:
   ```bash
   python dashboard/app.py
   ```

5. Access the dashboard at `http://localhost:5000`.

---

This project is now fully regrouped and ready for use! Let me know if you need further assistance. 🚀
