#include <WiFi.h>
#include <HTTPClient.h>
#include <DHT.h>
#include <Adafruit_BMP085.h>

#define DHTPIN 4 // Pin donde está conectado el sensor DHT
#define DHTTYPE DHT22 // Tipo de sensor DHT

DHT dht(DHTPIN, DHTTYPE);
Adafruit_BMP085 bmp;

const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";
const char* serverName = "http://<your-server-ip>:1880/data";

void setup() {
  Serial.begin(115200);
  
  dht.begin();
  bmp.begin();

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
}

void loop() {
  if(WiFi.status() == WL_CONNECTED) {
    HTTPClient http;

    float temp = dht.readTemperature();
    float humidity = dht.readHumidity();
    float pressure = bmp.readPressure() / 100.0F;

    String data = "temperature=" + String(temp) + "&humidity=" + String(humidity) + "&pressure=" + String(pressure);
    
    http.begin(serverName);
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");

    int httpResponseCode = http.POST(data);

    if(httpResponseCode > 0) {
      String response = http.getString();
      Serial.println(httpResponseCode);
      Serial.println(response);
    } else {
      Serial.print("Error on sending POST: ");
      Serial.println(httpResponseCode);
    }
    
    http.end();
  }
  
  delay(60000); // Enviar datos cada 60 segundos
}
