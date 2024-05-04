---
layout: post
title:  "Segmentación de clientes"
date:   2023-07-27 11:00:47 -0400

---


## <i>RFM</i>

RFM, es una técnica segmentación de clientes en torno a 3 métricas: <i>recency</i> (compras recientes);
<i>frequency</i> (frecuencia de compras); y <i>monetary</i> (valor monetario de las compras).

Las líneas a continuación, muestran un ejemplo utilizando datos de compras a través de internet.


```python
import pandas as pd

import numpy as np

import matplotlib.pyplot as plt

import matplotlib as mpl

import seaborn as sns
```

En la celda inferior, se establecen algunas opciones para gráficos.


```python
plt.style.use('fivethirtyeight')

plt.rcParams['figure.figsize'] = (9, 6)

mpl.rcParams['lines.linewidth'] = 2

mpl.rcParams['font.size'] = 10

plt.rc(
    "axes",
    labelweight="bold",
    
    labelsize="large",
    
    titleweight="bold",
    
    titlesize=14,
    
    titlepad=10,
)
```

## <i>Carga de datos y ajustes


```python
p0 = '../OnlineRetail.csv'

df = pd.read_csv(p0, encoding= 'ISO-8859-1',
                 sep = ';', decimal=',')

df['InvoiceDate'] = pd.to_datetime(df['InvoiceDate'], format='%d/%m/%Y %H:%M')
```


```python
df.Country.nunique()
```




    38



Los clientes de la tienda proceden de diversos (38) países, para evitar la posibilidad de que al crear los grupos <i>(clusters)</i>, estos varien según el área geográfica, el ejemplo se restringirá a clientes de un solo país.

En la celda inferior, añado una nueva variable, 'ingreso'.


```python
df = df.loc[(df.Country == 'United Kingdom')]

df['ingreso'] = df['Quantity'].multiply(df['UnitPrice'])

df = df.loc[~ df['CustomerID'].isna()]
```


```python
print(f"Número de facturas: {df.InvoiceNo.nunique()}",
      f"\n\nNúmero de clientes: {df.CustomerID.nunique()}")
```

    Número de facturas: 19857 
    
    Número de clientes: 3950



```python
df[['InvoiceDate','ingreso']].set_index('InvoiceDate').plot()

plt.title('Ingresos en 2011')

plt.legend('')

plt.xlabel('Fecha de factura')

plt.show()
```


 
 
 <div>
 
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/07/27/assets/output_12_0.png?raw=true" /> 
  
</div>   

 

Resaltan algunas facturas que presentan devoluciones, en la tabla de datos están identificadas con letra 'C'; sin embargo, decidí prescindir de ellas, puesto que no representan un ingreso real.


```python
def remove_negs(X):

    filt = X.loc[(df['ingreso'] <= 10)]['ingreso'].to_numpy()

    for ing in filt:

        if ing < -1000:
            
            min, max = X['ingreso'].agg(['min', 'max'])
            
            if min == -1 * max:
                
                rem = X.loc[(df['ingreso'] == min) | (df['ingreso'] == max)]
                
                X.drop(rem.index, inplace=True)
            
    return X
```


```python
df = remove_negs(df)
```


```python
df = df.loc[(df['ingreso'] > 0)]
```

## RFM 

Para facilitar el cálculo de las métricas del RFM, usarémos la siguiente función.


```python
df_ag_ing = df.groupby('CustomerID')['ingreso'].sum().to_frame('ingreso')
```


```python
def rfm(datos):
    
    compraMasReciente = df['InvoiceDate'].max()
    
    df_rfm = (df.groupby('CustomerID')
              
              .agg({'InvoiceDate': lambda x: (compraMasReciente - x.max()).days,

                              'InvoiceNo': 'count',

                              'ingreso': 'sum'
                             })
             )
    
    df_rfm.columns = 'Reciente Frecuencia Monetario'.split()

    df_rfm['rank_R'] = pd.qcut(df_rfm['Reciente'],
                               
                               q = 5, labels = (5, 4, 3, 2, 1))

    df_rfm['rank_F'] = pd.qcut(df_rfm['Frecuencia'],
                               
                               q = 5, labels = (1, 2, 3, 4, 5))

    df_rfm['rank_M'] = pd.qcut(df_rfm['Monetario'],
                               
                               q = 5, labels = (1, 2, 3, 4, 5))

    


    df_rfm['puntajes'] = df_rfm[['rank_R', 'rank_F', 'rank_M']].astype(str).agg('-'.join, axis=1)

    df_rfm['puntaje_rfm'] = df_rfm[['rank_R', 'rank_F', 'rank_M']].astype(int).sum(axis=1)

    

    min_, d1,  q1, q2, q3, d2, max_ = (np.min(df_rfm['puntaje_rfm']),
                              
                              np.quantile(df_rfm['puntaje_rfm'], 0.10), 
                              
                              np.quantile(df_rfm['puntaje_rfm'], 0.25),
                              
                              np.quantile(df_rfm['puntaje_rfm'], 0.5),
                              
                              np.quantile(df_rfm['puntaje_rfm'], 0.75),

                              np.quantile(df_rfm['puntaje_rfm'], 0.9),
                              
                              np.max(df_rfm['puntaje_rfm'])
                           
                           )

    df_rfm['segmento'] = np.select(
        
    [
        df_rfm['puntaje_rfm'].between(min_, d1, inclusive='left'),

        df_rfm['puntaje_rfm'].between(d1, q1, inclusive='left'),

        df_rfm['puntaje_rfm'].between(q1, q2, inclusive='left'),

        df_rfm['puntaje_rfm'].between(q2, q3, inclusive='left'),

        df_rfm['puntaje_rfm'].between(q3, d2, inclusive='left'),

        df_rfm['puntaje_rfm'].between(q3, max_, inclusive='both'),
    ], 
        [
            'muy bajo',

            'bajo',
            
            'medio-bajo',

            'medio-alto',
            
            'alto',
            
            'superior'
        ]
        
        )
    
    return df_rfm                               
```

Las métricas pueden ponderarse, sobre la base de los intereses y objetivos del negoció en particular, en este caso solo se trata de un ejemplo, de modo que la función simplemente genera puntajes a partir de los quintiles.


```python
d0 = rfm(df)
```


```python
d0.iloc[:, 3:].head()
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
      <th>rank_R</th>
      <th>rank_F</th>
      <th>rank_M</th>
      <th>puntajes</th>
      <th>puntaje_rfm</th>
      <th>segmento</th>
    </tr>
    <tr>
      <th>CustomerID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12747.0</th>
      <td>5</td>
      <td>4</td>
      <td>5</td>
      <td>5-4-5</td>
      <td>14</td>
      <td>superior</td>
    </tr>
    <tr>
      <th>12748.0</th>
      <td>5</td>
      <td>5</td>
      <td>5</td>
      <td>5-5-5</td>
      <td>15</td>
      <td>superior</td>
    </tr>
    <tr>
      <th>12749.0</th>
      <td>5</td>
      <td>5</td>
      <td>5</td>
      <td>5-5-5</td>
      <td>15</td>
      <td>superior</td>
    </tr>
    <tr>
      <th>12820.0</th>
      <td>5</td>
      <td>4</td>
      <td>4</td>
      <td>5-4-4</td>
      <td>13</td>
      <td>alto</td>
    </tr>
    <tr>
      <th>12821.0</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1-1-1</td>
      <td>3</td>
      <td>muy bajo</td>
    </tr>
  </tbody>
</table>
</div>



El nuevo <i>dataset</i> contiene los segmentos del RFM. A continuación, una forma de visualizarlos:


```python
d01 = d0.segmento.value_counts().to_frame('ingreso').sort_values('ingreso', ascending=False)
```


```python
d01 = d01.reset_index()
```


```python
d01['segmento'] = pd.Categorical(d01['segmento'],
                                 categories = ['superior', 'alto', 'medio-alto',
                                               'medio-bajo', 'bajo', 'muy bajo'],
                                ordered=True)

d01 = d01.sort_values('segmento')
```


```python
colores = sns.color_palette("vlag")
```


```python
plt.pie(d01.ingreso, #d0.segmento.value_counts()[ord],
        
        labels=d01.segmento,

        colors = colores,
        
        autopct='%.0f%%')

plt.title('Proporción de segmentos')

plt.show()
```

<div>
 
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/07/27/assets/output_29_0.png?raw=true" /> 
  
</div>
    



¿Cómo se distribuye el ingreso entre estos segmentos?


```python
dq = df_ag_ing.join(d0['segmento'])
```


```python
dt = (((dq.reset_index().groupby('segmento')[['CustomerID','ingreso']]

        .agg({

            'ingreso': lambda x: round((x.sum() / dq.ingreso.sum()) * 100, 2),
            
            'CustomerID': lambda x: round((x.nunique() / df.CustomerID.nunique()) * 100, 2),

            
            
        }
            
            
            )

    ))).sort_values('ingreso', ascending=False)
```


```python
dt.columns = ['Ingreso', 'N°Clientes']

ax = sns.barplot(y = 'value', x = 'segmento',

                 hue='variable',

                 palette=colores,
                 
                 data = dt.reset_index().melt(id_vars='segmento'))

ax.bar_label(ax.containers[0], fontsize=10)

ax.bar_label(ax.containers[1], fontsize=10)

h1, l1 = ax.get_legend_handles_labels()
h2, l2 = ax.get_legend_handles_labels()

ax.legend(handles=h1[1:], labels=l1[1:])
ax.legend(handles=h1[2:], labels=l1[2:])

plt.title('Proporción de ingreso y clientes\npor segmentos')

plt.ylabel('')

plt.ylim((0, 60))

plt.yticks(color='w')

plt.show()
```

<div>
 
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/07/27/assets/output_33_0.png?raw=true" /> 
  
</div>
    


A continuación se muestra cómo es la distribución de la variable 'ingreso' entre los segmentos.


```python

((dq.groupby('segmento')['ingreso']
     .describe())
     .sort_values('max', ascending=False)) .style.format(precision=2)
```




<style type="text/css">
</style>
<table id="T_40846">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_40846_level0_col0" class="col_heading level0 col0" >count</th>
      <th id="T_40846_level0_col1" class="col_heading level0 col1" >mean</th>
      <th id="T_40846_level0_col2" class="col_heading level0 col2" >std</th>
      <th id="T_40846_level0_col3" class="col_heading level0 col3" >min</th>
      <th id="T_40846_level0_col4" class="col_heading level0 col4" >25%</th>
      <th id="T_40846_level0_col5" class="col_heading level0 col5" >50%</th>
      <th id="T_40846_level0_col6" class="col_heading level0 col6" >75%</th>
      <th id="T_40846_level0_col7" class="col_heading level0 col7" >max</th>
    </tr>
    <tr>
      <th class="index_name level0" >segmento</th>
      <th class="blank col0" >&nbsp;</th>
      <th class="blank col1" >&nbsp;</th>
      <th class="blank col2" >&nbsp;</th>
      <th class="blank col3" >&nbsp;</th>
      <th class="blank col4" >&nbsp;</th>
      <th class="blank col5" >&nbsp;</th>
      <th class="blank col6" >&nbsp;</th>
      <th class="blank col7" >&nbsp;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_40846_level0_row0" class="row_heading level0 row0" >superior</th>
      <td id="T_40846_row0_col0" class="data row0 col0" >559.00</td>
      <td id="T_40846_row0_col1" class="data row0 col1" >7111.74</td>
      <td id="T_40846_row0_col2" class="data row0 col2" >16446.64</td>
      <td id="T_40846_row0_col3" class="data row0 col3" >901.21</td>
      <td id="T_40846_row0_col4" class="data row0 col4" >2429.41</td>
      <td id="T_40846_row0_col5" class="data row0 col5" >3628.50</td>
      <td id="T_40846_row0_col6" class="data row0 col6" >5880.84</td>
      <td id="T_40846_row0_col7" class="data row0 col7" >259657.30</td>
    </tr>
    <tr>
      <th id="T_40846_level0_row1" class="row_heading level0 row1" >alto</th>
      <td id="T_40846_row1_col0" class="data row1 col0" >576.00</td>
      <td id="T_40846_row1_col1" class="data row1 col1" >2282.73</td>
      <td id="T_40846_row1_col2" class="data row1 col2" >3991.96</td>
      <td id="T_40846_row1_col3" class="data row1 col3" >303.09</td>
      <td id="T_40846_row1_col4" class="data row1 col4" >1105.39</td>
      <td id="T_40846_row1_col5" class="data row1 col5" >1604.11</td>
      <td id="T_40846_row1_col6" class="data row1 col6" >2372.03</td>
      <td id="T_40846_row1_col7" class="data row1 col7" >72882.09</td>
    </tr>
    <tr>
      <th id="T_40846_level0_row2" class="row_heading level0 row2" >medio-bajo</th>
      <td id="T_40846_row2_col0" class="data row2 col0" >1031.00</td>
      <td id="T_40846_row2_col1" class="data row2 col1" >513.84</td>
      <td id="T_40846_row2_col2" class="data row2 col2" >1417.36</td>
      <td id="T_40846_row2_col3" class="data row2 col3" >20.80</td>
      <td id="T_40846_row2_col4" class="data row2 col4" >260.43</td>
      <td id="T_40846_row2_col5" class="data row2 col5" >388.25</td>
      <td id="T_40846_row2_col6" class="data row2 col6" >602.30</td>
      <td id="T_40846_row2_col7" class="data row2 col7" >44534.30</td>
    </tr>
    <tr>
      <th id="T_40846_level0_row3" class="row_heading level0 row3" >medio-alto</th>
      <td id="T_40846_row3_col0" class="data row3 col0" >945.00</td>
      <td id="T_40846_row3_col1" class="data row3 col1" >1084.41</td>
      <td id="T_40846_row3_col2" class="data row3 col2" >1030.19</td>
      <td id="T_40846_row3_col3" class="data row3 col3" >120.03</td>
      <td id="T_40846_row3_col4" class="data row3 col4" >570.96</td>
      <td id="T_40846_row3_col5" class="data row3 col5" >833.78</td>
      <td id="T_40846_row3_col6" class="data row3 col6" >1268.98</td>
      <td id="T_40846_row3_col7" class="data row3 col7" >12393.70</td>
    </tr>
    <tr>
      <th id="T_40846_level0_row4" class="row_heading level0 row4" >bajo</th>
      <td id="T_40846_row4_col0" class="data row4 col0" >574.00</td>
      <td id="T_40846_row4_col1" class="data row4 col1" >244.28</td>
      <td id="T_40846_row4_col2" class="data row4 col2" >129.19</td>
      <td id="T_40846_row4_col3" class="data row4 col3" >6.20</td>
      <td id="T_40846_row4_col4" class="data row4 col4" >147.14</td>
      <td id="T_40846_row4_col5" class="data row4 col5" >225.88</td>
      <td id="T_40846_row4_col6" class="data row4 col6" >317.79</td>
      <td id="T_40846_row4_col7" class="data row4 col7" >876.00</td>
    </tr>
    <tr>
      <th id="T_40846_level0_row5" class="row_heading level0 row5" >muy bajo</th>
      <td id="T_40846_row5_col0" class="data row5 col0" >234.00</td>
      <td id="T_40846_row5_col1" class="data row5 col1" >130.56</td>
      <td id="T_40846_row5_col2" class="data row5 col2" >53.22</td>
      <td id="T_40846_row5_col3" class="data row5 col3" >2.90</td>
      <td id="T_40846_row5_col4" class="data row5 col4" >94.97</td>
      <td id="T_40846_row5_col5" class="data row5 col5" >130.72</td>
      <td id="T_40846_row5_col6" class="data row5 col6" >172.05</td>
      <td id="T_40846_row5_col7" class="data row5 col7" >241.06</td>
    </tr>
  </tbody>
</table>




¿Qué productos se venden más en cada segmento?


```python
dq = df.set_index('CustomerID').join(d0['segmento'])

seg_grupo = dq.groupby('segmento')

for ind, conj in seg_grupo:

    print(f'Segmento:{ind}')

    print('=' * 75)

    print(conj.nlargest(5, 'Quantity'), end = '\n\n')
```

    Segmento:alto
    ===========================================================================
               InvoiceNo StockCode                         Description  Quantity   
    CustomerID                                                                     
    12931.0       562439     84879       ASSORTED COLOUR BIRD ORNAMENT      2880  \
    16333.0       543057     84077   WORLD WAR 2 GLIDERS ASSTD DESIGNS      2592   
    16029.0       539101     22693  GROW A FLYTRAP OR SUNFLOWER IN TIN      2400   
    16029.0       543669     22693  GROW A FLYTRAP OR SUNFLOWER IN TIN      2400   
    16333.0       574294     21915              RED  HARMONICA IN BOX       2100   
    
                       InvoiceDate  UnitPrice         Country  ingreso segmento  
    CustomerID                                                                   
    12931.0    2011-08-04 18:06:00       1.45  United Kingdom  4176.00     alto  
    16333.0    2011-02-03 10:50:00       0.21  United Kingdom   544.32     alto  
    16029.0    2010-12-16 10:35:00       0.94  United Kingdom  2256.00     alto  
    16029.0    2011-02-11 11:22:00       0.94  United Kingdom  2256.00     alto  
    16333.0    2011-11-03 15:47:00       1.06  United Kingdom  2226.00     alto  
    
    Segmento:bajo
    ===========================================================================
               InvoiceNo StockCode                        Description  Quantity   
    CustomerID                                                                    
    12875.0       538420     17096  ASSORTED LAQUERED INCENSE HOLDERS      1728  \
    15118.0       561638     84568    GIRLS ALPHABET IRON ON PATCHES       1440   
    16377.0       546863     17003                BROCADE RING PURSE        720   
    12891.0       551595     21498                RED RETROSPOT WRAP        600   
    16377.0       546863     15036          ASSORTED COLOURS SILK FAN       600   
    
                       InvoiceDate  UnitPrice         Country  ingreso segmento  
    CustomerID                                                                   
    12875.0    2010-12-12 12:03:00       0.17  United Kingdom   293.76     bajo  
    15118.0    2011-07-28 14:54:00       0.17  United Kingdom   244.80     bajo  
    16377.0    2011-03-17 15:41:00       0.25  United Kingdom   180.00     bajo  
    12891.0    2011-05-03 11:33:00       0.34  United Kingdom   204.00     bajo  
    16377.0    2011-03-17 15:41:00       0.65  United Kingdom   390.00     bajo  
    
    Segmento:medio-alto
    ===========================================================================
               InvoiceNo StockCode                     Description  Quantity   
    CustomerID                                                                 
    16308.0       573995     16014     SMALL CHINESE STYLE SCISSOR      3000  \
    14101.0       547037     21967        PACK OF 12 SKULL TISSUES      2160   
    16308.0       552172     16014     SMALL CHINESE STYLE SCISSOR      2000   
    16308.0       564272     16014     SMALL CHINESE STYLE SCISSOR      2000   
    15299.0       536809     84950  ASSORTED COLOUR T-LIGHT HOLDER      1824   
    
                       InvoiceDate  UnitPrice         Country  ingreso    segmento  
    CustomerID                                                                      
    16308.0    2011-11-02 11:24:00       0.32  United Kingdom    960.0  medio-alto  
    14101.0    2011-03-20 10:37:00       0.25  United Kingdom    540.0  medio-alto  
    16308.0    2011-05-06 13:03:00       0.32  United Kingdom    640.0  medio-alto  
    16308.0    2011-08-24 10:52:00       0.32  United Kingdom    640.0  medio-alto  
    15299.0    2010-12-02 16:48:00       0.55  United Kingdom   1003.2  medio-alto  
    
    Segmento:medio-bajo
    ===========================================================================
               InvoiceNo StockCode                          Description  Quantity   
    CustomerID                                                                      
    13135.0       554868     22197                 SMALL POPCORN HOLDER      4300  \
    18087.0       544612     22053                EMPIRE DESIGN ROSETTE      3906   
    14609.0       560599     18007  ESSENTIAL BALM 3.5g TIN IN ENVELOPE      3186   
    15749.0       540815     21108   FAIRY CAKE FLANNEL ASSORTED COLOUR      3114   
    15749.0       550461     21108   FAIRY CAKE FLANNEL ASSORTED COLOUR      3114   
    
                       InvoiceDate  UnitPrice         Country  ingreso    segmento  
    CustomerID                                                                      
    13135.0    2011-05-27 10:52:00       0.72  United Kingdom  3096.00  medio-bajo  
    18087.0    2011-02-22 10:43:00       0.82  United Kingdom  3202.92  medio-bajo  
    14609.0    2011-07-19 17:04:00       0.06  United Kingdom   191.16  medio-bajo  
    15749.0    2011-01-11 12:55:00       2.10  United Kingdom  6539.40  medio-bajo  
    15749.0    2011-04-18 13:20:00       2.10  United Kingdom  6539.40  medio-bajo  
    
    Segmento:muy bajo
    ===========================================================================
               InvoiceNo StockCode                       Description  Quantity   
    CustomerID                                                                   
    17394.0       551836     23144   ZINC T-LIGHT HOLDER STARS SMALL       216  \
    17752.0       539048     16225                 RATTLE SNAKE EGGS       192   
    16405.0       543623     21326  AGED GLASS SILVER T-LIGHT HOLDER       168   
    15753.0       543456     21326  AGED GLASS SILVER T-LIGHT HOLDER       144   
    17060.0       546996     84978  HANGING HEART JAR T-LIGHT HOLDER       144   
    
                       InvoiceDate  UnitPrice         Country  ingreso  segmento  
    CustomerID                                                                    
    17394.0    2011-05-04 13:22:00       0.72  United Kingdom   155.52  muy bajo  
    17752.0    2010-12-15 16:19:00       0.42  United Kingdom    80.64  muy bajo  
    16405.0    2011-02-10 15:24:00       0.55  United Kingdom    92.40  muy bajo  
    15753.0    2011-02-08 12:41:00       0.55  United Kingdom    79.20  muy bajo  
    17060.0    2011-03-18 13:27:00       1.06  United Kingdom   152.64  muy bajo  
    
    Segmento:superior
    ===========================================================================
               InvoiceNo StockCode                         Description  Quantity   
    CustomerID                                                                     
    12901.0       573008     84077   WORLD WAR 2 GLIDERS ASSTD DESIGNS      4800  \
    12901.0       554272     21977  PACK OF 60 PINK PAISLEY CAKE CASES      2700   
    17949.0       573261     22197                      POPCORN HOLDER      1992   
    17450.0       567423     23288           GREEN VINTAGE SPOT BEAKER      1944   
    17450.0       567423     23285            PINK VINTAGE SPOT BEAKER      1944   
    
                       InvoiceDate  UnitPrice         Country  ingreso  segmento  
    CustomerID                                                                    
    12901.0    2011-10-27 12:26:00       0.21  United Kingdom  1008.00  superior  
    12901.0    2011-05-23 13:08:00       0.42  United Kingdom  1134.00  superior  
    17949.0    2011-10-28 12:32:00       0.72  United Kingdom  1434.24  superior  
    17450.0    2011-09-20 11:05:00       1.08  United Kingdom  2099.52  superior  
    17450.0    2011-09-20 11:05:00       1.08  United Kingdom  2099.52  superior  
    

