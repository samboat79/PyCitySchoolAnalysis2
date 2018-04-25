
# ANALYSIS
TREND 1:Comparing the performance of school Types, charter schools performs better than District School.
TREND 2:There is negative correlation on school performances between total school budget and average sores
TREND 3:The size of a school also have a great influence on students scores. The smaller the class (<1000) 
the higher their overall passing Rate.                                                                                      


```python
#import all dependencies 
import pandas as pd
import os
import numpy as np
```


```python
#import files from files source
schools_file = os.path.join("schools_complete.csv")
students_file = os.path.join("students_complete.csv")
```


```python
#Reading files into dataFrame
schools_df = pd.read_csv(schools_file)
students_df = pd.read_csv(students_file)

```


```python
#Change the 'name' column of schools_df to 'school'
schools_df = schools_df.rename(columns={"name":"school"})
```

# District Summary


```python
'''
District Summary
Create high-level snapshot (in table form) of district's key metrics, including:
- total schools
- total students (formatted with thousands separators)
- total budget (formatted as currency)
- average math score 
- average reading score
- % passing math
- % passing reading
- overall passing rate (average of % passing math & % passing reading)
'''
```




    "\nDistrict Summary\nCreate high-level snapshot (in table form) of district's key metrics, including:\n- total schools\n- total students (formatted with thousands separators)\n- total budget (formatted as currency)\n- average math score \n- average reading score\n- % passing math\n- % passing reading\n- overall passing rate (average of % passing math & % passing reading)\n"




```python
#calculate total number of students by counting student ids
#use a variable instead of direct calculation because it is needed to determine 
#percentage of students passing reading and math

total_students = students_df['Student ID'].count()
total_students
```




    39170




```python
#calculate number of students with passing math scores -- create data frame with passing scores,
passing_math = students_df.loc[students_df['math_score'] >= 70, ['math_score']]
pct_passing_math = (passing_math['math_score'].count() / total_students) * 100
pct_passing_math
```




    74.9808526933878




```python
#calculate number of students with passing read scores -- create data frame with passing scores,
passing_reading = students_df.loc[students_df['reading_score'] >= 70, ['reading_score']]
pct_passing_reading = (passing_reading['reading_score'].count() / total_students) * 100

```


```python
#calculate number of students with passing read scores -- create data frame with passing scores,
passing_reading = students_df.loc[students_df['reading_score'] >= 70, ['reading_score']]
pct_passing_reading = (passing_reading['reading_score'].count() / total_students) * 100

```


```python
# Average of % passing reading and % passing match
overall_passing_rate = ((pct_passing_math + pct_passing_reading) / 2)
overall_passing_rate

```




    80.39315802910392




```python
# create district_summary dataframe for display
district_summary = pd.DataFrame({
    'Total Schools': schools_df['School ID'].count(),
    'Total Students': [total_students],
    'Total Budget': schools_df['budget'].sum(),
    'Average Math Score': students_df['math_score'].mean(),
    'Average Reading Score': students_df['reading_score'].mean(),
    '% Passing Math': [pct_passing_math],
    '% Passing Reading': [pct_passing_reading],
    'Overall Passing Rate': [overall_passing_rate]})
```


```python
# format 'Total Students' with thousands separator and 'Total Budget' with currency and thousands sep
district_summary['Total Students'] = district_summary['Total Students'].map('{:,}'.format)
district_summary['Total Budget'] = district_summary['Total Budget'].map('${:,.2f}'.format)

#reassemble summary columns in display order
district_summary = district_summary[['Total Schools', 'Total Students', 'Total Budget', 'Average Math Score',
                                    'Average Reading Score', '% Passing Math', '% Passing Reading', 
                                    'Overall Passing Rate']]

#show summary
district_summary
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
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428.00</td>
      <td>78.985371</td>
      <td>81.87784</td>
      <td>74.980853</td>
      <td>85.805463</td>
      <td>80.393158</td>
    </tr>
  </tbody>
</table>
</div>



# School Summary


```python
'''
School Summary
Create overview table that summarizes key metrics about each school:
- School name
- School type (Charter / District)
- Total school budget
- Budget per student
- Average math score
- Average reading score
- % passing math
- % passing reading
- Overall passing rate (average of % passing math & % passing reading)
'''
```




    '\nSchool Summary\nCreate overview table that summarizes key metrics about each school:\n- School name\n- School type (Charter / District)\n- Total school budget\n- Budget per student\n- Average math score\n- Average reading score\n- % passing math\n- % passing reading\n- Overall passing rate (average of % passing math & % passing reading)\n'




```python
# create dataframe containing only students who pass math
students_passing_math = students_df.loc[students_df['math_score'] > 70]
```


```python
# reduce dataframe to include only high school name and a new field that will be used by the groupby for the 
students_passing_math_red = pd.DataFrame({'school': students_passing_math['school'],
                                         'nbr_students_passing_math': 0})
```


```python
# group students passing math by high school and count. The count of students in the students_passing_math_red 
#Dataset will go in the nbr_students_passing_math column.
students_passing_math_by_school = students_passing_math_red.groupby(['school']).count().reset_index()

```


```python
# create dataframe containing only students who pass reading
students_passing_reading = students_df.loc[students_df['reading_score'] > 70]
```


```python
# reduce dataframe to include only high school and a new, empty field to contain the student counter created
# by the groupby in the next step.
students_passing_reading_red = pd.DataFrame({'school': students_passing_reading['school'],
                                          'nbr_students_passing_reading': 0})
```


```python
# group students passing reading by high school and count, reset index so all columns can be used for processing
students_passing_reading_by_school = students_passing_reading_red.groupby(['school']).count().reset_index()
```


```python
#group students by school to calculate average math and reading scores per school
school_group = students_df.groupby(['school'])
```


```python
# build dataframe with calculated average score. reset the index so all columns can be used for processing
avg_scores_df = pd.DataFrame({'avg_math_score': school_group['math_score'].mean(),
                              'avg_reading_score': school_group['reading_score'].mean()}).reset_index()
```


```python
# create 'super table' of schools_df, students passing reading, students passing math and average math scores.
super_school_df = pd.merge(schools_df, avg_scores_df, on = 'school') \
.merge(students_passing_math_by_school, on = 'school') \
.merge(students_passing_reading_by_school, on = 'school') 
```


```python
pct_students_passing_math_school = (super_school_df['nbr_students_passing_math'] / super_school_df['size']) * 100
```


```python
#calculate percentage of students passing reading
pct_students_passing_reading_school = (super_school_df['nbr_students_passing_reading'] / \
                                      super_school_df['size']) * 100

```


```python
#calculate overall passing rate
overall_passing_rate_school = ((pct_students_passing_math_school + pct_students_passing_reading_school) / 2)
```


```python
# create district_summary dataframe for display
school_summary_df = pd.DataFrame({'School Name': super_school_df['school'],
                               'School Type': super_school_df['type'],
                               'Total Students': super_school_df['size'],
                               'Total School Budget': super_school_df['budget'],
                               'Per Student Budget': (super_school_df['budget'] / super_school_df['size']),
                               'Average Math Score': super_school_df['avg_math_score'],
                               'Average Reading Score': super_school_df['avg_reading_score'],
                               '% Passing Math': pct_students_passing_math_school,
                               '% Passing Reading': pct_students_passing_reading_school,
                               'Overall Passing Rate': overall_passing_rate_school})
school_summary_df.head()
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
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Per Student Budget</th>
      <th>School Name</th>
      <th>School Type</th>
      <th>Total School Budget</th>
      <th>Total Students</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>63.318478</td>
      <td>78.813850</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>71.066164</td>
      <td>655.0</td>
      <td>Huang High School</td>
      <td>District</td>
      <td>1910635</td>
      <td>2917</td>
    </tr>
    <tr>
      <th>1</th>
      <td>63.750424</td>
      <td>78.433367</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>71.091896</td>
      <td>639.0</td>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>1884411</td>
      <td>2949</td>
    </tr>
    <tr>
      <th>2</th>
      <td>89.892107</td>
      <td>92.617831</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>91.254969</td>
      <td>600.0</td>
      <td>Shelton High School</td>
      <td>Charter</td>
      <td>1056600</td>
      <td>1761</td>
    </tr>
    <tr>
      <th>3</th>
      <td>64.746494</td>
      <td>78.187702</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>71.467098</td>
      <td>652.0</td>
      <td>Hernandez High School</td>
      <td>District</td>
      <td>3022020</td>
      <td>4635</td>
    </tr>
    <tr>
      <th>4</th>
      <td>89.713896</td>
      <td>93.392371</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>91.553134</td>
      <td>625.0</td>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>917500</td>
      <td>1468</td>
    </tr>
  </tbody>
</table>
</div>




```python
#format 'Total School Budget' & 'Per Student Budget' as currency
school_summary_df['Total School Budget'] = school_summary_df['Total School Budget'].map('${:,.2f}'.format)
school_summary_df['Per Student Budget'] = school_summary_df['Per Student Budget']
```


```python
#Reassemble columns in display order, reset index to 'School Name' and remove index label.
school_summary_df = school_summary_df[['School Name', 'School Type', 'Total Students', 'Total School Budget',
                                 'Per Student Budget', 'Average Math Score', 'Average Reading Score',
                                 '% Passing Math', '% Passing Reading', 'Overall Passing Rate']] \
                 .set_index('School Name').rename_axis(None)

# display school summary
school_summary_df
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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>63.318478</td>
      <td>78.813850</td>
      <td>71.066164</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>63.750424</td>
      <td>78.433367</td>
      <td>71.091896</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>$1,056,600.00</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>89.892107</td>
      <td>92.617831</td>
      <td>91.254969</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>64.746494</td>
      <td>78.187702</td>
      <td>71.467098</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500.00</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>89.713896</td>
      <td>93.392371</td>
      <td>91.553134</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>90.932983</td>
      <td>93.254490</td>
      <td>92.093736</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>89.558665</td>
      <td>93.864370</td>
      <td>91.711518</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>$3,124,928.00</td>
      <td>628.0</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>64.630225</td>
      <td>79.300643</td>
      <td>71.965434</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>90.632319</td>
      <td>92.740047</td>
      <td>91.686183</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>91.683992</td>
      <td>92.203742</td>
      <td>91.943867</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400.00</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>90.277778</td>
      <td>93.444444</td>
      <td>91.861111</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>64.066017</td>
      <td>77.744436</td>
      <td>70.905226</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>63.852132</td>
      <td>78.281874</td>
      <td>71.067003</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916.00</td>
      <td>644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>65.753925</td>
      <td>77.510040</td>
      <td>71.631982</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130.00</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>90.214067</td>
      <td>92.905199</td>
      <td>91.559633</td>
    </tr>
  </tbody>
</table>
</div>



# Top Performing School


```python
'''
Top Performing Schools (By Passing Rate)
Highlight the 5 top performing schools based on Overall Passing Rate
- School Name
- School type (Charter / District)
- Total students
- Total school budget
- Budget per student
- Average Math, Reading Scores
- % Passing Math and Reading (score > 70)
- Overall Passing Rate
'''
```




    '\nTop Performing Schools (By Passing Rate)\nHighlight the 5 top performing schools based on Overall Passing Rate\n- School Name\n- School type (Charter / District)\n- Total students\n- Total school budget\n- Budget per student\n- Average Math, Reading Scores\n- % Passing Math and Reading (score > 70)\n- Overall Passing Rate\n'




```python
# use nlargest on school summary created in previous step to find top 5
top_performing_schools_df = school_summary_df.nlargest(5, 'Overall Passing Rate')

```


```python
#show dataframe
top_performing_schools_df
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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>90.932983</td>
      <td>93.254490</td>
      <td>92.093736</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>91.683992</td>
      <td>92.203742</td>
      <td>91.943867</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400.00</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>90.277778</td>
      <td>93.444444</td>
      <td>91.861111</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>89.558665</td>
      <td>93.864370</td>
      <td>91.711518</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>90.632319</td>
      <td>92.740047</td>
      <td>91.686183</td>
    </tr>
  </tbody>
</table>
</div>



# Bottom Performing Schools (By Passing Rate)


```python
'''
Bottom Performing Schools (By Passing Rate)
Highlight the 5 worst performing schools based on Overall Passing Rate
- School Name
- School type (Charter / District)
- Total students
- Total school budget
- Budget per student
- Average Math, Reading Scores
- % Passing Math and Reading (score > 70)
- Overall Passing Rate
'''
```




    '\nBottom Performing Schools (By Passing Rate)\nHighlight the 5 worst performing schools based on Overall Passing Rate\n- School Name\n- School type (Charter / District)\n- Total students\n- Total school budget\n- Budget per student\n- Average Math, Reading Scores\n- % Passing Math and Reading (score > 70)\n- Overall Passing Rate\n'




```python
# use nsmallest on school summary created above to find bottom 5
bottom_performing_schools_df = school_summary_df.nsmallest(5, 'Overall Passing Rate')
```


```python
#show dataframe
bottom_performing_schools_df
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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>64.066017</td>
      <td>77.744436</td>
      <td>70.905226</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>63.318478</td>
      <td>78.813850</td>
      <td>71.066164</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>63.852132</td>
      <td>78.281874</td>
      <td>71.067003</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>63.750424</td>
      <td>78.433367</td>
      <td>71.091896</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>64.746494</td>
      <td>78.187702</td>
      <td>71.467098</td>
    </tr>
  </tbody>
</table>
</div>



# Math Score By Grade


```python
''''
Math Scores By Grade
For each school, display average math scores by grade, one grade per column
'''

def build_grade_dataframe(grade_criteria):
    '''
    Creates initial grade-level dataframes for math and reading score processing
    args:
    grade_criteria: string, the grade level to be included in dataset
    '''
    return avg_scores_by_high_school_df.loc[avg_scores_by_high_school_df['grade'] == grade_criteria]
```


```python
#use groupby to create a dataset grouped by high school and grade, then average the columns and reset the indexes
avg_scores_by_high_school_df = students_df.groupby(['school', 'grade']).mean().reset_index()

```


```python
#place each grade in its own dataframe
ninth_grade_df = build_grade_dataframe('9th')
tenth_grade_df = build_grade_dataframe('10th')
eleventh_grade_df = build_grade_dataframe('11th')
twelfth_grade_df = build_grade_dataframe('12th')
```


```python
#reduce grade dataframes to school, math score, reading score
ninth_grade_red_df = pd.DataFrame({"school": ninth_grade_df["school"],
                                   "reading_score_9th": ninth_grade_df['reading_score'],
                                   "math_score_9th": ninth_grade_df['math_score']})
tenth_grade_red_df = pd.DataFrame({"school": tenth_grade_df["school"],
                                   "reading_score_10th": tenth_grade_df['reading_score'],
                                   "math_score_10th": tenth_grade_df['math_score']})
eleventh_grade_red_df = pd.DataFrame({"school": eleventh_grade_df["school"],
                                   "reading_score_11th": eleventh_grade_df['reading_score'],
                                   "math_score_11th": eleventh_grade_df['math_score']})
twelfth_grade_red_df = pd.DataFrame({"school": twelfth_grade_df["school"],
                                   "reading_score_12th": twelfth_grade_df['reading_score'],
                                   "math_score_12th": twelfth_grade_df['math_score']})
                                                                       
```


```python
#merge the dataframes into a single dataframe
all_scores_by_grade_hs_df = pd.merge(ninth_grade_red_df, tenth_grade_red_df, on = 'school')\
.merge(eleventh_grade_red_df, on = 'school')\
.merge(twelfth_grade_red_df, on = 'school') 
```


```python
#create new display dataframe
math_scores_by_grade_df = pd.DataFrame({'school': all_scores_by_grade_hs_df['school'],
                                        '9th': all_scores_by_grade_hs_df['math_score_9th'],
                                        '10th': all_scores_by_grade_hs_df['math_score_10th'],
                                        '11th': all_scores_by_grade_hs_df['math_score_11th'],
                                        '12th': all_scores_by_grade_hs_df['math_score_12th']})
```


```python
#reassemble columns in display order, set index to school name and delete label
math_scores_by_grade_df = math_scores_by_grade_df[['school', '9th', '10th', '11th', '12th']]\
                 .set_index('school').rename_axis(None)
```


```python
#display dataframe
math_scores_by_grade_df
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
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>



# Reading Score By Grade


```python
'''
Reading Scores By Grade
For each school, display average reading scores by grade, one grade per column
Uses combined math/reading score dataset created in 'Math Scores By Grade' summary
'''
```




    "\nReading Scores By Grade\nFor each school, display average reading scores by grade, one grade per column\nUses combined math/reading score dataset created in 'Math Scores By Grade' summary\n"




```python
#Create new dataframe of reading scores
reading_scores_by_grade_df = pd.DataFrame({'school': all_scores_by_grade_hs_df['school'],
                                        '9th': all_scores_by_grade_hs_df['reading_score_9th'],
                                        '10th': all_scores_by_grade_hs_df['reading_score_10th'],
                                        '11th': all_scores_by_grade_hs_df['reading_score_11th'],
                                        '12th': all_scores_by_grade_hs_df['reading_score_12th']})
```


```python
#Reassemble columns in display order, set index to school and delete index label
reading_scores_by_grade_df = reading_scores_by_grade_df[['school', '9th', '10th', '11th', '12th']]\
                 .set_index('school').rename_axis(None)
    
#display dataframe
reading_scores_by_grade_df
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
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>



# Score by School Spending


```python
'''
Scores by School Spending
School performance by average spending per student
- Avg math and reading scores
- % passing math and reading
- overall passing rate
- categorized spending range (per student) -- >$585, $585-$615, $615-$645, $645-$675
Based on school_summary_df dataframe from 'School Summary' step
'''
```




    "\nScores by School Spending\nSchool performance by average spending per student\n- Avg math and reading scores\n- % passing math and reading\n- overall passing rate\n- categorized spending range (per student) -- >$585, $585-$615, $615-$645, $645-$675\nBased on school_summary_df dataframe from 'School Summary' step\n"




```python
#dataframe
school_summary_spending = school_summary_df.copy()
school_summary_spending
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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>63.318478</td>
      <td>78.813850</td>
      <td>71.066164</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>63.750424</td>
      <td>78.433367</td>
      <td>71.091896</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>$1,056,600.00</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>89.892107</td>
      <td>92.617831</td>
      <td>91.254969</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>64.746494</td>
      <td>78.187702</td>
      <td>71.467098</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500.00</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>89.713896</td>
      <td>93.392371</td>
      <td>91.553134</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>90.932983</td>
      <td>93.254490</td>
      <td>92.093736</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>89.558665</td>
      <td>93.864370</td>
      <td>91.711518</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>$3,124,928.00</td>
      <td>628.0</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>64.630225</td>
      <td>79.300643</td>
      <td>71.965434</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>90.632319</td>
      <td>92.740047</td>
      <td>91.686183</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>91.683992</td>
      <td>92.203742</td>
      <td>91.943867</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400.00</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>90.277778</td>
      <td>93.444444</td>
      <td>91.861111</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>64.066017</td>
      <td>77.744436</td>
      <td>70.905226</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>63.852132</td>
      <td>78.281874</td>
      <td>71.067003</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916.00</td>
      <td>644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>65.753925</td>
      <td>77.510040</td>
      <td>71.631982</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130.00</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>90.214067</td>
      <td>92.905199</td>
      <td>91.559633</td>
    </tr>
  </tbody>
</table>
</div>




```python
#create bins to hold values
bins = [0, 585, 615, 645, 675]


#Remove $ from 'Per Student Budget' and cast as a float to allow calculations using that column
school_summary_spending['Per Student Budget'] = school_summary_spending['Per Student Budget']\
.replace( '[\$]','',regex=True).astype('float')
```


```python
#create groups for each bin
view_groups = ['>$585', '\$585-\$615', '\$615-\$645', '\$645-\$675']
```


```python
#create new column to hold spending range category for each school
school_summary_spending['Spending Ranges (Per Student)'] = pd.cut(school_summary_spending['Per Student Budget'],
                                                                  bins, labels=view_groups)

```


```python
#Use selected columns to build dataset for display
school_summary_spending = school_summary_spending[['Average Math Score', 'Average Reading Score',
                                          '% Passing Math', '% Passing Reading', 'Overall Passing Rate',
                                          'Spending Ranges (Per Student)']]
```


```python
#group by Spending Range and average all scores for summary
scores_by_school_spending_df = school_summary_spending.groupby('Spending Ranges (Per Student)').mean()
```


```python
#display dataframe
scores_by_school_spending_df
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&gt;$585</th>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>90.350436</td>
      <td>93.325838</td>
      <td>91.838137</td>
    </tr>
    <tr>
      <th>\$585-\$615</th>
      <td>83.599686</td>
      <td>83.885211</td>
      <td>90.788049</td>
      <td>92.410786</td>
      <td>91.599418</td>
    </tr>
    <tr>
      <th>\$615-\$645</th>
      <td>79.079225</td>
      <td>81.891436</td>
      <td>73.021426</td>
      <td>83.214343</td>
      <td>78.117884</td>
    </tr>
    <tr>
      <th>\$645-\$675</th>
      <td>76.997210</td>
      <td>81.027843</td>
      <td>63.972368</td>
      <td>78.427809</td>
      <td>71.200088</td>
    </tr>
  </tbody>
</table>
</div>



# Scores By School Size


```python
'''
Scores by School Size
School performance by number of students
- Avg math and reading scores
- % passing math and reading
- overall passing rate
- categorized spending range (per student) -- Small(<1000), Medium(1000-2000), Large(2000-5000)
Based on school_summary_df dataframe from 'School Summary' step
'''
```




    "\nScores by School Size\nSchool performance by number of students\n- Avg math and reading scores\n- % passing math and reading\n- overall passing rate\n- categorized spending range (per student) -- Small(<1000), Medium(1000-2000), Large(2000-5000)\nBased on school_summary_df dataframe from 'School Summary' step\n"




```python
#school_summary_size is a deep copy of school_summary_df, so changes to this dataframe do not effect the original
#dataframe
school_summary_size = school_summary_df.copy()
school_summary_size

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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>63.318478</td>
      <td>78.813850</td>
      <td>71.066164</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>63.750424</td>
      <td>78.433367</td>
      <td>71.091896</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>$1,056,600.00</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>89.892107</td>
      <td>92.617831</td>
      <td>91.254969</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>64.746494</td>
      <td>78.187702</td>
      <td>71.467098</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500.00</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>89.713896</td>
      <td>93.392371</td>
      <td>91.553134</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>90.932983</td>
      <td>93.254490</td>
      <td>92.093736</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>89.558665</td>
      <td>93.864370</td>
      <td>91.711518</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>$3,124,928.00</td>
      <td>628.0</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>64.630225</td>
      <td>79.300643</td>
      <td>71.965434</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>90.632319</td>
      <td>92.740047</td>
      <td>91.686183</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>91.683992</td>
      <td>92.203742</td>
      <td>91.943867</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400.00</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>90.277778</td>
      <td>93.444444</td>
      <td>91.861111</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>64.066017</td>
      <td>77.744436</td>
      <td>70.905226</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>63.852132</td>
      <td>78.281874</td>
      <td>71.067003</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916.00</td>
      <td>644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>65.753925</td>
      <td>77.510040</td>
      <td>71.631982</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130.00</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>90.214067</td>
      <td>92.905199</td>
      <td>91.559633</td>
    </tr>
  </tbody>
</table>
</div>




```python
#create bins to hold values
bins = [0, 1000, 2000, 5000]
```


```python
#create groups for each bin
view_groups = ['Small (<1000)', 'Medium (1000-2000)', 'Large (2000-5000)']
view_groups
```




    ['Small (<1000)', 'Medium (1000-2000)', 'Large (2000-5000)']




```python
#cast school size to numeric to allow for calculations
school_summary_size['Total Students'] = pd.to_numeric(school_summary_size['Total Students'])
```


```python
#categorize school size into Small/Medium/Large bins
school_summary_size['School Size'] = pd.cut(school_summary_size['Total Students'],
                                                                  bins, labels=view_groups)
```


```python
#Assemble selected columns for summary
school_summary_size = school_summary_size[['Average Math Score', 'Average Reading Score',
                                          '% Passing Math', '% Passing Reading', 'Overall Passing Rate',
                                          'School Size']]

```


```python
#group by size bins and average scores
scores_by_school_size_df = school_summary_size.groupby('School Size').mean()

#display dataframe
scores_by_school_size_df
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt;1000)</th>
      <td>83.821598</td>
      <td>83.929843</td>
      <td>91.158155</td>
      <td>92.471895</td>
      <td>91.815025</td>
    </tr>
    <tr>
      <th>Medium (1000-2000)</th>
      <td>83.374684</td>
      <td>83.864438</td>
      <td>89.931303</td>
      <td>93.244843</td>
      <td>91.588073</td>
    </tr>
    <tr>
      <th>Large (2000-5000)</th>
      <td>77.746417</td>
      <td>81.344493</td>
      <td>67.631335</td>
      <td>80.190800</td>
      <td>73.911067</td>
    </tr>
  </tbody>
</table>
</div>




```python
'''
Scores by School Type (District/Charter)
School performance by school type
- school type
- Avg math and reading scores
- % passing math and reading
- overall passing rate
Based on school_summary_df dataframe from 'School Summary' step
'''
```




    "\nScores by School Type (District/Charter)\nSchool performance by school type\n- school type\n- Avg math and reading scores\n- % passing math and reading\n- overall passing rate\nBased on school_summary_df dataframe from 'School Summary' step\n"




```python
#school_summary_type is a deep copy of school_summary_df, so changes to this dataframe do not effect the original
#dataframe
school_summary_type = school_summary_df.copy()
```


```python
#assemble selected columns for display
school_summary_type = school_summary_type[['School Type','Average Math Score', 'Average Reading Score',
                                          '% Passing Math', '% Passing Reading', 'Overall Passing Rate']]
```


```python
#group by school type and average columns
scores_by_school_type_df = school_summary_type.groupby('School Type').mean()

#display dataframe
scores_by_school_type_df
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Type</th>
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
      <td>90.363226</td>
      <td>93.052812</td>
      <td>91.708019</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>64.302528</td>
      <td>78.324559</td>
      <td>71.313543</td>
    </tr>
  </tbody>
</table>
</div>


