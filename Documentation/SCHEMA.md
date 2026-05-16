# ESQUEMA DE BASE DE DATOS - ETL DASH HUS

Documentación técnica del esquema relacional para la gestión de 207 indicadores de desempeño hospitalario.

## 📊 Diagrama Entidad-Relación

```
┌──────────────────────┐
│   indicadores        │
├──────────────────────┤
│ PK id_indicador      │
│ nombre_indicador     │
│ FK id_proceso        │
│ FK id_objetivo       │
│ frecuencia           │
│ meta                 │
│ unidad_medida        │
└──────────────────────┘
          │ 1
          │
          ├─────┐
          │     │ 1
          │   ┌─────────────────┐
          │   │   mediciones    │
          │   ├─────────────────┤
          │   │ PK id_medicion  │
          │   │ FK id_indicador │
          │   │ fecha           │
          │   │ valor           │
          │   └─────────────────┘
          │
          │ N
┌─────────────────────────┐
│  procesos               │
├─────────────────────────┤
│ PK id_proceso           │
│ codigo_proceso          │
│ nombre_proceso          │
└─────────────────────────┘

┌──────────────────────────────┐
│  objetivos_estrategicos      │
├──────────────────────────────┤
│ PK id_objetivo               │
│ codigo_objetivo              │
│ nombre_objetivo              │
└──────────────────────────────┘

┌──────────────────┐
│   areas          │
├──────────────────┤
│ PK id_area       │
│ nombre_area      │
└──────────────────┘
```

---

## 📋 Definición de Tablas

### 1. TABLA: indicadores

**Descripción**: Almacena la definición y metadatos de los 207 indicadores de desempeño.

```sql
CREATE TABLE indicadores (
    id_indicador INT AUTO_INCREMENT PRIMARY KEY COMMENT 'ID único del indicador',
    nombre_indicador VARCHAR(500) NOT NULL UNIQUE COMMENT 'Nombre descriptivo del indicador (ej: OPORTUNIDAD EN LA ATENCIÓN)',
    id_proceso INT COMMENT 'Foreign key a procesos',
    id_objetivo INT COMMENT 'Foreign key a objetivos_estrategicos',
    clasificacion VARCHAR(255) COMMENT 'Clasificación del indicador (ej: Estructura, Proceso, Resultado)',
    tipo_dato VARCHAR(100) COMMENT 'Tipo de dato: Porcentaje, Minutos, Número, etc.',
    frecuencia VARCHAR(50) DEFAULT 'Mensual' COMMENT 'Periodicidad de medición',
    meta DECIMAL(10, 2) COMMENT 'Valor objetivo/meta del indicador',
    unidad_medida VARCHAR(100) COMMENT 'Unidad de medida (%, minutos, personas, etc.)',
    responsable VARCHAR(255) COMMENT 'Área responsable de reportar el indicador',
    formula TEXT COMMENT 'Fórmula de cálculo del indicador',
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (id_proceso) REFERENCES procesos(id_proceso) ON DELETE SET NULL,
    FOREIGN KEY (id_objetivo) REFERENCES objetivos_estrategicos(id_objetivo) ON DELETE SET NULL,
    
    INDEX idx_nombre_indicador (nombre_indicador),
    INDEX idx_id_proceso (id_proceso),
    INDEX idx_id_objetivo (id_objetivo),
    INDEX idx_frecuencia (frecuencia)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='207 indicadores de desempeño hospitalario';
```

**Registros**: 207 (uno por cada indicador)

**Ejemplo de datos**:
```
id | nombre_indicador                                          | id_proceso | meta   | unidad_medida | frecuencia
---|------------------------------------------------------------|-----------|---------|-----------|-----------
1  | OPORTUNIDAD EN ATENCIÓN CONSULTA URGENCIAS                | 1         | 95.00  | %           | Mensual
2  | TIEMPO PROMEDIO RESPUESTA A REFERENCIA                   | 2         | 48.00  | horas       | Mensual
3  | PORCENTAJE DE PACIENTES QUE REINGRESAN < 72 HORAS       | 1         | 5.00   | %           | Mensual
```

---

### 2. TABLA: mediciones

**Descripción**: Almacena los valores históricos mensuales de cada indicador (2010-2026).

```sql
CREATE TABLE mediciones (
    id_medicion INT AUTO_INCREMENT PRIMARY KEY COMMENT 'ID único de la medición',
    id_indicador INT NOT NULL COMMENT 'Foreign key a indicadores',
    id_unidad_auditable INT COMMENT 'ID de la unidad auditora',
    id_proceso INT COMMENT 'ID del proceso',
    fecha DATE NOT NULL COMMENT 'Fecha de la medición (primer día del mes)',
    valor DECIMAL(10, 2) COMMENT 'Valor numérico de la medición',
    estado VARCHAR(50) COMMENT 'Estado: Medida, Sin medición, Pendiente',
    semaforo TINYINT COMMENT 'Indicador semáforo: 1=Rojo, 2=Amarillo, 3=Verde',
    observaciones TEXT COMMENT 'Notas o aclaraciones sobre la medición',
    fecha_ingreso TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (id_indicador) REFERENCES indicadores(id_indicador) ON DELETE CASCADE,
    FOREIGN KEY (id_proceso) REFERENCES procesos(id_proceso) ON DELETE SET NULL,
    
    UNIQUE KEY unique_medicion (id_indicador, id_unidad_auditable, fecha),
    INDEX idx_fecha (fecha),
    INDEX idx_id_indicador (id_indicador),
    INDEX idx_semaforo (semaforo),
    INDEX idx_estado (estado),
    INDEX idx_fecha_rango (fecha) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='~39,744 mediciones mensuales (207 indicadores × 192 meses)';
```

**Registros**: ~39,744 (207 indicadores × 192 meses históricos = 2010-2026)

**Ejemplo de datos**:
```
id_medicion | id_indicador | fecha      | valor | semaforo | estado
------------|-------------|------------|-------|----------|----------
1           | 1           | 2010-01-01 | 92.50 | 2        | Medida
2           | 1           | 2010-02-01 | 93.25 | 3        | Medida
3           | 1           | 2010-03-01 | NULL  | NULL     | Sin medición
4           | 2           | 2010-01-01 | 45.30 | 3        | Medida
```

**Cobertura temporal**:
- Inicio: Enero 2010
- Fin: Marzo 2026 (datos actuales)
- Granularidad: Mensual
- Meses totales: 192

---

### 3. TABLA: procesos

**Descripción**: Define los procesos operativos del hospital relacionados con los indicadores.

```sql
CREATE TABLE procesos (
    id_proceso INT AUTO_INCREMENT PRIMARY KEY COMMENT 'ID único del proceso',
    codigo_proceso VARCHAR(50) UNIQUE NOT NULL COMMENT 'Código único (ej: 00PU01-V3)',
    nombre_proceso VARCHAR(255) NOT NULL COMMENT 'Descripción completa del proceso',
    descripcion TEXT COMMENT 'Información detallada del proceso',
    responsable_proceso VARCHAR(255) COMMENT 'Encargado del proceso',
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_codigo_proceso (codigo_proceso),
    INDEX idx_nombre_proceso (nombre_proceso)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='Procesos operativos asociados a indicadores';
```

**Registros**: ~15+ procesos (deducido de los 207 indicadores)

**Ejemplo de datos**:
```
id_proceso | codigo_proceso | nombre_proceso
-----------|----------------|---------------------------------------------
1          | 00PU01-V3      | Atención al Paciente de Urgencias
2          | 00AD01-V2      | Admisión de Pacientes
3          | 00QU01-V3      | Quirófano y Atención Quirúrgica
4          | 00HU01-V1      | Hospitalización de Usuarios
```

---

### 4. TABLA: objetivos_estrategicos

**Descripción**: Define los objetivos estratégicos institucionales alineados con los indicadores.

```sql
CREATE TABLE objetivos_estrategicos (
    id_objetivo INT AUTO_INCREMENT PRIMARY KEY COMMENT 'ID único del objetivo',
    codigo_objetivo VARCHAR(50) UNIQUE NOT NULL COMMENT 'Código único del objetivo',
    nombre_objetivo VARCHAR(500) NOT NULL COMMENT 'Nombre del objetivo estratégico',
    descripcion TEXT COMMENT 'Descripción detallada del objetivo',
    tipo_objetivo VARCHAR(100) COMMENT 'Tipo: Institucional, Departamental, Operativo',
    vigencia_inicio DATE COMMENT 'Fecha de inicio de vigencia',
    vigencia_fin DATE COMMENT 'Fecha de fin de vigencia',
    
    INDEX idx_codigo_objetivo (codigo_objetivo),
    INDEX idx_nombre_objetivo (nombre_objetivo)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='Objetivos estratégicos del hospital';
```

**Registros**: ~4-6 objetivos principales

**Ejemplo de datos**:
```
id_objetivo | nombre_objetivo                          | tipo_objetivo
------------|------------------------------------------|---------------
1           | Atención Centrada en el Usuario         | Institucional
2           | Atención Oportuna y Eficiente           | Institucional
3           | Atención Segura y de Calidad            | Institucional
4           | Sostenibilidad Financiera               | Institucional
5           | Desarrollo del Talento Humano           | Institucional
```

---

### 5. TABLA: areas

**Descripción**: Clasifica los indicadores por unidades auditoras o áreas administrativas.

```sql
CREATE TABLE areas (
    id_area INT AUTO_INCREMENT PRIMARY KEY COMMENT 'ID único del área',
    codigo_area VARCHAR(50) UNIQUE COMMENT 'Código del área',
    nombre_area VARCHAR(255) NOT NULL UNIQUE COMMENT 'Nombre del área (ej: Gestión Servicio de Urgencias)',
    descripcion TEXT COMMENT 'Descripción del área',
    responsable_area VARCHAR(255) COMMENT 'Director o responsable del área',
    
    INDEX idx_nombre_area (nombre_area)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='Áreas o unidades auditoras del hospital';
```

**Registros**: ~8-12 áreas principales

**Ejemplo de datos**:
```
id_area | nombre_area
--------|---------------------------------------------
1       | Gestión Servicio de Urgencias
2       | Gestión Quirófano
3       | Gestión Hospitalización
4       | Gestión Procedimientos Diagnósticos
5       | Gestión Consulta Externa
```

---

## 🔑 Relaciones y Constraints

### Foreign Keys (Claves Foráneas)

```sql
-- indicadores → procesos
ALTER TABLE indicadores ADD FOREIGN KEY (id_proceso) 
REFERENCES procesos(id_proceso) ON DELETE SET NULL;

-- indicadores → objetivos_estrategicos
ALTER TABLE indicadores ADD FOREIGN KEY (id_objetivo) 
REFERENCES objetivos_estrategicos(id_objetivo) ON DELETE SET NULL;

-- mediciones → indicadores
ALTER TABLE mediciones ADD FOREIGN KEY (id_indicador) 
REFERENCES indicadores(id_indicador) ON DELETE CASCADE;

-- mediciones → procesos
ALTER TABLE mediciones ADD FOREIGN KEY (id_proceso) 
REFERENCES procesos(id_proceso) ON DELETE SET NULL;
```

### Índices para Optimización

```sql
-- Indicadores
CREATE INDEX idx_indicadores_proceso ON indicadores(id_proceso);
CREATE INDEX idx_indicadores_objetivo ON indicadores(id_objetivo);
CREATE INDEX idx_indicadores_frecuencia ON indicadores(frecuencia);

-- Mediciones
CREATE INDEX idx_mediciones_indicador ON mediciones(id_indicador);
CREATE INDEX idx_mediciones_fecha ON mediciones(fecha);
CREATE INDEX idx_mediciones_semaforo ON mediciones(semaforo);
CREATE INDEX idx_mediciones_fecha_indicador ON mediciones(fecha, id_indicador);

-- Procesos
CREATE INDEX idx_procesos_codigo ON procesos(codigo_proceso);

-- Objetivos
CREATE INDEX idx_objetivos_codigo ON objetivos_estrategicos(codigo_objetivo);

-- Áreas
CREATE INDEX idx_areas_nombre ON areas(nombre_area);
```

---

## 📈 Vistas Útiles para Análisis

### Vista 1: Desempeño por Indicador

```sql
CREATE VIEW v_desempenio_indicadores AS
SELECT 
    i.id_indicador,
    i.nombre_indicador,
    i.meta,
    i.unidad_medida,
    p.nombre_proceso,
    COUNT(m.id_medicion) as total_mediciones,
    AVG(m.valor) as promedio,
    MAX(m.valor) as maximo,
    MIN(m.valor) as minimo,
    STDDEV(m.valor) as desviacion_estandar,
    MAX(m.fecha) as ultima_medicion
FROM indicadores i
LEFT JOIN procesos p ON i.id_proceso = p.id_proceso
LEFT JOIN mediciones m ON i.id_indicador = m.id_indicador
GROUP BY i.id_indicador, i.nombre_indicador, i.meta, i.unidad_medida, p.nombre_proceso;
```

### Vista 2: Tendencia Temporal

```sql
CREATE VIEW v_tendencia_temporal AS
SELECT 
    YEAR(m.fecha) as anio,
    MONTH(m.fecha) as mes,
    i.nombre_indicador,
    AVG(m.valor) as valor_promedio,
    COUNT(*) as mediciones_registradas
FROM mediciones m
JOIN indicadores i ON m.id_indicador = i.id_indicador
GROUP BY YEAR(m.fecha), MONTH(m.fecha), i.nombre_indicador
ORDER BY anio DESC, mes DESC;
```

### Vista 3: Indicadores por Objetivo

```sql
CREATE VIEW v_indicadores_por_objetivo AS
SELECT 
    o.id_objetivo,
    o.nombre_objetivo,
    COUNT(DISTINCT i.id_indicador) as total_indicadores,
    AVG(m.valor) as promedio_desempenio,
    COUNT(m.id_medicion) as total_mediciones
FROM objetivos_estrategicos o
LEFT JOIN indicadores i ON o.id_objetivo = i.id_objetivo
LEFT JOIN mediciones m ON i.id_indicador = m.id_indicador
GROUP BY o.id_objetivo, o.nombre_objetivo;
```

---

## 📊 Estadísticas de Datos

### Volumen de Datos Esperado

```
Tabla                    | Registros | Tamaño Aprox.
------------------------|-----------|----------------
indicadores             | 207       | ~50 KB
mediciones              | ~39,744   | ~5-10 MB
procesos                | 15        | ~5 KB
objetivos_estrategicos  | 6         | ~2 KB
areas                   | 10        | ~2 KB
```

### Cobertura Temporal

```
Período: Enero 2010 - Marzo 2026
Total de meses: 192
Indicadores activos: 207
Mediciones máximas teóricas: 39,744 (207 × 192)
Mediciones reales: ~35,000-39,000 (algunos meses sin datos)
```

---

## 🔍 Consultas Analíticas Clave

### Consulta 1: Top 10 Indicadores con Desempeño Deficiente

```sql
SELECT 
    i.nombre_indicador,
    i.meta,
    ROUND(AVG(m.valor), 2) as promedio_actual,
    ROUND((AVG(m.valor) / i.meta * 100), 2) as cumplimiento_pct,
    COUNT(m.id_medicion) as mediciones
FROM indicadores i
LEFT JOIN mediciones m ON i.id_indicador = m.id_indicador
WHERE m.valor IS NOT NULL
GROUP BY i.id_indicador
HAVING cumplimiento_pct < 80
ORDER BY cumplimiento_pct ASC
LIMIT 10;
```

### Consulta 2: Evolución Mensual de un Indicador

```sql
SELECT 
    DATE_FORMAT(m.fecha, '%Y-%m') as periodo,
    ROUND(AVG(m.valor), 2) as valor,
    ROUND(((AVG(m.valor) / i.meta) * 100), 2) as pct_meta
FROM mediciones m
JOIN indicadores i ON m.id_indicador = i.id_indicador
WHERE i.nombre_indicador = 'OPORTUNIDAD EN ATENCIÓN CONSULTA URGENCIAS'
GROUP BY DATE_FORMAT(m.fecha, '%Y-%m')
ORDER BY m.fecha DESC
LIMIT 24;
```

### Consulta 3: Indicadores con Mayor Variabilidad

```sql
SELECT 
    i.nombre_indicador,
    ROUND(AVG(m.valor), 2) as promedio,
    ROUND(STDDEV(m.valor), 2) as desviacion,
    ROUND((STDDEV(m.valor) / AVG(m.valor) * 100), 2) as coef_variacion,
    COUNT(m.id_medicion) as mediciones
FROM indicadores i
LEFT JOIN mediciones m ON i.id_indicador = m.id_indicador
WHERE m.valor IS NOT NULL
GROUP BY i.id_indicador
HAVING coef_variacion IS NOT NULL
ORDER BY coef_variacion DESC
LIMIT 15;
```

### Consulta 4: Comparativa Año Actual vs Año Anterior

```sql
SELECT 
    i.nombre_indicador,
    ROUND(AVG(CASE WHEN YEAR(m.fecha) = 2025 THEN m.valor END), 2) as valor_2025,
    ROUND(AVG(CASE WHEN YEAR(m.fecha) = 2024 THEN m.valor END), 2) as valor_2024,
    ROUND(AVG(CASE WHEN YEAR(m.fecha) = 2025 THEN m.valor END) - 
          AVG(CASE WHEN YEAR(m.fecha) = 2024 THEN m.valor END), 2) as diferencia
FROM indicadores i
LEFT JOIN mediciones m ON i.id_indicador = m.id_indicador
WHERE m.valor IS NOT NULL
GROUP BY i.id_indicador
ORDER BY ABS(diferencia) DESC
LIMIT 20;
```

---

## ⚙️ Procedimientos Almacenados Útiles

### Procedimiento 1: Validar Integridad de Datos

```sql
DELIMITER //

CREATE PROCEDURE sp_validar_integridad()
BEGIN
    SELECT '=== VALIDACIÓN DE INTEGRIDAD ===' as validacion;
    
    -- Indicadores huérfanos
    SELECT 'Indicadores sin proceso asignado' as tipo, COUNT(*) as cantidad
    FROM indicadores WHERE id_proceso IS NULL;
    
    -- Mediciones con indicador inválido
    SELECT 'Mediciones con indicador inválido' as tipo, COUNT(*) as cantidad
    FROM mediciones WHERE id_indicador NOT IN (SELECT id_indicador FROM indicadores);
    
    -- Mediciones duplicadas
    SELECT 'Mediciones duplicadas' as tipo, COUNT(*) - COUNT(DISTINCT CONCAT(id_indicador, fecha)) as cantidad
    FROM mediciones;
    
    -- Valores nulos en campos requeridos
    SELECT 'Indicadores sin nombre' as tipo, COUNT(*) as cantidad
    FROM indicadores WHERE nombre_indicador IS NULL;
    
END //

DELIMITER ;
```

---

## 🔐 Recomendaciones de Seguridad y Rendimiento

### Backups Regulares

```bash
# Backup diario a las 2 AM
mysqldump -u root -p123456 hospital_kpi > /backups/hospital_kpi_$(date +%Y%m%d).sql

# Restaurar desde backup
mysql -u root -p123456 hospital_kpi < /backups/hospital_kpi_20260315.sql
```

### Monitoreo de Desempeño

```sql
-- Ver tablas más grandes
SELECT 
    TABLE_NAME,
    ROUND(((data_length + index_length) / 1024 / 1024), 2) as size_mb
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'hospital_kpi'
ORDER BY size_mb DESC;
```

### Mantenimiento de Índices

```sql
-- Optimizar todas las tablas
OPTIMIZE TABLE indicadores;
OPTIMIZE TABLE mediciones;
OPTIMIZE TABLE procesos;
OPTIMIZE TABLE objetivos_estrategicos;
OPTIMIZE TABLE areas;

-- Analizar tabla para actualizar estadísticas
ANALYZE TABLE mediciones;
```

---

## 📝 Nota Final

Este esquema está optimizado para:
- ✅ Análisis histórico de 16 años de datos
- ✅ Consultas de tendencia temporal
- ✅ Agregaciones por proceso y objetivo
- ✅ Reportes de desempeño y cumplimiento
- ✅ Escalabilidad para nuevos indicadores

Versión: 1.0  
Fecha: Mayo 2026  
Responsable: ETL DASH HUS
