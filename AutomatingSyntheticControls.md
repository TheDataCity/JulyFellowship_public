# Synthetic Control Methods (SCM)
Synthetic control methods, first developed in [Abadie and Gardeazabal, 2003](10.1257/000282803321455188) are an analysis tool used to evaluate the effect of an 'event' on the outcomes of a company or region (a 'unit'). Evaluation often relies on having a control unit, which does not experience the event, for comparision of outcomes between the affected region and the control.

The synthetic control method creates this control out of other existing companies to try and isolate the effect of the event. In the first instance, we do this by creating a weighted average of other units which has outcomes very close to the effected unit before intervention. This weighting is then used at times after the intervention as a prediction of how the effected unit would have performed without the event occuring.

## Some maths
Formulating this mathematically, consider a region $N$ which has a reported outcome such as gdp, $y(t)$ 
at times $t=0,1,2,...,T_0,T_0+1,...,T_1$. At $t=T_0$, the region experiences an event, perhaps a policy change or increased tax relief in comparison to other regions in its country. 

The scalar values $y^n_t$ (the gdp of region $n$ at time $t$) can be written as a vector $\bf{y^n}$ of shape $T \times 1$.

In order to create the synthetic control method we truncate the vector with $T=T_0$ including all observations $t=0,1,...,T_0-1$, before the intervention occurs.
```math
$$ \mathbf{y^n}= \left(\begin{array}{c} y^n_0 \\ y^n_1 \\ .\\ . \\ y^n_T \end{array}\right)$$
```
The vector exists for $n=0,1,...N$ (N+1) region (units) with components $y^n_t$ for each of the companies at each time. We'll assign company $N$ as the unit to be modelled, and thus $n=0,1,2,...N-1$ are the ($N$) donor units.

The synthetic control method proposes that we can create a sum

$$ \mathbf{y^N}=\sum_{n=0}^{N-1} w_n y_n + \varepsilon$$

which describes the effected unit in terms of the donor units and some error $\varepsilon$ which is a $T\times 1$ vector with components $\varepsilon_t$.

Assembling the donor units into a matrix of shape $T_0 \times N$, which we call $\bf{X}$, allows us to simplify this sum.

$$ \bf{X}= \left(\begin{array}{c} y^0 &  y^1 & . & . & y^{N-1} \end{array} \right) $$

And now the model reads

$$ y^N=\bf{X}\bf{w} + \varepsilon$$

where $\bf w$ is an $N \times 1$ vector with components $w_n$, the weightings of each donor unit. 
```math
$$\mathbf{w}= \left(\begin{array}{c} w_0 \\ w_1 \\ . \\ . \\ w_{N-1} \end{array}\right)$$
```
The weights sum to one and be non-negative in order to be a weighted average of the donor units. 

$$\sum_{n=0}^{N-1}w_n=1, \quad \rm{and} \quad w_n\geq 0 \quad \forall \; n.$$

**Negative weights and a sum greater than one imply extrapolation beyond the known data set.**
For sufficiently many $N$ ($N\gg T$) this can hold with $\varepsilon=\bf 0$, but in a typical data set, the error is non-zero.

The optimal weights $w^n$ are found by linear regression, minimising the total error $\varepsilon$ in the prediction, which is defined by the quadratic programming statement

$$\min\mathbf{(y^N-Xw).T V(y^N-Xw)}, \qquad = \min\sum_0^{t=T_0-1}v_{t}\cdot\varepsilon_t^2$$

where $V$ is a diagonal $T\times T$ matrix which weights the relative contribution of the error in each point $y^n_t$. For classical linear regression, $V=I_T$ the $T\times T$ identity matrix.
Solving this problem gives us the closest match to the treated unit in question up until the time before the intervention $t=T_0-1$, as a weighted average of the donor units.

The weights calculated can then be used to forecast the evolution of the synthetic company after the intervention,
```math
$$\mathbf{\tilde{y}^{N}_{\tau<t<T}} = \mathbf{X_{T_0<t\leq T_1}}\cdot \mathbf{w}.$$
```
where $X_{T_0<t\leq T_1}$ is the gdp of the donor regions after the intervention in the effected region,
and we can now write the effect of the treatment as the difference between the observed gdp and the predicted uneffected gdp.
```math
$$\mathrm{effect} =\mathbf{y^N_{T_{0}<t\leq T_{1}}} -\mathbf{\tilde{y}^N_{T_{0}<t\leq T_{1}}} .$$
```
## Challenges of synthetic control methods
At first glance, the synthetic control method seems well defined but questions arise around a few key steps. How confident can we be that the synthetic control accurately predicts the counterfactual outcome?

- How do you choose the donor units to be a good representation of the effected region before the event? Having too large a donor set risks creating a synthetic control that gives an excellent result (low $\varepsilon$) mathematically but which has little or no economic meaning. This is similar to [spurious correlation generator](https://www.tylervigen.com/spurious-correlations) outputs. This can be avoided by having enough data of high enough accuracy and precision in the pre-control period, but this is unlikely with real economic data. To avoid this problem, the collection of donor units is often manually chosen by the economist studying them with prior knowledge of their industries and populations.

- How do you avoid over or under fitting in the linear regression that could effect the synthetic company when it's forecast forwards? This is a fairly big question that arises in many SCM works and there are a few ways to begin to deal with it. Unexpected values in the pre-intervention data could result in one donor unit being over-represented to try and fit that specific value, at the expense of the overall fit.

### Prediction error and other metrics
To assess these questions amongst others, we start with the the pre-treatment (fitted region) and post-treatment error between the treated unit and its synthetic counterpart, using the *Root Mean Squared Prediction Error*

$$ RMSPE=\sqrt{\sum_{t=t_0}^{T}\left(\mathbf{y^N_t}-\sum_{j=0}^{N-1}\mathbf{w_j}\mathbf{y^j_t}\right)^2}$$

The RMSPE is a measure of how much the synthetic control deviates from the observed unit over a set of predicted times. The pre-treatment error is between $t_0=0$ & $T=T_0-1$, and the post-treatment error is between $t_0=T_0$ & $T=T_1$. [Abadie, 2021](https://pubs.aeaweb.org/doi/pdfplus/10.1257/jel.20191450) and others provide some ways of using these to answer some of our questions.

### Is the donor set a good set?
[Springford](https://www.cer.eu/sites/default/files/pbrief_costofbrexit_8.6.22_0.pdf) approach this question by randomly dropping countries from their initial set of 22 countries used to produce a doppelganger UK out of 5, 10, 15, 20 countries and also then dropping the country $j$ which contributes the highest $w_j$. The resulting weightings were then averaged to produce a composite set of weightings used for the analysed doppelganger UK.

A formalised approach to choosing the donor set by [Abadie et al, 2010](https://www.tandfonline.com/doi/abs/10.1198/jasa.2009.ap08746) was to run placebo tests on each unit in the donor set and record the value of the RMSPE for each unit. If the unit scores an $RMSPE(j)\geq n\cdot RMSPE_N$ (the treated unit), for a chosen threshold $n$, then that unit $j$ is dropped from the set of donor units. The synthetic control is then recalculated on the new reduced set of donor units.

### Choosing the $V$ weighting matrix
The weighting matrix $V$ is a diagonal matrix that weights the relative contribution to the loss function of each time period in the training period.

1. The identity matrix, weights the outcome at each time equally and tries to fit them all equally.
 
   $$V=I_{T_0 \times T_0}$$
 
2. The inverse of the variance of the outcomes of the $N$ countries at time $t$. This effectively rescales the time period to have a variance of $1$ across the countries.

   $$V(t,t)=\frac{(N-1)}{\sum_{n=0}^{N-1} (y^n_t-\bar y_t)^2 }$$

3. A data driven approach to selecting $V$ using 'out-of-sample' RMSPE. This approach requires the partitioning of the $0<t<T_0$ region into a training and validation set. A bilevel optimisation is done, heuristically: the weights are computed on the training set for multiple options $V$, then the RMSPE for the validation set is calculated. $V$ is chosen such that the set of weights give the smallest RMSPE in the validation set, rather than in the training set. This approach requires enough temporal data to have a training and a validation set, and enough knowledge of which pre-treatment outcomes should be in each.

# The devolution of Scotland
In 1999, Scotland gained [devolved powers](https://www.deliveringforscotland.gov.uk/scotland-in-the-uk/devolution/) within the United Kingdom. As part of this, the Scottish parliament has the ability to decide some of their own taxation, their own health & social care spending and policies, amongst others.

We are interested in whether devolution increased or reduced the economic growth of Scotland.

We use ONS data for the growth of gdp of regions of the United Kingdom, found [here](https://www.escoe.ac.uk/research/historical-data/regional-data/), to assess whether devolution has resulted in improved economic performance for Scotland. The data is provided from 1979 Q2 through to 2020. 
![image](DevolutionofScotland.png 'Comparison of gdp in scotland and synthetic scotland')
We start with the simplest SCM,

- the weights $w$ must sum to one.
- be non-negative.
- $V=I_{T\times T}$. (we treat all years equally when minimising $\varepsilon$).

The result is plotted in the figure below, and this synthetic scotland is composed of the North West (59%), South West (15%) and  Northern Ireland (26%). At the first glance, we see that Scotland looks like it has outperformed its synthetic counterpart but can we be sure that this is significant? Now we must attempt to quantify the significance.

## Robustness checking & statistical significance
We began in this section by creating a synthetic counterpart for each region of the UK as a weighted average of all the other ones, as we did with Scotland. Next, we calculate the Root Mean Square Errors in each case for the fitted period (pre-1999).

| Region                  | Fitted RMSPE        |
|-------------------------|--------------------|
| North East              | 0.00699            | 
| Yorkshire and The Humber| 0.01172            | 
| East Midlands           | 0.02007            |
| East of England         | 0.01385            | 
| London                  | 0.00844            |
| South East              | 0.00849            |
| South West              | 0.11269            | 
| West Midlands           | 0.06162            |
| North West              | 0.00519            |
| Wales                   | 0.02197            |
| Scotland                | 0.01873            |
| Northern Ireland        | 0.03201            |

This shows that using our selected collection of donors (the other regions of the UK) we can create synthetic controls of all other regions. Of these the synthetic control for North West England most closely matches its measured performance and the synthetic control for South West England least well matches its measured performance. If we had a usefully larger donor set, we would expect these RMSPE measures to be smaller for all regions.

The fitted RMSPE help us to assess the suitability of this control set for modelling Scotland. If we are able to build each region out of the other regions with an appropriately small RMSPE, this suggests that the control set are similar enough to model each others behaviour with no interventions. RMSPE(scotland)=$0.019$ and all of the RMSPE reported are $<0.189$ (using a threshold of 10 RMSP(scotland)) suggests that they are sufficiently similar to make a good control group. The threshold of 10 RMSPE(scotland) is chosen arbitrarily and this is somewhat of a challenge for automation - where should the threshold be? A threshold of 5 would remove the South West from the analysis for example.

### Quantifying the statistical significance of devolution

Following [Abadie and Gardeazabal, 2003](10.1257/000282803321455188).

Now we have the apparent economic effect of devolution, and we know that our donor set is appropriately chosen. We next want to understand whether the deviation of the trajectory of real scotland froom the synthetic counterpart can be attributed to the treatment or to other circumstances which affected any number of the regions we used to build the synthetic region.

We have already calculated the RMSPE errors in the fitted time period, including Scotland in the donor group. The RMSPE errors can be used to evaluate the statistical significance of Scotland's deviation from the control.

The premise of this is that we assess the likelihood that the outcome would have happened without the intervention of devolution (the null hypothesis), by way of a p-value. The null hypothesis can then be rejected or accepted at a threshold $p$ level.

1. The RRMSPE of region $n$ is written as the ratio of the mean squared error of the forecasted period to that of the fitted period. The RRMSPE quantifies how much the region deviates from its counterfactual in the post-treatment period, relative to the pre-treatment period.
   
$$ RRMSPE_n = \frac{\sqrt{\sum_{\tau+1}^T (y^n(t)-\tilde y^n(t))^2}}{\sqrt{\sum_{0}^{\tau} (y^n(t)-\tilde y^n(t))^2}}.$$

2. We calculate the fitted and prediction errors for each region, and the ratio of these RMSPE errors.

| Region                  | Fitted RMSPE         | Predicted RMSPE         | RRMSPE            |
|-------------------------|--------------------|--------------------|------------------------|
| North East              | 0.00699   | 0.07914            | 11.32     |
| Yorkshire and The Humber| 0.01172   | 0.05412            | 4.62      |
| East Midlands           | 0.02007   | 0.13023            | 6.49      |
| East of England         | 0.01385   | 0.17605            | 12.71                  |
| London                  | 0.00844   | 0.41118            | 48.69                  |
| South East              | 0.00849   | 0.36180            | 42.61                  |
| South West              | 0.11269   | 0.24622            | 2.18                   |
| West Midlands           | 0.06162   | 0.15027            | 2.44                   |
| North West              | 0.00519   | 0.06722            | 12.94                  |
| Wales                   | 0.02197   | 0.03136            | 1.43                   |
| Scotland                | 0.01873   | 0.08092            | 4.32                   |
| Northern Ireland        | 0.03201   | 0.09031            | 2.82                   |

3. For the treated region of Scotland, we can now generate a p-value based upon the relative size of the $RRMSPE$ for each region.

$$ p_N=\frac{\sum_0^{N-1}{A(RRMSPE_n,RRMSPE_N)}}{N} $$

where
```math
$$A = \left\{\begin{array}{cc} 1 & RRMSPE_n> RRMSPE_N \\ 0 & RRMSPE_n< RRMSPE_N \\ \end{array}\right.$$
```
The p-value attempts to quantify the likelihood that the observed economic growth would have happened without the treatment at $t=T_0$.
Computing this from the RRMSPE values in the above table returns a p-value of $0.64$ to 2sf. The meaning of $p=0.64$ is that there is a 64% chance that the observed economic growth would have occured without the intervention of devolution. 

# Extensions of SCM
Since the original development of SCM, there have been additions to the method in order to capture more features of the region, or company, modelled as well as relaxations of assumptions in order to extend the method beyond its original scope. 

Following [Ferman & Pinto, 2021](https://doi.org/10.3982/QE1596) it's not uncommon for modelling to use *demeaned* outcomes which reduce the difference in scale to allow for matching trends.
In this setup, the mean value of each column (company) (in $\bf y$, and in $\bf X$) is subtracted from the columns values before the regression is done. Relaxing the requirement that the weights sum to one (form a weighted average) allows us model a unit which is larger or smaller than the donor set.


## Including more than one outcome
Sometimes, it is helpful to aim to match more than one outcome over time because this helps to make the most similar unit to the effected unit. For example, in the case of Scotland and the regions of the UK we might wish to ensure that our composite company has similar employment rate or similar level of schooling (measured as % of population over 25 with secondary school qualifications). The motivation for this is to ensure that the outcome is being driven by similar underlying characteristics in the regions.

Assuming that there are $m$ outcomes each observed at $T_0$ times, there are a few ways that we can think about doing this:

1. Perform SCM on each of the $m=0,1,...,M-1$ outcomes in turn, averaging the resultant weightings $w_{j,m}$ across the outcomes for $\bar{w}_j$.
2. Perform SCM on a concatenated vector of length $mT\times 1$ for each region, resulting in a single weighting vector that includes all of the outcomes. This is simple to implement but one disadvantage of this is that the minimisation will likely favour matching outcomes of greater magnitude. This could be accounted for via normalisation and via the $V$ matrix. Concatenation can also be used where the additional information is not time series data.
3. SCM with an unconstrained intercept term which accounts for the systematic difference in scale between the different outcome variables. 
4. Perform SCM on an averaged vector of length $T\times 1$ where each entry is the average of all the $m$ outcomes at the given time. Needs to have a value for each outcome at each time, concatenation is more flexible. This likely also needs some consideration of normalisation. Without normalisation, the vector of averaged outcomes is given by

$\tilde y^n_t =\frac{1}{M}\sum_{m=0}^{M-1}y^n_{t,m}$.

It is not immediately clear which of these will suit a problem at hand and some packages available tend to use a combination of approaches then further assess which gives the most robust result in that scenario.

# Synthetic companies
To generate synthetic companies for analysis we need to think about:
- choosing a good control group (and validating it)
- which information about the control and the studied company matters most
- hyperparameter exploration i.e. $V$ and whether extrapolation away from $\sum w=1$ might be acceptable (further to that, demeaned outcomes are also an option for multiple outcomes).
- how do we deal with missing or incomplete data. SVD seems to lend itself to filtering out noisy data.

## Disaggregate data
In the development of synthetic companies, there are many companies with information that we could use to fit the outcomes of the treated company. In this scenario, there is a risk of interpolation bias where wildly differing companies are averaged to create the synthetic control.
This is where the penalised synthetic control estimator [Abadie](https://economics.mit.edu/sites/default/files/publications/A%20Penalized%20Synthetic%20Control%20Estimator%20for%20Disagg.pdf) can be used to control the trade off between fitting well the treated unit by using many untreated units and using untreated units that have predictor values close to the treated unit. 

The quadratic programme reads as follows

$$\min\mathbf{(y^N-Xw).T V(y^N-Xw)}+\lambda \bf (y^N J_{1,N}-X).T (y^NJ_{1,N}-X) w \cdot J_{1,N}.T $$

where $J_{1,N}$ is a vector of ones of shape $1\times N$.
The parameter $\lambda$ controls the balance within the solution of interpolation and extrapolation biases.
$\lambda\to 0$ recovers classic synthetic control, $\lambda\to \infty$ is a nearest neighbour match. For each value of $\lambda$ there ought to be a unique solution (see Abadie, convex hulls and Delaunay geometry, I believe it but I can't prove it).

## Matching and synthetic control estimator
https://www.nber.org/system/files/working_papers/w26624/w26624.pdf

This is effectively a generalisation of the penalised method but using two different weighting vectors. Generates the matching estimator and synthetic control estimater and then computes a weighted average of those two. That weighting is validated using out of sample validation. It's also in principle possible to average penalised sc and matching.

### Using SVD/POD/PCA for disaggregate data.
TBC for a large volume of individual level data, SVD seems to be standard for reducing/filtering noise. This seems to be used for 'Robust Synthetic Control' but I haven't explored beyond this.

It's the same theory as POD in FD, if we filter the values first and set a threshold in the unitary matrix then we remove noisy/idiosyncratic values from the outcomes and that means we're not fitting idiosyncracies.







