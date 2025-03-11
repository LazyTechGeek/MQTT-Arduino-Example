# MQTT-Arduino-Example
MQTT Arduino Sample Code

Description: Make Alexa Call People Out for Leaving the Toilet Seat Up! (Node-RED Tutorial)

Link to YouTube video: https://youtu.be/HoYak0LTelg

This is the code for sample Arduino to connect to MQTT and turn on or off the built in LED.

- **Full Code here**
```
#include <WiFi.h>
#include <PubSubClient.h>
#include "secrets.h"  // Import secrets


// Wifi details
const char* ssid = "SSID";
const char* password = "SSID PASSWORD";

// MQTT Server details
const char* mqtt_username = "MQTT USERNAME";
const char* mqtt_password = "MQTT PASSWORD*";
const char* mqtt_server = "MQTT SERVER IP";

//MQTT Client and Topic Details
const char* mqtt_client = "Demo_Client1"; // MQTT Client name (MUST BE UNIQUE)
const char topic[] = "demo/test";  // Explicitly defining the topic

///////////////////////////
// DO NOT EDIT THE BELOW //"!"
///////////////////////////

WiFiClient espClient;
PubSubClient client(espClient);

#define LED_PIN 2  // Set to the correct LED GPIO for your ESP32 board

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);

  pinMode(LED_PIN, OUTPUT);  // Initialize LED pin

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Connected to WiFi");

  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);

  reconnect();  // Ensure MQTT connection and subscription
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
}

void callback(char* receivedTopic, byte* payload, unsigned int length) {
  Serial.print("Message received [");
  Serial.print(receivedTopic);
  Serial.print("]: ");

  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.println(message);

  if (strcmp(receivedTopic, topic) == 0) {  // Check if message is for the correct topic
    if (message == "on") {
      digitalWrite(LED_PIN, HIGH);
      Serial.println("LED turned ON");
    } else if (message == "off") {
      digitalWrite(LED_PIN, LOW);
      Serial.println("LED turned OFF");
    }
  }
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");

    // Connect with MQTT username & password
    if (client.connect(mqtt_client, mqtt_username, mqtt_password)) {
      Serial.println("Connected to MQTT broker");
      client.subscribe(topic);  // Subscribe to topic after successful connection
    } else {
      Serial.print("Failed, rc=");
      Serial.print(client.state());
      Serial.println(" retrying in 5 seconds");
      delay(5000);
    }
  }
}

```
