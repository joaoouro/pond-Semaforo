# pond-Semaforo

## Componentes Utilizados

- 1x **ESP32**
- 1x **LCD 16x2 com módulo I2C**
- 1x **LED Vermelho**
- 1x **LED Verde**
- 1x **LED Azul** (substituindo o LED amarelo)
- 3x **Resistores de 330Ω** 
- **Jumpers** 
- **Protoboard**

## Esquema de Conexão

### Conexões dos LEDs
- **LED Vermelho**:
  - Anodo (perna longa) -> GPIO **23** do ESP32 
  - Cátodo (perna curta) -> **GND** 

- **LED Verde**:
  - Anodo (perna longa) -> GPIO **22** do ESP32 
  - Cátodo (perna curta) -> **GND**

- **LED Azul (substituindo o amarelo)**:
  - Anodo (perna longa) -> GPIO **5** do ESP32 
  - Cátodo (perna curta) -> **GND**

### Conexões do LCD I2C
- **VCC** -> Conectar ao pino **5V** do ESP32 (dependendo do módulo LCD)
- **GND** -> Conectar ao **GND** do ESP32
- **SDA** -> Conectar ao **GPIO 21** do ESP32
- **SCL** -> Conectar ao **GPIO 22** do ESP32

> **Nota:** Como o GPIO **22** já está sendo usado pelo LED verde, foram alterados os pinos SDA e SCL no código para **GPIO 16** e **GPIO 17**, respectivamente.

## Funcionamento
O semáforo alterna entre as fases de vermelho, amarelo e verde seguindo a lógica abaixo:
- **6 segundos** no vermelho
- **2 segundos** no amarelo
- **2 segundos** no verde
- **+2 segundos** no verde (simulando um tempo adicional para pedestres terminarem a travessia)
- **2 segundos** no amarelo novamente

Além dos LEDs, o LCD exibe mensagens para indicar o estado atual do semáforo:
- "Semaforo: Vermelho" seguido de "Pare!"
- "Semaforo: Amarelo" seguido de "Atencao!"
- "Semaforo: Verde" seguido de "Siga com cuidado!"

## Código
O código foi escrito na **Arduino IDE** e faz uso da biblioteca `LiquidCrystal_I2C` para controlar o LCD. Foi utilizado **ponteiros** no código para controlar o tempo de cada fase do semáforo de forma mais flexível.

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define redLed 23 // Pino do LED vermelho
#define yellowPin 5  // Pino do LED amarelo (substituído pelo LED azul)
#define greenLed 22 // Pino do LED verde

// Inicialização do LCD (endereço I2C padrão é 0x27, e display 16x2)
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Ponteiro para controlar o tempo de delay do semáforo
int delayTimeRed = 6000;
int delayTimeYellow = 2000;
int delayTimeGreen = 2000;
int *ponteiroDelay = &delayTimeRed;

void rLed() {
  // Liga o LED vermelho e desliga o LED amarelo e verde
  digitalWrite(yellowPin, LOW);
  digitalWrite(greenLed, LOW);
  digitalWrite(redLed, HIGH);
  
  // Atualiza o LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Semaforo: Vermelho");
  lcd.setCursor(0, 1);
  lcd.print("Pare!");
}

void gLed() {
  // Liga o LED verde e desliga o LED amarelo e vermelho
  digitalWrite(yellowPin, LOW);
  digitalWrite(redLed, LOW);
  digitalWrite(greenLed, HIGH);
  
  // Atualiza o LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Semaforo: Verde");
  lcd.setCursor(0, 1);
  lcd.print("Siga com cuidado!");
}

void yLed() {
  // Liga o LED amarelo (substituído pelo LED azul) e desliga o LED vermelho e verde
  digitalWrite(yellowPin, HIGH);
  digitalWrite(redLed, LOW);
  digitalWrite(greenLed, LOW);
  
  // Atualiza o LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Semaforo: Amarelo");
  lcd.setCursor(0, 1);
  lcd.print("Atencao!");
}

void setup() {
  // Inicializa os pinos dos LEDs como saída
  pinMode(redLed, OUTPUT);
  pinMode(greenLed, OUTPUT);
  pinMode(yellowPin, OUTPUT);
  
  // Configura os pinos I2C para evitar conflito
  Wire.begin(16, 17); // SDA = GPIO 16, SCL = GPIO 17

  // Inicializa o LCD
  lcd.init();
  lcd.backlight();
}

void loop() {
  // Fase Vermelho
  *ponteiroDelay = delayTimeRed;
  rLed();
  delay(*ponteiroDelay);
  
  // Fase Amarelo
  *ponteiroDelay = delayTimeYellow;
  yLed();
  delay(*ponteiroDelay);
  
  // Fase Verde
  *ponteiroDelay = delayTimeGreen;
  gLed();
  delay(*ponteiroDelay);
  
  // Tempo adicional para pedestres
  delay(*ponteiroDelay);
  
  // Fase Amarelo novamente
  *ponteiroDelay = delayTimeYellow;
  yLed();
  delay(*ponteiroDelay);
}

```

## Futuras Melhorias
- Implementar um **sensor de presença** para pedestres, de forma que o tempo de verde possa ser ajustado automaticamente.
- Adicionar uma **buzzer** para alertar pedestres durante a transição do verde para o vermelho.


### Avaliador: Mariana

| Critério                                                                                                 | Contempla (Pontos) | Contempla Parcialmente (Pontos) | Não Contempla (Pontos) | Observações do Avaliador |
|---------------------------------------------------------------------------------------------------------|--------------------|----------------------------------|--------------------------|---------------------------|
| Montagem física com cores corretas, boa disposição dos fios e uso adequado de resistores                | 2             |                             |                         |              Utilizou os fios preto e vermelho para negativo e positivo, respectivamente, mas os jumpers ligados aos leds poderiam ter sido mais bem escolhidos. Além de não utilizar o led amarelo          |
| Temporização adequada conforme tempos medidos com auxílio de algum instrumento externo                  | 3              |                           |                         |             Certo            |
| Código implementa corretamente as fases do semáforo e estrutura do código (variáveis representativas e comentários) | 3              |                           |                         |           Certo                |
| Extra: Implmeentou um componente de liga/desliga no semáforo e/ou usou ponteiros no código | 0.75              |                           |                         |         Não utilizou componente liga/desliga, mas fez um bom uso dos ponteiros.          |
|  |                                                             |  | |*Pontuação Total* 8,75|



### Avaliador: Nicolas Ramon

| Critério                                                                                                 | Contempla (Pontos) | Contempla Parcialmente (Pontos) | Não Contempla (Pontos) | Observações do Avaliador |
|---------------------------------------------------------------------------------------------------------|--------------------|----------------------------------|--------------------------|---------------------------|
| Montagem física com cores corretas, boa disposição dos fios e uso adequado de resistores                |               |               1.5               |                         |          Não utilizou led amarelo, e jumpers bagunçados. Entretanto resistores foram bem utilizados               |
| Temporização adequada conforme tempos medidos com auxílio de algum instrumento externo                  | 3              |                           |                         |                           |
| Código implementa corretamente as fases do semáforo e estrutura do código (variáveis representativas e comentários) | 3              |                           |                         |                          |
| Extra: Implmeentou um componente de liga/desliga no semáforo e/ou usou ponteiros no código | 1              |                           |                         |                 Fez um ótimo uso dos ponteiros          |
|  |                                                             |  | |*Pontuação Total* 8,5|