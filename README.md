# PROYECTO-FINAL
**DESCRIPCION DE PROPUESTA**


Producción de A pilar (molduras automotrices) mediante el proceso de moldeo por inyección de plástico.
Dentro del proceso de fabricación de molduras A pilar está la preparación de materia prima.
Consta de un depósito de alimentación de materia prima (resina plástica) y otro depósito dónde es succionada para aplicar calor a una temperatura en un rango de 50°C a  60°C para así eliminar la humedad del material. Una vez que se elimina la humedad del material este es bombeado hacía la tolva mediante una manguera hacía la IMM (Inyection Machine Mould) 

______________________

**OBJETIVO**


Para mejorar el proceso de control de temperatura y nivel de material dentro de los depósitos de materia prima del alimentador y secador implementaremos dos sensores DHT22, 2 sensores ultrasónicos HC-SR04, 2 LCD y una tarjeta ESP32.
Cuando el nivel de material dentro de los tanques sea bajo se mandará una alarma y un mensaje de “Vacío”.
Cuando el nivel de temperatura salga del rango de 50°C a 60°C se mandará una alarma y un mensaje de “temperatura alta o baja”.
Cuando el nivel de humedad del tanque sea mayor a 5% se mandara una alarma y un mensaje de “humedad inestable”

_____________________

**MATERIALES**

*Para el control de humedad y temperatura utilizaremos el sensor DHT22.
*Para el control del material dentro del depósito utilizaremos el sensor HC.SR04.
*Para la interpretación de los datos de humedad, temperatura y nivel de material dentro de los depósitos utilizaremos 2 LCD
*Para el control del circuito utilizaremos la tarjeta ESP32


________________________
**DESCRIPCION DE LOS COMPONENTES**

DHT22: Sensor de Humedad y Temperatura de Alta Precisión
Descripción: El DHT22, también conocido como AM2302, es un sensor digital de humedad y temperatura que ofrece mediciones precisas y de alta calidad. Está compuesto por un pequeño circuito integrado y un sensor capacitivo de humedad y temperatura. 
Es ampliamente utilizado en aplicaciones que requieren un monitoreo preciso del entorno.
Características Principales:
1.	Medición de Humedad y Temperatura:
•	Proporciona mediciones digitales de humedad relativa y temperatura ambiente.
2.	Bajo Consumo de Energía:
•	Opera con un bajo consumo de energía, lo que lo hace adecuado para aplicaciones con restricciones de energía.
3.	Tiempo de Respuesta Rápido:
•	Proporciona mediciones actualizadas en intervalos cortos de tiempo.
Aplicaciones Comunes:
•	Sistemas de monitoreo de clima interior y exterior.
•	Proyectos de automatización del hogar.
•	Dispositivos de control ambiental y climatización.
•	Estaciones meteorológicas y proyectos DIY (hazlo tú mismo).


 
HC-SR04: Sensor Ultrasónico de Distancia
 El HC-SR04 es un sensor ultrasónico de distancia ampliamente utilizado para medir distancias sin contacto de manera precisa y eficiente. Este dispositivo utiliza ondas ultrasónicas para calcular la distancia entre el sensor y un objeto, proporcionando así una solución efectiva para proyectos de detección de proximidad.
Características Principales:
•	Principio de Funcionamiento: Emite pulsos ultrasónicos y mide el tiempo que tardan en regresar después de rebotar en un objeto.
•	Rango de Medición: Generalmente, tiene un rango de medición de 2 cm a 4 metros.
•	Bajo Costo: Es una opción asequible para aplicaciones de medición de distancia.
Cómo Funciona:
1.	Trigger (Disparador): Se envía un pulso de 10 µs para activar el sensor.
2.	Transmisión de Pulsos Ultrasónicos: El sensor emite una serie de pulsos ultrasónicos.
3.	Recepción de Eco: El sensor espera el eco reflejado desde el objeto.
4.	Cálculo de Distancia: La distancia se calcula midiendo el tiempo que tarda en regresar el eco.

 



El ESP32 es un microcontrolador revolucionario que se ha vuelto esencial en la tecnología moderna. Es un sistema en chip (SoC) de bajo costo y bajo consumo de energía con una combinación de capacidades Wi-Fi y Bluetooth. 

1.	30 Pines de E/S: Ofrece una amplia variedad de pines para la conexión de sensores, actuadores y otros dispositivos.
2.	Alimentación: Puede ser alimentado a través de un puerto MicroUSB o mediante otros métodos de suministro de energía.
3.	Wi-Fi y Bluetooth Integrados: Facilita la conectividad inalámbrica para la comunicación y el control remoto.
4.	Capacidades de Programación: Se puede programar utilizando el entorno de desarrollo de Arduino o herramientas específicas de Espressif.
Aplicaciones Comunes:
1.	Proyectos de IoT (Internet de las Cosas): Ideal para dispositivos conectados a la red que requieren comunicación inalámbrica.
2.	Sistemas Embebidos: Puede ser utilizado en sistemas embebidos para controlar y monitorear dispositivos.
3.	Desarrollo de Prototipos: Ampliamente utilizado en el desarrollo de prototipos debido a su facilidad de programación y su capacidad de conexión a una variedad de dispositivos.



______________________

**MODO DE CONEXION**


![](https://github.com/marcorea97/PROYECTO-FINAL/blob/main/MODO%20DE%20CONEXION%20PROYECTO.png)

 ______________________
 **CODIGO A UTILIZAR**

Código de programación para monitorear la temperatura y nivel de los depósitos.

#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;

// Update these with values suitable for your network.

 
const int trigPin = 4;
const int echoPin = 2;
const int trigPin2 = 17;
const int echoPin2 = 16;
const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "3.76.79.164";
String username_mqtt="RicardoR";
String password_mqtt="12345678";

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
  pinMode(trigPin, OUTPUT); //pin como salida
  pinMode(echoPin, INPUT);  //pin como entrada
  pinMode(trigPin2, OUTPUT); //pin como salida
  pinMode(echoPin2, INPUT);  //pin como entrada
  digitalWrite(trigPin, LOW);//Inicializamos el pin con 0
  digitalWrite(trigPin2, LOW);//Inicializamos el pin con 0
}

void loop()
{

  long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros
  long d2; //distancia en centimetros

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(trigPin, LOW);
  
  t = pulseIn(echoPin, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm

  digitalWrite(trigPin2, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(trigPin2, LOW);
  
  t = pulseIn(echoPin2, HIGH); //obtenemos el ancho del pulso
  d2 = t/59;             //escalamos el tiempo a una distancia en cm

Serial.print("NIVEL DE ALIMENTADOR: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms
delay(1000);

Serial.print("NIVEL SECADOR: ");
  Serial.print(d2);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms
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
    doc["NOMBRE"] = "TANQUE CON NIVEL DE SENSOR Y ALARMA DE TEMPERATURA";
    doc["DISTANCIA"]= String(d);
    doc["DISTANCIA2"]= String(d2);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("TANQUE CON NIVEL DE SENSOR Y ALARMA DE TEMPERATURA", output.c_str());
  }
}


**IMPORTACION A NODE-RED**

Realizando todas las conexiones se tiene lo siguiente:

![](https://github.com/marcorea97/PROYECTO-FINAL/blob/main/original-C6916F64-87DF-453C-BA86-9A1B5D900FA0%20(1).jpeg)





_____________________________________

**RESULTADOS**

Al utilizar el software Node-RED, obtenemos una herramienta comparable a un HMI (Interfaz Hombre-Máquina) a nivel industrial, que nos permite visualizar datos estadísticos mediante gráficos y obtener los valores en tiempo real. Cada una de las variables es crucial para monitorear el proceso y evitar que se detenga debido a ajustes en las mismas.

Para verificar que el sistema está funcionando correctamente, se pone en operación y se ajusta a las condiciones necesarias, permitiendo que se muestren las alertas configuradas en el código. El programa está diseñado para generar cinco señales, de modo que el operador pueda tomar las acciones necesarias para garantizar que el proceso continúe sin interrupciones.

SEÑAL DE HUMEDAD INESTABLE


Esta señal se nos presentara cuando exista un cambio en el valor de la humedad dentro del secador


![](https://github.com/marcorea97/PROYECTO-FINAL/blob/main/original-EA10A9DE-0649-4D64-8DCA-F6E1F1AAC1A4.jpeg)


SEÑAL DEL NIVEL DEL ALIMENTADOR


Esta señal se activará cuando el nivel del alimentador esté bajo. Gracias a esta alerta visual y sonora en la pantalla, el operador podrá reabastecer el material sin interrumpir el proceso, lo que permite continuar con la operación sin necesidad de detenerse.


![](https://github.com/marcorea97/PROYECTO-FINAL/blob/main/original-AF3F75A4-8AFD-420F-B1C8-789E3AEA33BF.jpeg)




SEÑAL NIVEL DEL SECADOR (BAJO)


Esta señal permitirá al operador monitorear el nivel del secador en tiempo real. Cuando el nivel esté bajo, el sistema le alertará mediante señales visuales y sonoras, lo que le permitirá añadir material al secador sin interrumpir el proceso. De esta manera, el operador podrá optimizar la producción sin necesidad de detenerse para recargar el secador 

![](https://github.com/marcorea97/PROYECTO-FINAL/blob/main/original-510AAE4E-A7F5-48D7-94A4-8FE9806BED86.jpeg)


SEÑAL DEL NIVEL DE TEMPERATURA (BAJA)

Esta señal indica que la temperatura dentro del secador está por debajo del nivel óptimo. Ante esta alerta, el operador deberá ajustar la temperatura para evitar que el estado de la materia prima se vea afectado al pasar al siguiente proceso. Gracias a esta alarma, el operador podrá tomar las medidas necesarias para asegurar que el material no sufra desperfectos durante la transición al siguiente paso de producción.


![](
https://github.com/marcorea97/PROYECTO-FINAL/blob/main/original-FF8A1C21-0B3D-4739-BEAF-F165A0A878E2.jpeg)


SEÑAL DEL NIVEL DE TEMPERATURA (ALTA)

Esta señal indica que la temperatura dentro del secador es excesiva. Si la temperatura se mantiene en ese nivel, el material podría sufrir alteraciones en su composición y calidad. El operador deberá tomar medidas inmediatas para ajustar la temperatura y asegurar que se mantenga dentro de los niveles óptimos. Gracias a esta alerta, el operador tendrá la oportunidad de corregir la situación antes de que afecte el proceso


![](https://github.com/marcorea97/PROYECTO-FINAL/blob/main/original-8BB8F168-BA06-4CD6-A84A-883A03F13774.jpeg)


_______________________________

**CONCLUSION**

Gracias al uso del software Wokwi y Node-RED, se adquiere un conocimiento profundo sobre el funcionamiento de los sistemas de monitoreo hombre-máquina. Esto permite comprender cómo se utilizan estas herramientas y cómo podemos aprovechar sus capacidades en un entorno industrial para optimizar los procesos y mejorar la eficiencia


