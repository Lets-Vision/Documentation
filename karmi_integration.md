# 🔌 Guía de Integración: `run.py` con Otras Aplicaciones

Para integrar el rastreo ocular de Karmi en cualquier aplicación externa (Web, Desktop o Mobile), no es necesario modificar el código de Python. `run.py` funciona como un **Microservicio de Visión** independiente.

## 📡 Protocolo de Conexión

El script `run.py` levanta un servidor WebSocket en la siguiente dirección:
- **URL**: `ws://localhost:8765`

Toda aplicación externa debe actuar como un **Cliente WebSocket** para recibir los eventos en tiempo real.

---

## 💻 Implementación en JavaScript / React

Dado que Lets Vision utiliza React, esta es la forma estándar de integrarlo:

### 1. Hook de Conexión Base
```javascript
import { useEffect, useState } from 'react';

const useKarmiTracker = () => {
    const [eyeState, setEyeState] = useState({ eyes: [], lastEvent: null });

    useEffect(() => {
        const socket = new WebSocket('ws://localhost:8765');

        socket.onmessage = (event) => {
            const data = JSON.parse(event.data);
            
            // Si el mensaje es de tipo 'eyes', actualizamos el estado visual
            if (data.type === 'eyes') {
                setEyeState(prev => ({ ...prev, eyes: data.eyes }));
            } 
            // Si es un gesto (guinho o parpadeo), disparamos una acción
            else {
                setEyeState(prev => ({ ...prev, lastEvent: data }));
                handleGesture(data);
            }
        };

        return () => socket.close();
    }, []);

    const handleGesture = (gesture) => {
        console.log("Gesto detectado:", gesture.action);
        if (gesture.action === 'guinho' && gesture.type === 'derecho') {
            // Acción para guiño derecho (ej: Siguiente página)
        }
    };

    return eyeState;
};
```

---

## 🐍 Implementación en Python (Cliente)

Si estás creando otra herramienta en Python y quieres recibir los datos de Karmi:

```python
import asyncio
import websockets
import json

async def listen_karmi():
    uri = "ws://localhost:8765"
    async with websockets.connect(uri) as websocket:
        while True:
            message = await websocket.recv()
            data = json.loads(message)
            
            if "action" in data:
                print(f"🔥 Evento: {data['action']} | Duración: {data['duration']} frames")

asyncio.get_event_loop().run_until_complete(listen_karmi())
```

---

## 🛠️ Comandos de Control (Bidireccional)

`run.py` no solo envía datos, también puede **recibir órdenes** del cliente para cambiar su comportamiento dinámicamente:

### Cambiar de modelo de IA desde el Cliente
Puedes enviar un JSON al socket para que el tracker cambie de modelo (ej: cambiar de un modelo .h5 a uno .onnx más rápido):

```javascript
const switchModel = (index) => {
    socket.send(JSON.stringify({
        action: "switch_model",
        index: index // Índice del modelo en la lista de MODELS
    }));
};
```

---

## 🏗️ Flujo de Trabajo Recomendado

1.  **Lanzar el Tracker**: Ejecuta `run.py` (o `start.bat`).
2.  **Verificar el Puerto**: Asegúrate de que ninguna otra aplicación (como otro servidor web) esté usando el puerto `8765`.
3.  **Conexión**: Inicia tu aplicación. Al conectar el WebSocket, verás en la consola de Python el mensaje `✅ Cliente conectado`.
4.  **Consumo**: Filtra los mensajes por `type`. Usa `eyes` para animaciones visuales (como un avatar que parpadea) y los objetos con `action` para lógica de navegación (Enter, Escape, Flechas).
