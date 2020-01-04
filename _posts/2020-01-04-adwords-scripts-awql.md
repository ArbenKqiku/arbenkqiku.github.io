---
title: "Extract data with AdWords script and AWQL (AdWords Query Language)"
date: 2020-01-01
tags: [digital marketing, adwords scripts, google ads]
header:
  image: "/images/article-1-google-adwords-scripts/image-9.jpg"
excerpt: "Learn how to extract data with AdWords scripts and AWQL (AdWords Query Language)"
mathjax: "true"
---

# Index
In this post, you will learn:

* What are Google AdWords API reports
* Access account data at the MCC level by creating an AdWords script
* How to build a query by using the AdWords Query Language (AWQL)
* How to access report data by using a query

# Google AdWords API reports

Google AdWords API reports are simply reports that focus on one particular kind of data. For instance, the “keywords performance report” allows to query data related to keywords. In general, you can combine 3 types of fields to make a report:

* *Attribute fields*: this is data related to settings or other fixed data, such an Ad group’s status, bidding strategy, etc.
* *Segment fields*: here you can find dimensions that will allow you to further segment your data. For example, you can segment data by date, conversion type or device.
* *Metric fields*: these are quantitative indicators such as conversions, average cpc, conversion rate, etc.

The complete list of attribute, segment and metric fields available for the “keywords performance report” can be found [here](https://developers.google.com/adwords/api/docs/appendix/reports/keywords-performance-report).

In the image below, you can see all the attribute, segment and metric fields available for the “keywords performance report”:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-1-google-adwords-scripts/image-10.jpg" alt="linearly separable data">

The “keywords performance report” is just an example. In fact, on the left side of the link that you just opened, you’ll see the full list of reports available.

<img src="{{ site.url }}{{ site.baseurl }}/images/article-1-google-adwords-scripts/image-1.jpg" alt="linearly separable data">

If you want to query a particular report, it is important to first look at what fields are available for that report. For instance, the attribute field “criteria”, which refers to keywords, is not available in the “ad group performance” report.

# Access account data from MCC

The first thing that you have to do, is to create an empty AdWords script. To do so, go under tools & settings > scripts and click on the blue plus on the top left corner.

Afterwards, you’ll need to authorize the script to make changes on your behalf.

<img src="{{ site.url }}{{ site.baseurl }}/images/article-1-google-adwords-scripts/image-2.jpg" alt="linearly separable data">

Let’s start writing some code. First of all, I’ll create this script at the MCC level. If you created the script at the account level, no problem, you can skip this step. Here is the code:

```javascript
function main(){

// replace xyz-xyz-xyzy with an account number
var accountId = ["xyz-xyz-xyzy"];
MccApp.select(MccApp.accounts().withIds(accountId).get().next());

}
```
To start with, it is important to define a variable which contains the account that we are targeting, namely the variable accountId.

```javascript
var accountId = ["xyz-xyz-xyzy"];
```

In this following part, we are telling Google Ads to look for the account with the id of the variable previously defined, namely accountId:
```javascript
MccApp.accounts().withIds(accountId);
```
However, Google Ads doesn’t return that account yet, you need to give an explicit order, with the following piece of code. Without it, you would be empty handed.
```javascript
get().next();
```

Also, we still didn’t select anything, the account is simply in the memory of Google Ads, but we still cannot get data from it. By using the next piece of code, we are explicitly selecting that account. From now on, all the data that we’ll be retrieving will come from the account that we have just selected:

```javascript
MccApp.select();
```

# Building a query with the AdWords Query Language (AWQL)

In order to extract data from a report, we need to create a query. In order to create a query, we need to use the AdWords Query Language (AWQL). The AWQL is very similar to SQL, but there are some major differences that I’ll not discuss here. Here is the [official page](https://developers.google.com/adwords/api/docs/guides/awql) if you would like to know more about it.

Anyway, let’s dissect the query that we are going to build to retrieve data:

```javascript
// define query
  var query = "SELECT CampaignName, AdGroupName, Criteria, Impressions, Clicks, Conversions" +
              " FROM KEYWORDS_PERFORMANCE_REPORT" +
              " WHERE Conversions > 2" +
              " DURING 20190505,20191031";
```

This query is composed of four elements (although there are more that you can find on the AWQL official page that I listed above), also called clauses:

* *SELECT*: here you define the fields that you would like to have in your report. It is important to use the same field names found in each report’s page. For instance, on the “keywords performance report”, keywords are defined as “criteria”. Don’t ask me why, I don’t know, I just follow the rules. Also, it is important to separate each field by a comma.
*	*FROM*: here you define the report from which you are getting the data. In our case, we are using the “keywords performance report”. It is important to separate each word with an underscore.
*	*WHERE*: this is sort of a filter. In our case, we’re getting keywords that have generated more than 2 conversions.
*	*DURING*: here we define the period of reference. In this case, from the 05.05.2019 until the 31.10.2019. It is also possible to define more dynamic periods such as THIS_MONTH or YESTERDAY.

You may have noticed that I have added a + between each clause. We could have built the query without it, but I find it more readable in this way. In JavaScript, when adding a + between 2 strings, you are basically concatenating them.

Example:

```javascript
var a = "Google";
var b = " AdWords";
var c = " Scripts";

var finalString = a + b + c;
```

Can you guess what the result is going to be? Exactly: Google AdWords Scripts. In Google AdWords scripts, if you want to see the result of a variable, type:

```javascript
Logger.log(name of your variable);
```

In our case, that would be:

```javascript
Logger.log(finalString);
```

Now that we have built our query, let’s jump to the next step.

# Accessing the data defined by the query:

Here is the code:

```javascript
var report = AdsApp.report(query);
var rows = report.rows();

while(rows.hasNext()){
    Logger.log(rows.next());
  }
```

This following piece of code, allows us to select a particular report. The argument of this piece of code is an AWQL query. Luckily for us, we can use the query that we have previously defined. In essence, this piece of code says: give me the report defined in the query, and store it in the variable named report:

```javascript
var report = AdsApp.report(query);
```

The next step, is to get all the rows of that particular report. We do this by using the following piece of code. The variable named rows stores all the rows that match the query previously defined. Without it, we cannot access the data contained in the report:

```javascript
var rows = report.rows();
```

Now I’ll spend a couple of minutes on the while loop. With a while loop we are basically saying this to Google Ads: as long as the condition in the round brackets is true, keep doing the instructions mentioned inside the while loop:

```javascript
while("thisIstrue"){
    "doThis"
  }
```

The following piece of code, basically means “as long as there is another row, keep going”:

```javascript
rows.hasNext(); //is there another row? Value can be either true or false
```

The following piece of code means "print the next row":

```javascript
Logger.log(rows.next());
```

For instance, let’s image that we had a table with three rows:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-1-google-adwords-scripts/image-11.jpg" alt="linearly separable data">

After the first iteration, rows.hasNext(); will bump into the following row:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-1-google-adwords-scripts/image-3.jpg" alt="linearly separable data">

So, given that a row exists, the result of row.hasNext() is true, so it will execute the code inside the for loop, namely:

```javascript
Logger.log(rows.next());
```

In the next iteration, row.hasNext() will bump into the second row:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-1-google-adwords-scripts/image-4.jpg" alt="linearly separable data">

Given that there is another row, row.hasNext() is true, so code inside of the while loop will be executed. With Logger.log(rows.next()), the second row will be printed.

Later, row.hasNext() will bump into the last row, namely:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-1-google-adwords-scripts/image-5.jpg" alt="linearly separable data">

Therefore, row.hasNext() will be true and the code inside the while loop will be once again executed. As a consequence, Logger.log(rows.next()) will print the third row.

After the third row there are no more rows, so row.hasNext(), which answers the question: “is there another row?” will be false. Therefore, the while loop will stop.

Now that the code is complete, we can preview the results by clicking on the preview button on the bottom right corner:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-1-google-adwords-scripts/image-6.jpg" alt="linearly separable data">

Now, to preview the results click on Logs:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-1-google-adwords-scripts/image-7.jpg" alt="linearly separable data">

If you have done everything correctly, here is what you should see (I covered the actual campaign names):

<img src="{{ site.url }}{{ site.baseurl }}/images/article-1-google-adwords-scripts/image-8.jpg" alt="linearly separable data">

Each timestamp represents the data of a particular row. From the first row, we can see that that particular keyword has generated 14718 impressions and 23.18 conversions. If you keep going to the right you’ll see the rest of the data, namely the ad group name, keyword and clicks.

Congratulations! You have just created a Google AdWords script that retrieves data from a Google AdWords report . In this case, we’ve used the “keywords performance report”, but you can apply the same process to any report.

[Here](https://github.com/ArbenKqiku/portfolio-code/blob/master/adwords-script-awql-query.js) you can find the entire code used in this post.

If you have any questions, have a look at the official [Google AdWords scripts group](https://groups.google.com/forum/#!forum/adwords-scripts). It is possible that the question that you would like to ask has already been asked. The group is moderated by actual Google employees who are extremely competent and are pretty reactive (they usually reply within a day).

If you are interested in Google AdWords scripts and would like to learn more, you can add me on [LinkedIn](https://www.linkedin.com/in/arben-kqiku-301457117/). Also, in the agency that I work for, [comtogether](https://www.comtogether.com/) we develop custom scripts for our clients. If you have a particular request, hit me a message!

Happy coding,

*Arben*
