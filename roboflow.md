```
pip install opencv-python requests pyserial
```

## Solicitando API Roboflow e mostrando a imagem

```
import cv2, requests

API_KEY = "v9v7VyiohQ8xxki5hUZs"
MODEL = "seven-segments-display-digits-sn4km/1"

img = cv2.imread("multimetro-de-alta-precisao-dt-9919-cem-true-rms-53a7b060d00d87de6517464774056546-640-0.webp")

# envia pra API
_, img_encoded = cv2.imencode(".jpg", img)
res = requests.post(
    f"https://serverless.roboflow.com/{MODEL}",
    params={"api_key": API_KEY},
    files={"file": img_encoded.tobytes()}
).json()

preds = sorted(res.get("predictions", []), key=lambda p: p["x"])

# monta valor
valor = "".join(str(p["class"]) for p in preds)
print("Valor:", valor)

# desenha
for i, p in enumerate(preds):
    x, y, w, h = int(p["x"]), int(p["y"]), int(p["width"]), int(p["height"])
    x1, y1 = int(x - w/2), int(y - h/2)
    x2, y2 = int(x + w/2), int(y + h/2)

    cv2.rectangle(img, (x1, y1), (x2, y2), (0,255,0), 1)
    cv2.putText(img, str(p["class"]), (x1, y1-5),
                cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0,255,0), 1)

# mostra e salva
cv2.imshow("Resultado", img)
cv2.imwrite("resultado.jpg", img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
## Solicitando API Roboflow e mostrando json
```
import cv2, requests

img = cv2.imread("dog.960.jpg")
_, img_encoded = cv2.imencode(".jpg", img)

res = requests.post(
    "https://serverless.roboflow.com/fiap-lgfiw/3",
    params={"api_key": "v9v7VyiohQ8xxki5hUZs"},
    files={"file": img_encoded.tobytes()}
)

print(res.json())
```

## Solicitando API Roboflow e mostrando json e ligando um led
```
import cv2, requests, serial, time

# ===== CONFIG =====
API_KEY = "v9v7VyiohQ8xxki5hUZs"
MODEL = "fiap-lgfiw/3"
PORTA = "COM12"

# conecta Arduino
arduino = serial.Serial(PORTA, 9600)
time.sleep(2)

# lê imagem
img = cv2.imread("dog.960.jpg")
_, img_encoded = cv2.imencode(".jpg", img)

# inferência
res = requests.post(
    f"https://serverless.roboflow.com/{MODEL}",
    params={"api_key": API_KEY},
    files={"file": img_encoded.tobytes()}
).json()

# pega previsões ordenadas
preds = sorted(res.get("predictions", []), key=lambda p: p["x"])

# monta valor
valor = "".join(str(p["class"]) for p in preds)
print("Valor detectado:", valor)

# ===== REGRA =====
if str(valor) == "dogs":
    arduino.write(b'l')
    print("LED LIGADO")
else:
    arduino.write(b'd')
    print("LED DESLIGADO")
```

## camera
```
import cv2
import requests
import serial
import time

# ===== CONFIG =====
API_KEY = "v9v7VyiohQ8xxki5hUZs"
MODEL = "fiap-lgfiw/3"
PORTA = "COM12"
INTERVALO = 60.0  # segundos entre inferências

# conecta Arduino
arduino = serial.Serial(PORTA, 9600)
time.sleep(2)

# abre webcam
cap = cv2.VideoCapture(0)

if not cap.isOpened():
    print("Erro ao abrir a webcam.")
    exit()

ultimo_envio = 0

while True:
    ret, frame = cap.read()
    if not ret:
        print("Erro ao capturar frame.")
        break

    agora = time.time()

    # faz inferência a cada INTERVALO segundos
    if agora - ultimo_envio >= INTERVALO:
        ultimo_envio = agora

        _, img_encoded = cv2.imencode(".jpg", frame)

        try:
            res = requests.post(
                f"https://serverless.roboflow.com/{MODEL}",
                params={"api_key": API_KEY},
                files={"file": img_encoded.tobytes()},
                timeout=10
            ).json()

            preds = res.get("predictions", [])
            classes = [str(p["class"]).strip().lower() for p in preds]

            print("Classes detectadas:", classes)

            if "dogs" in classes:
                arduino.write(b'l')
                print("LED LIGADO")
            else:
                arduino.write(b'd')
                print("LED DESLIGADO")

            # desenha boxes
            for p in preds:
                x = int(p["x"])
                y = int(p["y"])
                w = int(p["width"])
                h = int(p["height"])
                label = str(p["class"])
                conf = float(p["confidence"])

                x1 = int(x - w / 2)
                y1 = int(y - h / 2)
                x2 = int(x + w / 2)
                y2 = int(y + h / 2)

                cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 0), 2)
                cv2.putText(
                    frame,
                    f"{label} {conf:.2f}",
                    (x1, max(y1 - 10, 20)),
                    cv2.FONT_HERSHEY_SIMPLEX,
                    0.6,
                    (0, 255, 0),
                    2
                )

        except Exception as e:
            print("Erro na inferência:", e)

    cv2.imshow("Webcam", frame)

    # tecla q para sair
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
arduino.close()
```
