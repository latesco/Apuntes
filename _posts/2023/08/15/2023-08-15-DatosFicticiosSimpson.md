---
layout: post
title:  "Generando Datos ficticios con Paradoja de Asociación"
date:   2023-08-15 10:45:55 -0400

---

##  <i>Paradoja de Simpson</i>

En las líneas que siguen presentan el bosquejo de función, para generar datos con un comportamiento que similar al de variables en las que se observa la Paradoja de Simpson.

Los datos son contínuos; la paradoja puede darse en datos cualitativos también, y el fenómeno ocurre cuando entre dos variables emerge una asociación, desaparece o se revierte al estas ser subdivididas en grupos.

Los datos ficticios product de la siguiente función emulan el comportamiento observado cuando la asociación se revierte.


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
      <th>45</th>
      <td>1.0</td>
      <td>Grupo1</td>
      <td>60.0</td>
      <td>7.467050</td>
      <td>0.134008</td>
      <td>75.068108</td>
      <td>63.550811</td>
      <td>74.802415</td>
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
      <th>61</th>
      <td>2.0</td>
      <td>Grupo2</td>
      <td>40.0</td>
      <td>10.713497</td>
      <td>-1.796158</td>
      <td>59.630836</td>
      <td>58.938111</td>
      <td>61.748087</td>
    </tr>
    <tr>
      <th>90</th>
      <td>2.0</td>
      <td>Grupo2</td>
      <td>40.0</td>
      <td>10.331006</td>
      <td>1.901065</td>
      <td>62.563077</td>
      <td>59.481571</td>
      <td>60.961250</td>
    </tr>
    <tr>
      <th>84</th>
      <td>3.0</td>
      <td>Grupo3</td>
      <td>20.0</td>
      <td>14.728116</td>
      <td>4.837080</td>
      <td>54.293311</td>
      <td>53.233958</td>
      <td>49.086571</td>
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
      <th>53</th>
      <td>2.0</td>
      <td>Grupo2</td>
      <td>40.0</td>
      <td>2.355506</td>
      <td>2.055529</td>
      <td>46.766541</td>
      <td>70.813526</td>
      <td>44.554509</td>
    </tr>
    <tr>
      <th>79</th>
      <td>3.0</td>
      <td>Grupo3</td>
      <td>20.0</td>
      <td>16.543172</td>
      <td>-1.701529</td>
      <td>51.384815</td>
      <td>50.655043</td>
      <td>52.346756</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2.0</td>
      <td>Grupo2</td>
      <td>40.0</td>
      <td>12.835404</td>
      <td>4.634769</td>
      <td>70.305577</td>
      <td>55.923209</td>
      <td>66.113152</td>
    </tr>
    <tr>
      <th>51</th>
      <td>2.0</td>
      <td>Grupo2</td>
      <td>40.0</td>
      <td>10.104172</td>
      <td>-0.556361</td>
      <td>59.651983</td>
      <td>59.803868</td>
      <td>60.494618</td>
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
    plot_bgcolor= 'rgba(0,0,0,0)')
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
                         mode='lines', line_color='rgba(255, 0, 0, 1)'))



fig.add_trace(go.Scatter(x = df['X'], y = df['y_est'],
              
              mode='lines', line_color="black"))


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