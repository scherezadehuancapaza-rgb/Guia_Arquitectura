# Plantillas Base para Proyectos

Esta carpeta contiene los archivos de configuración base que deben copiar a la raíz de todo proyecto nuevo.

## ¿Cómo usarlas?
1. Copia el archivo `.gitignore` a la raíz de tu proyecto.
2. Copia el archivo `.flake8` a la raíz de tu proyecto.
3. Copia el archivo `requirements.txt` a la raíz de tu proyecto.
4. Renombra o crea un archivo `.env` basándote en las variables que necesites (NUNCA subas el `.env` a GitHub).
5. Ejecuta `pip install -r requirements.txt` para instalar el entorno.
(Nota: No fijamos versiones exactas en requirements  para que, cuando empiecen un proyecto nuevo, pip instale las versiones más recientes y estables, evitando el error de compilación que tuvimos con pandas).