---
layout: post
title:  "Python, Generadores y SQLite"
date:   2022-09-29 01:55:51 -0400

---

## Dos Maneras de Consultar un Archivo sin Cargarlo a RAM

Ocasionalmente puede resultar conveniente evitar posibles congestiones en la memoria del computador; los generadores y SQLite, son dos recursos que, entre otras cosas, permiten efectuar cálculos sobre un archivo sin cargarlo a la memoria RAM.

### Generadores

Un generador, es una función que se comporta a la manera de un objeto sobre el cual es posible hacer iteraciones. En el caso particular de un archivo de datos, tiene una conexion con la ruta del archivo, entregando un solo resultado a la vez; con la ayuda de un <i>bucle</i> podrán hacerse calculos sobre filas o columnas. 


```python
import os
import sys

p0 = round(os.path.getsize('./data/OnlineRetail.csv') / 1024 ** 2, 2)
print(str(p0) + ' Mb')
```

    43.99 Mb


Este archivo separado por comas, no es muy grande realmente; pero sirve como ejemplo, puede obtenerse facilmente a través de  internet.

Con la línea de código, ejecutada en la celda inferior, creamos un generador. Es la misma sintaxis de una <i>list comprehension</i>, solo que encapsulada en parentesis, en lugar de corchetes.

El <i>encoding</i>, es necesario debido a que el formato del archivo separa las columnas con ';' en vez de ','.


```python
genr = (fila for fila in open('./data/OnlineRetail.csv',
                              encoding = 'ISO-8859-1'))
```

Este archivo permite la separación por columnas, y guarda el encabezado del archivo <i>OnlineRetail.csv</i> en el objeto <i>cols</i>


```python
genr = (s.rstrip().split(';') for s in genr)
cols = next(genr)

cols
```




    ['InvoiceNo',
     'StockCode',
     'Description',
     'Quantity',
     'InvoiceDate',
     'UnitPrice',
     'CustomerID',
     'Country']



Esto siguiente, es equivalente a consultar la primera fila del archivo, después del encabezado.


```python
dct = (dict(zip(cols, d)) for d in genr)

next(dct)
```




    {'InvoiceNo': '536365',
     'StockCode': '85123A',
     'Description': 'WHITE HANGING HEART T-LIGHT HOLDER',
     'Quantity': '6',
     'InvoiceDate': '01/12/2010 08:26',
     'UnitPrice': '2,55',
     'CustomerID': '17850',
     'Country': 'United Kingdom'}



La indole de los generadores hace que solo puedan hacerse lecturas de modo progresivo; es decir, una vez que lees una fila, ya solo te entregará la siguiente. De lo contrario debes repetir el proceso. Como en el caso siguiente:


```python
genr = (fila for fila in open('./data/OnlineRetail.csv', encoding = 'ISO-8859-1'))
genr = (s.rstrip().split(';') for s in genr)
cols = next(genr)
dct = (dict(zip(cols, d)) for d in genr)
```


```python
print(str(sys.getsizeof(dct)) + ' bytes')
```

    112 bytes


Vemos que lo que el peso del generador en la memoria es apenas una infima fracción del archivo.

Calculémos, por ejemplo, el total de la cantidad de mercancias vendidas en el Reino Unido:


```python
q = (int(d['Quantity'])  for d in dct if d['Country'] == 'United Kingdom')

sum(q)
```




    4263829




```python
genr = (fila for fila in open('./data/OnlineRetail.csv', encoding = 'ISO-8859-1'))
genr = (s.rstrip().split(';') for s in genr)
cols = next(genr)
dct = (dict(zip(cols, d)) for d in genr)


Ventas = (int(d['Quantity']) * float(d['UnitPrice'].replace(',', '.')) for d in dct)

sum(Ventas)
```




    9747747.93400317



Claro está, dependiendo de los cálculos que se quieran hacer, este método puede no ser el más cómodo del mundo.

Sin embargo, <i>SQLite</i> ofrece muchas ventajas en cuanto a facilidad para hacer muchas operaciones.

### SQLite


```python
import sqlite3
import pandas as pd
```


```python
ruta = './data/OnlineRetail.csv'
con = sqlite3.connect(os.path.join('./data/', 'OnlineRetail.db'))

cur = con.cursor()
```

Con <i>sqlite</i> puede crearse una base de datos, sin servidor. Incluso puede crearse en la memoria del computador; claro que para este ejemplo, eso sería contradictorio con el propósito de no congestionar la memoria RAM.

El siguiente <i>bucle</i> importa fracciones del archivo y las agrega a una base de datos <i>sqlite</i>.


```python
for parte in pd.read_csv(ruta, encoding = 'ISO-8859-1',
                        sep = ';',
                        decimal = ',',
                        chunksize = 500):
    parte.to_sql('onlineretail', con, if_exists = 'append')
```


```python
r = sys.getsizeof(parte)
r = round(r/ 1024**2, 2)
print(str(r) + ' Mb')
```

    0.15 Mb


Lo que queda cargado es solo una fracción de 409 filas


```python
parte.shape # las últimas 409 filas del archivo
```




    (409, 8)



De las cuales se puede prescindir impunemente.


```python
del parte 
```

Ahora se pueden ejecutar todo tipo de consultas sobre el archivo convertido a base de datos.


```python
cur.execute("""SELECT SUM(UnitPrice * Quantity) FROM onlineretail;
            """).fetchall() # EL mismo cálculo hecho arriba con generadores
```




    [(9747747.93400317,)]



El número de registros o transacciones por países.


```python
cur.execute("""SELECT Country, COUNT(*) as N
            from onlineretail
            GROUP BY Country
            ORDER BY N DESC;
            """).fetchall()
```




    [('United Kingdom', 495478),
     ('Germany', 9495),
     ('France', 8557),
     ('EIRE', 8196),
     ('Spain', 2533),
     ('Netherlands', 2371),
     ('Belgium', 2069),
     ('Switzerland', 2002),
     ('Portugal', 1519),
     ('Australia', 1259),
     ('Norway', 1086),
     ('Italy', 803),
     ('Channel Islands', 758),
     ('Finland', 695),
     ('Cyprus', 622),
     ('Sweden', 462),
     ('Unspecified', 446),
     ('Austria', 401),
     ('Denmark', 389),
     ('Japan', 358),
     ('Poland', 341),
     ('Israel', 297),
     ('USA', 291),
     ('Hong Kong', 288),
     ('Singapore', 229),
     ('Iceland', 182),
     ('Canada', 151),
     ('Greece', 146),
     ('Malta', 127),
     ('United Arab Emirates', 68),
     ('European Community', 61),
     ('RSA', 58),
     ('Lebanon', 45),
     ('Lithuania', 35),
     ('Brazil', 32),
     ('Czech Republic', 30),
     ('Bahrain', 19),
     ('Saudi Arabia', 10)]



Pero <i>Pandas</i> tiene un método <i>.read_sql</i> que convierte el resultado de una <i>query</i> en <i>dataframe</i>.


```python
ds = pd.read_sql("""SELECT Country, COUNT(*) as N
            from onlineretail
            GROUP BY Country
            ORDER BY N DESC;
            """, con)

ds
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Country</th>
      <th>N</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>United Kingdom</td>
      <td>495478</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Germany</td>
      <td>9495</td>
    </tr>
    <tr>
      <th>2</th>
      <td>France</td>
      <td>8557</td>
    </tr>
    <tr>
      <th>3</th>
      <td>EIRE</td>
      <td>8196</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Spain</td>
      <td>2533</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Netherlands</td>
      <td>2371</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Belgium</td>
      <td>2069</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Switzerland</td>
      <td>2002</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Portugal</td>
      <td>1519</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Australia</td>
      <td>1259</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Norway</td>
      <td>1086</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Italy</td>
      <td>803</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Channel Islands</td>
      <td>758</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Finland</td>
      <td>695</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Cyprus</td>
      <td>622</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Sweden</td>
      <td>462</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Unspecified</td>
      <td>446</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Austria</td>
      <td>401</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Denmark</td>
      <td>389</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Japan</td>
      <td>358</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Poland</td>
      <td>341</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Israel</td>
      <td>297</td>
    </tr>
    <tr>
      <th>22</th>
      <td>USA</td>
      <td>291</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Hong Kong</td>
      <td>288</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Singapore</td>
      <td>229</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Iceland</td>
      <td>182</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Canada</td>
      <td>151</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Greece</td>
      <td>146</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Malta</td>
      <td>127</td>
    </tr>
    <tr>
      <th>29</th>
      <td>United Arab Emirates</td>
      <td>68</td>
    </tr>
    <tr>
      <th>30</th>
      <td>European Community</td>
      <td>61</td>
    </tr>
    <tr>
      <th>31</th>
      <td>RSA</td>
      <td>58</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Lebanon</td>
      <td>45</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Lithuania</td>
      <td>35</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Brazil</td>
      <td>32</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Czech Republic</td>
      <td>30</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Bahrain</td>
      <td>19</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Saudi Arabia</td>
      <td>10</td>
    </tr>
  </tbody>
</table>
</div>



Total de ingresos, cantidades y promedio de precios por países 


```python
ds = pd.read_sql("""SELECT Country, SUM(Quantity * UnitPrice) AS ingresos,
            SUM(Quantity) AS Cantidad, AVG(UnitPrice) AS precio
            FROM onlineretail
            GROUP BY Country;""", con)

ds
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Country</th>
      <th>ingresos</th>
      <th>Cantidad</th>
      <th>precio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Australia</td>
      <td>1.370773e+05</td>
      <td>83653</td>
      <td>3.220612</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Austria</td>
      <td>1.015432e+04</td>
      <td>4827</td>
      <td>4.243192</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bahrain</td>
      <td>5.484000e+02</td>
      <td>260</td>
      <td>4.556316</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Belgium</td>
      <td>4.091096e+04</td>
      <td>23152</td>
      <td>3.644335</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Brazil</td>
      <td>1.143600e+03</td>
      <td>356</td>
      <td>4.456250</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Canada</td>
      <td>3.666380e+03</td>
      <td>2763</td>
      <td>6.030331</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Channel Islands</td>
      <td>2.008629e+04</td>
      <td>9479</td>
      <td>4.932124</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Cyprus</td>
      <td>1.294629e+04</td>
      <td>6317</td>
      <td>6.302363</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Czech Republic</td>
      <td>7.077200e+02</td>
      <td>592</td>
      <td>2.938333</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Denmark</td>
      <td>1.876814e+04</td>
      <td>8188</td>
      <td>3.256941</td>
    </tr>
    <tr>
      <th>10</th>
      <td>EIRE</td>
      <td>2.632768e+05</td>
      <td>142637</td>
      <td>5.911077</td>
    </tr>
    <tr>
      <th>11</th>
      <td>European Community</td>
      <td>1.291750e+03</td>
      <td>497</td>
      <td>4.820492</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Finland</td>
      <td>2.232674e+04</td>
      <td>10666</td>
      <td>5.448705</td>
    </tr>
    <tr>
      <th>13</th>
      <td>France</td>
      <td>1.974039e+05</td>
      <td>110480</td>
      <td>5.028864</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Germany</td>
      <td>2.216982e+05</td>
      <td>117448</td>
      <td>3.966930</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Greece</td>
      <td>4.710520e+03</td>
      <td>1556</td>
      <td>4.885548</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Hong Kong</td>
      <td>1.011704e+04</td>
      <td>4769</td>
      <td>42.505208</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Iceland</td>
      <td>4.310000e+03</td>
      <td>2458</td>
      <td>2.644011</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Israel</td>
      <td>7.907820e+03</td>
      <td>4353</td>
      <td>3.633131</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Italy</td>
      <td>1.689051e+04</td>
      <td>7999</td>
      <td>4.831121</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Japan</td>
      <td>3.534062e+04</td>
      <td>25218</td>
      <td>2.276145</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Lebanon</td>
      <td>1.693880e+03</td>
      <td>386</td>
      <td>5.387556</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Lithuania</td>
      <td>1.661060e+03</td>
      <td>652</td>
      <td>2.841143</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Malta</td>
      <td>2.505470e+03</td>
      <td>944</td>
      <td>5.244173</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Netherlands</td>
      <td>2.846615e+05</td>
      <td>200128</td>
      <td>2.738317</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Norway</td>
      <td>3.516346e+04</td>
      <td>19247</td>
      <td>6.012026</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Poland</td>
      <td>7.213140e+03</td>
      <td>3653</td>
      <td>4.170880</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Portugal</td>
      <td>2.936702e+04</td>
      <td>16180</td>
      <td>8.582976</td>
    </tr>
    <tr>
      <th>28</th>
      <td>RSA</td>
      <td>1.002310e+03</td>
      <td>352</td>
      <td>4.277586</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Saudi Arabia</td>
      <td>1.311700e+02</td>
      <td>75</td>
      <td>2.411000</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Singapore</td>
      <td>9.120390e+03</td>
      <td>5234</td>
      <td>109.645808</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Spain</td>
      <td>5.477458e+04</td>
      <td>26824</td>
      <td>4.987544</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Sweden</td>
      <td>3.659591e+04</td>
      <td>35637</td>
      <td>3.910887</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Switzerland</td>
      <td>5.638535e+04</td>
      <td>30325</td>
      <td>3.403442</td>
    </tr>
    <tr>
      <th>34</th>
      <td>USA</td>
      <td>1.730920e+03</td>
      <td>1034</td>
      <td>2.216426</td>
    </tr>
    <tr>
      <th>35</th>
      <td>United Arab Emirates</td>
      <td>1.902280e+03</td>
      <td>982</td>
      <td>3.380735</td>
    </tr>
    <tr>
      <th>36</th>
      <td>United Kingdom</td>
      <td>8.187806e+06</td>
      <td>4263829</td>
      <td>4.532422</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Unspecified</td>
      <td>4.749790e+03</td>
      <td>3300</td>
      <td>2.699574</td>
    </tr>
  </tbody>
</table>
</div>


