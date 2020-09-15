# Visualizing school workforce in England
## From the [MakeoverMonday](https://www.makeovermonday.co.uk/#) prompt for the week of 2020-9-14
- Description of the data can be found on [uk's goverment website](https://explore-education-statistics.service.gov.uk/find-statistics/school-workforce-in-england#releaseHeadlines-charts) 
- The specific data file used here can be downloaded from [data.world](https://data.world/makeovermonday/2020w37-school-workforce-in-england)


```python
import numpy as np
import pandas as pd

import altair as alt
```


```python
# import the data
dat = pd.read_csv('data-school-workforce-in-england.csv')
dat
# I did some data checks:
# some columns have no variation: location, school_type, age_category
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
      <th>location</th>
      <th>location_code</th>
      <th>geographic_level</th>
      <th>time_period</th>
      <th>gender</th>
      <th>school_type</th>
      <th>grade</th>
      <th>age_category</th>
      <th>average_mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>England</td>
      <td>E92000001</td>
      <td>country</td>
      <td>201011</td>
      <td>Female</td>
      <td>Total state-funded schools</td>
      <td>Classroom teachers</td>
      <td>Total</td>
      <td>34174.7</td>
    </tr>
    <tr>
      <td>1</td>
      <td>England</td>
      <td>E92000001</td>
      <td>country</td>
      <td>201011</td>
      <td>Female</td>
      <td>Total state-funded schools</td>
      <td>Head teachers</td>
      <td>Total</td>
      <td>60289.8</td>
    </tr>
    <tr>
      <td>2</td>
      <td>England</td>
      <td>E92000001</td>
      <td>country</td>
      <td>201011</td>
      <td>Female</td>
      <td>Total state-funded schools</td>
      <td>Other Leadership teachers</td>
      <td>Total</td>
      <td>49442.5</td>
    </tr>
    <tr>
      <td>3</td>
      <td>England</td>
      <td>E92000001</td>
      <td>country</td>
      <td>201011</td>
      <td>Female</td>
      <td>Total state-funded schools</td>
      <td>Total</td>
      <td>Total</td>
      <td>36366.9</td>
    </tr>
    <tr>
      <td>4</td>
      <td>England</td>
      <td>E92000001</td>
      <td>country</td>
      <td>201011</td>
      <td>Male</td>
      <td>Total state-funded schools</td>
      <td>Classroom teachers</td>
      <td>Total</td>
      <td>35494</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>155</td>
      <td>England</td>
      <td>E92000001</td>
      <td>country</td>
      <td>201920</td>
      <td>Total</td>
      <td>Total state-funded schools</td>
      <td>Total</td>
      <td>Total</td>
      <td>40537</td>
    </tr>
    <tr>
      <td>156</td>
      <td>England</td>
      <td>E92000001</td>
      <td>country</td>
      <td>201920</td>
      <td>Unclassified</td>
      <td>Total state-funded schools</td>
      <td>Classroom teachers</td>
      <td>Total</td>
      <td>34244</td>
    </tr>
    <tr>
      <td>157</td>
      <td>England</td>
      <td>E92000001</td>
      <td>country</td>
      <td>201920</td>
      <td>Unclassified</td>
      <td>Total state-funded schools</td>
      <td>Head teachers</td>
      <td>Total</td>
      <td>63910.4</td>
    </tr>
    <tr>
      <td>158</td>
      <td>England</td>
      <td>E92000001</td>
      <td>country</td>
      <td>201920</td>
      <td>Unclassified</td>
      <td>Total state-funded schools</td>
      <td>Other Leadership teachers</td>
      <td>Total</td>
      <td>53403.1</td>
    </tr>
    <tr>
      <td>159</td>
      <td>England</td>
      <td>E92000001</td>
      <td>country</td>
      <td>201920</td>
      <td>Unclassified</td>
      <td>Total state-funded schools</td>
      <td>Total</td>
      <td>Total</td>
      <td>36962.3</td>
    </tr>
  </tbody>
</table>
<p>160 rows × 9 columns</p>
</div>




```python
dat['gender'].unique()
```




    array(['Female', 'Male', 'Total', 'Unclassified'], dtype=object)




```python
dat['year'] = [str(x)[:4] for x in dat['time_period']]
dat['gender_sign'] = ["\u2640" if i == "Female" else "\u2642" for i in dat['gender']]
dat['grade'] = ["Leadership teachers" if i == "Other Leadership teachers" else i for i in dat['grade']]
```


```python
data = dat[["year", "gender", "gender_sign", "grade", "average_mean"]]\
       .loc[(dat['gender'].isin(['Female', 'Male'])) & (dat['grade'] != 'Total')].copy()
data.shape
```




    (60, 5)




```python
data['average_mean'] = pd.to_numeric(data['average_mean'])
data.dtypes
```




    year             object
    gender           object
    gender_sign      object
    grade            object
    average_mean    float64
    dtype: object




```python
# check if the total entries are correct:
tcheck_gender = pd.pivot_table(data, 
                               values='average_mean', 
                               index=['grade'],
                               columns=['gender'], aggfunc=np.mean)

tcheck_gender 
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
      <th>gender</th>
      <th>Female</th>
      <th>Male</th>
    </tr>
    <tr>
      <th>grade</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Classroom teachers</td>
      <td>34758.97</td>
      <td>35700.61</td>
    </tr>
    <tr>
      <td>Head teachers</td>
      <td>64056.41</td>
      <td>72011.81</td>
    </tr>
    <tr>
      <td>Leadership teachers</td>
      <td>50997.19</td>
      <td>54859.20</td>
    </tr>
  </tbody>
</table>
</div>




```python
# pay pattern in 2019
chart = alt.Chart(data.loc[data['year'] == '2019']).properties( width=50, height=200 ) 
text = chart.mark_text().encode(
    alt.X('year', title='Year', scale=alt.Scale(zero=False)),
    alt.Y('average_mean', title='Average Pay', scale=alt.Scale(zero=False)),
    color = "grade", text = "gender_sign", 
    column = alt.Column('grade', sort=['Classroom teachers', 'Other Leadership teachers']))
line = chart.mark_line().encode(
    alt.X('year', title='Year', scale=alt.Scale(zero=False)),
    alt.Y('average_mean', title='Average Pay'),
    color = "grade", strokeDash = 'year')
text 
```





<div id="altair-viz-5ed73af0b0f64722a9d8cc47ea3a06f4"></div>
<script type="text/javascript">
  (function(spec, embedOpt){
    let outputDiv = document.currentScript.previousElementSibling;
    if (outputDiv.id !== "altair-viz-5ed73af0b0f64722a9d8cc47ea3a06f4") {
      outputDiv = document.getElementById("altair-viz-5ed73af0b0f64722a9d8cc47ea3a06f4");
    }
    const paths = {
      "vega": "https://cdn.jsdelivr.net/npm//vega@5?noext",
      "vega-lib": "https://cdn.jsdelivr.net/npm//vega-lib?noext",
      "vega-lite": "https://cdn.jsdelivr.net/npm//vega-lite@4.8.1?noext",
      "vega-embed": "https://cdn.jsdelivr.net/npm//vega-embed@6?noext",
    };

    function loadScript(lib) {
      return new Promise(function(resolve, reject) {
        var s = document.createElement('script');
        s.src = paths[lib];
        s.async = true;
        s.onload = () => resolve(paths[lib]);
        s.onerror = () => reject(`Error loading script: ${paths[lib]}`);
        document.getElementsByTagName("head")[0].appendChild(s);
      });
    }

    function showError(err) {
      outputDiv.innerHTML = `<div class="error" style="color:red;">${err}</div>`;
      throw err;
    }

    function displayChart(vegaEmbed) {
      vegaEmbed(outputDiv, spec, embedOpt)
        .catch(err => showError(`Javascript Error: ${err.message}<br>This usually means there's a typo in your chart specification. See the javascript console for the full traceback.`));
    }

    if(typeof define === "function" && define.amd) {
      requirejs.config({paths});
      require(["vega-embed"], displayChart, err => showError(`Error loading script: ${err.message}`));
    } else if (typeof vegaEmbed === "function") {
      displayChart(vegaEmbed);
    } else {
      loadScript("vega")
        .then(() => loadScript("vega-lite"))
        .then(() => loadScript("vega-embed"))
        .catch(showError)
        .then(() => displayChart(vegaEmbed));
    }
  })({"config": {"view": {"continuousWidth": 400, "continuousHeight": 300}}, "data": {"name": "data-69ce14cdcd0120cd25fd71a765328e5e"}, "mark": "text", "encoding": {"color": {"type": "nominal", "field": "grade"}, "column": {"type": "nominal", "field": "grade", "sort": ["Classroom teachers", "Other Leadership teachers"]}, "text": {"type": "nominal", "field": "gender_sign"}, "x": {"type": "nominal", "field": "year", "scale": {"zero": false}, "title": "Year"}, "y": {"type": "quantitative", "field": "average_mean", "scale": {"zero": false}, "title": "Average Pay"}}, "height": 200, "width": 50, "$schema": "https://vega.github.io/schema/vega-lite/v4.8.1.json", "datasets": {"data-69ce14cdcd0120cd25fd71a765328e5e": [{"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 36985.4}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 68869.6}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 53832.5}, {"year": "2019", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 37885.1}, {"year": "2019", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 77361.5}, {"year": "2019", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 57406.6}]}}, {"mode": "vega-lite"});
</script>




```python
# compare pay change in female teachers between 2010 and 2019
alt.Chart(data.loc[(data['gender'] == "Female") & (data['year'].isin(['2010', '2019']))]).mark_bar().encode(
    alt.X('year', title='Year', scale=alt.Scale(zero=False)),
    alt.Y('average_mean', title='Average Pay', scale=alt.Scale(zero=False)),
    color = 'grade',
    column = alt.Column('grade', sort=['Classroom teachers', 'Other Leadership teachers']))\
.properties( width=60, height=300 ) 
```





<div id="altair-viz-97262356af694786b5f9a7324d938eb4"></div>
<script type="text/javascript">
  (function(spec, embedOpt){
    let outputDiv = document.currentScript.previousElementSibling;
    if (outputDiv.id !== "altair-viz-97262356af694786b5f9a7324d938eb4") {
      outputDiv = document.getElementById("altair-viz-97262356af694786b5f9a7324d938eb4");
    }
    const paths = {
      "vega": "https://cdn.jsdelivr.net/npm//vega@5?noext",
      "vega-lib": "https://cdn.jsdelivr.net/npm//vega-lib?noext",
      "vega-lite": "https://cdn.jsdelivr.net/npm//vega-lite@4.8.1?noext",
      "vega-embed": "https://cdn.jsdelivr.net/npm//vega-embed@6?noext",
    };

    function loadScript(lib) {
      return new Promise(function(resolve, reject) {
        var s = document.createElement('script');
        s.src = paths[lib];
        s.async = true;
        s.onload = () => resolve(paths[lib]);
        s.onerror = () => reject(`Error loading script: ${paths[lib]}`);
        document.getElementsByTagName("head")[0].appendChild(s);
      });
    }

    function showError(err) {
      outputDiv.innerHTML = `<div class="error" style="color:red;">${err}</div>`;
      throw err;
    }

    function displayChart(vegaEmbed) {
      vegaEmbed(outputDiv, spec, embedOpt)
        .catch(err => showError(`Javascript Error: ${err.message}<br>This usually means there's a typo in your chart specification. See the javascript console for the full traceback.`));
    }

    if(typeof define === "function" && define.amd) {
      requirejs.config({paths});
      require(["vega-embed"], displayChart, err => showError(`Error loading script: ${err.message}`));
    } else if (typeof vegaEmbed === "function") {
      displayChart(vegaEmbed);
    } else {
      loadScript("vega")
        .then(() => loadScript("vega-lite"))
        .then(() => loadScript("vega-embed"))
        .catch(showError)
        .then(() => displayChart(vegaEmbed));
    }
  })({"config": {"view": {"continuousWidth": 400, "continuousHeight": 300}}, "data": {"name": "data-e05f249a2e805cb24ebaf5e31a2dcadc"}, "mark": "bar", "encoding": {"color": {"type": "nominal", "field": "grade"}, "column": {"type": "nominal", "field": "grade", "sort": ["Classroom teachers", "Other Leadership teachers"]}, "x": {"type": "nominal", "field": "year", "scale": {"zero": false}, "title": "Year"}, "y": {"type": "quantitative", "field": "average_mean", "scale": {"zero": false}, "title": "Average Pay"}}, "height": 300, "width": 60, "$schema": "https://vega.github.io/schema/vega-lite/v4.8.1.json", "datasets": {"data-e05f249a2e805cb24ebaf5e31a2dcadc": [{"year": "2010", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34174.7}, {"year": "2010", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 60289.8}, {"year": "2010", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 49442.5}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 36985.4}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 68869.6}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 53832.5}]}}, {"mode": "vega-lite"});
</script>




```python
# complete time series for female teachers
alt.Chart(data.loc[data['gender'] == "Female"]).mark_line().encode(
    alt.X('year', title='Year', scale=alt.Scale(zero=False)),
    alt.Y('average_mean', title='Average Pay'),
    color = "grade")
```





<div id="altair-viz-0f39aadebf604f11b3555a9e4a67187f"></div>
<script type="text/javascript">
  (function(spec, embedOpt){
    let outputDiv = document.currentScript.previousElementSibling;
    if (outputDiv.id !== "altair-viz-0f39aadebf604f11b3555a9e4a67187f") {
      outputDiv = document.getElementById("altair-viz-0f39aadebf604f11b3555a9e4a67187f");
    }
    const paths = {
      "vega": "https://cdn.jsdelivr.net/npm//vega@5?noext",
      "vega-lib": "https://cdn.jsdelivr.net/npm//vega-lib?noext",
      "vega-lite": "https://cdn.jsdelivr.net/npm//vega-lite@4.8.1?noext",
      "vega-embed": "https://cdn.jsdelivr.net/npm//vega-embed@6?noext",
    };

    function loadScript(lib) {
      return new Promise(function(resolve, reject) {
        var s = document.createElement('script');
        s.src = paths[lib];
        s.async = true;
        s.onload = () => resolve(paths[lib]);
        s.onerror = () => reject(`Error loading script: ${paths[lib]}`);
        document.getElementsByTagName("head")[0].appendChild(s);
      });
    }

    function showError(err) {
      outputDiv.innerHTML = `<div class="error" style="color:red;">${err}</div>`;
      throw err;
    }

    function displayChart(vegaEmbed) {
      vegaEmbed(outputDiv, spec, embedOpt)
        .catch(err => showError(`Javascript Error: ${err.message}<br>This usually means there's a typo in your chart specification. See the javascript console for the full traceback.`));
    }

    if(typeof define === "function" && define.amd) {
      requirejs.config({paths});
      require(["vega-embed"], displayChart, err => showError(`Error loading script: ${err.message}`));
    } else if (typeof vegaEmbed === "function") {
      displayChart(vegaEmbed);
    } else {
      loadScript("vega")
        .then(() => loadScript("vega-lite"))
        .then(() => loadScript("vega-embed"))
        .catch(showError)
        .then(() => displayChart(vegaEmbed));
    }
  })({"config": {"view": {"continuousWidth": 400, "continuousHeight": 300}}, "data": {"name": "data-cf4c407adf6d861695acafd6f7577d5d"}, "mark": "line", "encoding": {"color": {"type": "nominal", "field": "grade"}, "x": {"type": "nominal", "field": "year", "scale": {"zero": false}, "title": "Year"}, "y": {"type": "quantitative", "field": "average_mean", "title": "Average Pay"}}, "$schema": "https://vega.github.io/schema/vega-lite/v4.8.1.json", "datasets": {"data-cf4c407adf6d861695acafd6f7577d5d": [{"year": "2010", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34174.7}, {"year": "2010", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 60289.8}, {"year": "2010", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 49442.5}, {"year": "2011", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34134.1}, {"year": "2011", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 60923.2}, {"year": "2011", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 49649.3}, {"year": "2012", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 33824.6}, {"year": "2012", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 61308.4}, {"year": "2012", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 49655.1}, {"year": "2013", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34075.8}, {"year": "2013", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 62353.6}, {"year": "2013", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 50124.4}, {"year": "2014", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34151.6}, {"year": "2014", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 63462.3}, {"year": "2014", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 50502.1}, {"year": "2015", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34397.1}, {"year": "2015", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 64497.3}, {"year": "2015", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 50899.6}, {"year": "2016", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34674.4}, {"year": "2016", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 65425.9}, {"year": "2016", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 51366.4}, {"year": "2017", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 35178.7}, {"year": "2017", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 66173.7}, {"year": "2017", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 51854.2}, {"year": "2018", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 35993.3}, {"year": "2018", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 67260.3}, {"year": "2018", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 52645.8}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 36985.4}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 68869.6}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 53832.5}]}}, {"mode": "vega-lite"});
</script>




```python
points = alt.Chart(data, title = "Gender pay gaps among teachers in England").mark_text(size = 15).encode(
    alt.X('year', title='Year', 
          scale=alt.Scale(zero = False), 
          axis = alt.Axis(labelAngle = 0, values= [2010, 2013, 2016, 2019])),
    alt.Y('average_mean', title='Average Pay', axis = alt.Axis(titleAngle = 0, titleX=-90)),
    color = "grade", text = "gender_sign"
).properties( width=300, height=400 ) 

labels = alt.Chart(data).mark_text(align='left', dx = 10, dy = 10).encode(
    alt.X('year', aggregate='max'),
    alt.Y('average_mean', aggregate='max'),
    alt.Text('grade'),
    alt.Color('grade', legend=None))

points +labels
```





<div id="altair-viz-c4774ae0c282461b98612b802377bdce"></div>
<script type="text/javascript">
  (function(spec, embedOpt){
    let outputDiv = document.currentScript.previousElementSibling;
    if (outputDiv.id !== "altair-viz-c4774ae0c282461b98612b802377bdce") {
      outputDiv = document.getElementById("altair-viz-c4774ae0c282461b98612b802377bdce");
    }
    const paths = {
      "vega": "https://cdn.jsdelivr.net/npm//vega@5?noext",
      "vega-lib": "https://cdn.jsdelivr.net/npm//vega-lib?noext",
      "vega-lite": "https://cdn.jsdelivr.net/npm//vega-lite@4.8.1?noext",
      "vega-embed": "https://cdn.jsdelivr.net/npm//vega-embed@6?noext",
    };

    function loadScript(lib) {
      return new Promise(function(resolve, reject) {
        var s = document.createElement('script');
        s.src = paths[lib];
        s.async = true;
        s.onload = () => resolve(paths[lib]);
        s.onerror = () => reject(`Error loading script: ${paths[lib]}`);
        document.getElementsByTagName("head")[0].appendChild(s);
      });
    }

    function showError(err) {
      outputDiv.innerHTML = `<div class="error" style="color:red;">${err}</div>`;
      throw err;
    }

    function displayChart(vegaEmbed) {
      vegaEmbed(outputDiv, spec, embedOpt)
        .catch(err => showError(`Javascript Error: ${err.message}<br>This usually means there's a typo in your chart specification. See the javascript console for the full traceback.`));
    }

    if(typeof define === "function" && define.amd) {
      requirejs.config({paths});
      require(["vega-embed"], displayChart, err => showError(`Error loading script: ${err.message}`));
    } else if (typeof vegaEmbed === "function") {
      displayChart(vegaEmbed);
    } else {
      loadScript("vega")
        .then(() => loadScript("vega-lite"))
        .then(() => loadScript("vega-embed"))
        .catch(showError)
        .then(() => displayChart(vegaEmbed));
    }
  })({"config": {"view": {"continuousWidth": 400, "continuousHeight": 300}}, "layer": [{"mark": {"type": "text", "size": 15}, "encoding": {"color": {"type": "nominal", "field": "grade"}, "text": {"type": "nominal", "field": "gender_sign"}, "x": {"type": "nominal", "axis": {"labelAngle": 0, "values": [2010, 2013, 2016, 2019]}, "field": "year", "scale": {"zero": false}, "title": "Year"}, "y": {"type": "quantitative", "axis": {"titleAngle": 0, "titleX": -90}, "field": "average_mean", "title": "Average Pay"}}, "height": 400, "title": "Gender pay gaps among teachers in England", "width": 300}, {"mark": {"type": "text", "align": "left", "dx": 10, "dy": 10}, "encoding": {"color": {"type": "nominal", "field": "grade", "legend": null}, "text": {"type": "nominal", "field": "grade"}, "x": {"type": "nominal", "aggregate": "max", "field": "year"}, "y": {"type": "quantitative", "aggregate": "max", "field": "average_mean"}}}], "data": {"name": "data-9aa712486742a479d2d33b0fd5cf6557"}, "$schema": "https://vega.github.io/schema/vega-lite/v4.8.1.json", "datasets": {"data-9aa712486742a479d2d33b0fd5cf6557": [{"year": "2010", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34174.7}, {"year": "2010", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 60289.8}, {"year": "2010", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 49442.5}, {"year": "2010", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 35494.0}, {"year": "2010", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 67401.5}, {"year": "2010", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 53598.7}, {"year": "2011", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34134.1}, {"year": "2011", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 60923.2}, {"year": "2011", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 49649.3}, {"year": "2011", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 35294.6}, {"year": "2011", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 68268.1}, {"year": "2011", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 53749.2}, {"year": "2012", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 33824.6}, {"year": "2012", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 61308.4}, {"year": "2012", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 49655.1}, {"year": "2012", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 34805.7}, {"year": "2012", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 68944.4}, {"year": "2012", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 53703.8}, {"year": "2013", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34075.8}, {"year": "2013", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 62353.6}, {"year": "2013", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 50124.4}, {"year": "2013", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 35023.2}, {"year": "2013", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 70376.8}, {"year": "2013", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 54218.7}, {"year": "2014", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34151.6}, {"year": "2014", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 63462.3}, {"year": "2014", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 50502.1}, {"year": "2014", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 34980.4}, {"year": "2014", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 71504.5}, {"year": "2014", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 54374.2}, {"year": "2015", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34397.1}, {"year": "2015", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 64497.3}, {"year": "2015", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 50899.6}, {"year": "2015", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 35201.1}, {"year": "2015", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 72716.6}, {"year": "2015", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 54769.7}, {"year": "2016", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 34674.4}, {"year": "2016", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 65425.9}, {"year": "2016", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 51366.4}, {"year": "2016", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 35471.4}, {"year": "2016", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 73569.3}, {"year": "2016", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 55100.0}, {"year": "2017", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 35178.7}, {"year": "2017", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 66173.7}, {"year": "2017", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 51854.2}, {"year": "2017", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 36020.1}, {"year": "2017", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 74510.7}, {"year": "2017", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 55534.4}, {"year": "2018", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 35993.3}, {"year": "2018", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 67260.3}, {"year": "2018", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 52645.8}, {"year": "2018", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 36830.5}, {"year": "2018", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 75464.7}, {"year": "2018", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 56136.7}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Classroom teachers", "average_mean": 36985.4}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Head teachers", "average_mean": 68869.6}, {"year": "2019", "gender": "Female", "gender_sign": "\u2640", "grade": "Leadership teachers", "average_mean": 53832.5}, {"year": "2019", "gender": "Male", "gender_sign": "\u2642", "grade": "Classroom teachers", "average_mean": 37885.1}, {"year": "2019", "gender": "Male", "gender_sign": "\u2642", "grade": "Head teachers", "average_mean": 77361.5}, {"year": "2019", "gender": "Male", "gender_sign": "\u2642", "grade": "Leadership teachers", "average_mean": 57406.6}]}}, {"mode": "vega-lite"});
</script>


