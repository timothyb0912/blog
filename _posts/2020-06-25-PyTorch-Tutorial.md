---
title: "PyTorch Tutorial"
description: "A tutorial on how to use PyTorch for discrete choice modeling."
comments: true
hide: false
categories: [practice]
---
# PyTorch Tutorial
1. TOC
{:toc}

Dear Young Tim,

I know you remember the *good old days*: writing out probability equations, analytically deriving gradients and hessians of the log-likelihood, and then painstakingly implementing those derivatives in code.

Throughout your career, especially in school, you will forego research efforts because the time to analyze and implement the derivatives seemed greater than the gain from doing so.
> Gallery of asymmetric choice models?
No.  
Gradient-based MCMC for your decision tree + choice model paper?
No.  
Mixed Logit models with non-normal mixing distributions in PyLogit?
No.

Despite the good reasons for my decisions, loss aversion and questions of "what-if" will plague you.

Moreover, this behavioral pattern will persist even after you graduate from UC Berkeley.
Somehow, you will find yourself with less free time and more competing activities that you wish to fill that time with.
Every year you will say more "no's" as you ruthlessly triage new research ideas based, in part, on their projected execution time.
Your feeling of loss due to time constraints will increase.

Far from coming to engage in a pity party, I'm here to announce that there is hope for change.
You can stop this wound from bleeding.
Across projects, you can lower projected execution times and their variance.
Unshackle yourself from some feelings of constraint-based loss by eliminating time for gradient and hessian derivations.

A technique called automatic differentiation enables automatic gradient and hessian computation once you write the code to compute one's log-likelihood and probabilities.
PyTorch and Tensorflow began supporting sparse matrices in 2017 and 2016, respectively, making it possible to use these packages' automatic differentiation tools for discrete choice models.
In other words, discrete choice modellers could have been living the dream for three to four years!
No more passing up on research ideas because derivations take too long.
Write your log-likelihood and probability functions in PyTorch or Tensorflow, and you'll get the exact gradients and hessians automatically!

Of course, despite the enabling technology existing, uptake of these tools has been slow.
For example, I began using PyTorch in 2019, after failed attempts in 2018.
My biggest stumbling block was not having a comprehensive understanding of what I needed and how I would interface with other third-party packages (NumPy, SciPy, etc.).
How could I use these packages to do something I already knew how to do?
I found Tensorflow's learning curve to be terribly steep--so steep that I never came back to it!
PyTorch was much easier for me to begin with, so that's what this post is about.

I hope this post will hasten your adoption of automatic differentiation.
Below, I describe the high-level elements you'll need when replacing one's hand-coded derivatives with PyTorch computed derivatives.
Moreover, I introduce PyTorch in a non-trivial discrete choice setting, replicating the Mixed Logit "B" model of
> Brownstone, David, and Kenneth Train. "Forecasting new product penetration with flexible substitution patterns." Journal of econometrics 89, no. 1-2 (1998): 109-129.


To be clear, this is not merely a "toy" exercise for demonstration.
I am doing this work to revise my paper
> Brathwaite, Timothy. "Check yourself before you wreck yourself: Assessing discrete choice models through predictive simulations." arXiv preprint arXiv:1806.02307 (2018).


After reading this post, you should be able to begin estimating whatever other model you think of.

With this context in mind, let's get started!

# Overview
To keep matters simple, I'll describe how to use PyTorch to perform maximum likelihood estimation of one's discrete choice model.
The expectation is that, as in PyLogit, you'll use `scipy.optimize.minimize` for parameter optimization.
In this setting, the following five objects are sufficient for getting started.
1. A loss function written in PyTorch.
2. An input object containing all data needed to compute probabilities for the choice observations of interest.
3. A PyTorch model (i.e. a subclass of `torch.nn.Module`) that packages all model parameters for optimization and the probability computation logic.
4. Helper functions to get the gradient and hessian of the log-likelihood as Numpy arrays for use with `scipy.optimize.minimize`.
5. A closure for `scipy.optimize.minimize` that takes in a passed array of parameters and returns the loss and gradient of the loss.

Let's import the needed modules for this notebook and then look at some examples of the five objects listed above.

# Example


```python
# Py3 Built-in module for type-hint annotations
from typing import Dict, Tuple, List, Callable, Optional

# Third-party modules for:
# class definition
import attr
# Optimization
import scipy.optimize as opt
# Numerical computation
import torch
import torch.nn as nn
import numpy as np
# Utility functions for pytorch optimization
import botorch.optim.numpy_converter as numpy_converter
import botorch.optim.utils as optim_utils
from botorch.optim.numpy_converter import TorchAttr
# Input/Output of data
import pandas as pd
```

## Loss Function
The simplest of the objects listed in the overview is the loss function.
At its core, the loss function is a python function that returns a scalar based (even if only indirectly) on the parameters being optimized.

Here is an example of the loss-function I typically use when estimating discrete choice models: the log-loss, i.e. the negative of the log-likelihood.

If this is your first time using PyTorch, then one thing to recognize is that `torch.Tensor` is PyTorch's equivalent of Numpy's `ndarray`.
PyTorch Tensors and Numpy arrays share the same locations in your computer's memory, and converting between these two data containers is memory-efficient (i.e., without data-copying).


```python
def log_loss(probs: torch.Tensor,
             targets: torch.Tensor) -> torch.Tensor:
    """
    Computes the log-loss (i.e., the negative log-likelihood) for given
    long-format tensors of probabilities and choice indicators.

    Parameters
    ----------
    probs : 1D torch.Tensor.
        The probability of choosing each row's alternative for the given choice situation.
    targets : 1D torch.Tensor.
        A Tensor of zeros and ones indicating the chosen row for each choice situation. Should have the same size as `probs`.

    Returns
    -------
    neg_log_likelihood : scalar torch.Tensor.
        The negative log-likelihood computed from `probs` and `targets`.
    """
    log_likelihood = torch.sum(targets * torch.log(probs))
    return -1 * log_likelihood
```

## Input Object
Many interrelated objects are often needed to calculate discrete choice model probabilities.
For instance, PyLogit's multinomial logit models rely not just on a design matrix but on multiple sparse "mapping matrices" that relate rows of the design matrix to observation or alternative identifiers.
Another example is the mixed logit model.
It relies on a set of generated normal random variates for each observation.
Because these randomly drawn values are to remain constant during one's estimation process, they need to be passed into one's probability calculating function along with the design matrix that one typically thinks of when using a `predict` function.

The input object packages all these related objects needed to compute probabilities.
The official PyTorch method for packaging these objects would be to use the following classes:
- `torch.utils.data.DataLoader`
- `torch.utils.data.Dataset`
- `torch.utils.data.IterableDataset`
- `torch.utils.data.Sampler` (or subclasses)

However, these PyTorch objects seemed more complicated than necessary for estimating a discrete-choice model using batch-optimization methods (instead of mini-batch optimization techniques).
The input object below is, in my opinion, a simpler data class that still provides the needed packaging functionality.


```python
@attr.s
class InputMixlB:
    """
    Stores the observation-specific data needed to compute probabilities in the
    Mixed Logit B model.
    """
    # Needed attributes are:
    # design matrix
    design: torch.Tensor = attr.ib()
    # rows_to_choice_situation, i.e, rows_to_obs
    obs_mapping: torch.sparse.FloatTensor = attr.ib()
    # rows_to_decision_makers, i.e, rows_to_mixers
    mixing_mapping: torch.sparse.FloatTensor = attr.ib()
    # list of normal random variates
    normal_rvs: List[torch.Tensor] = attr.ib()

    @classmethod
    def from_df(cls,
                df: pd.DataFrame,
                mixing_seed: int=601,
                num_draws: int=250) -> 'InputMixlB':
        """
        Creates a class instance from a dataframe with the requisite data.

        Parameters
        ----------
        df : pandas DataFrame.
            Should be a long-format dataframe containing the following columns:
            `[alt_id, obs_id, choice]`.
        mixing_seed : optional, int.
            Denotes the random seed to use when generating the normal random
            variates for Monte Carlo integration in the maximum simulated
            likelihood procedure.
        num_draws : optional, int.
            Denotes the number of random draws to use for Monte Carlo
            integration in the maximum simulated likelihood procedure.

        Returns
        -------
        Instantiated 'InputMixlB' object.
        """
        ...
```

Let me explain what is going on here.

First, I am using the `attr.s` decorator to signal that I want to create instances of this class using the `attr` library.
This library has [multiple benefits](https://www.attrs.org/en/stable/why.html), but the most relevant ones in this example are that I can avoid writing an `__init__` function and I can access data using human readable property names.
Next, I use `attr.ib()` to declare the attributes of the class.
Using PyLogit terminology, I am storing the design matrix, the `rows_to_choice_situation` and `rows_to_decision_makers` mapping matrices, and the normal random variates for estimating the maximum simulated likelihood.

After declaring the class attributes, I define a class method to initialize an instance of the class.
This is denoted by the `@classmethod` decorator.
The method, although not displayed above for brevity, encapsulates the logic (all 56 lines of it) to take a given dataframe of data for the Mixed Logit Model B and create the class attributes described above.
The way to use it is by typing `mixlb_input = InputMixlB.from_df(df)` where `df` is as described in the `from_df` docstring.
See [here](https://github.com/timothyb0912/check-yourself/blob/develop/notebooks/miscellaneous/_20-tb-check-the-mixlb-model.py#L111) for an example of the `from_df` method in use and [here](https://github.com/timothyb0912/check-yourself/blob/develop/src/models/model_inputs.py#L40) for the full definition of the method.

The main point is that once you have an input object, you can pass the object or the object's attributes to the `forward` method of your PyTorch model to compute the needed probabilities.
Note that all the attributes are pytorch objects.
These are the only type of objects that one should pass to your model's `forward` method if one wants to use automatic differentiation on the resulting log-likelihood.


## PyTorch Model
The most obviously needed object for using PyTorch's automatic differentiation capabilities is a PyTorch model.
In particular, one's PyTorch model must be a subclass of `nn.Module` that overwrites / implements the forward method.
Since we're estimating discrete choice models, the forward method should take an input object or that object's attributes, and the forward method should output a 1D Tensor of predicted probabilities in "long-format."

Note that one's PyTorch model should do more than capture the logic needed to compute the discrete choice model's probabilities.
One's PyTorch model should also store any data needed to compute the probabilities that does not change across observations.

As an example, let's look at the skeleton of the PyTorch model used to compute the probabilities for Brownstone and Train's (1998) Mixed Logit B model.


```python
@attr.s
class DesignInfoMixlB:
    """Put generic information about the design matrix here."""
    ...


@attr.s(eq=False, repr=False)
class MIXLB(nn.Module):
    """"
    PyTorch implementation of `Mixed Logit B` in [1].

    References
    ----------
    [1] Brownstone, David, and Kenneth Train. "Forecasting new product
    penetration with flexible substitution patterns." Journal of
    econometrics 89.1-2 (1998): 109-129.
    """
    # Should store all needed information required to specifcy the
    # computational steps needed to calculate the probability function
    # corresponding to `Mixed Logit B`

    # Info denoting the design columns, the indices for normal and lognormally
    # distributed parameters, etc.
    design_info = attr.ib(init=False, default=DesignInfoMixlB())

    # Standard deviation constant for lognormally distributed values
    log_normal_std = attr.ib(init=False, default=torch.tensor(0.8326))

    ####
    # Needed constants for numerical stability
    ####
    # Minimum and maximum value that should be exponentiated
    min_exponent_val = attr.ib(init=False, default=torch.tensor(-700))
    max_exponent_val = attr.ib(init=False, default=torch.tensor(700))
    # MNL models and generalizations only have probability = 1
    # when the linear predictor = infinity
    max_prob_value = attr.ib(init=False, default=torch.tensor(1-1e-16))
    # MNL models and generalizations only have probability = 0
    # when the linear predictor = -infinity
    min_prob_value = attr.ib(init=False, default=torch.tensor(1e-40))

    def __attrs_post_init__(self):
        # Make sure that we call the constructor method of nn.Module to
        # initialize pytorch specific parameters, as they are not automatically
        # initialized by attrs
        super().__init__()

        # Needed paramters tensors for the module:
        #   - parameter "means" and "standard deviations"
        self.means =\
            nn.Parameter(torch.arange(len(self.design_info.column_names),
                                      dtype=torch.double))
        self.std_deviations =\
            nn.Parameter(torch.arange(len(self.design_info.normal_coef_names),
                                      dtype=torch.double))
        # Enforce parameter constraints
        self.constrain_means()

    def constrain_means(self):
        """
        Ensures that we don't compute the gradients for mean parameters that
        should be constrained to zero.
        """
        # Note that parameters 21 (non_ev) and 22 (non_cng) are constrained to
        # zero because those columns are for random effects with mean zero by
        # specification.
        self.constrained_means =\
            optim_utils.fix_features(self.means, {21: None, 22: None})

    def forward(self,
                design_2d: torch.Tensor,
                rows_to_obs: torch.sparse.FloatTensor,
                rows_to_mixers: torch.sparse.FloatTensor,
                normal_rvs_list: List[torch.Tensor]) -> torch.Tensor:
        """
        Compute the probabilities for `Mixed Logit B`.

        Parameters
        ----------
        design_2d : 2D torch.Tensor.
            Denotes the design matrix whose coefficients are to be computed.
        rows_to_obs : 2D torch.sparse.FloatTensor.
            Denotes the mapping between rows of `design_2d` and the
            choice observations the probabilities are being computed for.
        rows_to_mixers : 2D torch.sparse.FloatTensor.
            Denotes the mapping between rows of `design_2d` and the
            decision-makers the coefficients are randomly distributed over.
        normal_rvs_list : list of 2D torch.Tensor.
            Should have length `self.design_info.num_mixing_vars`. Each element
            of the list should be of shape `(rows_to_mixers.size()[1],
            num_draws)`. Each element should represent a draw from a standard
            normal distribution.

        Returns
        -------
        average_probabilities : 1D torch.Tensor
            Denotes the average of the probabilities for each alternative for
            each choice situation, across all random draws of the model
            coefficients.
        """
        # Get the coefficient tensor for all observations
        # This will be a 3D tensor
        coefficients =\
            self.create_coef_tensor(design_2d, rows_to_mixers, normal_rvs_list)
        # Compute the long-format systematic utilities per row and random draw.
        # This will be a 2D tensor
        systematic_utilities =\
            self._calc_systematic_utilities(design_2d, coefficients)
        # Compute the long-format probabilities for each row and random draw.
        # This will be a 2D tensor
        probabilities_per_draw =\
            self._calc_probs_per_draw(systematic_utilities, rows_to_obs)
        # Compute the long-format, average probabilities across draws.
        # This will be a 1D tensor.
        average_probabilities = torch.mean(probabilities_per_draw, 1)
        return average_probabilities

    def create_coef_tensor(
            self,
            design_2d: torch.Tensor,
            rows_to_mixers: torch.sparse.FloatTensor,
            normal_rvs_list: List[torch.Tensor]
        ) -> torch.Tensor:
        ...

    def _calc_systematic_utilities(self,
                                   design_2d: torch.Tensor,
                                   coefs: torch.Tensor) -> torch.Tensor:
        ...

    def _calc_probs_per_draw(
            self,
            sys_utilities: torch.Tensor,
            rows_to_obs: torch.sparse.FloatTensor
        ) -> torch.Tensor:
        ...



```

The model is the most complex object described in this post so let's spend some time unpacking what is going on above.

The big picture is as follows.
The code snippet above subclasses `nn.Module` and implements the `forward` method of the model to compute the choice probabilities.
These are the important steps shown above.
Everything else is an implementation detail.
Now, I'll proceed chronologically from the top of the cell and explain more of these specifics.

The first subtle action is how I invoke the `attr.s` decorator.
Unlike the case of the `InputMixlB` object, I now pass the decorator arguments `eq=False, repr=False`.
Both arguments are necessary for PyTorch and the `attr` module to operate nicely together.
The `eq=False` argument allows hashing of the `nn.Module` and thereby permits use of the module's internal C++ functions.
The `repr=False` argument tells `attr` not to overwrite the default string representation given by PyTorch's `nn.Module` class.
For more information about why these workarounds are necessary, see [here](https://stackoverflow.com/questions/57291307/pytorch-module-with-attrs-cannot-get-parameter-list).

Next, I define two model specific attributes: `design_info` and `log_normal_std`.
The `design_info` attribute contains:
- attributes denoting the column names of the design matrix,
- the column indices for normal and log-normally distributed parameters, and
- mappings relating the design columns to the lognormal or normally distributed coefficient names.

The `log_normal_std` attribute is a constant denoting the standard deviation of the normal random variates used in the log-normal mixing distributions of the Mixed Logit B model.
The common feature of these two attributes is that they both represent model-specific information that does not change across observations.
I therefore stored them on the model object itself.

After these, the next four attributes are minimum and maximum values to be exponentiated and treated as probabilities, respectively.
These attributes are not model-specific, as I tend to use them across all my discrete choice models to avoid underflow and overflow.
However, since these values do not change across observations, I also stored them on the model object.

Note, while defining these attributes, I passed `init=False` and `default=something`.
The `init=False` keyword argument tells `attr` that these attributes do not need to be passed when initializing an instance of the model class.
The `default=something` keyword argument tells `attr` that if these attributes are not passed as arguments to the `__init__` function, then use `something` as the value of the attribute instead (whatever `something` is).

Now, with the model-specific attributes stored as attributes, the PyTorch-specific attributes become the center of attention.
To set these attributes, I use the `__attrs_post_init__` function, which is called after executing the `__init__` function of the `MIXLB` object.
This `__attrs_post_init__` function, as defined above, will then call the `__init__` function of `nn.Module` (via `super().__init__()`) to perform PyTorch specific initialization actions.
Then, I assign the `nn.Parameter` attributes `means` and `std_deviations`.
The Mixed Logit B model optimizes these parameters during estimation.
Finally, I use the PyTorch enhancing `BoTorch.optim.utils` module to enforce the model-specific parameter constraints using the call `self.constrain_means`.
I emphasize this step to point out that arbitrary parameter constraints can be imposed on one's model.

At long last, we come to the `forward` method.
As noted before, this method encapsulates the logic needed to compute the probabilities for Mixed Logit B.
Importantly, this logic is not all written in one monolithic `forward` method.
Because the model relies on a set of intricate sub-steps, especially `create_coef_tensor`, these substeps are defined as their own methods on the `MIXLB` class.
These methods ease the reading of the `forward` method and clarify its logic.
Note that for the sake of brevity, the sub-step methods are not displayed above.
See [here](https://github.com/timothyb0912/check-yourself/blob/develop/src/models/mixlb.py) for the complete definition of the sub-step methods and of the class in general.

To be completely honest, it's not only the sub-step methods that were hidden from display above.
To avoid displaying a long cell, I also hid the "helper" functions that enable painless interfacing between Numpy, Scipy, and PyTorch.
Let's look at these helper functions below.

## Helper functions
PyTorch and Numpy/Scipy interact in four main situations:
- setting a Numpy array of parameters on a PyTorch model
- getting a Numpy array of parameters from a PyTorch model
- getting a Numpy array of parameter gradients from a PyTorch model
- getting a Numpy array of the hessian of one's PyTorch model parameters.

The helper functions below address the first three of these situations.


```python
@attr.s(eq=False, repr=False)
class MIXLB(nn.Module):
    """"
    PyTorch implementation of `Mixed Logit B` in [1].

    References
    ----------
    [1] Brownstone, David, and Kenneth Train. "Forecasting new product
    penetration with flexible substitution patterns." Journal of
    econometrics 89.1-2 (1998): 109-129.
    """
    ...

    def get_params_numpy(self) -> Tuple[
            np.ndarray, Dict[str, TorchAttr], Optional[np.ndarray]]:
        """
        Syntatic sugar for `botorch.optim.numpy_converter.module_to_array`.

        Returns
        -------
        param_array : 1D np.ndarray
            Model parameters values.
        param_dict : dict.
            String representations of parameter names are keys, and the values
            are TorchAttr objects containing shape, dtype, and device
            information about the correpsonding pytorch tensors.
        bounds : optional, np.ndarray or None.
            If at least one parameter has bounds, then these are returned as a
            2D ndarray representing the bounds for each paramaeter. Otherwise
            None.
        """
        return numpy_converter.module_to_array(self)

    def set_params_numpy(self, new_param_array: torch.Tensor) -> None:
        """
        Sets the model's parameters using the values in `new_param_array`.

        Parameters
        ----------
        new_param_array : 1D ndarray.
            Should have one element for each element of the tensors in
            `self.parameters`.

        Returns
        -------
        None.
        """
        # Get the property dictionary for this module
        _, property_dict, _ = self.get_params_numpy()
        # Set the parameters
        numpy_converter.set_params_with_array(
            self, new_param_array, property_dict)
        # Constrain the parameters
        self.constrain_means()

    def get_grad_numpy(self) -> None:
        """
        Returns the gradient of the model parameters as a 1D numpy array.
        """
        grad =\
            np.concatenate(list(x.grad.data.numpy().ravel()
                                for x in self.parameters()),
                           axis=0)
        return grad
```

Luckily, each of the methods above are simple.
`get_params_numpy` and `set_params_numpy` provide accessors to BoTorch's pre-existing functions, each requiring no more than three lines of code.
The `get_grad_numpy` function uses natively available PyTorch attributes and methods in a single assignment statement.

The fourth situation, computing the hessian, did not appear to be accessible solely via the model object after calculation of the loss.
It seemed that no matter what, I would need to pass both the model (or its attributes) as well as the loss to a function in order to compute the hessian.
I could be wrong about this as I have not investigated fully!
Nevertheless, given my impressions from limited information so far, it seemed inappropriate to create a method on the model object that computed the hessian.
I left this task to be explicitly performed by the user outside of a model method.

Now, with all the objects and methods described so far, we can finally optimize our model's parameters according to our chosen loss function.

Let's create one last object below, the optimization closure, to help us perform precisely this optimization.

## Optimization closure

The purpose of creating an optimization closure is to create an objective function that is compatible with `scipy.optimize.minimize`.
Scipy [describes](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.minimize.html) the needed optimization function with the following statements.
First, Scipy specifies that the optimization function should take a 1D array of parameters as inputs and return a floating point scalar as output.
> fun ; callable  

>The objective function to be minimized.  
`fun(x, *args) -> float`  
where x is an 1-D array with shape (n,) and args is a tuple of the fixed parameters needed to completely specify the function.

Next, we extract a second set of requirements for our optimization function by considering the fact that we wish to perform gradient-based optimization.
To use gradient-based optimization, we have to provide the gradient, i.e., the *jacobian* for 1D objective function outputs.
It is important to note that Scipy allows the gradient to be provided in one of two ways.
One way is that we can provide a callable, separate from the optimization function `fun`, that will compute the gradient.
The second way is that we provide the gradient as an output of the optimization function.

This requirement is described as follows.  
>jac ; {callable, ‘2-point’, ‘3-point’, ‘cs’, bool}, optional  

>[...] If jac is a Boolean and is True, fun is assumed to return and objective and gradient as and (f, g) tuple.

For computational efficiency, it makes sense to avoid repeating oneself.
In particular, since many operations in computing the gradient are  performed when computing the loss, we should avoid computing the loss twice.
Instead, we will compute the loss once (to be returned as `float` from `fun`) and we will immediately compute the gradient for return.
This is shown in the closure below.


```python
def make_scipy_closure(
        input_obj: InputMixlB,
        targets: torch.Tensor,
        model: MIXLB,
        loss_func: Callable,
        ) -> Callable:
    """
    Creates the optimization function for use with scipy.optimize.minimize.

    Parameters
    ----------
    input_obj : InputMixlB.
        Container of the inputs for the model's probability function.
    targets : 1D torch.Tensor
        A Tensor of zeros and ones indicating which row was chosen for each
        choice situation. Should have the same size as
        `(input_obj.design.size()[0],)`.
    model : MIXLB.
        Should have a forward object that computes the probabilities of
        the given discrete choice model.
    loss_func : callable.
        Should take as inputs, `model` outputs and `targets`. Should return
        the value of the loss as well as the gradient of the loss.

    Returns
    -------
    optimization_func : callable
        Takes a set of parameters as a 1D numpy array and returns the
        corresponding loss function value and gradient corresponding to the
        passed parameters.
    """
    def closure(params):
        # params -> loss, grad
        # Load the parameters onto the model
        model.set_params_numpy(params)
        # Ensure the gradients are summed starting from zero
        model.zero_grad()
        # Calculate the probabilities
        probabilities =\
            model(design_2d=input_obj.design,
                  rows_to_obs=input_obj.obs_mapping,
                  rows_to_mixers=input_obj.mixing_mapping,
                  normal_rvs_list=input_obj.normal_rvs)
        # Calculate the loss
        loss = loss_func(probabilities, targets)
        # Compute the gradient.
        loss.backward()
        # Get the gradient.
        grad = model.get_grad_numpy()
        # Get a float version of the loss for scipy.
        loss_val = loss.item()
        return loss_val, grad
    return closure
```

Of course, in your own projects, you'll want to replace the `input_obj`, `model`, and `loss_func` with the custom objects you've created.
Examples of creating these objects are provided above!

# Conclusions: Putting it all together

Alright, the five objects above provide all one needs for optimizing the parameters of your model.
Within the `scipy.optimize.minimize` function, you'll pass the output of `make_scipy_closure` as `fun` and you'll pass `jac=True`.
That should be all!

The resulting optimization routine should roughly look as follows:

```python
# Load the data for the model.
df = pd.read_csv(DATA_PATH)

# Instantiate the model inputs.
input_mixlb = InputMixlB.from_df(df)

# Instantiate the model.
mixlb_model = MIXLB()

# Create pytorch targets with 32-bit precision.
choice_tensor =\
    torch.from_numpy(df[CHOICE_COLUMN].values.astype(np.float32))

# Choose initial values for the optimization.
initial_values =\
    np.zeros(mixlb_model.get_params_numpy().size, dtype=float)

# Create the optimization function for scipy
scipy_objective =\
    make_scipy_closure(input_obj=input_mixlb,
                       targets=choice_tensor,
                       model=mixlb_model,
                       loss_func=log_loss)

# Perform the parameter optimization.
# Use your method of choice.
optimization_results =\
    opt.minimize(scipy_objective,
                 initial_values,
                 jac=True,
                 method='bfgs')
```

To see such an approach in action, check out [this file](https://github.com/timothyb0912/check-yourself/blob/develop/notebooks/miscellaneous/_19-tb-estimate-mixlb-pytorch.py) in the "Check Yourself Before You Wreck Yourself" Github repository.
In that file, you can also see how the hessian is computed (relevant lines [here](https://github.com/timothyb0912/check-yourself/blob/develop/notebooks/miscellaneous/_19-tb-estimate-mixlb-pytorch.py#L259)). It's pretty simple, so I've excluded it from this already long post.

Now, go forth and estimate models with abandon!
