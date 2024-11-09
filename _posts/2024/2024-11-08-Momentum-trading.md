---
title: Momentum Trading
date: 2024-11-08 12:00:00 -500
categories: [Python, DS]
tags: [ds]
image:
  path: /assets/img/post/MT.webp
---
# Momentum Trading
El trading de impulso o momentum es una de las estrategias más sencillas de operar, en la que los traders compran y venden activos basándose en la fuerza de su movimiento reciente en el mercado. 

Para nuestro caso, vamos a ver el movimiento de los últimos 10 años de las acciones `NASDAQ-100`, con la intención de crear un ranking de sus valores por año y además creando un portafolio con las mejores acciones por año, y finalmente graficar con respecto al `NASDAQ-100` original.

### ¿Qué es el NASDAQ-100?

El índice Nasdaq 100 agrupa a las 100 empresas más grandes y con mayor actividad de cotización en la bolsa Nasdaq. Este índice abarca una amplia variedad de sectores, como tecnología, manufactura, atención médica, entre otros, pero excluye a las empresas del sector financiero, como bancos comerciales e instituciones de inversión.

## Codificación 

Para poder obtener los símbolos o `ticker` de las acciones del `NASDAQ-100` utilizamos una simulación de ser navegador web y de esta forma llamar a la API nos entregue el `json` con la información necesaria de la siguiente forma:

```
headers ={"user-agent": 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36'}

result = requests.get('https://api.nasdaq.com/api/quote/list-type/nasdaq100', headers=headers)

nasdaq_stocks = pd.DataFrame(result.json()['data']['data']['rows'])['symbol'].unique().tolist()

nasdaq_stocks
```
Para obtener los datos con respecto al tiempo hace 10 años atrás y poder obtener el porcentaje de `retorno_anual` de todas las acciones, en la función de `transform` se utilizo `252` porque un año financiero contiene en promedio 252 días hábiles

```
end_date = date.today()

start_date = end_date-pd.DateOffset(365*10)

prices_df = yf.download(tickers=nasdaq_stocks, start= start_date, end=end_date)

prices_df = prices_df.stack()

prices_df['retorno_anual'] = prices_df.groupby(level=1)['Adj Close'].transform(lambda x: x.pct_change(252))
```

Se agrupo con respecto a su `Ticker` , que vendría siendo el símbolo de cada acción, con una frecuencia de 1 año, así finalmente poder generar el ranking por cada año

```
dato_anual = test.groupby([pd.Grouper(freq='1YE'), 'Ticker'])['retorno_anual'].last().to_frame()

dato_anual['ranking']= dato_anual.groupby(level=0)['retorno_anual'].transform(lambda x: x.rank(ascending=False))
```

Entonces ya tenemos el ranking por año, pero queremos las mejores 5 acciones por año, fácilmente podemos generarlo del siguiente modo

```
datos_filtrados = dato_anual[dato_anual['ranking']<6]
```

Finalmente, se crea un diccionario con los tickets de los mejores 5.

![TP5.webp](/assets/img/post/TP5.webp)

Cabe destacar que para la información correspondiente para el año 2025 es hasta lo que lleva del año, en mi caso `2024-11-07`

![TP1.webp](/assets/img/post/TP1.webp)

Para la creación del portafolio se utiliza el promedio  porque el peso de cada acción es equivalente y graficamos lo que tenemos.

![TP2.webp](/assets/img/post/TP2.webp)

![MT1.webp](/assets/img/post/MT1.webp)

Para terminar se comparamos nuestro portafolio con el `NASDAQ-100` observando lo siguiente

![MT2.webp](/assets/img/post/MT2.webp)

### Conclusión

En este proyecto hemos realizado un ranking de los mejores valores de las acciones `NASDAQ-100`, creando un portafolio de inversión con las mejores 5 por por año y comparando sus retornos con un grafico de serie de tiempo. 

Aunque es cierto que al seleccionar solo las 5 mejores acciones estamos introduciendo cierto sesgo, el objetivo de este trabajo es identificar las acciones más destacadas de cada año para poder realizar un análisis de mercado posterior. La intención es comprender qué factores les dieron fuerza en su momento, qué estaba ocurriendo en ese periodo y cómo evoluciona el mercado a lo largo del tiempo. Por ejemplo, Netflix dominó todo el año 2015 al ocupar el primer puesto, pero luego perdió relevancia en los años siguientes. Entender las razones detrás de estos cambios es clave para el análisis.

Tener más indicadores, como el que vimos de medias móviles en los gráficos de velas, nos hará tener un perfil mas robusto de inversión y mayor seguridad al ver una acción. 