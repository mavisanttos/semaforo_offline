# **Documentação: Semáforo Offline**

**Autor:** Maria Vitória dos Santos

**Data:** 31 de outubro de 2025  

**Disciplina/Contexto:** Programação

---
## 1. Objetivo do projeto

&emsp; O objetivo deste projeto é desenvolver um semáforo capaz de controlar o fluxo de trânsito por meio de uma temporização pré-definida para cada uma de suas três fases — vermelha, amarela e verde —, assegurando um ciclo contínuo de sinalização. Além disso, o sistema inclui um display que exibe a contagem regressiva de cada fase, proporcionando maior clareza e previsibilidade.

&emsp; O sistema segue a seguinte lógica de temporização:
- 6 segundos na luz vermelha
- 2 segundos na luz amarela
- 4 segundos na luz verde

---

## 2. Vídeo de Demonstração

&emsp; O vídeo a seguir demonstra a montagem física do protótipo em funcionamento e comprova a autoria do projeto. A temporização de cada fase (6s, 4s, 2s) é validada pelo display.

&emsp; [Link]()

---

## 3. Manual de Instruções: Montagem e Operação

&emsp; Este manual detalha os componentes necessários e os passos para a montagem completa do semáforo com display, usando um Arduino Uno como controlador central.

### Componentes e Especificações

| Componente           | Quantidade | Especificação                  |
|-----------------------|-------------|----------------------------------|
| Microcontrolador      | 1           | Arduino Uno                     |
| Protoboard            | 1           | 1560 furos                      |
| Estrutura             | 1           | Modelo de Semáforo              |
| LED Difuso            | 1           | Vermelho                        |
| LED Difuso            | 1           | Amarelo                         |
| LED Difuso            | 1           | Verde                           |
| Resistor              | 3           | 10k Ω (Ohm)                     |
| Display LCD           | 1           | Módulo I2C                      |
| Jumper                | x           | Macho-Fêmea                     |
| Jumper                | x           | Macho-Macho                     |
| Cabo USB              | 1           | USB-B para USB-A                |

---

### Passos de Monatgem (Hardware)

- Passo 1: Posicione os três LEDs (Vermelho, Amarelo, Verde) no modelo de semáforo, mantendo a ordem vertical correta.

- Passo 2: Conexão 

---

### 🔌 Justificativa das Conexões

- **LEDs (Diodo Emissor de Luz):**  
  Cada LED possui duas pernas:  
  - **Anodo (Perna longa):** lado positivo (+), conectado ao pino digital do Arduino.  
  - **Catodo (Perna curta):** lado negativo (-), conectado ao **GND**.

- **Resistores (220 Ω):**  
  Limitam a corrente elétrica que flui dos pinos de 5V, protegendo os LEDs contra sobrecorrente.

- **Pinos Digitais (10, 11, 12):**  
  Configurados como **OUTPUT**.  
  - `HIGH` → LED acende  
  - `LOW` → LED apaga

- **GND (Terra):**  
  Serve como referência “zero” do circuito, conectando o catodo de todos os LEDs.

---

## Programação e Lógica (Software)

&emsp; Com o hardware montado, a lógica de software controla o sistema. A adição do display exige uma lógica de "não-bloqueio" (non-blocking).

### Lógica de Funcionamento

- Usamos a função millis(), que funciona como um relógio interno, segundo a lógica:

  - O loop() roda milhares de vezes por segundo.

  - Armazenamos o "estado" atual (ex: ESTADO_VERMELHO).

  - A cada segundo (verificado por if (millis() - ultimoTick >= 1000)), o contador é diminuído em 1 e o display é atualizado.

  - Quando o contador chega a zero, o programa muda o estado (ex: de VERMELHO para VERDE), acende o LED correto e reinicia o contador para o tempo da nova fase.

---

### Código Fonte

```cpp
/*
 * ====================================================================
 * Projeto: Semáforo do Butantã v2.0 (com Contagem Regressiva)
 * Autor: [Seu Nome Completo]
 * Descrição: Controla um semáforo de 3 LEDs e um display TM1637
 * usando lógica não-bloqueante (millis())
 * ====================================================================
 */

// --- 1. Inclusão de Bibliotecas ---
#include <TM1637Display.h>

// --- 2. Definição de Pinos (Hardware) ---
// Pinos dos LEDs
const int pinoLedVermelho = 12;
const int pinoLedAmarelo  = 11;
const int pinoLedVerde    = 10;
// Pinos do Display TM1637
const int pinoDisplayCLK  = 9;
const int pinoDisplayDIO  = 8;

// --- 3. Definição de Tempos (Lógica) ---
const int TEMPO_VERMELHO = 6; // 6 segundos
const int TEMPO_VERDE    = 4; // 4 segundos
const int TEMPO_AMARELO  = 2; // 2 segundos

// --- 4. Definição de Estados (Máquina de Estados) ---
const int ESTADO_VERMELHO = 1;
const int ESTADO_VERDE    = 2;
const int ESTADO_AMARELO  = 3;

// --- 5. Variáveis Globais ---
// Inicializa o objeto do display
TM1637Display display(pinoDisplayCLK, pinoDisplayDIO);

int estadoAtual;           // Armazena o estado (cor) atual
int contadorTempo;         // Contador regressivo (o que aparece no display)
unsigned long ultimoTick;  // Armazena o tempo da última atualização (em ms)

// --- 6. Função de Configuração (setup) ---
void setup() {
  // Configura os pinos dos LEDs como SAÍDA
  pinMode(pinoLedVermelho, OUTPUT);
  pinMode(pinoLedAmarelo, OUTPUT);
  pinMode(pinoLedVerde, OUTPUT);

  // Configuração inicial do display
  display.setBrightness(0x0a); // Define o brilho (0x00 a 0x0f)
  display.clear();

  // Inicia o semáforo
  ultimoTick = millis(); // Marca o tempo inicial
  mudarEstado(ESTADO_VERMELHO); // Começa pelo estado Vermelho
}

// --- 7. Loop Principal (Execução contínua) ---
void loop() {
  
  // A mágica acontece aqui: Verificador de tempo não-bloqueante
  // "Já se passou 1 segundo (1000ms) desde o último tick?"
  if (millis() - ultimoTick >= 1000) {
    
    ultimoTick = millis(); // Marca o novo "último tick"
    contadorTempo--;       // Decrementa o contador
    
    display.showNumberDec(contadorTempo); // Atualiza o display

    // Verifica se o tempo para o estado atual acabou
    if (contadorTempo == 0) {
      // Se o tempo zerou, muda para o próximo estado
      if (estadoAtual == ESTADO_VERMELHO) {
        mudarEstado(ESTADO_VERDE);
      } else if (estadoAtual == ESTADO_VERDE) {
        mudarEstado(ESTADO_AMARELO);
      } else if (estadoAtual == ESTADO_AMARELO) {
        mudarEstado(ESTADO_VERMELHO); // Volta ao início do ciclo
      }
    }
  }
  
  // O resto do loop é livre para fazer outras tarefas (se necessário)
  // pois não está preso em um delay().
}

// --- 8. Função Auxiliar: Mudar Estado ---
// Esta função organiza a transição entre as fases do semáforo
void mudarEstado(int novoEstado) {
  
  estadoAtual = novoEstado; // Atualiza o estado global
  
  // Apaga todos os LEDs (boa prática antes de acender o próximo)
  digitalWrite(pinoLedVermelho, LOW);
  digitalWrite(pinoLedAmarelo, LOW);
  digitalWrite(pinoLedVerde, LOW);
  
  // Controla qual LED acender e qual tempo definir
  switch (estadoAtual) {
    
    case ESTADO_VERMELHO:
      digitalWrite(pinoLedVermelho, HIGH); // Acende o Vermelho
      contadorTempo = TEMPO_VERMELHO;      // Reinicia o contador
      break;
      
    case ESTADO_VERDE:
      digitalWrite(pinoLedVerde, HIGH);    // Acende o Verde
      contadorTempo = TEMPO_VERDE;       // Reinicia o contador
      break;
      
    case ESTADO_AMARELO:
      digitalWrite(pinoLedAmarelo, HIGH);  // Acende o Amarelo
      contadorTempo = TEMPO_AMARELO;     // Reinicia o contador
      break;
  }
  
  // Atualiza o display IMEDIATAMENTE com o novo tempo
  display.showNumberDec(contadorTempo);
}
```

---

## 4. Avaliação de Pares

Esta seção registra os resultados das avaliações realizadas por outros alunos.

| Avaliador | Montagem Física | Lógica do Código | Temporização (Vídeo) | Comentários Gerais |
|------------|------------------|------------------|------------------------|--------------------|
| [Nome do Avaliador 1] | [Nota/Feedback] | [Nota/Feedback] | [Nota/Feedback] | [Comentário geral do avaliador 1] |
| [Nome do Avaliador 2] | [Nota/Feedback] | [Nota/Feedback] | [Nota/Feedback] | [Comentário geral do avaliador 2] |

---
