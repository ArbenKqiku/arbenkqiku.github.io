---
title: "Decision frameworks"
date: 2026-02-07
tags: [decision-frameworks]
permalink: "how-to-implement-decision-frameworks"
header:
  image: "images/article-4-kafka-streaming/21-schedule/project-diagram.png"
excerpt: "decision-frameworks"
mathjax: "true"
---

# Table of Contents
[Introduction](#introduction)

# Introduction
The analytics industry is obsessed with *data-driven decision-making*. On the first part, *data-driven*, we’ve done an outstanding job. We have powerful tools that let us measure almost every interaction users have with our websites and products.

But when it comes to the second part, *decision-making*, the industry seems stuck. We produce more data than ever, yet this abundance does not always translate into better decisions.

# What is a decision framework?
A decision framework is a way to structure choices so they can be discussed and evaluated deliberately. It makes choices *explicit* by clearly defining the alternatives. And it makes them *comparable* by evaluating those alternatives along a shared set of dimensions, so tradeoffs become visible.

Most business decisions can be seen as a tradeoff between two dimensions: potential and efficiency.

Potential is about how big the opportunity is. It answers the question: if this worked well, how much impact could it have? In digital marketing, potential can be approximated by things like audience size or monthly search volume.

Efficiency is about execution quality. It describes how effectively that potential is turned into results. In digital marketing, this can be measured using metrics such as conversion rate, cost per lead, or profit on ad spend.

# An example from the financial industry

In a past project with a company in the financial services industry, the sales team raised a familiar concern: they were not getting enough leads.

In this case, leads came from two main funnels. One funnel targeted prospects, people who were not yet customers. The other targeted existing customers, who could be interested in additional products. These two funnels behaved very differently, but were often discussed together under the same label: “leads”.

When we analyzed the data, both funnels showed strengths, but on different dimensions. The existing customer funnel converted very well. Trust was already established, and sales conversations were easier. However, the total volume of leads was limited by the size of the existing customer base.

The prospect funnel showed the opposite pattern. Conversion rates were lower and journeys were more complex, but the potential volume was much larger. There were simply many more people who could enter this funnel.

At that point, the question was no longer which funnel performs better. Each performed well in a different way. The real question was where to invest optimization effort.

Using a simple decision framework helped make the tradeoff clear. The prospect funnel had higher potential but lower efficiency, whereas the existing customer funnel had higher efficiency but limited potential, as displayed in the image below:

<img src="{{ site.url }}{{ site.baseurl }}/images/article-6-decision-frameworks/image-1.png" alt="Decision Matrix showing Prospects in the Higher Potential/Lower Efficiency quadrant and Existing Customers in the Lower Potential/Higher Efficiency quadrant.">

# Context matters

Data alone is not enough to make good decisions. It is important to talk to stakeholders, because they often have strategic, operational, or organizational context that is not visible in the data.

In this case, a conversation with the head of sales revealed an important constraint: the company had recently hired many new salespeople. As a result, the immediate priority was not lead quality, but lead volume. Given that context, focusing on the prospect funnel was the option most aligned with what the company was trying to achieve.

This is why stakeholder input is a key part of decision-making.

Our course [Data Analysis With R](https://www.teamsimmer.com/all-courses/data-analysis-with-r/) culminates with the Stakeholder Pressure Test. Throughout the course, we analyze Simo Ahava’s blog. At the end, we pressent the analysis and recommendations to Simo, explaining where additional effort would have the biggest impact on traffic growth.

The purpose of this exercise is to show that conversations with stakeholders help refine the analysis until it leads to decisions that actually matter.

# What is our role as analysts?

Our job isn't just to report data, but to define the proxies that measure potential and efficiency. In [Data Analysis With R](https://www.teamsimmer.com/all-courses/data-analysis-with-r/), we build a custom proxy for efficiency: measuring exactly how much traffic each "second" of published content generates across different topics (ex.: Server-Side Tag Management). We use R for this because it allows us to bake this specific business logic directly into our scripts. This makes the entire analytical process transparent, and reproducible for future decisions.



In the financial industry example, we used the lead to sale conversion rate.