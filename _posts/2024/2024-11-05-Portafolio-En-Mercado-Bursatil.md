---
title: Portafolio En Mercado Bursátil
date: 2024-11-05 12:00:00 -500
categories: [Python, DS]
tags: [ds]
image:
  path: /assets/img/post/PF.webp
---
![PF.webp](/assets/img/post/PF.webp)
# Portafolio En Mercado Bursátil

La presente investigación tiene como propósito diseñar y evaluar un modelo de portafolio de inversiones simplificado, empleando datos históricos desde 2018, con el fin de analizar el impacto de diferentes variables en el rendimiento y el riesgo de la cartera, y contribuir al desarrollo de estrategias de inversión más eficientes.

Este proyecto está enfocado como programación con fines educativos.

## Primer Paso
Importar los paquetes requeridos para trabajar:

**Yfinance** Para acceder a la información de las acciones que deseamos en un dataframe.
**Pandas** Para trabajar y modificar los dataframe.
**maplotlib.pyplot** Para generar los gráficos.

## Segundo Paso
Crear una lista con los nombres cuales que deseamos invertir, en mi caso son `NVIDIA` y `Bitcoin` , y al mismo tiempo, creamos un diccionario con los nombres de las acciones y su respectivo proporción de inversión:

```
miembros = ['NVDA','BTC-USD']
peso = {'NVDA':50, 'BTC-USD':50}
```

## Tercer Paso
Cargamos la información de la lista de miembros, primero para comprender y validar que nuestro código este trabajando bien nos vamos asegurar con el primer valor, es decir, con las acciones de `NVIDIA` del siguiente modo:

```
basedatos = yf.Ticker(miembros[0]).history(period='max').reset_index()[['Date','Open']]
basedatos['Date'] = basedatos['Date'].dt.strftime('%Y/%m/%d')
basedatos = basedatos.rename(columns = {'Open':miembros[0]})
```

Cargamos con la función `yf.Ticker`  al igual que se hace un llamado histórico con `history(period='max')` y reiniciamos sus índex porque existe un problema con la fecha de no hacerlo. Luego ajustamos la fecha a nuestro tipo de `Año/mes/dia`, y finalmente se llama la columna `Open`.
## Cuarto Paso
Generalizar para poder correr el código con todos nuestras acciones, como ya tenemos el camino medio hecho por el bloque anterior creamos el siguiente código: 

```
if (len(miembros)>1):
    for x in range(1,len(miembros)):
    newdata=yf.Ticker(miembros[x]).history(period='max').reset_index([['Date','Open']]
newdata['Date'] = newdata['Date'].dt.strftime('%Y/%m/%d')
newdata = newdata.rename(columns = {'Open':miembros[x]})
basedatos = pd.merge(basedata, newdata)
```

**Muy importante dejar el código anterior**, puesto que por hayamos creado llamado generalizado necesita que exista nuestro `basedatos`. 
## Quinto Paso
Ver cuanto rentabilizo nuestra acción con respecto al instante de tiempo que deseamos, para mi es del `2018-01-01` hasta el `2024-11-05` de hoy con este simple código podemos ver:

```
for x in miembros:
    basedatos[x] = basedatos[x]/(basedatos[x].iloc[0])
```
## Sexto Paso
Generar una función que utilice el peso, el listado de nuestras acciones y finalmente llamarlo portafolio. De la siguiente manera:

```
def portafolio(peso,data,nombre):
    data[nombre] = sum([int(peso[x])*data[x]/100 for x in list(peso.keys())])
    return data
```

## Séptimo Paso
Visualizar nuestra cartera en función de distintas proporciones
![PF1.webp](/assets/img/post/PF1.webp)

Observamos que la opción del portafolio 2, que corresponde a 90% de nuestra inversión en `NVDA` es la que mas genera, luego nuestro portafolio 1 que tienen igual proporción de inversión es buena, y la que menos renta cuando le damos mayor relevancia a `Bitcoin`

# Conclusión
En proyecto se enseñó  como realizar portafolio simple de 2 inversiones con distintas proporciones para poder entender la importancia de diversificar, al igual que el precio de oportunidad al hacerlo. Podemos realizar un portafolio mas grande si se desea expandiendo el código con las inversiones deseadas según el ticket que tenga presente en Yahoo! finanzas, al igual que jugar con las proporciones que deseamos invertir en nuestra cartera. Esta es una de muchas formas de entender como se mueve el mercado y como es posible invertir, ya habíamos trabajado para realizar grafico de velas. 

Querido lector espero que continúe su aventura de aprendizaje, puesto que siempre existe algo nuevo que aprender, o a veces en donde somos expertos se nos señala algo que no vimos.
