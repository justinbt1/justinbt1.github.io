---
title: 'Subsurface Data Science Hackathon'
date: 2019-02-13
permalink: /posts/is-the-average-of-averages-the-same-as-the-overall-average
tags:
  - data science
  - machine learning
  - timeseries
---
## Write up of the UK’s first Subsurface Data Science Hackathon
Recently I attended the London OGA Data Science Hackathon, the first hackathon in the UK to focus on the use of machine learning with subsurface data. The event brought together oil and gas professionals from a range of companies and disciplines including; geoscientists, engineers, developers and data scientists.

The weekend kicked off with a bootcamp on Friday looking at skills a digital geoscientist might need, data wrangling in Pandas, building web apps and APIs in Flask. Then it was dinner and then time to form teams for the hackathon, I joined team Mystic Bit along with some colleagues.

## Team Mystic Bit
Our teams goal was to use machine learning to predict in real time, the facies ahead of the drill bit using well log data. The ability to predict upcoming changes in rock type would allow for faster decision making in oil and gas drilling operations, improving well targeting and increasing drilling safety.

Now we had our goal it was time to get to work, we decided to split into sub-teams each focusing on a particular task, predicting the gamma log ahead of the drill bit, facies prediction using predicted logs combined with local geology and a web-app. Each subteam took a pair programming approach with a geoscientist and data scientist both working closely together, all overseen by our great team leader who kept us all on track, thanks Dan!

## Gamma Response Prediction
Connor and Patrick tackled the task of predicting the gamma ray response ahead of the drill bit. Training Gradient Boosting Decision Tree Regressors on time lagged data already recorded during drilling. Uncertainty was captured using a quartile loss function, the range of which can be seen on the diagram to the left. This was a challenging task that involved extensive feature engineering to build the time lagged data set as well as the training of over 30 machine learning models.

## Facies Prediction
I was on the team working on predicting facies using well log data, however we didn’t have any labelled facies to train our model on so we had to get creative. We generated a synthetic facies log using K-Means Clustering, an unsupervised machine learning algorithm that clustered the data into five distinct facies.

We then used a Random Forest algorithm to identify the most important features, these were then used to train a Random Forest containing 100 separate Decision Trees. Then using leave-one-oil-well-out cross validation, we were able to predict the facies of a blind well with a 94% accuracy average.

Though we ran out of time to combine this with real time syn-drilling prediction this is a good proof of concept to show how it would work in practice. We also tried combining the geology from surrounding wells in the model, however due to extensive faulting in the area this actually made our predictions marginally worse instead of better!

## We won best executed project!
After all this hard work it was great to get to present what we had achieved to the panel of industry experts. And were surprised when we won best executed project, and thrilled with the prize of framed North Sea core sections.

It was really impressive to see what could be achieved in such a short period of time, when geoscientists and data scientists work closely together. To me this makes a good argument for more multidisciplinary teams in the industry, breaking down the walls and embedding dedicated data scientists and data engineers into geoscience teams.

Thanks Agile Scientific and the OGA for organising such a great event!
