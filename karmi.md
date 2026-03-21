# Proyecto Karmi - Documentación Técnica Avanzada

Este documento proporciona una guía profunda sobre el funcionamiento interno de Karmi, explicando líneas clave de código y la ejecución de los modelos de IA.

---

## 🛠️ 1. Preparación de Imágenes (`feed.py`)

Este script transforma un dataset bruto en una estructura usable para entrenamiento.

### Líneas Clave Explicadas:
- **Línea 27: `partes = file.split("_")`**: El dataset MRL usa un formato de nombre específico. Esta línea descompone el nombre para extraer metadatos.
- **Línea 32: `estado_ojo = partes[4]`**: Accede al quinto elemento del nombre, que indica el estado (`0` para cerrado, `1` para abierto). Es el "cerebro" de la organización.
- **Línea 38/42: `shutil.copy(...)`**: En lugar de mover, copia los archivos. Esto permite mantener el dataset original intacto mientras se crea la carpeta `dataset/` estructurada.

---

## 🧠 2. Entrenamiento del Modelo (`train.py`)

Crea la red neuronal convolucional (CNN) adaptada a 64x64.

### Lógica de Preprocesado (Crucial para el éxito):
- **Línea 36: `cv2.addWeighted(img, 1.2, ..., -30)`**: Aplica un aumento de contraste del 20% y reduce el brillo. **¿Por qué?** Para que el modelo aprenda a ver pupilas incluso en condiciones de poca luz o con cámaras de baja calidad.

### Arquitectura de la Red:
- **Líneas 88-90 (Data Augmentation)**: `RandomFlip`, `RandomRotation`. Permite que la IA reconozca ojos incluso si el usuario inclina la cabeza o está de lado.
- **Líneas 96, 102, 108: `layers.Dropout(0.25)`**: Apaga aleatoriamente el 25% de las neuronas durante el entrenamiento. Esto evita que el modelo "se memorice" las fotos y lo obliga a aprender patrones reales.

---

## 👁️ 3. Ejecución en Tiempo Real (`run.py`)

El motor principal que procesa el video y envía los datos.

### Ejecución de Modelos (H5 vs ONNX):
El script detecta automáticamente el tipo de modelo por su extensión:
- **Modelos H5 (TensorFlow)**: Se ejecutan con `model.predict(b)`. Son los estándar, pero pueden ser pesados.
- **Modelos ONNX**: Se ejecutan con `m['model'].run(...)`. **Recomendado:** Son hasta 3 veces más rápidos en CPUs convencionales.

### Líneas Críticas en `run.py`:
- **Línea 29-45: `kill_previous()`**: Busca si el puerto `8765` de Windows está ocupado por una ejecución anterior colgada y lo "mata" forzosamente. Sin esto, el script fallaría al reiniciar.
- **Línea 162: `FaceLandmarker.create_from_options(...)`**: Carga el archivo `face_landmarker.task`. Es lo que permite pasar de detectar una "masa" a detectar 468 puntos faciales exactos.
- **Línea 292 (Sincronización)**: `cv2.addWeighted(img, 1.2, ..., -0.12)`. **Vital:** Aplica exactamente el mismo preproceso que en `train.py`. Si estas líneas no coinciden, la IA fallará al predecir.
- **Líneas 302-343: `class Predictor`**: Crea un hilo (thread) independiente. Mientras OpenCV muestra el video a 30 FPS, la IA puede ir a su propio ritmo sin congelar la imagen.

---

## 🚀 Cómo ejecutar modelos específicos

1. **Selección Dinámica**: Una vez abierto el programa, usa la barra deslizante **"Modelo"** en la parte superior de la ventana de video.
2. **Prioridad de archivos**: El sistema cargará todos los archivos `.h5` y `.onnx` que estén en la misma carpeta que `run.py`.
3. **Optimización**: Si quieres máxima fluidez, asegúrate de seleccionar `ocec_l.onnx` o `ocec_n.onnx` desde el selector en pantalla.

---

## 📡 Protocolo de Comunicación
El script emite un JSON continuo por WebSocket (`ws://localhost:8765`):
- `type: "eyes"`: Envía qué ojos están detectados como abiertos.
- `action: "guinho"`: Se dispara solo cuando se confirma un guiño tras pasar el umbral de frames.
