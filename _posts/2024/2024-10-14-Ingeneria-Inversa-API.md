---
title: Ingeniería Inversa API
date: 2024-10-14 12:00:00 -500
categories: [Python, DS]
tags: [ds]
image:
  path: /assets/img/post/ING.webp
---
# Ingeniería inversa API

Vamos a ver una aplicación sencilla de ingeniería inversa en un problema real para poder obtener la información del sitio web. 

Para nuestra aventura utilizaremos la siguiente pagina: `https://www.liquidos.cl/categorias/despacho-gratis`

Si intentamos obtener la información de la forma clásica de `web scraping`, no podremos obtener los datos puesto es una sitio `dinámico`, no carga toda su información al instante como seria en un caso perfecto de forma `estática`.  

Es por esto que debemos ver la información del `back end` del sitio y comprender como funciona. 

### Entonces hacemos los siguientes pasos:

1. Inspeccionar elementos con nuestro browser de preferencia
![p1.webp](/assets/img/post/p1.webp)

2. Dirigirse a `Network` para ver que elementos son llamados
![p2.webp](/assets/img/post/p2.webp)

3. Seleccionar el filtro de `Fetch/XHR`
![p3.webp](/assets/img/post/p3.webp)

4. Observar que `API` esta siendo llamada cuando ingresamos al sitio, es una buen ejercicio volver a cargar el sitio para ver el orden. ![p4.webp](/assets/img/post/p4.webp)

5. Ingresamos a cada link  hasta encontrar la información deseada
![p5.webp](/assets/img/post/p5.webp)

6. Una vez ingresado a link correcto observaremos un ladrillo de texto, podemos cerciorarse  que corresponde al correcto buscando con `Crt + f` los términos deseados. Si utilizamos el navegador `brave` podemos utilizar su función `pretty print` para que sea mas amigable a la vista.

Existen 2 formas de traer estos datos, la primera y mas usada es mediante la función `curl` llamar el link del la `API` con la información. La otra forma que es la que vamos a utilizar es descargar la información del `API` y de esta manera dejar registro atemporal y reproducible.

### Manos a la obra 
Tenemos nuestro archivo `liquidos.JSON` que generemos gracias al `API`, sabemos que es `JSON` por su forma:

![p6.webp](/assets/img/post/p6.webp)

Mi herramienta favorita para este tipo de actividades es `Python` , como sabemos que vamos a trabajar con un `json` importamos su librería.  Vamos a ver el script por partes para entender como funciona

```
import json
  
with open('Liquidos.json') as json_file:

    data = json.load(json_file)
```
con la función `with open` podemos abrir un archivo local, en nuestro caso lo convertimos a un `json_file`, entonces cargamos los datos con la variable `data` que llame la función `json.load` que carga archivos `.json` para nuestra manipulación de datos. 

```
print(data['data']['products'][0]['name'])
print(data['data']['products'][0]['prices'][0]['price'])
```
Con los `print` descubrimos la ruta del archivo que llama las variables que nosotros queremos recolectar, en nuestro caso es el `nombre` y su `precio`. 

Nuestra ***raíz*** `data['data']['products'][x]` nos entrega toda la información de los `17` elementos que deseamos y con esta podemos definir un ciclo para extraer toda la información.

```
lista_productos = []

  

with open('Liquidos.json') as json_file:

    data = json.load(json_file)

    nombre = []

    precio = []

    for x in range(17):

        nombre = (data['data']['products'][x]['name'])

        precio =(data['data']['products'][x]['prices'][0]['price'])

  

        producto = {

            'Producto': nombre,

            'Precio': precio

        }

        lista_productos.append(producto)
```
Nos entrega una lista con los productos que deseamos de una forma amigable la vista, si deseamos generar un `dataframe` véase el código en [GITHUB](https://github.com/smrj23/Scraps/blob/main/Liquidosdf.py).

Entonces en la post aprendimos como realizar `Ingeniería inversa` para un caso real extracción de datos, una manera mas sencilla y amigable para entender como funciona `.json`, y escribir un script en `Python` para unir toda la aventura. Recordemos que para nuestras aventuras de `web scraping` van a variar según lo que enfrentemos, pero espero ***querido lector*** que estés cada vez mas cerca de conseguirlo.  
