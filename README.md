# **PRÁCTICA 6 ENVIO DE DATOS A UNA RED**

## **INSTALACION DE NODEJS** ##

1. Nos dirigimos a la siguiente pagina para descargar node-red https://nodejs.org/en

2. Descargar el archivo 18.16.0 LTS como se muestra en la siguente imagen.

![](https://github.com/DiegoJm10/Node-red-instalcacion/raw/main/Node.js%20-%20Google%20Chrome%2014_06_2023%2005_04_00%20p.%20m..png?raw=true)

3. Seguir los pasos para su instalación 

## UNA VEZ INSTALADO SE REALIZAN LOS SIGUIENTES PASOS 

1. Se abre el programa "CMD" en modo administrador.
2. Una vez abierto se ponen los siguientes comandos:

              1. Primer comando  **npm install -g --unsafe-perm node-red**

Aparecera la siguiente imagen 

![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/nodejs%201.png?raw=true)



                    2. Para arrancar nodejs se usa el comando **node- red** 

Aparecera lo siguiente ya inicio el nodejs se deja abierto para su uso 

![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/arranque%20nodejs.png?raw=true)

3. Nos Dirigimos al navegador buscamos la pagina **localhost:1880**

Aparecera la siguiente imagen lugar de trabajo de nodejs


![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/nodejs%20trabajo.png?raw=true)

## **REALIZACION Y CONFIGURACION DE ESTRUCTURA EN NODEJS** ##

1. Se realiza la siguiente estructura 

![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/Estructura%20nodejs.png?raw=true)

### Configuracion y herramientas usadas 

* Para DAIyM/Axel se utilizo  **mqtt in** "nombre utilizado en codigo"
* Para json se utilizo        **json**
* Para TEMPERATURA se utilizo    **function** "nombre utilizado en codigo "
* Para HUMEDAD se utilizo         **function**"nombre utilizado en codigo "
* Para Temperatura se utilizo     **gauge**
* Para Temperatura vs Humedad se utilizo    **chart**
* Para Humedad  se utilizo               **gauge**


------------------------------------------
* Para la configuracion de mqtt in se utilizo la siguiente 
![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/mqtt%20in.png?raw=true)

* Para json solo se escogio Always convert to JavaScrip Object

* Para la function de temperatura se agrego el siguiente codigo 

```
msg.payload = msg.payload.TEMPERATURA;
msg.topic = "TEMPERATURA";
return msg;
```

* Para la function de humedad se agrego el siguiente comando 
```
msg.payload = msg.payload.HUMEDAD;
msg.topic = "HUMEDAD";
return msg;

```


* Para la configuracion de gauge de temperatura se utilizo la siguiente 
![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/conf%20temperatura.png?raw=true)


* Para la configuracion de gauge de Temperatura vs Humedad  se utilizo la siguiente 
![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/conf%20t%20vs%20h.png?raw=true)

* Para la configuracion de gauge de Humedad  se utilizo la siguiente 
![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/conf%20H.png?raw=true)


# **CONEXION Y CODIGO EN WOKWI

## Para la simulación nos dirigimos a la página  

woki : https://wokwi.com

#### Dashboard de página a utilizar 
![](https://github.com/AxelMld/Practica-3-Sensor-Ultrasonico-/blob/main/dash.png?raw=true)


## Para está práctica se utilizó 

* Módulo ESP32 
* Sensor DHT22

## Pasos a Seguir 

1. Seleccionar el módulo ESP32.
2. Escoger los dispositivos a utilizar anteriormente mensionados. 
3. Realizar las conexiones como se muestra en el apartado "conexión".
4. Agregar las librerias mensiadas en el apartado Librerías 




## **Conexión**

![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/conexion.png?raw=true)


## **Librerías**
1. ArduinoJson
2. DHT sensor library for ESPx
3. PubSubClient

## Código utilizado 


```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="AXELM";
String password_mqtt="1234";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("DAIyM/AxelM", output.c_str());
  }
}


```


# **Resultados**

## 1. Resultado de la pagina WOKWI 

 
![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/resultado%20wokwi.png?raw=true)



## 2. Resultado en la red node-red. 

![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/resultado%20en%20el%20nodejs.png?raw=true)


## 3. Primer  resultado del dashboard 

 
![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/primer%20resultado%20en%20el%20dash.png?raw=true)

## 4. Segunda  resultado del dashboard 

 
![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/segundo%20resultado%20en%20el%20dash.png?raw=true)

![](https://github.com/AxelMld/Practica6-enviodedatos/blob/main/segundo%20resultado.png?raw=true)


Notas: Tener cuidado con el nombramiento de la red, asi como nombramiento del client-publish




# Creditos 

**Axel Miranda.**

