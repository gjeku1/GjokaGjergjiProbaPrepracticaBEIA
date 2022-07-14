#include <ArduinoJson.h>
#include "DHT.h"
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <NTPClient.h>
#include <WiFiUdp.h>
DHT dht(12, DHT11);
// **********************WiFi*************
String msgStr = "";
char mess[512];
const char *ssid = "LANCOMBEIA"; // Enter your WiFi name
const char *password = "beialancom";  // Enter WiFi password
int counter;



//--------------- MQTT Broker
const char *mqtt_broker = "mqtt.beia-telemetrie.ro";
const char *topic = "training/GjokaGjergji/esp8266test";
const char *mqtt_username = "";
const char *mqtt_password = "";
const int mqtt_port = 1883;
WiFiClient espClient;
PubSubClient client(espClient);
StaticJsonDocument<512> doc;
//-----Callback function -------
void callback(char*topic, byte* payload, unsigned int length)
{
  Serial.print("Message arrived in topic: ");
  Serial.println(topic);
  Serial.print("Message:");
  for (int i = 0; i < length; i++)
  {
    Serial.print((char)payload[i]);
  }
  Serial.println();
  Serial.print("Message size :");
  Serial.println(length);
  Serial.println();
  Serial.println("-----------------------");
}
//---------Wi fi Setup function --------------

void wifi_setup()
{
  delay(10);
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED)
  { 
    delay(5000);
    Serial.println("Connecting to WiFi..");
  }
  Serial.println("Connected to the WiFi network");
  Serial.println("Ip Address: ");
  Serial.print(WiFi.localIP());


}


//--------Reconnect function 
void reconnect()
{
  while (!client.connected())
  {
    String client_id = "Gjoka: ";
    client_id += String(WiFi.macAddress());
    Serial.printf("The client %s connects to the public mqtt broker\n", client_id.c_str());
    if (client.connect(client_id.c_str(), mqtt_username, mqtt_password))
    {
      Serial.println("Public emqx mqtt broker connected");
    }
    else
    {
      Serial.print("failed with state ");
      Serial.print(client.state());
      delay(2000);
    }
  }

}


///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
void setup()
{// Set software serial baud to 9600;
  Serial.begin(115200);
  Serial.print("Start program");
  dht.begin();
  Serial.print("Start wifi setup");
  wifi_setup();
  


  //****************connecting to a mqtt broker
  Serial.print("Conncction at mqtt server");
  client.setServer(mqtt_broker, mqtt_port);
  client.setCallback(callback);
}




///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
void loop()
{ //counter++;
  
  if (!client.connected())
  {
    reconnect();//try to reconnect
  }

  //**********************For sensors***************






  //*********************************************

   sensorDHT();

 // mqtt_Publish(10, "Temperatura");

  delay(50);
}


/////////////////////////////////////////////////////




void mqtt_Publish(float i , char* k)
{
  doc["sennsor"] = k;
  doc["value"] = i;

  serializeJson(doc, mess);
  client.publish(topic, mess);
  client.subscribe(topic);
  msgStr = "";
  delay(50);

}

//-----------Sensor DHT
void sensorDHT()
{
  int8_t h = dht.readHumidity(); //Read Humidity
  int8_t t = dht.readTemperature(); //Read Temperature

  // Check if valid number if non NaN (not a number) will be send.
  if (isnan(t) || isnan(h))
  {
    Serial.println("DHT22 couldn’t be read");
  } else
  {
    Serial.println ("Humidity    : ");
    Serial.print(h);
    //mqtt_Publish(h, "Humidity");
    Serial.println(" %");
    Serial.print("Temperature : ");
    Serial.print(t);
    //mqtt_Publish(t, "Temperature");
    Serial.println(" C");
  doc["TEMP"] = t;
  doc["HUM"] = h;

  serializeJson(doc, mess);
  client.publish(topic, mess);
  client.subscribe(topic);
  msgStr = "";
  delay(50);
    delay(5000);
  }
}

