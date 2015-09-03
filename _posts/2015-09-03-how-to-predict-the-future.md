---
layout: post
title:  "How to Predict the Future"
author: Scott Lowe
date:   2015-09-03 14:03:00
categories: planning
featured_image: /images/preordain.jpg
---

In Q2, the dev team invested a lot of time in tooling and infrastructure. We rewrote the entire [Castle web app][app] using all the lessons we've learned along the way, and we've seen huge gains in productivity since, but it didn't come without a cost.

Every engineering decision is a tradeoff, and the decision to invest our time in tools and infrastructure as opposed to directly in features meant stagnation for our users, both our external customers and our internal team. Worse still, I seriously underestimated the amount of work involved in a full rewrite so we missed the deadline by a large margin, and that took a toll on the entire team's morale. So for Q3, the dev team has focused on reliably shipping new features, and here's what we've learned about setting realistic deadlines in the process:

##  understand the destination

The better we understand where we are headed, the more accurately we'll be able to predict when we will arrive. In software, this means designing our solutions with enough detail to remove all ambiguity from the end result, **before we start building anything**. To this end, Castle has adopted a three-step process to make sure we accurately flesh out the details of each new feature we build: first we begin with a simple text description of the feature, then we create wireframes to outline how the users will interact with the feature, and finally, we build a list of all the technical details that need changed under the hood in order to make the wireframes function as expected.

##  measure your velocity

Knowing the distance to our destination won't really help unless we know how quickly we can travel. Physics tells us that velocity is just distance over time which means that given a distance and velocity we can calculate our travel time, and we can directly apply this knowledge when setting deadlines. By measuring how many items we regularly check off of our technical details checklists each week, we arrive at a measure for the team's velocity (Castle uses a four-week running average). Obviously, it takes time for this data to come in, so if you're just starting, don't commit to any deadlines until you have a baseline measure after week one.

##  account for surprises

Projects often take longer than expected, and in my experience, surprises are usually the culprit. Understanding the destination helps to mitigate this risk, but the reality is that it's often very difficult to see all the obstacles clearly before beginning the journey. Thankfully, this step is actually the easiest of the three: just include a simple multiple when calculating your deadlines. For beginners, I recommend tripling the result of dividing distance by velocity. As you gain experience with understanding your destination and find yourself able to reliably hit, or better, beat your deadlines, decrease your multiple.

Follow these simple rules and you, too, can predict the future.

[app]:  https://app.entercastle.com/
