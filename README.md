# PROYECTO_MODULO5
##1.Titulo de un proyecto
Proyecto EDA con Pandas de Python con los datos proporcionados por el tutor del módulo. 

##2.Descripción del proyecto
Este proyecto realiza una serie de pasos del análisis de datos en Python para obtener unas conclusiones sobre las datos recibidos.

##3.Estructura del proyecto
El paso a paso seguido para la realización del proyecto es el siguiente:
###3.1. INSTALAR E IMPORTAR
Instalación numpy:
pip install numpy  No ha funcionado ese comando y se ha usado el siguiente:
PS C:\Users\sarav\Documents\Cursos\Data analytics The Power\7. Python for data\PROYECTO> py --version
Python 3.13.0
PS C:\Users\sarav\Documents\Cursos\Data analytics The Power\7. Python for data\PROYECTO> py -m pip --version
pip 24.2 from C:\Users\sarav\AppData\Local\Programs\Python\Python313\Lib\site-packages\pip (python 3.13)
PS C:\Users\sarav\Documents\Cursos\Data analytics The Power\7. Python for data\PROYECTO> py -m pip install numpy

Instalación Pandas
pip3 install pandas  No ha funcionado y al final he tenido que poner esto en la terminal:
py -m pip install pandas

Importar Numpy y Pandas:
import numpy as np
import pandas as pd 

###3.2. CARGA DE DATOS
3.2.1. Abrir un archivo CSV
df_banco = pd.read_csv("C:/Users/sarav/Documents/Cursos/Data analytics The Power/7. Python for data/PROYECTO/DatosProyecto/bank-additional.csv")

3.2.2. Abrir un archivo Excel
df_cliente = pd.read_excel("C:/Users/sarav/Documents/Cursos/Data analytics The Power/7. Python for data/PROYECTO/DatosProyecto/customer-details.xlsx")

###3.3. LIMPIEZA DE DATOS
3.3.1. Info general / Tipos de datos
df_banco.info()
df_banco.head()
se observan varias columnas que deberían ser numéricas y están como str
•	cons.price.idx
•	cons.conf.idx
•	euribor3m
•	nr.employed
También la columna fecha debería ser formato datetime
•	date

Se hace lo mismo con el archivo cliente.
df_cliente.info()
df_cliente.head()
En este archivo se observa que el formato de las columnas es correcto

3.3.2. Valores nulos
df_banco.isnull()
salen varios campos con true y la mayoría con False
df_banco.isnull().sum()
Salen bastantes campos nulos y habrá que ver como de relevantes son y qué hacer con ellos.

df_cliente.isnull()
sale todo con False
df_cliente.isnull().sum()
no tiene nulos
 

3.3.3. Para los valores NA y np.nan
df_banco.isna().sum()
 

df_cliente.isna().sum()
sigue dando 0 todo

En este caso ambos códigos dan el mismo resultado 

3.3.4. Filas duplicadas
df_banco.duplicated().sum()
df_cliente.duplicated().sum()
sale en ambas: np.int64(0)  no hay filas duplicadas

Se puede determinar que los datos de cliente no tienen limpieza que aplicarle, por lo que me focalizaré en limpiar banco.
Primero limpiaré individualmente cada archivo y luego cuando los junte vuelvo a limpiar conjuntamente.

###3.4. TRANSFORMACION DE DATOS
3.4.1. Eliminar columnas innecesarias
df_banco.drop('Unnamed: 0',axis = 1,inplace = True)
df_cliente.drop('Unnamed: 0',axis = 1,inplace = True)

3.4.2. Convertir columnas de str a float
df_banco['cons.price.idx'] = df_banco['cons.price.idx'].apply(
lambda x: float(str(x).replace(",", ".")) 
if pd.notnull(x) and str(x) != "False" 
else None
)
Compruebo:
df_banco.head()
df_banco['cons.price.idx'].dtype
resultado: dtype('float64')  todo OK

hago lo mismo con las siguientes columnas:
•	cons.conf.idx
•	euribor3m
•	nr.employed
uso el mismo código pero adaptándolo a cada nombre de columna distinto.

3.4.3. Convertir columnas de fechas de str a formato date
Primero veo como se ven los datos de esa columna de fecha:
df_banco['date'].head()
Tengo que transformar las palabras en español de los meses a números. Primero creo un diccionario con los meses.
meses = {
"enero": "01",
"febrero": "02",
"marzo": "03",
"abril": "04",
"mayo": "05",
"junio": "06",
"julio": "07",
"agosto": "08",
"septiembre": "09",
"octubre": "10",
"noviembre": "11",
"diciembre": "12"
}

Defino la función de cambio de meses, la aplico y cambio a formato fecha:
def cambiar_mes(fecha):
if pd.notnull(fecha):
fecha = str(fecha)
for mes, num in meses.items():
if mes in fecha:
fecha = fecha.replace(mes, num)
return fecha
return fecha

df_banco['date'] = df_banco['date'].apply(cambiar_mes)

df_banco['date'] = pd.to_datetime(df_banco['date'], format='%d-%m-%Y')

compruebo:
df_banco['date'].dtype
df_banco['date'].head()
resultado: Name: date, dtype: datetime64[us]  todo OK

3.4.4. Análisis de valores nulos
Quiero ver cuanto % del total representan los valores nulos:
(df_banco.isnull().sum() / len(df_banco)) * 100
 
Los datos menores a un 5% de repetición los voy a dejar igual porque no representan demasiada frecuencia:
-	Job
-	Marital
-	Education
-	Housing
-	loan
-	Cons.price.idx
-	Date
No voy a borrarlos.
En los que son campo texto voy a cambiar los nulos para que figuren como “unknown”:

df_banco['job'] = df_banco['job'].fillna('unknown')
df_banco['marital'] = df_banco['marital'].fillna('unknown')
df_banco['education'] = df_banco['education'].fillna('unknown')
df_banco['housing'] = df_banco['housing'].fillna('unknown')
df_banco['loan'] = df_banco['loan'].fillna('unknown')

En los que son campo numérico voy a cambiar los nulos por la media de ese campo: 
df_banco['cons.price.idx'] = df_banco['cons.price.idx'].fillna(df_banco['cons.price.idx'].mean())

En el campo de fecha voy a eliminar las filas:
df_banco = df_banco.dropna(subset=['date'])

Los datos medios como el campo age son importantes pero manejables.
La opción por la que opto aquí es sustituir los valores nulos por la media de la edad ya que así se mantiene la distribución al ser una variable continua.
df_banco['age'] = df_banco['age'].fillna(df_banco['age'].mean())

Los más altos (>20%): default y euribor3m hay que verlos detenidamente.
Default (20%)
Lo normal es que nulo signifique que se desconoce el dato. Opto por rellenarlo con ceros.
df_banco['default'] = df_banco['default'].fillna(0)

Euribor (21%)
Esta es muy importante ya que al ser una variable económica puedo distorsionar los datos si meto la media o perder datos si elimino las filas. Aunque opto por sustituir por la media porque creo que es más lógico.
df_banco['euribor3m'] = df_banco['euribor3m'].fillna(df_banco['euribor3m'].mean())

###3.5. MERGE/UNION DE ARCHIVOS
Se usa el método Merge porque hay una o más columnas comunes (la de ID).
Se realizó una unión tipo LEFT JOIN utilizando el dataset de campañas como base (df_banco), con el objetivo de conservar la totalidad de los registros y enriquecerlos con la información del cliente disponible (df_cliente).
Además, se tiene en cuenta que la columna de ID en cada dataframe se llama distinto.
df_final = df_banco.merge(df_cliente, how = 'left', left_on = 'id_', right_on = 'ID')

###3.6. REVISION DE DATOS AL CONJUNTO TOTAL
df_final.shape
df_final.head()
df_final.isnull().sum()
resultado:
 
Esto indica que hay muchas filas en df_banco que no tienen match en df_cliente y por tanto se rellenan con NaN.
Hay aproximadamente 22.000 registros sin Info de cliente.
Se decide dejar esos campos tal cual sin cambiarlos porque representar información real de que no hay datos de cliente y así no se introducen sesgos.

###3.7. ANALISIS DESCRIPTIVO
df_final['y'].value_counts()
La variable objetivo presenta un fuerte desbalanceo, con aproximadamente un 89% de respuestas negativas frente a un 11% de positivas, lo que indica que la contratación del producto es poco frecuente

3.6.1. Job vs contratación 
df_final.groupby('job')['y'].value_counts(normalize=True)
 
Se ve que quienes más contratan son los jubilados (25,25%), los estudiantes (31,36%) y los desempleados (14,44%). Podría estar relacionado con una mayor predisposición al ahorro o a la contratación de productos financieros seguros.
Los que menos contratan son los blue-collar (6,9%), los que trabajan en servicios (8%) y los emprendedores (8%).

3.6.2. Análisis edad vs contratación
df_final.groupby('y')['age'].describe()
 
La variable edad no muestra diferencias significativas entre los clientes que contratan y los que no, ya que tanto la media como la mediana son muy similares en ambos grupos. Esto sugiere que la edad, por sí sola, no es un factor determinante en la decisión de contratación.
Lo que si se puede concluir es que hay una mayor dispersión de edad entre las personas que si contratan que las que no, por los datos de desviación típica.

3.6.3. Análisis contact vs contratación
df_final.groupby('contact')['y'].value_counts(normalize=True)
 
El canal de contacto influye significativamente en la tasa de contratación. Las campañas realizadas mediante teléfono móvil (cellular) presentan una tasa de conversión (~14.7%) significativamente superior a las realizadas mediante teléfono fijo (~5.1%), lo que indica una mayor efectividad del canal móvil.
Esto podría deberse a una mayor disponibilidad y accesibilidad del cliente a través del teléfono móvil.

3.6.4. Duración llamada vs contratación
df_final.groupby('y')['duration'].mean()
 
La duración de la llamada es uno de los factores más determinantes en la contratación. Las llamadas que terminan en conversión tienen una duración media significativamente superior (~552 segundos) frente a aquellas que no (~220 segundos), lo que indica una fuerte relación entre el tiempo de interacción y el éxito de la campaña.

3.6.5. Poutcome vs contratación
df_final.groupby('poutcome')['y'].value_counts(normalize=True)
 
La categoría Poutcome es el resultado de la campaña anterior. Toma 3 valores diferentes: FAILURE si la campaña anterior falló, NONEXISTENT si no hubo campaña previa y SUCCESS si la campaña anterior fue exitosa.
Se ve claramente que si un cliente contrató antes, es muy probable que vuelva a contratar porque tiene un porcentaje de si muy alto (65,4%)
El resultado de la campaña anterior (poutcome) muestra una fuerte influencia en la probabilidad de contratación. Los clientes que previamente habían aceptado una oferta presentan una tasa de conversión muy elevada (~65%), mientras que aquellos sin historial previo (~9%) o con experiencias negativas (~14%) muestran tasas significativamente inferiores.

##4. Instalación y requisitos
Este proyecto se ha ejecutado mediante Visual Studio Code con las extensiones de Python y de Jupyter Notebook.

##5. Resultados y conclusiones
Los resultados se visualizan al ejecutar los códigos.

##6.Contribuciones
Toda sugerencia para mejorar el proyecto es bienvenida.

##7.Autores y Agradecimientos
Autora: Sara Viana Martínez (https://github.com/saravianam) 

