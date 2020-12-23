---
title: Flashcards
subtitle: Flashcard
---

<iframe src="../flash.html" width="100%" height="800" frameBorder="0"></iframe>

### FAQs

#### Why

I wanted to write a general security question bank so that people can understand the gaps in their knowledge, and then be taken to resources to help. Most security resources are either awesome lists which are overwhelming, clearly targetted at a job, or not consumable. Quick quizzes with constructive feedback I think help mitigate those problems, and I can expand my own knowledge as people (inevitably) correct me.

#### How is this built

A HTML page that uses a CSS Library called [Bulma](https://bulma.io). The questions and scoring are performed using JavaScript. Data is stored within Github and pulled as a JSON file. The HTML page is served within an iframe to separate JS/CSS from my main website. You can also directly navigate to the HTML page if you wish via [here.](https://www.colecornford.com/flash.html)

#### Where do you source questions / answers ?

I have a JSON file within my github that contains:

1. Questions and Answers
2. Specific answer feedback
3. Relevant resources for education
4. The Security Domain the question helps with

This JSON file was populated by googling common interview questions, as well as a mix of my own industry knowledge, learning from peers, entry level technology, coding questions, and thought experiments.

#### I want to correct you / submit a question.

Send a PR to my github repository please, or ping me on LinkedIn.

#### I cheated / found your answers / broke your system.

Cool! Security is about finding loopholes, so I encourage you to push boundaries. 

As for this little app, you're basically cheating yourself out of a learning experience. There isn't any real sophistication here, the data is public, I'm just trying to help your personal development.
