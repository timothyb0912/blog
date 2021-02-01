---
toc: true
layout: post
title: "Project Lifecycle(s)"
description: "Data science and research project life-cycles."
categories: [philosophy]
comments: true
hide: false
---
# Project life-cycles in data science and research
1. TOC
{:toc}

Dear Young Tim,

## The horrors of an unplanned project
You will work on many projects throughout your life.
They will be diverse: research projects, data-science projects, software projects, writing-only projects.
Across them all, one truth will remain constant.
For each project, you will want to start with (and work through) a plan for completing every step.

Without such a plan, you risk experiencing familiar calamities.
Sometimes, you build the wrong thing.
In these cases, you have no quick way of finding out whether your proposed idea will work.
As a result, you incur costly errors in time, effort, and money from beginning detailed construction without a vetted plan.

At other times, we build the right thing, but we fail to do so in a predictable manner.
We face a greater amount of unexpected setbacks than necessary.
When we work without a comprehensive[^1] plan, we add unnecessary uncertainty to our projects.
For example, we will be ruefully surprised by any task that we need to complete but fail to expect.
This fragility reduces the reliability of our project timeline.
As you might imagine, an inability to deliver on time degrades the project experience for us, our stakeholders, and our collaborators.

Lastly, working without a detailed plan and overview of the required tasks is costly.
This cost remains even if we manage to build the right thing and complete the project on time.
Fundamentally, working without a plan hinders our efforts.
In particular, it increases our probability of acting inefficiently.
Without a plan, we have a greater chance of taking actions at the wrong time, e.g., premature optimization.
Relatedly, we may feel overwhelmed without a plan that specifies a time for each necessary task.
And in this state, we may incorrectly try to do too much or too little at once.

Moreover, without a plan, our stakeholders will be full of questions.
They will justifiably want to know what is happening in the project and when they can expect particular results.
As punishment, you will have to spend your time responding to the random arrival of ~disturbances~ questions.
In other words, if you are working without a plan, it means your collaborators and stakeholders are too.

## The benefits of planning
Conversely, if we operate with a plan, then we can reap many benefits.
For example, we are able to share the plan with our colleagues and stakeholders.
The plan can then set our partners' expectations about what is happening with the project at any time.
Additionally, thorough plans have further uses.
They reduce the number of unexpected events in our project delivery process.
Detailed plans thereby reduce variability and increase the usefulness of our project timelines.
And finally, a detailed plan affords us an opportunity to use best practices at every step.

This last point is critical.
A thorough project plan breaks the project delivery process into logical steps.
After separating and grouping concerns, we frequently consider best practices for these sub-components.
E.g., imagine that we've identified data processing as one project delivery step.
Then, data validation would increase our data processing quality (i.e., trustworthiness).
Similarly, imagine that statistical modeling is another step.
Then, experiment tracking would increase our modeling quality (i.e., reproducibility).
In each case, we use the structure of our plan to intervene on our project to promote the best possible outcome.

## Project lifecycle overview
Okay, we've described the consequences of working with and without a comprehensive project plan.
To ease future planning efforts, we'll review a common lifecycle for research and data-science projects.
At a high-level, such projects entail the following steps.
- **Observe** the current state of your system of interest.
- **Specify** what one wishes to change about this system and what the requirements of one's users and stakeholders are.
- **Understand**, or learn, why the system does not already behave as one wishes.
- **Sketch** a procedure, a software architecture, and a timeline for effecting your desired system change.
- **Prototype** the sketched design: achieve a minimally acceptable version of one's desired changes.
- **Design**, in detail, a procedure and software architecture for effecting all desired system changes.
- **Estimate** the amount of time required for construction, and share this information with stakeholders.
- **Construct** the designed architecture, execute the designed procedure, and ensure that we test all and externally review all intermediate steps.
- **Communicate** the constructed architecture, procedure, and educational material.
- **Maintain** the constructed system, collect and act on user feedback, and make continual refactorings for simplicity.

We'll describe each of these steps in greater detail below.

### Observe
This is where we passively observe our system in action, and we make note of the state or behavior that motivates our project.
For example, does our travel mode choice model perform worse for cyclists than for automobile drivers?
Is our production model making any systematic errors?
Under what conditions do we observe the unwanted system state or behavior?
In other words, this first step establishes what the problem or reason for the project is.

### Specify
This is the step where we detail the user, technical, and stakeholder requirements of the project.
Perhaps most importantly, what effect to do we wish bring about as a result of this project, and what system state should exist upon completion of the project?
What are the answers to the who/what/where/when/why/high-level-how questions of our project?
For instance,
- What requirements do we need to meet to ensure the end user of our project achieves the desired state or behavior?
   - What inputs must we take from users?
   - What outputs must we provide for users?
   - What contexts and characteristics of the users do we need to account for?
- What technical requirements must our project fulfill?
E.g., are there
   - latency requirements?
   - user interface requirements?
   - code quality standards?
   - testing requirements?
   - requirements needed for credible causal inference?
   - statistical requirements?
- What stakeholder requirements do we need to meet?
   - Who are our stakeholders?
   What do they care about?
   - Do we need to meet certain service level agreements (e.g. predictive accuracy)?
   - What metrics will we use to judge the outcomes, effects, and impact of this project?  
   How will we define the utility of this project, especially relative to the action of keeping the status quo?  
   Do we need to improve particular metrics by certain amounts?
   - Do we need to produce particular reports and documentation with any specified formats or elements?
   - Do we need to create services or software packages to programmatically expose our work to others?
   - What are our budgetary constraints in time and cost for this project?

### Understand
In this stage of a project, we aim to reach a shared understanding with our stakeholders.
We want to align our understanding of the primary problems and solutions.
Specifically, we want to find out what causes the undesired state or behavior of our system.
We also want to develop hypotheses about how we can alter the system to achieve our goals.

The work in this stage spans multiple dimensions: engineering, causal, statistical, communicative.
E.g., in this stage, we form causal graphs that are plausible to our stakeholders and (if possible) at least some users.
Based on one or more agreed upon causal graphs, what interventions do we hypothesize will lead to the project goal?
In terms of engineering, this is where we begin making substantive decisions about our data engineering pipeline.
We'll decide what data to gather, what transformations of that data are useful for our problem, and where we should store the refined data.
From a statistical perspective, we thoroughly explore our data to understand it and our problem.
And, from a communications standpoint, we shift our focus from our problem statement to our (tentative) solution methods.
We begin to think of how we would describe the motivations, intuitions, and mechanisms for our proposed solutions.

### Sketch
In this part of the process we document and clarify the thinking that began in the last stage.
We:

Sketch the architecture of the software.  
Sketch the data preparation / causal inference / statistical inference / decision analysis workflows.  
Sketch the testing procedure for all parts of the software and models.  
Sketch how to assess business / research / end-user value.  
Sketch the outline of any communication materials.  
Sketch the project timeline.

### Prototype
In the prototyping phase of one's project, we have options.
At minimum, we build a minimal working prototype of whatever software, scientific, and writing products we need.
However, we may take actions such as building multiple types of prototypes to help ourselves answer differing questions.

Some words about tools are in order here.
For prototyping causal and statistical inference procedures, Jupyter notebooks will likely be your tool of choice.
You'll typically want to estimate at least one statistical model that appears reasonable.
You'll want to produce one causal graph that appears reasonable, and go all the way through a heavily caveated causal (i.e., counterfactual) simulation.
For writing projects, the prototype is typically the outline and the rough draft.
For software projects, prototyping typically involves building out some functionalities and meeting some requirements but not others.
Business-wise, prototyping often means building out at least one functionality to the point that users can use it and provide feedback.

### Design
Once the prototype is complete, you will be in a much better state to understand what is truly needed to fulfill the requirements of one's projects.
At this point, once should create a detailed design and set of plans for iterating to the first complete version of one's project: the first version that meets all specified requirements.
Designing should happen at multiple levels.
In particular, one should detail how the project should function internally to produce the desired outputs.

From the perspective of having to write code for one's project, the following elements should always be part of one's design.
One should pay purposefully plan the architecture of one's code.
What are the major "objects" of one's system?
What data should they store and what behaviors should they have?
How should these objects relate to each other?

In tandem, one should plan a testing strategy to make sure the code functions as intended and meets all requirements.
Note of course that testing should take place on various levels.
For example, one should test:
- individual functions and methods with unit tests,
- data to make sure it conforms to expectations and that we process it correctly,
- conformance with user requirements through acceptance tests and behavior driven development,
- the project's inference methods with its datasets to ensure that one produces sound inferences and that one understands the limitations of one's techniques in context,
- any estimated models through integration tests that check for counterfactual and inferential sensibility of model results,
- the building of all communications materials,
- for graceful error handling.

And if we're being truthful about how frequently we make errors, we will also include a design for how we will refactor and fix bugs.
What metrics will we track about our source code to help ensure code quality, identify code smells, and guide refactoring?
How will we make sure our code follows and uses relevant (compositions of) design patterns?
How will we perform such tracking automatically?
How will we make it easy for ourselves and others to understand the current design of our code so they can help refactor?

A similar set of concerns exist with our inferential activities.
We should carefully design our causal inference and statistical workflow, from end to end.
Given that this workflow will have a matching computational graph, we should be specific about each step's inputs and outputs.
This workflow should include:
- data engineering pipeline
- exploratory data analysis,
- (causal) model specification,
- prior predictive checks,
- statistical testing of our causal model,
- statistical testing of our inferential techniques using prior predictive simulation,
- model estimation / inference,
- model iteration,
- experiment tracking,
- model evaluation,
   - posterior predictive checks
   - cross-validation
   - model-explanation
   - model-exploration
- counterfactual (decision) analysis.  
In the best case scenario, we would also plan how to analyze the conditions under which our techniques fail to work.  
How does our technique degrade as we come close to satisfying such conditions?

Lastly, we need to design our communications process.
Since we want to share our work with others, we should design a process that will maximize the chance of successful communication.
At minimum, our process should include:
- sketching out initial communication ideas,
- creating first outlines,
- producing communication drafts,
- editing our materials,
- producing the materials (e.g. building website, docs, article, etc.),
- circulating the final documents.

Additionally, we may need to repeat our communications process across audiences and materials.
E.g., people who may later edit or read our code need technical documentation.
People who wish to understand the details of our analysis need scientific reports.
People who briefly engage our work need high-level overviews of our main findings.
Servers that will programmatically call or import our work need APIs, HTTP endpoints, and data infrastructure.
So on and so forth.

### Estimate
Once we've created a detailed design for our project, we can begin estimating how long it will take us to complete it.
We can create a project timeline.
At this point, I will merely give general guidance here and point you to others who have thought about this much more than myself.

First, it is best to under-promise and over deliver.
In other words, be conservative in one's project timeline estimation.
There are always known unknowns.
These are the tasks that we know we cannot complete immediately, without reference to outside sources.
Then there are unknown unknowns.
These are truly unforseen difficulties that slow down one's efforts.
Unfortunately, unknown unknowns happen at all time scales: months, weeks, days, and hours.
Accordingly, to deal with unknown unknowns, include estimates for frequent and unplanned time blocks of varying lengths (hours, days, weeks, and months).

Second, it helps to have some rules of thumb when doing project estimation.
For week-long or longer projects, I find it helpful to multiply my original (i.e., optimistic) time estimate by 3.
This adjustment increased timeline accuracy, especially for tasks I have not completed before.
For shorter efforts on the order of an hour or two, I have found that 1 clock-hour typically corresponds to a planned 20-30 minutes worth of 1-2 minute actions.
Inevitably some of the short actions take longer than expected and I end up with an elapsed hour.

Third, one should expect one's schedule to change.
Plan for this.
One should store one's schedule in a format that facilitates rapid change.
Recently, I've been using spreadsheet programs that allow me to create a new sheet whenever I have a large change to the schedule.
I change existing cells for small schedule changes.
Find what works for you!

Fourth, plan on having a steady stream of outputs for one's stakeholders.
One should plan one's projects in waves such that on a regular cadence one has full implementations of various desired features.
Starting from a prototype, one should implement in vertical efforts that result in one complete product iteration at a time.
This will ensure your stakeholders are always in a position where they have a product to use and evaluate.
The alternative is developing horizontally.
Here, one's project is built incrementally and one goes from zero finished features to everything all at once.
This strategy leaves one's stakeholders frustrated at being "in the dark" while you tinker for long periods of time.

Fifth, pay attention to resource availability.
Will one have access to the resources one needs at all points in the project or does one have time-varying resource constraints?
One's own time may not have a constant rate of availability.
The availability of one's collaborators may not be constant throughout the project.
Find out as many of these constraints as possible and account for them in one's estimation and planning.

### Construct
Once you have a detailed design and a publicly shared project-plan, you can begin construction.
This is an iterative process of testing, coding, estimating, evaluating, revising, refactoring, and communication.
Feature by feature, one transforms the prototype into a product that meets all of the specified project requirements.

The idea for this stage of development is to follow test-driven development practices.
One should start by writing a failing test that reflects the requirement that you will write code to fulfill.
For direct tests of whether user requirements are fulfilled, it is a good idea to write behavioral tests in the human-readable Gherkin language.
For tests of internal functionality, make use of unit tests as usual.
In either case, first write a failing test, then write the code to pass the test.

Once our code passes all the new tests, we've implemented the new functionality!
Great.
Now, we should use the feature to produce business or scientific value.
I.e., conduct statistical, causal, and decision analyses.
We should make the substantive assessments and predictions that motivated the project.
The idea is to bring the project to a point that one's stakeholders will care about as quickly as possible.
Swift results will comfort your stakeholders and increase your credibility.
Additionally, quick iteration speeds increase one's opportunities to assess and course-correct one's work.

After producing preliminary results, we should take the opportunity to refactor our code.
Think about how to simplify our software.
Check our adherence to design principles.
Check to see if any compositions of design patterns fulfills our code's purposes.
And perform static analyses on our code to see if we've introduced unnecessary complexity or coupling.
Analyze our class diagrams, sequence diagrams, source code metrics.

Lastly, don't forget to communicate during the construction.
This should be taking place at multiple levels.
You should be writing documentation for your code in the form of comments, docstrings, and examples.
You should be updating your stakeholders on major delays that occur.
You should be documenting the results of your analyses as they are so far.
Do not wait until the end of one's project to begin describing one's methods and results.

### Communicate
We should communicate our results and findings continuously, but especially once we have finished a feature or project version.
I consider communication to include actions such as:
- updating online documentation for the software we've built;
- preparing reports that detail our findings;
- preparing web-interfaces to let stakeholders use our tools; and
- preparing packages and HTTP endpoints so users can programmatically access our work.

Note that communication involves many different tasks.
Accordingly, it's important to use an automated workflow to manage this process.
Specifically, use a continuous integration system to ensure that at all times (e.g. nightly or upon merge to our stable branch), one is able to build the documentation, report, and user-interface for one's project.
Through continuous preparation, we reduce the stress incurred when trying to share our work.
Moreover, by preparing our materials, we increase the likelihood of successful sharing.
Helpful tools for these tasks include
- [Sphinx](https://pypi.org/project/Sphinx/) and [Docusaurus](https://docusaurus.io/) for online documentation,
- [Jupyter Book](https://jupyterbook.org/intro.html) and [Latex](https://www.latex-project.org/get/) for reports,
- [Landslide](https://github.com/adamzap/landslide) and [Auditorium](https://github.com/apiad/auditorium) for creating presentations from markdown,
- [Streamlit](https://www.streamlit.io/) for online user-interfaces,
- [FastAPI](https://fastapi.tiangolo.com/) to enable programmatic communication through REST APIs,
- [CML](https://github.com/iterative/cml) and [TravisCI](https://travis-ci.com/) for continuous integration through Github Actions and Travis respectively.

### Maintain
Once we have released a working version of our project for stakeholders to interact with, we should maintain and improve our software.
As with all the other steps in the project lifecycle, this step takes place at multiple scales.
At the most basic level, we should be able to reproduce our analysis and use the software developed as part of the project.
We can verify this with the continuous integration techniques described in the last section.
Whenever the continuous integration pipeline raises an error because it cannot complete its tests, we should update our project to fix these issues.

At a higher level, we can maintain the software by continually refactoring to simplify the design of the code while preserving functionality.
This level of maintenance is a preventive action.
Refactoring makes bugs (i.e., errors in one's construction and/or design) easier to fix when they happen and easier to avoid in general.
Moreover, refactoring leaves the code in a simpler state than it statrted.
Other developers or researchers can then leverage this simplicity to extend the software for other uses.

Finally, one should respond to bug reports when other users discover them.
Bug fixes will make one's code even better by repairing previous incorrect or undesired behavior, potentially even giving one's code new capabilities.

## Footnotes
[^1]: Comprehensive in this case includes some amount of "buffer time" to deal with unknown, but expected complications or problems.

<!-- ## Related
  Involved concepts
     1. Causal inference workflow
        1. Draw a-priori causal graph
        2. Collect data
        3. Test causal assumptions in a-priori graph.
        4. Use causal discovery to explore uncertainty over causal graphs.
        5. Identify whether treatment effects are identifiable
        6. Estimate functional causal model for one's graph.
        7. Use models + simulator (i.e. complete generative model) to get posterior distribution over treatment effects (and over probability of one decision being better than another).
        8. Run real experiment
        9. Update one's distribution of belief in each treatment being the best action to take.
     2. Statistical analysis workflow
        0. Exploratory data analysis
        1. Visualize and understand priors via prior predictive checks.
        2. Select a working prior that is not horribly contradicted by one's data.
        3. Use working prior to assess calibration of one's inference techniques given one's dataset.
        4. Use inference procedures that have been validated through simulation-based calibration to make inferences.
        5. Perform posterior predictive checks to understand one's fitted model.
        6. Adjust one's model as necessary to ensure that the relevant aspects of reality are all reliably reproduced by one's model.
     3. Software development workflow
        1. Understand the problem
        2. Craft rationale and vision statement.
        3. Delineate all the users and stakeholders of the software
        3. Collect requirements for project from all perspectives, users, and stakeholders.
        4. Prototype.
        5. Create architecture.
        6. Create project timeline.
        7. Implementation loops (fail test, implement code to pass test, refactor and pass test).
        8. Software maintenance
     4. Writing workflow
        1. Understand problem.
        2. Determine what we want to say about the problem.
        3. Draft an outline of our statement
        4. Implement a first draft following the outline.
        5. Determine all computational outputs that are needed for inclusion in the writing.
        6. Revise computational workflows to ensure all artifacts are produced appropriately for inclusion in the written material.
        7. Revise the outline based on the draft.
        8. Edit first draft in light of revised outline.
        9. Edit the second draft.
     5. Business development workflow
        1. Understand problem
        2. Prototype solution to show business value
        3. Implement substantial business changes for production, as quickly as possible and in phases.
        4. Refine based on feedback from users and operations.
     6. Scientific method
        1. Observe.
        2. Question.
        3. Hypothesize.
        4. Predict.
        5. Test.
        6. Analyze.
        7. Replicate.
        8. Subject to external review.
        9. Disseminate.
-->
