
# PyCity Schools Analysis

+ Average math and reading scores stay consistent across grade level when grouped by school.  There is no major improvement in scores from any school.
+ Math passing rates are always consistently lower across every metric, but the difference between math and reading passing rates is greater amoung lower performing schools, large schools, and higher spending per student which all seem to correlate.  
+ The top 5 schools are all charter schools while the bottom 5 all district schools. 
+ In general (one exception), per student spending is higher in bottom performing schools than top performing.  
+ Schools under 2000 student have much higher passing rates than those with student populations above 2000.  A comparision of 95 to 75%.  The same phenomenon is seen with high and low per student spending brackets and district versus charter schools.  






```python
#Dependencies
import pandas as pd
import numpy as np
import os

# define file path
schools_file = os.path.join('Resources','schools_complete.csv')
students_file = os.path.join('Resources', 'students_complete.csv')

# read schools file
schools_df = pd.read_csv(schools_file)

#read student file
students_df = pd.read_csv(students_file)
```

## District Summary


```python
#create array of unique school names
unique_school_names = schools_df['name'].unique()
#gives the length of unique school names to give us how many schools
school_count = len(unique_school_names)

#district student count
dist_student_count = schools_df['size'].sum()

#student count from student file (to verify with district student count)
total_student_rec = students_df['name'].count()

#total budget
total_budget = schools_df['budget'].sum()

#calculations for number and % passing reading
num_passing_reading = students_df.loc[students_df['reading_score'] >= 70]['reading_score'].count()
perc_pass_reading = num_passing_reading/total_student_rec
perc_pass_reading

#calculations for number and % passing math
num_passing_math = students_df.loc[students_df['math_score'] >= 70]['math_score'].count()
perc_pass_math = num_passing_math/total_student_rec
perc_pass_math

#average math score calculation
avg_math_score = students_df['math_score'].mean()
avg_math_score 

#average reading score calculation
avg_reading_score = students_df['reading_score'].mean()
avg_reading_score

#Overall Passing Rate Calculations
overall_pass = np.mean([perc_pass_reading, perc_pass_math])

# district dataframe from dictionary

district_summary = pd.DataFrame({
    
    "Total Schools": [school_count],
    "Total Students": [dist_student_count],
    "Total Budget": [total_budget],
    "Average Reading Score": [avg_reading_score],
    "Average Math Score": [avg_math_score],
    "% Passing Reading":[perc_pass_reading],
    "% Passing Math": [perc_pass_math],
    "Overall Passing Rate": [overall_pass]

})

#store as different df to change order
dist_sum = district_summary[["Total Schools", "Total Students", "Total Budget", "Average Reading Score", "Average Math Score", '% Passing Reading', '% Passing Math', 'Overall Passing Rate']]

#format cells
dist_sum.style.format({"Total Budget": "${:,.2f}", "Average Reading Score": "{:.1f}", "Average Math Score": "{:.1f}", "% Passing Math": "{:.1%}", "% Passing Reading": "{:.1%}", "Overall Passing Rate": "{:.1%}"})
```




<style  type="text/css" >
</style>  
<table id="T_b489ad64_8f6e_11e7_ba76_d4619d1535aa" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Total Schools</th> 
        <th class="col_heading level0 col1" >Total Students</th> 
        <th class="col_heading level0 col2" >Total Budget</th> 
        <th class="col_heading level0 col3" >Average Reading Score</th> 
        <th class="col_heading level0 col4" >Average Math Score</th> 
        <th class="col_heading level0 col5" >% Passing Reading</th> 
        <th class="col_heading level0 col6" >% Passing Math</th> 
        <th class="col_heading level0 col7" >Overall Passing Rate</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_b489ad64_8f6e_11e7_ba76_d4619d1535aa" class="row_heading level0 row0" >0</th> 
        <td id="T_b489ad64_8f6e_11e7_ba76_d4619d1535aarow0_col0" class="data row0 col0" >15</td> 
        <td id="T_b489ad64_8f6e_11e7_ba76_d4619d1535aarow0_col1" class="data row0 col1" >39170</td> 
        <td id="T_b489ad64_8f6e_11e7_ba76_d4619d1535aarow0_col2" class="data row0 col2" >$24,649,428.00</td> 
        <td id="T_b489ad64_8f6e_11e7_ba76_d4619d1535aarow0_col3" class="data row0 col3" >81.9</td> 
        <td id="T_b489ad64_8f6e_11e7_ba76_d4619d1535aarow0_col4" class="data row0 col4" >79.0</td> 
        <td id="T_b489ad64_8f6e_11e7_ba76_d4619d1535aarow0_col5" class="data row0 col5" >85.8%</td> 
        <td id="T_b489ad64_8f6e_11e7_ba76_d4619d1535aarow0_col6" class="data row0 col6" >75.0%</td> 
        <td id="T_b489ad64_8f6e_11e7_ba76_d4619d1535aarow0_col7" class="data row0 col7" >80.4%</td> 
    </tr></tbody> 
</table> 



## School Summary


```python
#groups by school
by_school = students_df.groupby(['school'])

#counts students per school and creates DataFrame
# students_per_school = pd.DataFrame([by_school['Student ID'].count(), by_school)
                                  
#adds budget per student
schools_df['Per Student Budget'] = schools_df['budget']/schools_df['size']

schools_df.rename(columns = {'name': 'school'}, inplace = True)

#creates dataframe of avg math and reading score by school
avg_math_by_sch = by_school['math_score'].mean().round(1).reset_index()
avg_read_by_sch = by_school['reading_score'].mean().round(1).reset_index()
avg_scores = pd.merge(avg_math_by_sch, avg_read_by_sch, on=('school'))
avg_scores.rename(columns = {'math_score': 'Average Math Score', 'reading_score': 'Average Reading Score'}, inplace=True)

#school level passing scores counts by using conditional and only keeping school and math score
pass_math = students_df.loc[students_df['math_score'] >=70][['school', 'math_score']]
pass_math_by_sch = pass_math.groupby('school').count().reset_index()
pass_math_by_sch.rename(columns = {"math_score": "# passing math"}, inplace=True)

#same as above for reading
pass_read = students_df.loc[students_df['reading_score'] >=70][['school', 'reading_score']]
pass_read_by_sch = pass_read.groupby('school').count().reset_index()
pass_read_by_sch.rename(columns = {"reading_score": "# passing reading"}, inplace=True)

#merge math and reading data
pass_count = pd.merge(pass_math_by_sch, pass_read_by_sch, on=('school'))


#merge all so far on school
sch_summary = pd.merge(schools_df, avg_scores, on=('school'))
sch_summary = pd.merge(sch_summary, pass_count, on=('school'))


#add percent passing columns
sch_summary['% Passing Math'] = sch_summary['# passing math']/sch_summary['size']
sch_summary['% Passing Reading'] = sch_summary['# passing reading']/sch_summary['size']

#delete extraneous columns
del sch_summary['# passing math']
del sch_summary['# passing reading']

# create Overall Passing Rat columns
sch_summary['Overall Passing Rate'] = (sch_summary['% Passing Math']+sch_summary['% Passing Reading'])/2
#formatting and adjustments for aesthetics
sch_summary.rename(columns = {'school': "School Name", "type": "School Type", "size":"Total Students", "budget": "Total School Budget"}, inplace = True)
sch_summary.set_index('School Name', inplace=True)
sch_summary.style.format({'Total Students': '{:,}', "Total School Budget": "${:,}", "Per Student Budget": "${:.0f}", "% Passing Math": "{:.1%}", "% Passing Reading": "{:.1%}", "Overall Passing Rate": "{:.1%}"})

```




<style  type="text/css" >
</style>  
<table id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >School ID</th> 
        <th class="col_heading level0 col1" >School Type</th> 
        <th class="col_heading level0 col2" >Total Students</th> 
        <th class="col_heading level0 col3" >Total School Budget</th> 
        <th class="col_heading level0 col4" >Per Student Budget</th> 
        <th class="col_heading level0 col5" >Average Math Score</th> 
        <th class="col_heading level0 col6" >Average Reading Score</th> 
        <th class="col_heading level0 col7" >% Passing Math</th> 
        <th class="col_heading level0 col8" >% Passing Reading</th> 
        <th class="col_heading level0 col9" >Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >School Name</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row0" >Huang High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow0_col0" class="data row0 col0" >0</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow0_col1" class="data row0 col1" >District</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow0_col2" class="data row0 col2" >2,917</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow0_col3" class="data row0 col3" >$1,910,635</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow0_col4" class="data row0 col4" >$655</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow0_col5" class="data row0 col5" >76.6</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow0_col6" class="data row0 col6" >81.2</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow0_col7" class="data row0 col7" >65.7%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow0_col8" class="data row0 col8" >81.3%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow0_col9" class="data row0 col9" >73.5%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row1" >Figueroa High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow1_col0" class="data row1 col0" >1</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow1_col1" class="data row1 col1" >District</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow1_col2" class="data row1 col2" >2,949</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow1_col3" class="data row1 col3" >$1,884,411</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow1_col4" class="data row1 col4" >$639</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow1_col5" class="data row1 col5" >76.7</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow1_col6" class="data row1 col6" >81.2</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow1_col7" class="data row1 col7" >66.0%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow1_col8" class="data row1 col8" >80.7%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow1_col9" class="data row1 col9" >73.4%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row2" >Shelton High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow2_col0" class="data row2 col0" >2</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow2_col1" class="data row2 col1" >Charter</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow2_col2" class="data row2 col2" >1,761</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow2_col3" class="data row2 col3" >$1,056,600</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow2_col4" class="data row2 col4" >$600</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow2_col5" class="data row2 col5" >83.4</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow2_col6" class="data row2 col6" >83.7</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow2_col7" class="data row2 col7" >93.9%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow2_col8" class="data row2 col8" >95.9%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow2_col9" class="data row2 col9" >94.9%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row3" >Hernandez High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow3_col0" class="data row3 col0" >3</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow3_col1" class="data row3 col1" >District</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow3_col2" class="data row3 col2" >4,635</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow3_col3" class="data row3 col3" >$3,022,020</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow3_col4" class="data row3 col4" >$652</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow3_col5" class="data row3 col5" >77.3</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow3_col6" class="data row3 col6" >80.9</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow3_col7" class="data row3 col7" >66.8%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow3_col8" class="data row3 col8" >80.9%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow3_col9" class="data row3 col9" >73.8%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row4" >Griffin High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow4_col0" class="data row4 col0" >4</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow4_col1" class="data row4 col1" >Charter</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow4_col2" class="data row4 col2" >1,468</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow4_col3" class="data row4 col3" >$917,500</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow4_col4" class="data row4 col4" >$625</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow4_col5" class="data row4 col5" >83.4</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow4_col6" class="data row4 col6" >83.8</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow4_col7" class="data row4 col7" >93.4%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow4_col8" class="data row4 col8" >97.1%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow4_col9" class="data row4 col9" >95.3%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row5" >Wilson High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow5_col0" class="data row5 col0" >5</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow5_col1" class="data row5 col1" >Charter</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow5_col2" class="data row5 col2" >2,283</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow5_col3" class="data row5 col3" >$1,319,574</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow5_col4" class="data row5 col4" >$578</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow5_col5" class="data row5 col5" >83.3</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow5_col6" class="data row5 col6" >84</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow5_col7" class="data row5 col7" >93.9%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow5_col8" class="data row5 col8" >96.5%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow5_col9" class="data row5 col9" >95.2%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row6" >Cabrera High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow6_col0" class="data row6 col0" >6</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow6_col1" class="data row6 col1" >Charter</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow6_col2" class="data row6 col2" >1,858</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow6_col3" class="data row6 col3" >$1,081,356</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow6_col4" class="data row6 col4" >$582</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow6_col5" class="data row6 col5" >83.1</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow6_col6" class="data row6 col6" >84</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow6_col7" class="data row6 col7" >94.1%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow6_col8" class="data row6 col8" >97.0%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow6_col9" class="data row6 col9" >95.6%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row7" >Bailey High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow7_col0" class="data row7 col0" >7</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow7_col1" class="data row7 col1" >District</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow7_col2" class="data row7 col2" >4,976</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow7_col3" class="data row7 col3" >$3,124,928</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow7_col4" class="data row7 col4" >$628</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow7_col5" class="data row7 col5" >77</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow7_col6" class="data row7 col6" >81</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow7_col7" class="data row7 col7" >66.7%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow7_col8" class="data row7 col8" >81.9%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow7_col9" class="data row7 col9" >74.3%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row8" >Holden High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow8_col0" class="data row8 col0" >8</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow8_col1" class="data row8 col1" >Charter</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow8_col2" class="data row8 col2" >427</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow8_col3" class="data row8 col3" >$248,087</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow8_col4" class="data row8 col4" >$581</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow8_col5" class="data row8 col5" >83.8</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow8_col6" class="data row8 col6" >83.8</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow8_col7" class="data row8 col7" >92.5%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow8_col8" class="data row8 col8" >96.3%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow8_col9" class="data row8 col9" >94.4%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row9" >Pena High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow9_col0" class="data row9 col0" >9</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow9_col1" class="data row9 col1" >Charter</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow9_col2" class="data row9 col2" >962</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow9_col3" class="data row9 col3" >$585,858</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow9_col4" class="data row9 col4" >$609</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow9_col5" class="data row9 col5" >83.8</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow9_col6" class="data row9 col6" >84</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow9_col7" class="data row9 col7" >94.6%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow9_col8" class="data row9 col8" >95.9%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow9_col9" class="data row9 col9" >95.3%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row10" >Wright High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow10_col0" class="data row10 col0" >10</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow10_col1" class="data row10 col1" >Charter</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow10_col2" class="data row10 col2" >1,800</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow10_col3" class="data row10 col3" >$1,049,400</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow10_col4" class="data row10 col4" >$583</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow10_col5" class="data row10 col5" >83.7</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow10_col6" class="data row10 col6" >84</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow10_col7" class="data row10 col7" >93.3%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow10_col8" class="data row10 col8" >96.6%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow10_col9" class="data row10 col9" >95.0%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row11" >Rodriguez High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow11_col0" class="data row11 col0" >11</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow11_col1" class="data row11 col1" >District</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow11_col2" class="data row11 col2" >3,999</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow11_col3" class="data row11 col3" >$2,547,363</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow11_col4" class="data row11 col4" >$637</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow11_col5" class="data row11 col5" >76.8</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow11_col6" class="data row11 col6" >80.7</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow11_col7" class="data row11 col7" >66.4%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow11_col8" class="data row11 col8" >80.2%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow11_col9" class="data row11 col9" >73.3%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row12" >Johnson High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow12_col0" class="data row12 col0" >12</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow12_col1" class="data row12 col1" >District</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow12_col2" class="data row12 col2" >4,761</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow12_col3" class="data row12 col3" >$3,094,650</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow12_col4" class="data row12 col4" >$650</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow12_col5" class="data row12 col5" >77.1</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow12_col6" class="data row12 col6" >81</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow12_col7" class="data row12 col7" >66.1%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow12_col8" class="data row12 col8" >81.2%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow12_col9" class="data row12 col9" >73.6%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row13" >Ford High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow13_col0" class="data row13 col0" >13</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow13_col1" class="data row13 col1" >District</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow13_col2" class="data row13 col2" >2,739</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow13_col3" class="data row13 col3" >$1,763,916</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow13_col4" class="data row13 col4" >$644</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow13_col5" class="data row13 col5" >77.1</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow13_col6" class="data row13 col6" >80.7</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow13_col7" class="data row13 col7" >68.3%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow13_col8" class="data row13 col8" >79.3%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow13_col9" class="data row13 col9" >73.8%</td> 
    </tr>    <tr> 
        <th id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aa" class="row_heading level0 row14" >Thomas High School</th> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow14_col0" class="data row14 col0" >14</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow14_col1" class="data row14 col1" >Charter</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow14_col2" class="data row14 col2" >1,635</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow14_col3" class="data row14 col3" >$1,043,130</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow14_col4" class="data row14 col4" >$638</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow14_col5" class="data row14 col5" >83.4</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow14_col6" class="data row14 col6" >83.8</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow14_col7" class="data row14 col7" >93.3%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow14_col8" class="data row14 col8" >97.3%</td> 
        <td id="T_6f476a38_8f6f_11e7_8f52_d4619d1535aarow14_col9" class="data row14 col9" >95.3%</td> 
    </tr></tbody> 
</table> 



## Top Performing Schools by Passing Rate


```python
# sort values by passing rate and then only print top 5 
top_5 = sch_summary.sort_values("Overall Passing Rate", ascending = False)
top_5.head().style.format({'Total Students': '{:,}', "Total School Budget": "${:,}", "Per Student Budget": "${:.0f}", "% Passing Math": "{:.1%}", "% Passing Reading": "{:.1%}", "Overall Passing Rate": "{:.1%}"})
```




<style  type="text/css" >
</style>  
<table id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aa" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >School ID</th> 
        <th class="col_heading level0 col1" >School Type</th> 
        <th class="col_heading level0 col2" >Total Students</th> 
        <th class="col_heading level0 col3" >Total School Budget</th> 
        <th class="col_heading level0 col4" >Per Student Budget</th> 
        <th class="col_heading level0 col5" >Average Math Score</th> 
        <th class="col_heading level0 col6" >Average Reading Score</th> 
        <th class="col_heading level0 col7" >% Passing Math</th> 
        <th class="col_heading level0 col8" >% Passing Reading</th> 
        <th class="col_heading level0 col9" >Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >School Name</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aa" class="row_heading level0 row0" >Cabrera High School</th> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow0_col0" class="data row0 col0" >6</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow0_col1" class="data row0 col1" >Charter</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow0_col2" class="data row0 col2" >1,858</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow0_col3" class="data row0 col3" >$1,081,356</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow0_col4" class="data row0 col4" >$582</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow0_col5" class="data row0 col5" >83.1</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow0_col6" class="data row0 col6" >84</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow0_col7" class="data row0 col7" >94.1%</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow0_col8" class="data row0 col8" >97.0%</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow0_col9" class="data row0 col9" >95.6%</td> 
    </tr>    <tr> 
        <th id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aa" class="row_heading level0 row1" >Thomas High School</th> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow1_col0" class="data row1 col0" >14</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow1_col1" class="data row1 col1" >Charter</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow1_col2" class="data row1 col2" >1,635</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow1_col3" class="data row1 col3" >$1,043,130</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow1_col4" class="data row1 col4" >$638</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow1_col5" class="data row1 col5" >83.4</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow1_col6" class="data row1 col6" >83.8</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow1_col7" class="data row1 col7" >93.3%</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow1_col8" class="data row1 col8" >97.3%</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow1_col9" class="data row1 col9" >95.3%</td> 
    </tr>    <tr> 
        <th id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aa" class="row_heading level0 row2" >Pena High School</th> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow2_col0" class="data row2 col0" >9</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow2_col1" class="data row2 col1" >Charter</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow2_col2" class="data row2 col2" >962</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow2_col3" class="data row2 col3" >$585,858</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow2_col4" class="data row2 col4" >$609</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow2_col5" class="data row2 col5" >83.8</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow2_col6" class="data row2 col6" >84</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow2_col7" class="data row2 col7" >94.6%</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow2_col8" class="data row2 col8" >95.9%</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow2_col9" class="data row2 col9" >95.3%</td> 
    </tr>    <tr> 
        <th id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aa" class="row_heading level0 row3" >Griffin High School</th> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow3_col0" class="data row3 col0" >4</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow3_col1" class="data row3 col1" >Charter</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow3_col2" class="data row3 col2" >1,468</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow3_col3" class="data row3 col3" >$917,500</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow3_col4" class="data row3 col4" >$625</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow3_col5" class="data row3 col5" >83.4</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow3_col6" class="data row3 col6" >83.8</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow3_col7" class="data row3 col7" >93.4%</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow3_col8" class="data row3 col8" >97.1%</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow3_col9" class="data row3 col9" >95.3%</td> 
    </tr>    <tr> 
        <th id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aa" class="row_heading level0 row4" >Wilson High School</th> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow4_col0" class="data row4 col0" >5</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow4_col1" class="data row4 col1" >Charter</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow4_col2" class="data row4 col2" >2,283</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow4_col3" class="data row4 col3" >$1,319,574</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow4_col4" class="data row4 col4" >$578</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow4_col5" class="data row4 col5" >83.3</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow4_col6" class="data row4 col6" >84</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow4_col7" class="data row4 col7" >93.9%</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow4_col8" class="data row4 col8" >96.5%</td> 
        <td id="T_6fb2535c_8f6f_11e7_982b_d4619d1535aarow4_col9" class="data row4 col9" >95.2%</td> 
    </tr></tbody> 
</table> 



## Bottom Performing Schools by Passing Rate


```python
#bottom 5 schools from worse to best
#take tail of top5 sort and re-sort from worst to best
bottom_5 = top_5.tail()
bottom_5 = bottom_5.sort_values('Overall Passing Rate')
bottom_5.style.format({'Total Students': '{:,}', "Total School Budget": "${:,}", "Per Student Budget": "${:.0f}", "% Passing Math": "{:.1%}", "% Passing Reading": "{:.1%}", "Overall Passing Rate": "{:.1%}"})
```




<style  type="text/css" >
</style>  
<table id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aa" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >School ID</th> 
        <th class="col_heading level0 col1" >School Type</th> 
        <th class="col_heading level0 col2" >Total Students</th> 
        <th class="col_heading level0 col3" >Total School Budget</th> 
        <th class="col_heading level0 col4" >Per Student Budget</th> 
        <th class="col_heading level0 col5" >Average Math Score</th> 
        <th class="col_heading level0 col6" >Average Reading Score</th> 
        <th class="col_heading level0 col7" >% Passing Math</th> 
        <th class="col_heading level0 col8" >% Passing Reading</th> 
        <th class="col_heading level0 col9" >Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >School Name</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aa" class="row_heading level0 row0" >Rodriguez High School</th> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow0_col0" class="data row0 col0" >11</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow0_col1" class="data row0 col1" >District</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow0_col2" class="data row0 col2" >3,999</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow0_col3" class="data row0 col3" >$2,547,363</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow0_col4" class="data row0 col4" >$637</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow0_col5" class="data row0 col5" >76.8</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow0_col6" class="data row0 col6" >80.7</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow0_col7" class="data row0 col7" >66.4%</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow0_col8" class="data row0 col8" >80.2%</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow0_col9" class="data row0 col9" >73.3%</td> 
    </tr>    <tr> 
        <th id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aa" class="row_heading level0 row1" >Figueroa High School</th> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow1_col0" class="data row1 col0" >1</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow1_col1" class="data row1 col1" >District</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow1_col2" class="data row1 col2" >2,949</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow1_col3" class="data row1 col3" >$1,884,411</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow1_col4" class="data row1 col4" >$639</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow1_col5" class="data row1 col5" >76.7</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow1_col6" class="data row1 col6" >81.2</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow1_col7" class="data row1 col7" >66.0%</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow1_col8" class="data row1 col8" >80.7%</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow1_col9" class="data row1 col9" >73.4%</td> 
    </tr>    <tr> 
        <th id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aa" class="row_heading level0 row2" >Huang High School</th> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow2_col0" class="data row2 col0" >0</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow2_col1" class="data row2 col1" >District</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow2_col2" class="data row2 col2" >2,917</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow2_col3" class="data row2 col3" >$1,910,635</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow2_col4" class="data row2 col4" >$655</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow2_col5" class="data row2 col5" >76.6</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow2_col6" class="data row2 col6" >81.2</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow2_col7" class="data row2 col7" >65.7%</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow2_col8" class="data row2 col8" >81.3%</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow2_col9" class="data row2 col9" >73.5%</td> 
    </tr>    <tr> 
        <th id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aa" class="row_heading level0 row3" >Johnson High School</th> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow3_col0" class="data row3 col0" >12</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow3_col1" class="data row3 col1" >District</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow3_col2" class="data row3 col2" >4,761</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow3_col3" class="data row3 col3" >$3,094,650</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow3_col4" class="data row3 col4" >$650</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow3_col5" class="data row3 col5" >77.1</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow3_col6" class="data row3 col6" >81</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow3_col7" class="data row3 col7" >66.1%</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow3_col8" class="data row3 col8" >81.2%</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow3_col9" class="data row3 col9" >73.6%</td> 
    </tr>    <tr> 
        <th id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aa" class="row_heading level0 row4" >Ford High School</th> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow4_col0" class="data row4 col0" >13</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow4_col1" class="data row4 col1" >District</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow4_col2" class="data row4 col2" >2,739</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow4_col3" class="data row4 col3" >$1,763,916</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow4_col4" class="data row4 col4" >$644</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow4_col5" class="data row4 col5" >77.1</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow4_col6" class="data row4 col6" >80.7</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow4_col7" class="data row4 col7" >68.3%</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow4_col8" class="data row4 col8" >79.3%</td> 
        <td id="T_701a6f28_8f6f_11e7_9c6b_d4619d1535aarow4_col9" class="data row4 col9" >73.8%</td> 
    </tr></tbody> 
</table> 



## Math Scores by Grade


```python
#creates grade level average math scores for each school 
ninth_math = students_df.loc[students_df['grade'] == '9th'].groupby('school')["math_score"].mean().reset_index()
ninth_math.rename(columns = {'math_score': "9th"}, inplace=True)
tenth_math = students_df.loc[students_df['grade'] == '10th'].groupby('school')["math_score"].mean().reset_index()
tenth_math.rename(columns = {'math_score': "10th"}, inplace=True)
eleventh_math = students_df.loc[students_df['grade'] == '11th'].groupby('school')["math_score"].mean().reset_index()
eleventh_math.rename(columns = {'math_score': "11th"}, inplace=True)
twelfth_math = students_df.loc[students_df['grade'] == '12th'].groupby('school')["math_score"].mean().reset_index()
twelfth_math.rename(columns = {'math_score': "12th"}, inplace=True)

#merges the math score averages by school and grade together
math_scores = pd.merge(ninth_math, tenth_math, on = 'school').merge(eleventh_math, on = 'school').merge(twelfth_math, on = 'school')

#formatting
math_scores.rename(columns = {'school':'School Name'}, inplace = True)
math_scores.set_index('School Name', inplace = True)
math_scores.style.format({'9th': '{:.1f}', "10th": '{:.1f}', "11th": "{:.1f}", "12th": "{:.1f}"})
```




<style  type="text/css" >
</style>  
<table id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >9th</th> 
        <th class="col_heading level0 col1" >10th</th> 
        <th class="col_heading level0 col2" >11th</th> 
        <th class="col_heading level0 col3" >12th</th> 
    </tr>    <tr> 
        <th class="index_name level0" >School Name</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row0" >Bailey High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow0_col0" class="data row0 col0" >77.1</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow0_col1" class="data row0 col1" >77.0</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow0_col2" class="data row0 col2" >77.5</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow0_col3" class="data row0 col3" >76.5</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row1" >Cabrera High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow1_col0" class="data row1 col0" >83.1</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow1_col1" class="data row1 col1" >83.2</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow1_col2" class="data row1 col2" >82.8</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow1_col3" class="data row1 col3" >83.3</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row2" >Figueroa High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow2_col0" class="data row2 col0" >76.4</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow2_col1" class="data row2 col1" >76.5</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow2_col2" class="data row2 col2" >76.9</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow2_col3" class="data row2 col3" >77.2</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row3" >Ford High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow3_col0" class="data row3 col0" >77.4</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow3_col1" class="data row3 col1" >77.7</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow3_col2" class="data row3 col2" >76.9</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow3_col3" class="data row3 col3" >76.2</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row4" >Griffin High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow4_col0" class="data row4 col0" >82.0</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow4_col1" class="data row4 col1" >84.2</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow4_col2" class="data row4 col2" >83.8</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow4_col3" class="data row4 col3" >83.4</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row5" >Hernandez High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow5_col0" class="data row5 col0" >77.4</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow5_col1" class="data row5 col1" >77.3</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow5_col2" class="data row5 col2" >77.1</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow5_col3" class="data row5 col3" >77.2</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row6" >Holden High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow6_col0" class="data row6 col0" >83.8</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow6_col1" class="data row6 col1" >83.4</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow6_col2" class="data row6 col2" >85.0</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow6_col3" class="data row6 col3" >82.9</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row7" >Huang High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow7_col0" class="data row7 col0" >77.0</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow7_col1" class="data row7 col1" >75.9</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow7_col2" class="data row7 col2" >76.4</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow7_col3" class="data row7 col3" >77.2</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row8" >Johnson High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow8_col0" class="data row8 col0" >77.2</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow8_col1" class="data row8 col1" >76.7</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow8_col2" class="data row8 col2" >77.5</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow8_col3" class="data row8 col3" >76.9</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row9" >Pena High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow9_col0" class="data row9 col0" >83.6</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow9_col1" class="data row9 col1" >83.4</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow9_col2" class="data row9 col2" >84.3</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow9_col3" class="data row9 col3" >84.1</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row10" >Rodriguez High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow10_col0" class="data row10 col0" >76.9</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow10_col1" class="data row10 col1" >76.6</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow10_col2" class="data row10 col2" >76.4</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow10_col3" class="data row10 col3" >77.7</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row11" >Shelton High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow11_col0" class="data row11 col0" >83.4</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow11_col1" class="data row11 col1" >82.9</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow11_col2" class="data row11 col2" >83.4</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow11_col3" class="data row11 col3" >83.8</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row12" >Thomas High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow12_col0" class="data row12 col0" >83.6</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow12_col1" class="data row12 col1" >83.1</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow12_col2" class="data row12 col2" >83.5</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow12_col3" class="data row12 col3" >83.5</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row13" >Wilson High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow13_col0" class="data row13 col0" >83.1</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow13_col1" class="data row13 col1" >83.7</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow13_col2" class="data row13 col2" >83.2</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow13_col3" class="data row13 col3" >83.0</td> 
    </tr>    <tr> 
        <th id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aa" class="row_heading level0 row14" >Wright High School</th> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow14_col0" class="data row14 col0" >83.3</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow14_col1" class="data row14 col1" >84.0</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow14_col2" class="data row14 col2" >83.8</td> 
        <td id="T_70a29dc6_8f6f_11e7_ac97_d4619d1535aarow14_col3" class="data row14 col3" >83.6</td> 
    </tr></tbody> 
</table> 



## Reading Scores by Grade


```python
#creates grade level average reading scores for each school
ninth_reading = students_df.loc[students_df['grade'] == '9th'].groupby('school')["reading_score"].mean().reset_index()
ninth_reading.rename(columns = {"reading_score": "9th"}, inplace=True)
tenth_reading = students_df.loc[students_df['grade'] == '10th'].groupby('school')["reading_score"].mean().reset_index()
tenth_reading.rename(columns = {"reading_score": "10th"}, inplace=True)
eleventh_reading = students_df.loc[students_df['grade'] == '11th'].groupby('school')["reading_score"].mean().reset_index()
eleventh_reading.rename(columns = {"reading_score": "11th"}, inplace=True)
twelfth_reading = students_df.loc[students_df['grade'] == '12th'].groupby('school')["reading_score"].mean().reset_index()
twelfth_reading.rename(columns = {"reading_score": "12th"}, inplace=True)

#merges the reading score averages by school and grade together
reading_scores = pd.merge(ninth_reading, tenth_reading, on = 'school').merge(eleventh_reading, on = 'school').merge(twelfth_reading, on = 'school')
reading_scores.rename(columns = {'school':'School Name'}, inplace = True)
reading_scores.set_index('School Name', inplace = True)
reading_scores.style.format({'9th': '{:.1f}', "10th": '{:.1f}', "11th": "{:.1f}", "12th": "{:.1f}"})

```




<style  type="text/css" >
</style>  
<table id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >9th</th> 
        <th class="col_heading level0 col1" >10th</th> 
        <th class="col_heading level0 col2" >11th</th> 
        <th class="col_heading level0 col3" >12th</th> 
    </tr>    <tr> 
        <th class="index_name level0" >School Name</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row0" >Bailey High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow0_col0" class="data row0 col0" >81.3</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow0_col1" class="data row0 col1" >80.9</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow0_col2" class="data row0 col2" >80.9</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow0_col3" class="data row0 col3" >80.9</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row1" >Cabrera High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow1_col0" class="data row1 col0" >83.7</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow1_col1" class="data row1 col1" >84.3</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow1_col2" class="data row1 col2" >83.8</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow1_col3" class="data row1 col3" >84.3</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row2" >Figueroa High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow2_col0" class="data row2 col0" >81.2</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow2_col1" class="data row2 col1" >81.4</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow2_col2" class="data row2 col2" >80.6</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow2_col3" class="data row2 col3" >81.4</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row3" >Ford High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow3_col0" class="data row3 col0" >80.6</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow3_col1" class="data row3 col1" >81.3</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow3_col2" class="data row3 col2" >80.4</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow3_col3" class="data row3 col3" >80.7</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row4" >Griffin High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow4_col0" class="data row4 col0" >83.4</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow4_col1" class="data row4 col1" >83.7</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow4_col2" class="data row4 col2" >84.3</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow4_col3" class="data row4 col3" >84.0</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row5" >Hernandez High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow5_col0" class="data row5 col0" >80.9</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow5_col1" class="data row5 col1" >80.7</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow5_col2" class="data row5 col2" >81.4</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow5_col3" class="data row5 col3" >80.9</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row6" >Holden High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow6_col0" class="data row6 col0" >83.7</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow6_col1" class="data row6 col1" >83.3</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow6_col2" class="data row6 col2" >83.8</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow6_col3" class="data row6 col3" >84.7</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row7" >Huang High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow7_col0" class="data row7 col0" >81.3</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow7_col1" class="data row7 col1" >81.5</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow7_col2" class="data row7 col2" >81.4</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow7_col3" class="data row7 col3" >80.3</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row8" >Johnson High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow8_col0" class="data row8 col0" >81.3</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow8_col1" class="data row8 col1" >80.8</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow8_col2" class="data row8 col2" >80.6</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow8_col3" class="data row8 col3" >81.2</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row9" >Pena High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow9_col0" class="data row9 col0" >83.8</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow9_col1" class="data row9 col1" >83.6</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow9_col2" class="data row9 col2" >84.3</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow9_col3" class="data row9 col3" >84.6</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row10" >Rodriguez High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow10_col0" class="data row10 col0" >81.0</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow10_col1" class="data row10 col1" >80.6</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow10_col2" class="data row10 col2" >80.9</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow10_col3" class="data row10 col3" >80.4</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row11" >Shelton High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow11_col0" class="data row11 col0" >84.1</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow11_col1" class="data row11 col1" >83.4</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow11_col2" class="data row11 col2" >84.4</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow11_col3" class="data row11 col3" >82.8</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row12" >Thomas High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow12_col0" class="data row12 col0" >83.7</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow12_col1" class="data row12 col1" >84.3</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow12_col2" class="data row12 col2" >83.6</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow12_col3" class="data row12 col3" >83.8</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row13" >Wilson High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow13_col0" class="data row13 col0" >83.9</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow13_col1" class="data row13 col1" >84.0</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow13_col2" class="data row13 col2" >83.8</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow13_col3" class="data row13 col3" >84.3</td> 
    </tr>    <tr> 
        <th id="T_710f8d14_8f6f_11e7_8652_d4619d1535aa" class="row_heading level0 row14" >Wright High School</th> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow14_col0" class="data row14 col0" >83.8</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow14_col1" class="data row14 col1" >83.8</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow14_col2" class="data row14 col2" >84.2</td> 
        <td id="T_710f8d14_8f6f_11e7_8652_d4619d1535aarow14_col3" class="data row14 col3" >84.1</td> 
    </tr></tbody> 
</table> 



## Scores by School Spending


```python
# merge schools and student datasets
merged_df = pd.merge(students_df, schools_df, on = 'school')

#add a column and assign bins for spending per student
merged_df.loc[(merged_df['Per Student Budget'] < 585), "spending_bin"] = "< $585"
merged_df.loc[((merged_df['Per Student Budget'] >= 585) & (merged_df['Per Student Budget'] < 615)), "spending_bin"] = "$585 - 614" 
merged_df.loc[((merged_df['Per Student Budget'] >= 615) & (merged_df['Per Student Budget'] < 645)), "spending_bin"] = "$615 - 644" 
merged_df.loc[(merged_df['Per Student Budget'] >= 645), "spending_bin"] = "> $644" 

# group by spending bin
by_spending = merged_df.groupby('spending_bin')

#find average math and reading score for each spending bin
avg_scores_by_spend = by_spending['math_score', 'reading_score'].mean().reset_index()

#find # of students passing in each spending bin by using conditional and rename column
pass_read_by_spend = merged_df[merged_df['reading_score'] >= 70].groupby('spending_bin')['reading_score'].count().reset_index()
pass_read_by_spend.rename(columns = {'reading_score': '# pass reading'}, inplace=True)

pass_math_by_spend = merged_df[merged_df['math_score'] >= 70].groupby('spending_bin')['math_score'].count().reset_index()
pass_math_by_spend.rename(columns = {'math_score': '# pass math'}, inplace=True)

#find # of students in each spending bin to calculate percentage below
size_by_spend = by_spending['name'].count().reset_index()
size_by_spend.rename(columns = {'name':'size'}, inplace = True)

# merge so far
scores_by_spend = pd.merge(avg_scores_by_spend, pass_read_by_spend, on = "spending_bin").merge(pass_math_by_spend, on='spending_bin').merge(size_by_spend, on='spending_bin')
# add columns for % passing math and reading
scores_by_spend['% Passing Reading'] = scores_by_spend['# pass reading']/scores_by_spend['size']
scores_by_spend['% Passing Math'] = scores_by_spend['# pass math']/scores_by_spend['size']
# keep only data needed for table
scores_by_spend = scores_by_spend[['spending_bin', 'math_score', 'reading_score', '% Passing Math', '% Passing Reading']]
# add column for overall passing rate
scores_by_spend['Overall Passing Rate'] = (scores_by_spend['% Passing Reading']+ scores_by_spend['% Passing Math'])/2
#reorder rows
scores_by_spend = scores_by_spend.reindex([2,0,1,3])
#formatting
scores_by_spend.rename(columns = {'spending_bin':'Spending Per Student','math_score': 'Average Math Score', 'reading_score':'Average Reading Score'}, inplace=True)
scores_by_spend.set_index('Spending Per Student', inplace=True)
scores_by_spend.style.format({'Average Math Score': '{:.1f}', 'Average Reading Score': '{:.1f}', '% Passing Math': '{:.1%}', '% Passing Reading':'{:.1%}', 'Overall Passing Rate': '{:.1%}'})

```




<style  type="text/css" >
</style>  
<table id="T_210fdc82_8f70_11e7_9330_d4619d1535aa" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Average Math Score</th> 
        <th class="col_heading level0 col1" >Average Reading Score</th> 
        <th class="col_heading level0 col2" >% Passing Math</th> 
        <th class="col_heading level0 col3" >% Passing Reading</th> 
        <th class="col_heading level0 col4" >Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Spending Per Student</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_210fdc82_8f70_11e7_9330_d4619d1535aa" class="row_heading level0 row0" >< $585</th> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow0_col0" class="data row0 col0" >83.4</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow0_col1" class="data row0 col1" >84.0</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow0_col2" class="data row0 col2" >93.7%</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow0_col3" class="data row0 col3" >96.7%</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow0_col4" class="data row0 col4" >95.2%</td> 
    </tr>    <tr> 
        <th id="T_210fdc82_8f70_11e7_9330_d4619d1535aa" class="row_heading level0 row1" >$585 - 614</th> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow1_col0" class="data row1 col0" >83.5</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow1_col1" class="data row1 col1" >83.8</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow1_col2" class="data row1 col2" >94.1%</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow1_col3" class="data row1 col3" >95.9%</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow1_col4" class="data row1 col4" >95.0%</td> 
    </tr>    <tr> 
        <th id="T_210fdc82_8f70_11e7_9330_d4619d1535aa" class="row_heading level0 row2" >$615 - 644</th> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow2_col0" class="data row2 col0" >78.1</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow2_col1" class="data row2 col1" >81.4</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow2_col2" class="data row2 col2" >71.4%</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow2_col3" class="data row2 col3" >83.6%</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow2_col4" class="data row2 col4" >77.5%</td> 
    </tr>    <tr> 
        <th id="T_210fdc82_8f70_11e7_9330_d4619d1535aa" class="row_heading level0 row3" >> $644</th> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow3_col0" class="data row3 col0" >77.0</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow3_col1" class="data row3 col1" >81.0</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow3_col2" class="data row3 col2" >66.2%</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow3_col3" class="data row3 col3" >81.1%</td> 
        <td id="T_210fdc82_8f70_11e7_9330_d4619d1535aarow3_col4" class="data row3 col4" >73.7%</td> 
    </tr></tbody> 
</table> 



## Scores by School Size


```python
# bin by size of student body
merged_df.loc[(merged_df['size'] < 1000), "size_class"] = "Small (<1000)"
merged_df.loc[((merged_df['size'] >= 1000) & (merged_df['size'] <= 2000)), "size_class"] = "Medium (1000-2000)" 
merged_df.loc[(merged_df['size'] > 2000), "size_class"] = "Large (>2000)" 

#groupby school size bins
by_size = merged_df.groupby("size_class")

#get average scores for math and reading by size bin
avg_scores_by_size = by_size['math_score', 'reading_score'].mean().reset_index()

#get # of students passing math and reading by size of school
pass_read_by_size = merged_df[merged_df['reading_score'] >= 70].groupby("size_class")['reading_score'].count().reset_index()
pass_read_by_size.rename(columns = {'reading_score': '# pass reading'}, inplace=True)

pass_math_by_size = merged_df[merged_df['math_score'] >= 70].groupby("size_class")['math_score'].count().reset_index()
pass_math_by_size.rename(columns = {'math_score': '# pass math'}, inplace=True)

#get number of students in each size bin
size_by_size = by_size['name'].count().reset_index()
size_by_size.rename(columns = {'name':'size'}, inplace = True)

#merge to use in calculation
scores_by_size = pd.merge(avg_scores_by_size, pass_read_by_size, on = "size_class").merge(pass_math_by_size, on="size_class").merge(size_by_size, on="size_class")
#calculate %s
scores_by_size['% Passing Reading'] = scores_by_size['# pass reading']/scores_by_size['size']
scores_by_size['% Passing Math'] = scores_by_size['# pass math']/scores_by_size['size']
# get rid of columns not needed
scores_by_size = scores_by_size[["size_class", 'math_score', 'reading_score', '% Passing Math', '% Passing Reading']]
# calculate overall passing rate
scores_by_size['Overall Passing Rate'] = (scores_by_size['% Passing Reading']+ scores_by_size['% Passing Math'])/2

#formatting
scores_by_size = scores_by_size.reindex([2,1,0])
scores_by_size.rename(columns = {"size_class": "School Size",'math_score': 'Average Math Score', 'reading_score':'Average Reading Score'}, inplace=True)
scores_by_size.set_index('School Size', inplace=True)
scores_by_size.style.format({'Average Math Score': '{:.1f}', 'Average Reading Score': '{:.1f}', '% Passing Math': '{:.1%}', '% Passing Reading':'{:.1%}', 'Overall Passing Rate': '{:.1%}'})

```




<style  type="text/css" >
</style>  
<table id="T_58ec7b24_8f70_11e7_a340_d4619d1535aa" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Average Math Score</th> 
        <th class="col_heading level0 col1" >Average Reading Score</th> 
        <th class="col_heading level0 col2" >% Passing Math</th> 
        <th class="col_heading level0 col3" >% Passing Reading</th> 
        <th class="col_heading level0 col4" >Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >School Size</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_58ec7b24_8f70_11e7_a340_d4619d1535aa" class="row_heading level0 row0" >Small (<1000)</th> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow0_col0" class="data row0 col0" >83.8</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow0_col1" class="data row0 col1" >84.0</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow0_col2" class="data row0 col2" >94.0%</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow0_col3" class="data row0 col3" >96.0%</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow0_col4" class="data row0 col4" >95.0%</td> 
    </tr>    <tr> 
        <th id="T_58ec7b24_8f70_11e7_a340_d4619d1535aa" class="row_heading level0 row1" >Medium (1000-2000)</th> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow1_col0" class="data row1 col0" >83.4</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow1_col1" class="data row1 col1" >83.9</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow1_col2" class="data row1 col2" >93.6%</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow1_col3" class="data row1 col3" >96.8%</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow1_col4" class="data row1 col4" >95.2%</td> 
    </tr>    <tr> 
        <th id="T_58ec7b24_8f70_11e7_a340_d4619d1535aa" class="row_heading level0 row2" >Large (>2000)</th> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow2_col0" class="data row2 col0" >77.5</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow2_col1" class="data row2 col1" >81.2</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow2_col2" class="data row2 col2" >68.7%</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow2_col3" class="data row2 col3" >82.1%</td> 
        <td id="T_58ec7b24_8f70_11e7_a340_d4619d1535aarow2_col4" class="data row2 col4" >75.4%</td> 
    </tr></tbody> 
</table> 



## Scores by School Type


```python
# group by type of school
by_type = merged_df.groupby("type")

#find average score by type
avg_scores_by_type = by_type['math_score', 'reading_score'].mean().reset_index()

#find # passing by type of school
pass_read_by_type = merged_df[merged_df['reading_score'] >= 70].groupby("type")['reading_score'].count().reset_index()
pass_read_by_type.rename(columns = {'reading_score': '# pass reading'}, inplace=True)

pass_math_by_type = merged_df[merged_df['math_score'] >= 70].groupby("type")['math_score'].count().reset_index()
pass_math_by_type.rename(columns = {'math_score': '# pass math'}, inplace=True)

#find number of students by type of school
size_by_type = by_type['name'].count().reset_index()
size_by_type.rename(columns = {'name':'size'}, inplace = True)

#merge data for calculations
scores_by_type = pd.merge(avg_scores_by_type, pass_read_by_type, on = "type").merge(pass_math_by_type, on="type").merge(size_by_type, on="type")
scores_by_type['% Passing Reading'] = scores_by_type['# pass reading']/scores_by_type['size']
scores_by_type['% Passing Math'] = scores_by_type['# pass math']/scores_by_type['size']
# only keep needed columns
scores_by_type = scores_by_type[["type", 'math_score', 'reading_score', '% Passing Math', '% Passing Reading']]
# calc passing rate for each type
scores_by_type['Overall Passing Rate'] = (scores_by_type['% Passing Reading']+ scores_by_type['% Passing Math'])/2
#formatting
scores_by_type.rename(columns = {"type": "School Size",'math_score': 'Average Math Score', 'reading_score':'Average Reading Score'}, inplace=True)
scores_by_type.set_index('School Size', inplace=True)
scores_by_type.style.format({'Average Math Score': '{:.1f}', 'Average Reading Score': '{:.1f}', '% Passing Math': '{:.1%}', '% Passing Reading':'{:.1%}', 'Overall Passing Rate': '{:.1%}'})

```




<style  type="text/css" >
</style>  
<table id="T_ed1eb852_8f70_11e7_a523_d4619d1535aa" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Average Math Score</th> 
        <th class="col_heading level0 col1" >Average Reading Score</th> 
        <th class="col_heading level0 col2" >% Passing Math</th> 
        <th class="col_heading level0 col3" >% Passing Reading</th> 
        <th class="col_heading level0 col4" >Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >School Size</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_ed1eb852_8f70_11e7_a523_d4619d1535aa" class="row_heading level0 row0" >Charter</th> 
        <td id="T_ed1eb852_8f70_11e7_a523_d4619d1535aarow0_col0" class="data row0 col0" >83.4</td> 
        <td id="T_ed1eb852_8f70_11e7_a523_d4619d1535aarow0_col1" class="data row0 col1" >83.9</td> 
        <td id="T_ed1eb852_8f70_11e7_a523_d4619d1535aarow0_col2" class="data row0 col2" >93.7%</td> 
        <td id="T_ed1eb852_8f70_11e7_a523_d4619d1535aarow0_col3" class="data row0 col3" >96.6%</td> 
        <td id="T_ed1eb852_8f70_11e7_a523_d4619d1535aarow0_col4" class="data row0 col4" >95.2%</td> 
    </tr>    <tr> 
        <th id="T_ed1eb852_8f70_11e7_a523_d4619d1535aa" class="row_heading level0 row1" >District</th> 
        <td id="T_ed1eb852_8f70_11e7_a523_d4619d1535aarow1_col0" class="data row1 col0" >77.0</td> 
        <td id="T_ed1eb852_8f70_11e7_a523_d4619d1535aarow1_col1" class="data row1 col1" >81.0</td> 
        <td id="T_ed1eb852_8f70_11e7_a523_d4619d1535aarow1_col2" class="data row1 col2" >66.5%</td> 
        <td id="T_ed1eb852_8f70_11e7_a523_d4619d1535aarow1_col3" class="data row1 col3" >80.9%</td> 
        <td id="T_ed1eb852_8f70_11e7_a523_d4619d1535aarow1_col4" class="data row1 col4" >73.7%</td> 
    </tr></tbody> 
</table> 




```python

```
