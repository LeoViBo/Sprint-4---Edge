# Sprin 4 - Edge Computing & Computer Systems
## Grupo - International Business of FIAP (IBF):

RM92699 - Arielly Gleice Geovana de Oliveira

RM97673 - Tulio Damaceno de Resende

RM551869 - Fabrizio Maia Apparicio

RM551684 - Victor Miguel Gouveia Asfur

RM551715 - Leonardo Viotti Bonini

## Explicação do Projeto

Nosso projeto é usar uma válvula que, quando instalada em um cano, detecta o fluxo de água que está passando pelo cano, e depois informa para um aplicativo para o usuário o quanto de água esta sendo gasta. Para isso comprar uma valvula que mede o fluxo de água e com isso conectamos ela a um Arduino.

Para nossa arquitetura IoT seria o arduino com um módulo de bluetooth que se conecta a celulares onde temos um aplicativo que irá se conectar com as vávlulas permitindo a transferencia de dados para passar o fluxo de água e abrir e fechar a válvula. Além de outras informações mostradas somente no aplicativo como o nome da valvula, em que parte da casa está instalada e em que tipo de imovel está instalado, residencia, comercial, industrial, etc... para que em outra parte da aplicação seja feito um cálculo que estipula o valor da conta de água. Para melhor entendimento coloquei um arquivo em .py onde fizemos a simulação dessa aplicação.

Como não temos o aplicativo ainda, não usamos um modulo bluetooth e então conectamos o arduino diretamente no computador, de lá via o node-red ele envia os dados para um dashboard que simularia parte do aplicativo, mostrando o fluxo de água do sensor e mostrando o valor da conta de água da casa que é calculada no próprio Arduino. Ao mesmo tempo ele envia esses mesmos valores via mqtt para para WaterWatch/FluxoDeAgua e WaterWatch/ContaDeAgua respectivamente, e assim via um aplicativo de celular, o MyMQTT, podemos ver esses valores no celular.


## Código Fonte Arduino

#include <ArduinoJson.h>

byte statusLed = 13;
byte sensorInterrupt = 0; 
byte sensorPin = 8;
float calibrationFactor = 4.5;
volatile byte pulseCount;
float flowRate;
unsigned flowMilliLitres;
unsigned long totalMilliLitres;
unsigned long oldTime;
float valorConta;
 
void setup()
{
 
Serial.begin(9600);
 
pinMode(statusLed, OUTPUT);
digitalWrite(statusLed, HIGH);
 
pinMode(sensorPin, INPUT);
digitalWrite(sensorPin, HIGH);
 
pulseCount = 0;
flowRate = 0.0;
flowMilliLitres = 0;
totalMilliLitres = 0;
oldTime = 0;
 
attachInterrupt(sensorInterrupt, pulseCounter, FALLING);
}
 
void loop()
{
  delay(2000);
  _Valvula();
  _calcularConta();
  _json();
}

void _json(){
  // Crie um objeto JSON para armazenar os dados
  StaticJsonDocument<128> json;

  json["volume_Total"] = totalMilliLitres;
  json["valor_Conta"] = valorConta;

  // Serialize o objeto JSON para uma string
  String jsonString;
  serializeJson(json, jsonString);

  // Envie o JSON pela porta serial
  Serial.println(jsonString);
}

void pulseCounter()
{
  pulseCount++;
}

void _Valvula()
{
  detachInterrupt(sensorInterrupt);
  flowRate = ((1000.0 / (millis() - oldTime)) * pulseCount) / calibrationFactor;
  oldTime = millis();
  flowMilliLitres = (flowRate / 60) * 1000;
  
  totalMilliLitres += flowMilliLitres;
  
  pulseCount = 0;
  
  attachInterrupt(sensorInterrupt, pulseCounter, FALLING);
}

void _calcularConta()
{
  valorConta = 0;
  float valorEsgoto = 0;
  float conversaoFluxo = totalMilliLitres / 1000000.0;
  if (conversaoFluxo <= 10){333
    valorConta = 33.52;
  } else if (conversaoFluxo >= 11 && conversaoFluxo <= 20){
    valorConta = 33.52 + (conversaoFluxo - 10) * 3.94;
  } else if (conversaoFluxo >= 21 && conversaoFluxo <= 30){
    valorConta = 33.52 + (conversaoFluxo - 10) * 6.01;
  } else if (conversaoFluxo >= 31 && conversaoFluxo <= 50){
    valorConta = 33.52 + (conversaoFluxo - 10) * 6.01;
  } else if (conversaoFluxo > 50){
    valorConta = 33.52 + (conversaoFluxo - 10) * 7.20;
  }
  valorEsgoto = valorConta * 80 / 100;
  valorConta = valorConta + valorEsgoto;
}

