---
layout: post
title:  "Datos ficticios con Paradoja de Asociación"
date:   2023-08-15 10:45:55 -0400

---

##  <i>Paradoja de Simpson</i>

En las líneas que siguen se bosqueja una función para generar variables, cuya asociación presente un comportamiento contradictorio, similar al que se observa entre datos que muestran una reversion en su relación, <i>(Paradoja de Simpson)</i>.

En este caso, los datos que genera la función son contínuos. 

La paradoja de Simpson, es un fenómeno en el que, entre dos variables, emerge una asociación, que desaparece o se revierte al estas ser subdivididas en grupos


```python
import pandas as pd
import numpy as np
import plotly.graph_objects as go
from scipy import stats
from IPython.display import Image
```

## <i>Función</i>


```python
def generAsociacion(n_obs = 100, pendiente = -1.5):
    
    # definiendo algunas constantes
    n = n_obs
    
    coeficientes = np.repeat(pendiente, 3)
    
    medias = [5, 10, 15] # medias grupales
    
    dispersX = np.repeat(2, 3)
    
    disperE = np.repeat(3, 3)
    
    nivel = 20 # distancia de nube de obs.
    
    nombres = 'Grupo'
    
    Nro = np.linspace(1, len(medias), 3)
    
    grupos = [nombres + str(int(i)) for i in Nro]
    
    contenedor = []
    
    if pendiente > 0:
        w = Nro[::-1]
        k = -1
        
    else:
        w = Nro
        k = 1
    
    # semilla para nros pseudo-aleatorios
    rng = np.random.default_rng(seed=1000)
    
    # diccionario para crear dataframes
    for i in range(len(Nro)):
        
        dct = {'NroGrupo': np.repeat(Nro[i], n),
               
               'Grupos': np.repeat(grupos[i], n),
           
               'nivel': np.repeat(w[i] * nivel, n),
           
               'X': rng.normal(loc= medias[i], scale=dispersX[i],
                           size=n),
           
               'Errs': rng.normal(loc=0, scale=disperE[i], size=n),
              }
    
        dct['y'] = dct['nivel'] + coeficientes[i] * dct['X'] + dct['Errs']

        dfx = pd.DataFrame(dct)

        contenedor.append(dfx)
        
    df = pd.concat(contenedor)
    
    # estimación marginal
    
    y_est = stats.linregress(df['X'], df['y'])
    
    Q = y_est.slope
    
    df['y_est'] = y_est.intercept + Q * df['X']
    
    
    # estimaciones por grupo
    
    dfg = df.groupby('Grupos')
    
    listadf = []
    
    for _, grupo in dfg:
        
        y_reg_g = stats.linregress(grupo['X'], grupo['y'])
        
        grupo['y_reg_g'] = y_reg_g.intercept + y_reg_g.slope * grupo['X']
        
        listadf.append(grupo)
        
    dfn = pd.concat(listadf)
    
    return dfn
```


```python
df = generAsociacion(pendiente= 2)
```


```python
df.sample(10)
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
      <th>NroGrupo</th>
      <th>Grupos</th>
      <th>nivel</th>
      <th>X</th>
      <th>Errs</th>
      <th>y</th>
      <th>y_est</th>
      <th>y_reg_g</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>35</th>
      <td>3.0</td>
      <td>Grupo3</td>
      <td>20.0</td>
      <td>18.519156</td>
      <td>-3.576909</td>
      <td>53.461403</td>
      <td>47.847475</td>
      <td>55.895996</td>
    </tr>
    <tr>
      <th>37</th>
      <td>2.0</td>
      <td>Grupo2</td>
      <td>40.0</td>
      <td>12.317913</td>
      <td>-1.122405</td>
      <td>63.513421</td>
      <td>56.658483</td>
      <td>65.048600</td>
    </tr>
    <tr>
      <th>63</th>
      <td>1.0</td>
      <td>Grupo1</td>
      <td>60.0</td>
      <td>3.201027</td>
      <td>-1.595958</td>
      <td>64.806097</td>
      <td>69.612170</td>
      <td>66.542964</td>
    </tr>
    <tr>
      <th>74</th>
      <td>3.0</td>
      <td>Grupo3</td>
      <td>20.0</td>
      <td>14.190508</td>
      <td>-1.483739</td>
      <td>46.897278</td>
      <td>53.997815</td>
      <td>48.120927</td>
    </tr>
    <tr>
      <th>68</th>
      <td>3.0</td>
      <td>Grupo3</td>
      <td>20.0</td>
      <td>13.260128</td>
      <td>-2.222704</td>
      <td>44.297552</td>
      <td>55.319742</td>
      <td>46.449788</td>
    </tr>
    <tr>
      <th>79</th>
      <td>1.0</td>
      <td>Grupo1</td>
      <td>60.0</td>
      <td>4.775196</td>
      <td>-1.098441</td>
      <td>68.451952</td>
      <td>67.375519</td>
      <td>69.590714</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2.0</td>
      <td>Grupo2</td>
      <td>40.0</td>
      <td>8.562891</td>
      <td>5.018289</td>
      <td>62.144072</td>
      <td>61.993790</td>
      <td>57.323985</td>
    </tr>
    <tr>
      <th>91</th>
      <td>2.0</td>
      <td>Grupo2</td>
      <td>40.0</td>
      <td>10.183418</td>
      <td>-6.467369</td>
      <td>53.899468</td>
      <td>59.691271</td>
      <td>60.657640</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3.0</td>
      <td>Grupo3</td>
      <td>20.0</td>
      <td>13.540184</td>
      <td>-1.365096</td>
      <td>45.715272</td>
      <td>54.921825</td>
      <td>46.952822</td>
    </tr>
    <tr>
      <th>90</th>
      <td>3.0</td>
      <td>Grupo3</td>
      <td>20.0</td>
      <td>13.247176</td>
      <td>-1.921617</td>
      <td>44.572735</td>
      <td>55.338144</td>
      <td>46.426524</td>
    </tr>
  </tbody>
</table>
</div>




```python
gr = df.groupby('Grupos')
```


```python
import plotly.graph_objects as go
```


```python
fig = go.Figure(layout=go.Layout(
    paper_bgcolor= 'rgba(0,0,0,0)',
    plot_bgcolor= 'rgba(0,0,0,0)',
    width = 820,
    height = 620
)
                
               )

fig.add_trace(go.Scatter(x = gr.get_group('Grupo1')['X'],
                         y = gr.get_group('Grupo1')['y'],
                         mode='markers',
                         marker = dict(color='rgba(0, 0, 255, 0.7)',
                              size=10,
                              line=dict(
                                  color='#000000',
                                  width=2))
                        ))


fig.add_trace(go.Scatter(x = gr.get_group('Grupo1')['X'],
                         y = gr.get_group('Grupo1')['y_reg_g'],
                         mode='lines', line_color='rgba(0, 0, 255, 1)'))


fig.add_trace(go.Scatter(x = gr.get_group('Grupo2')['X'],
                         y = gr.get_group('Grupo2')['y'],
                         mode='markers',
                marker = dict(color='rgba(0, 255, 0, 0.7)',
                              size=10,
                              line=dict(
                                  color='#000000',
                                  width=2))))


fig.add_trace(go.Scatter(x = gr.get_group('Grupo2')['X'],
                         y = gr.get_group('Grupo2')['y_reg_g'],
                         mode='lines', line_color='rgba(0, 255, 0, 1)'))


fig.add_trace(go.Scatter(x = gr.get_group('Grupo3')['X'],
                         y = gr.get_group('Grupo3')['y'],
                         mode='markers',
                marker = dict(color='rgba(255, 0, 0, 0.7)',
                              size=10,
                              line=dict(
                                  color='#000000',
                                  width=2)
                              ))
             )


fig.add_trace(go.Scatter(x = gr.get_group('Grupo3')['X'],
                         y = gr.get_group('Grupo3')['y_reg_g'],
                         mode='lines',
                         line_color='rgba(255, 0, 0, 1)'))



fig.add_trace(go.Scatter(x = df['X'], y = df['y_est'],
                         mode='lines',
                         line = {'width':3},
                         line_color="rgb(255, 204, 0, 1)"))


fig.update(layout_showlegend=False)

fig.update_xaxes(showline=True, linecolor= '#000000',
                ticks='outside', showgrid=True, gridcolor='lightgray'
)

fig.update_yaxes(showline=True, linecolor='#000000',
        ticks='outside', showgrid=True, gridcolor='lightgray')

# fig.show()
fig.write_image('chartScatter.png')
Image('chartScatter.png')
```




    

<div>
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/08/15/assets/output_10_0.png?raw=true"> 
</div>
    


