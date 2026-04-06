# Projeto-cau-
#include <WiFi.h>                // Biblioteca para conexão Wi-Fi no ESP32
#include <WiFiClientSecure.h>   // Cliente Wi-Fi com suporte a conexão segura (SSL/TLS)
#include <PubSubClient.h>       // Biblioteca para comunicação MQTT

#define ID_MQTT "SENAI_IOT01"              // ID único do cliente MQTT
#define pubSensor1IotSenai "Sensor1IotSenai" // Tópico para publicação de dados
#define subLed1IotSenai "Led1IotSenai"       // Tópico para receber comandos do LED
#define USERNAME_MQTT "SENAI_IOT01_web_client" // Usuário do broker MQTT
#define PASSWORD_MQTT "Aa123456789"           // Senha do broker MQTT
#define LED_PIN 25                            // Pino onde o LED está conectado

const char* SSID = "Wokwi-GUEST";  // Nome da rede Wi-Fi
const char* BROKER_MQTT = "3bb749045ed3461faf6be8192d8d1b8d.s1.eu.hivemq.cloud"; // Endereço do broker MQTT

WiFiClientSecure wifiClient; // Cria um cliente Wi-Fi seguro (SSL)
PubSubClient MQTT(wifiClient); // Cria o cliente MQTT usando o Wi-Fi seguro

bool ledState = false; // Armazena o estado do LED

// Função para ligar/desligar o LED
void setLed(bool estado) {
  digitalWrite(LED_PIN, estado ? HIGH : LOW);
}

// Função chamada quando chega mensagem MQTT
void callback(char* topic, byte* payload, unsigned int length) {
  String msg;

  // Converte payload (bytes) em string
  for (unsigned int i = 0; i < length; i++)
    msg += (char)payload[i];

  // Define estado do LED baseado na mensagem
  bool novoEstado = (msg == "1" || msg == "ON");

  ledState = novoEstado;
  setLed(ledState);

  Serial.println("LED " + String(ledState ? "ON" : "OFF") + " (MQTT)");
}

// Setup (executa uma vez)
void setup() {
  Serial.begin(115200);
  delay(1000);

  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);

  // Conecta ao Wi-Fi
  WiFi.begin(SSID, "");
  Serial.print("Conectando ao WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi conectado!");

  wifiClient.setInsecure(); // Ignora certificado SSL (apenas testes)

  MQTT.setServer(BROKER_MQTT, 8883); // Broker MQTT
  MQTT.setCallback(callback);        // Função de recebimento
}

// Loop principal
void loop() {
  // Garante conexão Wi-Fi
  if (WiFi.status() != WL_CONNECTED) {
    delay(500);
    return;
  }

  // Garante conexão MQTT
  if (!MQTT.connected()) {
    Serial.print("Conectando ao MQTT...");
    if (MQTT.connect(ID_MQTT, USERNAME_MQTT, PASSWORD_MQTT)) {
      Serial.println("conectado!");
      MQTT.subscribe(subLed1IotSenai);
    } else {
      Serial.println("falhou, tentando novamente...");
      delay(2000);
    }
  }

  MQTT.loop(); // Mantém comunicação ativa
}
