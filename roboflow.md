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

