# PyCitySchools

## Background
In this repository, I'm using pandas and jupyter notebook to analyze district-wide standarized test results. I've been giving access to every student's math and reading scores [students_complete.csv](PyCitySchools/Resources/students_complete.csv), as well as various information on the schools they attend [schools_complete.csv](PyCitySchools/Resources/schools_complete.csv). I will be aggregating the data in different views to show trends in school performance.

## Observable Trends
- On average, students in the district tend to do better at reading than at math, which is evident from both the average scores for each exam as well as the percentage of students that passed each exam. The average reading test score is 81.88, compared to 78.99 for math. ~86% of the students in the district passed the reading test, while ~75% passed the math test.
- Schools that spend a higher budget per student do not have higher overall test results. In fact, the schools with the highest overall average test scores are the ones that have the lowest budget per student (<585 dollars).
- Small and medium sized schools outperform large sized schools on overall passing performance (90-91% passing vs. 58%). This is largely driven by a low % of students passing math exams at large schools (~70%), compared to >90% at small and medium sized schools.

## Analysis

### Import Dependencies and Setup
```python
# Import dependencies
import pandas as pd

# Store files into Pandas DataFrames
schoolData = pd.read_csv("Resources/schools_complete.csv")
studentData = pd.read_csv("Resources/students_complete.csv")

# Combine the data into a single dataFrame. 
districtData = pd.merge(studentData, schoolData, how="left", on=["school_name", "school_name"])
```

### District Summary
```python
# Calculate the Totals (Schools and Students)
totalSchoolCount = len(districtData["school_name"].unique())
totalStudentCount = len(districtData["Student ID"])

# Calculate the Total Budget
totalBudget = schoolData["budget"].sum()

# Calculate the Average Scores
avgMath = districtData["math_score"].mean()
avgReading = districtData ["reading_score"].mean()

# Calculate the Percentage Pass Rates
passedReadingCount = districtData[districtData["reading_score"] > 69]
passedMathCount = districtData[districtData["math_score"] > 69]
passedBothCount = districtData[(districtData["reading_score"] > 69) & (districtData["math_score"] > 69)]

passedReading = (len(passedReadingCount) / float(totalStudentCount))*100
passedMath = (len(passedMathCount) / float(totalStudentCount))*100
passedBoth = (len(passedBothCount) / float(totalStudentCount))*100

# Make a district summary DataFrame using a list of Dictionaries
districtSummary = [
    {"Total Schools": totalSchoolCount, "Total Students": totalStudentCount,
     "Total Budget": totalBudget, "Average Math Score": avgMath,
     "Average Reading Score": avgReading, "% Passing Math": passedMath, 
     "% Passing Reading": passedReading, "% Overall Passing": passedBoth}
]
districtSummaryDF = pd.DataFrame(districtSummary)

# Using the "map" function to format the Total Students and Total Budget column
districtSummaryDF["Total Students"] = districtSummaryDF["Total Students"].map("{:,}".format)
districtSummaryDF["Total Budget"] = districtSummaryDF["Total Budget"].map("${:,}".format)
districtSummaryDF["Average Math Score"] = districtSummaryDF["Average Math Score"].map("{:.2f}".format)
districtSummaryDF["Average Reading Score"] = districtSummaryDF["Average Reading Score"].map("{:.2f}".format)
districtSummaryDF["% Passing Math"] = districtSummaryDF["% Passing Math"].map("{:.2f}%".format)
districtSummaryDF["% Passing Reading"] = districtSummaryDF["% Passing Reading"].map("{:.2f}%".format)
districtSummaryDF["% Overall Passing"] = districtSummaryDF["% Overall Passing"].map("{:.2f}%".format)

# Display the Data Frame
districtSummaryDF
```

### School Summary
```Python
# change the index of the school data to school names
updatedSchoolData = schoolData.set_index("school_name")

# Determine the School Type
schoolType = updatedSchoolData["type"]

# Calculate the total student count
studentCount = updatedSchoolData["size"]

# Calculate the total school budget 
schoolBudget = updatedSchoolData["budget"]

# Calculate per student budget and add it to dataframe
perStudentBudget = schoolBudget/studentCount
updatedSchoolData["Per Student Budget"] = perStudentBudget

# Calculate the average test scores 

# Use a groupbyfunction 
groupedStudentData = studentData.groupby(["school_name"])

# Calculate the average reading and math scores
avgSchoolReading = groupedStudentData["reading_score"].mean()
avgSchoolMath = groupedStudentData["math_score"].mean()

# Get the students who passed math and passed reading by creating separate filtered DataFrames.
passedReadingDF = studentData[studentData["reading_score"] > 69]
passedMathDF = studentData[studentData["math_score"] > 69]
passedBothDF = studentData[(studentData["reading_score"] > 69) & (studentData["math_score"] > 69)]

# Calculate percentage pass rates:

# Use a groupbyfunction for the 3 data frames
groupedReadingData = passedReadingDF.groupby(["school_name"])
groupedMathData = passedMathDF.groupby(["school_name"])
groupedBothData = passedBothDF.groupby(["school_name"])

# Calculate the percentage pass rates
passedReadingPct = (groupedReadingData["Student ID"].count()/studentCount)*100
passedMathPct = (groupedMathData["Student ID"].count()/studentCount)*100
passedBothPct = (groupedBothData["Student ID"].count()/studentCount)*100

# Convert to DataFrame --> save DF as it will be used later
schoolSummaryNumericDF = pd.DataFrame({
    "School Type": schoolType, "Total Students": studentCount,
    "Total School Budget": schoolBudget, "Per Student Budget": perStudentBudget,
    "Average Math Score": avgSchoolMath, "Average Reading Score": avgSchoolReading,
    "% Passing Math": passedMathPct, "% Passing Reading": passedReadingPct,
    "% Overall Passing": passedBothPct
})

# format the output using Map
schoolSummaryDF = schoolSummaryNumericDF.rename_axis("") # changing the name of the index column to nothing
schoolSummaryDF["Total Students"] = schoolSummaryNumericDF["Total Students"].map("{:,}".format)
schoolSummaryDF["Total School Budget"] = schoolSummaryNumericDF["Total School Budget"].map("${:,}".format)
schoolSummaryDF["Per Student Budget"] = schoolSummaryNumericDF["Per Student Budget"].map("${:.0f}".format)
schoolSummaryDF["Average Math Score"] = schoolSummaryNumericDF["Average Math Score"].map("{:.2f}".format)
schoolSummaryDF["Average Reading Score"] = schoolSummaryNumericDF["Average Reading Score"].map("{:.2f}".format)
schoolSummaryDF["% Passing Math"] = schoolSummaryNumericDF["% Passing Math"].map("{:.2f}%".format)
schoolSummaryDF["% Passing Reading"] = schoolSummaryNumericDF["% Passing Reading"].map("{:.2f}%".format)
schoolSummaryDF["% Overall Passing"] = schoolSummaryNumericDF["% Overall Passing"].map("{:.2f}%".format)

# Display the DataFrame
schoolSummaryDF
```

### Top Performing Schools (By % Overall Passing)
```python
# Sort and show top five schools
top5Schools = schoolSummaryDF.sort_values("% Overall Passing", ascending=False)
top5Schools.head()
```

### Bottom Performing Schools (By % Overall Passing)
```python
# Sort and show bottom five schools
bottom5Schools = schoolSummaryDF.sort_values("% Overall Passing", ascending=True)
bottom5Schools.head()
```

### Math Scores by Grade
```python
# Create data series of scores by grade levels 
ninthGraders = studentData[studentData["grade"] == "9th"]
tenthGraders = studentData[studentData["grade"] == "10th"]
eleventhGraders = studentData[studentData["grade"] == "11th"]
twelfthGraders = studentData[studentData["grade"] == "12th"]

# Group each by school name
grouped9thMath = ninthGraders.groupby(["school_name"])["math_score"].mean()
grouped10thMath = tenthGraders.groupby(["school_name"])["math_score"].mean()
grouped11thMath = eleventhGraders.groupby(["school_name"])["math_score"].mean()
grouped12thMath = twelfthGraders.groupby(["school_name"])["math_score"].mean()

# Combine into a single data frame 
mathScoresDF = pd.DataFrame({
    "9th": grouped9thMath, 
    "10th": grouped10thMath,
    "11th": grouped11thMath, 
    "12th": grouped12thMath
})

# Using Map to format DataFrame
mathScoresDF = mathScoresDF.rename_axis("") # changing the name of the index column to nothing
mathScoresDF["9th"] = mathScoresDF["9th"].map("{:.2f}".format)
mathScoresDF["10th"] = mathScoresDF["10th"].map("{:.2f}".format)
mathScoresDF["11th"] = mathScoresDF["11th"].map("{:.2f}".format)
mathScoresDF["12th"] = mathScoresDF["12th"].map("{:.2f}".format)

# Display the DataFrame
mathScoresDF
```

### Reading Score by Grade 
```python
# using the data series from the above exercise, group each by school name
grouped9thReading = ninthGraders.groupby(["school_name"])["reading_score"].mean()
grouped10thReading = tenthGraders.groupby(["school_name"])["reading_score"].mean()
grouped11thReading = eleventhGraders.groupby(["school_name"])["reading_score"].mean()
grouped12thReading = twelfthGraders.groupby(["school_name"])["reading_score"].mean()

# Combine into a single data frame 
readingScoresDF = pd.DataFrame({
    "9th": grouped9thReading, 
    "10th": grouped10thReading,
    "11th": grouped11thReading, 
    "12th": grouped12thReading
})

# Using Map to format DataFrame
readingScoresDF = readingScoresDF.rename_axis("") # changing the name of the index column to nothing
readingScoresDF["9th"] = readingScoresDF["9th"].map("{:.2f}".format)
readingScoresDF["10th"] = readingScoresDF["10th"].map("{:.2f}".format)
readingScoresDF["11th"] = readingScoresDF["11th"].map("{:.2f}".format)
readingScoresDF["12th"] = readingScoresDF["12th"].map("{:.2f}".format)

# Display the DataFrame
readingScoresDF
```

### Scores by School Spending
```python
# Ignore warnings
import warnings
warnings.filterwarnings("ignore")

# Establish the bins 
bins = [0, 585, 630, 645, 1000]
groupNames = ["<$585", "$585-630", "$630-645", "$645+"]

# Create the new column based on the bins
# We'll use the numeric version of the schoolSummary DF
schoolSummaryNumericDF["Spending Ranges (Per Student)"] = \
pd.cut(schoolSummaryNumericDF["Per Student Budget"], bins, labels = groupNames)

# Calculate averages for the desired columns using groupby 
spendingAvgMath = schoolSummaryNumericDF.groupby(["Spending Ranges (Per Student)"])["Average Math Score"].mean()
spendingAvgReading = schoolSummaryNumericDF.groupby(["Spending Ranges (Per Student)"])["Average Reading Score"].mean()
spendingAvgMathPassing = schoolSummaryNumericDF.groupby(["Spending Ranges (Per Student)"])["% Passing Math"].mean()
spendingAvgReadingPassing = schoolSummaryNumericDF.groupby(["Spending Ranges (Per Student)"])["% Passing Reading"].mean()
spendingAvgBothPassing = schoolSummaryNumericDF.groupby(["Spending Ranges (Per Student)"])["% Overall Passing"].mean()
# Assemble into DataFrame
schoolSpendingDF = pd.DataFrame({
    "Average Math Score": spendingAvgMath, 
    "Average Reading Score": spendingAvgReading,
    "% Passing Math": spendingAvgMathPassing, 
    "% Passing Reading": spendingAvgReading,
    "% Overall Passing": spendingAvgBothPassing
})

# Format DataFrame using map
schoolSpendingDF["Average Math Score"] = schoolSpendingDF["Average Math Score"].map("{:.2f}".format)
schoolSpendingDF["Average Reading Score"] = schoolSpendingDF["Average Reading Score"].map("{:.2f}".format)
schoolSpendingDF["% Passing Math"] = schoolSpendingDF["% Passing Math"].map("{:.2f}%".format)
schoolSpendingDF["% Passing Reading"] = schoolSpendingDF["% Passing Reading"].map("{:.2f}%".format)
schoolSpendingDF["% Overall Passing"] = schoolSpendingDF["% Overall Passing"].map("{:.2f}%".format)

# Display the DataFrame
schoolSpendingDF
```

### Scores by School Size
```python
# Establish the bins.
bins2 = [0, 1000, 2000, 5000]
groupNames2 = ["Small (<1000)", "Medium (1000-2000)", "Large (2000-5000)"]

# Categorize the spending based on the bins 
# Use the numeric DF (schoolSummaryNumericDF)
schoolSummaryNumericDF["School Size"] = \
pd.cut(schoolSummaryNumericDF["Total Students"], bins2, labels = groupNames2)

# Calculate averages for the desired columns. 
sizeAvgMath = schoolSummaryNumericDF.groupby(["School Size"])["Average Math Score"].mean()
sizeAvgReading = schoolSummaryNumericDF.groupby(["School Size"])["Average Reading Score"].mean()
sizeAvgMathPassing = schoolSummaryNumericDF.groupby(["School Size"])["% Passing Math"].mean()
sizeAvgReadingPassing = schoolSummaryNumericDF.groupby(["School Size"])["% Passing Reading"].mean()
sizeAvgBothPassing = schoolSummaryNumericDF.groupby(["School Size"])["% Overall Passing"].mean()

# Assemble into DataFrame
schoolSizeDF = pd.DataFrame({
    "Average Math Score": sizeAvgMath, 
    "Average Reading Score": sizeAvgReading,
    "% Passing Math": sizeAvgMathPassing, 
    "% Passing Reading": sizeAvgReadingPassing,
    "% Overall Passing": sizeAvgBothPassing
})

# Format Data Frame Using Map
schoolSizeDF["Average Math Score"] = schoolSizeDF["Average Math Score"].map("{:.2f}".format)
schoolSizeDF["Average Reading Score"] = schoolSizeDF["Average Reading Score"].map("{:.2f}".format)
schoolSizeDF["% Passing Math"] = schoolSizeDF["% Passing Math"].map("{:.2f}%".format)
schoolSizeDF["% Passing Reading"] = schoolSizeDF["% Passing Reading"].map("{:.2f}%".format)
schoolSizeDF["% Overall Passing"] = schoolSizeDF["% Overall Passing"].map("{:.2f}%".format)

# Display results
schoolSizeDF
```

### Scores by School Type
```python
# Create new series using groupby 
# Calculate averages for the desired columns. 
typeAvgMath = schoolSummaryNumericDF.groupby(["School Type"])["Average Math Score"].mean()
typeAvgReading = schoolSummaryNumericDF.groupby(["School Type"])["Average Reading Score"].mean()
typeAvgMathPassing = schoolSummaryNumericDF.groupby(["School Type"])["% Passing Math"].mean()
typeAvgReadingPassing = schoolSummaryNumericDF.groupby(["School Type"])["% Passing Reading"].mean()
typeAvgBothPassing = schoolSummaryNumericDF.groupby(["School Type"])["% Overall Passing"].mean()

# Assemble into DataFrame
schoolTypeDF = pd.DataFrame({
    "Average Math Score": typeAvgMath, 
    "Average Reading Score": typeAvgReading,
    "% Passing Math": typeAvgMathPassing, 
    "% Passing Reading": typeAvgReadingPassing,
    "% Overall Passing": typeAvgBothPassing
})

# Format Data Frame Using Map
schoolTypeDF["Average Math Score"] = schoolTypeDF["Average Math Score"].map("{:.2f}".format)
schoolTypeDF["Average Reading Score"] = schoolTypeDF["Average Reading Score"].map("{:.2f}".format)
schoolTypeDF["% Passing Math"] = schoolTypeDF["% Passing Math"].map("{:.2f}%".format)
schoolTypeDF["% Passing Reading"] = schoolTypeDF["% Passing Reading"].map("{:.2f}%".format)
schoolTypeDF["% Overall Passing"] = schoolTypeDF["% Overall Passing"].map("{:.2f}%".format)

# Display results
schoolTypeDF
```
