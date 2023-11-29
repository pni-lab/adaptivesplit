---
title: "External validation of machine learning models: registered models and adaptive sample splitting"
subject: Preprint
short_title: AdaptiveSplit Preprint

exports:
  - format: pdf
    template: springer
    output: exports/adaptivesplit_manuscript.pdf    
  - format: docx
    output: exports/adaptivesplit_manuscript.docx

abbreviations:
  cv: cross-validation

#bibliography:
#  - bibliography.bib   
---

+++ {"part": "key-points"}
**Key Points:**
- todo
+++

+++ {"part": "abstract"}
Predictive modeling with biomedical data is a key approach to improve the understanding of complex biological systems and to develop new diagnostic tools. However, even when a proper internal validation strategy is applied, the use of complex machine learning approaches and extensive data pre-processing and feature engineering pipelines can result in overfitting and poor generalizability. External validation (i.e. test model inference with fixed model parameters on independent data) is a critical step to establish the quality of predictive models, but it is often neglected due to the high cost and time required for the acquisition of additional. 
Here we propose that prospective external validation should be considered an essential, final step in the model discovery procedure and the end of training and beginning of validation should be separated by the public disclosure or pre-registration of feature processing steps and model weights. Furthermore, we propose a novel approach to optimize the trade-off between efforts spent on training and external validation in such studies. We show in four different datasets that the proposed adaptive splitting approach can successfully identify the optimal time to stop acquiring data for training. As compared to fixed splitting strategies, with AdaptiveSplit the predictive performance achievable with any "sample size budget" can be maximized without risking a low powered, and thus inconclusive, external validation. 
The proposed design and splitting approach (implemented in the Python package "AdaptiveSplit") may contribute to addressing issues of replicability, effect size inflation and generalizability in predictive modeling studies.
+++

## Introduction
Multivariate predictive models integrate information across multiple variables to construct predictions of a specific outcome and hold promise for delivering more accurate estimates than traditional univariate methods ([](https://doi.org/10.1038/nn.4478)). For instance, in case of predicting individual behavioral and psychometric characteristics from brain data, such models can provide higher statistical power and better replicability, as compared to conventional mass-univariate analyses ([](https://doi.org/10.1038/s41586-023-05745-x)). Predictive models can be powered by many different algorithms, from simple linear regression-based models to deep neural networks. With increasing model complexity, the model will be more prone to overfit its training dataset, resulting in biased, overly optimistic in-sample estimates of predictive performance and often decreased generalizability to data not seen during model fit ([](https://doi.org/10.1016/j.neubiorev.2020.09.036)). Internal validation approaches, like cross-validation (cv) provide means for an unbiased evaluation of predictive performance during model discovery by repeatedly holding out parts of the discovery dataset for testing purposes ([](https://doi.org/10.1201/9780429246593); [](doi:10.1001/jamapsychiatry.2019.3671)). 

However, internal validation approaches, in practice, still tend to yield overly optimistic performance estimates ([](https://doi.org/10.1080/01621459.1983.10477973); [](https://doi.org/10.1016/j.biopsych.2020.02.016); [](https://doi.org/10.1038/s41746-022-00592-y)). There are several reasons for this kind of effect sie inflation. First predictive modelling approaches typically display a high level of "analytical flexibility" and pose a large number of possible methodological choices in terms of feature preprocessing and model architecture, which emerge as uncontrolled (e.g. not cross-validated) "hyperparameters" during model discovery. Seemingly 'innocent' adjustments of such parameters can also lead to overfitting, if it happens outside of the cv loop. The second reason for inflated internally validated performance estimates is 'leakage' of information from the test dataset to the training dataset ([](https://doi.org/10.1016/j.patter.2023.100804)). Information leakage has many faces. It can be a consequence of, for instance, feature standardization in a non cv-compliant way or, in medical imaging, the co-registration of brain data to a study-specific template. Therefore, it is often very hard to notice, especially in complex workflows
Another reason for overly optimistic internal validation results may be that even the highest quality discovery datasets can only yield an imperfect representation of the real world. Therefore, predictive models might capitalize on associations that are specific to the dataset at hand and simply fail to generalize "out-of-the-distribution", e.g. to different populations. Finally, some models might also be overly sensitive to unimportant characteristics of the training data, like subtle differences between batches of data acquisition or center-effects ([](https://doi.org/10.1038/s42256-020-0197-y); [](https://doi.org/10.1093/gigascience/giac082)).

The obvious solution for these problems is *external validation*; that is, to evaluate the model's predictive performance on independent ('external') data that is guaranteed to be unseen during the the whole model discovery procedure. There is a clear agreement in the community that external validation is critical for establishing machine learning model quality ([](https://doi.org/10.1186/1471-2288-14-40); [](https://doi.org/10.1016/j.patter.2020.100129); [](https://doi.org/10.1148/ryai.210064); [](https://doi.org/10.1038/s41586-023-05745-x); [](doi:10.1001/jamapsychiatry.2019.3671)). Acquiring external validation data prospectively, after the final model has already been published, provides the highest possible guarantees for the results' reliability (e.g. [](https://doi.org/10.1038/s41467-019-13785-z); [](10.31219/osf.io/utkbv)). However, the amount of data to be used for model discovery and external validation can have crucial implications on the predictive power, replicability and validity of predictive models and is, therefore, subject of intense discussion ([](https://doi.org/10.1002/sim.8766); [](https://doi.org/10.1002/sim.9025); [](https://doi.org/10.1038/s41586-022-04492-9); [](https://doi.org/10.1038/s41586-023-05745-x); [](https://doi.org/10.1038/s41593-022-011109); [](https://doi.org/10.1038/s41593-022-01110-9); [](10.52294/51f2e656-d4da-457e-851e-139131a68f14); [](https://doi.org/10.1101/2023.06.16.545340); [](#supplementary-table-1)). Finding the optimal sample sizes is especially challenging for biomedical research, where this trade-off needs to consider both ethical and economic reasons. As a consequence, to date only around 10\% of predictive modeling studies include an external validation of the model ([](https://doi.org/10.1093/jamia/ocac002)). Those few studies performing true external validation often perform it on retrospective data (like [](https://doi.org/10.1038/s41591-020-1142-7) or [](10.31219/osf.io/utkbv)) or ins separate, prospective studies ([](https://doi.org/10.1038/s41467-019-13785-z); [](10.31219/osf.io/utkbv)). Both approaches can result in a suboptimal use of data and may slow down the dissemination process of new results.

In this manuscript we emphasize that maximal reliability and transparency during external validation can be achieved with prospective data acquisition preceded by "freezing" and publicly depositing (e.g. pre-registering) the whole feature processing workflow and all model weights. Furthermore, we present a novel adaptive design for predictive modeling studies with prospective data acquisition. With the proposed approach, the sample sizes for training and external validation do not have to be decided at the beginning of the study. Instead, they are dynamically determined based on the model's learning curve and the remaining samples' ability to confirm the model's external validity (statistical power). When the model meets predefined conditions during the ongoing data acquisition process, the training procedure concludes, and the model's weights are finalized and publicly deposited, e.g pre-registered. The subsequent data acquisition then serves as an independent external validation sample. The adaptive splitting procedure guarantees that this validation sample provides the necessary power to confirm the validity of the model.

## Background

#### The anatomy of a prospective predictive modelling study

Let us consider the following scenario: a research group plans to involve a fixed number of participants in a study with the aim of constructing a predictive model, and at the same time, evaluate its external validity. How many participants should they allocate for model discovery and how many for external validation to get the highest performing model as well as conclusive validation results?

In most cases it is very hard to make an educated guess about the optimal split of the total sample size into discovery and external validation samples prior to data acquisition. A possible approach is to use simplistic rules-of-thumb. Splitting data with a 80-20\% ratio (a.k.a Pareto-split, [](https://doi.org/10.1080/00207390802213609)) is probably the most common method, but a 90-10\% or a  50-50\% may also be plausible choices (Rykar \& Saha, 2015, Steyerberg, 2001). However, as illustrated on {numref}`fig1`, such prefixed sample sizes are likely sub-optimal in many cases and the optimal strategy is actually determined by the dependence of the model performance on training sample size, that is, the "learning curve". For instance, in case of a significant but generally low model performance ({numref}`fig1`A: flat learning curve) the model does not benefit a lot from adding more data to the training set but, on the other hand, it may require a larger external validation set for conclusive evaluation, due to the lower predictive effect size. This is visualized by the "power curve" on {numref}`fig1`, which shows the statistical power of external validation with the remaining sample as a function of sample size used for model discovery. The optimal strategy will be different, however, if the learning curve shows a persistent increase, without a strong saturation effect, meaning that predictive performance can be significantly enhanced by training the model on larger samples ({numref}`fig1`B).
In this case, the stronger predictive performance that can be achieved with larger training sample size, at the same time, allows a smaller external validation sample to be still conclusive.
Finally, in some situations, model performance may rapidly get strong and reach a plateau at a relatively low sample size ({numref}`fig1`C). In such cases, the optimal strategy might be to stop early with the discovery phase and allocate resources for a more powerful external validation. 

:::{figure} figures/fig1.png
:name: fig1
**Examples of different optimal discovery and external validation sample sizes compared to a predefined 80-20\% Pareto-split.** \
**(A)** If the planned sample size and the model performance is low, the predefined external validation sample size might provide low statistical power to detect a significant model performance. **(B)** External validation of highly accurate models is well-powered; increasing the training sample size (against the external validation sample size) might result in a better performing final model. **(C)** Continuing training on the plateau of the learning curve will result in a negligible or biologically not relevant model performance improvement. 
In this case, a larger external validation sample (for more robust external performance estimates) or ‘early stopping’ of the data acquisition process might be desirable.
:::

#### Transparent reporting of external validation: registered models
One of the main criteria of external validation is that the external data has to be independent of the data used during model discovery ([](https://doi.org/10.1016/j.jclinepi.2015.04.005)]; [](10.1186/1471-2288-14-40); [](10.1038/s41586-023-05745-x)). Regardless of the splitting strategy, an externally validated predictive modelling study must provide strong guarantees for this independence criterion. 
In case of prospective studies, this can be realized by means of pre-registration ([](10.1038/s41586-023-05745-x)]; [](https://doi.org/10.1016/j.tics.2019.07.009)) ({numref}`fig2`A), including the plans for both model discovery and external validation. 

However, as the concept of preregistration was originally developed for confirmatory research, it does not fit well to exploratory nature of the model discovery phase in typical predictive modelling endeavors. Specifically, while preregistration necessitates that as many parameters of the analysis as possible are fixed before data acquisition, predictive modelling studies often involve a large number of hyperparameters (e.g. model architecture, feature preprocessing steps, regularization parameters, etc.) that are not known in advance and need to be optimized during the model discovery phase. This is especially true for complex machine learning models, like deep neural networks, where the number of hyperparameters can easily reach hundreds or thousands. In such cases, the preregistration of the discovery phase would be either impossible or would require a large number of assumptions and simplifications, which would make the preregistration less informative and transparent.
And alternative approach is to perform the pre-registration phase after model discovery but before the external validation {numref}`fig2`B) (as done e.g. in [](10.1038/s41467-019-13785-z); [](10.31219/osf.io/utkbv)). In this case, more freedom is granted for the discovery phase, while the external validation remains equally conclusive, as long as the pre-registration of the external validation includes all details of the *finalized* model (including the feature pre-processing workflow). This can easily be done by attaching the data and the reproducible analysis code used during the discovery phase or, alternatively, a serialized version of the fitted model (i.e. a file that contains all model weights). We refer to such models as **registered models**.

:::{figure} figures/fig2.png
:name: fig2
**The registered model design and the proposed adaptive sample splitting procedure for prospective predictive modeling studies.** \
 (A) Predictive modelling combined with conventional preregistration. In this case the pre-registration precedes data acquisition and requires that as many details of the analysis are fixed as possible. Given the potentially large number of coefficients to be optimized and the importance of hyperparameter optimization, conventional preregistration exhibits a limited compatibility with predictive modelling studies. (B) Here we propose that in case of predictive modelling studies, public registration should only happen after the model is trained and finalized. The registration step in this case includes publicly depositing the finalized model, with all its parameters as well as all feature preprocessing steps. External validation is performed with the resulting *registered model*. This practice ensures a transparent, clear separation of model discovery and external validation. (C) The "registered model" design allows a flexible, adaptive splitting of the "sample size budget" into discovery and external validation phases. The proposed adaptive sample splitting procedure starts with fixing (and potentially pre-registering) a stopping rules (R1). During the training phase, one or more candidate models are trained and the splitting rule is repeatedly evaluated as the data acquisition proceeds. When the splitting rule "activates", the model gets finalized (e.g. by being fit on the whole training sample) and publicly deposited/registered (R2). Finally, data acquisition continues and the prospective external validation is performed on the newly acquired data.
:::

#### The adaptive splitting design

Here, we introduce a novel design for prospective predictive modeling studies that leverages the flexibility of model discovery with registered models. Our aim is to determine an optimal splitting strategy, adaptively, during data acquisition. This strategy balances the model performance and the statistical power of the external validation ({numref}`fig2`C). The proposed design involves continuous model fitting and tuning throughout the discovery phase, for example, after every 10 new participants, and evaluating a 'stopping rule' to determine if the desired compromise between model performance and statistical power of the external validation has been achieved. This marks the end of the discovery phase and the start of the external validation phase, as well as the point at which the model must be publicly and transparently reported/pre-registered. Importantly, the pre-registration should precede the continuation of data acquisition, i.e., the start of the external validation phase.
In the present work, we propose and evaluate a concrete, customizable implementation for the splitting rule. 

## Methods and Implementation

#### Components of the stopping rule


The stopping rule of the proposed adaptive splitting design (for short "AdaptiveSplit") can be formalized as function $S$:

:::{math}
S_\Phi(\mathbf{X}_{act}, \mathbf{y}_{act}, \mathcal{M}) \quad \quad S: \mathbb{R}^2 \longrightarrow \{True, False\}
:::

where $\Phi$ denotes parameters of the rule (detailed in the next paragraph), $\mathbf{X}_{act} \in \mathbb{R}^2$ and $\mathbf{y}_{act} \in \mathbb{R}$ is the data (a matrix consisting of $n_{act} > 0$ observations and an fixed number of features $p$) and prediction target, respectively, as acquired so far and $\mathcal{M}$ is the machine learning model to be trained. The discovery phase ends if and only if the stopping rule returns $True$.

##### **Hard sample size thresholds**

Our stopping rule is designed so that it can force a minimum size for both the discovery and the external validation samples, $t_{min}$ and $v_{min}$, both being free parameters of the stopping rule.

Specifically:

:::{math}
:label: eq-min-rule
    \text{Min-rule:} \quad n_{act} \geq t_{min}
:::

:::{math}
:label: eq-max-rule
    \text{Max-rule:} \quad n_{act} \geq n_{total} – v_{min}
:::

where $n_{act}$ and $n_{total}$ are the actual sample size (e.g. participants measured so far) and the total sample sizes (i.e. the sample size budget), respectively, so that $n_{total} >= n_{act} > 0$.
Setting $t_{min}$ and $v_{min}$ may be useful to prevent early stopping at the beginning of the training procedure, where predictive performance and validation power estimates are not yet reliable due to the small $n_{act}$ or to ensure that a minimal validation sample size, even if stopping criteria are never met. If $t_{min}$ and $v_{min}$ are set so that $t_{min} + v_{min} = n_{total}$ then our approach falls back to training a registered model with predefined training and validation sample sizes.

##### **Forecasting Predictive Performance via Learning Curve Analysis**

Taking internally validated  performance estimates of the candidate model as a function of training sample size, also known as learning curve analysis, is  a widely used approach to gain deeper insights into model training dynamics (see examples on {numref}`fig1`). In the proposed stopping rule, we will rely on learning curve analysis to provide estimates of the current predictive performance and the expected gain when adding new data to the discovery sample. 

Performance estimates can be unreliable or noisy in many cases, for instance with low sample sizes or when using leave-one-out cross-validation ([](https://doi.org/10.1016/j.neuroimage.2017.06.061)). To obtain stable and reliable learning curves, we propose to calculate multiple cross-validated performance estimates from ssub-samples sampled without replacement from the actual data set. The proposed procedure is detailed in {numref}`alg-learning-curve`.

:::{prf:algorithm} Bootstrapped Learning Curve Analysis
:label: alg-learning-curve

1. **Require** $\mathbf{X}_{act}, \mathbf{y}_{act}, \mathcal{M}$
2. **Set** $n_b \gets \texttt{<number of bootstrap iterations>}$
3. **For** $t \gets 1$ to $n_{act}$   *(loop over sample sizes)*

    4. **For** $i \gets 1$ to $n_b$ *(bootstrap iterations)*

        5. **Set** $\mathbf{b} \gets$ sample $t$ indices from $<1, \dots, n_{act}>$ without replacement
        6. **Set** $\mathbf{X}_b \gets \mathbf{X}_{act}[\mathbf{b}]$
        7. **Set** $\mathbf{y}_b \gets \mathbf{y}_{act}[\mathbf{b}]$
        8. **Set** $\mathbf{s}[i] \gets$ cross-validated performance score of $\mathcal{M}$ fitted to $(\mathbf{y}_b, \mathbf{X}_b)$

    9. **End For**
    10. **Set** $\textbf{l}_{act}[t] \gets median(\mathbf{s})$

11. **End For**
12. **Return** $\textbf{l}_{act}$  *(bootstrapped learning curve)*
:::

The learning curve analysis allows the discovery phase to be stopped if the expected gain in predictive performance is lower than a predefined relevance threshold and can be used for instance for stopping model training earlier in well-powered experiments and retain more data for the external validation phase. Specifically, the stopping rule $S$ will return $True$ if the *Min-rule* ([Eq. %s](#eq-min-rule)) is $True$ or the following is true:

:::{math}
:label: eq-perf-rule
    \text{Performance-rule:} \quad \hat{s}_{total} - s_{act} \leq s_{min}
:::

where $s_{act}$ is the actual bootstrapped predictive performance score (i.e. the last element of $\textbf{l}_{act}$, as returned by {numref}`alg-learning-curve`, $\hat{s}_{total}$ is a estimate of the (unknown) predictive performance $s_{total}$ (i.e. the predictive performance of the model trained the whole sample) and $\epsilon_{s}$ is the smallest predictive effect of interest. Note that, setting $\epsilon_{s} = -\infty$ deactivates the *Performance-rule* ([Eq. %s](#eq-perf-rule)). 

While $s_{total}$ is typically unknown at the time of evaluating the stopping rule $S$, there are various approaches of obtaining an estimate $\hat{s}_{total}$. In the base implementation of AdaptiveSplit, we stick to a simplistic method: we extrapolate the learning curve $l_{act}$ based on its tangent line at $n_{act}$, i.e. assuming that the latest growth rate will remain constant for the remaining samples. While in most scenarios this is an overly optimistic estimatate, it still provides a useful upper bound for the maximally achievable predictive performance with the given sample size and can successfully detect if the learning curve has already reached a flat plateau (like on {numref}`fig1`C).

##### **Statistical power of the external validation sample**

Even if the learning curve did not reach a plateau, we still need to make sure that we stop the training phase early enough to save a sufficient amount of data for a successful external validation. Given the actual predictive performance estimate $s_{act}$ and the size of the remaining, to-be-acquired sample $s_{total} - s{act}$, we can estimate the probability that the external validation correctly rejects the null hypothesis (i.e. zero predictive performance). 
This type of analysis, known as power calculation, allows us to determine the optimal stopping point that guarantees the desired statistical power during the external validation.
Specifically, the stopping rule $S$ will return $True$ if the *Performance-rule* ([Eq. %s](#eq-perf-rule)) is $False$ and the following is true:

:::{math}
:label: eq-pow
    \text{Power-rule:} \quad POW_\alpha(s_{act}, n_{val}) \leq v_{pow}
:::

where $POW_\alpha(s, n)$ is the power of a validation sample of size $n$ to detect an effect size of $s$ and $n_{val} = n_{total}-n_{act}$ is the size of the validation sample if stopping, i.e. the number of remaining (not yet measured) participants in the experiment.
Given that machine learning model predictions are often non-normally distributed ([](https://doi.org/10.1093/gigascience/giac082)), our implementation is based on a bootstrapped power analysis for permutation tests [refs], as shown in {numref}`alg-power-rule`. Our implementation is, however, simple to extend with other parametric or non-parametric estimators like Pearson Correlation and Spearman Rank Correlation.

:::{prf:algorithm} Calculation of the Power-rule
:label: alg-power-rule

1. **Require** $\mathbf{X}_{act}, \mathbf{y}_{act}, n_{validation},  \mathcal{M}, \alpha$
2. **Set** $n_b \gets \texttt{<number of bootstrap iterations>}$
3. **Set** $n_{\pi} \gets \texttt{<number of permutations>}$
4. **Set** $\mathbf{\hat{y}}_{act} \gets \text{cross-validated prediction from } \mathbf{X}_{act} \text{with} \mathcal{M}$

    5. **For** $i \gets 1$ to $n_b$

        6. **Set** $\mathbf{b} \gets$ sample $t$ indices from $<1, \dots, n_{val}>$ with replacement
        7. **Set** $\mathbf{y}_b \gets \mathbf{y}_{act}[\mathbf{b}]$
        8. **Set** $\mathbf{\hat{y}}_b \gets \mathbf{\hat{y}}_{act}[\mathbf{b}]$
        9. **Set** $r_{obs} = correlation( \mathbf{y}_b, \mathbf{\hat{y}}_b)$

        10. **For** $j \gets 1$ to $n_p$

            11. **Set** $\pmb{\pi} \gets$ permute$(<1, \dots, n_{val}>)$
            12. **Set** $\mathbf{y}_{\pi} \gets \mathbf{y}_{b}[\pmb{\pi}]$
            13. **Set** $\mathbf{\hat{y}}_{\pi} \gets \mathbf{\hat{y}}_{b}[\pmb{\pi}]$
            14. **Set** $\mathbf{r}_{null}[j] = correlation( \mathbf{y}_{\pi}, \mathbf{\hat{y}}_{\pi})$

        15. **End For**
        16. **Set** $\mathbf{p}[i] \gets \# (\mathbf{r}_{null} > r_{obs}) / n_{perm}$    

    17. **End For**

18. **Set** $power = \# (\mathbf{p} < \alpha) / n_{b}$
19. **Return** $power$ 
:::

Note that depending on the aim of external validation, the *Power-rule* can be swapped to, or extended with, other conditions. For instance, if we are interested in accurately estimating the predictive effect size, we could condition the stopping rule on the width of the confidence interval for the prediction performance. 

Calculating the validation power ({numref}`alg-power-rule`) for all available sample sizes ($n = 1 \dots n_{act}$) defines the so-called "validation power curve" (see {numref}`fig1`), that represents the expected ratio of true positive statistical tests on increasing sample size calculated on the external validation set. Various extrapolations of the power curve can predict the expected stopping point during the course of the experiment.

#### Stopping Rule

Our proposed stopping rule integrates the $\text{Min-rule}$, the $\text{Min-rule}$, the $\text{Peerformance-rule}$ and the $\text{Power-rule}$ in the following way:

\begin{equation}
    \begin{split}
     S_\Phi(\mathbf{X}_{act}, \mathbf{y}_{act}, \mathcal{M}) = \quad & \\ 
    &\text{Min-rule} \quad AND \\
    & ( \\
    & \quad \text{Max-rule} \quad OR \\
    & \quad\text{Performance-rule} \quad OR \\
    & \quad \text{Power-rule} \\
    &)
    \end{split}
\end{equation}

where $\Phi = <t_{min}, v_{min}, s_{min}, v_{pow}, \alpha>$ are parameters of the stopping rule: minimum training sample size, minimum validation sample size, minimum effect of interest and target power for the external validation and the significance threshold, respectively.

We have implemented the proposed stopping rule in the Python package “*adaptivesplit*[^adaptive-split]. The software can be used together with a wide variety of machine learning tools and provides an easy-to-use interface to work with scikit-learn ([](https://doi.org/10.48550/arXiv.1201.0490)) models.

#### Empirical evaluation

We evaluate the proposed stopping rule, as implemented in the package *adaptivesplit* in four publicly available datasets; the Autism Brain Imaging Data Exchange (ABIDE) [](https://doi.org/10.1038/mp.2013.78), the Human Connectome Project ([](https://doi.org/10.1016/j.neuroimage.2013.05.041)), the Information eXtraction from Images (IXI)[^ixi] and the Breast Cancer Wisconsin (BCW) [refs] datasets (Fig. 3). 

##### **ABIDE**
We obtained preprocessed data from Autism Brain Imaging Data Exchange (ABIDE) dataset [](https://doi.org/10.1038/mp.2013.78) involving 866 participants (Autism Spectrum Disorder: 402, neurotypical control: 464). Preprocessed regional time-series data were obtained as shared (https://osf.io/hc4md) by [](https://doi.org/10.1016/j.neuroimage.2019.02.062), which were based on preprocessed image data provided by the Preprocessed Connectome Project [](10.3389/conf.fninf.2013.09.00041).

Tangent correlation across the time series of the n= 122 regions of the BASC (Multi-level bootstrap analysis of stable clusters) [65] brain parcellation was computed with nilearn (http://nilearn.github.io/) [66,67].

The resulting functional connectivity estimates were considered features for a predictive model of autism diagnosis. The model was trained with a L2-regularized logistic regression model (scikit-learn, https://scikit-learn.org/stable/) [68] with a nested cross-validation procedure for hyperparameter optimization (see {numref}`alg-learning-curve` and {numref}`alg-power-rule`).

##### **HCP**
The Human Connectome Project dataset contains imaging and behavioral data of approximately 1,200 healthy subjects [55]. Preprocessed resting state functional magnetic resonance imaging (fMRI) connectivity data (partial correlation matrices) [56] as published with the HCP1200 release (N= 999 participants with functional connectivity data) were used to build models that predict individual fluid intelligence scores (Gf), measured with Penn Progressive Matrices [57].


##### **IXI**
The IXI dataset is published by the Neuroimage Analysis Center, from Imperial College London, in the United Kingdom, and it is part of the project Brain Development. It consists of approximately 600 structural MRI images from a diverse population of healthy individuals, including both males and females across a wide age range. The dataset contains high-resolution brain images and associated demographic information, making it suitable for studying age-related changes in brain structure and function. We used gray matter probability maps generated from T1–weighted MR images with Freesurfe ([](https://doi.org/10.1016/j.neuroimage.2012.01.021)). The gray matter probability maps were used as features for a predictive model of age. 

##### **BCW**
The Breast Cancer Wisconsin (BCW, [](10.1117/12.148698)) dataset contains diagnostic features computed from a digitized image of a fine needle aspirate (FNA) of a breast mass. The dataset includes 30 different features such as the mean radius, mean texture, mean perimeter, mean area, mean smoothness, mean compactness, mean concavity, etc. The dataset also includes the diagnosis (M = malignant, B = benign) as the target variable. 


The chosen datasets include both classification and regression tasks, and span a wide range in terms the number of participants, number of predictive features, achievable predictive effect size and data homogeneity.
Our analyses aim to contrast the proposed adaptive splitting method to the application of fixed training and validation sample sizes, specifically to using 50, 60 (a.k.a Pareto-split) or 90\% of the total sample size for training and the rest for external validation.
We simulate various "sample size budgets" (total sample sizes, $n_{total}$) with random sampling without replacement. 
For a given total sample size, we simulate the prospective data acquisition procedure by incrementing $n_{act}$; starting with 10\% of the total sample size and going up with increments of five. In each step, the stopping rule was evaluated with "AdaptiveSplit", but fitting a Ridge model (for regression tasks) or a L2-regularized logistic regression (for classification tasks). Model fit always consisted of a cross-validated fine-tuning of the regularization parameter (list values), resulting in bootstrapping a nested cross-validation-based estimate of prediction performance and validation power (see {numref}`alg-learning-curve` and {numref}`alg-power-rule`).
This procedure was iterated until the stopping rule returned True. The corresponding sample size was then considered thee final training sample.
With all four splits approaches (adaptive, Pareto, Half-split, 90-10\% split), we trained the previously described Ridge or regularized logistic regression model  on the training sample and obtained predictions for the sample left out for external validation. 
This whole procedure is repeated 100 times for each simulated sample size budget in each dataset, to estimate the confidence intervals for the models performance in the external validation and its statistical significance.
In all analyzes, the adaptive splitting procedure is performed with a target power of $v_{pow} = 0.8$, an $alpha = 0.05$,  $t_{tmin} = n_{total}/3$, $v_{min}=12$, $s_{min}=-\infty$.  P-values were calculated using a permutation test with 5000 permutations. 

## Results

The results of our empirical analyses of four large, openly available datasets confirmed that the proposed adaptive splitting approach can successfully identify the optimal time to stop acquiring data for training and maintain a good compromise between maximizing both predictive performance and external validation power with any "sample size budget".

In all three samples, the applied models yielded a statistically significant predictive performance at much lower sample sizes than the total size of the dataset, i.e. all datasets were well powered for the analysis. The models explained X\% of the variance in cognitive abilities based on functional brain connectivity in the HCP dataset and Y\% in age, based on structural MRI data (gray matter probability maps) in the IXI dataset. Classification accuracy was X for autism diagnosis (functional connectivity) in the ABIDE dataset and Y for breast cancer diagnosis in the BCW dataset (type of data).

The datasets varied not only in the achievable predictive performance but also in the shape of the learning curve, with different sample sizes (Fig) and thus, they provided a good opportunity to evaluate the performance of our stopping rule in various circumstances.

We found that adaptively splitting the data provided external validation performances that were comparable to the commonly used Pareto split (80-20\%) in most cases ({numref}`fig3` left column). As expected half-split tended to provide worse predictive performance due to the smaller training sample . In contrast, 90-10% tended to display only slightly higher performances than the Pareto and the Adaptive splitting techniques.
This small achievement came with a big cost in terms of the statistical power in the external validation sample, where the 90-10% split very often gave inconclusive results ($p\geq0.05$) (Fig. {numref}`fig3`right column).
Although to a lesser degree, Pareto split also frequently failed to yield a conclusive external validation. Adaptive splitting (as well as half-split) provided sufficient statistical power for the external validation in most cases.

In sum, when contrasting splitting approaches based on fixed validation size with adaptive splitting, using the latter was always the preferable strategy to maximize power and statistical significance during external validation. The benefit of adaptively splitting the data acquisition for training and validation provides the largest benefit in lower sample size regimes.
In case of larger sample sizes, the fixed Pareto split (20-80%) provided also good results, giving similar external validation performances to adaptive splitting.

:::{figure} figures/fig3.png
:name: fig3
Comparison of splitting methods on external validation performance (left column) and p-values (right column) at various $n_{total}$. We see how the AdaptiveSplit's overall performance is comparable to Pareto's on each of the three neuroimaging datasets, albeit adaptively splitting data works better than pareto at low sample sizes, where external validation performance can be slightly lower but statistical significance is always higher. We can also observe how AdaptiveSplit outperforms 50-50 and 90-10 splits, which have low external validation scores and statistical significance, respectively.
:::

## Discussion

Adaptively splitting data allows to keep the external validation performance "in check" by determining the best trade-off between model performance and statistical validity, allowing the generation of reliable results even with  smaller sample sizes. This feature makes our design a strong choice for developing dependable and repeatable models for prospective data acquisition and model registration purposes. Model finalization is facilitated by adaptively setting training and external validation sample sizes, which saves time for acquisition and publication while simultaneously increasing model dependability when the it is appropriately registered. Including aspects that assure a good final model, such as data splitting, cross-validation, and bootstrapping for robust performance estimation, the adaptive split design makes it simple to avoid general pitfalls in predictive modeling, e.g. overfitting, data leakage or lack of reproducibility. Indeed, as a result of its bootstrap-based learning and power curves estimates on which the stopping rule is evaluated, the adaptive split method enables results' reproducibility on the same data and seems to give comparable results on different datasets when the same set of parameters is used (see Fig. 3).

-----

Issues with BWAS replication and cross-validation failure are addressed by our adaptive splitting design. By determining the optimal time to stop data acquisition for training, we can ensure that the model is not overfitted to the training data and can generalize well to unseen data. This is particularly important in biomedical research where the cost of data acquisition can be high and the number of available samples is often limited.

In the context of predictive model discovery, the adaptive splitting design can be beneficial in several ways. Firstly, it allows for the registration of models at an early stage of the study, enhancing transparency and repeatability. Secondly, it provides a flexible approach to data splitting, which can be adjusted according to the specific needs of the study. For instance, if the external validation sample size is a concern, the adaptive splitting design can be adjusted to ensure a sufficient size of the validation sample.

When comparing the adaptive splitting design with more commonly used splitting methods such as the Pareto split, we find that the adaptive approach provides a more balanced trade-off between model performance and statistical validity. While the Pareto split often results in overfitting due to a large training sample size, the adaptive splitting design can prevent this issue by dynamically adjusting the training and validation sample sizes.

In conclusion, the adaptive splitting design provides a robust and flexible approach to data splitting in predictive modeling studies. It addresses several common issues in the field, including overfitting, cross-validation failure, and the challenge of model registration. By implementing this design in our Python package "AdaptiveSplit", we hope to contribute to the advancement of predictive modeling research and to promote the adoption of registered models in the field.





## Outlook

The adaptive splitting design, as implemented in our Python package "AdaptiveSplit", represents a significant step forward in the field of predictive modeling. It addresses several common issues, including overfitting, cross-validation failure, and the challenge of model registration, thereby enhancing the robustness and reliability of predictive models. 

The flexibility of the adaptive splitting design allows for a variety of stopping rules to be implemented, catering to the specific needs of different research questions. This flexibility, combined with the ability to register models at an early stage of the study, greatly enhances the transparency and repeatability of predictive modeling studies. 

As we continue to refine and expand the capabilities of "AdaptiveSplit", we anticipate that it will play a pivotal role in advancing the field of predictive modeling. We look forward to seeing how researchers will utilize this tool to develop more robust and reliable predictive models, and to the new insights that these models will provide.

-----



## Conclusion
Model registration for predictive modeling studies is an essential opportunity to improve the credibility and reproducibility of scientific works based on predictive modeling, since evidences by recent studies show how predictive models are often misused and/or misspecified (Kapoor \& Narayanan, 2021, other refs?). We presented the adaptive split design, a methodology allowing the examination of models' performance during prospective data acquisition in order to fix and finalize model for registration purposes. When we compare the adaptive training/external validation splitting strategy to more commonly used splitting methods, we find that flexibility in the definition of training / external validation sample size allows to deliver a model that is able to find the best trade-off between predictive performance and statistical significance even when a large sample size isn't available, making the definition of an early model possible while data collection is still ongoing and so enhancing transparency and study repeatability. Model registration for predictive modeling-based study design may boost trust in model creation and performance in several disciplines of research, particularly those that include the prediction of complex outcomes as, for instance, biomedical science.

[^adaptive-split]: https://github.com/pni-lab/adaptivesplit
[^ixi]: https://brain-development.org/ixi-dataset/