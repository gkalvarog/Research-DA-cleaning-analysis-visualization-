# Data Analysis Process of a Research file
## Cleaning, Transform & Visualize
### guide [here](https://www.youtube.com/watch?v=pKvWD0f18Pc&list=WL&index=22&t=90s).

```python
import pandas as pd
import os
```


```python
# present working directory PWD -> get CURRENT WORKING DIRECTORY CWD
pwd = os.getcwd()
```


```python
dataframe = pd.read_excel(pwd + "/Data - Survey Monkey Output - edited.xlsx", sheet_name="edited_data")

# To see the dataframe, unlock the line below. 
# dataframe
```


```python
df_copy = dataframe.copy()
```


```python
# Wanna see all the columns of the data frame? Unlock the line below.
# df_copy.columns
```


```python
# Get rid of: 'Start Date', 'End Date', 'Email Address', 'First Name', 'Last Name', 'Custom Data 1'
columns_to_drop = ['Start Date', 'End Date', 'Email Address', 'First Name', 'Last Name', 'Custom Data 1']
df_copy = df_copy.drop(columns = columns_to_drop)

# Wanna see the new data frame? Unlock the line below.
# df_copy

# Wanna see all the columns of the new data frame? Unlock the line below.
# df_copy.columns
```


```python
# Shall not we unpivot this table? (Unpivot => Transform from Wide into Long format). Can also be called Melt.

# 1: we need to work with demographic data, so let us get it now:
demographic_columns = list(df_copy.columns)[:8]

# To see them, unlock below:
# demographic_columns

# 2: let us take the other columns:
other_columns = list(df_copy.columns)[8:]

# To see them, unlock below:
# other_columns

# Finally, Melt. Let us also create df_melted, a new version.
df_melted = df_copy.melt(id_vars=demographic_columns, value_vars=other_columns, var_name ="Question + Subquestion" , value_name = "Answer")

# Unlock to give a look.
# df_melted
```


```python
questions_import = pd.read_excel(pwd + "/Data - Survey Monkey Output - edited.xlsx", sheet_name="questions")

# Let us see the questions:
# questions_import

# Let us already drop the columns we do not need. But before, let us make a copy:
questions = questions_import.copy()

questions = questions.drop(columns = ['Raw Question', 'Raw Subquestion', 'Subquestion',	'Question + Subquestion'])

# If there are NaN values, use dropna()
questions = questions.dropna()

# Unlock to  see the new:
# questions
```


```python
# Time to join. A left join will keep the left sheet and insert in it items from the right sheet, dropping the rest of the right sheet.
df_merged = pd.merge(left=df_melted, right=questions, how="left", left_on="Question + Subquestion", right_on="Question + Subquestion VALUES ONLY")

print(f"Original Data: {len(df_melted)}.\nMerged Data: {len(df_merged)}")

# Unlock to see the merge:
# df_merged
```

    Original Data: 17028.
    Merged Data: 17028



```python
# How many people answered to each question? Careful: if we leave the NaN values in the Answers column, they will be counted.

# Create a new data frame with the removal of the NaN values:
respondents = df_merged[df_merged["Answer"].notna()]
new_respondents = respondents.groupby("Question")["Respondent ID"].nunique().reset_index()

# In order to rename a column, we use 'rename()' and as an argument, we send in a dictionary. In it, the key is the previous name, and the value is the new name. 
new_respondents.rename(columns={"Respondent ID": "Respondents"}, inplace=True)

# new_respondents
```


```python
# Time to join again.
df_merged_2 = pd.merge(left=df_merged, right=new_respondents, how="left", left_on="Question", right_on="Question")

# Unlock to see the new merge:
# df_merged_2
```


```python
# Time to find the amount of equal answers per question. We need to take the questions and group by answer and count the number of IDs per answer.

# Create a new data frame with the removal of the NaN values (if it is the case)
same_answers = df_merged # [df_merged["Answer"].notna()]
same_answers = same_answers.groupby(["Question + Subquestion", "Answer"])["Respondent ID"].nunique().reset_index()

# In order to rename a column, we use 'rename()' and as an argument, we send in a dictionary. In it, the key is the previous name, and the value is the new name. 
same_answers.rename(columns={"Respondent ID": "Same Answer"}, inplace=True)

# Unlock to see it
# same_answers
```


```python
# Time to join again.
df_merged_3 = pd.merge(left=df_merged_2, right=same_answers, how="left", left_on=["Question + Subquestion", "Answer"], right_on=["Question + Subquestion", "Answer"])

# Insert a 0 instead of NaN
df_merged_3["Same Answer"].fillna(0, inplace=True)

# Unlock to see the new merge:
# df_merged_3
```


```python
output = df_merged_3.copy()

# Rename the columns to make the reading easier.
columns_to_modify = {
    "Identify which division you work in. - Response": "Division Primary",
    "Identify which division you work in. - Other (please specify)": "Division Secundary",
    "Which of the following best describes your position level? - Response": "Position",
    "Which generation are you apart of? - Response": "Generation",
    "Please select the gender in which you identify. - Response": "Gender",
    "Which duration range best aligns with your tenure at your company? - Response": "Tenure",
    "Which of the following best describes your employment type? - Response": "Employment Type"
    }

output.rename(columns=columns_to_modify, inplace=True)

# Unlock to see the output:
output

# Export the file if you wanna work on Tableau, Power BI, Looker, etc... Remember to remove the Index (1st col)
output.to_excel(pwd + "/Final Output.xlsx", index=False)
```


```python

```
