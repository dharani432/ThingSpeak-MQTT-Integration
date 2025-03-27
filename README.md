# ThingSpeak-MQTT-Integration
import random
import time
import paho.mqtt.client as mqtt

# --- Cloud MQTT configuration for ThingSpeak ---
TS_CHANNEL_ID = "2841467"          # Your ThingSpeak Channel ID
TS_USERNAME = "BS0ZNi0LDDUZHRwMBDsoOxE"            # ThingSpeak Write API Key (used as username)
TS_CLIENT_ID = "BS0ZNi0LDDUZHRwMBDsoOxE"           # Unique MQTT Client ID
TS_BROKER = "mqtt3.thingspeak.com"
TS_PASSWORD = "v1TXVyjwOT44dPjD3z/wils5"
TS_PORT = 1883
TS_TOPIC = f"channels/{TS_CHANNEL_ID}/publish"

is_connected = False  # Used to track connection status

# --- Function to simulate environmental sensor values ---
def generate_sensor_readings():
    temperature = round(random.uniform(-50, 50), 2)
    humidity = round(random.uniform(0, 100), 2)
    co2 = round(random.uniform(300, 2000), 2)
    return {
        "field1": temperature,
        "field2": humidity,
        "field3": co2
    }

# --- Callback for successful connection ---
def handle_connect(client, userdata, flags, rc):
    global is_connected
    if rc == 0:
        is_connected = True
        print("Connection successful.")
    else:
        print(f"Connection failed")

# --- Callback when message is published ---
def handle_publish(client, userdata, mid):
    print("Data Published")

# --- Configure and initialize the MQTT client ---
mqtt_client = mqtt.Client(client_id=TS_CLIENT_ID, protocol=mqtt.MQTTv311, transport="tcp")
mqtt_client.username_pw_set(username=TS_USERNAME, password="v1TXVyjwOT44dPjD3z/wils5")
mqtt_client.on_connect = handle_connect
mqtt_client.on_publish = handle_publish

print("Trying to Connect")
mqtt_client.connect(TS_BROKER, TS_PORT, 60)
mqtt_client.loop_start()

# --- Wait until connection is established ---
while not is_connected:
    print("Waiting for connection")
    time.sleep(1)

# --- Loop to send sensor data every 25 seconds ---
while True:
    sensor_data = generate_sensor_readings()
    message = f"field1={sensor_data['field1']}&field2={sensor_data['field2']}&field3={sensor_data['field3']}"
    print(f"Sending data")

    result = mqtt_client.publish(TS_TOPIC, message)
    if result.rc != mqtt.MQTT_ERR_SUCCESS:
        print("Failed to sed data")

    time.sleep(25)
