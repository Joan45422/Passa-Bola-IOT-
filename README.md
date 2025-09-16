# Passa-Bola-IOT-
1. Nomes dos Integrantes
Joan Ferreira RM 562913

Felipe Rodrigues RM 565341

Felipe Bonilha RM 562356

Levi RM 563279

Gabriel Salles RM 563584

2. Descrição do Projeto
Este projeto consiste em um sistema de monitoramento de distância em tempo real, usando um microcontrolador ESP32 e um sensor ultrassônico HC-SR04. Os dados coletados são enviados para a plataforma ThingSpeak através de uma conexão Wi-Fi, permitindo a visualização remota e em tempo real em um painel interativo.

O objetivo principal é criar uma solução de baixo custo para auxiliar árbitros e torcedores a confirmarem lances de gol, aumentando a precisão e a justiça nas partidas de futebol.

4. Arquitetura Proposta (Diagrama e Explicação)
Diagrama de Blocos
Snippet de código

graph TD
    A[Sensor Ultrassônico HC-SR04] --> B[Microcontrolador ESP32];
    B --> C[Módulo Wi-Fi];
    C --> D[Servidor ThingSpeak na Nuvem];
    D --> E[Dashboard/Visualização de Dados];
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#bbf,stroke:#333,stroke-width:2px
    style C fill:#ccf,stroke:#333,stroke-width:2px
    style D fill:#cdf,stroke:#333,stroke-width:2px
    style E fill:#fcf,stroke:#333,stroke-width:2px
Explicação da Arquitetura
Sensor Ultrassônico HC-SR04: Este componente é responsável por emitir pulsos de ultrassom e captar o eco. O tempo de retorno do pulso é a base para o cálculo da distância.

Microcontrolador ESP32: O "cérebro" do sistema. Ele lê os dados do sensor, realiza o cálculo de distância e gerencia a comunicação com a rede Wi-Fi para o envio das informações.

Módulo Wi-Fi: Parte integrante do ESP32, ele permite a conexão sem fio à internet.

Servidor ThingSpeak: A plataforma em nuvem que recebe, armazena e processa os dados de distância. O acesso é feito através de uma chave API, garantindo a segurança dos dados.

Dashboard/Visualização de Dados: O ThingSpeak organiza os dados recebidos em gráficos e painéis, que podem ser acessados de qualquer dispositivo com internet, permitindo o monitoramento remoto.

4. Recursos Necessários
Hardware:

1x Placa de Microcontrolador ESP32

1x Sensor Ultrassônico HC-SR04

1x Protoboard

Cabos de conexão (Jumpers)

Computador com a IDE do Arduino instalada

Software:

IDE do Arduino (com o suporte para ESP32 instalado)

Conta no ThingSpeak

Bibliotecas WiFi.h e HTTPClient.h (já inclusas na IDE do ESP32)

5. Instruções de Uso do Código
Configuração da IDE do Arduino:

Certifique-se de que o suporte para a placa ESP32 está instalado. Se não estiver, adicione o URL do Gerenciador de Placas nas preferências da IDE e instale o pacote esp32.

Selecione sua placa ESP32 em Ferramentas > Placa.

Configuração do ThingSpeak:

Crie uma conta e um novo canal. Dê um nome ao seu canal e a um campo, por exemplo, "Distância (cm)".

Na aba "Chaves de API", copie a sua chave de gravação (Write API Key).

Fiação (Conexão Física):

VCC do sensor HC-SR04 → 3.3V/5V do ESP32

GND do sensor HC-SR04 → GND do ESP32

TRIG do sensor → GPIO 18 do ESP32

ECHO do sensor → GPIO 17 do ESP32

Configuração e Upload do Código:

No código, substitua WIFI_SSID, WIFI_PASSWORD e THINGSPEAK_API_KEY pelas suas próprias credenciais.

Conecte o ESP32 ao computador, selecione a porta correta em Ferramentas > Porta e clique em "Carregar".

Verificação:

Abra o Monitor Serial na IDE para acompanhar as medições e o status de conexão.

Acesse a página do seu canal no ThingSpeak para ver o gráfico de distância sendo atualizado a cada 15 segundos.

Código fonte:

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
