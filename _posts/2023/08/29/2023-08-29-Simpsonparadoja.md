---
layout: post
title:  "Paradoja de Simpson"
date:   2023-08-29 08:45:15 -0400

---

## <i>Paradoja de Asociación</i>

<div>
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/08/29/assets/plotly_output.gif?raw=true" />
</div>

<br>
El comportamiento de los datos en el <i>gif</i>, luce contradictorio, la tendencia general de los datos va en una dirección , mientras la tendencia por grupos es contraria. Usualmente se le conoce como <i>Paradoja de Simpson</i>, y en [Wikipedia](https://es.wikipedia.org/wiki/Paradoja_de_Simpson) aparece definido, en términos generales, del modo siguiente:

La paradoja de Simpson, es un fenómeno en el cual una tendencia aparece en varios grupos de datos; pero desaparece cuando estos grupos se combinan.

Merece atención porque puede desvirtuar los resultados de algunos estudios, en los que se podría terminar asignando causas espurias a algunos efectos.

En el ámbito de la investigación experimental, donde la aleatorización sea realmente efectiva, no es de esperar la ocurrencia de este fenómeno. Sin embargo, en el área de las ciencias sociales, ocurre con frecuencia.

Los remedios comunes consisten en encontrar un modo de tratar adecuadamente el factor de confusión <i>(confounding)</i>, causante de la distorsión para así neutralizar el efecto indeseado.

### Ejemplos.

En el artículo de Wikipedia acerca de este asunto (versión en inglés), entre varios ejemplos mencionan los promedios de bateo.


```python
import pandas as pd
```


```python
df = pd.read_html("https://en.wikipedia.org/wiki/Simpson%27s_paradox")
```

Lo siguiente es solo para darle formato a la tabla extraida.


```python
df[3].columns = """Nombre HT-1995 P-1995 HT-1996
                P-1996 
                Comb-1 Comb-2""".split()
```


```python
def resaltar(x):
    return 'font-weight: bold' if x in [0.253, 0.321, 0.31] else ''
```


```python
s = df[3].style.applymap(resaltar)
```


```python
s1 = s.format({'P-1995':'{:.3f}', 'P-1996':'{:.3f}',
         'Comb-2':'{:.3f}'})
s1
```




<style type="text/css">
#T_96aa7_row0_col6, #T_96aa7_row1_col2, #T_96aa7_row1_col4 {
  font-weight: bold;
}
</style>
<table id="T_96aa7">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_96aa7_level0_col0" class="col_heading level0 col0" >Nombre</th>
      <th id="T_96aa7_level0_col1" class="col_heading level0 col1" >HT-1995</th>
      <th id="T_96aa7_level0_col2" class="col_heading level0 col2" >P-1995</th>
      <th id="T_96aa7_level0_col3" class="col_heading level0 col3" >HT-1996</th>
      <th id="T_96aa7_level0_col4" class="col_heading level0 col4" >P-1996</th>
      <th id="T_96aa7_level0_col5" class="col_heading level0 col5" >Comb-1</th>
      <th id="T_96aa7_level0_col6" class="col_heading level0 col6" >Comb-2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_96aa7_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_96aa7_row0_col0" class="data row0 col0" >Derek Jeter</td>
      <td id="T_96aa7_row0_col1" class="data row0 col1" >12/48</td>
      <td id="T_96aa7_row0_col2" class="data row0 col2" >0.250</td>
      <td id="T_96aa7_row0_col3" class="data row0 col3" >183/582</td>
      <td id="T_96aa7_row0_col4" class="data row0 col4" >0.314</td>
      <td id="T_96aa7_row0_col5" class="data row0 col5" >195/630</td>
      <td id="T_96aa7_row0_col6" class="data row0 col6" >0.310</td>
    </tr>
    <tr>
      <th id="T_96aa7_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_96aa7_row1_col0" class="data row1 col0" >David Justice</td>
      <td id="T_96aa7_row1_col1" class="data row1 col1" >104/411</td>
      <td id="T_96aa7_row1_col2" class="data row1 col2" >0.253</td>
      <td id="T_96aa7_row1_col3" class="data row1 col3" >45/140</td>
      <td id="T_96aa7_row1_col4" class="data row1 col4" >0.321</td>
      <td id="T_96aa7_row1_col5" class="data row1 col5" >149/551</td>
      <td id="T_96aa7_row1_col6" class="data row1 col6" >0.270</td>
    </tr>
  </tbody>
</table>




HT: cociente entre hits y turnos al bate, en el año

P: promedio de bateo

Comb: agregado de ambos años 

Se observa que, en los años 1995-1996,  <i>David Justice (ATL)</i> , tuvo mayor promedio de bateo que <i>Derek Jeter (NY)</i>; sin embargo, al combinar los promedios de las dos temporadas la relación se invierte.

La ocurrencia, en este contexto, se debe a la diferencia considerable de turnos al bate entre ambos jugadores, en esas dos temporadas. Se dice también que este fenómeno se observa al menos una vez cada temporada.

#### Regresión


El siguiente ejemplo utiliza datos ficticios generados por una función, cuyo código se encuentra [aquí](https://latesco.github.io/apuntes/2023/08/15/DatosFicticiosSimpson.html).

Los datos son los mísmos que se ven en el gráfico del <i>gif</i> arriba.


```python
import DatosFicticiosSimpson as ds
import statsmodels.api as sm
import statsmodels.formula.api as smf
from linearmodels import PanelOLS
```


```python
df = ds.generAsociacion()
```


```python
mod_marginal = smf.ols('y ~ X', data = df).fit()
```


```python
mod_marginal.params
```




    Intercept    6.670389
    X            1.832375
    dtype: float64



La relación entre X e y es positiva, a la luz del resultado.


```python
mod_marginal.pvalues
```




    Intercept    7.510754e-09
    X            7.490094e-49
    dtype: float64



Condicionando por los grupos.


```python
mod_condic = smf.ols('y ~ X + Grupos - 1',  data = df).fit()
```


```python
mod_condic.params
```




    Grupos[Grupo1]    20.327267
    Grupos[Grupo2]    40.889122
    Grupos[Grupo3]    60.513787
    X                 -1.560381
    dtype: float64



Vemos que la asociación se invierte; lo cual resulta previsible al observar el gráfico; adémas son datos ficticios generados deliberadamente para que emulen  ese comportamiento.


```python
mod_condic.pvalues
```




    Grupos[Grupo1]    2.643887e-112
    Grupos[Grupo2]    6.305009e-129
    Grupos[Grupo3]    1.357479e-132
    X                  1.000014e-46
    dtype: float64



Un ejemplo, con datos no ficticios, es el que se observa entre las variables tasa de accidentes fatales y el impuesto a las bebidas alcohólicas, del conjunto de datos <i>Fatalities</i>.

<i>Fatalities</i>, es un conjunto de datos contenido en el paquete <i>AER</i> del <i>software R</i>. Allí se reporta la mortalidad en accidentes de transito en los EEUU, durante los años 1982-1988.

Con el modulo <i>statsmodels</i>, es posible obtener casi cualquier <i>dataset</i> de R.


```python
fatal = sm.datasets.get_rdataset('Fatalities',"AER")
```


```python
pd.set_option("display.max_columns", 9)
```


```python
fatal.data.head(3)
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
      <th>state</th>
      <th>year</th>
      <th>spirits</th>
      <th>unemp</th>
      <th>...</th>
      <th>milestot</th>
      <th>unempus</th>
      <th>emppopus</th>
      <th>gsp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>al</td>
      <td>1982</td>
      <td>1.37</td>
      <td>14.4</td>
      <td>...</td>
      <td>28516.0</td>
      <td>9.7</td>
      <td>57.799999</td>
      <td>-0.022125</td>
    </tr>
    <tr>
      <th>1</th>
      <td>al</td>
      <td>1983</td>
      <td>1.36</td>
      <td>13.7</td>
      <td>...</td>
      <td>31032.0</td>
      <td>9.6</td>
      <td>57.900002</td>
      <td>0.046558</td>
    </tr>
    <tr>
      <th>2</th>
      <td>al</td>
      <td>1984</td>
      <td>1.32</td>
      <td>11.1</td>
      <td>...</td>
      <td>32961.0</td>
      <td>7.5</td>
      <td>59.500004</td>
      <td>0.062798</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 34 columns</p>
</div>




```python
df = fatal.data
```


```python
df['mortalidad'] = df['fatal'] / df['pop'] * 10000
```

Al observar la relación marginal entre el impuesto y la incidencia de accidentes fatales, dentro de los grupos.


```python
res_1982 = smf.ols('mortalidad ~ beertax',
               data = df.loc[(df.year == 1982)]).fit()

res_1985 = smf.ols('mortalidad ~ beertax',
               data = df.loc[(df.year == 1985)]).fit()

res_1988 = smf.ols('mortalidad ~ beertax',
            data = df.loc[(df.year == 1988)]).fit()
```


```python
res_1982.params
```




    Intercept    2.010381
    beertax      0.148460
    dtype: float64




```python
res_1985.params
```




    Intercept    1.771162
    beertax      0.391755
    dtype: float64




```python
res_1988.params
```




    Intercept    1.859073
    beertax      0.438755
    dtype: float64



Obtenémos un resultado, según el cual, la relación es positiva (coeficientes de <i>beertax</i>), dentro de cada subconjunto de datos anuales: la mortalidad se incrementa junto con el impuesto; contrario a lo que el sentido común podría anticipar. 

Incluso la regresión con todo el conjunto de datos


```python
res_glob = smf.ols('mortalidad ~ beertax', data = df).fit()
```


```python
print(res_glob.params)
```

    Intercept    1.853308
    beertax      0.364605
    dtype: float64


Arroja un resultado que apunta a una relación positiva entre la mortalidad en accidentes y el impuesto a la bebidas alcohólicas.

Sin embargo, utilizando un modelo de efectos fijos, es decir, condicionando por estado y tiempo.


```python
datos = df.set_index(['state', 'year'])
```


```python
mod = PanelOLS(datos.mortalidad, datos['beertax'], entity_effects = True)
```


```python
res_grupos = mod.fit(cov_type='clustered', cluster_entity=True)
```


```python
res_grupos.params
```




    beertax   -0.655874
    Name: parameter, dtype: float64



El coeficiente cambia de signo: la relación se invierte.

Al incluir el factor tiempo y las posibles diferencias existentes entre estados, la situación cambia, y parece indicar que, a la larga, la relación entre el impuesto y la mortalidad en accidentes de transito tiene sentidos opuestos.

El coeficiente es significativo al 95%.


```python
res_grupos.pvalues
```




    beertax    0.023885
    Name: pvalue, dtype: float64



Estos son los p-valores, en los subconjuntos, 82, 85, 88 y con el conjunto completo.


```python
print(res_1982.pvalues[1], res_1985.pvalues[1], res_1988.pvalues[1], res_glob.pvalues[1], end = ' ')
```

    0.43465799447072817 0.014789150230907159 0.0105031104670172 1.0821720591364002e-08 

Por supuesto que no estamos pretendiendo indicar cuál especificación sería apropiada en un modelo para estos datos, un modelo adecuado, seguramente incluiría otras variables; la mención surge solo como ejemplo de dos variables cuya relación se invierte, al condicionar por grupos.

El asunto es que no siempre existe claridad sobre cuál asociación debe considerarse espuria; la marginal o la condicional. Existen ejemplos, para uno y otro caso, aunque al parecer suele ser más frecuente considerar espuria a la relación marginal.

Incluso, si en un algún caso pudiera determinarse con certeza que un comportamiento como este a nivel muestral, refleja fielmente al de la población; seguramente habría que concluir que que ambas asociaciones, la marginal y la condicional, son correctas.

Al parecer, en muchos casos es la información contextual disponible acerca del asunto que se estudia, lo que ofrece alguna orientación para decidir al respecto.
