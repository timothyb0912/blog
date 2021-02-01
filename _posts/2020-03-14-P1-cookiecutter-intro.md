---
toc: true
layout: post
title: "On the need for structure... directory structure"
description: "A description of benefits from and suggestions for setting up a good project directory structure."
categories: [practice]
comments: true
hide: false
---
# On the need for structure... directory structure
1. TOC
{:toc}

Dear Young Tim,

It's March again...
In 2014, you will begin conducting your own, original research.
This work will be an ongoing effort.
First, it will become your Master's thesis, and then, part of your doctoral dissertation.

And so, you will start with gusto.
You'll start with much good intent for cyclists all throughout the United States.
And you'll start without a single shred of good practice or organization.

By March 2016, you will have learned the hard way.
You will have spent hundreds of hours tracking down lost or moved files, debugging cryptic errors that never appeared in 2014, and struggling to recall / recreate how your code was designed in the first place.

I'm sorry I couldn't spare you in advance in my time.
But, hopefully, I can be of more assistance now.
So, without further ado, here's what I think you did not understand then about the need for structure in one's project directory, and here's how I think you can do better now.

## Why lacking structure hurts

### You
The first reason why you should have a clear directory structure is that without one, **you** will suffer.
Most acutely, you will suffer when you need to recover old work at the request of others or yourself but you cannot do so easily.
Grant projects and research opportunities that otherwise should have been pleasant experiences will be painful toils as you exert huge amounts of energy to recreate old, unorganized projects.
Indeed... you will feel real sadness when you wish to extend your own work but you cannot do so.

The second reason why you should have a clear directory structure is that you will feel the sting of a thousand little cuts without it.
Not only will you waste time as you retrospectively try to recreate projects, you will waste time in new projects as you reinvent old wheels instead of easily reusing previous work.
E.g., how do we need to package our tex files for uploading to arXiv again?
Eventually, you will add insult to injury as you realize how silly you were to end up in this situation in the first place.

### Society
Beyond yourself, there are at least a couple of additional reasons to use a clear directory structure for each research project.

First, there is no easy way to "stand on the shoulders of giants" without a well structured project, so **all of society** suffers.
Each person who wishes to use your work will face an unnecessarily hard task of learning how to interact with one's work (i.e. how it's structured) in order to use the raw materials that led to your research discoveries.
This is a collective waste of society's time as each person attempts to figure out the same thing (possibly incorrectly).

Secondly, without a clear project structure, it becomes more difficult for society to trust your work.
If someone cannot easily see *how* your project works, then they are less likely to be convinced *that* your project works.
It is then both a waste of your time as a researcher and their time as a research consumer to have to perform additional efforts to validate the project's results because one's work was not immediately understandable.

## How to structure your directories
Here, then, are three guiding principles for how to structure one's project directory to avoid the aforementioned burdens on yourself and society.

### Use a template
Use a template for your project directories.

Templates have at least three reasons supporting their use:
1. They remove much guesswork around setting up a clear project structure.
2. It is far easier to be a critic than a creator.
   You will be more likely to edit a decent project structure than to create a good project structure from nothing.
3. Humans are lazy (see reason two).
   Young Tim, it is likely that you will stick with most defaults of what you begin with.
   Accordingly, it's better to begin with something good rather than begin with something bad.
   I.e. begin a good template instead of a monolith empty directory, or worse, a directory mixed with pre-existing work.

### Keep an open mind
While its good to use a template, it is even better to modify the template as needed to meet one's project needs.

The basic idea is simple.
Be not defined by the structure of the template, but define a structure based on the template.

### Have the big picture in mind
When defining a structure for one's project, keep in mind all aspects of the project.

For instance, your research projects will almost always consist of four parts.
You will likely
- write code to do something (`pl.create_choice_model(...)`)
- produce some analytic results based on data (e.g. charts, tables, statistics, etc.)
- write documents of various kinds (research article, blog post, project documentation, etc.)
- write more code to do things (e.g., pdflatex doc.tex; bibtex doc.aux; pdflatex doc.tex; pdflatex doc.tex)

The structure of your project directory should encompass all the parts above so that you keep the project contained in one location.

## What good structure brings
Once you adopt the project structuring philosophy above, you will begin to experience subtle-yet-tangible benefits to your life. Here are three such benefits.

One: you will become a magician.
Having a template for a good project structure minimizes your recurring research efforts.
And this transfer of a variable cost to a fixed cost is nothing short of magical!
It's these sort of changes that will drive major productivity gains in your life.

Two: you will become a more social reseacher.
I know.
It's hard to believe me since you're already so social...
Hear me out.
You have a sense of pride.
We both know it.
When your project structure is a mess, you will not want to share your work with anyone.
But when you've got a good project structure, you'll proudly share it with everyone and their families.
And we already know that sharing your work is good for you and good for everyone else too.

Three: you will be proud of yourself.
As mundane as it may be, your project structure is another one of your creations.
It's an object whose design you can take pride in.
All your life you've admired people whose creations were deliberate, useful, awesome, and full of humor / good-cheer.
Use your project structure to display the same traits, =].

## Practical resources
Okay, okay.
By now, you're sold on the general idea.
Let's talk specifics then.

You'll typically work on projects that fall into one or more of the following three categories.

1. There will be "analysis" projects where the primary aim is to analyze one or more datasets or demonstrate some method of analysis.
2. There will be software projects where the primary aim is to produce reusable tools in code for yourself and others.
3. And, there will be writing projects where the primary aim is to produce some written document, with or without supporting data analysis and code.

Corresponding to each of these categories there are (or one should make) reusable templates that set up a good project structure for the task at hand.
For analysis projects, I recommend templates based on [Cookiecutter](https://github.com/cookiecutter/cookiecutter) / [Cookiecutter-DataScience](https://github.com/drivendata/cookiecutter-data-science) tools.
For the [PyLogit](https://github.com/timothyb0912/pylogit)'s in your future, I recommend templates based on [Pyscaffold](https://github.com/pyscaffold/pyscaffold) / [Pyscaffoldext-dsproject](https://github.com/pyscaffold/pyscaffoldext-dsproject) tools.
For your writing projects, however, I have a wider range of recommendations...

For blogs, I recommend templates based on tools from the future like [fast_template](https://github.com/fastai/fast_template).
(And yes [you should blog](https://medium.com/@racheltho/why-you-yes-you-should-blog-7d2544ac1045) during [graduate school](https://thesiswhisperer.com/2017/08/23/why-you-should-blog-during-your-phd/)...)
For books, I recommend templates based on [Jupyter-Book](https://jupyterbook.org/intro.html).
For articles, I recommend you make your own =). For some inspiration, see resources such as [the following](https://education.github.community/t/github-latex-for-collaborative-writing/35875/7).

That's all for now.
I have more recommendations (for instance, basing PyTorch projects on the [PyTorch-Template-Project](https://github.com/victoresque/pytorch-template)), but I'll leave you to explore.
Go forth, and set up projects with ease and care!

You and everyone one else will thank you in the future.
