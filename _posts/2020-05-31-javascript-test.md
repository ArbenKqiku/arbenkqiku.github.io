---
title: "javascript test"
date: 2020-05-31
tags: [docker, google cloud, big query, cloud runner]
permalink: "javascript-test"
header:
  image: "/images/article-3-docker-tutorial/nasa-picture.jpeg"
excerpt: "Create a Docker Image and deploy it on Google Cloud as a Cron Job by using R"
mathjax: "true"
---

```javascript

function getCurrentDay(){
  
  var today = new Date();
  
  var weekDay = today.getDay();
  
  var weekDays = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
  
  var currentWeekDay = weekDays[weekDay];
  
  return currentWeekDay;
  
}

```