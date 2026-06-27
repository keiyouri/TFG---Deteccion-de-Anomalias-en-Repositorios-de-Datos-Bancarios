# Detección de anomalías mediante IA en repositorios de datos bancarios

**Trabajo de Fin de Grado** — Grado en Informática (GIF)  
Universidad Alfonso X el Sabio · Junio 2026  
Autor: Mario Serrano Lorenzo  
Tutor: Jose Manuel Corpás

---

## Descripción

Este proyecto aborda el diseño, construcción y evaluación de un sistema de detección de anomalías basado en inteligencia artificial, aplicado a un repositorio de datos de referencia representativo del dominio de la banca privada y de inversión.

Se construyó un repositorio sintético de cinco tablas interrelacionadas (contrapartidas, clientes, instrumentos, contratos y transacciones), combinando datos públicos del [Global LEI Index (GLEIF)](https://www.gleif.org) con datasets sintéticos de KYC y riesgo transaccional. Sobre este repositorio se inyectaron 13 tipos de anomalías realistas, documentadas en la literatura regulatoria (BCBS 239, FATF).

El núcleo experimental consiste en un análisis comparativo entre **Isolation Forest** (no supervisado, en tres configuraciones iterativas) y **XGBoost con SMOTE** (supervisado), evaluando sus fortalezas, limitaciones y complementariedad.

## Estructura del repositorio

```
├── README.md
├── notebook.ipynb                     # Jupyter Notebook con todo el pipeline
├── data/
│   ├── counterparty.csv               # 100.000 registros — GLEIF Golden Copy (filtrado)
│   ├── client.csv                     # 2.000 registros — Dataset KYC sintético
│   ├── instrument.csv                 # 500 registros — Generado sintéticamente
│   ├── contract.csv                   # 5.209 registros — Generado sintéticamente
│   ├── transaction.csv                # 50.000 registros — Dataset KYC sintético
│   └── model_dataset.csv              # Dataset unificado con 38 features (listo para modelado)
└── results/
    ├── isolation_forest_results.png           # Iteración 1: modelo base (con leakage)
    ├── isolation_forest_optimized.png         # Iteración 2: sin leakage
    ├── isolation_forest_semisupervised.png    # Iteración 3: semi-supervisado
    └── comparativa_final.png                  # IF vs XGBoost+SMOTE
```

> **Nota:** Si la estructura de tu repositorio difiere, ajusta las rutas de los ficheros en el notebook.

## Modelo de datos

```
counterparty (100k) ←── LEI ──→ contract (5.2k) ──→ instrument (500)
                                    ↕ client_id
                               client (2k) ←── client_id ──→ transaction (50k)
```

Cinco tablas con integridad referencial completa. Cada transacción está vinculada a un cliente, un contrato y, a través de este, a una contrapartida y un instrumento financiero.

## Anomalías inyectadas

| Tabla | Anomalías | % | Tipos |
|---|---|---|---|
| counterparty | 1.996 | 2,0% | Inconsistencia geográfica, entidad activa expirada, duplicados, nombre legal ausente |
| client | 200 | 10,0% | Perfil de riesgo contradictorio, opacidad sospechosa, desajuste sector-riesgo |
| instrument | 40 | 8,0% | Nocional extremo, calificación crediticia incoherente |
| contract | 260 | 5,0% | Fecha de inicio futura, contrato terminado con instrumento activo |
| transaction | 1.500 | 3,0% | Importe extremo, estructuración (smurfing), evasión de sanciones |

La etiqueta final a nivel de cliente requiere evidencia de anomalía en **≥2 capas** del repositorio, resultando en 59 clientes anómalos (2,9%).

## Resultados

| Métrica | IF semi-supervisado | XGBoost+SMOTE (CV) |
|---|---|---|
| ROC-AUC | 0,7713 | 0,8132 |
| PR-AUC | 0,1400 | 0,1276 |
| Precision | 0,1556 | 0,1579 |
| Recall | 0,2373 | 0,3559 |
| F1-score | 0,1879 | 0,2188 |
| Supervisado | No | Sí |
| Requiere etiquetas | No | Sí |

**Hallazgo principal:** la ganancia de rendimiento del modelo supervisado sobre el no supervisado es marginal, lo que sugiere que Isolation Forest es una alternativa viable y pragmática para entornos bancarios donde no se dispone de datos etiquetados.

## Tecnologías

- **Entorno:** Google Colaboratory (GPU NVIDIA Tesla T4, 12,7 GB RAM)
- **Lenguaje:** Python 3.12
- **Datos:** pandas, NumPy
- **Modelado:** scikit-learn (Isolation Forest, StandardScaler, métricas), XGBoost, imbalanced-learn (SMOTE)
- **Visualización:** Matplotlib
- **Almacenamiento:** Google Drive

## Instalación y ejecución

### Opción 1: Google Colab (recomendada)

1. Sube el notebook y la carpeta `data/` a tu Google Drive.
2. Abre el notebook en Google Colab.
3. Ajusta la variable `BASE` a la ruta de tu carpeta en Drive.
4. Ejecuta las celdas en orden.

### Opción 2: Entorno local

```bash
# Clonar el repositorio
git clone https://github.com/tu-usuario/tu-repo.git
cd tu-repo

# Instalar dependencias
pip install pandas numpy scikit-learn xgboost imbalanced-learn matplotlib

# Abrir el notebook
jupyter notebook notebook.ipynb
```

## Fuentes de datos

| Fuente | Uso | Enlace |
|---|---|---|
| GLEIF Golden Copy | Tabla counterparty | [gleif.org](https://www.gleif.org/en/lei-data/gleif-golden-copy/download-the-golden-copy) |
| Synthetic KYC and Transaction Risk Dataset | Tablas client y transaction | [Kaggle](https://www.kaggle.com/datasets) |

## Referencias principales

- Liu, F.T., Ting, K.M., Zhou, Z.H. (2008). *Isolation Forest.* IEEE ICDM.
- Chen, T., Guestrin, C. (2016). *XGBoost: A Scalable Tree Boosting System.* ACM SIGKDD.
- Chawla, N.V. et al. (2002). *SMOTE: Synthetic Minority Over-sampling Technique.* JAIR.
- Basel Committee on Banking Supervision (2013). *BCBS 239: Principles for Effective Risk Data Aggregation.*
- FATF/OECD (2023). *International Standards on Combating Money Laundering.*

## Licencia

Este proyecto se desarrolló como Trabajo de Fin de Grado con fines académicos.
