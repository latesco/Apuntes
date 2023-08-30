---
layout: post
title:  "Paradoja de Simpson"
date:   2023-08-29 08:45:15 -0400

---

## <i>Paradoja de Asociación</i>


<div>
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/08/29/assets/plotly_output.gif?raw=true" />
</div>


El comportamiento de los datos en el <i>gif</i>, muestra una paradoja de asociación, la tendencia general de los datos va en una dirección , mientras la tendencia por grupos es contraria. Usualmente se le conoce como <i>Paradoja de Simpson</i>, y en [Wikipedia](https://en.wikipedia.org/wiki/Simpson%27s_paradox) aparece definido, en términos generales, del modo siguiente:

La paradoja de Simpson, es un fenómeno en el cual una tendencia aparece en varios grupos de datos; pero desaparece cuando estos grupos se combinan.  

Merece atención porque puede desvirtuar los resultados de algunos estudios, en los que se podría terminar asignando causas espurias a algunos efectos. 

En situaciones de investigación experimental, en las cuales la aleatorización es realmente efectiva, no es de esperar la ocurrencia de este fenómeno. En el área de las ciencias sociales, ocurre con frecuencia.

Los remedios comunes consisten en encontrar un modo de tratar adecuadamente el factor que causa la confusión <i>(confounding)</i>, causante de la distorsión para así neutralizar el efecto indeseado.

### Ejemplos.

En el artículo de Wikipedia, mencionado arriba, dan varios ejemplos. Uno de ellos referido a promedios de bateo.


```python
import pandas as pd
import os
```


```python
df = pd.read_html("https://en.wikipedia.org/wiki/Simpson%27s_paradox")
```

Renombrando las columnas de la  tabla de Wikipedia.

```python
df[3].columns ="""Nombre hitsVsTurnos-1995 promed-1995 hitsVsTurnos-1996
                promed-1996 
                Combinado-1 Combinado-2""".split()
```


```python
def resaltar(x):
    return 'font-weight: bold' if x in [0.253, 0.321, 0.31] else ''
```


```python
s = df[3].reset_index().style.applymap(resaltar)
```


```python
s1 = s.format({'promed-1995':'{:.3f}', 'promed-1996':'{:.3f}',
         'Combinado-2':'{:.3f}'})
```


```python
s1.set_properties(**{'text-align': 'center'})
```




<style type="text/css">
#T_a7cb9_row0_col0, #T_a7cb9_row0_col1, #T_a7cb9_row0_col2, #T_a7cb9_row0_col3, #T_a7cb9_row0_col4, #T_a7cb9_row0_col5, #T_a7cb9_row0_col6, #T_a7cb9_row1_col0, #T_a7cb9_row1_col1, #T_a7cb9_row1_col2, #T_a7cb9_row1_col4, #T_a7cb9_row1_col6, #T_a7cb9_row1_col7 {
  text-align: center;
}
#T_a7cb9_row0_col7, #T_a7cb9_row1_col3, #T_a7cb9_row1_col5 {
  font-weight: bold;
  text-align: center;
}
</style>
<table id="T_a7cb9">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_a7cb9_level0_col0" class="col_heading level0 col0" >index</th>
      <th id="T_a7cb9_level0_col1" class="col_heading level0 col1" >Nombre</th>
      <th id="T_a7cb9_level0_col2" class="col_heading level0 col2" >hitsVsTurnos-1995</th>
      <th id="T_a7cb9_level0_col3" class="col_heading level0 col3" >promed-1995</th>
      <th id="T_a7cb9_level0_col4" class="col_heading level0 col4" >hitsVsTurnos-1996</th>
      <th id="T_a7cb9_level0_col5" class="col_heading level0 col5" >promed-1996</th>
      <th id="T_a7cb9_level0_col6" class="col_heading level0 col6" >Combinado-1</th>
      <th id="T_a7cb9_level0_col7" class="col_heading level0 col7" >Combinado-2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_a7cb9_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_a7cb9_row0_col0" class="data row0 col0" >0</td>
      <td id="T_a7cb9_row0_col1" class="data row0 col1" >Derek Jeter</td>
      <td id="T_a7cb9_row0_col2" class="data row0 col2" >12/48</td>
      <td id="T_a7cb9_row0_col3" class="data row0 col3" >0.250</td>
      <td id="T_a7cb9_row0_col4" class="data row0 col4" >183/582</td>
      <td id="T_a7cb9_row0_col5" class="data row0 col5" >0.314</td>
      <td id="T_a7cb9_row0_col6" class="data row0 col6" >195/630</td>
      <td id="T_a7cb9_row0_col7" class="data row0 col7" >0.310</td>
    </tr>
    <tr>
      <th id="T_a7cb9_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_a7cb9_row1_col0" class="data row1 col0" >1</td>
      <td id="T_a7cb9_row1_col1" class="data row1 col1" >David Justice</td>
      <td id="T_a7cb9_row1_col2" class="data row1 col2" >104/411</td>
      <td id="T_a7cb9_row1_col3" class="data row1 col3" >0.253</td>
      <td id="T_a7cb9_row1_col4" class="data row1 col4" >45/140</td>
      <td id="T_a7cb9_row1_col5" class="data row1 col5" >0.321</td>
      <td id="T_a7cb9_row1_col6" class="data row1 col6" >149/551</td>
      <td id="T_a7cb9_row1_col7" class="data row1 col7" >0.270</td>
    </tr>
  </tbody>
</table>




La columna 'hitsVsTurnos' presenta el cociente entre el número de <i>hits</i> y los turnos al bate; las columnas portando el nombre 'promed', se refieren al promedio para el año en cuestión; mientras que las columnas con el prefijo 'Combinado', se refieren a los valores agregados para ambos años.

Se observa que, en los años 1995-1996,  <i>David Justice (ATL)</i> , tuvo mayor promedio de bateo que <i>Derek Jeter (NY)</i>; sin embargo, al combinar los promedios de las dos temporadas la relación se invierte.

La ocurrencia, en este contexto, se debe a la diferencia considerable de turnos al bate entre ambos jugadores, en esas dos temporadas. Se dice también que este fenómeno se observa al menos una vez cada temporada.

#### Regresión

<i>Fatalities</i>, es un conjunto de datos contenido en el paquete <i>AER</i> del <i>software R</i>. Allí se reporta la mortalidad en accidentes de transito en los EEUU, durante los años 1982-1988.

En el algunos ejemplos se le utiliza para examinar la relación entre el impuesto a bebidas alcohólicas y la mortalidad en accidentes de transito.


```python
import statsmodels.api as sm
import statsmodels.formula.api as smf
from linearmodels import PanelOLS
```

Con el modulo <i>statsmodels</i>, es posible obtener casi cualquier <i>dataset</i> de R.


```python
fatal = sm.datasets.get_rdataset('Fatalities',"AER")
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
      <th>income</th>
      <th>emppop</th>
      <th>beertax</th>
      <th>baptist</th>
      <th>mormon</th>
      <th>drinkage</th>
      <th>...</th>
      <th>nfatal2124</th>
      <th>afatal</th>
      <th>pop</th>
      <th>pop1517</th>
      <th>pop1820</th>
      <th>pop2124</th>
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
      <td>10544.152</td>
      <td>50.692</td>
      <td>1.539</td>
      <td>30.356</td>
      <td>0.328</td>
      <td>19.0</td>
      <td>...</td>
      <td>32</td>
      <td>309.438</td>
      <td>3.942e+06</td>
      <td>208999.594</td>
      <td>221553.438</td>
      <td>290000.062</td>
      <td>28516.0</td>
      <td>9.7</td>
      <td>57.8</td>
      <td>-0.022</td>
    </tr>
    <tr>
      <th>1</th>
      <td>al</td>
      <td>1983</td>
      <td>1.36</td>
      <td>13.7</td>
      <td>10732.798</td>
      <td>52.147</td>
      <td>1.789</td>
      <td>30.334</td>
      <td>0.343</td>
      <td>19.0</td>
      <td>...</td>
      <td>35</td>
      <td>341.834</td>
      <td>3.960e+06</td>
      <td>202000.078</td>
      <td>219125.469</td>
      <td>290000.156</td>
      <td>31032.0</td>
      <td>9.6</td>
      <td>57.9</td>
      <td>0.047</td>
    </tr>
    <tr>
      <th>2</th>
      <td>al</td>
      <td>1984</td>
      <td>1.32</td>
      <td>11.1</td>
      <td>11108.791</td>
      <td>54.168</td>
      <td>1.714</td>
      <td>30.312</td>
      <td>0.359</td>
      <td>19.0</td>
      <td>...</td>
      <td>34</td>
      <td>304.872</td>
      <td>3.989e+06</td>
      <td>196999.969</td>
      <td>216724.094</td>
      <td>288000.156</td>
      <td>32961.0</td>
      <td>7.5</td>
      <td>59.5</td>
      <td>0.063</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 34 columns</p>
</div>




```python
df = fatal.data
```


```python
df['mortalidad'] = df.fatal / (df['pop'] * 1000)
```

Si se observa la relación marginal entre el impuesto y la incidencia de accidentes fatales.


```python
res = smf.ols('mortalidad ~ beertax', data = df).fit()
```


```python
res.params
```




    Intercept    1.853e-07
    beertax      3.646e-08
    dtype: float64



Obtenémos un resultado, según el cual, esa relación es positiva; es decir, a medida que se incrementa el impuesto, también se incrementan los accidentes fatales. Contrario a lo que se podría anticipar.

Sin embargo, utilizando un modelo de efectos fijos, es decir observando la asociación condicionada por las diferencias entre estado y tiempo.


```python
data = df.set_index(['state', 'year'])
```


```python
mod = PanelOLS(data.mortalidad, data['beertax'], entity_effects = True)
```


```python
res = mod.fit(cov_type='clustered', cluster_entity=True)
```


```python
res.params
```




    beertax   -6.559e-08
    Name: parameter, dtype: float64



La relación se invierte; siendo coeficientes significativos al 95%.

En estos casos puntuales quiza resulte sencillo dilucidar cuál es la fuente de distorsión y cuál es la asociación correcta.

El asunto es que no siempre existe claridad en lo referente a cuando es es la asociación marginal o la condicional, la que debe considerarse espuria. Existen ejemplos, para uno y otro caso, al parecer suele ser más frecuente considerar como espuria a la marginal.

De hecho, si se encontrará que el comportamiento de las variables en la muestra refleja fielmente el de la población de la que fueron extraidas, tendría que concluirse que ambas asociaciones, tanto la marginal como la condicional, son correctas.

En muchos casos es la información contextual disponible, acerca del asunto que se estudia, lo que ofrece una orientación para tomar una decisión al respecto. 
