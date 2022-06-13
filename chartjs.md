# Visualize Data in Your Flask App Using ChartJS

You can tell very powerful stories using data. 

![ChartJS Demo](images/data_visualization/chartjs/chartjs_demo.png)

Should you want to 'see' or understand deeply the data generated in your application, there are a handful of libraries that can help. One of them is ChartJS, the focus of this article. [ChartJs](https://www.chartjs.org/docs/latest/) is a free JavaScript library for creating charts in the browser (HTML-based charts). It is very easy to use, though basic understanding of JavaScript is required.

## What We Will Do?

We will build a simple flask application for a class teacher to record the mean scores of all the subjects in a class throughout a 3-term year. Example data we will need may include the following:

* **Subjects**: Math, English, Science, History, and Computer Science
* **Mean Scores**: [70, 80, 90, 100, 95]
* **Term**: [1, 2, 3]

We will assume that the teacher has already calculated the mean scores of each subject, so we don't need to stress about it.

## Table of Contents

1. Create a simple flask app
2. Add web forms to the app
3. Enable user login
4. Display the mean scores of each subject per term
5. Display the mean scores of each subject in a line chart during each term