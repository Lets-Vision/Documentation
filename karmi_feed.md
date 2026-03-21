# 📂 Documentación Técnica: `feed.py` (ETL de Datos)

Este módulo implementa el proceso de **Extracción, Transformación y Carga (ETL)** necesario para preparar el dataset MRL Eye para redes neuronales de clasificación binaria.

## 🏗️ Lógica Estructural

El script está diseñado para procesar el dataset **MRL Eye**, el cual contiene más de 84,000 imágenes. La complejidad reside en que el dataset viene en una estructura de carpetas por sujeto (s0001, s0002...), pero para el entrenamiento con `Keras` o `FastAPI`, necesitamos una estructura basada en clases (subdirectorios por categoría).

### 🔬 Análisis Profundo de Funciones

#### 1. Gestión de Nomenclatura del Dataset MRL
El nombre del archivo en el dataset MRL sigue el patrón:
`s[ID]_[GENDER]_[AGE]_[GLASSES]_[EYE_STATE]_[REFLECTIONS]_[LIGHTING]_[ID_IMG].png`

```python
partes = file.split("_")
estado_ojo = partes[4] # Índice 4: Estado del ojo
```
*   **Decisión de Ingeniería**: Se utiliza el índice `4` como único discriminador para definir la etiqueta (`label`). 
    *   `1` -> Mapeado a la clase `abiertos/`.
    *   `0` -> Mapeado a la clase `cerrados/`.
*   **Robustez**: El script incluye un bloque `try-except IndexError` para ignorar archivos que no cumplan con el estándar de nomenclatura de 8 campos, evitando que el proceso se detenga por archivos residuales o corruptos.

#### 2. Implementación de `os.walk` para Recursividad
```python
for root, dirs, files in os.walk(ruta_origen):
```
*   **Eficiencia**: `os.walk` es un generador, lo que permite al script empezar a copiar archivos inmediatamente sin tener que indexar los 84,000 archivos en memoria primero, lo cual es crítico en sistemas con RAM limitada.

---

## 🛠️ Flujo de Ejecución para Desarrolladores

1.  **Instalación de Dependencias**: Requiere `python` y la librería `shutil` (estándar). No requiere GPU para esta fase.
2.  **Configuración de Rutas**: Modificar la variable `ruta_origen` al inicio del script si el dataset se descargó con un nombre distinto.
3.  **Ejecución**: No requiere parámetros por línea de comandos. 

---

## ⚠️ Consideraciones de Almacenamiento
*   **Duplicación vs Movimiento**: Se utiliza `shutil.copy` en lugar de `shutil.move`. 
    *   *Razón*: El movimiento de archivos es una operación atómica dentro del mismo volumen, pero si el proceso se interrumpe, el dataset original queda incompleto. El copiado garantiza la integridad del origen.
*   **Espacio en Disco**: Asegúrate de tener al menos **2GB extra** de espacio libre, ya que se crearán copias de las imágenes procesadas.
