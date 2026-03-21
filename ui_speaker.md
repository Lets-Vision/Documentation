# 🎙️ Documentación del Módulo: Speaker (`main.jsx`)

El módulo **Speaker** es una herramienta de síntesis de voz asistida diseñada para personas con discapacidades motoras severas. Utiliza una interfaz de "Teclado por Rangos" optimizada para el sistema de seguimiento ocular Karmi.

## 🏗️ Arquitectura de la Interfaz

La aplicación se basa en un diseño de **5 botones principales** en disposición horizontal. Esta configuración minimiza los movimientos oculares necesarios y maximiza el tamaño de los objetivos de selección.

### 🧩 Componentes Destacados

#### 1. Monitor Ocular (`EyeMonitor`)
Un componente visual que representa el estado de apertura de los ojos en tiempo real.
- **Lógica**: Utiliza un gradiente radial `rgba(0,230,118,${value})` donde `value` es el nivel de apertura enviado por el tracker.
- **Feedback**: Si el valor es > 0.6, se considera "abierto" y se aplica un efecto de brillo (`boxShadow`).

#### 2. Sistema de Diccionario Dinámico
La aplicación carga un archivo de texto con miles de palabras en español para ofrecer autocompletado inteligente.
```javascript
const patron = secuenciaRangos.map((r) => r.regex).join(".*");
const regex = new RegExp(`^${patron}`, "i");
```
*   **Funcionamiento**: Al seleccionar un rango (ej. "ABCDEF"), el sistema filtra el diccionario. Al seleccionar un segundo rango (ej. "GHIJKLM"), busca palabras que empiecen por una letra del primer grupo seguida de una del segundo.

---

## 👁️ Integración con Karmi (EyeGestureContext)

Esta aplicación no lee directamente del WebSocket, sino que consume el `EyeGestureContext`.

### 1. Consumo de Datos
```javascript
const { eyes, eyeData, lastAction, switchModel } = useEyeGestureData();
```
- **`eyes`**: Lista de ojos detectados.
- **`eyeData`**: Valores numéricos (0.0 a 1.0) de apertura.
- **`lastAction`**: El último evento de gesto (parpadeo, guiño o navegación).

### 2. Timer de Auto-Selección (Dwell Time)
Para evitar la necesidad de un "click" físico, la interfaz utiliza un sistema de espera (Dwell Time) de **2.5 segundos**.
```javascript
timerRef.current = setTimeout(() => {
    ejecutarAccion(actuales[indiceFoco]);
}, 2500);
```
- **Lógica**: Si el cursor se mantiene sobre un botón durante 2.5s, la acción se ejecuta automáticamente. Una barra de progreso verde en la parte inferior del botón indica visualmente el tiempo restante.

---

## 🤖 Integración con IA (Asistente Ollama)

El botón **ASISTENTE** permite pasar el texto construido a un modelo de lenguaje local (Ollama).
- **Endpoint**: `http://localhost:11434/api/generate`
- **Modelo**: `armador1`
- **Propósito**: Tomar palabras sueltas o frases incompletas y transformarlas en oraciones gramaticalmente correctas y naturales antes de ser habladas.

---

## ⚙️ Flujos de Usuario (Modos)

1.  **PRINCIPAL**: Selección de rangos de letras u acceso al menú de opciones.
2.  **MENU_OPCIONES**: Acciones de sistema (Hablar, Borrar, Asistente).
3.  **SELECCION_PALABRA**: Visualización de las sugerencias del diccionario filtradas por los rangos seleccionados.

---

## 🎨 Estética y UX
- **Colores**: Fondo `#0a0a0a` (negro puro) para reducir la fatiga ocular y botones con bordes `#00e676` (verde neón) para un alto contraste.
- **Accesibilidad**: Feedback por voz integrado; el sistema anuncia "Opciones" al navegar sobre el botón correspondiente para guiar al usuario sin que tenga que mirar fijamente las etiquetas pequeñas.
