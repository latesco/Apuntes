---
layout: post
title:  "Automating an Excel Report"
date:   2023-07-05 12:15:25 -0400

---



The following lines show one way of generating an Excel report, through the use of Python. The report is about the montly sales of a store with establishments in several regions.

This is an example, in which we have four csv files (one for each region: west, east, north, and south). The report will consist on combining those 4 files into one, performing some calculations and produce a summary on: profit-margin, percent-change in profit, percent-change on sales, and obtaining the average of sales and profit by category and sub-categories.

For this we are using the packages <i>openpyxl</i> and <i>Pandas</i>.


```python
import pandas as pd
import os
from pathlib import Path 
import calendar
from datetime import date
```


```python
from IPython.display import Image 
```


```python
from openpyxl import Workbook, drawing
from openpyxl.utils import get_column_letter
from openpyxl import load_workbook
from openpyxl.formatting.rule import IconSet, FormatObject, DataBar
from openpyxl.formatting.rule import Rule
from openpyxl.styles import (
    Font, Color, Alignment,
    Border, Side,
    PatternFill,
    NamedStyle
    )
import plotly.express as px 
import plotly.io as pio
```

The path to the location of the files.


```python
p0 = Path('./data')
```

As was previously mentioned, the four <i>csv</i> files, have the same number of columns and data types, the only difference is that the data in each one of the files comes from different regions, so they have the same format; for example the dataset of the east region:

<div>
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/07/05/assets/output_10_0.png?raw=true" />
</div>
    



And this way would look each one of the other 3 files.

## Code

I am using two classes here, the first one <i>(procs)</i>  for procesing the datasets; the other one <i>(formatos)</i> is intended for applying formats and inserting charts.


```python
class procs:

    def __init__(self, path):
        
        self.path = path.glob('*.csv')
    
    def load_df(self):
        dfs = []
        
        for reg in self.path:
            file = pd.read_csv(reg)
            rn = os.path.basename(reg)
            region_name = rn.replace('.csv', '')
            file['Region'] = region_name
            dfs.append(file)
        df = pd.concat(dfs)
         
        # dataframe new features---------------------------------------- #
    
        df.columns = [e.replace(' ', '_') for e in df.columns]
        df['Order_Date'] = pd.to_datetime(df['Order_Date'], format='%m/%d/%Y')
        df['Ship_Date'] = pd.to_datetime(df['Ship_Date'], format='%m/%d/%Y')
        df['days_to_ship'] = (df['Ship_Date'] - df['Order_Date']).astype('timedelta64[ns]')
        df = df.assign(order_month = df['Order_Date'].dt.month,
               order_year = df['Order_Date'].dt.year)
        df['order_month'] = df['order_month'].apply(lambda x: calendar.month_abbr[x])
        
        return df

    def filt_df(self, date1):

        X = self.load_df()
    
        Z = X.copy()

        codes = pd.read_csv('./data/state-abbr/state_abbr.csv')

        date1 = pd.to_datetime(date1)

        delta = pd.to_timedelta(30, 'D')

        delta_i = pd.to_timedelta(60, 'D')

        delta_ii = pd.to_timedelta(52, 'W')

        date2 = date1 - delta

        date3 = date1 - delta_i

        date4 = date1 - delta_ii
    
        curr_df = Z.loc[(Z['Order_Date'].between(date2, date1))]

        past_df = Z.loc[(Z['Order_Date'].between(date3, date2))]

        ## Chart 1 ------------------------------------------ #

        prev_year_df = Z.loc[(Z['Order_Date'].between(date4, date1))]

        dx = prev_year_df.merge(codes, on='State', how='left')

        dx_gr = dx.groupby('Code')['Sales'].sum().reset_index()

        ref = date1.strftime('%b %Y')

        fig = px.choropleth(locations=dx_gr['Code'], 
                    locationmode="USA-states", color=dx_gr['Sales'],
                    color_continuous_scale="Viridis", 
                    scope="usa",
                    #title='Sales By State'
                    )
        fig.update_layout(
                        {
                            "paper_bgcolor": "rgba(0, 0, 0, 0)",
                            "plot_bgcolor": "rgba(0, 0, 0, 0)"
                        }
                    )
        fig.update_layout(
            title=dict(text=f"Sales By State, {ref}",
            font=dict(size=20)
            
            )
        )
        pio.write_image(fig, './data/map.png')

        ## Chart 2 -------------------------------------------------- #
        qc = (prev_year_df.set_index('Order_Date')
                .groupby([pd.Grouper(freq='M'), 'Segment'])['Profit']
                .mean().reset_index()
        )
        qc['Order_Date'] = qc['Order_Date'].dt.strftime('%b-%Y')#.dt.date



        qx = (prev_year_df.set_index('Order_Date')
                .groupby(pd.Grouper(freq='M'))['Sales']
                .mean().reset_index()
        )
        qx['Order_Date'] = qx['Order_Date'].dt.strftime('%b-%Y')#.dt.date

        fig = px.bar(qc, x='Order_Date', y='Profit', color='Segment',
                #title = 'Avg. Of Sales and Profit, Over The Last 12 Months'
                )
        fig.add_scatter(x=qx['Order_Date'], y=qx['Sales'],
                mode='markers+lines',
                name = 'Sales')
        fig.layout.paper_bgcolor = "rgba(0, 0, 0, 0)"
        fig.layout.plot_bgcolor = "rgba(0, 0, 0, 0)"
        fig.layout.legend.orientation = 'h'
        fig.layout.legend.yanchor = 'top'
        fig.layout.legend.xanchor = 'right'
        fig.layout.legend.y = 1.02
        fig.layout.legend.x = 1
        fig.layout.margin.pad = 12
        fig.update_yaxes(gridcolor = 'gray')
        fig.update_xaxes(type='category')
        fig.update_layout(
            title=dict(text='Avg. Of Sales and Profit, Over The Last 12 Months',
            font=dict(size=20)
            
            )
        )
        
        pio.write_image(fig, './data/bar.png')

        # Card values -------------------------------------------------#
        

        sales_pct_chg = round(((curr_df['Sales'].sum() / past_df['Sales'].sum()) - 1) * 100, 2)
    
        profit_margin_1 = round((curr_df['Profit'].sum() / curr_df['Sales'].sum()) * 100, 2)

        profit_margin_2 = round((past_df['Profit'].sum() / past_df['Sales'].sum()) * 100, 2)

        chg_profit = round(profit_margin_1 - profit_margin_2, 2)

        val_cards = pd.DataFrame({'profit_margin':  profit_margin_1, 
                        'profit_marg_change': chg_profit,
                        'sales_pct_chg': sales_pct_chg}, index=[1])
    
    
        return val_cards, curr_df, prev_year_df


```

The following lines of code will format the spreadsheet.


```python
class xl_report(procs):

    def __init__(self, path, date):
        super().__init__(path)
        self.path = path.glob('*.csv')
        self.date = date
        self.book = Workbook()

    def sheet_rep(self):

        datex = pd.to_datetime(self.date)
        period = calendar.month_abbr[datex.month]+ ' ' +str(datex.year)   

        cds_vals, df, pdf = self.filt_df(self.date)

        dc = (df.groupby(['Category','Sub-Category'])[['Profit', 'Sales']]
        .agg({'Sales': 'mean', 'Profit': 'mean'}))


        dc = dc.reset_index()

        cs = cds_vals.values

        dp = (df.set_index('Order_Date').groupby(pd.Grouper(freq='W'))['Profit']
                .mean()
                .reset_index()
        )

        dp['Order_Date'] = dp['Order_Date'].dt.date

        dp['Order_Date'] = dp['Order_Date'].astype('str')

       

        cs_names = ['Category', 'Sub\nCategory', 'Sales', 'Profit']

        sheet1 = self.book.active

        sheet1.title='report'

        # values for table ---------------------------------------------- #

        data = [cs_names] + dc.values.tolist()

        for rowy, row in enumerate(data, start=9):
            
            for colx, value in enumerate(row, start=2):
                
                sheet1.cell(column=colx, row=rowy, value=value)
    

        sheet1['A1'] = f"Sales Monthly Report, {period}"
        
        sheet1['B2'] = "Profit-Marging"

        sheet1['F2'] = "Profit-Change"

        sheet1['J2'] = "Sales Percent Change"

        # title table --------------------------------------- #

        sheet1['B8'] = 'Avg. Sales and Profit By Categories'



        sheet1['B3'] =  cs[0][0] 
        
        sheet1['F3'] =  cs[0][1] 
        
        sheet1['J3'] = cs[0][2] 

        sheet1['B3'].number_format = '0.0%'
        sheet1['F3'].number_format = '0.0%'
        sheet1['J3'].number_format = '0.0%'

        # cells merging -------------------------- #
        
        sheet1.merge_cells(start_row=1, start_column=1,
                               end_row=1, end_column=14)

        sheet1.merge_cells(start_row=2, start_column=2,
                               end_row=2, end_column=5)

        sheet1.merge_cells(start_row=2, start_column=6,
                               end_row=2, end_column=9)

        sheet1.merge_cells(start_row=2, start_column=10,
                               end_row=2, end_column=13)

        sheet1.merge_cells(start_row=3, start_column=2,
                   end_row=6, end_column=5)

        sheet1.merge_cells(start_row=3, start_column=6,
                   end_row=6, end_column=9)

        sheet1.merge_cells(start_row=3, start_column=10,
                   end_row=6, end_column=13)
        
        # table cells merging ---------------------------- #

        # title cell merge
        sheet1.merge_cells(start_row=8, start_column=2,
                   end_row=8, end_column=5)

        # --------------------------------------------------- #

        sheet1.merge_cells(start_row=10, start_column=2,
                   end_row=13, end_column=2)
        
        sheet1.merge_cells(start_row=10, start_column=2,
                   end_row=13, end_column=2)

        sheet1.merge_cells(start_row=14, start_column=2,
                   end_row=22, end_column=2) 

        sheet1.merge_cells(start_row=23, start_column=2,
                   end_row=26, end_column=2)

        # ------------------- inserting Charts --------------------------- #

        img1 = drawing.image.Image('./data/bar.png')
        img1.anchor = 'G8'
        sheet1.add_image(img1)
        
        # 2nd chart
        img2 = drawing.image.Image('./data/map.png')
        img2.anchor = 'B28'
        sheet1.add_image(img2)


        return sheet1


    def formato(self):
        sheet1 = self.book['report']
        sheet1.sheet_view.showGridLines = False

        #Headers ---------------------------------- #
        form_he = NamedStyle('form_he')
        form_he.fill = PatternFill(fgColor = "001a33",
                             fill_type='solid')
        form_he.alignment = Alignment(horizontal='center',
                                    vertical= 'center')
        form_he.font = Font(bold=True, size=36, color='ffffff')
        form_he.border = Border(bottom=Side(border_style='thick'), 
                            top=Side(border_style='thick'))
        ## Applying the format ------------------------------------- ##
        for cell in sheet1[1]:
            cell.style = form_he
        ## 2nd Row 
        form_she = NamedStyle('form_she')
        form_she.fill = PatternFill(fgColor = "00264d",
                             fill_type='solid')
        form_she.border = Border(left = Side(border_style='hair'),
                       bottom=Side(border_style='hair'),
                       right=Side(border_style='hair'),
                       top=Side(border_style='hair'))
        form_she.alignment = Alignment(horizontal='center',
                                    vertical= 'center')
        form_she.font = Font(bold=True, size=14, color='ffffff')
        sheet1.row_dimensions[2].height = 18

        # formatting ------------------------------------------------- #
        
        for i in range(2, 14):
            letter = get_column_letter(i)
            sheet1[letter+str(2)].style = form_she
            
        # Cards ------------------------------------------------------- #

        form_ca = NamedStyle('form_ca')
        
        form_ca.alignment = Alignment(horizontal='center',
                              vertical='center')
        
        form_ca.font = Font(bold=True, size=30)
        
        form_ca.border = Border(left = Side(border_style='double', color='004080'),
                       bottom=Side(border_style='double', color='004080'),
                       right=Side(border_style='double', color='004080'),
                       top=Side(border_style='double', color='004080'))
        ## Formatting --------------------------------------------------- #
        for i in range(2, 14):
            letter = get_column_letter(i)
            for j in range(3, 7):
                sheet1[letter+str(j)].style = form_ca

        ## Cols Width ---------------------------------------------------- # 
        sheet1.column_dimensions['A'].width=4
        sheet1.column_dimensions['N'].width=4
                
        for col in range(2, sheet1.max_column):
            
            idx_col = get_column_letter(col)
            
            sheet1.column_dimensions[idx_col].width = 12
            
            sheet1.row_dimensions[1].height = 45
        
        # Format for table ----------------------------------------------- #
        sheet1.row_dimensions[7].height = 6
        sheet1.row_dimensions[8].height = 27 
        sheet1.row_dimensions[9].height = 33
        

        form_t = NamedStyle('form_t')

        form_t.alignment = Alignment(horizontal='center',
                                    vertical= 'center')
        form_t.fill = PatternFill(fgColor='004080', fill_type='solid')
        form_t.font = Font(bold=True, size=13, color='ffffff')
        form_t.border = Border(bottom=Side(border_style='double', color='004080'),
                            right= Side(border_style='double', color='004080'),
                            left=Side(border_style='double', color='004080'),
                            top=Side(border_style='double', color='004080'))

        for i in range(2, 6):
            letter = get_column_letter(i)
            sheet1[letter+str(8)].style = form_t
        
        # title table
        for i in range(2, 6):
            letter = get_column_letter(i)
            sheet1[letter+str(9)].style = form_t

        form_tr = NamedStyle('form_tr')
        form_tr.fill = PatternFill(fgColor='004d99', fill_type='solid')
        form_tr.alignment = Alignment(horizontal='center',
                                    vertical= 'center')
        form_tr.border = Border(bottom=Side(border_style='hair'),
                            right= Side(border_style='hair'),
                            left=Side(border_style='hair'),
                            top=Side(border_style='hair'))

        n = sheet1.max_row
        for i in range(10, n+1):
            sheet1['B'+str(i)].style = form_tr
        
        form_tr1 = NamedStyle('form_tr1')
        form_tr1.alignment = Alignment(
                                    vertical= 'center')
        form_tr1.fill = PatternFill(fgColor='e6f2ff', fill_type='solid')
        form_tr1.font = Font(bold=True, size=10, 

                        )
        
        form_tr1.border = Border(bottom=Side(border_style='hair'),
                            right= Side(border_style='hair'),
                            left=Side(border_style='hair'),
                            top=Side(border_style='hair'))
        
        for i in range(2, 4):
            letter = get_column_letter(i)
            for j in range(10, n+1):
                sheet1[letter+str(j)].style = form_tr1


        
        for row in range(10, n+1):
            sheet1["D{}".format(row)].number_format = '#,##0.00'
            sheet1["E{}".format(row)].number_format = '#,##0.00'
        

        ## Conditional Format 

        fst = FormatObject(type='min')
        sec = FormatObject(type='max')
        data_bar = DataBar(cfvo=[fst, sec], color="638EC6", 
        showValue=None, minLength=None, maxLength=None)

        rule1 = Rule(type='dataBar', dataBar=data_bar)

        sheet1.conditional_formatting.add('D10:E27', rule1)

        # cards

        sheet1.conditional_formatting.add('B3:E6', rule1)
        sheet1.conditional_formatting.add('F3:I6', rule1)
        sheet1.conditional_formatting.add('J3:M6', rule1)

        # Freezing second row
        sec_row = sheet1['A2']
        sheet1.freeze_panes = sec_row

        return sheet1
```

Now, if one picks a date for example: Dec 28, 2015


```python
qz = xl_report(p0, '2015-12-28')
```


```python
qz.sheet_rep()
```




    <Worksheet "report">




```python
qz.formato()
```




    <Worksheet "report">




```python
qz.book.save('./data/test_class.xlsx')
```

The book saved has the following format and values.

<div>
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/07/05/assets/output_23_0.png?raw=true" />
</div>

