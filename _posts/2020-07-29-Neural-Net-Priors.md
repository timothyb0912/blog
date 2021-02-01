---
toc: true
layout: post
title: "On Priors for Neural Networks"
description: "A review of two research papers pointing to the need for carefully chosen priors for one's neural networks."
categories: [reading]
comments: true
hide: false
---
# On Priors for Neural Networks

Dear Young Tim,

I hope this note reaches you long before you need it.
At some point, you will want to experiment with neural networks.
And when you do, you will want to learn both how they work and how you can make them work well.
Essentially, this means learning how to design (i) likelihoods, i.e. neural network architectures and activations, (ii) priors, i.e. regularization schemes, and (iii) training / inference procedures.

In this post, I offer you an opinionated introduction to constructing and using neural networks.
I come to you as a sort of statistical, Mardi Gras Indian, [Spy Boy](https://www.mardigrasneworleans.com/history/mardi-gras-indians/ranks).
Indeed, I have scouted ahead and brought back strange reports of success and despair.

I know our procession is a Bayesian one, a practical and practicing one at that.
Accordingly, like any thorough Bayesian model, this story begins with the prior.
Not with the main character, the likelihood, the neural network itself.
But instead with the background, the prior, the context through which we regularize and understand the entire drama.

In time, you will find that training often becomes easier with a good prior and that using the prior predictive distribution is a great way to understand neural architectures.
You will learn that the fastest way to create neural networks that generalize well is to use good priors during training, and you will spend a lot of time and computational resources figuring out how to do so.
This post aims to provide you with a framework for thinking about priors / regularization for neural networks.
In the end this framework should save you much time, computation, and confusion, but at the moment, let's begin with the basics.

### Helpful priors from Frequentists
The first report I have for you is that Frequentists who build neural networks care **a lot** about neural network priors.
Moreover, Frequentists are commonly successful at constructing priors that enable them to achieve impressive out-of-sample performance.

Yes, of course, you didn't expect to see such concerns from Frequentists... I know.
If it's any consolation, it's July 2020 for me.
By this point, after the coronavirus pandemic, after the global recession, after the continued racism... little will surprise you anymore.
But for now, soak it all in.

To start with, Frequentists will not speak of prior distributions when describing how they construct their neural networks.
Instead, they will equivalently speak of regularization methods.
However, have no fear.
As discussed below, most regularization methods have explicit counterparts in Bayesian priors.

For example, placing a mean zero Gaussian prior over one's weights (i.e. neural network parameters) leads to L2-weight decay (see Table 1, p.2821 [here](http://www.yaroslavvb.com/papers/chen-on.pdf)).
Similarly, placing a mean zero [Laplace prior](https://www.researchgate.net/profile/Peter_Williams19/publication/2719575_Bayesian_Regularisation_and_Pruning_using_a_Laplace_Prior/links/58fde123aca2728fa70f6aab/Bayesian-Regularisation-and-Pruning-using-a-Laplace-Prior.pdf) over one's weights leads to L1-weight decay.
Perhaps less familiarly, data augmentation is also a way encoding prior information (see for example, [here](http://cbcl.mit.edu/publications/ps/niyogi_poggio_girosi_1998.pdf) and [here](https://onlinelibrary.wiley.com/doi/abs/10.1002/sim.902)).
And so on, and so forth.
Stay tuned for future posts that provide a more comprehensive list of such comparisons.

For now, it suffices to say that much research on designing good regularization methods for deep learning is implicitly about designing good priors for one's neural networks.
And this brings us to today's first paper of interest.
`Do Deep Nets Really Need to be Deep?` by Ba and Carauna ([2014](https://arxiv.org/abs/1312.6184)).

Ba and Carauna's paper is one of the first to introduce the concept of knowledge/model distillation: taking the knowledge obtained from a given model, called the teacher, and passing that knowledge to a second model, called the student.
The teacher packages its knowledge using predictions (or intermediate quantities) on the dataset used to train the student model.
A regularization term is then used to penalize the student model for deviations between its predictions or intermediate quantities and those of the teacher.
As one might expect given the preceding discussion about regularization often equaling prior specification, placing a prior on the divergence of your neural network's predictions from those of a teacher's predictions leads to model distillation.
See [here](https://www.tandfonline.com/doi/abs/10.1080/01621459.1996.10476713) for historical examples of such priors on predictive functionals in statistics using "human expert teachers," and see [here](https://arxiv.org/pdf/1912.00874.pdf) for a modern bayesian perspective on model distillation.

The important findings from Ba and Carauna's paper are the following.
> "[M]odels trained on targets predicted by other models can be more accurate than models trained on the original labels" (p. 6)

   > "The mechanisms above can be seen as forms of regularization that help prevent overfitting in the student model" (p. 7)

   > "Although there is a consistent performance gap between the two models due to the difference in size, the smaller shallow model was eventually able to achieve a performance comparable to the larger shallow net by learning from a better teacher, and the accuracy of both models continues to increase as teacher accuracy increases. [...] We see little evidence that shallow models have limited capacity or representational power. Instead, the main limitation appears to be the learning and regularization procedures used to train the shallow models." (pp. 7-8)

Or, put differently, the provision of a good prior (in this paper, an accurate teacher model and choice of penalization function)
- can be even more important for neural network performance than providing access to the real data (see also [here](https://www.aaai.org/ocs/index.php/AAAI/AAAI17/paper/view/14967/14447)),
- helps prevent overfitting, and
- might be the primary missing factor needed to construct performant neural networks (at least for some network classes).

### Harmful priors from Bayesians
My second report is that Bayesians have been building neural networks with priors that do more harm than good.

One of the most recent clarion calls was Wenzel et al.'s `How Good is the Bayes Posterior in Deep Neural Networks Really?` ([2019](https://arxiv.org/abs/2002.02405)).
Wenzel et al.'s motivating observation was that, in the papers they considered and in their experience, tempering the posterior distribution often led to better predictive performance compared to using the actual posterior distribution.
The authors view the lackluster performance of the true posterior distribution as a problem because it suggests problems with the likelihood, prior, or training procedure.

To investigate the sources of the posterior distribution's dismal performance, Wenzel et al. perform a series of computational experiments.
Based on these trials, the authors assign high probability to the posterior distribution's underperformance being due to the prior distributions used with Bayesian neural networks, as opposed to the likelihood or training procedures.

In particular, the authors look at a standard, multivariate normal distribution prior (i.e., L2-weight decay).
First, they find that with an increasing number of parameters, the standard normal prior leads to unintentially informative prior predictive distributions.
They uncover prior predictive distributions that are much more extreme than desired.

Second, the authors realize that as they decrease the number of observations in their training set, the predictive performance of their posterior distribution degrades faster than the predictive performance of the maximum likelihood estimate (which implicitly assumes an improper uniform distribution).
Since the effect of the prior increases as one decreases the number of observations, this indicates that the standard normal prior is worse than the improper uniform prior in the settings they considered.
For yet another example of problems arising from using the standard normal prior with neural networks, see the "soap-bubble pathology" ([Farquhar et al., 2019](https://arxiv.org/pdf/1907.00865.pdf)).

These observations all stand as examples of "the [hidden dangers](https://statmodeling.stat.columbia.edu/2013/11/21/hidden-dangers-noninformative-priors/) of noninformative priors" or default priors.
Researchers often adopt such priors for computational or political ease, but they may have deleterious effects on one's posterior performance and inference.
In fact, one should perhaps expect such negative consequences given the typical, over-parameterized neural network context with a high ratio of parameters to one's number of observations.

### Conclusion
The papers `How Good is the Bayes Posterior in Deep Neural Networks Really?` and `Do Deep Nets Really Need to be Deep?` show that prior distributions are important to neural network construction.
These papers show that priors can have either helpful or harmful effects on one's predictive ability, relative to maximum likelihood estimation.
To give oneself the best chance of constructing neural networks that perform well, and to reduce the chance of having our predictive performance harmed unnecessarily, the takeaway is that we should carefully choose the priors for our models.
