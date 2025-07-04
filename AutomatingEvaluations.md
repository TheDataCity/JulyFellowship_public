# Automating Evaluations.

Economists love talking about evaluations. They especially love talking about how we need more and better evaluations of interventions designed to strengthen the economy in some way. This is often, conveniently, felt to require more money to pay economists.

There is less talk about how we might make evaluations more efficient, and yet the scope for that is large and growing.

Today, good evaluations of economic interventions in the UK tend to require lots of closed data, often securely held by the UK government, and accessed with large restrictions only after a long delay. Economies of scale are lost and reproducibility is limited. Common behaviours of high performing teams such as agile working, sharing work early, gathering feedback, and iterating quickly, are impossible when the timescales for safe data release are measured in months not minutes.

We think that there is a better way. Through three examples, we'll set out why.

## 1. MediaCity: The economic impact of the BBC's move to Salford Quays.

In 2011 The BBC started moving some of its staff from London to Salford Quays in Greater Manchester. One of the reasons for the move was to [create jobs in Greater Manchester beyond those that directly moved](https://www.nao.org.uk/wp-content/uploads/2013/05/10143-001_The-BBCs-move-to-Salford.pdf).

Did this happen?

In [late 2016](https://tomforth.co.uk/fiveyearsinmanchester/) and [early 2017](https://www.tomforth.co.uk/movingtomanchester/) Tom Forth made use of some [city-level data on the creative economy](https://www.nesta.org.uk/report/the-geography-of-creativity-in-the-uk/) built up from individual company sources by Nesta. It was too early to say much, but the early results seemed positive.

Later in 2017, [Paul Swinney at Centre for Cities made a more economically rigorous effort to judge that](https://www.centreforcities.org/reader/move-public-sector-jobs-london/relocation-bbc-activities-salford/). He was more sceptical. The work used data on historical location and financial performance for individual companies via the [Business Structure Database](https://www.enterpriseresearch.ac.uk/our-work/data-sources/business-structure-database/). By looking at businesses in economic sectors related to the BBC, the work showed that by 2016, five years after the move began, far fewer jobs than hoped had been created. The consideration of displacement (companies relocating to Salford Quays from another part of Greater Manchester) was important.

In the following years, Max Nathan, Henry Overman, and others worked on more advanced methods. The way that their synthetic control method addressed limitations in Forth and Swinney's work is detailed on page 6 of their paper [Multipliers from a major public sector relocation: The BBC moves to Salford](https://discovery.ucl.ac.uk/id/eprint/10203261/1/Nathan_dp2042.pdf).

## 2. Brexit: The economic impact of leaving the EU.

Synthetic controls are a common tool in economics and have been widely used in the evaluation of the impact of Brexit on the UK's economy. A full summary of such methods used for this application is found in John Springford's latest estimate of [The Economic Impact of Brexit](https://consoc.org.uk/publications/the-economic-impact-of-brexit/). In this example the technique involves creating a counterfactual time series of British economic performance before Brexit from a combination of other data series. The assumption of the technique is that British economic performance in a counterfactual where we voted to Remain in the EU would have continued on that path and the impact of Brexit can be deduced from the variation from that path.

## 3. Small business loans.

The third evaluation examples is the [SQW evaluation of the imapct of startup-up loans for the British Business Bank](https://www.sqw.co.uk/insights-and-publications/SUL_Evaluation). This does not use a synthetic control method, but does use The Data City data to create a control group of companies to compare against the treatment group of companies that received a start-up loan. Regression methods are used to try and decompose the factors that led to increased business growth following the intervention with the goal of isolating the impact of receiving a start-up loan.

## Automating evaluations using The Data City's unique and uniquely accessible data.

These examples of evaluations of economic impact require at least,

* Detailed historical company level financial data.
* An alternative to SIC codes to better classify companies into related sectors of activity.
* Detailed company location data and the ability to quickly create aggregate company performance measures for any geography.
* Quick and easy ways to create control groups of similar companies or synthetical controls to estimate impact of interventions.

All of the data to do this is built in to The Data City's Industry Engine, but we have not yet explored building tools to accelerate evaluations using our data.

In the coming eight weeks we will do this as part of our first data science fellowship. With Paul Swinney joining us just as we finish, we'll have the perfect change to see if we've managed to improve on his early work. We might even update that early evaluation of the impact of the BBC's move to Salford Quays.