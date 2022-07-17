# PyCitySchools

## Background
In this repository, I'm using pandas and jupyter notebook to analyze district-wide standarized test results. I've been giving access to every student's math and reading scores [students_complete.csv](PyCitySchools/Resources/students_complete.csv), as well as various information on the schools they attend [schools_complete.csv](PyCitySchools/Resources/schools_complete.csv). I will be aggregating the data in different views to show trends in school performance.

## Observable Trends
- On average, students in the district tend to do better at reading than at math, which is evident from both the average scores for each exam as well as the percentage of students that passed each exam. The average reading test score is 81.88, compared to 78.99 for math. ~86% of the students in the district passed the reading test, while ~75% passed the math test.
- Schools that spend a higher budget per student do not have higher overall test results. In fact, the schools with the highest overall average test scores are the ones that have the lowest budget per student (<585 dollars).
- Small and medium sized schools outperform large sized schools on overall passing performance (90-91% passing vs. 58%). This is largely driven by a low % of students passing math exams at large schools (~70%), compared to >90% at small and medium sized schools.
