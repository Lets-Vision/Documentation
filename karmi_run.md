# 👁️ Documentación Técnica: `run.py` (Inferencia e Integración)

Este es el orquestador principal del sistema. Su arquitectura es **multihilo (Asíncrona)** para garantizar una experiencia de usuario sin latencia perceptual.

## 🚀 Arquitectura de Inferencia

### 1. El Predictor Multihilo (`Predictor` class)
La inferencia de IA es el proceso que más CPU/GPU consume. Para evitar que el video se "congele" durante la predicción, se implementa un hilo demonio:
```python
threading.Thread(target=self._w, daemon=True).start()
```
*   **Lógica**: El hilo principal captura video y detecta rostros. Envía el recorte del ojo a una cola y continúa trabajando. El hilo del `Predictor` toma el recorte, predice y actualiza el resultado de forma atómica.
*   **Beneficio**: El usuario ve 30 FPS constantes mientras la IA predice a su máxima velocidad posible.

### 2. Detección Facial de Alta Estabilidad
En lugar de Haar Cascades (que son inestables ante rotaciones), usamos un modelo de landmarks:
```python
landmarker = mp.tasks.vision.FaceLandmarker.create_from_options(options)
```
*   **MediaPipe**: Proporciona una malla de 468 puntos. El script selecciona específicamente las coordenadas de las comisuras oculares (`L/R_CORNERS`) para calcular el centro dinámico del ojo.

### 3. Lógica de Gestos y Máquina de Estados
El script no envía datos binarios crudos, sino que procesa eventos:
- `p_counter`: Acumulador de frames para parpadeos (ambos ojos).
- `l/r_counter`: Acumuladores para guiños (wink).
- `MIN_FRAMES (5)`: Umbral para filtrar ruido o parpadeos involuntarios.
- `LONG_PRESS (25)`: Umbral para acciones de sistema como 'Enter' o 'Escape'.

---

## 📡 Integración de Red (WebSockets)

El servidor WebSocket se levanta en `localhost:8765`.

### API de Salida (JSON):
| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `type` | String | Tipo de mensaje (`eyes`, `guinho`, `parpadeo`) |
| `eyes` | Array | Lista de ojos abiertos actualmente |
| `long_press` | Bool | Solo en gestos, indica si el cierre fue sostenido |
| `duration` | Int | Cantidad de frames que duró el gesto |

---

## ⚙️ Depuración y Optimización para Devs

### Uso de Modelos ONNX
Si el sistema detecta archivos `.onnx`, utiliza `onnxruntime`. 
- **Modo CHW**: Los modelos ONNX suelen esperar el formato `(Batch, Canales, Alto, Ancho)`. El script transpone la imagen automáticamente:
```python
img = np.transpose(img, (2, 0, 1)) # De HWC a CHW
```

### Liberación de Recursos
El sistema incluye un sistema de limpieza activa para el puerto 8765 usando `taskkill` en sistemas Windows, asegurando que el servidor pueda reiniciar de inmediato tras un error `EADDRINUSE`.
