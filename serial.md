# Controle de LED via Serial (Python + Arduino)

## Descrição
Este projeto permite controlar um LED no Arduino utilizando Python via comunicação serial.

- `l` → Liga o LED  
- `d` → Desliga o LED  

---

## Código Python (VS Code)

```python
import serial  # Importa a biblioteca

# Loop para conexão com o Arduino
while True:
    try:
        arduino = serial.Serial('COM12', 9600)
        print('Arduino conectado')
        break
    except:
        pass

# Loop principal
while True:
    cmd = input('Digite "l" para ligar e "d" para desligar: ')

    if cmd == 'l':
        arduino.write('l'.encode())

    elif cmd == 'd':
        arduino.write('d'.encode())

    arduino.flush()
```

## Código Arduino

```
char comando;

void setup() {
  Serial.begin(9600);   // mesma velocidade do Python
  pinMode(13, OUTPUT);  // LED (pino 13)
}

void loop() {
  if (Serial.available() > 0) {
    comando = Serial.read();

    if (comando == 'l') {
      digitalWrite(13, HIGH);  // liga LED
      Serial.println("LED LIGADO");
    }

    else if (comando == 'd') {
      digitalWrite(13, LOW);   // desliga LED
      Serial.println("LED DESLIGADO");
    }
  }
}
```
