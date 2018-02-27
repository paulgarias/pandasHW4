

```python
import pandas as pd
import numpy as np
```


```python
df_schools  = pd.read_csv("raw_data/schools_complete.csv")
df_students = pd.read_csv("raw_data/students_complete.csv")
```

## District Summary


```python
passing_rate = 0.5*(len(df_students[df_students.math_score>65])/len(df_students) + len(df_students[df_students.reading_score>65])/len(df_students))
df_summary = pd.DataFrame({"Summary Item":
                           ['Total schools', 'Total students','Total budget','Average Math Score',
                           'Average Reading Score','Percentage of student passing Math (>65)',
                            'Percentage of student passing Reading (>65)','Overall passing rate'],
                          "Data":['{:02d}'.format(len(df_schools)),len(df_students),'$'+'{:,}'.format(df_schools['budget'].sum()),
                                  df_students['math_score'].mean(),df_students['reading_score'].mean(),
                                  '{:4.2f}'.format(100*len(df_students[df_students.math_score>65])/len(df_students))+"%",
                                  '{:4.2f}'.format(100*len(df_students[df_students.reading_score>65])/len(df_students))+"%",
                                  '{:4.2f}'.format(100*passing_rate)+"%"]})
```


```python
df_summary[['Summary Item','Data']]
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
      <th>Summary Item</th>
      <th>Data</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Total schools</td>
      <td>15</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Total students</td>
      <td>39170</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Total budget</td>
      <td>$24,649,428</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Average Math Score</td>
      <td>78.9854</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Average Reading Score</td>
      <td>81.8778</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Percentage of student passing Math (&gt;65)</td>
      <td>83.11%</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Percentage of student passing Reading (&gt;65)</td>
      <td>94.26%</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Overall passing rate</td>
      <td>88.69%</td>
    </tr>
  </tbody>
</table>
</div>



## School Summary


```python
df_schools = df_schools.sort_values('name').reset_index(drop=True)
```


```python
df_student_count = pd.DataFrame(df_students.groupby('school').count()['name'])
df_student_count = df_student_count.reset_index(drop=False)
df_student_count.columns = ['name','Number_of_students']
```


```python
df_schools_2= df_schools.set_index('name').join(df_student_count.set_index('name'))
df_schools_2 = df_schools_2.reset_index(drop=False)
df_schools_2['Budget per student']=df_schools_2.budget/df_schools_2.Number_of_students
df_mean_math = pd.DataFrame(df_students.groupby('school')['math_score'].mean()).reset_index(drop=False)
df_mean_math.columns = ['name','mean_math_score']
df_schools_3 = df_schools_2.set_index('name').join(df_mean_math.set_index('name'))
df_schools_3 = df_schools_3.reset_index(drop=False)
```


```python
df_mean_reading = pd.DataFrame(df_students.groupby('school')['reading_score'].mean()).reset_index(drop=False)
df_mean_reading.columns = ['name','mean_reading_score']
df_schools_4 = df_schools_3.set_index('name').join(df_mean_reading.set_index('name'))
df_schools_4 = df_schools_4.reset_index(drop=False)
```


```python
# Need the percentage of students that are passing math in each school
# 1. Get the number of students in each school
#    df_schools_4['Number_of_students']
# 2. In each of those school, get the number of students that have passed math 
number_students_passing = df_students[df_students.math_score>65].groupby('school').count()
df_schools_4['Percentage_passing_math']=100*number_students_passing.reset_index()['math_score']/df_schools_4['Number_of_students']
```


```python
number_students_passing_reading = df_students[df_students.reading_score>65].groupby('school').count()
df_schools_4['Percentage_passing_reading']=100*number_students_passing_reading.reset_index()['reading_score']/df_schools_4['Number_of_students']
```


```python
df_schools_4['Overall Passing Rate'] = 0.5*(df_schools_4['Percentage_passing_reading']+df_schools_4['Percentage_passing_math'])
```


```python
list_of_items =['name','type','Number_of_students','Budget per student','mean_math_score','mean_reading_score','Percentage_passing_math','Percentage_passing_reading','Overall Passing Rate']
schools_summary = df_schools_4.copy()
schools_summary = schools_summary[list_of_items]
schools_summary['Percentage_passing_math'] = schools_summary['Percentage_passing_math'].map('{:4.2f}%'.format)
schools_summary['Percentage_passing_reading'] = schools_summary['Percentage_passing_reading'].map('{:4.2f}%'.format)
schools_summary['Overall Passing Rate'] = schools_summary['Overall Passing Rate'].map('{:4.2f}%'.format)
schools_summary['Budget per student'] = schools_summary['Budget per student'].map('${:,}'.format)
schools_summary
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
      <th>name</th>
      <th>type</th>
      <th>Number_of_students</th>
      <th>Budget per student</th>
      <th>mean_math_score</th>
      <th>mean_reading_score</th>
      <th>Percentage_passing_math</th>
      <th>Percentage_passing_reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bailey High School</td>
      <td>District</td>
      <td>4976</td>
      <td>$628.0</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>75.60%</td>
      <td>91.90%</td>
      <td>83.75%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cabrera High School</td>
      <td>Charter</td>
      <td>1858</td>
      <td>$582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>2949</td>
      <td>$639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>74.87%</td>
      <td>91.90%</td>
      <td>83.38%</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ford High School</td>
      <td>District</td>
      <td>2739</td>
      <td>$644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>76.20%</td>
      <td>90.80%</td>
      <td>83.50%</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>1468</td>
      <td>$625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Hernandez High School</td>
      <td>District</td>
      <td>4635</td>
      <td>$652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>75.47%</td>
      <td>91.46%</td>
      <td>83.46%</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Holden High School</td>
      <td>Charter</td>
      <td>427</td>
      <td>$581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Huang High School</td>
      <td>District</td>
      <td>2917</td>
      <td>$655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>75.28%</td>
      <td>91.77%</td>
      <td>83.53%</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Johnson High School</td>
      <td>District</td>
      <td>4761</td>
      <td>$650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>75.38%</td>
      <td>92.06%</td>
      <td>83.72%</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>Charter</td>
      <td>962</td>
      <td>$609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Rodriguez High School</td>
      <td>District</td>
      <td>3999</td>
      <td>$637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>75.54%</td>
      <td>91.52%</td>
      <td>83.53%</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Shelton High School</td>
      <td>Charter</td>
      <td>1761</td>
      <td>$600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Thomas High School</td>
      <td>Charter</td>
      <td>1635</td>
      <td>$638.0</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Wilson High School</td>
      <td>Charter</td>
      <td>2283</td>
      <td>$578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Wright High School</td>
      <td>Charter</td>
      <td>1800</td>
      <td>$583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
  </tbody>
</table>
</div>



## Top Performing Schools by Passing Rate


```python
df_schools_top = df_schools_4.sort_values('Overall Passing Rate',ascending=False)
df_schools_top = df_schools_top[list_of_items]
df_schools_top['Percentage_passing_math'] = df_schools_top['Percentage_passing_math'].map('{:4.2f}%'.format)
df_schools_top['Percentage_passing_reading'] = df_schools_top['Percentage_passing_reading'].map('{:4.2f}%'.format)
df_schools_top['Overall Passing Rate'] = df_schools_top['Overall Passing Rate'].map('{:4.2f}%'.format)
df_schools_top['Budget per student'] = df_schools_top['Budget per student'].map('${:,}'.format)
df_schools_top.head(5)
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
      <th>name</th>
      <th>type</th>
      <th>Number_of_students</th>
      <th>Budget per student</th>
      <th>mean_math_score</th>
      <th>mean_reading_score</th>
      <th>Percentage_passing_math</th>
      <th>Percentage_passing_reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Cabrera High School</td>
      <td>Charter</td>
      <td>1858</td>
      <td>$582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>1468</td>
      <td>$625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Holden High School</td>
      <td>Charter</td>
      <td>427</td>
      <td>$581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>Charter</td>
      <td>962</td>
      <td>$609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Shelton High School</td>
      <td>Charter</td>
      <td>1761</td>
      <td>$600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
  </tbody>
</table>
</div>



## Bottom Performing Schools by Passing Rate


```python
df_schools_bottom = df_schools_4.sort_values('Overall Passing Rate')
df_schools_bottom = df_schools_bottom[list_of_items]
df_schools_bottom['Percentage_passing_math'] = df_schools_bottom['Percentage_passing_math'].map('{:4.2f}%'.format)
df_schools_bottom['Percentage_passing_reading'] = df_schools_bottom['Percentage_passing_reading'].map('{:4.2f}%'.format)
df_schools_bottom['Overall Passing Rate'] = df_schools_bottom['Overall Passing Rate'].map('{:4.2f}%'.format)
df_schools_bottom['Budget per student'] = df_schools_bottom['Budget per student'].map('${:,}'.format)
df_schools_bottom.head(5)
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
      <th>name</th>
      <th>type</th>
      <th>Number_of_students</th>
      <th>Budget per student</th>
      <th>mean_math_score</th>
      <th>mean_reading_score</th>
      <th>Percentage_passing_math</th>
      <th>Percentage_passing_reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>2949</td>
      <td>$639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>74.87%</td>
      <td>91.90%</td>
      <td>83.38%</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Hernandez High School</td>
      <td>District</td>
      <td>4635</td>
      <td>$652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>75.47%</td>
      <td>91.46%</td>
      <td>83.46%</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ford High School</td>
      <td>District</td>
      <td>2739</td>
      <td>$644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>76.20%</td>
      <td>90.80%</td>
      <td>83.50%</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Huang High School</td>
      <td>District</td>
      <td>2917</td>
      <td>$655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>75.28%</td>
      <td>91.77%</td>
      <td>83.53%</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Rodriguez High School</td>
      <td>District</td>
      <td>3999</td>
      <td>$637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>75.54%</td>
      <td>91.52%</td>
      <td>83.53%</td>
    </tr>
  </tbody>
</table>
</div>



## Math scores by grade


```python
pd.DataFrame(df_students.groupby('grade').mean()['math_score']).reset_index()
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
      <th>grade</th>
      <th>math_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10th</td>
      <td>78.941483</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11th</td>
      <td>79.083548</td>
    </tr>
    <tr>
      <th>2</th>
      <td>12th</td>
      <td>78.993164</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9th</td>
      <td>78.935659</td>
    </tr>
  </tbody>
</table>
</div>



## Reading scores by grade


```python
pd.DataFrame(df_students.groupby('grade').mean()['reading_score']).reset_index()
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
      <th>grade</th>
      <th>reading_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10th</td>
      <td>81.874410</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11th</td>
      <td>81.885714</td>
    </tr>
    <tr>
      <th>2</th>
      <td>12th</td>
      <td>81.819851</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9th</td>
      <td>81.914358</td>
    </tr>
  </tbody>
</table>
</div>



## Scores by school size


```python
print("Max budget per student: $" + str(df_schools_4['Budget per student'].max()))
print("Min budget per student: $" + str(df_schools_4['Budget per student'].min()))
```

    Max budget per student: $655.0
    Min budget per student: $578.0



```python
bins=[575.,595.,615.,635.,655.]
group_bins = ["\$575 to \$595","\$595 to \$615","\$615 to \$635","\$635 to \$655"]
```


```python
df_schools_4['View Budget per student groups']=pd.cut(df_schools_4['Budget per student'].map(np.float),bins,labels=group_bins)
```


```python
df_schools_scores_budget = df_schools_4.groupby('View Budget per student groups').mean()[['mean_math_score','mean_reading_score','Percentage_passing_math','Percentage_passing_reading','Overall Passing Rate']]
df_schools_scores_budget['Percentage_passing_math'] = df_schools_scores_budget['Percentage_passing_math'].map('{:5.2f}%'.format)
df_schools_scores_budget['Percentage_passing_reading'] = df_schools_scores_budget['Percentage_passing_reading'].map('{:5.2f}%'.format)
df_schools_scores_budget['Overall Passing Rate'] = df_schools_scores_budget['Overall Passing Rate'].map('{:5.2f}%'.format)
df_schools_scores_budget
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
      <th>mean_math_score</th>
      <th>mean_reading_score</th>
      <th>Percentage_passing_math</th>
      <th>Percentage_passing_reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>View Budget per student groups</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>\$575 to \$595</th>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>\$595 to \$615</th>
      <td>83.599686</td>
      <td>83.885211</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>\$615 to \$635</th>
      <td>80.199966</td>
      <td>82.425360</td>
      <td>87.80%</td>
      <td>95.95%</td>
      <td>91.88%</td>
    </tr>
    <tr>
      <th>\$635 to \$655</th>
      <td>77.866721</td>
      <td>81.368774</td>
      <td>78.96%</td>
      <td>92.79%</td>
      <td>85.88%</td>
    </tr>
  </tbody>
</table>
</div>




```python
print("Max number of student: " + str(df_schools_4['size'].max()))
print("Min number of student: " + str(df_schools_4['size'].min()))
```

    Max number of student: 4976
    Min number of student: 427



```python
bins=[400,2000,3500,5000]
group_bins = ["Small (400 to 2000)","Medium (2000 to 3500)","Large (3500 to 5000)"]
```


```python
df_schools_4['View number of student groups']=pd.cut(df_schools_4['size'],bins,labels=group_bins)
```


```python
df_schools_scores_size = df_schools_4.groupby('View number of student groups').mean()[['mean_math_score','mean_reading_score','Percentage_passing_math','Percentage_passing_reading','Overall Passing Rate']]
df_schools_scores_size['Percentage_passing_math'] = df_schools_scores_size['Percentage_passing_math'].map('{:5.2f}%'.format)
df_schools_scores_size['Percentage_passing_reading'] = df_schools_scores_size['Percentage_passing_reading'].map('{:5.2f}%'.format)
df_schools_scores_size['Overall Passing Rate'] = df_schools_scores_size['Overall Passing Rate'].map('{:5.2f}%'.format)
df_schools_scores_size

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
      <th>mean_math_score</th>
      <th>mean_reading_score</th>
      <th>Percentage_passing_math</th>
      <th>Percentage_passing_reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>View number of student groups</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (400 to 2000)</th>
      <td>83.502373</td>
      <td>83.883125</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>Medium (2000 to 3500)</th>
      <td>78.429493</td>
      <td>81.769122</td>
      <td>81.59%</td>
      <td>93.62%</td>
      <td>87.60%</td>
    </tr>
    <tr>
      <th>Large (3500 to 5000)</th>
      <td>77.063340</td>
      <td>80.919864</td>
      <td>75.50%</td>
      <td>91.74%</td>
      <td>83.62%</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_schools_scores_type = df_schools_4.groupby('type').mean()[['mean_math_score','mean_reading_score','Percentage_passing_math','Percentage_passing_reading','Overall Passing Rate']]
df_schools_scores_type['Percentage_passing_math'] = df_schools_scores_type['Percentage_passing_math'].map('{:5.2f}%'.format)
df_schools_scores_type['Percentage_passing_reading'] = df_schools_scores_type['Percentage_passing_reading'].map('{:5.2f}%'.format)
df_schools_scores_type['Overall Passing Rate'] = df_schools_scores_type['Overall Passing Rate'].map('{:5.2f}%'.format)
df_schools_scores_type
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
      <th>mean_math_score</th>
      <th>mean_reading_score</th>
      <th>Percentage_passing_math</th>
      <th>Percentage_passing_reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>100.00%</td>
      <td>100.00%</td>
      <td>100.00%</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>75.48%</td>
      <td>91.63%</td>
      <td>83.55%</td>
    </tr>
  </tbody>
</table>
</div>


