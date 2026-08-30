mkdir C:\Kontalo
cd C:\Kontalo

python --version

python -m venv .venv
.venv\Scripts\Activate.ps1

pip install --upgrade pip
pip install Flask requests psycopg[binary] SQLAlchemy pandas numpy scikit-learn

# Crear el archivo principal:
# C:\Kontalo\app.py
# Pegá dentro de app.py el código completo de Kontalo.

python app.py