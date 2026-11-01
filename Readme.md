´´´
Manual de Arquitectura Base (Clean Architecture + Flask)
Este manual describe cómo configurar la estructura inicial del proyecto para cumplir con los principios SOLID, Código Limpio y las buenas prácticas de la industria.

Fase 1: Configuración del Entorno y Seguridad
El objetivo es aislar el proyecto y proteger las credenciales desde el día 1.

Crear el entorno virtual:

bash
python -m venv venv
# Activarlo (Windows): .\venv\Scripts\activate
Crear requirements.txt: Listar las dependencias fijando versiones para evitar conflictos futuros.

Configurar el .gitignore:

Ignorar siempre: venv/, __pycache__/, .pytest_cache/, *.db (bases de datos locales) y .env (secretos).

Gestión de Secretos (.env.example):

Regla de oro: NUNCA escribir contraseñas o claves secretas en el código (evitar Hardcoded Secrets).

Crear un .env.example con los nombres de las variables (ej. FLASK_SECRET_KEY=, SMTP_CLAVE=).

Cada desarrollador debe crear su propio .env local.

Linter Config (.flake8): Crear el archivo para configurar la longitud máxima de línea (ej. 120 caracteres) y excluir carpetas pesadas.

Fase 2: Estructura de Carpetas (Clean Architecture)
El objetivo es aplicar el Principio de Responsabilidad Única (SRP) separando el código por capas lógicas.

Crear la siguiente estructura de directorios:
```text
text
proyecto/
├── datos/               # Archivos Excel/CSV (Insumos)
├── dominio/             # CAPA 1: Reglas de negocio puras y dataclasses
├── aplicacion/          # CAPA 2: Servicios y casos de uso
├── infraestructura/     # CAPA 3: Base de datos, APIs externas, correos
├── interfaces/          # CAPA 4: Vistas web (Rutas, HTML, CSS, JS)
├── pruebas/             # Tests unitarios (Pytest)
├── app.py               # Punto de entrada (App Factory)
├── configuracion.py     # Variables de entorno y configuraciones
└── importar_datos.py    # Script para poblar la base de datos
Nota: Crear un archivo __init__.py vacío en cada carpeta para que Python las reconozca como paquetes.
```
Fase 3: Capa de Infraestructura y Datos
El objetivo es aislar la base de datos para que, si cambiamos de motor (ej. de SQLite a PostgreSQL), no afecte al resto del sistema.

Separación de Conexión y Esquema (SRP):

base_datos.py: Solo tiene la función obtener_conexion().

esquema_db.py: Solo tiene la función crear_tablas() con las consultas CREATE TABLE.

Patrón Repository (repositorio.py):

Crear una clase que englobe todas las consultas SQL (obtener_todos(), insertar(), etc.).

El resto del sistema NO debe escribir SQL directo; debe llamar a esta clase.

Lectores de Archivos (lector_excel.py):

Usar Pandas para leer los datos y convertir los nulos (NaN) a None antes de pasarlos a la base de datos.

Fase 4: Capa de Dominio y Aplicación
El objetivo es que la lógica clínica o de negocio esté protegida y no dependa de Flask ni de SQLite.

Modelos (dominio/modelos.py): Usar @dataclass para crear las entidades (ej. Usuario, Producto, Resumen). No usar herencia innecesaria; preferir composición.

Reglas (dominio/reglas.py): Poner las fórmulas, validaciones y cálculos matemáticos puros. Estas funciones deben recibir datos y devolver datos, sin imprimir en pantalla ni guardar en base de datos.

Servicios (aplicacion/servicio_xxx.py): Crear clases que orquesten los casos de uso. Por ejemplo, ServicioImportador llama al LectorExcel (infraestructura) y al Repositorio (infraestructura) para cumplir su objetivo.

Fase 5: Capa de Interfaces (Frontend y Controladores)
El objetivo es manejar la comunicación con el usuario final.

App Factory (app.py):

Crear una función crear_app() que configure Flask.

Cargar la secret_key de forma segura desde configuracion.py (que lee el .env).

Rutas / Blueprints (interfaces/rutas.py):

Las rutas solo reciben la petición (request) y devuelven una respuesta (render_template o jsonify).

NUNCA poner lógica de negocio ni consultas SQL dentro de las rutas.

Herencia de Plantillas (DRY):

Crear un base.html con el <head>, el CSS y el pie de página (Footer).

Usar {% extends "base.html" %} en las demás vistas (login.html, dashboard.html) para no repetir código HTML.

JavaScript (Fetch API):

Evitar recargar la página completa. Usar fetch para consumir APIs de Flask (ej. /api/buscar) y renderizar los resultados dinámicamente en el DOM.

Fase 6: Aseguramiento de Calidad (Calidad y Pruebas)
El objetivo es demostrar que el software funciona y que el código es legible.

Pruebas Unitarias (Pytest):

Crear pruebas para la capa de Dominio (cálculos y reglas).

Crear pruebas para los Servicios de Aplicación. Usar unittest.mock si es necesario para simular la base de datos y que las pruebas corran rápido.

Código Limpio (Flake8):

Ejecutar flake8 . --exclude=venv,datos,__pycache__ antes de cada commit.

Regla de oro: Ningún commit se sube si Flake8 arroja errores.

Asegurarse de que los nombres de variables y funciones sean descriptivos (en español o inglés, pero consistentes).

Fase 7: Control de Versiones (Git & GitHub)
El objetivo es mantener un historial limpio y trabajo colaborativo sin conflictos.

Inicialización: git init y primer commit (feat: estructura inicial del proyecto).

Ramas (Branching):

No trabajar siempre en main. Crear ramas por módulo: feature/modulo-login, feature/modulo-reportes.

Hacer Pull Request (PR) o merge cuando el módulo esté terminado y probado.

Mensajes de Commit Profesionales:

feat: (Nueva funcionalidad)

fix: (Corrección de errores)

docs: (Cambios en documentación/README)

refactor: (Mejora del código sin cambiar su comportamiento)

README.md Completo:

Debe incluir: Descripción, Arquitectura, Stack, Pasos de Instalación (clonar, venv, pip install), y cómo correr las pruebas.
´´´