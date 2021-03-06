TODO:

Geocomputation: https://geocompr.robinlovelace.net/transport.html
Teaching resources: https://www.carlboettiger.info/teaching


# Content updates / additions

* data in/out
* data cleaning
* date processing
* text processing
* linear modeling
* kriging?

code style:
* more line returns  

Viz style:
* log transform color ramps and axis, not data...


# Structure updates

Add a datacamp course/chapter for each/most of the 14 case studies
Use datacamp education for instruction/assignments
Then class time for questions/case studies
Build index of student projects on website to facilitate?
first draft submitted earilier for video feedback.

File Structure

* put all case studies in .Rmd with github markdown output (include summary of project, name, etc.)
* Use travis CI to ensure content builds
* gitignore data
* Grand checklist for completed tasks
* update links to course repository


General repository rules:
* never use an absolute path (e.g. ~/Desktop/RClass/...)
* never commit large files (100MB max, better < 5mb) 
    * instructions if you did this accidentally
* push before your pull error
  ```>>> git push origin refs/heads/master
   ! [rejected]        master -> master (fetch first)
    error: failed to push some refs to 
    hint: Updates were rejected because the remote contains work that you do
    hint: not have locally. This is usually caused by another repository pushing
    hint: to the same ref. You may want to first integrate the remote changes
    hint: (e.g., 'git pull ...') before pushing again.
    hint: See the 'Note about fast-forwards' in 'git push --help' for details.```

Grading changes
30% datacamp
30% final project
30% participation: tasks, and case studies
10% leadership (package presentation, leading a study group)

Package presentation?  IF keeping, use a Rmd structure inside course repositories so they can be opened on the fly.

For grade: include letter to instructor with grade request.

Schedule Page (one row per week with):
date, readings, datacamp, present last week's case study, package presentation, tasks, casestudies.


## Notes on content


## Structure of case studies

* Fold hints in a button
* Fold "Extra Work" in a button


## Typical week/session
Finish last week's case study and commit, read chapter, take DataCamp course, perform tasks and commit to github, meet in class for: questions & case studies

## Datacamp
Weekly courses due before topic in class

### Assigned
* [Introduction to R](https://www.datacamp.com/courses/free-introduction-to-r)
* [Exploratory Data Analysis in R: Case Study](https://www.datacamp.com/courses/exploratory-data-analysis-in-r-case-study)
* [Spatial Analysis in R with sf and raster](https://www.datacamp.com/courses/spatial-analysis-in-r-with-sf-and-raster)


## Tasks to add
early:

* set up datacamp account
* explore available datacamp courses and select 3-5 interesting ones (list these in README)
* Data visualization: https://dcl-2017-04.github.io/curriculum/vis-perception.html


## Git Classroom
* setup git account
* use github classroom
* set up course folder structure
* update profile on github
* Add short description to top-level README in github repository



### Project related tasks
* convert project to separate tasks and upload to git repository

late:
earn an additional 5,000 datacamp XP in a topic/course of your choosing


## Every Semester
* update google sheets links - do I still need to do this with classroom?
* Create new assignment
* Update github classroom links in task 2
*
