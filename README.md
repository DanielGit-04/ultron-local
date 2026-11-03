# ultron-local

Ultrón es una IA personal local inspirada en JARVIS, pensada para ayudarte en tareas diarias y de estudio con foco en privacidad.

## Objetivo del proyecto

- Activación por voz (tu propia voz) y conversación natural.
- Asistencia práctica: clima, estudio, noticias relevantes y apoyo general.
- Personalidad propia, con capacidad de contradecir decisiones peligrosas.
- Respuesta en tiempo real ante cambios y eventos.
- Conocimiento amplio apoyado por fuentes objetivas y verificables.

## Principios no negociables

1. **Privacidad local por defecto**  
   Las conversaciones se procesan y almacenan en local.
2. **Internet con mínima exposición**  
   Solo se envían consultas técnicas necesarias (por ejemplo, clima o RSS).
3. **Nada de IA en nube para conversación privada**  
   Se prioriza modelo local (ej. Ollama) para evitar compartir chats con terceros.
4. **Fuentes verificables**  
   La información externa debe venir de documentación o fuentes contrastables.
5. **Seguridad operativa**  
   Confirmación explícita antes de acciones sensibles.

## Arquitectura recomendada (MVP)

```text
Micrófono
  -> Wake word (local)
  -> Voz a texto (local)
  -> Modelo local (Ollama)
  -> Herramientas (clima, noticias, estudio)
  -> Texto a voz (local)
```

## Empezar paso a paso (para no programadores)

### 1) Requisitos mínimos recomendados

- Windows 11 x64
- 16 GB RAM o más (32 GB ideal)
- 20–50 GB libres en disco

### 2) Instalar Ollama

1. Descargar: `https://ollama.com/download/windows`
2. Instalar con opciones por defecto.
3. Abrir PowerShell y ejecutar:

```powershell
ollama run qwen3:8b
```

Esto descarga y ejecuta un modelo local.

### 3) Instalar Python

1. Descargar: `https://www.python.org/downloads/windows/`
2. Marcar **Add python.exe to PATH**
3. Instalar y verificar:

```powershell
python --version
```

### 4) Crear proyecto local

```powershell
cd $HOME\Documents
mkdir ultron-local
cd ultron-local
python -m venv .venv
```

Activar entorno virtual:

```powershell
.\.venv\Scripts\Activate.ps1
```

Si PowerShell bloquea scripts:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

### 5) Instalar dependencia mínima

```powershell
python -m pip install requests
```

### 6) Crear primer `main.py`

```python
import requests

MODELO = "qwen3:8b"
DIRECCION_OLLAMA = "http://localhost:11434/api/chat"

def preguntar_a_ultron(pregunta):
    respuesta = requests.post(
        DIRECCION_OLLAMA,
        json={
            "model": MODELO,
            "stream": False,
            "messages": [
                {
                    "role": "system",
                    "content": (
                        "Tu nombre es Ultron. Responde siempre en español, "
                        "de forma clara y útil. Si no sabes algo, dilo. "
                        "No inventes información."
                    ),
                },
                {
                    "role": "user",
                    "content": pregunta,
                },
            ],
        },
        timeout=120,
    )

    respuesta.raise_for_status()
    datos = respuesta.json()
    return datos["message"]["content"]


print("Ultron está preparado.")
print("Escribe 'salir' para terminar.\n")

while True:
    pregunta = input("Tú: ").strip()

    if pregunta.lower() == "salir":
        print("Ultron: Hasta luego.")
        break

    if not pregunta:
        continue

    try:
        respuesta = preguntar_a_ultron(pregunta)
        print(f"\nUltron: {respuesta}\n")
    except requests.exceptions.ConnectionError:
        print(
            "\nNo puedo conectar con Ollama. "
            "Comprueba que Ollama está abierto.\n"
        )
    except requests.exceptions.RequestException as error:
        print(f"\nHa ocurrido un error: {error}\n")
```

### 7) Ejecutar

Con Ollama activo en segundo plano:

```powershell
python main.py
```

Prueba:

```text
Hola Ultron, ¿quién eres?
```

## Qué hace cada bloque del código

- `import requests`: permite conectar Python con Ollama local.
- `MODELO`: define qué modelo usar.
- `DIRECCION_OLLAMA`: endpoint local (`localhost`) del servidor Ollama.
- `preguntar_a_ultron(...)`: envía pregunta y devuelve respuesta.
- `while True`: mantiene el chat activo.
- `try/except`: muestra errores claros si Ollama no está disponible.

## Roadmap recomendado

1. Chat por texto (hecho en MVP)
2. Voz a texto local (`faster-whisper`)
3. Texto a voz local (`piper`)
4. Wake word local (`openWakeWord`)
5. Herramientas (clima, noticias RSS, estudio)
6. Memoria local (SQLite cifrado)
7. Automatización con permisos y confirmación

## Recordatorio de privacidad realista

Si Ultrón consulta internet, algunos metadatos técnicos (IP, dominio consultado) pueden salir del equipo.  
Objetivo de este proyecto: que **el contenido de tus conversaciones** permanezca local y que internet reciba solo lo mínimo imprescindible.
