#include <WiFi.h>
#include <HTTPClient.h>

// Configurações do Sensor Ultrassônico
#define TRIG_PIN 18
#define ECHO_PIN 17

// Configurações do Wi-Fi
#define WIFI_SSID "Wokwi-GUEST"
#define WIFI_PASSWORD ""

// Configurações do ThingSpeak
#define THINGSPEAK_API_KEY "VZU9IRGEB4JOYO84"
#define THINGSPEAK_CHANNEL_ID 1234567

WiFiClient client;

float distance;
float duration;

void setup() {
  Serial.begin(9600);

  // Configura os pinos do sensor
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  // Conecta à rede Wi-Fi
  Serial.print("Conectando ao WiFi...");
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi conectado!");
  Serial.print("IP: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  // Envia um pulso de 10 microsegundos para o pino TRIG
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Calcula a duração do pulso de echo
  duration = pulseIn(ECHO_PIN, HIGH);

  // Calcula a distância (em cm) com base na duração do pulso
  distance = duration * 0.01715;

  // Exibe a distância no Monitor Serial
  Serial.print("Distância: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Envia os dados para o ThingSpeak via HTTP
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String serverPath = "http://api.thingspeak.com/update?api_key=" + String(THINGSPEAK_API_KEY) +
                        "&field1=" + String(distance);

    http.begin(serverPath);
    int httpResponseCode = http.GET();

    if (httpResponseCode > 0) {
      Serial.print("Dados enviados para ThingSpeak. Código HTTP: ");
      Serial.println(httpResponseCode);
    } else {
      Serial.print("Erro ao enviar dados. Código HTTP: ");
      Serial.println(httpResponseCode);
    }

    http.end();
  } else {
    Serial.println("Erro de conexão Wi-Fi");
  }

  // O delay de 15 segundos é o mínimo recomendado pelo ThingSpeak.
  // Você pode ajustá-lo, mas não deve ser menor que 15000 ms.
  delay(150);
}
