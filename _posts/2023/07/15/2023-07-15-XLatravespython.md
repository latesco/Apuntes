# <i>Excel a través de Python</i>

##  <i>Python</i>, <i>Openpyxl</i>

<i>Openpyxl</i>, es una librería de <i>Python</i> para lectura y escritura de archivos en <i>Excel</i>— no es la única, pero si pionera —; tiene capacidades que ofrecen la posibilidad de convertir actividades repetitivas y tediosas en un proceso mucho más simple y rápido.

Se trata de operaciones que pueden ejecutarse directamente sobre la hoja de cálculo, y que incluso podrían resultar triviales para quienes sean usuarios habituales de  Excel; pero que de ser necesario repetirlas muchas veces, sobre distintas hojas o libros de excel, pueden tornarse muy tediosas y con alta propensión a errores.

Existe <i>VBA</i> <i>(macros)</i>, por supuesto, pero <i>Python</i> — a través de varios de sus paquetes, <i>openpyxl</i> entre ellos —  es visto como una buena alternativa (y quiza, posible sustituto); sobre todo para  quienes realicen en Excel tareas que involucren <i>data cleaning</i> (trad. literal limpieza de datos) y reportes.

Las funciones disponibles de este paquete son muchas y variadas. Vale la pena consultar la documentación que es bastante explicita sobre su instalación y uso: https://openpyxl.readthedocs.io/en/stable/tutorial.html

### Ejemplo

A modo de ilustración de lo dicho arriba. Considerémos los siguientes datos, guardados en un libro de excel llamado 'Ejemplo01.xlsx'.


<div>
 
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/07/15/assets/output_10_0.png" alt=""> 
</div>

Son datos ficticios de unas 'ventas mensuales'. Suponiendo que sobre un archivo como este deben efectuarse algunos cálculos: total columna 'B' y agregar otra columna en 'C' con los cambios porcentuales; una tabla para el conteo de meses con cambio porcentual negativo; y además añadir algunos gráficos y formato a la hoja (encabezado: titulo, colores, etc).

Si las operaciones requeridas en el parráfo anterior, han sido previamente agrupadas en funciones o clases. el procedimiento se reduce simplemente a la ejecución de los siguientes comandos.

Importamos el libro de excel, escr:


```python
libro = formato_calculo('Ejemplo01.xlsx')
```

Procesamos los cálculos: el total y la columna de cambios porcentuales


```python
libro.total()
```




    <Worksheet "Despues">



Añadimos un titulo a la hoja y le damos un color a los encabezados


```python
libro.formatos()
```




    <Worksheet "Despues">



Ahora agregamos dos gráficos:


```python
libro.grafico_linea()
```




    <Worksheet "Despues">




```python
libro.grafico_barra()
```




    <Worksheet "Despues">



Y guardámos los cambios, puede ser en otro libro o el mismo libro excel, actualmente abierto.


```python
libro.libro.save('Ejemplo01_Procesado.xlsx')
```

El <i>gif</i> muestra la hoja de cálculo 'antes', con los datos originales, y la hoja 'después' con los resultados de las operaciones hechas arriba.

<div>
 
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/07/15/assets/xl_final.gif?raw=true"/>  
</div>

Como vemos, puede ahorrarse bastante tiempo, si las operaciones a realizar pueden ser preparadas por adelantado; esto no siempre es posible, pero en muchos casos si lo es.

En las líneas siguientes se  describe la codificación prevía necesaria para crear la clase <i>formato_calculo</i>, usada en las celdas de arriba.

### <i>Algunas funciones</i>:

Usarémos los siguientes datos para mostrar el uso de algunas funciones, valores distintos, pero lo importante es que sea la misma estructura.


```python
datos = [['Mes', 'Monto'], ['Ene', 411], ['Feb', 536], ['Mar', 467], ['Abr', 460],
 ['May', 370], ['Jun', 385],['Jul', 478], ['Ago', 493], ['Sep', 615], ['Oct', 620],
        ['Nov', 630], ['Dic', 650]]
```

Para crear un libro de excel se procede del modo siguiente.


```python
from openpyxl import Workbook
```

Usando este modulo <i>Workbook</i>, creamos un libro que puede trabajarse desde el editor de texto.


```python
lib = Workbook()
```

Para ingresar los datos, en este caso, conviene usar usar un <i>bucle</i> <i>(for loop)</i>, sobre un objeto que representa la hoja de calculo activa en el libro excel.


```python
ho = lib.active
```


```python
for f in datos:
    ho.append(f)
```

Para guardar el libro.


```python
lib.save('Ejemplo02.xlsx')
```

Si, en lugar de crear un libro Excel, se trata mas bien de importarlo, el procedimiento es:


```python
from openpyxl import load_workbook
```


```python
libro = load_workbook(filename = 'Ejemplo02.xlsx')
```

En el objeto 'hoja' guardamos la hoja de cálculo activa del libro cargado y le damos por nombre 'Ventas'.


```python
hoja = libro.active
hoja.title = 'Ventas'
```

Isertamos dos filas, en la parte superior de la hoja, y añadimos un texto a la celda 'A1' que servirá de titulo para la hoja de cálculo.


```python
hoja.insert_rows(idx=1, amount=2)
        
hoja['A1'] = "Montos Mensuales"
        
hoja.merge_cells(start_row=1, 
                  start_column=1, 
                  end_row=2, 
                  end_column=3)
```

Calculamos el total de la columna 'B' y lo ingresamos en la celda 'B17'.


```python
hoja['A17'] = 'Total'

hoja['B17'] = "=SUM(B4:B15)" # fórmula

```

Añadimos otra columna, correspondiente a la letra 'C' de la hoja, que contendrá los cambios porcentuales entre las cantidades presentes en la columna 'B' de la hoja de cálculo.


```python
maxfila = hoja.max_row - 1

hoja['C3'] = 'Pct'

for i in range(4, maxfila):
    if i == 4:
        hoja[f"C{i}"] = 0
    else:
        hoja[f"C{i}"] = f"=ROUND((B{i} / B{i - 1} - 1) * 100, 2)"
```

Ahora creamos una pequeña tabla que contendrá la cantidad de meses con cambios porcentuales negativos y positivos.


```python
hoja['A21'] = "Conteos"

hoja['A22'] = 'Positivos'

hoja['A23'] = 'Negativos'

hoja['A24'] = 'Ceros'

```


```python
hoja['B22'] = '=COUNTIF($C$4:$C$15,">0")'

hoja['B23'] = '=COUNTIF($C$4:$C$15,"<0")'

hoja['B24'] = '=COUNTIF($C$4:$C$15,"0")'
```


```python
libro.save('Ejemplo02_Proc.xlsx')
```

Al guardar los cambios y abrir el libro en Excel.


    
<div>
 
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/07/15/assets/output_54_0.png" alt=""> 
</div>


Con esto ya los resultados y las operaciones de cálculo de porcentaje y suma, que nos propusimos hacer, ya se encuentran en el libro. Ahora si es necesario darĺe algun formato y añadir colores se puede proceder como sigue.

 ### Formatos

Para dar el formato a la hoja, necesitarémos algunos modulos extra de la librería <i>Openpyxl</i>


```python
from openpyxl.utils import get_column_letter

from openpyxl.styles import (
    Font, Color, Alignment,
    Border, Side,
    PatternFill,
    NamedStyle
    )
```

Si se decide deshechar los bordes de celdas.


```python
hoja.sheet_view.showGridLines = False
```

Cada uno de los modulos importados arriba sirve para controlar alguna caraćterística de la hoja excel, que deseémos modificar: <i>Font</i> tiene que ver con el tamaño color y tipo de fuente; <i>Border</i>, para cambiar los bordes de las celdas; <i>PatternFill</i>, permite cambiar el color de la celda; y <i>NamedStyle</i>, permite agrupar varias características como un estilo, para luego aplicarlo en conjunto a una celda en especifíco.

Para dar color a las celdas y designar el tamaño de fuente.


```python
enc = NamedStyle('enc')
```


```python
enc.border = Border(bottom=Side(border_style='double', color='ffffff'), 
                      top=Side(border_style='double', color='ffffff'))
```


```python
enc.font = Font(bold=True, size=15, color='ffffff')


enc.alignment = Alignment(horizontal='center',
                                    vertical= 'center')
        
enc.fill = PatternFill(fgColor = "003366",
                             fill_type='solid')
```

El siguiente <i>loop</i> le asigna las características creadas arriba a los encabezados de las 3 columnas.


```python
for celda in hoja[3]:
    celda.style = enc
```


```python
for celda in hoja[17]:
    
    celda.style = enc
```

Reduciendo la altura de la fila 16.


```python
hoja.row_dimensions[16].height = 7
```

Incrementando la altura de las filas 3 y 17.


```python
hoja.row_dimensions[3].height = 22
hoja.row_dimensions[17].height = 22
```

A continuación agregamos formato al tituo de la hoja de cálculo.


```python
enc2 =  NamedStyle('enc2')

enc2.border = Border(bottom=Side(border_style='thick', color='ffffff'), 
                      top=Side(border_style='thick', color='ffffff'))

enc2.font = Font(bold=True, size=18, color='ffffff')

enc2.alignment = Alignment(horizontal='center',
                                    vertical= 'center')
        
enc2.fill = PatternFill(fgColor = "003366",
                             fill_type='solid')


```

El <i>bucle</i> le asigna las características arriba especificadas las celdas de interés, las correspondientes a las 2 primeras filas, cuyas celdas fueron combinadas más arriba.


```python
for i in range(1, 3):
    
    for celda in hoja[i]:
        
        celda.style = enc2
```

Aumentamos el ancho de columnas; <i>get_column_letter</i>, es una función que al darle un número retorna la letra de la columna correspondiente a ese número; resulta útil en <i>bucles</i>.


```python
for col in range(1, hoja.max_column + 1):
    
    idx_col = get_column_letter(col)
    
    hoja.column_dimensions[idx_col].width = 14
```

El siguiente procedimiento cambiará la alineación, aumentará la fuente y le dará color distinto a las filas impares. 


```python
fila = NamedStyle('ffila')

fila.alignment = Alignment(horizontal='center',
                            vertical='center')
        
fila.font = Font(size=13)
```


```python
filapar = PatternFill(fgColor = 'ecf7f9',
                   fill_type='solid')
```


```python
for j in range(4, 16):
    
    for celda in hoja[j]:
        
        if j % 2 != 0:
            
            celda.style = fila
            
            celda.fill = filapar
            
        else:
            
            celda.style = fila
```

Lo siguiente es para formatear las filas de la otra tabla para conteo de positivos y negativos.


```python
hoja.merge_cells(start_row=21, 
                 start_column=1, 
                 end_row=21, 
                 end_column=2)
```


```python
for celda in hoja[21]:
    
    celda.style = enc
```


```python
for j in range(22, 25):
    
    for celda in hoja[j]:
        
        if j % 2 != 0:
            
            celda.style = fila
            
            celda.fill = filapar
            
        else:
            
            celda.style = fila
```


```python
hoja.row_dimensions[21].height = 22
```

Guardámos los cambios


```python
libro.save('Ejemplo02_Proc.xlsx')
```

       
<div>
 
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/07/15/assets/output_90_0.png" alt=""> 
</div>


### Gráficos

Para añadir gráficos, existen varias alternativas: linea, barras, tortas, etc. Para cada una existe un modulo por separado, y la documentación de <i>Openpyxl</i> ofrece las especificaciones necesarias.<br>

En este caso usarémos <i>LineChart</i> y <i>BarChart</i>, pues se trata de gráficos de linea y barra. El modulo llamado <i>Reference</i>, se usa para contener los datos.


```python
from openpyxl.chart import BarChart, LineChart, Reference
```


```python
linea = LineChart()

linea.type = "line"
linea.style = 10
linea.title = "Montos Mensuales"
linea.y_axis.title = '$'
linea.x_axis.title = 'Mes'

data = Reference(worksheet=hoja,

                 min_row=3,

                 max_row=15,

                 min_col=1,

                 max_col=2)
```


```python
linea.add_data(data, from_rows=False,
                 titles_from_data=True
              )

hoja.add_chart(linea, "E3")
```


```python
barra = BarChart()
barra.type = "col"
barra.style = 9
barra.title = "Positivos y negativos"
barra.y_axis.title = 'Conteo'
barra.x_axis.title = 'Categoria'


data1 = Reference(worksheet=hoja,

                 min_row=21,

                 max_row=24,

                 min_col=1,

                 max_col=2)


barra.add_data(data1, titles_from_data=True)

hoja.add_chart(barra, "E22")

```


```python
libro.save('Ejemplo02_Proc.xlsx')
```

Como vemos es posible controlar todo tipo de características de la hoja de cálculo con este paquete.

Sin embargo, un modo más ventajoso consiste en agrupar estos comandos en una función o una clase que agrupe varias funciones, de modo de poder aplicar estos comandos de forma reiterada y sin tener que escribirlos de nuevo. De nuevo, su valor consiste en abreviar tareas que se cumplen de modo repetitivo.

Si se tiene un archivo con estos datos:


```python
from openpyxl import Workbook
from openpyxl import load_workbook
from openpyxl.chart import BarChart, LineChart, Reference
```


```python
from openpyxl.utils import get_column_letter
from openpyxl.styles import (
    Font, Color, Alignment,
    Border, Side,
    PatternFill,
    NamedStyle
    )
```


```python
class formato_calculo:
    
    def __init__(self, ruta_archivo):
        
        self.libro = load_workbook(filename=ruta_archivo)
        
        # definiendo colores, bordes de celdas, etc.
        
        enc = NamedStyle('enc_col')
        
        enc.border = Border(bottom=Side(border_style='double',
                                        color='ffffff'), 
                            top=Side(border_style='double',
                                     color='ffffff'))
        
        enc.font = Font(bold=True, size=15, color='ffffff')
        
        enc.alignment = Alignment(horizontal='center',
                                    vertical= 'center')
        
        enc.fill = PatternFill(fgColor = "003366",
                             fill_type='solid')
        
        self.libro.add_named_style(enc)
        
        # encabezados de columnas
        
        enc2 =  NamedStyle('form_titulo')
        
        enc2.border = Border(bottom=Side(border_style='thick',
                                         color='ffffff'), 
                             top=Side(border_style='thick',
                                      color='ffffff'))
        
        enc2.font = Font(bold=True, size=18, color='ffffff')
        
        enc2.alignment = Alignment(horizontal='center',
                                    vertical= 'center')
        
        enc2.fill = PatternFill(fgColor = "003366",
                             fill_type='solid')
        
        
        self.libro.add_named_style(enc2)
        
        # formato del resto de filas
        
        fl = NamedStyle('form_fila')
        
        fl.alignment = Alignment(horizontal='center',
                                   
                                   vertical='center')
        
        fl.font = Font(size=13)
        
        self.libro.add_named_style(fl)
        
        
    def total(self):
        
        
        pag = self.libro.active
        
        pag.title = 'Antes'
        
        hoja = self.libro.copy_worksheet(pag)
        
        hoja.insert_rows(idx=1, amount=2)
        
        hoja['A1'] = "Montos Mensuales"
        
        hoja.merge_cells(start_row=1, 
                         start_column=1, 
                         end_row=2, 
                         end_column=3)
        
        maxfila = hoja.max_row
        
        minfila = hoja.min_row
        
        hoja.title = 'Despues'
        
        loc_1 = str(maxfila+2) # localización celda
        
        loc_2 = str(minfila+2) # localización celda
        
        hoja['A'+loc_1] = 'Total'
        
        hoja['B'+loc_1] = "=SUM(B4:B15)"
        
        hoja['C'+loc_2] = 'C_Pct' # nombre columna cambio porcentual
        
        # cálculo cambio porcentual mes a mes columna C
        min_nro_fila = minfila + 3
        
        max_nro_fila = hoja.max_row - 1 # el rango no incluye el límite superior
        
        for i in range(min_nro_fila, max_nro_fila):
            
            if i == 4:
                
                hoja[f"C{i}"] = 0
                
            else:
                
                hoja[f"C{i}"] = f"=ROUND((B{i} / B{i - 1} - 1) * 100, 2)"
                
        # segunda tabla
        
        ix01 = hoja.min_row + 3
        ix02 = hoja.max_row - 2
        nf = hoja.max_row + 4
        
        hoja['A'+str(nf)] = "Conteos"
        
        hoja.merge_cells(start_row=nf, 
                 start_column=1, 
                 end_row=nf, 
                 end_column=2)
        
        hoja['A'+str(nf+1)] = 'Positivos'
        hoja['A'+str(nf+2)] = 'Negativos'
        hoja['A'+str(nf+3)] = 'Ceros'
        
        hoja['B'+str(nf+1)] =  f'=COUNTIF($C${ix01}:$C${ix02},">0")'
        hoja['B'+str(nf+2)] = f'=COUNTIF($C${ix01}:$C${ix02},"<0")'
        hoja['B'+str(nf+3)] = f'=COUNTIF($C${ix01}:$C${ix02},"0")'
        
        return hoja
    
    def formatos(self):
        
        # hoja = self.libro.active
        
        hoja = self.libro['Despues']
        
        hoja.sheet_view.showGridLines = False # bordes de línea
        
        hoja.row_dimensions[16].height = 7
        
        hoja.row_dimensions[3].height = 22
        
        hoja.row_dimensions[17].height = 22
        
        hoja.row_dimensions[21].height = 22
        
        # formato a titulo de la hoja 
        
        for i in range(1, 3):
            
            for celda in hoja[i]:
                
                celda.style = 'form_titulo'
                
        # formato a los encabezados de columnas
        
        for celda in hoja[3]:
            
            celda.style = 'enc_col'
            
        for celda in hoja[17]:
            
            celda.style = 'enc_col'
            
        # formato para resto de filas
        idx = hoja.min_row + 3
        idx2 = idx + 12
        
        filapar = PatternFill(fgColor = 'ecf7f9',
                   fill_type='solid')
        
        for j in range(idx, idx2):
            
            for celda in hoja[j]:
                
                if j % 2 != 0:
                    
                    celda.style = 'form_fila'
                    
                    celda.fill = filapar
                    
                else:
                    
                    celda.style = 'form_fila'
                    
        # ancho de columnas
                    
        for col in range(1, hoja.max_column + 1):
            
            idx_col = get_column_letter(col)
            
            hoja.column_dimensions[idx_col].width = 14
            
        # segunda tabla
        for celda in hoja[21]:
            
            celda.style = 'enc_col'
            
        ## filas de 2nda tabla
        
        for j in range(22, 25):
            
            for celda in hoja[j]:
                
                if j % 2 != 0:
                    
                    celda.style = 'form_fila'
                    
                    celda.fill = filapar
                    
                else:
                    
                    celda.style = 'form_fila'
                    
        return hoja
    
    def grafico_linea(self,
                      ejex="Mes",
                      ejey="$",
                      titulo="Montos Mensuales"):
        
        # hoja = self.libro.active
        hoja = self.libro['Despues']
        
        linea = LineChart()
        
        linea.type = "line"
        
        linea.style = 10
        
        linea.title = titulo
        
        linea.y_axis.title = ejey
        
        linea.x_axis.title = ejex
        
        data = Reference(worksheet=hoja,
                         
                         min_row=3,
                         
                         max_row=15,
                         
                         min_col=1,
                         
                         max_col=2)
        
        linea.add_data(data, from_rows=False,
                       
                       titles_from_data=True
                      )
        
        hoja.add_chart(linea, "E3")
        
        return hoja
    
    def grafico_barra(self,
                      ejex="Categoria",
                      
                      ejey="Conteo",
                      
                      titulo="Cambio % Positivo-Negativo"):
        
        # hoja = self.libro.active
        hoja = self.libro['Despues']
        
        barra = BarChart()
        
        barra.type = "col"
        
        barra.style = 9
        
        barra.title = "Positivos y negativos"
        
        barra.y_axis.title = 'Conteo'
        
        barra.x_axis.title = 'Categoria'
        
        data1 = Reference(worksheet=hoja,
                          
                          min_row=21,
                          
                          max_row=24,
                          
                          min_col=1,
                          
                          max_col=2)
        
        barra.add_data(data1, titles_from_data=True)
        
        hoja.add_chart(barra, "E22")
        
        return hoja

```

En conclusión, estos modulos y funciones pueden ser muy útiles para situaciones en las que puedan anticiparse el tipo de operaciones y formatos que serán requeridos, y sobre todo si se trata de tareas que deben ejecutarse de modo reiterado.
