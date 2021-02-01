---
toc: true
layout: post
title: "First things first: On protocols and unit tests"
description: "A description and demonstration of using protocols to start test driven development."
categories: [practice]
comments: true
hide: false
---
# First things first: Protocols, type annotations, and unit tests

## Motivation

Dear Young Tim,

Strive to be more trustworthy.
Strive to be more effective, efficient, and well-prepared for problems.
Strive to be great: to others, to your (future) self, and in how you create.
These goals complement each other.
Together, they help you "be" better, and they point to better ways of being.

Concretely, these goals highlight the primacy of, the need for, and the benefits of automated software testing.
Indeed, trusting your code or your results is hard without tests that provide a basic level of claim verification.
Being well prepared for coding errors is hard without tests to help pinpoint the issues.
And lastly, refactoring or extending your work is unlikely to be a great experience for your future self without tests to confirm your software's functionality.

## The problem
Alright then.
Given its importance, you can understand why [testing before coding](https://en.wikipedia.org/wiki/Test-driven_development#Test-driven_development_cycle) is the right thing to do.
You shouldn't begin a journey without a way of knowing when you've reached your destination.
Unfortunately, purity aside, testing before coding is hard!

This hardness stems, in part, from uncertainty.
Without code, we are uncertain of what we should test.
Even with existing code, it can be hard to know where to begin testing.

## The solution
To address this testing barrier, we'll remove some of our uncertainty through analysis, decision-making, and codification.
In particular, recall September's project lifecycle [description](https://timothyb0912.github.io/blog/philosophy/2020/09/30/Project-Lifecycle.html).

We should first understand the system we want to change.
Specifically, we should perform an [object analysis](https://www.umsl.edu/~sauterv/analysis/488_f01_papers/quillin.htm) of the system.
What are or what should be the system's basic objects and entities?
What are their functions and attributes?

Next, make some design decisions: sketch the basics of how the code should function.
What functions do you call?
What objects do you instantiate?
How do those objects interact?
We'll codify these decisions in function signatures and object protocols.

Finally, we'll test to ensure that we respect our signatures and protocols throughout the codebase.
This is our most basic way to make our software trustworthy: ensuring that it does what we say it does.

## Description
Of course, the described process will be more helpful if you know what I'm referring to.
By function signatures I'm referring to [type declarations](https://www.python.org/dev/peps/pep-0484/) of your functions' inputs and outputs.
Compare
```python
def calculate_loss(predictions, targets):
    ...
```
versus
```python
def calculate_loss(
  predictions: np.ndarray,
  targets: np.ndarray
) -> np.ndarray:
    ...
```
Immediately, the second definition provides answers to questions.
What types of inputs are expected?
What types of outputs are expected?

By [protocols](https://www.python.org/dev/peps/pep-0544/), I'm referring to objects that serve as placeholders and guidelines for how other objects should look and behave.
For illustration, consider the following base class.
It defines required attributes and methods for objects that provide or implement the protocol.
```python
from typing import Protocol

class Model(Protocol):

    num_design_cols : int

    @classmethod
    def from_params(cls, params: np.ndarray) -> "Model":
        pass

    def predict(self, inputs: np.ndarray) -> np.ndarray:
        pass

    def simulate(
        self,
        inputs: np.ndarray,
        num_simulations: int,
        seed : int=25
    ) -> np.ndarray:
        pass

    def save(self, output_path: str) -> bool:
        pass
```
This Protocol specifies that your model classes should have
- a `num_design_cols` attribute of integer type,
- a `from_params` method that instantiates the class from a numpy array,
- a `predict` method that takes a numpy array of inputs and returns a numpy array of outputs,
- a `simulate` method that requires an integer number of simulations and takes an optional, integer random seed, alongside one's input and output numpy arrays,
- a `save` method that takes an output path to save one's model parameters to and returns a boolean indicating success of the process.

Ideally, the attributes and functionality of one's objects will be informed by one's [object analysis](https://www.umsl.edu/~sauterv/analysis/488_f01_papers/quillin.htm).
How do you want to interact with one's object's?
Perhaps you wish to do the following.
```python
from my_project import (
    load_data,
    load_params,
    Model,
    FINAL_PARAMETER_PATH
)

design, targets = load_data()
params = load_params()

model = Model.from_params(params)

predictions = model.predict(design)
targets_simulated = model.simulate(
    design, num_simulations=100, seed=901
)

# Further training and/or analysis
...

model.save(FINAL_PARAMETER_PATH)
```
As written, the protocol above supports such a workflow.

In the best case scenario, protocols will help you prototype faster.
They'll focus your attention on high level design decisions, sans implementation details.
Thinking through and solving problems at this stage can save hours and weeks of effort later on.

Additionally, protocols should increase the modularity of your code.
By referring to protocols in one's signatures, you can change any providing object's implementation choices without affecting any other parts of one's code, so long as the provider adheres to the protocol.
Indeed, protocol adherence enables the abstraction needed to support each provider's uniqueness in their implementation.
Having commonality in method/attribute presence and type signature is what affords other objects/clients the luxury of ignoring the details of how each provider does what it does.

## Implementation
Once we've defined the type signatures and protocols for our project's functions and methods, we're ready to begin testing.
Specifically, we should test the basics.
We'll start with testing the "[happy path](https://en.wikipedia.org/wiki/Happy_path)."
Our type signatures declare that given "valid" inputs of specified types (however we define valid), we will get back outputs of specific types.
We will test that these statements are true.

For such contract testing (i.e., input-output type checking), the libraries [Typeguard](https://typeguard.readthedocs.io/en/latest/userguide.html) and [PyContracts](http://andreacensi.github.io/contracts/) should be helpful.
I especially recommend PyContracts because of it's ability to conveniently disable all type checking when not running tests using `contracts.disable_all()`
You can get started after installation by adding the following decorator to functions in your source code with type signatures.
```python
import contracts
from contracts import contract

@contract
def my_func(arg_1: Type1, arg_2: Type2) -> ReturnType:
    pass

contracts.disable_all()
```
Then, one can include following tests such as the following.
```python
import my_project

def test_my_func_signature():
    # Define valid function arguments
    arg_1 = ...
    arg_2 = ...

    # Enable signature testing
    # Comment out the following line if using typeguard
    my_project.contracts.enable_all()

    # Exercise the function
    result = my_project.my_func(arg_1, arg_2)
    return
```
Alternately, one can comment out the `contracts.enable_all()` command in one's test, keep one's source file free of decorators, and use `pytest --typeguard-package=src/my_project .` from one's project root to run one's tests.
Decorating all of one's functions is tedious and `typeguard` provides a useful workaround for such concerns.
Either way, such tests will verify that your code satisfies its advertised type signatures, at least under the tested circumstances.

Next, our protocols declare the necessary attributes, methods, and type signatures of our objects.
We simply need to test that our instantiated objects actually implement the protocol.
This is as simple as creating tests like the following
```python
import my_project
from test_fixtures import load_test_inputs

# Enable contract testing
my_project.contracts.enable_all()

def test_protocol_implementation(
    model_class=my_project.MyFancyModel,
    model_protocol=my_project.Model,
) -> bool:
    # Replace with any alternative instantiation process if necessary
    model = model_class()
    assert isinstance(model, model_protocol)
    return True


def test_protocol_signatures(model_class=my_project.MyFancyModel,) -> bool:
    design = load_test_inputs()
    # Replace with any alternative instantiation process if necessary
    model = model_class()
    # Test that we can execute the function under test without error
    # Unexpected types on input or output we will raise an error thanks to
    # `contracts.enable_all()`
    result = model.predict(design)
    return True
```
Here, we rely on the fact that `isinstance(model, model_protocol)` will raise an error if `model` doesn't have all of the required attributes and implement all of the methods required by `model_protocol`.
We then again rely on `contracts` to perform the actual type signature testing of our method inputs and outputs.

## Extension
The contract testing procedure just described is only as useful as the types it is testing for.
In particular, we will catch more of our own errors as our types become more specific.
For instance, we can catch more errors using `contracts` and
```python
def loss(
    targets: "array[N], (N>0, (0|1))",
    predictions: "array[N], (N>0,>0,<1)",
) -> "int,>=0":
    ...
```
as compared to using standard and less specific type hints with
```python
from numbers import Number

def loss(
    targets: np.ndarray,
    predictions: np.ndarray,
) -> Number:
    ...
```
The former type signature will check for
- binary values in `targets`,
- values in the unit interval within `predictions`,
- unidimensionality and equal shape of both `targets` and `predictions,`
- and non-negative scalars as outputs.

I've silently violated these conditions in the past, and I have spent far too much time searching for the errors because of these types of issues (all puns intended).
No more.
Note that the [python-vtypes](https://smarie.github.io/python-vtypes/) package provides similar and even more general capabilities.
`python-vtypes` has the added ability of letting you define types that serve as drop-in replacements for regular python types, no decorators necessary.
Note that this enables `typeguard` to also check for more specific types than those natively defined in the `typing` module.

## Workflow integration
From the last section, you will have created detailed types.
They should encapsulate the important essence of what you expect from your object attributes, function inputs, and function outputs.
You will also have taken the advice of the [Implementation](##Implementation) section, so you will have defined simple tests of your function signatures.
You should feel great about this!
However, you should go even further.

You should use your initial signature and protocol tests as a starting point for the virtuous cycle of test-driven development.
Specifically, you now have tests but no functioning code.
By design, you have failing tests.
You should now write the most minimal code that passes one of your tests.
Note, this code will likely NOT be the code that you want.
This is done purposefully.
That the incorrect function passes the tests but doesn't do what we want means that we are missing other tests!
We should create a test suite that specifies our function's full set of necessary conditions.
Then we'll write code to iteratively pass each of those tests.
After this process or some number of repetitions of it, we'll have implemented and fully tested a desired unit of code.

## Example
With all of the above in mind, here's a concrete example.
As with [previous posts](https://timothyb0912.github.io/blog/practice/2020/06/25/PyTorch-Tutorial.html), this is part of an effort to revise my paper
> Brathwaite, Timothy. "Check yourself before you wreck yourself: Assessing discrete choice models through predictive simulations." arXiv preprint arXiv:1806.02307 (2018).

Here was the context.
When trying to understand estimated models, I become Picasso.
Well, not the same, but I do make pictures.
Plots, specifically.
Lots of them.

I decided to share this code in a python package, [Checkrs](https://github.com/timothyb0912/checkrs).
It would be great to be able to automatically test that my functions plot what they say they are plotting.
Unfortunately, writing unit tests for matplotlib is a [terrible experience](https://towardsdatascience.com/unit-testing-python-data-visualizations-18e0250430).
Since [Altair](https://altair-viz.github.io/) can output plots in a standardized [Vega-Lite](http://vega.github.io/vega-lite/) JSON, I figured it would be easy to test.

To get started in porting my matplotlib functions to Altair, I followed the process described above.
I started by defining protocols of the basic objects: the chart.
I called it a view because I think of each chart as one way to view one's data.
This protocol would therefore be a base object for all charts.
```python
@attr.s
class View(Protocol):
    """
    Base class for Checkrs visualizations. Provides a view of one's data.
    """
    theme : PlotTheme

    @classmethod
    def from_chart_data(cls, data: ChartData) -> "View":
        """
        Instantiates the view from the given `ChartData`.
        """
        pass

    def draw(self, backend: str) -> ViewObject:
        """
        Renders the view of the data using a specified backend.
        """
        pass

    def save(self, filename: str) -> bool:
        """
        Saves the view of the data using the appropriate backend for the
        filename's extension. Returns True if saving succeeded.
        """
        pass
```
It's the basics.
- I need to instantiate the chart from data,
- I need to store the styling options for the plot (`theme`),
- I need to draw the chart using some library (`backend`),
- and I need to save the chart.

Protocol in hand, I wrote the basic tests.
```python
def test_draw_signature(self):
        """
        GIVEN a chart instantiated with valid_chart_data
        WHEN we call the draw method with any valid keyword-argument
        THEN we receive the appropriate matplotlib.Figure or an altair.Chart
        """
        for backend, view in product(self.backends, self.charts_all):
            chart = view.from_chart_data(data=self.data)
            manipulable_object = chart.draw(backend=backend)
            self.assertIsInstance(manipulable_object, (ggplot, TopLevelMixin))

    def test_save_functionality(self):
        """
        GIVEN a chart instantiated with valid_chart_data
        WHEN we call the save method with any valid keyword-argument
        THEN the appropriate file will be saved to its appropriate location
        AND we will be returned a boolean indicating saving success
        """
        if not os.path.isdir(self.temp_dir):
            os.mkdir(self.temp_dir)  # Make a directory to hold the test plots
        filename = os.path.join(self.temp_dir, "test_filename")
        try:
            for ext, view in product(self.extensions, self.charts_all):
                chart = view.from_chart_data(data=self.data)
                full_path_current = filename + ext
                # Ensure missing file, create the file, ensure existing file
                self.assertFalse(os.path.exists(full_path_current))
                result = chart.save(full_path_current)
                self.assertIsInstance(result, bool)
                self.assertTrue(os.path.exists(full_path_current))
        finally:
            # Clear up test plots even if failure happens
            shutil.rmtree(self.temp_dir, ignore_errors=True)
```
I'm of course skipping details of these tests, but the basic points are the following.
`self.charts_all` is a list of `View`s.
Then, `chart = view.from_chart_data(data=self.data)` tests the constructor method.
It also tests whether the implementing class has a theme and the methods defined by `View`.
Next, one function tests `chart.draw` and another function tests `chart.save`.
This fully tests the protocol. Have a look at the following three files for all the details of the [protocol construction](https://github.com/timothyb0912/checkrs/blob/stable/src/checkrs/base.py), implementing [chart construction](https://github.com/timothyb0912/checkrs/blob/stable/src/checkrs/sim_cdf.py#L279), and [tests](https://github.com/timothyb0912/checkrs/blob/stable/tests/unit/test_chart_protocol.py).

To top it off, I set up Github Actions [workflow](https://github.com/timothyb0912/checkrs/blob/stable/.github/workflows/testing.yml) to perform the tests and display the results automatically upon pushing changes to the package repository.
Both the original tests and the continuous testing via Github increase the package's trustworthiness.
So, despite having to learn by fire along the way, it's a win in the end!

!["Losses to Learnings"](https://media2.giphy.com/media/3o6vY7UsuMPx3Yj9Xa/giphy.gif)
