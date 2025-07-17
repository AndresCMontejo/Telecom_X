### **INTRODUCCIÓN**

El análisis de datos es una herramienta clave para las empresas, ya que permite comprender problemáticas a lo largo del tiempo mediante el uso de gráficos e indicadores. Estos análisis ayudan a detectar tendencias, comportamientos de los clientes, y lo más importante: conocer cuántos clientes permanecen o abandonan una empresa.

En este proyecto analizaremos cómo una empresa enfrenta una alta tasa de cancelaciones. A través de la extracción y análisis de datos, identificaremos los factores que inciden en la pérdida de clientes, enfocándonos especialmente en la columna **"churn"** o **"evasión"**, ya que esta variable representa una de las mayores preocupaciones en términos de retención. Entender el comportamiento de los clientes que cancelan permite a las empresas diseñar mejores estrategias para conservarlos.

El método para realizar un análisis efectivo incluye: comprender el archivo, extraer los datos, transformarlos en un DataFrame utilizando la biblioteca `pandas`, limpiar la información, ajustar los tipos de datos cuando sea necesario, y finalmente, crear visualizaciones que nos permitan interpretar mejor los patrones.

---

### **EXTRACCIÓN DE DATOS**

```python
import pandas as pd
import requests
import json

datos_clientes = requests.get('https://raw.githubusercontent.com/ingridcristh/challenge2-data-science-LATAM/refs/heads/main/TelecomX_Data.json')
resultado = json.loads(datos_clientes.text)
df = pd.DataFrame(resultado)
df.to_json('TelecomX_ClientesT', orient='records')
df.head()
```
### **LIMPIEZA Y TRATAMIENTO DE DATOS**

Para evitar errores al trabajar con columnas, realizamos una estandarización de los nombres: convertimos todo a minúsculas, reemplazamos los espacios y puntos por guiones bajos, y actualizamos el DataFrame:
```python
df_f.columns = df_f.columns.str.lower().str.replace('.', '_').str.replace(' ', '_')
df_f.info()
```
Además, sustituimos textos que pueden generar conflictos como "No phone service" o "No internet service" por un simple "No":
```python
df_f = df_f.replace('No phone service', 'No')
df_f = df_f.replace('No internet service', 'No')
df_f.sample(n=100)
```
También corregimos el tipo de dato en columnas mal identificadas, como 'account_charges_total', que debe ser float64:
```python
df_f['account_charges_total'] = pd.to_numeric(df_f['account_charges_total'], errors='coerce')
df_f.info()
```
### **VALIDANDO OTRAS INCONSISTENCIAS**
```python
Para detectar otros posibles valores atípicos o inconsistentes, revisamos las columnas de tipo object y describimos el DataFrame:
for col in df_f.select_dtypes(include='object').columns:
    print(col, df_f[col].unique())

df_f.describe(include='all')
```
### **ESTANDARIZACIÓN Y TRANSFORMACIÓN DE DATOS**
Sustituimos valores "Yes" y "No" por 1 y 0 para poder analizarlos fácilmente con gráficos, por ejemplo en la columna 'customer_seniorcitizen':
```python
df_f['customer_seniorcitizen'] = df_f['customer_seniorcitizen'].map({'No': '0', 'Yes': '1'}).astype(int)
df_f.info()
```
Creamos una nueva columna 'cuentas_diarias' dividiendo los cargos mensuales entre 30.44 (promedio de días por mes):
```python
df_f['cuentas_diarias'] = df_f['account_charges_monthly'] / 30.44
```
Reemplazamos nuevamente todos los "No" por 0 y "Yes" por 1:
```python
df_f = df_f.replace('No', '0')
df_f = df_f.replace('Yes', '1')
df_f.sample(n=100)
```
Para asegurar que los nuevos valores 0 y 1 estén correctamente tipados como enteros (int64), realizamos la siguiente conversión en múltiples columnas booleanas:
```python
import numpy as np
df_f = df_f.replace('', np.nan)

cols_to_convert = ['churn', 'customer_partner', 'customer_dependents', 'phone_phoneservice', 'phone_multiplelines',
                   'internet_onlinesecurity', 'internet_onlinebackup', 'internet_deviceprotection',
                   'internet_techsupport', 'internet_streamingtv', 'internet_streamingmovies',
                   'account_paperlessbilling']

df_f = df_f.dropna(subset=cols_to_convert)
df_f[cols_to_convert] = df_f[cols_to_convert].astype('int64')
df_f.sample(n=20)
```
### **ANÁLISIS DESCRIPTIVO**
Utilizamos la función .describe() para obtener un resumen estadístico de las variables numéricas del DataFrame, como media, mediana, mínimos y máximos.

### **DISTRIBUCIÓN DE EVASIÓN**
Creamos un gráfico de barras con seaborn y matplotlib para visualizar cuántos clientes cancelaron el servicio (churn = 1) y cuántos lo mantuvieron (churn = 0):
import seaborn as sns
import matplotlib.pyplot as plt
```python
fig, ax = plt.subplots(figsize=(4, 2))
sns.countplot(data=df_f, x='churn')

ax.set_title('Distribución de Clientes: Churn vs No Churn', loc='left', fontsize=16)
ax.set_xlabel('Churn (0 = No, 1 = Sí)', fontsize=14)
ax.set_ylabel('Cantidad de Clientes')
plt.show()
```

### **Recuento de evasión por variables categóricas**
En esta sección, el objetivo es analizar la evasión del servicio (churn) en función de variables categóricas. Se comienza examinando la relación entre el género de los clientes y su decisión de conservar o cancelar el servicio.

Primero, se crean dos listas (Genero_Female y Genero_Male) con los valores 'Female' y 'Male' respectivamente. A partir de ellas, se generan dos DataFrames filtrados que contienen únicamente a los clientes femeninos y masculinos:
```python
Genero_Female = ['Female']
Female = df_f.query('@Genero_Female in customer_gender')
Genero_Male = ['Male']
Male = df_f.query('@Genero_Male in customer_gender')
```
### **A continuación, se visualiza la distribución general de clientes por género:**
```python
# Total de clientes Femeninos y Masculinos
fig, ax = plt.subplots(figsize=(6, 4))
sns.countplot(data=df_f, x='customer_gender', hue='customer_gender', ax=ax, palette='pastel', legend=False)
ax.set_title('Distribución total de Clientes por Género', fontsize=16)
ax.set_xlabel('Género', fontsize=14)
ax.set_ylabel('Cantidad de Clientes', fontsize=14)
plt.show()
```
Luego, se analiza la evasión en función del género, mediante gráficos de barras aplicando sns.countplot a la columna churn:
```python
# Distribución de evasión de clientes femeninos
fig, ax = plt.subplots(figsize=(4,2))
sns.countplot(data=Female, x='churn', hue='churn', palette='colorblind', legend=False)
ax.set_title('Evasión de clientes Femeninos', loc='left', fontsize=16)
ax.set_xlabel('Churn (0 = No, 1 = Sí)', fontsize=14)
ax.set_ylabel('Cantidad de Clientes')
plt.show()
```
Posteriormente, se combinan ambos géneros en una misma visualización para obtener una perspectiva general de la evasión por género:
```python
# Gráfica general de Evasión por Género
fig, ax = plt.subplots(figsize=(6, 4))
sns.countplot(data=df_f, x='customer_gender', hue='churn', palette='pastel', ax=ax)
ax.set_title('Evasión por Género', fontsize=16)
ax.set_xlabel('Género', fontsize=14)
ax.set_ylabel('Cantidad de Clientes', fontsize=14)
ax.legend(title='Evasión (1 = Sí, 0 = No)')
plt.show()
```
Evasión por tipo de contrato
Se analiza el comportamiento de la evasión según el tipo de contrato (account_contract), categorizado como "month-to-month", "one year" y "two year":
```python
# Gráfica de Evasión por Tipo de Contrato
fig, ax = plt.subplots(figsize=(6, 4))
sns.countplot(data=df_f, x='account_contract', hue='churn', palette='pastel', ax=ax)
ax.set_title('Evasión por tipo de contrato', fontsize=16)
ax.set_xlabel('Tipo de contrato', fontsize=14)
ax.set_ylabel('Cantidad de Clientes', fontsize=14)
ax.legend(title='Evasión (1 = Sí, 0 = No)')
plt.show()
```
### **Evasión por método de pago**
Del mismo modo, se visualiza la evasión en función del método de pago (account_paymentmethod). Dado que existen múltiples categorías, se ajusta la rotación de etiquetas para mejorar la legibilidad y se añade una etiqueta con el valor numérico sobre cada barra:
```python
# Gráfica de Evasión por Método de Pago
fig, ax = plt.subplots(figsize=(10, 5))
sns.countplot(data=df_f, x='account_paymentmethod', hue='churn', palette='pastel', ax=ax)
ax.set_title('Evasión por Método de Pago', fontsize=16)
ax.set_xlabel('Método de Pago', fontsize=14)
ax.set_ylabel('Cantidad de Clientes', fontsize=14)
ax.legend(title='Evasión (1 = Sí, 0 = No)')
plt.xticks(rotation=30, ha='right')

# Agregar etiquetas numéricas
for p in ax.patches:
    height = p.get_height()
    if height > 0:
        ax.annotate(f'{int(height)}',
                    (p.get_x() + p.get_width() / 2., height),
                    ha='center', va='bottom', fontsize=10)
plt.tight_layout()
plt.show()
```
### **Conteo de evasión por variables numéricas**

Gasto total del cliente
Se utiliza un gráfico de caja (boxplot) para analizar el gasto total (account_charges_total) frente a la evasión:
```python
# Boxplot: Total Gastado vs Evasión
plt.figure(figsize=(6,4))
sns.boxplot(data=df_f, x='churn', hue='churn', y='account_charges_total', palette='pastel', legend=False)
plt.title('Distribución del Total Gastado según Evasión')
plt.xlabel('Evasión (1 = Sí, 0 = No)')
plt.ylabel('Total Gastado')
plt.show()
```
### **Tiempo de contrato**
Se aplica la misma lógica para visualizar la relación entre la evasión y el tiempo que el cliente ha estado con el servicio (customer_tenure):
```python
# Boxplot: Tiempo de Contrato vs Evasión
plt.figure(figsize=(6,4))
sns.boxplot(data=df_f, x='churn', hue='churn', y='customer_tenure', palette='pastel', legend=False)
plt.title('Distribución del Tiempo de Contrato según Evasión')
plt.xlabel('Evasión (1 = Sí, 0 = No)')
plt.ylabel('Tiempo de Contrato (meses)')
plt.show()
```
### **Gasto diario**
Se evalúa la relación entre el gasto diario de cada cliente y la cancelación del servicio:
```python
# Boxplot: Cuenta Diaria vs Evasión
plt.figure(figsize=(6, 4))
sns.boxplot(data=df_f, x='churn', hue='churn', y='cuentas_diarias', palette='pastel', legend=False)
plt.title('Relación entre Cuenta Diaria y Evasión')
plt.xlabel('Evasión (1 = Sí, 0 = No)')
plt.ylabel('Cuenta Diaria')
plt.show()
```
### **Total de servicios contratados**
Se crea una nueva columna llamada total_servicios, que representa la cantidad de servicios contratados por cliente. Esta variable se calcula sumando los valores binarios de múltiples columnas del DataFrame:
```python
# Relación entre número de servicios contratados y evasión
df_f['total_servicios'] = df_f[['phone_phoneservice', 'phone_multiplelines', 'internet_onlinesecurity', 'internet_onlinebackup',
                                'internet_deviceprotection', 'internet_techsupport', 'internet_streamingtv', 'internet_streamingmovies']].sum(axis=1)

plt.figure(figsize=(6, 4))
sns.boxplot(data=df_f, x='churn', hue='churn', y='total_servicios', palette='pastel', legend=False)
plt.title('Cantidad de Servicios vs Evasión')
plt.xlabel('Evasión (1 = Sí, 0 = No)')
plt.ylabel('Cantidad de Servicios Contratados')
plt.show()
```
### **Conclusiones**
Tras realizar un análisis detallado del DataFrame, incluyendo la limpieza, transformación y visualización de los datos, se identificaron patrones relevantes en el comportamiento de los clientes que cancelan el servicio frente a quienes lo conservan:

La proporción de clientes femeninos y masculinos es similar, y ambos grupos muestran tasas comparables de cancelación y retención.

El método de pago electronic check se asocia con una tasa significativamente más alta de cancelación.

Los clientes que cancelan el servicio suelen tener un gasto acumulado menor y menor tiempo de permanencia en comparación con aquellos que lo conservan.

Sin embargo, los clientes que se dan de baja tienden a tener un gasto diario más alto.

Los clientes que retienen el servicio usualmente tienen más servicios contratados.

Recomendaciones:

Implementar promociones o beneficios que reduzcan la percepción de alto costo diario, fomentando la contratación de paquetes con múltiples servicios.

Promover métodos de pago más estables o prácticos, destacando sus ventajas frente a opciones como electronic check.

Se observó que muchos clientes que usan electronic check no tienen contratados servicios clave como internet_deviceprotection (1,557 clientes) e internet_onlinebackup (1,550 clientes). Se sugiere diseñar campañas dirigidas a este segmento para promover estos servicios como parte fundamental de su seguridad digital.

Finalmente, incentivar la contratación a largo plazo, ya que los usuarios con contratos mensuales muestran mayor propensión a cancelar que aquellos con contratos de uno o dos años.
