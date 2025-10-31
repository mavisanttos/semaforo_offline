# **Documentação: Semáforo Offline**

**Autor:** Maria Vitória dos Santos

**Data:** 31 de outubro de 2025  

**Disciplina/Contexto:** Programação

---
## 1. Objetivo do projeto

&emsp; O objetivo deste projeto é desenvolver um semáforo capaz de controlar o fluxo de trânsito por meio de uma temporização pré-definida para cada uma de suas três fases — vermelha, amarela e verde —, assegurando um ciclo contínuo de sinalização. Além disso, o sistema inclui um display que exibe a contagem regressiva de cada fase, proporcionando maior clareza e previsibilidade.

&emsp; O sistema segue a seguinte lógica de temporização:
- 6 segundos na luz vermelha
- 4 segundos na luz verde
- 2 segundos na luz amarela

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

- Passo 1: Coloque o Arduino Uno ao lado da protoboard e utilize jumpers Macho-Macho (marrons) para conectar um pino **GND** ao barramento azul da protoboard. Essa linha servirá como "linha negativa" do protótipo.

- Passo 2: Conecte os LEDs (Vermelho, Amarelo e Verde) utilizando jumpers Macho-Fêmea (cada jumper com a respectiva cor do seu LED de ligação). Utilizando a perna longa (Anodo, +), conecte cada um dos LEDs em uma fileira horizontal.

- Passo 3: Na mesma fileira onde os Anodos estão conectados, conecte um resistor de 10k Ω.

- Passo 4: Conecte a outra ponta de cada resistor, utilizando jumpers Macho-Macho (cada jumper com a respectiva cor do seu LED de ligação), aos respectivos pinos de sinais do Arduino:
  - Resistor do LED vermelho ➔ pino 12
  - Resistor do LED amarelo ➔ pino 11
  - Resistor do LED verde ➔ pino 10
 
- Passo 5: Utiliza jumpers Macho-Fêmea (marrons) e conecte a perna curta (Catodo, -) de cada LED diretamente à linha azul da protoboard, a qual definimos como GND do sistema.

- Passo 6: Conecte o Display LCD utilizando jumpers Macho-Fêmea:
  - VCC ➔ 5V do arduino (jumper laranja)
  - GND ➔ barramento azul (jumper marrom)
  - SDA ➔ pino A4 (jumper branco)
  - SCL ➔ pino A5 (jumper roxo)
 
- Passo 7: Encaixe os LEDs nos respectivos furos da estrutura de semáforo.

- Passo 8: Conecte o cabo USB ao Arduino e carregue o código disponível.

- Passo 9: O circuito deve funcionar com os LEDs acendendo na sequência correta e o display deve mostrar as mensagens e a contagem regressiva em sincronia. 

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
// Inclusão de Bibliotecas
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Definição da Classe 'Led'
class Led {
private:
  // Atributo: armazena o pino do LED
  int _pin;

public:
  // Construtor: é chamado quando um objeto Led é criado
  // "this->" é um ponteiro para o próprio objeto
  Led(int pin) {
    this->_pin = pin;
  }

  // Método: configura o pino como saída e desliga
  void setup() {
    pinMode(_pin, OUTPUT);
    apagar();
  }

  // Método: acende o LED
  void acender() {
    digitalWrite(_pin, HIGH);
  }

  // Método: apaga o LED
  void apagar() {
    digitalWrite(_pin, LOW);
  }
};

// Definição da Classe 'DisplayLCD' 
class DisplayLCD {
private:
  // Atributo: um ponteiro para o objeto real da biblioteca
  LiquidCrystal_I2C* _lcd;

public:
  // Construtor: recebe um PONTEIRO para um objeto LCD já criado
  DisplayLCD(LiquidCrystal_I2C* lcd) {
    this->_lcd = lcd;
  }

  // Método: inicializa o LCD
  void setup() {
    _lcd->init();
    _lcd->backlight();
    _lcd->clear();
  }

  // Método: limpa a tela
  void limpar() {
    _lcd->clear();
  }

  // Método: mostra a mensagem de estado
  void mostrarMensagemEstado(const char* msg, int coluna) {
    _lcd->setCursor(coluna, 0); // Linha 0
    _lcd->print(msg);
  }

  // Método: atualiza a contagem de tempo
  void atualizarTempo(int tempo) {
    _lcd->setCursor(4, 1); // Linha 1
    _lcd->print("Tempo: ");
    _lcd->print(tempo);
    _lcd->print("s  "); // Espaços para apagar lixo
  }
};

// Definição da Classe 'Semaforo'
class Semaforo {
private:
  // --- Atributos de Componentes (PONTEIROS) ---
  // A classe Semaforo possui LEDs e um Display
  Led* _ledVermelho;
  Led* _ledAmarelo;
  Led* _ledVerde;
  DisplayLCD* _display;
  LiquidCrystal_I2C* _lcdHardware;

  // Atributos de Estado
  enum Estado { VERMELHO, VERDE, AMARELO };
  Estado _estadoAtual;

  int _contadorTempo;
  unsigned long _ultimoTick;

  // Atributos de Configuração (Tempos)
  const int _tempoVermelho;
  const int _tempoVerde;
  const int _tempoAmarelo;

public:
  // Construtor: recebe os pinos e os tempos.
  // Usa uma "lista de inicialização" para os tempos (mais eficiente)
  Semaforo(int pinV, int pinA, int pinVd, int tV, int tA, int tVd, int lcdAddr)
    : _tempoVermelho(tV), _tempoVerde(tVd), _tempoAmarelo(tA)
  {
    // Usa 'new' para criar os objetos na memória (heap) e armazenamos seus endereços nos nossos ponteiros.
    _ledVermelho = new Led(pinV);
    _ledAmarelo  = new Led(pinA);
    _ledVerde    = new Led(pinVd);

    // Cria o objeto de hardware do LCD
    _lcdHardware = new LiquidCrystal_I2C(lcdAddr, 16, 2);
    // Cria nosso "wrapper" e passa o ponteiro para o hardware
    _display = new DisplayLCD(_lcdHardware);
  }

  // Destrutor
  ~Semaforo() {
    delete _ledVermelho;
    delete _ledAmarelo;
    delete _ledVerde;
    delete _display;
    delete _lcdHardware;
  }

  // Método: configura todos os componentes filhos
  void setup() {
    // Usa '->' (operador de ponteiro) para acessar métodos
    _ledVermelho->setup();
    _ledAmarelo->setup();
    _ledVerde->setup();
    _display->setup();

    _ultimoTick = millis();
    mudarEstado(VERMELHO); // Define o estado inicial
  }

  // Método: lógica principal (deve ser chamada no loop do Arduino)
  void loop() {
    // A lógica 'millis()' permanece a mesma,
    // mas agora está encapsulada dentro da classe.
    if (millis() - _ultimoTick >= 1000) {
      _ultimoTick = millis();
      _contadorTempo--;
      
      // Delega a tarefa de atualizar o tempo para o objeto Display
      _display->atualizarTempo(_contadorTempo);

      if (_contadorTempo == 0) {
        if (_estadoAtual == VERMELHO) {
          mudarEstado(VERDE);
        } else if (_estadoAtual == VERDE) {
          mudarEstado(AMARELO);
        } else if (_estadoAtual == AMARELO) {
          mudarEstado(VERMELHO);
        }
      }
    }
  }

private:
  // Método privado: só pode ser chamado de dentro desta classe
  void mudarEstado(Estado novoEstado) {
    _estadoAtual = novoEstado;

    // Apaga todos os LEDs
    _ledVermelho->apagar();
    _ledAmarelo->apagar();
    _ledVerde->apagar();
    _display->limpar();

    // Configura o novo estado
    switch (_estadoAtual) {
      case VERMELHO:
        _ledVermelho->acender();
        _contadorTempo = _tempoVermelho;
        _display->mostrarMensagemEstado("PARE!", 4);
        break;

      case VERDE:
        _ledVerde->acender();
        _contadorTempo = _tempoVerde;
        _display->mostrarMensagemEstado("SIGA!", 5);
        break;

      case AMARELO:
        _ledAmarelo->acender();
        _contadorTempo = _tempoAmarelo;
        _display->mostrarMensagemEstado("ATENCAO!", 3);
        break;
    }
    
    // Atualiza o tempo no display imediatamente
    _display->atualizarTempo(_contadorTempo);
  }
};

// Configuração Global e Loop Principal (Arduino)

// --- Constantes de Configuração ---
#define PINO_VERMELHO 12
#define PINO_AMARELO  11
#define PINO_VERDE    10
#define LCD_ENDERECO  0x27
#define TEMPO_VERMELHO 6
#define TEMPO_VERDE    4
#define TEMPO_AMARELO   2

// Objeto Principal
// Cria um ponteiro global para nosso objeto Semaforo
Semaforo* meuSemaforo;

void setup() {
  // Inicializa o objeto Semaforo na memória
  meuSemaforo = new Semaforo(
    PINO_VERMELHO, PINO_AMARELO, PINO_VERDE,
    TEMPO_VERMELHO, TEMPO_AMARELO, TEMPO_VERDE,
    LCD_ENDERECO
  );

  // Chama o método setup() do nosso objeto
  meuSemaforo->setup();
}

void loop() {
  // O loop principal do Arduino agora é limpo.
  // Ele apenas delega o trabalho para o método loop() do nosso objeto.
  meuSemaforo->loop();
}
```

---

### Programação Orientada a Objetos (POO)
&emsp; O conceito central da POO é o encapsulamento: agrupar dados (atributos) e comportamentos (métodos) que pertencem a uma mesma "coisa" em uma entidade chamada Classe. No projeto, identifica-se três entidades principais:

#### `class Led`
**Responsabilidade:**  
Sabe apenas como interagir com um único LED.

**Atributo:**  
- `_pin` → pino físico onde o LED está conectado.

**Métodos:**  
- `setup()`  
- `acender()`  
- `apagar()`

#### `class DisplayLCD`
**Responsabilidade:**  
Sabe apenas como interagir com o display LCD via interface I2C.

**Atributo:**  
- `_lcd` → ponteiro para o objeto da biblioteca que controla o display.

**Métodos:**  
- `setup()`  
- `limpar()`  
- `mostrarMensagemEstado()`  
- `atualizarTempo()`

#### `class Semaforo`
**Responsabilidade:**  
Funciona como a **"orquestradora"** ou **"cérebro"** do sistema.  
Contém toda a lógica de negócios — a **máquina de estados** e a **temporização** do semáforo.

**Atributos:**  
- Ponteiros para os objetos `Led` e `DisplayLCD` que ela gerencia  
- `_estadoAtual`  
- `_contadorTempo`  
- `_ultimoTick`

**Métodos:**  
- `setup()`  
- `loop()`  
- `mudarEstado()`

##### Vantagem da Arquitetura

&emsp; A principal vantagem dessa estrutura é percebida no `loop()` principal do Arduino:  

```c
void loop() {
  // O loop principal agora é limpo.
  // Ele apenas delega o trabalho para o objeto.
  meuSemaforo->loop();
}
```

&emsp; Toda a complexidade da lógica millis(), da troca de estados e do controle dos componentes está encapsulada dentro do objeto meuSemaforo.

### Ponteiros

&emsp; Os ponteiros são variáveis que, em vez de armazenar um valor (como `int = 5`), armazenam um **endereço de memória**. Eles são a ferramenta que nos permite implementar a **POO (Programação Orientada a Objetos)** de forma eficiente no **C++**.

#### `new` (Alocação Dinâmica)

**O que faz:**  
```cpp
_ledVermelho = new Led(pinV);
```

&emsp; O comando `new` aloca espaço na memória (chamada **Heap**) para um objeto `Led` e nos retorna o endereço onde esse objeto foi criado. Armazena esse endereço no ponteiro `_ledVermelho`. Isso dá controle total sobre o **tempo de vida** do objeto.


#### `*` (Declaração de Ponteiro)

**O que faz:**
```cpp
Led* _ledVermelho;
```

&emsp; O asterisco (`*`) diz ao compilador: “Esta variável não é um `Led`, mas sim um **ponteiro para um `Led`**.” Ela armazena o endereço que recebemos do `new`.

#### `->` (Operador Seta)

**O que faz:**
```cpp
_ledVermelho->acender();
```

&emsp; Quando temos um ponteiro, não podemos usar o ponto (`.`) para acessar seus métodos. Usamos a seta (`->`) para dizer: “Vá até o endereço de memória armazenado em `_ledVermelho` e, no objeto que está lá, chame o método `acender()`.”

#### `this->` (Ponteiro Interno)

**O que faz:**
```cpp
this->_pin = pin; // usado dentro do construtor da classe Led
```

&emsp; `this` é um ponteiro especial que todo objeto tem e que **aponta para si mesmo**. Usamos `this->_pin` para diferenciar o **atributo da classe (`_pin`)** do **parâmetro recebido pela função (`pin`)**.

##### Vantagens desta Arquitetura

**Manutenibilidade:**  
Se o LED vermelho parar de funcionar, sabemos que o problema está na fiação do pino 12 ou no código dentro da classe `Led` (ou `Semaforo`), e não perdido no meio da lógica do display.

**Reutilização:**  
Se quiséssemos adicionar um segundo semáforo, bastaria criar uma nova instância no `setup()`:
```cpp
meuSemaforoPedestre = new Semaforo(pinX, pinY, ...);
```
Não precisaríamos reescrever código algum — apenas chamá-lo no `loop()`:

```cpp
meuSemaforoPedestre->loop();
```

**Abstração:**
O loop() principal não precisa saber como um estado é mudado; ele apenas confia que o objeto meuSemaforo sabe fazer seu trabalho.

---

## 4. Avaliação de Pares

Esta seção registra os resultados das avaliações realizadas por outros alunos.

| Avaliador | Montagem Física | Lógica do Código | Temporização (Vídeo) | Comentários Gerais |
|------------|------------------|------------------|------------------------|--------------------|
| [Nome do Avaliador 1] | [Nota/Feedback] | [Nota/Feedback] | [Nota/Feedback] | [Comentário geral do avaliador 1] |
| [Nome do Avaliador 2] | [Nota/Feedback] | [Nota/Feedback] | [Nota/Feedback] | [Comentário geral do avaliador 2] |

---
