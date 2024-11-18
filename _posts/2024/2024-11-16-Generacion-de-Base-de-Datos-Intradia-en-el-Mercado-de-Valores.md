---
title: Generación de Base de Datos Intradía en el Mercado de Valores
date: 2024-11-16 12:00:00 -500
categories: [Python, SQL]
tags: [ds]
image:
  path: /assets/img/post/intra.webp
---
La presente investigación tiene como propósito generar una base de datos en el Mercado de Valores para operaciones intradía, que son aquellas operaciones en las que la compra y la venta de un determinado activo se realiza dentro de la misma sesión del mercado.

Veremos cómo utilizar una `API`, crear una función para definir nuestra base de datos en `SQLite`, guardar el archivo `.db` y verificar que el registro se haya realizado correctamente.

# Primer paso
Importar los paquetes requeridos:

**Pandas** Para trabajar y modificar los `dataframe`.

**SQLite3** Para crear nuestra base de datos `SQL`.

**Requests** Para acceder a la información de nuestra `API`.

# Segundo paso
Utilizaremos el sitio web de `alphavantage` para poder acceder a nuestra información, nos dirigimos a `https://www.alphavantage.co/support/#api-key` para obtener nuestra "Llave" única. Es totalmente gratis y no requiere un mail de verificación para generar el `API`.

# Tercer paso
Generamos una función para llamar nuestra acción deseada y crear un `dataframe` de la siguiente manera

```
def extraer_datos_intra(ticker, interval='1min'):
  url = f'https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol={ticker}&interval={interval}&outputsize=full&apikey="Utilizar_API_AQUI"'
  response = requests.get(url)
  data = response.json()

  time_series = data.get(f'Time Series ({interval})', {})
  if not time_series:
    print(f"No data para {ticker}")
    return None

  df = pd.DataFrame(time_series).T
  df.index.name = 'timestamp'
  df.columns = ['open', 'high', 'low', 'close', 'volume']
  df = df.sort_index()
  df.reset_index(inplace=True)
  df['ticker'] = ticker
  return df
```

La función funciona de la siguiente manera: `extraer_datos` obtiene la información del ticker (es decir, de la acción) y también un intervalo de tiempo específico, que en este caso está predefinido a 1 minuto.

A continuación, la variable `url` utiliza una función de Python para construir dinámicamente el `string`. El poder de las cadenas formateadas con `f'texto'` nos permite insertar variables dentro del texto usando `{}`. Por eso, vemos `{ticker}` y `{interval}` de esta manera.

La información se encuentra en formato `.json`. Para verificar si existen datos para el intervalo de tiempo del ticker correspondientes, definimos una condición. Si no se encuentran datos, se activa el `if not ...` y se ejecuta un `print` que nos indica que no hay datos disponibles para el ticker.

Las ultimas líneas del código, genera una transposición de una matriz, define el nombre del `index` a `timestamp`, guarda las columnas de `open, high, low, close, volume`, ordena el tiempo de `timestamp` de manera cronológica, reinicia el `index` para manipular los datos, y finalmente crea otra columna con el nombre de la acción. 

# Cuarto Paso
Generar nuestra base de datos con `SQLite`  de la siguiente manera:

```
conn = sqlite3.connect('intraday_stock.db')
cursor = conn.cursor()

cursor.execute('''
CREATE TABLE IF NOT EXISTS intra_valor (
        ticker TEXT,
        timestamp TEXT,
        open REAL,
        high REAL,
        low REAL,
        close REAL,
        volume INTEGER
        )
''')

conn.commit()
```

La primera línea genera una base de datos con el nombre de `intraday_stock`, luego creamos un `cursor` para poder generar nuestra tabla, con la sintaxis de `SQL`.  El `commit` es para guardar la tabla creada.

# Quinto Paso
Vamos definir diferente funciones para limpiar los datos, y poder escribir datos en nuestra base de datos. Para nuestra primera función será para no tener datos duplicados de la siguiente manera:

```
def obtener_ultimo_timestamp(ticker):
  query = """
  SELECT MAX(timestamp) FROM intra_valor WHERE ticker = ?
  """
  cursor.execute(query,(ticker,))
  result = cursor.fetchone()[0]
  return result
```

Vemos que utiliza el lenguaje `SQL` para llamar el valor máximo de tiempo `timestamp`, de esta manera igual podremos ir escribiendo nuevos valores deseados cada vez que corramos el código. La siguiente función es para guardar los datos de la siguiente manera:

```
def guardar_data_db(data,ultimo_timestamp):
  if data is not None:
    if ultimo_timestamp:
      data = data[data['timestamp'] > ultimo_timestamp]
    if not data.empty:
      data.to_sql('intra_valor', conn, if_exists='append',index=False)
      print(f"Data de {data['ticker'].iloc[0]} guardados con exito")
    else:
      print("No hay datos nuevos para guardar")
  else:
    print("No hay datos para guardar")
```

Las condiciones son las siguientes, si la data si existe, el ultimo `timestamp`  es comparado con el `ultimo_timestamp` del tiempo nuestra base de datos `SQL` y evita los duplicados, si la data no es vacía escribimos en el `SQL` los datos obtenidos de la `API`, y los agregamos en el caso de que si exista anteriormente. Si el `ultimo_timestamp` no es mayor que el de nuestro `timestamp` quiere decir que no hay datos nuevos para guardar y creamos un `print` que nos indique. Finalmente si la data no existe, llamar un `print` que nos diga que no hay datos para guardar.

# Sexto Paso
Definimos una ultima función que reúna toda las anteriores para hacer el trabajo mas simple y llamar esta cuando queramos con las acciones que deseamos. De la siguiente manera:

```
def main():
  tickers = ["MSFT","AAPL"]
  for ticker in tickers:
    ultimo_timestamp = obtener_ultimo_timestamp(ticker)
    data = extraer_datos_intra(ticker)
    guardar_data_db(data,ultimo_timestamp)

if __name__ == "__main__":
  main()
```

La función puede procesar múltiples tickers, lo que permite guardar el registro de diferentes acciones. De esta manera, podemos agregar fácilmente todos los tickers que deseemos.

# Séptimo Paso
Verificar que guardamos los datos, al correr el código completo tendremos un archivo `.db` con toda la información registrada, y querido lector para que vea que si funciona diríjase a la siguiente sitio web `https://sqliteviewer.app/` para comprobarlo usted mismo. En mi caso aquí esta el respaldo.

![intra1.webp](/assets/img/post/intra1.webp)
# Conclusión

Utilizamos una nueva herramienta para obtener datos, y con uno de los principios de las finanzas la cual es ***diversificar***, aprendimos como utilizar una `API` con su documentación publica, generamos una base de datos `SQL` dentro de Python para registrar intradía de las acciones que nosotros deseamos.

Para qué utilizar una base de datos SQL? Para expandir a niveles  industriales nuestra base de datos.

Por qué obtener los datos de la intradía? Para generar operaciones en las que la compra y la venta de un determinado activo se realiza dentro de la misma sesión del mercado.

Finalmente te agradezco querido lector por querer seguir expandir tus conocimientos, y recuerda que el conocimientos no tiene limites. 

