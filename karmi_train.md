# 🧠 Documentación Técnica: `train.py` (Arquitectura CNN)

Este módulo define la arquitectura de la **Red Neuronal Convolucional (CNN)** y el pipeline de entrenamiento para la clasificación de estados oculares.

## 🏗️ Arquitectura de la Red (Model Summary)

El modelo es una arquitectura secuencial optimizada para entradas de **64x64x3**.

### 1. Preprocesamiento de Datos (Data Pipeline)
Para garantizar la invarianza lumínica, se implementa un ajuste dinámico antes de la normalización:
```python
img = cv2.addWeighted(img, 1.2, np.zeros(img.shape, img.dtype), 0, -30) # Factor 1.2 / Brillo -30
img = img / 255.0 # Normalización [0, 1]
```
*   **Impacto**: Al aumentar el contraste en el entrenamiento, el modelo se vuelve experto en detectar la esclerótica (parte blanca del ojo) y la pupila en condiciones de infrarrojos o baja luz.

### 2. Capas de Generalización (Augmentation)
```python
layers.RandomFlip("horizontal"),
layers.RandomRotation(0.1),
layers.RandomZoom(0.1)
```
*   **Explicación**: Estas capas se ejecutan solo durante el entrenamiento (en inferencia se ignoran). Generan variaciones sintéticas de los ojos para evitar que la red dependa de la orientación absoluta de la cabeza del usuario.

### 3. Bloques Convolucionales Estructurados
Se utilizan 3 bloques de profundidad creciente:
- **Bloque 1**: 32 filtros (Extrae texturas básicas y bordes).
- **Bloque 2**: 64 filtros (Detecta formas de párpados y comisuras).
- **Bloque 3**: 128 filtros (Relaciones espaciales complejas).
- **BatchNormalization**: Estabiliza el aprendizaje reduciendo el desplazamiento de la covarianza interna.

---

## 📉 Proceso de Entrenamiento y Optimización

### Hiperparámetros
- **Learning Rate**: `0.0005` (Bajo para permitir una convergencia más fina y evitar el olvido catastrófico).
- **Pérdida**: `binary_crossentropy` (Ideal para clasificación binaria).
- **Callback EarlyStopping**: `patience=8`. Si la pérdida de validación no mejora en 8 épocas, el entrenamiento se detiene para evitar el *overfitting*.

### Métricas de Evaluación para Desarrolladores
No solo medimos la precisión (`Accuracy`), sino también:
- **Precision**: Qué tan confiable es cuando el modelo dice que el ojo está cerrado.
- **Recall**: Qué tantos cierres de ojos reales es capaz de detectar el sistema sin ignorarlos.

---

## 💾 Salida de Modelo
El script genera `modelo_ojos_64.h5` en formato Keras. Este archivo es compatible con TensorFlow Lite si se desea portar a dispositivos móviles más adelante.
