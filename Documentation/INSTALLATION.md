# GUÍA DE INSTALACIÓN Y USO - ETL DASH HUS

Guía paso a paso para configurar, ejecutar y utilizar el sistema ETL de indicadores hospitalarios.

---

## 🚀 Requisitos del Sistema

### Hardware Mínimo
- Procesador: Dual Core 2.0 GHz
- RAM: 4 GB (recomendado 8 GB)
- Almacenamiento: 2 GB disponible
- Conexión de red para MySQL remoto (opcional)

### Software Requerido

#### Windows / macOS / Linux
```
✓ Python 3.8 o superior
✓ MySQL 5.7 o superior (o MariaDB 10.5+)
✓ pip (gestor de paquetes Python)
✓ Git (control de versiones)
```

---

## 📦 Instalación de Dependencias

### Paso 1: Clonar el Repositorio

```bash
# Opción 1: Clonar desde GitHub
git clone https://github.com/vscopilot2025-droid/ETL_DASH_HUS.git
cd ETL_DASH_HUS

# Opción 2: Descargar como ZIP
# Descargar desde https://github.com/vscopilot2025-droid/ETL_DASH_HUS
# Extraer en carpeta local
```

### Paso 2: Crear Entorno Virtual (Recomendado)

#### En Windows (PowerShell)
```powershell
# Crear entorno virtual
python -m venv venv

# Activar entorno
.\venv\Scripts\Activate.ps1

# Si da error de ejecución:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
.\venv\Scripts\Activate.ps1
```

#### En macOS / Linux
```bash
# Crear entorno virtual
python3 -m venv venv

# Activar entorno
source venv/bin/activate
```

### Paso 3: Instalar Paquetes Python

```bash
# Asegurar que pip esté actualizado
pip install --upgrade pip

# Instalar dependencias
pip install -r requirements.txt

# Verificar instalación
pip list
```

**Paquetes instalados:**
```
pandas          # Procesamiento de datos
sqlalchemy      # ORM para bases de datos
pymysql         # Driver MySQL
jupyter         # Notebooks interactivos
openpyxl        # Lectura de Excel
python-dotenv   # Gestión de variables de entorno
```

### Paso 4: Configurar Variables de Entorno

Crear archivo `.env` en la raíz del proyecto:

```bash
# Base de Datos
DB_USER=root
DB_PASSWORD=123456
DB_HOST=127.0.0.1
DB_PORT=3304
DB_NAME=hospital_kpi

# Opcional
LOG_LEVEL=INFO
DEBUG=False
```

**Archivo `.env.example`:**
```
# Copiar este archivo a .env y actualizar valores
DB_USER=root
DB_PASSWORD=tu_contraseña
DB_HOST=localhost
DB_PORT=3306
DB_NAME=hospital_kpi
```

---

## 🗄️ Configuración de Base de Datos MySQL

### Paso 1: Descargar MySQL

#### Windows
1. Descargar desde: https://dev.mysql.com/downloads/mysql/
2. Ejecutar instalador
3. Seguir wizard de configuración
4. Puertos recomendados: 3304 (para no conflictar con otros servicios)

#### macOS
```bash
# Usando Homebrew
brew install mysql
brew services start mysql

# Configuración inicial
mysql_secure_installation
```

#### Linux (Ubuntu/Debian)
```bash
# Instalar MySQL
sudo apt-get update
sudo apt-get install mysql-server

# Iniciar servicio
sudo systemctl start mysql

# Configuración
sudo mysql_secure_installation
```

### Paso 2: Crear Base de Datos

```sql
-- Conectar a MySQL
mysql -u root -p

-- Crear base de datos
CREATE DATABASE IF NOT EXISTS hospital_kpi 
CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Crear usuario (opcional, si no se usa root)
CREATE USER 'etl_user'@'localhost' IDENTIFIED BY 'etl_password';
GRANT ALL PRIVILEGES ON hospital_kpi.* TO 'etl_user'@'localhost';
FLUSH PRIVILEGES;

-- Verificar
SHOW DATABASES;
USE hospital_kpi;
SHOW TABLES;
```

### Paso 3: Verificar Conexión

```bash
# Desde terminal
mysql -h 127.0.0.1 -P 3304 -u root -p123456 -e "SELECT 'Conexión OK' as status;"

# Desde Python
python -c "
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://root:123456@127.0.0.1:3304/hospital_kpi')
with engine.connect() as conn:
    print('✓ Conexión a BD exitosa')
"
```

---

## 📊 Ejecución del ETL

### Opción 1: Ejecutar Jupyter Notebook (Recomendado)

```bash
# Activar entorno virtual
# En Windows: .\venv\Scripts\Activate.ps1
# En Linux/macOS: source venv/bin/activate

# Iniciar Jupyter
jupyter notebook

# Se abrirá en http://localhost:8888
# Abrir archivo: python3.ipynb
```

#### Pasos en Jupyter:
1. **Instalar Kernel Jupyter** (si es necesario)
   ```bash
   python -m ipykernel install --user --name etl_dash --display-name "ETL DASH HUS"
   ```

2. **Seleccionar Kernel**: Kernel → Change kernel → ETL DASH HUS

3. **Ejecutar celdas secuencialmente:**
   - Cell 1-2: Imports y configuración
   - Cell 3-6: Lectura de CSV
   - Cell 7-24: Transformación de datos
   - Cell 25-30: Melt operation
   - Cell 31-50+: Carga a base de datos

4. **Validar resultados**:
   - Verificar que no haya errores
   - Revisar números de registros cargados

### Opción 2: Ejecutar desde Terminal

Crear archivo `run_etl.py`:

```python
import pandas as pd
from sqlalchemy import create_engine
import os
from dotenv import load_dotenv

# Cargar variables de entorno
load_dotenv()

DB_USER = os.getenv('DB_USER', 'root')
DB_PASSWORD = os.getenv('DB_PASSWORD', '123456')
DB_HOST = os.getenv('DB_HOST', '127.0.0.1')
DB_PORT = os.getenv('DB_PORT', '3304')
DB_NAME = os.getenv('DB_NAME', 'hospital_kpi')

# Crear motor de BD
connection_string = f"mysql+pymysql://{DB_USER}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_NAME}"
engine = create_engine(connection_string)

print("1. Leyendo CSV...")
df = pd.read_csv(
    "Indicadores.csv",
    encoding="latin-1",
    sep=",",
    engine="python",
    skipinitialspace=True
)
print(f"   ✓ {len(df)} filas leídas")

print("\n2. Transformando datos...")
# [Agregar lógica de transformación aquí]

print("\n3. Cargando a base de datos...")
# [Agregar lógica de carga aquí]

print("\n✓ ETL completado exitosamente")
```

Ejecutar:
```bash
python run_etl.py
```

### Opción 3: Ejecutar desde Línea de Comandos Interactivo

```python
python

import pandas as pd
from sqlalchemy import create_engine

# Leer CSV
df = pd.read_csv("Indicadores.csv", encoding="latin-1")

# Conectar a BD
engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3304/hospital_kpi")

# Cargar datos
df.to_sql('mediciones', con=engine, if_exists='replace', index=False)

print("Datos cargados exitosamente")
```

---

## ✅ Validación de Carga de Datos

### Verificación en Terminal

```bash
# Conectar a MySQL
mysql -u root -p123456 -D hospital_kpi

# Contar registros
SELECT 'Indicadores' as tabla, COUNT(*) as registros FROM indicadores
UNION ALL
SELECT 'Mediciones', COUNT(*) FROM mediciones
UNION ALL
SELECT 'Procesos', COUNT(*) FROM procesos
UNION ALL
SELECT 'Objetivos', COUNT(*) FROM objetivos_estrategicos
UNION ALL
SELECT 'Áreas', COUNT(*) FROM areas;
```

### Verificación desde Python

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3304/hospital_kpi")

# Contar registros
tables = ['indicadores', 'mediciones', 'procesos', 'objetivos_estrategicos', 'areas']

for table in tables:
    query = f"SELECT COUNT(*) as count FROM {table}"
    result = pd.read_sql(query, engine)
    print(f"{table}: {result.iloc[0]['count']} registros")
```

**Resultado esperado:**
```
indicadores: 207 registros
mediciones: 35000-39744 registros
procesos: 15+ registros
objetivos_estrategicos: 6+ registros
areas: 10+ registros
```

---

## 📊 Análisis de Datos

### Generar Reportes

#### Reporte 1: Top Indicadores Deficientes

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3304/hospital_kpi")

query = """
SELECT 
    i.nombre_indicador,
    i.meta,
    ROUND(AVG(m.valor), 2) as promedio,
    ROUND((AVG(m.valor) / i.meta * 100), 2) as cumplimiento_pct
FROM indicadores i
LEFT JOIN mediciones m ON i.id_indicador = m.id_indicador
WHERE m.valor IS NOT NULL AND i.meta IS NOT NULL
GROUP BY i.id_indicador
HAVING cumplimiento_pct < 80
ORDER BY cumplimiento_pct ASC
LIMIT 20;
"""

df_reportes = pd.read_sql(query, engine)
df_reportes.to_csv('reportes/top_deficiencias.csv', index=False)
print(df_reportes)
```

#### Reporte 2: Evolución Temporal

```python
query = """
SELECT 
    DATE_FORMAT(m.fecha, '%Y-%m') as periodo,
    ROUND(AVG(m.valor), 2) as valor_promedio
FROM mediciones m
WHERE m.id_indicador = 1
GROUP BY DATE_FORMAT(m.fecha, '%Y-%m')
ORDER BY m.fecha
"""

df_evolucion = pd.read_sql(query, engine)
print(df_evolucion)

# Gráfico de tendencia (requiere matplotlib)
import matplotlib.pyplot as plt
plt.figure(figsize=(12, 6))
plt.plot(df_evolucion['periodo'], df_evolucion['valor_promedio'], marker='o')
plt.xlabel('Período')
plt.ylabel('Valor Promedio')
plt.title('Evolución del Indicador')
plt.xticks(rotation=45)
plt.tight_layout()
plt.savefig('reportes/evolucion_temporal.png')
```

---

## 🛠️ Troubleshooting (Solución de Problemas)

### Error 1: "Connection refused"

**Causa**: MySQL no está corriendo

**Solución**:
```bash
# Windows
net start MySQL57
# o desde Services.msc buscar MySQL

# macOS
brew services start mysql

# Linux
sudo systemctl start mysql
```

### Error 2: "Access denied for user 'root'"

**Causa**: Credenciales incorrectas

**Solución**:
```bash
# Verificar usuario y contraseña
mysql -u root -p
# Ingresar contraseña

# Resetear contraseña (Windows)
mysqld --skip-grant-tables
# Luego en otra terminal:
mysql -u root
```

### Error 3: "No module named 'pandas'"

**Causa**: Dependencias no instaladas

**Solución**:
```bash
# Asegurarse que entorno virtual está activado
pip install -r requirements.txt
```

### Error 4: "Encoding issues" con caracteres especiales

**Causa**: Encoding del CSV no está configurado

**Solución**:
```python
# Asegurar encoding latin-1 o utf-8
df = pd.read_csv("Indicadores.csv", encoding="latin-1")
```

### Error 5: "File not found: Indicadores.csv"

**Causa**: El archivo CSV no está en la carpeta correcta

**Solución**:
```bash
# Verificar ubicación
ls -la Indicadores.csv

# O desde Python
import os
print(os.getcwd())  # Ver carpeta actual
```

---

## 📋 Tareas Recurrentes

### Actualización Mensual de Datos

```bash
# 1. Actualizar archivo CSV con nuevos datos
# 2. Ejecutar ETL
jupyter notebook python3.ipynb

# 3. Validar carga
python -c "
import pandas as pd
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://root:123456@127.0.0.1:3304/hospital_kpi')
result = pd.read_sql('SELECT MAX(fecha) FROM mediciones', engine)
print(f'Última medición cargada: {result.iloc[0][0]}')
"
```

### Backup de Base de Datos

```bash
# Backup completo
mysqldump -u root -p123456 hospital_kpi > backup_$(date +%Y%m%d_%H%M%S).sql

# Restaurar desde backup
mysql -u root -p123456 hospital_kpi < backup_20260315_143022.sql
```

### Limpiar Datos Antiguos

```sql
-- Eliminar mediciones anteriores a 2015 (si aplica)
DELETE FROM mediciones 
WHERE fecha < '2015-01-01';

-- Optimizar tabla
OPTIMIZE TABLE mediciones;
```

---

## 🔐 Consideraciones de Seguridad

### Para Desarrollo Local
```
✓ Usar credenciales débiles es aceptable (root:123456)
✓ Pero cambiar en producción
✓ Usar variables de entorno con .env
✓ Nunca commitear .env a Git
```

### Para Producción
```
✓ Cambiar contraseña de root
✓ Crear usuario específico con permisos limitados
✓ Habilitar SSL/TLS
✓ Hacer backups regulares
✓ Implementar auditoría de cambios
```

---

## 📞 Soporte

Para problemas con:
- **ETL**: Revisar logs del notebook
- **Base de Datos**: Ver archivo SCHEMA.md
- **Python**: Revisar documentación de pandas y SQLAlchemy

---

## ✨ Próximas Mejoras

- [ ] Dashboard de visualización (Power BI / Grafana)
- [ ] Alertas automáticas de bajo desempeño
- [ ] API REST para acceso a datos
- [ ] Sistema de caché para consultas frecuentes
- [ ] Versionamiento de datos históricos

---

**Guía de Instalación v1.0**  
**Última actualización: Mayo 2026**  
**Autor: ETL DASH HUS Team**
