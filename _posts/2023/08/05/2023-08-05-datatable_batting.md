---
layout: post
title:  "Baseball Batting Leaders, Simple Dashboard"
date:   2023-08-05 03:15:10 -0400

---

##  MLB Batting Leaders
<div>
  <img src="https://github.com/latesco/apuntes/blob/gh-pages/_posts/2023/08/05/assets/output.gif?raw=true" />
</div>

This task is about querying the Lahman Baseball database (freely downloadable from Lahman web site, and others) in order to extract the following information:

- Top five MLB batting leaders by year. 

- Team leader in: Average, RBI, and OPS by year.

The first query produce a table with the columns: Year, Name, Team, Country, Age, G (games), AB (at bat) , R (runs), H (hits), 2B (doubles), 3B (triples) , HR (home runs), RBI (runs batted in) , Avg (average), OBP (on base percentage), SLG (slugging percentage), OPS (on base plus slugging)

In the second one, seeks to extract the metrics: RBI, Avg, and OPS, for every team by year; the final result will show only which team was the leader in each category.

## Formulas

### On Base Percentage (OBP)

$$OBP=\frac{H+BB+HBP}{{AB+BB+SF+HBP}}$$ 

### Total Bases (TB) 

$$TB= (1 x 1B) + (2 x 2B) + (3 x 3B) + (4 x  HR)$$

### Sluging Percentage (SLG)

$$SLG=\frac{TB}{{AB}}$$

### On Base Plus Slugging (OPS)

$$OPS=OBP+SLG$$

The results will be displayed in a table, created with Plotly.

For this query the functions: RIGHT JOINS, TIMESTAMPDIFF, CONCAT, OVER, PARTITION BY; as well as the construct  CTE (Common Table Expressions) will be used.

The variant of SQL being used is MariaDB, a fork of MySQL.

## Code

These Python packages will allow connecting with the database, executting the query, and reading in the results as a dataframe.


```python
import pandas as pd

from sqlalchemy import create_engine

import os
```


```python
db = open('Baseball', 'r').readline().strip('\n')

engine = create_engine(db)

conn = engine.connect()
```

### The Queries.

#### Top Five Players by Year. 


```python
query_i = """

    WITH v AS (
    WITH w AS (
    SELECT t.playerID AS id,
    TIMESTAMPDIFF(YEAR, CONCAT(t.birthYear,'-', .t.birthMonth, '-', t.birthDay),
                 CONCAT(q.yearID,'-', t.birthMonth,'-', t.birthDay)) AS Age,

    CONCAT(t.nameFirst, ' ', t.nameLast) AS Name,
    SUM(G) AS G,
    SUM(q.AB) AS AB, SUM(q.R) AS R, SUM(q.H) AS H, SUM(q.2B) AS 2B,
    SUM(q.3B) AS 3B,SUM(q.HR) AS HR, SUM(q.RBI) AS RBI, SUM(q.BB) AS BB,
    SUM(q.SO) AS SO, SUM(SB) AS SB, SUM(q.CS) AS CS, SUM(SF) AS SF,
    SUM(HBP) AS HBP, q.yearID AS Year, q.teamID AS Team,
    t.birthCountry AS Country
    FROM Master t
    RIGHT JOIN Batting q
    ON t.playerID = q.playerID
    GROUP BY Year, Name
    )
    SELECT id, Year, Name, Team, Country, Age, G, AB, R, H,
    2B, 3B, HR, RBI, BB, SO, SB, CS, (H/AB) AS Avg, 
    ((H + BB + HBP) / (AB + BB + SF + HBP)) AS OBP,
    (((H - 2B - 3B - HR) + 2 * 2B + 3 * 3B + 4 * HR) / AB) AS SLG,
    ROW_NUMBER() OVER (PARTITION BY Year ORDER BY RBI DESC) AS Nro
    FROM w
    )

    SELECT Year, Name, Team, Country, Age, G, AB, R, H, 2B, 3B, HR, RBI,
    ROUND(Avg * 1000) AS Avg, ROUND(OBP * 1000) AS OBP,
    ROUND(SLG * 1000) AS SLG, ROUND((OBP + SLG),3) AS OPS
    FROM v
    WHERE Nro < 6
    ;
"""
```

Now, with the following code the results are read in as dataframe.


```python
d0 = pd.read_sql(query_i, con = conn)
```

#### Team Batting Leaders

Executing the next query produce the metrics: Average, RBI, and OPS, by team for each of the years recorded in the Lahman database.


```python
query_ii = """

    WITH v AS (
        WITH w AS (
        SELECT t.playerID, q.yearID AS Year, q.teamID AS Team, SUM(q.AB) AS AB,
        SUM(q.R) AS R, SUM(q.H) AS H, SUM(q.2B) AS 2B, SUM(q.3B) AS 3B,
        SUM(q.HR) AS HR, SUM(q.RBI) AS RBI, SUM(q.BB) AS BB, SUM(q.SO) AS SO,
        SUM(SB) AS SB, SUM(q.CS) AS CS, SUM(SF) AS SF,
        SUM(HBP) AS HBP
        FROM Master t
        RIGHT JOIN Batting q
        ON t.playerID = q.playerID
        GROUP BY Year, Team
        )
        SELECT  Year, Team, RBI, (H/AB) AS Avg,
        ((H + BB + HBP) / (AB + BB + SF + HBP)) AS OBP,
        (((H - 2B - 3B - HR) + 2 * 2B + 3 * 3B + 4 * HR) / AB) AS SLG
        FROM w
        )
        SELECT Year, Team, RBI, ROUND(Avg * 1000) AS Avg,
        ROUND((OBP + SLG),3) AS OPS
        FROM v
    ;
"""
```


```python
team_df = pd.read_sql(query_ii, con=conn)
```

### Country Flags.

One of the columns contains the country where the player was born; substituting this string of characters for the country flag, could save some space. The following function does that.


```python
from unicodedata import lookup

import pycountry
```


```python
def gen_iso_codes(X):

    countries = {}

    dx = X.copy()

    dx = dx.replace({'Country':{'P.R.': 'Puerto Rico',
                                'D.R.': 'Dominican Republic', 'CAN': 'Canada',
                                'USA': 'United States'} })

    input_countries = dx.Country.unique().tolist()

    for country in pycountry.countries:
        countries[country.name] = country.alpha_2

    codes = [countries.get(country, 'Unknown code') for country in input_countries]

    iso = dict(zip(input_countries, codes))

    dx['iso'] = dx['Country'].map(iso)

    dx.loc[dx.Country == 'Venezuela', 'iso'] = 'VE'

    dx.loc[dx.Country.str.contains('Curacao'), 'iso'] = 'CW'

    dx['Country'] = (dx['iso']
                  .apply(lambda x: lookup(f"REGIONAL INDICATOR SYMBOL LETTER {x[0]}") +
                         lookup(f"REGIONAL INDICATOR SYMBOL LETTER {x[1]}"))
                 )
    dx = dx.drop('iso', axis=1)

    ord_vars = ['Year', 'Name', 'Team', 'Country', 'Age', 'G', 'AB', 'R',
            'H', '2B', '3B', 'HR', 'RBI', 'Avg', 'OBP', 'SLG', 'OPS']

    dx = dx[ord_vars]

    return dx

```


```python
df = gen_iso_codes(d0)
```

## Results 

Using Dash-Plotly for displaying the results by year in an interactive manner.


```python
from dash import Dash, html, dcc, dash_table, State

import dash_bootstrap_components as dbc

from dash.dependencies import Output, Input

from dash.exceptions import PreventUpdate

import plotly.graph_objs as go

import plotly.express as px
```

A slider widget will serve for selecting the year to shown in the table.


```python
slider = dcc.Slider(1963, 2016, 4,

            marks = {i: '{}'.format(i) for i in range(1963, 2016, 5)
                },
            value=1973)
```

The code in the cell below provides styling for the table.


```python
dtable = dash_table.DataTable(
        columns = [{'name':i, 'id':i} for i in df.drop(['Year'], axis=1).columns],

        style_cell={'padding': '10px',
            'textAlign':'left',
            'backgroundColor': 'rgba(0,0,0,0)'},

        style_header = {'backgroundColor': 'rgba(0,0,0,0.1)', #'#3d3d29', 
            'fontWeight': 'bold'

            },

        style_cell_conditional=[
        {
            'if': {'column_id': c},
            'textAlign': 'center'
        } for c in df.drop('Name', axis=1).columns.tolist()
    ],
        style_as_list_view=True
        )
```

These are the recipients holding content in cards that will present the metrics for team leaders.


```python
# Card components
content01 = [
    dbc.CardHeader("RBI Team Leader"),
    dbc.CardBody(
        [
            html.H5("", className="card-title", id='card_num1'),
            html.H6("", className='card-title', id='card_num2'),
            html.P(
                "Runs Batted In",
                className="card-text", id='card_text1'),
        ]
    ),
]

content02 = [
    dbc.CardHeader("Avg Team Leader"),
    dbc.CardBody(
        [
            html.H5("", className="card-title", id='card_num3'),
            html.H6("", className='card-title', id='card_num4'),
            html.P(
                "Average",
                className="card-text",
            ),
        ]
    ),
]

content03 = [
    dbc.CardHeader("OPS Team Leader"),
    dbc.CardBody(
        [
            html.H5("", className="card-title", id='card_num5'),
            html.H6("", className="card-title", id='card_num6'),
            html.P(
                "On Base Plus Slugging", className="card-text",
            ),
        ]
    ),
]
```

Here is the design for the cards.


```python
cards = html.Div(
    [
        dbc.Row(
            [
                dbc.Col(''),
                dbc.Col(dbc.Card(content01, color='info', outline=True)),
                dbc.Col(
                    dbc.Card(content02, color='info', outline=True)
                ),
                dbc.Col(dbc.Card(content03, color='info', outline=True)), dbc.Col('')
            ],
            className="mb-4",
        ),

    ]
)
```

Initializing the app.


```python
app = Dash(__name__,
        external_stylesheets=[dbc.themes.DARKLY])
```

The general layout of the table.


```python
app.layout = html.Div(
    [
        html.H2("", style={"textAlign": 'center'},
            id='header'),
        html.Br(),
        cards,
        html.Br(),
        dbc.Row([dbc.Col(''), dbc.Col([dtable], width=10), dbc.Col('')
            ]),

        html.Br(),
        dbc.Row([dbc.Col(''), dbc.Col([slider], width=10), dbc.Col('')])

    ]
)
```

## Callbacks

This functions provide the ability to interact with the table, e.g. allowing the selection of a given year.


```python
@app.callback(
    Output('header', "children"),
    Input(slider, "value"),
)
def update_header(slider_value):
    return 'MLB Batting Leaders  ' + str(slider_value) + ' ' +u'\u26BE'

@app.callback(
    Output('card_num1', "children"),
    Output('card_num2', 'children'),
    Input(slider, "value"),
)
def update_card01(slider_value):
    filt_df = (team_df.loc[(team_df.Year == slider_value), ['Team', 'RBI']]
                .sort_values('RBI', ascending=False)

            )

    f = filt_df.iloc[0].values.tolist()
    return f[0], f[1]

@app.callback(
    Output('card_num3', "children"),
    Output('card_num4', 'children'),
    Input(slider, "value"),
)

def update_card02(slider_value):
    filt_df = (team_df.loc[(team_df.Year == slider_value), ['Team', 'Avg']]
                .sort_values('Avg', ascending=False)

            )

    f = filt_df.iloc[0].values.tolist()
    if f[1] == 1000:
        f[1] = '1.000'
    else:
        f[1] = '.' + str(int(f[1]))
    return f[0], f[1]


@app.callback(
    Output('card_num5', "children"),
    Output('card_num6', 'children'),
    Input(slider, "value"),
)
def update_card03(slider_value):
    filt_df = (team_df.loc[(team_df.Year == slider_value), ['Team', 'OPS']]
                .sort_values('OPS', ascending=False)

            )

    f = filt_df.iloc[0].values.tolist()
    if f[1] > 1:
        f[1] = round(f[1], 3)
    else:
        f[1] = str(f[1])
        f[1] = f[1].replace('0','')
    return f[0], f[1]


@app.callback(
    Output(dtable, "data"),
    Input(slider, "value"),
)
def update_table(slider_value):
    #vs = ['Name', 'Age', 'Avg', 'RBI', 'Team', 'Flag']
    vs = df.columns.tolist()
    vs = vs[1:]
    if not slider_value:
        return dash.no_update
    dff = df.loc[(df.Year == slider_value), vs]
    return dff.to_dict("records")
```


```python
if __name__ == "__main__":
    app.run_server(debug=True)
```

