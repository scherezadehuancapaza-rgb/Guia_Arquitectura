## Calidad de código con Flake8

Este proyecto usa [Flake8](https://flake8.pycqa.org/) para revisar el estilo y detectar errores en el código Python **antes** de que se conviertan en problemas (y en refactorizaciones costosas).

### Por qué usarlo: prevenir sale más barato que refactorizar

Refactorizar un código enredado meses después implica entender qué hacía cada parte, miedo a romper algo y horas de pruebas. Si el linter avisa mientras se escribe, se corrige en minutos, cuando todavía se tiene el contexto fresco.

Cómo sacarle más provecho:

1. **Que se ejecute solo**: con la extensión de Flake8 en VS Code los avisos aparecen subrayados en vivo, y con `pre-commit` Flake8 corre antes de cada commit y no deja pasar código problemático.
2. **Dejar la configuración fija en el proyecto**: un archivo `.flake8` en la raíz (ver sección "Configuración del proyecto") hace que cualquiera que abra el proyecto use las mismas reglas.
3. **Combinarlo con un formateador**: `black` arregla el formato (espacios, líneas largas, comillas) y Flake8 se concentra en lo que black no puede arreglar: variables sin usar, nombres no definidos y complejidad.
4. **Crear el hábito**: cuando salte un C901, no ignorarlo; es el momento justo para extraer una parte a una función con nombre claro. Ese pequeño paso, repetido, evita la gran refactorización futura.

> Flake8 no reemplaza el criterio: no sabe si los nombres son claros ni si la lógica tiene sentido, así que revisar el propio código de vez en cuando sigue siendo parte del trabajo.

### Instalación

```bash
pip install flake8
```

### Uso

```bash
# Revisar todo el proyecto
flake8 .

# Revisar un archivo
flake8 app.py

# Ver resumen con totales por tipo de error
flake8 . --statistics --count
```

### Cómo leer el reporte

Cada línea del reporte tiene este formato:

```
archivo.py:LÍNEA:COLUMNA: CÓDIGO texto final
```

Ejemplo:

```
app.py:3:1: F401 'os' imported but unused
app.py:10:80: E501 line too long (95 > 79 characters)
```

- **archivo.py:LÍNEA:COLUMNA**: dónde está el problema.
- **CÓDIGO**: letra + número que identifica la regla.
- **texto final**: descripción del problema.

### Indicadores principales

| Letra | Origen | Qué indica |
|-------|--------|------------|
| **F** | Pyflakes | Errores lógicos reales (los más importantes) |
| **E** | pycodestyle | Errores de estilo según PEP 8 |
| **W** | pycodestyle | Advertencias de estilo |
| **C** | McCabe | Complejidad del código |

#### F: errores lógicos (prioridad alta)

| Código | Significado |
|--------|-------------|
| F401 | Import no usado |
| F811 | Redefinición de un nombre sin usar |
| F821 | Nombre no definido (variable o función inexistente) |
| F841 | Variable local asignada pero nunca usada |
| F403 / F405 | `from x import *` y nombres que pueden venir de ahí |
| F541 | f-string sin ningún marcador `{}` |

#### E: errores de estilo

| Código | Significado |
|--------|-------------|
| E501 | Línea demasiado larga (más de 79 caracteres por defecto) |
| E302 | Faltan 2 líneas en blanco antes de una función o clase |
| E303 | Demasiadas líneas en blanco |
| E225 | Falta espacio alrededor de un operador |
| E231 | Falta espacio después de `,` |
| E265 | Comentario de bloque mal formado (falta espacio tras `#`) |
| E711 / E712 | Comparar con `None` o `True` usando `==` |
| E722 | `except:` sin especificar el tipo de excepción |
| E999 | Error de sintaxis (el archivo no se pudo analizar) |

#### W: advertencias

| Código | Significado |
|--------|-------------|
| W291 | Espacios al final de la línea |
| W293 | Línea en blanco con espacios |
| W391 | Línea en blanco al final del archivo |
| W605 | Secuencia de escape inválida en un string (ej. `"\d"`) |

#### C: complejidad

| Código | Significado |
|--------|-------------|
| C901 | La función es demasiado compleja (requiere `--max-complexity`) |

### Aclaración sobre E501

E501 **no mide cuánto código hay**, solo el **ancho de cada línea**. Aun así, una línea muy larga suele ser señal de que se está haciendo demasiado en un solo lugar, por lo que conviene respetar el límite.

Para detectar código espagueti de verdad se usa **C901** (complejidad ciclomática), que cuenta los caminos distintos dentro de una función (`if`, `for`, `while`, `except`...). Si una función supera el límite, conviene dividirla en funciones más pequeñas.

### Configuración del proyecto

Archivo `.flake8` en la raíz del repositorio:

```ini
[flake8]
max-line-length = 88
max-complexity = 10
exclude = .git,__pycache__,venv
```

- `max-line-length`: largo máximo de línea (79 es el estándar PEP 8; 88 es el valor de `black`).
- `max-complexity`: activa C901 y define el límite por función.
- `exclude`: carpetas que no se revisan.

Si se usa `black` como formateador, agregar también:

```ini
extend-ignore = E203
```

### Cómo corregir un E501

```python
# Dentro de paréntesis se puede partir la línea
resultado = funcion_larga(
    argumento_uno,
    argumento_dos,
    argumento_tres,
)

# Strings largos: se dividen y Python los une solos
mensaje = (
    "Este es un texto muy largo "
    "que se reparte en varias líneas"
)
```

### Buenas prácticas

- Prioriza los errores **F** y **E999**: son errores reales.
- Los **E** y **W** son de formato y se pueden corregir automáticamente con `black` o `autopep8`.
- No desactives E501 por completo; mejor ajusta `max-line-length`.
- Cuando salte un **C901**, extrae parte del código a una función con nombre claro.
- Regla práctica: cada función debe hacer una sola cosa y poder describirse en una frase.

### Ignorar reglas puntuales

```bash
# Desde la línea de comandos
flake8 --ignore=E501,W503

# En una línea específica del código
import os  # noqa: F401
```

### Integración con VS Code

1. Instalar la extensión de Python (o la de Flake8).
2. Los avisos aparecen subrayados mientras se escribe.
3. Opcional: usar `pre-commit` para ejecutar Flake8 antes de cada commit.
