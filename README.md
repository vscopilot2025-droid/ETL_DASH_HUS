# ETL DASH HUS - Sistema de Indicadores Hospitalarios

## 📋 Descripción General

Sistema ETL (Extract, Transform, Load) profesional para la gestión, transformación y carga de indicadores de desempeño del Hospital Universitario del Suroccidente (HUS) y Hospital General de Medellín. 

Este proyecto procesa **207 indicadores de calidad asistencial** recopilados entre **2010 y 2026**, transformándolos en una estructura de base de datos relacional optimizada para análisis, reportes y toma de decisiones estratégicas.

---

## 🎯 Objetivos del Proyecto

1. **Extraer**: Procesar datos brutos de indicadores desde archivos CSV con encoding latino-1
2. **Transformar**: Limpiar, validar y normalizar datos de múltiples períodos temporales
3. **Cargar**: Almacenar información estructurada en base de datos MySQL relacional
4. **Integrar**: Relacionar indicadores con procesos, objetivos estratégicos y áreas de gestión

---

## 📊 Datos Procesados

### Cantidad de Indicadores
- **Total**: 207 indicadores
- **Período**: 2010 - 2026 (16 años de histórico)
- **Periodicidad**: Mediciones mensuales (192 meses históricos por indicador)
- **Total de registros procesados**: ~39,744 registros de mediciones

### Categorías de Indicadores

#### 1. **Gestión de Urgencias** (Principal)
- Oportunidad en atención de demanda espontánea
- Tiempo promedio de respuesta a referencia
- Porcentaje de ingresos efectivos por referencia
- Pertinencia de remisiones aceptadas
- Reingresos en menos de 72 horas

#### 2. **Estructura de Datos**
Cada indicador contiene:
- **Código**: Identificación única
- **Nombre**: Descripción detallada
- **Clasificación**: Objetivo estratégico asociado
- **Tipo**: Dato o indicador
- **Unidad de medida**: Porcentaje, minutos, número
- **Periodicidad**: Mensual
- **Meta**: Valor objetivo establecido
- **Responsable**: Área responsable de medición

---

## 🗄️ Estructura de Base de Datos

### Tablas Generadas

#### 1. **indicadores**
Almacena la definición y metadatos de cada indicador
```sql
Campos: id_indicador, nombre_indicador, id_proceso, id_objetivo, 
         frecuencia, meta, unidad
Registros: 207
```

#### 2. **mediciones**
Almacena los valores históricos de mediciones mensuales
```sql
Campos: ID, Unidad_auditable, Código_proceso, Nombre_proceso, 
         Indicador, fecha, valor
Registros: 39,744 aproximadamente
```

#### 3. **procesos**
Define los procesos operativos del hospital
```sql
Campos: id_proceso, codigo_proceso, nombre_proceso
Ejemplo: "00PU01-V3" → "Atención al Paciente de Urgencias"
```

#### 4. **objetivos_estrategicos**
Alinea indicadores con objetivos institucionales
```sql
Campos: id_objetivo, codigo_objetivo, nombre_objetivo
Ejemplo: "Atención centrada en el usuario", "Oportuno", "Seguro"
```

#### 5. **areas**
Clasifica los indicadores por unidad auditora/área
```sql
Campos: id_area, nombre_area
Ejemplo: "Gestión Servicio de Urgencias"
```

---

## 🔄 Flujo ETL Detallado

### **Fase 1: EXTRACT (Extracción)**

1. **Lectura de datos brutos**
   - Archivo: `Indicadores.csv`
   - Codificación: latin-1
   - Separador: coma (,)
   - Libreación de espacios iniciales

2. **Carga inicial**
   ```python
   df = pd.read_csv(
       "Indicadores.csv",
       encoding="latin-1",
       sep=",",
       engine="python",
       skipinitialspace=True
   )
   ```

### **Fase 2: TRANSFORM (Transformación)**

1. **Limpieza de datos**
   - Eliminar columnas sin nombre (Unnamed)
   - Eliminar columnas de escala no necesarias
   - Remover filas completamente vacías
   - Eliminar registros "Sin medición"

2. **Normalización de estructura**
   - Identificar columnas base: ID, Unidad auditable, Código proceso, etc.
   - Detectar columnas de meses (Enero, Febrero, ... Diciembre)
   - Transformación de formato ancho a formato largo (MELT)

3. **Conversión de tipos y valores**
   - Convertir fechas: "Enero 2024" → "2024-01-01"
   - Convertir valores numéricos string a float
   - Validar integridad referencial

4. **Enriquecimiento de datos**
   - Asignación de IDs únicos a procesos
   - Asignación de IDs únicos a objetivos estratégicos
   - Asignación de IDs únicos a áreas
   - Validación de integridad referencial

### **Fase 3: LOAD (Carga)**

1. **Creación de tablas dimensionales**
   - Tabla de procesos (deduplicada)
   - Tabla de objetivos estratégicos (deduplicada)
   - Tabla de áreas (deduplicada)

2. **Carga de indicadores**
   - Insertar 207 registros únicos en tabla indicadores
   - Validar que no existan duplicados
   - Asegurar integridad de foreign keys

3. **Carga de mediciones**
   - Insertar ~39,744 registros de mediciones mensuales
   - Modo REPLACE para evitar duplicados
   - Validar fecha y valor para cada registro

---

## 💾 Configuración de Base de Datos

### Requisitos
- **Sistema**: MySQL 5.7 o superior
- **Puerto**: 3304 (configurable)
- **Usuario**: root
- **Contraseña**: 123456
- **Base de datos**: `hospital_kpi`

### Cadena de conexión
```python
engine = create_engine(
    "mysql+pymysql://root:123456@127.0.0.1:3304/hospital_kpi"
)
```

### Creación de base de datos
```sql
CREATE DATABASE IF NOT EXISTS hospital_kpi 
CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE hospital_kpi;

-- Las tablas se crean automáticamente durante la ejecución del ETL
```

---

## 🚀 Instalación y Uso

### Requisitos Python
```
pandas>=1.3.0
sqlalchemy>=1.4.0
pymysql>=1.0.0
openpyxl>=3.0.0
```

### Instalación de dependencias
```bash
pip install -r requirements.txt
```

### Ejecución del ETL
```bash
# Ejecutar el Jupyter Notebook
jupyter notebook python3.ipynb

# O ejecutar desde Python directamente
python -c "
import pandas as pd
from sqlalchemy import create_engine

# [Código del ETL aquí]
"
```

### Pasos de ejecución
1. Colocar archivo `Indicadores.csv` en la carpeta principal
2. Ejecutar celdas de forma secuencial en el notebook
3. Verificar conexión con la base de datos
4. Validar carga de datos con queries de prueba

---

## 📈 Resultados y Validación

### Métricas de Datos Cargados

| Concepto | Cantidad |
|----------|----------|
| **Indicadores únicos** | 207 |
| **Período temporal** | 2010-2026 (16 años) |
| **Mediciones mensuales** | ~39,744 registros |
| **Procesos identificados** | 1+ |
| **Objetivos estratégicos** | 4+ |
| **Áreas auditables** | 3+ |

### Validaciones Ejecutadas

✅ **Integridad referencial**
- Todas las mediciones vinculadas a indicadores existentes
- Todos los indicadores asociados a procesos y objetivos válidos

✅ **Calidad de datos**
- Eliminación de registros sin medición
- Conversión correcta de fechas
- Valores numéricos validados

✅ **Cobertura temporal**
- Período completo 2010-2026 cubierto
- Mediciones mensuales distribuidas

✅ **Deduplicación**
- Sin registros duplicados en tablas dimensionales
- Mediciones únicas por indicador-fecha

---

## 🔍 Ejemplos de Análisis con los Datos

### Consulta 1: Indicadores con peor desempeño
```sql
SELECT 
    nombre_indicador,
    AVG(valor) as promedio,
    MAX(valor) as maximo,
    MIN(valor) as minimo
FROM mediciones
GROUP BY Indicador
ORDER BY promedio ASC
LIMIT 10;
```

### Consulta 2: Evolución temporal de un indicador
```sql
SELECT 
    fecha,
    valor
FROM mediciones
WHERE Indicador = 'OPORTUNIDAD EN LA ATENCIÓN EN LA CONSULTA DE URGENCIAS'
ORDER BY fecha;
```

### Consulta 3: Indicadores por objetivo estratégico
```sql
SELECT 
    o.nombre_objetivo,
    COUNT(DISTINCT i.id_indicador) as total_indicadores,
    COUNT(DISTINCT m.fecha) as mediciones_registradas
FROM indicadores i
JOIN objetivos_estrategicos o ON i.id_objetivo = o.id_objetivo
LEFT JOIN mediciones m ON i.nombre_indicador = m.Indicador
GROUP BY o.nombre_objetivo;
```

---

## 🛠️ Arquitectura Técnica

### Diagrama de Flujo
```
┌─────────────────────┐
│ Indicadores.csv     │
│ (207 indicadores)   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  EXTRACT (Pandas)   │
│  - Lee CSV          │
│  - Codificación     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────────┐
│  TRANSFORM (Pandas/Python)      │
│  - Limpieza                     │
│  - Normalización                │
│  - Conversión de tipos          │
│  - Enriquecimiento              │
└──────────┬──────────────────────┘
           │
           ▼
┌──────────────────────────┐
│  LOAD (SQLAlchemy)       │
│  - Crear tablas          │
│  - Insertar registros    │
│  - Validar integridad    │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────────────────┐
│  Base de Datos MySQL (hospital_kpi)  │
│  ├─ indicadores (207)                │
│  ├─ mediciones (~39,744)             │
│  ├─ procesos                         │
│  ├─ objetivos_estrategicos           │
│  └─ areas                            │
└──────────────────────────────────────┘
```

### Tecnologías Utilizadas
- **Lenguaje**: Python 3.x
- **Procesamiento de datos**: Pandas
- **ORM**: SQLAlchemy
- **Base de datos**: MySQL 5.7+
- **Driver BD**: PyMySQL
- **Notebook**: Jupyter

---

## 📝 Notas Importantes

### Decisiones de Diseño

1. **Formato MELT**
   - Conversión de formato ancho a largo para mejor análisis temporal
   - Facilita agregaciones y comparaciones históricas

2. **Deduplicación**
   - Eliminación de indicadores duplicados por ID
   - Evita inconsistencias en integridad referencial

3. **Validaciones**
   - Filtrado de valores "Sin medición"
   - Conversión segura con `errors='coerce'` para manejo de excepciones

4. **Períodos históricos**
   - Cobertura completa 2010-2026
   - Granularidad mensual para detección de tendencias

---

## 🔐 Consideraciones de Seguridad

⚠️ **IMPORTANTE**
- Las credenciales de BD están hardcodeadas (desarrollo). Para producción usar variables de entorno
- Implementar autenticación adecuada
- Validar permisos de acceso a datos sensibles

### Mejoras recomendadas
```python
import os
from dotenv import load_dotenv

load_dotenv()

DB_USER = os.getenv('DB_USER')
DB_PASSWORD = os.getenv('DB_PASSWORD')
DB_HOST = os.getenv('DB_HOST', '127.0.0.1')
DB_PORT = os.getenv('DB_PORT', 3304)
DB_NAME = os.getenv('DB_NAME', 'hospital_kpi')

connection_string = f"mysql+pymysql://{DB_USER}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_NAME}"
```

---

## 📞 Soporte y Contacto

**Proyecto**: ETL DASH HUS
**Versión**: 1.0
**Última actualización**: Mayo 2026

Para consultas técnicas o reportar problemas, contactar al equipo de desarrollo.

---

## 📄 Licencia

Este proyecto es propiedad del Hospital Universitario del Suroccidente (HUS) y Hospital General de Medellín. Uso interno solamente.

---

## ✅ Checklist de Validación

- [x] 207 indicadores extraídos
- [x] Datos limpios y normalizados
- [x] ~39,744 mediciones cargadas
- [x] Integridad referencial validada
- [x] Tablas dimensionales creadas
- [x] Base de datos funcional
- [x] Documentación completa

---

**Generado automáticamente - Mayo 2026**
