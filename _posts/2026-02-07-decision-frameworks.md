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

# Am example from the financial industry

In a past project with a company in the financial services industry, the sales team raised a familiar concern: they were not getting enough leads.

In this case, leads came from two main funnels. One funnel targeted prospects, people who were not yet customers. The other targeted existing customers, who could be interested in additional products. These two funnels behaved very differently, but were often discussed together under the same label: “leads”.

When we analyzed the data, both funnels showed strengths, but on different dimensions. The existing customer funnel converted very well. Trust was already established, and sales conversations were easier. However, the total volume of leads was limited by the size of the existing customer base.

The prospect funnel showed the opposite pattern. Conversion rates were lower and journeys were more complex, but the potential volume was much larger. There were simply many more people who could enter this funnel.

At that point, the question was no longer which funnel performs better. Each performed well in a different way. The real question was where to invest optimization effort.

Using a simple decision framework helped make the tradeoff clear. The prospect funnel had higher potential but lower efficiency. The existing customer funnel had higher efficiency but limited potential. Putting both options on the same dimensions made the discussion easier and more focused.

<img src="{{ site.url }}{{ site.baseurl }}/images/article-6-decision-frameworks/image-1.png" alt="linearly separable data">

The framework did not produce a single correct answer. Instead, it made the tradeoff explicit. In this situation, the company had recently increased sales capacity, so lead volume mattered more than lead efficiency. Given that context, prioritizing the prospect funnel was a deliberate and informed choice.

The important point is that the framework did not change the data. It changed how the decision was made. The conversation moved away from debating metrics and toward discussing priorities, constraints, and timing.