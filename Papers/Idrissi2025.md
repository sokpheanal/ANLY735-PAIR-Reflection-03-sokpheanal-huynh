Unveil Sources of Uncertainty: Feature Contribution to Conformal
Prediction Intervals
Marouane Il Idrissia,b,e, Agathe Fernandes Machadoa, Ewen Gallicc,d, Arthur Charpentiera
aDépartement de Mathématiques, Université du Québec à Montréal, Montréal, QC Canada
bInstitut Intelligence et Données, Université Laval, Québec, QC Canada
cCNRS - Université de Montréal CRM – CNRS
dAix Marseille Univ, CNRS, AMSE, Marseille, France
eCorresponding Author - Email: ilidrissi.m@gmail.com
Abstract
Cooperativegametheorymethods,notablyShapleyvalues,havesignificantlyenhancedmachinelearning
(ML) interpretability. However, existing explainable AI (XAI) frameworks mainly attribute average
modelpredictions,overlookingpredictiveuncertainty. Thisworkaddressesthatgapbyproposinganovel,
model-agnosticuncertaintyattribution(UA)methodgroundedinconformalprediction(CP).Bydefining
cooperative games where CP interval properties—such as width and bounds—serve as value functions,
we systematically attribute predictive uncertainty to input features. Extending beyond the traditional
Shapley values, we use the richer class of Harsanyi allocations, and in particular the proportional
Shapley values, which distribute attribution proportionally to feature importance. We propose a Monte
Carlo approximation method with robust statistical guarantees to address computational feasibility,
significantly improving runtime efficiency. Our comprehensive experiments on synthetic benchmarks
and real-world datasets demonstrate the practical utility and interpretative depth of our approach. By
combining cooperative game theory and conformal prediction, we offer a rigorous, flexible toolkit for
understanding and communicating predictive uncertainty in high-stakes ML applications.
Keywords: Interpretable Machine Learning, Explainable Artificial Intelligence, Uncertainty
Attribution, Conformal Prediction, Shapley Values
1. Introduction
Overthepastseveralyears, thefieldofexplainableartificialintelligence(XAI)hasgainedsignificant
attention within machine-learning (ML) research [34, 45]. This interest is partly explained by the
growing need to interpret and justify models in high-stakes domains. For instance, in actuarial science,
interpretability clarifies premium calculations for policyholders [74]; in healthcare, it ensures that
diagnostic support systems rely on clinically meaningful features rather than spurious correlations [49];
and in criminal-risk assessment, it helps reduce historical biases and prevents sensitive attributes from
influencing predictions [11].
When decision-makers need to understand which features drive a model’s output, the theory of
cooperative games provides a resourceful framework: it enables a systematic methodology for feature
attributions. The Shapley value is the most widely used allocation rule for attributing predictions
to input features [40, 24, 41, 13]. Yet the Shapley values are only one member of a richer family of
allocations. The Harsanyi set of allocations [68] generalizes Shapley values and includes variants such
as the weighted and proportional Shapley values. This class of allocations yields a flexible spectrum of
importance measures rooted in different normative perspectives. Thus, it offers a unique playground
for defining a wide range of importance measures for a plethora of different goals.
Cooperative game theoretic approaches are already prominent in XAI through tools like SHAP [40],
which decompose model predictions into feature-wise contributions. While SHAP offers unparalleled
5202
yaM
91
]IA.sc[
1v81131.5052:viXra

insights into a model’s behavior, other key aspects may be of interest to practitioners. For instance,
quantifying predictive uncertainty has gained a lot of traction in modern applications, serving as
an important tool for assessing the reliability of model predictions. This capability is particularly
valuable in critical domains such as clinical diagnostics, autonomous driving, automated trading, and
energy production. Conformal prediction (CP) fulfills this need by producing finite-sample valid
predictionintervalsforvirtuallyanyMLmodelunderbroadassumptionsonthedata-generatingprocess
[58]. CP has demonstrated practical value in applications like environmental monitoring for climate
policymaking, where it effectively mitigates the risk of overconfident models [61]. The CP literature is
quickly expanding, with many recent theoretical and practical advances.
XAI and uncertainty quantification have often been combined to manage risks in high-stakes
settings [5, 53, 27, 25], leading to uncertainty attribution (UA) methods. In ML, UA seeks to identify
which features most influence a model’s predictive uncertainty [70]. Proxies for uncertainty include
information-theoretic quantities (e.g., entropy) [41, 13, 10, 31, 71, 12], predictive variance [6, 28], and
counterfactual explanations [4, 36, 50].
1.1. Contributions
Ourworkleveragesfeatureattributionmethodstodeepentheunderstandingofpredictiveuncertainty.
The main contributions are:
CP-based model-agnostic UA method to decompose uncertainty We introduce a novel
regression-model-agnostic UA approach to decompose uncertainty, measured using CP-based quantities
(e.g., width, midpoint, upper and lower values of the prediction intervals). This method effectively
provides actionable insights into how each feature affects predictive uncertainty.
Beyond Shapley: Harsanyi allocations and proportional Shapley values Building on
cooperative game theory, we employ the Harsanyi set of allocations and introduce the proportional
Shapley values. Whereas classical Shapley values implement an egalitarian redistribution of dividends,
proportional Shapley values allocate them in proportion to feature contributions. We use and compare
both allocations within our UA framework.
Statistically grounded approximations for faster computations Attribution methods
often suffer from high computational cost. Alongside an exact algorithm, we develop a Monte-Carlo
approximation scheme with provable statistical guarantees, granting explicit control over runtime.
Extensive empirical evaluation We validate our method through comprehensive experiments
on simulated and real-world datasets spanning diverse dimensions, sample sizes, and data types.
TheproofsofthetheoreticalresultspresentedinthearticlearepostponedtoAppendixAppendixB.
1.2. Related work
Feature attributions within the CP framework Several studies connect CP and XAI. Some
quantify uncertainty in feature-importance scores via CP [2, 3], while others use Shapley values as
conformity functions for CP intervals [29]. To date, only [42] directly attributes CP-derived uncertainty,
focusing on Shapley values with quantile-regression forests; UA methods tailored to CP remain largely
unexplored.
Other intersections of XAI and CP BeyondUA,substantialcrossoverexistsbetweenCP-based
uncertainty quantification and interpretable ML. Examples include oracle coaching combined with CP
[32]; conformal trees that enforce homogeneous outputs across leaves [21, 33]; and rule-based classifiers
exploiting rule-boundary geometry [46]. Feature-perturbation analyses extend to CP-specific quantities:
ConformaSight explains the size and coverage of non-adaptive CP sets via counterfactual perturbations
[75]; [38, 39] generate factual and counterfactual explanations of calibrated CP intervals; and [43] apply
permutation feature importance across training, calibration, and test sets for CP intervals in regression
models.
2

| 2. Conformal | predictions |     |     | and | cooperative |     | games |     |     |     |     |     |
| ------------ | ----------- | --- | --- | --- | ----------- | --- | ----- | --- | --- | --- | --- | --- |
This section first reviews the split conformal prediction (SCP) framework, also called inductive
in the literature [77], and then introduces the cooperative-game concepts required
| conformal | prediction |     |     |     |     |     |     |     |     |     |     |     |
| --------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
for our method.
|     | Let |     |     | )}n |     |     | n be a | sequence | of  |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------ | -------- | --- | --- | --- | --- |
Notations D := {(X i ,Y i ∈ (X ×Y) exchangeable random variables
i=1
drawn from an unknown distribution P , where X ⊆ Rd and Y ⊆ R. Let (X ,Y ) ∼ P
|     |     |     |     |     | X,Y |     |     |     |     |     | n+1 n+1 | X,Y |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------- | --- |
denote an additional data point outside D. Denote D = {1,...,d} and let P be its power set
D
(the set of all subsets of D). For each A ∈ P , let X(A) ∈ X ⊆ R|A| denote the sub-vector that
|     |     |     |     |     |     | D   | i   |     | A    |     |      |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- | --- | ---- | --- |
|     |     |     |     |     |     |     |     |     | X(∅) |     | X(D) |     |
retains only the features whose indices lie in A; by convention, := ∅ and := X . Define
|                   |     |                    |     |     |     |     |     |     | i   |     | i i |     |
| ----------------- | --- | ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| (cid:110)(cid:16) |     | (cid:17)(cid:111)n |     |     |     |     |     |     |     |     |     |     |
D := X(A),Y . Splitting [n] at random into two disjoint parts yields a training set DTr ⊂D
| A   | i   | i   |     |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
i=1
(with DTr ⊂D for each A) and a calibration set DCal ⊂D (with DCal ⊂D for each A).
|            | A A       |            |     |     |     |     |     |     | A   | A   |     |     |
| ---------- | --------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2.1. Split | conformal | prediction |     |     |     |     |     |     |     |     |     |     |
Constructing an SCP interval requires two ingredients: i) a fitted on
|     |     |     |     |     |     |     |     |     | model | f(cid:98) : | X → Y | DTr |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- | ----------- | ----- | --- |
by a learning algorithm A; ii) a conformity score s : X × Y × YX → R, used to form the set
| (cid:110) |                   |     |        | (cid:111) |           |           |          |          |     |           |     |     |
| --------- | ----------------- | --- | ------ | --------- | --------- | --------- | -------- | -------- | --- | --------- | --- | --- |
| S = s(X   | ,Y ,f(cid:98)):(X | ,Y  | )∈DCal |           | . The     | resulting | interval | is       |     |           |     |     |
|           | i i               | i   | i      |           |           |           |          |          |     |           |     |     |
|           |                   |     |        |           | (cid:110) |           | (cid:16) | (cid:17) |     | (cid:111) |     |     |
(1)
|     |     |     | C(cid:98)(X | n+1 ):= | y   | ∈Y : s | X n+1 ,y,f(cid:98) |     | ≤q 1−α | (S) , |     |     |
| --- | --- | --- | ----------- | ------- | --- | ------ | ------------------ | --- | ------ | ----- | --- | --- |
where q (S) is the empirical (1−α)-quantile of S for a chosen level α ∈ (0,1). Assuming no ties
1−α
| occur in | S, SCP | achieves | marginal |          | coverage       | [35]: |          |     |         |     |     |     |
| -------- | ------ | -------- | -------- | -------- | -------------- | ----- | -------- | --- | ------- | --- | --- | --- |
|          |        |          |          | (cid:16) |                |       | (cid:17) |     |         |     |     |     |
|          |        |          | 1−α≤P    |          |                |       |          |     |         | 1   |     |     |
|          |        |          |          |          | Y ∈C(cid:98)(X |       | ) ≤1−α+  |     |         | .   |     |     |
|          |        |          |          |          | n+1            |       | n+1      |     | #DCal+1 |     |     |     |
Tie-breaking randomization can tighten this bound to equality [69], but that refinement is not essential
for our purposes. Different choices of and give rise to distinct CP variants, three of which are
|            |        |     |     |     | f(cid:98) | s   |     |     |     |     |     |     |
| ---------- | ------ | --- | --- | --- | --------- | --- | --- | --- | --- | --- | --- | --- |
| summarized | below. |     |     |     |           |     |     |     |     |     |     |     |
Standard mean regression (SMR) When f(cid:98) is a mean-regression model, the SMR setting [77]
|        |           | (cid:16)        | (cid:17)                         | (cid:12)    | (cid:12) |             |     |     |           |     |     |     |
| ------ | --------- | --------------- | -------------------------------- | ----------- | -------- | ----------- | --- | --- | --------- | --- | --- | --- |
| adopts | the score | s x,y,f(cid:98) | =(cid:12)y−f(cid:98)(x)(cid:12), |             | yielding |             |     |     |           |     |     |     |
|        |           |                 |                                  | (cid:12)    | (cid:12) |             |     |     |           |     |     |     |
|        |           |                 |                                  |             |          | (cid:104)   |     |     | (cid:105) |     |     |     |
|        |           |                 |                                  | C(cid:98)(X | )=       | f(cid:98)(X | )±q |     | (S) .     |     |     |     |
|        |           |                 |                                  |             | n+1      |             | n+1 | 1−α |           |     |     |     |
It is important to note that the interval width is constant in . This lack of adaptability to the
X n+1
newly observed datapoint motivates the adoption of a more adaptive conformity score.
Local adaptive conformal prediction (LACP) In [35], the authors proposed to scale the
|     |     |     |     |     |     |     | (cid:16) | (cid:16) | (cid:17)(cid:17) | (cid:12) | (cid:12) |     |
| --- | --- | --- | --- | --- | --- | --- | -------- | -------- | ---------------- | -------- | -------- | --- |
residuals by an estimate of local dispersion, using s x,y, f(cid:98),σ =(cid:12) y−f(cid:98)(x)(cid:12) /σ (x). Here, f(cid:98) and σ
|     |     |     |     |     |     |     |     |     | (cid:98) | (cid:12) | (cid:12) (cid:98) | (cid:98) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- | -------- | ----------------- | -------- |
are typically models of the conditional mean and conditional dispersion (e.g., standard deviation, mean
| absolute | dispersion), | respectively. |     | The         | prediction |             | intervals | adopt    | the | form      |     |     |
| -------- | ------------ | ------------- | --- | ----------- | ---------- | ----------- | --------- | -------- | --- | --------- | --- | --- |
|          |              |               |     |             | (cid:104)  |             |           |          |     | (cid:105) |     |     |
|          |              |               |     | C(cid:98)(X | )=         | f(cid:98)(X | )±q       | (S)σ     | (X  | ) .       |     |     |
|          |              |               |     | n+1         |            | n+1         | 1−α       | (cid:98) | n+1 |           |     |     |
Conformalized quantile regression (CQR) Proposedby[56],CQRdiffersbyusingmodelsoflower
and upper levels of conditional quantiles. These models are fitted on DTr, leading to (cid:0) (cid:1),
|     |     |     |     |     |     |     |         |     |     |     | f(cid:98)= q (cid:98)low | ,q (cid:98)up |
| --- | --- | --- | --- | --- | --- | --- | ------- | --- | --- | --- | ------------------------ | ------------- |
|     |     |     |     |     |     |     | (cid:8) |     |     |     | (cid:9).                 |               |
along with the conformity score s(x,y,(q ,q ))=max q (x)−y, y−q (x) The interval is then
|     |     |     |     |     | (cid:98)low | (cid:98)up |     | (cid:98)low |     | (cid:98)up |     |     |
| --- | --- | --- | --- | --- | ----------- | ---------- | --- | ----------- | --- | ---------- | --- | --- |
given by
|     |     | C(cid:98)(X |     | )=[q        | (X  | )−q | (S), | q (X       | )+q | (S)]. |     |     |
| --- | --- | ----------- | --- | ----------- | --- | --- | ---- | ---------- | --- | ----- | --- | --- |
|     |     |             | n+1 | (cid:98)low | n+1 |     | 1−α  | (cid:98)up | n+1 | 1−α   |     |     |
Althoughusing(α/2,1−α/2)isastandardchoiceforlowerandupperquantilelevels(i.e.,for(q )),
(cid:98)low ,q (cid:98)up
| other choices | also | preserve | marginal |     | validity | [56]. |     |     |     |     |     |     |
| ------------- | ---- | -------- | -------- | --- | -------- | ----- | --- | --- | --- | --- | --- | --- |
3

| 2.2. | Cooperative |     | game | theory | for feature | attribution |     |     |     |     |     |     |
| ---- | ----------- | --- | ---- | ------ | ----------- | ----------- | --- | --- | --- | --- | --- | --- |
A (transferable-utility) cooperative game is a pair (D,v), where D ={1,...,d} is a set of players
and →R is the function. The goal of the value function is to quantify the value of each
|     | v :P | D   | value |     |     |     |     |     |     |     |     |     |
| --- | ---- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
coalition of players. An allocation (a.k.a., solution concept or payoff) [47] is a mapping ϕ :D →R.
v
An allocation is efficient if (cid:80) ϕ (j)=v(D)−v(∅), i.e., it redistributes v(D) among the players.
j∈D v
For any coalition A∈P , the Harsanyi dividend [22] is defined as φ (A)= (cid:80) (−1)|A|−|B|v(B)
|     |     |     |     | D   |     |     |     |     | v   | B⊆A |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
and represents the value created by the interaction among players in A. Here, can be interpreted as
v
the cumulative value of the players, while φ quantifies the added value of a coalition. The Harsanyi
v
| set | of allocations |          | [68] consists |      | of all | mappings | of the   | form     |       |         |       |     |
| --- | -------------- | -------- | ------------- | ---- | ------ | -------- | -------- | -------- | ----- | ------- | ----- | --- |
|     |                |          | (cid:88)      |      |        | with     |          | (cid:88) |       | and     | if    |     |
|     | ϕ              | (j)=     | λ             | (A)φ | (A),   |          | λ (A)≥0, | λ        | (A)=1 | λ (A)=0 | j ̸∈A |     |
|     | v              |          |               | j    | v      |          | j        |          | j     | j       |       |     |
|     |                | A∈PD:j∈A |               |      |        |          |          | j∈D      |       |         |       |     |
wheretheweightsystemλ:D×P →Rparameterizesthefamily. Theseallocationscanbeinterpreted
D
as redistributing the dividends back to the players. These allocations are efficient (Proposition Ap-
pendix A.2) and generalize many classes of allocations (Appendix Appendix A). In this context, the
well-known Shapley value arises as the egalitarian weight system λ (A)=1/|A|, yielding
j
|     |     |     |     |     |     |      | (cid:88) | φ (A) |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ---- | -------- | ----- | --- | --- | --- | --- |
|     |     |     |     |     |     | Shap | (j):=    | v     | ,   |     |     | (2) |
v
|A|
A∈PD:j∈A
which divides each dividend equally among coalition members (Appendix Appendix A.1).
(cid:14)(cid:80)
Using λPS(A)=v({j}) v({i}) characterizes the proportional Shapley values [60, 8],
|     |     | j   |     |        | i∈A     |     |          |          |          |     |     |     |
| --- | --- | --- | --- | ------ | ------- | --- | -------- | -------- | -------- | --- | --- | --- |
|     |     |     |     |        |         |     | (cid:88) | |v({j})| |          |     |     |     |
|     |     |     |     | P-Shap |         |     |          |          |          |     |     | (3) |
|     |     |     |     |        | v (j):= |     | (cid:80) |          | φ v (A), |     |     |     |
|v({j′})|
|     |     |     |     |     |     |     | A∈PD:j∈A | j′∈A |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------- | ---- | --- | --- | --- | --- |
which redistributes dividends in proportion to each of the players’ individual values.
Drawingananalogybetweenplayersandmodelfeatures,suchallocationshavebecomeacornerstone
of feature-importance analysis. Efficiency makes them ideal for decomposing v(D) into feature-level
contributions. Specific choices of tailor the interpretation: model explanation by decomposing a
v
predictionf(cid:98)(x)[78,40,66]; importancequantificationbydecomposingthemodel’svariance[48,19,23];
ormoreadvanceuncertaintyattributionsbyallocatingkernel-basedorinformation-theoreticuncertainty
| measures |             | [14, 9, | 71].           |       |              |               |            |           |     |     |     |     |
| -------- | ----------- | ------- | -------------- | ----- | ------------ | ------------- | ---------- | --------- | --- | --- | --- | --- |
| 3.       | Uncertainty |         | attribution    |       | of conformal |               | prediction | intervals |     |     |     |     |
| 3.1.     | Measures    |         | of uncertainty | based |              | on prediction | intervals  |           |     |     |     |     |
We introduce a regression model-agnostic UA method based on uncertainty measures based on CP.
More precisely, we introduce value functions derived from key quantities related to prediction intervals,
which can be applied to the SMR, LACP, or CQR methods for constructing CP intervals. Each choice
of value function defines a new cooperative game. We then propose to use the Shapley and proportional
Shapley allocations of these various games to define novel UA influence measures. The present work
focusesonthreevaluefunctionsbasedonCPintervals(CPintervalwidth, andthetwoboundarypoints
of the interval). Naturally, these value functions also depend on a data point, for which a prediction is
performed. However, it is essential to note that the theoretical results presented in this section will still
| hold | for | any value | function | related | to  | a prediction | interval. |     |     |     |     |     |
| ---- | --- | --------- | -------- | ------- | --- | ------------ | --------- | --- | --- | --- | --- | --- |
Let A be an algorithm that returns a regression model trained on DTr, and let be an instance
|     |     |     |     |     |     |     |     | f(cid:98) |     |     | x   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- | --- | --- | --- |
of X. For every A ∈ P , let f(cid:98)A be the output of A trained on D Tr, and let x be the subvector of
|     |     |     |     | D   |     |     |     |     | A   | A   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
X. Following Section 2.1, for every , let conformity scores ×YXA R and let
|     |           |     |     |     |     | A ∈ P | D         |     | s A : | X A ×Y | →   |     |
| --- | --------- | --- | --- | --- | --- | ----- | --------- | --- | ----- | ------ | --- | --- |
|     | (cid:110) |     |     |     |     |       | (cid:111) |     |       |        |     |     |
(A),Y (A),Y Cal . Then, asin(1), letC(cid:98)A (cid:0) x(A) (cid:1)betheCPintervalrelated
| S A | := s          | A (X | i ,f(cid:98)A | ):(X | i )∈D |     |     |     |     |     |     |     |
| --- | ------------- | ---- | ------------- | ---- | ----- | --- | --- | --- | --- | --- | --- | --- |
|     |               | i    |               | i    |       | A   |     |     |     |     |     |     |
| to  | the datapoint |      | x(A).         |      |       |     |     |     |     |     |     |     |
4

CP interval width Our first proposed CP-based measure of uncertainty is the width of the CP
interval, denoted ∆C(cid:98)(x) as in [43, 75]. To that end, we define the following value function:
(cid:16) (cid:17)
∀A∈P
D
, ∀x(A) ∈X
A
, v
wCP
(A,x):=∆C(cid:98)A x(A) .
Thus, this value function specializes to v (A)=2q (S ) for SMR, v (A)=2q (S)σ(x(A))
wCP 1−α A wCP 1−α (cid:98)
for LACP and v (A)=q (x(A))−q (x(A))+2q (S ) for CQR. For SMR, the width-based
wCP (cid:98)1−α/2 (cid:98)α/2 1−α A
feature importance remains the same across new observations.
Boundary points On top of the width of the interval, two key quantities would be its lower and
upper bounds. For example, when assessing flood risks, one may be more interested in the value of the
upper bound of the predicted river water level [26]. To that extent, we introduce the value functions
(cid:16) (cid:17) (cid:16) (cid:17)
∀A∈P
D
, ∀x(A) ∈X
A
, v
lowCP
(A):=infC(cid:98) x(A) , v
upCP
(A):=supC(cid:98) x(A) .
In the case of the SMR variant, we have v
lowCP
(A) = f(cid:98)(x(A))−q
1−α
(S
A
), for LACP, v
lowCP
(A) =
f(cid:98)(x(A))−q
1−α
(S
A
)σ
(cid:98)
(x(A)); and for CQR, v
lowCP
(A)=q
(cid:98)α/2
(x(A))−q
1−α
(S
A
).
Normalization and allocations We focus on the two allocations presented in Section 2.2, the
Shapley value Shap (j) and the proportional Shapley value P-Shap (j), where ω ∈{w,low,up}.
These two allocatio v nωsCoPffer desirable interpretative theoretical properti v eωsC.P
Proposition 3.1. For any data point x∈X, and value function v , with ω ∈{w, low, up},
ωCP
d d
(cid:88) (cid:88)
Shap (j,x)=v (D,x)−v (∅)= P-Shap (j,x).
vωCP ωCP ωCP vωCP
j=1 j=1
Since both the Shapley and proportional Shapley values are efficient, considering two allocation
schemesenablescomparingtheirrespectiverankings. Inpractice,agreementbetweenrankingsproduced
by the two allocations enhances confidence in the results, as it signals a lack of sensitivity to the
allocationchoice. Meanwhile,disagreementcanindicatethatthesituationismorecomplexandrequires
more attention. Moreover, the value functions can be normalized w.r.t. the baseline v (∅), where
ωCP
ω ∈{w,low,up}. Normalized value functions can be written, for x∈X, as
v (A,x(A))−v (∅)
∀A∈P , ∀x(A) ∈X v˜ (A,x(A)):= ωCP ωCP .
D A ωCP v (D,x)−v (∅)
ωCP CP
Hence, by leveraging Proposition 3.1, normalizing the value functions implies that the resulting
allocations will sum up to 1. However, since there are no general theoretical guarantees on the
monotonicity of the conformal scores of nested models, the allocations may fall outside the interval
[0,1], but still sum to 1.
3.2. Estimation and approximations
This section presents an exact and a Monte Carlo-based approximation scheme to compute the
Shapley and proportional Shapley values. The presented procedures are the same for any CP-based
value function.
Exact computations The procedure to compute the exact Shapley and proportional Shapley
valuesisfairlystraightforward,andcanbebrokendownintotwosteps. Thefirststeprequiresevaluating
the value functions. Thus, for every subset of features A ∈ P , we must train the corresponding
D
model f(cid:98)A using algorithm A on D
A
Tr, and then compute the conformity scores S
A
using D
A
Cal. As a
whole, we need to train 2d models and compute their conformity scores. Once we have access to the
conformity scores, for a new data point x∈X, computing the prediction intervals C(cid:98)A (x(A)) for every
A∈P allows extracting the evaluation of the value functions (e.g., width or boundary points of the
D
5

| Algorithm |     | 1 Exact |     | computation | procedure |     |     |     |     |     |     |
| --------- | --- | ------- | --- | ----------- | --------- | --- | --- | --- | --- | --- | --- |
Require: Data D = {(X ,Y )}n , new data DNew = {(X ,Y )}m , miscoverage level α ∈ (0,1),
|     |     |     |     | i   | i i=1 |     |     | i i | i=n+1 |     |     |
| --- | --- | --- | --- | --- | ----- | --- | --- | --- | ----- | --- | --- |
regression algorithm f, conformity score algorithm s, and a weight assignment →R that
λ:P D
associates a weight λ(A) to each subset A⊆D, where D ={1,...,d} is the set of variable indices
in X.
|     | Randomly | split |      | into two | disjoint            | datasets |     | and      |     |     |     |
| --- | -------- | ----- | ---- | -------- | ------------------- | -------- | --- | -------- | --- | --- | --- |
| 1:  |          |       | D    |          |                     |          |     | DTr DCal |     |     |     |
|     | for A∈P  |       | do   |          |                     |          |     |          |     |     |     |
| 2:  |          | D     |      |          |                     |          |     |          |     |     |     |
|     | Define   |       | Tr   |          |                     |          |     |          |     |     |     |
| 3:  |          | D     | ={(X | i,A      | ,Y i )} (Xi,Yi)∈DTr |          |     |          |     |     |     |
A
|     | Define | D   | Cal ={(X |     | ,Y )}          |     |     |     |     |     |     |
| --- | ------ | --- | -------- | --- | -------------- | --- | --- | --- | --- | --- | --- |
| 4:  |        |     | A        | i,A | i (Xi,Yi)∈DCal |     |     |     |     |     |     |
| 5:  | Define | D   | New ={X  |     | }              |     |     |     |     |     |     |
|     |        |     | A        | i,A | Xi∈DNew        |     |     |     |     |     |     |
|     | Train  | fˆ  | on       | Tr  |                |     |     |     |     |     |     |
| 6:  |        | A   | D        |     |                |     |     |     |     |     |     |
A
| 7:  | Compute |         | conformity |     | scores     | sˆ for | each (X  | ,Y )∈D Cal |     |     |     |
| --- | ------- | ------- | ---------- | --- | ---------- | ------ | -------- | ---------- | --- | --- | --- |
|     |         |         |            |     |            | i      |          | i i A      |     |     |     |
|     | for     | X ∈D    | New        | do  |            |        |          |            |     |     |     |
| 8:  |         | i       | A          |     |            |        |          |            |     |     |     |
|     |         | Compute | conformal  |     | prediction |        | interval |            |     |     |     |
| 9:  |         |         |            |     |            |        |          | C (X )     |     |     |     |
|     |         |         |            |     |            |        |          | A i        |     |     |     |
| 10: |         | Compute | associated |     | value      | v(A;X  | )        |            |     |     |     |
i
| 11: | end | for |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
12: end for
|     | Initialize | matrix | Φ∈Rm×d |     | with | zeros |     |     |     |     |     |
| --- | ---------- | ------ | ------ | --- | ---- | ----- | --- | --- | --- | --- | --- |
13:
∈DNew
| 14: | for X |     | do  |     |     |     |     |     |     |     |     |
| --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
i
| 15: | for | j ∈D | do  |     |     |     |     |     |     |     |     |
| --- | --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Compute
| 16: |     |       | ϕ     | i (j) |     |     |     |     |     |     |     |
| --- | --- | ----- | ----- | ----- | --- | --- | --- | --- | --- | --- | --- |
| 17: |     | Store | ϕ (j) | in Φ  |     |     |     |     |     |     |     |
|     |     |       | i     | i,j   |     |     |     |     |     |     |     |
|     | end | for   |       |       |     |     |     |     |     |     |     |
18:
19: end for
| 20: | return | Φ   |     |     |     |     |     |     |     |     |     |
| --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
interval). The second step entails aggregating the 2d evaluated value functions to return either the
Shapley or proportional Shapley values according to (2) or (3), respectively. This procedure is detailed
| in Algorithm |     | 1.  |     |     |     |     |     |     |     |     |     |
| ------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Approximation procedure Overall, the bulk of the computational strain lies in training the
models. A solution to drive the cost down would be to reduce the number of models to train from 2d
to a more controllable quantity. To address this, several strategies have been proposed in the literature.
[51, 37] proposed removing coalitions whose cardinality is above a certain threshold based on heuristics,
[30] proposed to randomly sample according to a distribution over the coalitions, or [78, 64] proposed a
Monte Carlo-type sampling of the permutations of players for the Shapley values. We generalize the
latter approach to a broad class of allocations to produce an approximation scheme with statistical
| guarantees |     | (see | Theorems | 1   | and 2). |     |     |     |     |     |     |
| ---------- | --- | ---- | -------- | --- | ------- | --- | --- | --- | --- | --- | --- |
TheShapleyandproportionalShapleyvaluesarepartofaclassofallocationsknownastheWeberset
[72] (see Appendix Appendix A.1), which relies on the ordering of players. By randomly sampling of
m
thed!possibleorderings, werequire, atworst, m×dmodelstotrain, bypassingtheexactcomputations’
exponential scaling. The way the sampling is done dictates the allocation being approximated. For
instance, a uniform sample of the permutations approximates the Shapley values. For the proportional
values, a different distribution is needed (see Appendix Appendix A.3). The complete approximation
procedureisdescribedinAlgorithm2,withextensivedetailsandexplanationsinAppendixAppendixA.
|         |     | (Statistical |     | properties |     | of Algorithm |     | 2).           |          |            |           |
| ------- | --- | ------------ | --- | ---------- | --- | ------------ | --- | ------------- | -------- | ---------- | --------- |
| Theorem |     | 1            |     |            |     |              |     | For any value | function | v, and any | j ∈ D the |
approximations Shap (cid:92) (j) and P-Shap (cid:92) (j) using Algorithm 2 are unbiased, strongly consistent, and
|                |     |        | v   |            |     | v    |         |                           |     |     |     |
| -------------- | --- | ------ | --- | ---------- | --- | ---- | ------- | ------------------------- | --- | --- | --- |
| asymptotically |     | normal |     | estimators | of  | Shap | (j) and | P-Shap (j), respectively. |     |     |     |
|                |     |        |     |            |     | v    |         | v                         |     |     |     |
Moreover, it is possible to reweigh already sampled permutations to produce estimates for both the
Shapley and proportional Shapley, without having to run Algorithm 2 again. This procedure relies on
6

| A   |     |     | B   |     |     |     |     | C   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
1.00
|       | X ,X  | ,X ,X |                 | 30000 |     |     |     |                    |      |     |
| ----- | ----- | ----- | --------------- | ----- | --- | --- | --- | ------------------ | ---- | --- |
|       | 12 16 | 8 13  |                 |       |     |     |     |                    |      |     |
| 0.225 |       |       |                 |       |     |     |     | )%(emitnurevitaleR |      |     |
|       |       |       | stesbusforebmuN |       | e   |     |     |                    |      |     |
|       |       |       |                 |       | s   |     |     |                    | 0.75 |     |
| 0.200 |       |       |                 |       | c a |     |     |                    |      |     |
eulaVyelpahS
|     |     |     |     | 20000 | st  |     |     |     |     |     |
| --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- |
or
| 0.175 |     |     |     |       | W   |     |     |     | 0.50 |     |
| ----- | --- | --- | --- | ----- | --- | --- | --- | --- | ---- | --- |
| 0.150 |     |     |     | 10000 |     |     |     |     |      |     |
0.25
0.125
0.00
| 0.100 |     |     |     |     | 0   |     |     |     |     |     |
| ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
0 1000 2000 3000 4000 5000 0 10002000300040005000 0 1000 2000 3000 4000 5000
Numberofpermutations Numberofpermutations Numberofpermutations
Figure1: Empiricalconvergenceforthefourmostimportantfeatures(A),numberoftrainedmodelsandworstcase
scenario m×d (B), and runtime relative to exact computations (C) for the empirical study of the Monte Carlo
approximationscheme
| an importance | sampling     | (IS) scheme |     | detailed | in Appendix     |     | Appendix | A.4.      |          |                 |
| ------------- | ------------ | ----------- | --- | -------- | --------------- | --- | -------- | --------- | -------- | --------------- |
|               | (Statistical | properties  | of  | the IS   | approximation). |     |          |           |          |                 |
| Theorem       | 2            |             |     |          |                 |     | For      | any value | function | v, and any j ∈D |
the approximations Shap (cid:91) and P-Shap (cid:92) using the importance sampling scheme described in
|     |     | v,IS (j) |     |     | v,IS (j) |     |     |     |     |     |
| --- | --- | -------- | --- | --- | -------- | --- | --- | --- | --- | --- |
Appendix Appendix A.4 are unbiased, strongly consistent, and asymptotically normal estimators of
| Shap (j) | and P-Shap | (j), respectively. |     |     |     |     |     |     |     |     |
| -------- | ---------- | ------------------ | --- | --- | --- | --- | --- | --- | --- | --- |
| v        |            | v                  |     |     |     |     |     |     |     |     |
4. Experiments
Further information on the datasets, data-generating processes, and supplemental experiments
appears in Appendix Appendix C. Reproducible codes, datasets, experiments, and figures are available
| in the accompanying |             | GitHub repository1. |                    |     |     |             |       |     |     |     |
| ------------------- | ----------- | ------------------- | ------------------ | --- | --- | ----------- | ----- | --- | --- | --- |
| 4.1. Empirical      | convergence | of                  | the approximations |     |     | and runtime | gains |     |     |     |
Beyond the theoretical guarantees in Section 3.2, we examine how the Monte Carlo approximation
behaves as the number of sampled permutations increases. We adopt a modified Sobol’–Levitan
benchmark [63, 44, 67] where is comprised of mutually independent
|        |                 | X      | =(X | ,...,X | )⊤  | ∼U(0,1)×16 |     |     |     |     |
| ------ | --------------- | ------ | --- | ------ | --- | ---------- | --- | --- | --- | --- |
|        |                 |        |     | 1      | 16  |            |     |     |     |     |
| random | variables, with | target |     |        |     |            |     |     |     |     |
16
|     |        | (cid:2) (cid:3) | (cid:89) exp[β | ]−1 |     |         |         |     |            |     |
| --- | ------ | --------------- | -------------- | --- | --- | ------- | ------- | --- | ---------- | --- |
|     | Y =exp | β⊤X +           |                | i   | +ϵ  | , where | β ∈R16, | and | ϵ ∼N(0,1). |     |
|     |        |                 |                |     |     | Y       |         |     | Y          |     |
β i
i=1
A sampled dataset of size is randomly split for training and calibration, respectively.
|     |     | 1,000 |     |     |     | 80%−20% |     |     |     |     |
| --- | --- | ----- | --- | --- | --- | ------- | --- | --- | --- | --- |
We decompose the CP interval width using the SMR method on linear models on 50 test samples using
the Shapley values as a baseline. For permutation counts m∈{100,200,...,5,000} we perform 150
repetitions of the Monte Carlo approximations, recording the estimated Shapley values, the number of
fitted linear models, and the effective runtime2. Figure 1 (and the full curves in Section Appendix C.1)
show convergence toward the exact values (Panel A), a model count far below the worst-case m×d
| (Panel B), | and, for example, | a   | ten-fold | speed-up | at  |     | (Panel | C). |     |     |
| ---------- | ----------------- | --- | -------- | -------- | --- | --- | ------ | --- | --- | --- |
m=1,000
1ThecontentsoftheGitHubrepositoryaremadeavailableforreviewasasupplementaryzipfile.
2Single-corerunsonanAMDRyzenPRO4750Uwith32GBRAM,R4.2.1.
7

| 4.2. | Feature | selection beyond | moment-dependent |     | importance |     |     |     |
| ---- | ------- | ---------------- | ---------------- | --- | ---------- | --- | --- | --- |
We next compare importance rankings based on CP intervals with traditional rankings derived from
the conditional mean (SHAP [40]) and conditional variance. Using a variant of Friedman’s benchmark
| [20], | as in [71], | let X =(X | ,...,X | )⊤ ∼U(0,1)×11 |     | and define |     |     |
| ----- | ----------- | --------- | ------ | ------------- | --- | ---------- | --- | --- |
1 11
−0.5)2+10X
|     |     | V =10sin(πX | 1 X 2 )+20(X | 3          |     | 4 +5X | 5 +ϵ V , ϵ V ∼N(0,1), |     |
| --- | --- | ----------- | ------------ | ---------- | --- | ----- | --------------------- | --- |
|     |     | Z =10sin(πX | X )+20(X     | −0.5)2+10X |     | +5X   | +ϵ , ϵ ∼N(0,1),       |     |
|     |     |             | 6 7          | 8          |     | 9     | 10 Z Z                |     |
and the target variable is distributed according to Y ∼N(Z,V). Thus (X ,...,X ) govern the mean,
|      |          |                         |     |      |           |     | 6   | 10  |
| ---- | -------- | ----------------------- | --- | ---- | --------- | --- | --- | --- |
|      |          | the heteroskedasticity, |     | and  | is noise. |     |     |     |
| (X 1 | ,...,X 5 | )                       |     | X 11 |           |     |     |     |
We generate a training set of size 10,000, and calibration and test sets of size 5,000 each. We fit
LightGBM models, with 30 trees each with a maximum depth of 10 leaves. On the training data, we fit
models of the conditional mean, the conditional variance (using the squared residuals), and along with
thecalibrationdata,wecomputetheLACPandCQR(usingthepinballlosswithlevelsequalto0.1and
0.9) intervals with coverage of (α=0.01). We then compute the Shapley values decompositions
99%
on the test set, resulting in Figure 2. We can notice that both LACP and CQR uncertainty measures
based on width are between the conditional mean and the conditional variance. This demonstrates that
these quantile-based measures do go beyond typical moment-based importance measures to quantify
importance, effectively adding a different interpretative layer to the XAI practitioner’s toolbox.
|     | Cond. | Mean | Cond. | Var | CQR | - Width | LACP - | Width |
| --- | ----- | ---- | ----- | --- | --- | ------- | ------ | ----- |
Featurevalue
X11
1.00
X10
X9
0.75
X8
X7
X6
0.50
X5
X4
X3 0.25
X2
X1
-5.0 -2.5 0.0 2.5 5.0 -5.0 -2.5 0.0 2.5 5.0 -5.0 -2.5 0.0 2.5 5.0 -5.0 -2.5 0.0 2.5 5.0 0.00
Featureattribution
Figure2: FeatureattributionforthemodifiedFriedmanexample. Thebarsmarkthe90%intervals.
| 4.3. | Real-world | datasets |     |     |     |     |     |     |
| ---- | ---------- | -------- | --- | --- | --- | --- | --- | --- |
We conduct experiments on real-world classical datasets for CP [56, 73], detailed in Table 1. They
are chosen for their diversity in size, dimensionality, and data types. of each dataset forms a
20%
held-out test set; the remainder is split 80:20 into training and calibration sets, respectively. We use
either exact computations or Monte Carlo approximations depending on the dataset.
|     |        |                     |     | For SMR | and | LACP we | fit linear regression | (LR), LightGBM |
| --- | ------ | ------------------- | --- | ------- | --- | ------- | --------------------- | -------------- |
|     | Models | and hyperparameters |     |         |     |         |                       |                |
(LGB),andrandom-forest(RF)models; forCQRweusequantileLR(Q-LR)andquantileRF(Q-RF).3
All models in LACP use LGB. Unless specified, package defaults are retained. For RF and Q-RF
σ (cid:98)
we set t he minimum node size to 20% of training cases, limit the number of trees to 75, and choose
√
mtry= d. LGB uses boosting rounds, except for UScrime, star, and blog, where rounds are
|     |     | 100 |     |     |     |     |     | 25  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
adopted. Q-LR employs the Frisch–Newton algorithm with default machine precision.
Selected results We present a few selected interesting results. We refer the interested reader
to Appendix Appendix C for more details on the remainder of the visualizations. First, we showcase
| 3RPackages: |     | stats,lightgbm,randomForest,quantreg,andquantregForest,respectively. |     |     |     |     |     |     |
| ----------- | --- | -------------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
8

Dataset Description n d Estimation Source
bike Bike rental data 17,379 12 Exact [18]
blog Number of comments per blog posts 52,397 238 m=50 [7]
casp Physicochemical properties of proteins 45,730 9 Exact [52]
concrete Concrete compressive strength 1,030 8 Exact [76]
facebook Engagement of facebook posts 79,788 37 m=200 [62]
UScrime Crime data in the US 1,993 101 m=200 [54]
Effect of reducing class size on test
star 2,161 38 m=200 [1]
scores
Table1: Realworlddatasetline-upforrealisticexperiments
how our approach differs from standard approaches regarding importance rankings. Figure 3 contrasts
importance rankings based on interval width (LACP) with those based on the conditional mean
(SHAP) for RF and LGB on the concrete data. The center of the ellipses represents the mean ranks
over the test set. Uncertainties, are represented in the shape of the ellipses. Vertical and horizontal
elongations are proportional to the ranking standard deviation for the conditional mean and the CP
interval width, respectively. One can notice that our proposed approach does not yield the same
importance rankings as the conditional mean-based feature attribution. This is explained by the CP
framework’srelianceonquantiles,whicharemuchmoregeneralthanmoment-basedstatistics,asalready
demonstrated in Section 4.2. Figure 4 depicts rank frequencies for (absolute value of) the Shapley and
RF LightGBM
Shapley P-Shapley Shapley P-Shapley
8 8 8 8
6 6 6 6
4 4 4 4
2 2 2 2
2 4 6 8 2 4 6 8 2 4 6 8 2 4 6 8
RankWidth
naem
.dnoCknaR
age bfs ca cement fiag flash h2o spt
Figure3: LACPwidth-basedvs. conditionalmean-basedimportancerankingsforRFandLGBmodelsontheconcrete
dataset.
proportional Shapley allocations of the upper bound (CQR) on the facebook dataset. Panel (A) shows
the complete rank distributions, while Panel (B) highlights the most frequent top-ranked features.
One can notice that, on the facebook dataset, the allocation choice can impact the features’ overall
ranking. Unchanged rankings can indicate stability (and thus confidence) between the allocations. In
contrast, the multiplicities of differing rankings enable a more complete depiction of the importance,
since a choice of allocation only tells one side of the story. Moreover, the overall feature ranking also
depend on the model choice. Not only the rankings differ between Q-RF and Q-LR models, but also
the frequencies at which these rankings are observed. This indicates a more nuanced and locally diverse
definition of uncertainties.
9

| A       |           | B    |     |         |     |     |           |     |
| ------- | --------- | ---- | --- | ------- | --- | --- | --------- | --- |
| Shapley | P-Shapley |      |     | Shapley |     |     | P-Shapley |     |
| X37     |           | 1.00 |     |         |     |     |           |     |
Quant.Reg.
0.75
100% testsetniycneuqerfknaR
|     |     | 0.50     | X12 | X27 | X1 X17 X3 |     |        |     |
| --- | --- | -------- | --- | --- | --------- | --- | ------ | --- |
|     |     |          |     |     |           | X12 |        | X27 |
|     |     | 75% 0.25 |     |     |           |     | X1 X17 |     |
X2
| X1  |     | 0.00 |     |     |     |     |     |     |
| --- | --- | ---- | --- | --- | --- | --- | --- | --- |
50%
| X37 |     | 1.00     |     |     |     |     |     |          |
| --- | --- | -------- | --- | --- | --- | --- | --- | -------- |
|     |     | 25% 0.75 |     |     |     |     |     | Quant.RF |
0.50
0.25
|     |         |      | X35 | X35 X30 | X30 X30    | X28 | X17 |       |
| --- | ------- | ---- | --- | ------- | ---------- | --- | --- | ----- |
| X1  |         | 0.00 |     |         |            |     | X28 | X4 X4 |
| 1   | 37 1 37 |      | 1   | 2       | 3 4 5      | 1   | 2 3 | 4 5   |
|     | Rank    |      |     |         | Rank(top5) |     |     |       |
Figure4: Rankfrequencyofallthefeatures(A)andtop5mostimportantfeature(B)overthetestdata. CQRupper
bound-basedimportancerankingsforQ-LRandQ-RFmodelsonthefacebookdatasetareused.
5. Conclusion
We proposed a regression-model-agnostic uncertainty attribution (UA) framework that combines
cooperative-game theory with conformal prediction (CP) to interpret predictive uncertainty feature-
wise. By defining cooperative games whose value functions encode CP-interval properties, we extend
attribution beyond mean predictions and offer insights suited to high-stakes settings. Building on the
Shapleyvalue,weemployedthebroaderHarsanyiallocationfamilyandhighlightedproportionalShapley
values, which distribute uncertainty proportionately to individual feature contributions. We introduced
a Monte Carlo approximation with proven unbiasedness, consistency, and asymptotic normality to
curb the high computational cost, making large-scale applications feasible. Experiments on synthetic
benchmarks and varied real-world datasets showed that CP-based uncertainty attributions diverge
markedly from moment-based rankings, illustrating our method’s added value. Future work includes
automatically defining optimal allocations specialized for XAI tasks, exploring other venues to link XAI
and the CP framework (e.g., beyond SCP, CP intervals for time series or sets) [77] and the statistical
study and adaptation of the approximation schemes based on surrogate models [40, 30].
References
[1] C. M. Achilles, B. A. Nye, J. B. Zaharias, and B. D. Fulton. The Lasting Benefits Study (LBS)
in Grades 4 and 5 (1990-1991): A Legacy from Tennessee’s Four-Year (K-3) Class-Size Study
(1985-1989), Project STAR. Technical report, Tennessee State University, Nashville. Center of
ExcellenceforResearchinBasicSkills.,January1993. URLhttps://eric.ed.gov/?id=ED356559.
| ERIC Number: | ED356559. |     |     |     |     |     |     |     |
| ------------ | --------- | --- | --- | --- | --- | --- | --- | --- |
[2] Amr Alkhatib, Henrik Bostrom, Sofiane Ennadir, and Ulf Johansson. Approximating score-based
explanation techniques using conformal regression. In Harris Papadopoulos, Khuong An Nguyen,
Henrik Boström, and Lars Carlsson, editors, Proceedings of the Twelfth Symposium on Conformal
|     |     | Applications, |     | volume | 204 of |     |     |     |
| --- | --- | ------------- | --- | ------ | ------ | --- | --- | --- |
and Probabilistic Prediction with Proceedings of Machine Learning
Research, pages 450–469. PMLR, 13–15 Sep 2023. URL https://proceedings.mlr.press/v204/
alkhatib23a.html.
[3] Amr Alkhatib, Henrik Boström, and Ulf Johansson. Estimating quality of approximated shapley
valuesusingconformalprediction. InSimoneVantini,MatteoFontana,AldoSolari,HenrikBoström,
andLarsCarlsson,editors,ProceedingsoftheThirteenthSymposiumonConformalandProbabilistic
Prediction with Applications, volume 230 of Proceedings of Machine Learning Research, pages 158–
174. PMLR, 09–11 Sep 2024. URL https://proceedings.mlr.press/v230/alkhatib24a.html.
10

[4] Javier Antoran, Umang Bhatt, Tameem Adel, Adrian Weller, and José Miguel Hernández-Lobato.
Getting a {clue}: A method for explaining uncertainty estimates. In International Conference on
Learning Representations, 2021. URL https://openreview.net/forum?id=XSLF1XFq5h.
[5] AlejandroBarredoArrieta, Natalia Díaz-Rodríguez, Javier DelSer, Adrien Bennetot, SihamTabik,
Alberto Barbado, Salvador Garcia, Sergio Gil-Lopez, Daniel Molina, Richard Benjamins, Raja
Chatila, and Francisco Herrera. Explainable Artificial Intelligence (XAI): Concepts, taxonomies,
opportunities and challenges toward responsible AI. Information Fusion, 58:82–115, June 2020.
ISSN 1566-2535. doi: 10.1016/j.inffus.2019.12.012. URL https://www.sciencedirect.com/
science/article/pii/S1566253519308103.
[6] FlorianBley,SebastianLapuschkin,WojciechSamek,andGrégoireMontavon.Explainingpredictive
uncertainty by exposing second-order effects. Pattern Recognition, 160:111171, 2025. ISSN 0031-
3203. doi: https://doi.org/10.1016/j.patcog.2024.111171. URL https://www.sciencedirect.
com/science/article/pii/S0031320324009221.
[7] Krisztian Buza. Blogfeedback. UCI Machine Learning Repository, 2014. DOI:
https://doi.org/10.24432/C58S3F.
[8] Sylvain Béal, Sylvain Ferrières, Eric Rémila, and Phillipe Solal. The proportional Shapley value
and applications. Games and Economic Behavior, 108:93–112, March 2018. ISSN 0899-8256.
doi: 10.1016/j.geb.2017.08.010. URL https://www.sciencedirect.com/science/article/pii/
S0899825617301446.
[9] Siu Lun Chau, Robert Hu, Javier González, and Dino Sejdinovic. RKHS-SHAP: Shapley Values
for Kernel Methods. In S. Koyejo, S. Mohamed, A. Agarwal, D. Belgrave, K. Cho, and A. Oh,
editors, Advances in Neural Information Processing Systems, volume 35, pages 13050–13063.
Curran Associates, Inc., 2022. URL https://proceedings.neurips.cc/paper_files/paper/
2022/file/54bb63eaec676b87a2278a22b1bd02a2-Paper-Conference.pdf.
[10] Jianbo Chen, Le Song, Martin Wainwright, and Michael Jordan. Learning to explain: An
information-theoretic perspective on model interpretation. In Jennifer Dy and Andreas Krause,
editors, Proceedings of the 35th International Conference on Machine Learning, volume 80 of
Proceedings of Machine Learning Research, pages 883–892. PMLR, 10–15 Jul 2018. URL https:
//proceedings.mlr.press/v80/chen18j.html.
[11] Alexandra Chouldechova. Fair prediction with disparate impact: A study of bias in recidivism
prediction instruments. Big Data, 5(2):153–163, 2017. doi: 10.1089/big.2016.0047. URL https:
//doi.org/10.1089/big.2016.0047. PMID: 28632438.
[12] Ian Covert, Scott M Lundberg, and Su-In Lee. Understanding global feature contributions with
additive importance measures. In H. Larochelle, M. Ranzato, R. Hadsell, M.F. Balcan, and H. Lin,
editors, Advances in Neural Information Processing Systems, volume 33, pages 17212–17223.
Curran Associates, Inc., 2020. URL https://proceedings.neurips.cc/paper_files/paper/
2020/file/c7bf0b7c1a86d5eb3be2c722cf2cf746-Paper.pdf.
[13] Ian Covert, Scott Lundberg, and Su-In Lee. Explaining by removing: A unified framework for
model explanation. Journal of Machine Learning Research, 22(209):1–90, 2021. URL http:
//jmlr.org/papers/v22/20-1316.html.
[14] Sébastien da Veiga. Kernel-based anova decomposition and shapley effects – application to global
sensitivity analysis, 2021. URL https://arxiv.org/abs/2101.05487.
[15] PierreDehez. OnHarsanyiDividendsandAsymmetricValues. International Game Theory Review,
19(03):1750012, September 2017. ISSN 0219-1989, 1793-6675. doi: 10.1142/S0219198917500128.
URL https://www.worldscientific.com/doi/abs/10.1142/S0219198917500128.
11

[16] Jean Derks. A new proof for Weber’s characterization of the random order values. Mathematical
Sciences, 49(3):327–334, May 2005. ISSN 0165-4896. doi: 10.1016/j.mathsocsci.2004.10.001.
Social
URL https://www.sciencedirect.com/science/article/pii/S0165489604000885.
[17] Jean Derks, Gerard van der Laan, and Valeri Vasil’ev. On harsanyi payoff vectors and the
weber set. Tinbergen Institute Discussion Papers 02-105/1, Tinbergen Institute, Oct 2002. URL
https://ideas.repec.org/p/tin/wpaper/20020105.html.
[18] Hadi Fanaee-T. Bike sharing. UCI Machine Learning Repository, 2013. DOI:
https://doi.org/10.24432/C5W894.
[19] Thomas Fel, Remi Cadene, Mathieu Chalvidal, Matthieu Cord, David Vigouroux, and Thomas
Serre. Look at the Variance! Efficient Black-box Explanations with Sobol-based Sensitivity
Analysis. In Advances in Neural Information Processing Systems, volume 34, pages 26005–
| 26014. | Curran Associates, | Inc., | 2021. URL |     |     |     |     |
| ------ | ------------------ | ----- | --------- | --- | --- | --- | --- |
https://proceedings.neurips.cc/paper/2021/
hash/da94cbeff56cfda50785df477941308b-Abstract.html.
| [20] Jerome | H. Friedman. | Multivariate | Adaptive | Regression | Splines. |               |         |
| ----------- | ------------ | ------------ | -------- | ---------- | -------- | ------------- | ------- |
|             |              |              |          |            |          | The Annals of | Statis- |
tics, 19(1):1–67, March 1991. ISSN 0090-5364, 2168-8966. doi: 10.1214/aos/1176347963.
URL
https://projecteuclid.org/journals/annals-of-statistics/volume-19/issue-1/
Multivariate-Adaptive-Regression-Splines/10.1214/aos/1176347963.full. Publisher:
| Institute | of Mathematical | Statistics. |     |     |     |     |     |
| --------- | --------------- | ----------- | --- | --- | --- | --- | --- |
[21] Natalia Martinez Gil, Dhaval Patel, Chandra Reddy, Giridhar Ganapavarapu, Roman Vaculin, and
JayantKalagnanam. Identifyinghomogeneousandinterpretablegroupsforconformaiprediction. In
Proceedings of the Fortieth Conference on Uncertainty in Artificial Intelligence,UAI’24.JMLR.org,
2024.
[22] John C. Harsanyi. A simplified bargaining model for the n-person cooperative game.
International
Economic Review, 4(2):194–220, 1963. ISSN 00206598, 14682354. URL http://www.jstor.org/
stable/2525487.
[23] Margot Herin, Marouane Il Idrissi, Vincent Chabridon, and Bertrand Iooss. Proportional marginal
effects for global sensitivity analysis. Quantification, 12(2):
|     |     |     | SIAM/ASA | Journal | on Uncertainty |     |     |
| --- | --- | --- | -------- | ------- | -------------- | --- | --- |
667–692, 2024. doi: 10.1137/22M153032X. URL https://doi.org/10.1137/22M153032X.
[24] Tom Heskes, Ioan Gabriel Bucur, Evi Sijben, and Tom Claassen. Causal shapley values: exploiting
causal knowledge to explain individual predictions of complex models. In Proceedings of the 34th
|               |                         |           |                     |            | Systems, NIPS | ’20, Red Hook, | NY, |
| ------------- | ----------------------- | --------- | ------------------- | ---------- | ------------- | -------------- | --- |
| International | Conference              | on Neural | Information         | Processing |               |                |     |
| USA,          | 2020. Curran Associates | Inc.      | ISBN 9781713829546. |            |               |                |     |
[25] Marouane Il Idrissi, Nicolas Bousquet, Fabrice Gamboa, Bertrand Iooss, and Jean-Michel Loubes.
Quantile-constrained wasserstein projections for robust interpretability of numerical and machine
learningmodels. ElectronicJournalofStatistics,18(2):2721–2770,2024. doi: 10.1214/24-EJS2268.
| URL | https://doi.org/10.1214/24-EJS2268. |     |     |     |     |     |     |
| --- | ----------------------------------- | --- | --- | --- | --- | --- | --- |
[26] Bertrand Iooss and Paul Lemaître. A Review on Global Sensitivity Analysis Methods. In
|             |            |                            |     |            | Systems, | volume 59, pages | 101– |
| ----------- | ---------- | -------------------------- | --- | ---------- | -------- | ---------------- | ---- |
| Uncertainty | Management | in Simulation-Optimization |     | of Complex |          |                  |      |
122. Springer US, Boston, MA, 2015. ISBN 978-1-4899-7546-1 978-1-4899-7547-8. doi: 10.1007/
978-1-4899-7547-8_5. URL http://link.springer.com/10.1007/978-1-4899-7547-8_5.
[27] Bertrand Iooss, Ron Kenett, and Piercesare Secchi. Different Views of Interpretability. In
InterpretabilityforIndustry4.0: StatisticalandMachineLearningApproaches,pages1–20.Springer
InternationalPublishing,Cham,2022. ISBN978-3-031-12402-0. doi: 10.1007/978-3-031-12402-0_1.
| URL | https://doi.org/10.1007/978-3-031-12402-0_1. |     |     |     |     |     |     |
| --- | -------------------------------------------- | --- | --- | --- | --- | --- | --- |
12

[28] Pascal Iversen, Simon Witzke, Katharina Baum, and Bernhard Y. Renard. Identifying drivers of
predictive aleatoric uncertainty, 2024. URL https://arxiv.org/abs/2312.07252.
[29] WilliamLopezJaramilloandEvgueniSmirnov. Shapley-valuebasedinductiveconformalprediction.
In Lars Carlsson, Zhiyuan Luo, Giovanni Cherubin, and Khuong An Nguyen, editors, Proceedings
|              |     |           |     |           |     |                   |     |            | Applications, | volume |     |
| ------------ | --- | --------- | --- | --------- | --- | ----------------- | --- | ---------- | ------------- | ------ | --- |
| of the Tenth |     | Symposium | on  | Conformal |     | and Probabilistic |     | Prediction | and           |        |     |
152 of Proceedings of Machine Learning Research, pages 52–71. PMLR, 08–10 Sep 2021. URL
https://proceedings.mlr.press/v152/jaramillo21a.html.
[30] Neil Jethani, Mukund Sudarshan, Ian Connick Covert, Su-In Lee, and Rajesh Ranganath. Fast-
| SHAP:        | Real-time | shapley                                      | value | estimation. |     | In            |     |            |             |             |     |
| ------------ | --------- | -------------------------------------------- | ----- | ----------- | --- | ------------- | --- | ---------- | ----------- | ----------- | --- |
|              |           |                                              |       |             |     | International |     | Conference | on Learning | Representa- |     |
| tions, 2022. | URL       | https://openreview.net/forum?id=Zq2G_VTV53T. |       |             |     |               |     |            |             |             |     |
[31] Neil Jethani, Adriel Saporta, and Rajesh Ranganath. Don’t be fooled: label leakage in explanation
methods and the importance of their quantitative evaluation. In Francisco Ruiz, Jennifer Dy, and
| Jan-Willem | van | de Meent, | editors, |     |             |     |          |               |            |               |     |
| ---------- | --- | --------- | -------- | --- | ----------- | --- | -------- | ------------- | ---------- | ------------- | --- |
|            |     |           |          |     | Proceedings | of  | The 26th | International | Conference | on Artificial |     |
Intelligence and Statistics, volume 206 of Proceedings of Machine Learning Research, pages 8925–
8953. PMLR, 25–27 Apr 2023. URL https://proceedings.mlr.press/v206/jethani23a.html.
[32] UlfJohansson,TuweLöfström,HenrikBoström,andCeciliaSönströd. Interpretableandspecialized
conformal predictors. In Alex Gammerman, Vladimir Vovk, Zhiyuan Luo, and Evgueni Smirnov,
editors, Proceedings of the Eighth Symposium on Conformal and Probabilistic Prediction and
Applications, volume 105 of Research, pages 3–22. PMLR, 09–11
|     |     |     |     | Proceedings |     | of Machine | Learning |     |     |     |     |
| --- | --- | --- | --- | ----------- | --- | ---------- | -------- | --- | --- | --- | --- |
Sep 2019. URL https://proceedings.mlr.press/v105/johansson19a.html.
[33] Ulf Johansson, Cecilia Sönströd, Tuwe Löfström, and Henrik Boström. Customized interpretable
| conformal | regressors. |       | In       |      |               |                               |            |     |                 |              |     |
| --------- | ----------- | ----- | -------- | ---- | ------------- | ----------------------------- | ---------- | --- | --------------- | ------------ | --- |
|           |             |       | 2019     | IEEE | International |                               | Conference |     | on Data Science | and Advanced |     |
| Analytics | (DSAA),     | pages | 221–230, |      | 2019.         | doi: 10.1109/DSAA.2019.00037. |            |     |                 |              |     |
[34] Lukas Klein, Carsten Lüth, Udo Schlegel, Till Bungert, Mennatallah El-Assady, and Paul Jäger.
Navigating the maze of explainable ai: A systematic approach to evaluating methods and metrics.
In A. Globerson, L. Mackey, D. Belgrave, A. Fan, U. Paquet, J. Tomczak, and C. Zhang, editors,
Advances in Neural Information Processing Systems, volume 37, pages 67106–67146. Curran As-
| sociates, | Inc., | 2024. URL |     |     |     |     |     |     |     |     |     |
| --------- | ----- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
https://proceedings.neurips.cc/paper_files/paper/2024/file/
7beb1ddb04401ba21869f23a77f3e4e1-Paper-Datasets_and_Benchmarks_Track.pdf.
[35] Jing Lei, Max G’Sell, Alessandro Rinaldo, Ryan J. Tibshirani, and Larry Wasserman.
Distribution-Free Predictive Inference for Regression. Journal of the American Statistical Asso-
ciation, 113(523):1094–1111, July 2018. ISSN 0162-1459. doi: 10.1080/01621459.2017.1307116.
URL https://doi.org/10.1080/01621459.2017.1307116. Publisher: ASA Website _eprint:
https://doi.org/10.1080/01621459.2017.1307116.
[36] Dan Ley, Umang Bhatt, and Adrian Weller. Diverse, global and amortised counterfactual explana-
| tions for  | uncertainty |            | estimates.                |             |     |        |      |            |               | Intelligence, | 36: |
| ---------- | ----------- | ---------- | ------------------------- | ----------- | --- | ------ | ---- | ---------- | ------------- | ------------- | --- |
|            |             |            |                           | Proceedings |     | of the | AAAI | Conference | on Artificial |               |     |
| 7390–7398, | 06          | 2022. doi: | 10.1609/aaai.v36i7.20702. |             |     |        |      |            |               |               |     |
[37] Genyuan Li, Carey Rosenthal, and Herschel Rabitz. High Dimensional Model Representations.
|             |     |          |           |     | A,105(33):7765–7777,August2001. |     |     |     | ISSN1089-5639,1520-5215. |     |     |
| ----------- | --- | -------- | --------- | --- | ------------------------------- | --- | --- | --- | ------------------------ | --- | --- |
| The Journal | of  | Physical | Chemistry |     |                                 |     |     |     |                          |     |     |
doi: 10.1021/jp010450t. URL https://pubs.acs.org/doi/10.1021/jp010450t.
[38] Helena Löfström, Tuwe Löfström, Ulf Johansson, and Cecilia Sönströd. Calibrated explanations:
Withuncertaintyinformationandcounterfactuals.ExpertSyst.Appl.,246(C),July2024.ISSN0957-
4174. doi: 10.1016/j.eswa.2024.123154. URL https://doi.org/10.1016/j.eswa.2024.123154.
13

[39] Tuwe Löfström, Helena Löfström, and Ulf Johansson. Calibrated explanations for multi-class. In
Simone Vantini, Matteo Fontana, Aldo Solari, Henrik Boström, and Lars Carlsson, editors,
Pro-
ceedings of the Thirteenth Symposium on Conformal and Probabilistic Prediction with Applications,
| volume | 230 of      |            |          | Research, |     | pages 175–194. | PMLR, | 09–11 | Sep 2024. |
| ------ | ----------- | ---------- | -------- | --------- | --- | -------------- | ----- | ----- | --------- |
|        | Proceedings | of Machine | Learning |           |     |                |       |       |           |
URL https://proceedings.mlr.press/v230/lofstrom24a.html.
[40] Scott M Lundberg and Su-In Lee. A unified approach to interpreting model predictions. In
I. Guyon, U. Von Luxburg, S. Bengio, H. Wallach, R. Fergus, S. Vishwanathan, and R. Gar-
| nett, editors, |          |           |             |     |            | Systems, | volume | 30. Curran | Asso- |
| -------------- | -------- | --------- | ----------- | --- | ---------- | -------- | ------ | ---------- | ----- |
|                | Advances | in Neural | Information |     | Processing |          |        |            |       |
ciates, Inc., 2017. URL https://proceedings.neurips.cc/paper_files/paper/2017/file/
8a20a8621978632d76c43dfd28b67767-Paper.pdf.
[41] Scott M. Lundberg, Gabriel Erion, Hugh Chen, Alex DeGrave, Jordan M. Prutkin, Bala Nair,
RonitKatz, JonathanHimmelfarb, NishaBansal, andSu-InLee. Fromlocalexplanationstoglobal
understanding with explainable ai for trees. Nature Machine Intelligence, 2(1):56–67, 2020. doi:
10.1038/s42256-019-0138-9.
[42] Nijat Mehdiyev, Maxim Majlatow, and Peter Fettke. Quantifying and explaining machine learning
uncertainty in predictive process monitoring: an operations research perspective.
|     |     |     |     |     |     |     |     | Annals | of  |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | --- |
Operations Research, April 2024. ISSN 1572-9338. doi: 10.1007/s10479-024-05943-4. URL
https://doi.org/10.1007/s10479-024-05943-4.
[43] Nijat Mehdiyev, Maxim Majlatow, and Peter Fettke. Integrating permutation feature importance
with conformal prediction for robust explainable artificial intelligence in predictive process mon-
itoring. Engineering Applications of Artificial Intelligence, 149:110363, 2025. ISSN 0952-1976.
| doi: https://doi.org/10.1016/j.engappai.2025.110363. |     |     |     |     | URL |     |     |     |     |
| ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
https://www.sciencedirect.com/
science/article/pii/S095219762500363X.
[44] Hyejung Moon, Angela M. Dean, and Thomas J. Santner and. Two-stage sensitivity-based group
screening in computer experiments. Technometrics, 54(4):376–387, 2012. doi: 10.1080/00401706.
| 2012.725994. | URL https://doi.org/10.1080/00401706.2012.725994. |     |     |     |     |     |     |     |     |
| ------------ | ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
[45] Maximilian Muschalik, Hubert Baniecki, Fabian Fumagalli, Patrick Kolpaczki, Barbara Hammer,
and Eyke Hüllermeier. shapiq: Shapley interactions for machine learning. In A. Globerson,
L. Mackey, D. Belgrave, A. Fan, U. Paquet, J. Tomczak, and C. Zhang, editors, Advances
|           |             |            | Systems, | volume | 37, | pages | 130324–130357. | Curran | Asso- |
| --------- | ----------- | ---------- | -------- | ------ | --- | ----- | -------------- | ------ | ----- |
| in Neural | Information | Processing |          |        |     |       |                |        |       |
ciates, Inc., 2024. URL https://proceedings.neurips.cc/paper_files/paper/2024/file/
eb3a9313405e2d4175a5a3cfcd49999b-Paper-Datasets_and_Benchmarks_Track.pdf.
[46] Sara Narteni, Alberto Carlevaro, Fabrizio Dabbene, Marco Muselli, and Maurizio Mongelli.
Confiderai: Conformal interpretable-by-design score function for explainable and reliable artificial
intelligence. In Harris Papadopoulos, Khuong An Nguyen, Henrik Boström, and Lars Carlsson,
editors, Proceedings of the Twelfth Symposium on Conformal and Probabilistic Prediction with
| Applications, | volume | 204 of      |     |            |          | Research, | pages | 485–487. | PMLR, |
| ------------- | ------ | ----------- | --- | ---------- | -------- | --------- | ----- | -------- | ----- |
|               |        | Proceedings |     | of Machine | Learning |           |       |          |       |
13–15 Sep 2023. URL https://proceedings.mlr.press/v204/narteni23a.html.
[47] Martin J. Osborne and Ariel Rubinstein. A course in game theory. MIT Press, Cambridge, Mass,
| 1994. ISBN | 978-0-262-15041-5 | 978-0-262-65040-3. |     |     |     |     |     |     |     |
| ---------- | ----------------- | ------------------ | --- | --- | --- | --- | --- | --- | --- |
[48] ArtB.Owen. Sobol’IndicesandShapleyValue. SIAM/ASAJournalonUncertaintyQuantification,
| 2(1):245–251, | January | 2014. ISSN | 2166-2525. | doi: | 10.1137/130936233. |     | URL |     |     |
| ------------- | ------- | ---------- | ---------- | ---- | ------------------ | --- | --- | --- | --- |
http://epubs.siam.
org/doi/10.1137/130936233.
[49] Frederik Pahde, Thomas Wiegand, Sebastian Lapuschkin, and Wojciech Samek. Ensuring medical
aisafety: Explainableai-drivendetectionandmitigationofspuriousmodelbehaviorandassociated
| data, 2025. | URL https://arxiv.org/abs/2501.13818. |     |     |     |     |     |     |     |     |
| ----------- | ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
14

[50] Iker Perez, Piotr Skalski, Alec Barns-Graham, Jason Wong, and David Sutton. Attribution
of predictive uncertainties in classification models. In James Cussens and Kun Zhang, editors,
Proceedings of the Thirty-Eighth Conference on Uncertainty in Artificial Intelligence, volume 180
|     | of          |            |          | Research, | pages | 1582–1591. | PMLR, | 01–05 Aug 2022. | URL |
| --- | ----------- | ---------- | -------- | --------- | ----- | ---------- | ----- | --------------- | --- |
|     | Proceedings | of Machine | Learning |           |       |            |       |                 |     |
https://proceedings.mlr.press/v180/perez22a.html.
[51] Herschel Rabitz, Ömer F. Aliş, Jeffrey Shorter, and Kyurhee Shim. Efficient input—output model
representations. Computer Physics Communications, 117(1-2):11–20, March 1999. ISSN 00104655.
|     | doi: 10.1016/S0010-4655(98)00152-0. |     |     | URL |     |     |     |     |     |
| --- | ----------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
https://linkinghub.elsevier.com/retrieve/pii/
S0010465598001520.
[52] Prashant Rana. Physicochemical properties of protein tertiary structure. UCI Machine Learning
|     | Repository, | 2013. DOI: | https://doi.org/10.24432/C5QW3H. |     |     |     |     |     |     |
| --- | ----------- | ---------- | -------------------------------- | --- | --- | --- | --- | --- | --- |
[53] Saman Razavi, Anthony Jakeman, Andrea Saltelli, Clémentine Prieur, Bertrand Iooss, Emanuele
Borgonovo,ElmarPlischke,SamueleLoPiano,TakuyaIwanaga,WilliamBecker,StefanoTarantola,
Joseph H.A. Guillaume, John Jakeman, Hoshin Gupta, Nicola Melillo, Giovanni Rabitti, Vincent
Chabridon, Qingyun Duan, Xifu Sun, Stefán Smith, Razi Sheikholeslami, Nasim Hosseini, Masoud
Asadzadeh,ArnaldPuy,SergeiKucherenko,andHolgerR.Maier.TheFutureofSensitivityAnalysis:
An essential discipline for systems modeling and policy support. Environmental Modelling &
Software, 137:104954, March 2021. ISSN 13648152. doi: 10.1016/j.envsoft.2020.104954. URL
https://linkinghub.elsevier.com/retrieve/pii/S1364815220310112.
[54] Michael Redmond. Communities and crime. UCI Machine Learning Repository, 2002. DOI:
https://doi.org/10.24432/C53W3X.
| [55] | Christian    | P. Robert        | and George | Casella.    |             |                  |         |                 |     |
| ---- | ------------ | ---------------- | ---------- | ----------- | ----------- | ---------------- | ------- | --------------- | --- |
|      |              |                  |            |             | Monte Carlo | Statistical      | Methods | (Springer Texts | in  |
|      | Statistics). | Springer-Verlag, | Berlin,    | Heidelberg, | 2005.       | ISBN 0387212396. |         |                 |     |
[56] Yaniv Romano, Evan Patterson, and Emmanuel Candes. Conformalized Quantile Regres-
sion. In Advances in Neural Information Processing Systems, volume 32. Curran Asso-
|     | ciates, | Inc., 2019. | URL |     |     |     |     |     |     |
| --- | ------- | ----------- | --- | --- | --- | --- | --- | --- | --- |
https://proceedings.neurips.cc/paper_files/paper/2019/hash/
5103c3584b063c431bd1268e9b5e76fb-Abstract.html.
[57] Gian Carlo Rota. On the foundations of combinatorial theory I. Theory of Möbius Functions.
Zeitschrift für Wahrscheinlichkeitstheorie und Verwandte Gebiete, 2(4):340–368, 1964. ISSN
|     | 0044-3719, | 1432-2064. | doi: 10.1007/BF00531932. |     | URL |     |     |     |     |
| --- | ---------- | ---------- | ------------------------ | --- | --- | --- | --- | --- | --- |
http://link.springer.com/10.1007/
BF00531932.
[58] Glenn Shafer and Vladimir Vovk. A tutorial on conformal prediction. J. Mach. Learn. Res., 9:
|     | 371–421, | June 2008. | ISSN 1532-4435. |     |     |     |     |     |     |
| --- | -------- | ---------- | --------------- | --- | --- | --- | --- | --- | --- |
[59] Lloyd S. Shapley. Notes on the n-Person Game – II: The Value of an n-Person Game. Researach
Memorandum ATI 210720, RAND Corporation, Santa Monica, California, August 1951. URL
https://www.rand.org/content/dam/rand/pubs/research_memoranda/2008/RM670.pdf.
[60] Lloyd S. Shapley. Games, pages 307–318. Princeton University Press,
|     |     | 17. | A Value for | n-Person |     |     |     |     |     |
| --- | --- | --- | ----------- | -------- | --- | --- | --- | --- | --- |
Princeton, 1953. ISBN 9781400881970. doi: doi:10.1515/9781400881970-018. URL https:
//doi.org/10.1515/9781400881970-018.
[61] Geethen Singh, Glenn Moncrieff, Zander Venter, Kerry Cawse-Nicholson, Jasper Slingsby, and
Tamara B. Robinson. Uncertainty quantification for probabilistic machine learning in earth
observation using conformal prediction. Scientific Reports, 14(1):16166, 2024. ISSN 2045-2322.
doi: 10.1038/s41598-024-65954-w. URL https://doi.org/10.1038/s41598-024-65954-w.
[62] Kamaljot Singh. Facebook comment volume. UCI Machine Learning Repository, 2015. DOI:
https://doi.org/10.24432/C5Q886.
15

[63] I.M. Sobol’ and Yu.L. Levitan. On the use of variance reducing multipliers in monte carlo
computations of a global sensitivity index. Computer Physics Communications, 117(1):52–61,
1999. ISSN 0010-4655. doi: https://doi.org/10.1016/S0010-4655(98)00156-8. URL https://www.
sciencedirect.com/science/article/pii/S0010465598001568.
[64] Eunhye Song, Barry L. Nelson, and Jeremy Staum. Shapley Effects for Global Sensitivity Analysis:
Theory and Computation. SIAM/ASA Journal on Uncertainty Quantification, 4(1):1060–1083,
January 2016. ISSN 2166-2525. doi: 10.1137/15M1048070. URL http://epubs.siam.org/doi/
10.1137/15M1048070.
[65] Erik Štrumbelj and Igor Kononenko. An efficient explanation of individual classifications using
game theory. Journal of Machine Learning Research, 11(1):1–18, 2010. URL http://jmlr.org/
papers/v11/strumbelj10a.html.
[66] Mukund Sundararajan and Amir Najmi. The Many Shapley Values for Model Explanation. In
Proceedings of the 37th International Conference on Machine Learning, pages 9269–9278. PMLR,
November 2020. URL https://proceedings.mlr.press/v119/sundararajan20b.html. ISSN:
2640-3498.
[67] Sonja Surjanovic and Derek Bingham. Virtual library of simulation experiments: Test functions
and datasets. Retrieved April 16, 2025, from http://www.sfu.ca/~ssurjano, 2025.
[68] Valeri Vasil’ev and Gerard van der Laan. The Harsanyi Set for Cooperative TU-Games. Working
Paper 01-004/1, Tinbergen Institute Discussion Paper, 2001. URL https://www.econstor.eu/
handle/10419/85790.
[69] Vladimir Vovk, Alexander Gammerman, and Glenn Shafer. Algorithmic Learning in a Ran-
dom World. Springer International Publishing, Cham, 2022. ISBN 978-3-031-06648-1 978-3-
031-06649-8. doi: 10.1007/978-3-031-06649-8. URL https://link.springer.com/10.1007/
978-3-031-06649-8.
[70] Hanjing Wang, Bashirul Azam Biswas, and Qiang Ji. Optimization-based uncertainty attribution
via learning informative perturbations. In ECCV (78), pages 237–253, 2024. URL https:
//doi.org/10.1007/978-3-031-73229-4_14.
[71] David Watson, Joshua O’Hara, Niek Tax, Richard Mudd, and Ido Guy. Explaining predictive
uncertainty with information theoretic shapley values. In Thirty-seventh Conference on Neural
Information Processing Systems, 2023. URL https://openreview.net/forum?id=6rabAZhCRS.
[72] Robert James Weber. Probabilistic values for games, page 101–120. Cambridge University Press,
1988.
[73] Ran Xie, Rina Foygel Barber, and Emmanuel Candes. Boosted conformal prediction intervals.
In The Thirty-eighth Annual Conference on Neural Information Processing Systems, 2024. URL
https://openreview.net/forum?id=Tw032H2onS.
[74] XiXin,GilesHooker,andFeiHuang. Whyyoushouldnottrustinterpretationsinmachinelearning:
Adversarialattacksonpartialdependenceplots,2024. URLhttps://arxiv.org/abs/2404.18702.
[75] Fatima Rabia Yapicioglu, Alessandra Stramiglio, and Fabio Vitali. ConformaSight: Conformal
prediction-based global and model-agnostic explainability framework. In Luca Longo, Sebastian
Lapuschkin, and Christin Seifert, editors, Explainable Artificial Intelligence, pages 270–293, Cham,
2024. Springer Nature Switzerland. ISBN 978-3-031-63800-8. doi: 10.1007/978-3-031-63800-8_14.
[76] I-Cheng Yeh. Concrete compressive strength. UCI Machine Learning Repository, 1998. DOI:
https://doi.org/10.24432/C5PK67.
16

[77] Margaux Zaffran. Post-hoc predictive uncertainty quantification: methods with applications to
|                   | forecasting. | phdthesis, Institut | Polytechnique | de Paris, June | 2024. URL |
| ----------------- | ------------ | ------------------- | ------------- | -------------- | --------- |
| electricity price |              |                     |               |                | https:    |
//theses.hal.science/tel-04720002.
[78] Erik Štrumbelj and Igor Kononenko. Explaining prediction models and individual predictions
with feature contributions. Systems, 41(3):647–665, December 2014.
|     |     | Knowledge and | Information |     |     |
| --- | --- | ------------- | ----------- | --- | --- |
ISSN 0219-3116. doi: 10.1007/s10115-013-0679-x. URL https://link.springer.com/article/
10.1007/s10115-013-0679-x. Company: Springer Distributor: Springer Institution: Springer
| Label: Springer | Number: | 3 Publisher: Springer | London. |     |     |
| --------------- | ------- | --------------------- | ------- | --- | --- |
17

Appendix A. Sets of Allocations and Monte Carlo Approximations
In this appendix, cooperative games are understood as a tuple where is a finite set of
|     |     |     |     |     |     |     |     |     | (D,v) |     | D   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- |
players, and v : P → R is a set function called the value function of the cooperative game. An
D
| allocation | is  | a set function |     | ϕ :D →R | related | to  | a cooperative |     | game (D,v). |     |     |     |
| ---------- | --- | -------------- | --- | ------- | ------- | --- | ------------- | --- | ----------- | --- | --- | --- |
v
Appendix A.1. Equivalent Formulations of the Shapley Values and Corresponding Sets of Allocations
The Shapley values Shap :P →R of a cooperative game (D,v) are a very well-known allocation.
|               |     |          |      | v D            |            |      |          |     |              |     |     |     |
| ------------- | --- | -------- | ---- | -------------- | ---------- | ---- | -------- | --- | ------------ | --- | --- | --- |
| It has become |     | popular  | as   | the only       | allocation | that | respects | the | four axioms: |     |     |     |
| • Efficiency: |     | (cid:80) | Shap | (i)=v(D)−v(∅); |            |      |          |     |              |     |     |     |
v
i∈D
• Symmetry: for ∈D, if for every A∈P , v(A∪{i})=v(A∪{i}), then Shap (i)=Shap (j);
|     |     |     | i,j |     |     |     | D   |     |     |     | v   | v   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
• Sensitivity (or Dummy/Null player): for i ∈ D, if for every A ∈ P , v(A∪{i}) = v(A), then
D
| Shap | (i)=0; |     |     |     |     |     |     |     |     |     |     |     |
| ---- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
v
• Linearity (or Additivity): for two cooperative games and (D,v′), Shap Shap
|      |     |     |     |     |     |     |     | (D,v) |     |     | =    | +   |
| ---- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | ---- | --- |
|      |     |     |     |     |     |     |     |       |     |     | v+v′ | v   |
| Shap | .   |     |     |     |     |     |     |       |     |     |      |     |
v′
Shapley found that the resulting allocation admits an analytical formula. The original formulation
writes [59]
(cid:88) |A|!(d−|A|−1)!
|     |     | ∀j  | ∈D, Shap | (j)= |      |        |     |     | [v(A∪{j})−v(A)]. |     |     | (A.1) |
| --- | --- | --- | -------- | ---- | ---- | ------ | --- | --- | ---------------- | --- | --- | ----- |
|     |     |     |          | v    |      |        |     | d!  |                  |     |     |       |
|     |     |     |          |      | A∈PD | : j∈/A |     |     |                  |     |     |       |
Remark (Shapley values and model explanations). The Shapley values have been extensively used in
the XAI literature for “model explanations” [78, 40]. For a model f(cid:98):X →Y and an observation x∈X,
it amounts to choosing a value function such that (see e.g., [66] for an overview). It has
v(D)=f(cid:98)(x)
become particularly popular since, for such value functions, thanks to the efficiency property, it allows
writing
|     |     |     |     |                |     | (cid:104)    | (cid:105) (cid:88) |      |      |     |     |     |
| --- | --- | --- | --- | -------------- | --- | ------------ | ------------------ | ---- | ---- | --- | --- | --- |
|     |     |     |     | f(cid:98)(x)−E |     | f(cid:98)(x) | =                  | Shap | (j), |     |     |     |
v
j∈D
where the summands are interpreted as parts of the prediction attributed to each feature of the learned
model.
With advances in the theoretical study of cooperative games, different sets of efficient allocations
have been introduced. For the purposes of this article, two of them are introduced: the Weber and
Harsanyi sets. They are particularly interesting since it has been shown that the Shapley values
are part of these sets of allocations, resulting in formulas equivalent to Eq. (A.1), allowing for finer
interpretations of the allocations, going beyond the axiomatic characterizations.
Shapley Values as Uniform Choice Over Random Orders: the Weber Set. The Weber set of allocations
of a cooperative game can be understood from a random order perspective. Let S be the set
|     |     |     | (D,v) |     |     |     |     |     |     |     | D   |     |
| --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
of permutations of (without replacement). For an ordering and for a any
|     |     |     | D   |     |     |     |     |     | π = (π ,...,π | )   | ∈ S |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- | --- | --- |
|     |     |     |     |     |     |     |     |     | 1             | d   | D   |     |
player j ∈D, let π(j) be the position of player j in the ordering π (i.e., π =j), and let πj be the
π(j)
set of players preceding in π, including (i.e., πj :π(j)≤π(i)}). Let be a
|     |     |     | j   |     |     | j   | :={i∈D |     |     |     | p:S D →[0,1] |     |
| --- | --- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | ------------ | --- |
probability mass function over S , called the random order distribution. The Weber set [72] is the set
D
| of allocations, |     | parametrized |        | by p, that | can     | be written        |         | as             |                           |         |                |       |
| --------------- | --- | ------------ | ------ | ---------- | ------- | ----------------- | ------- | -------------- | ------------------------- | ------- | -------------- | ----- |
|                 |     |              |        | (cid:88)   | (cid:2) | (cid:0) πj(cid:1) | (cid:0) | (cid:1)(cid:3) | (cid:2) (cid:0) πj(cid:1) | (cid:0) | (cid:1)(cid:3) |       |
|                 | ∀j  | ∈D,          | ϕ (j)= | p(π)       | v       | −v                | πj \{j} |                | =E v −v                   | πj      | \{j} ,         | (A.2) |
|                 |     |              | v      |            |         |                   |         |                | p                         |         |                |       |
π∈SD
which can be interpreted as the expectation w.r.t. of the marginal contributions of each player in all
p
| the possible |     | orderings. |     |     |     |     |     |     |     |     |     |     |
| ------------ | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
18

Proposition Appendix A.1 (Efficiency of the Weber set). Allocations of the form of Eq. (A.2) are
efficient.
Even though Proposition Appendix A.1 is a very well-known fact in the theory of cooperative games
[72], we provide a proof for completeness’ sake.
Proof of Proposition Appendix A.1. First,wenoticethat,foranyπ ∈S ,(cid:80) (cid:2) v (cid:0) πj(cid:1) −v (cid:0) πj \{j} (cid:1)(cid:3)
is a telescoping series, which is equal to
D j∈PD
v({π })−v(∅)+v({π ,π })−v({π })+···+v({π ,...,π })−v({π ,...,π })=v(D)−v(∅).
1 1 2 1 1 d 1 d−1
Thus,
 
(cid:88) (cid:88) p(π) (cid:2) v (cid:0) πj(cid:1) −v (cid:0) πj \{j} (cid:1)(cid:3) = (cid:88) p(π) (cid:88) (cid:2) v (cid:0) πj(cid:1) −v (cid:0) πj \{j} (cid:1)(cid:3) 
j∈PDπ∈SD π∈SD j∈PD
(cid:88)
=(v(D)−v(∅)) p(π)=v(D)−v(∅)
π∈SD
where the last equality comes from the fact that p is a probability mass function over S .
D
Different choices of random order distributions lead to different allocations. In particular, choosing
p as a discrete uniform distribution over S (i.e., p(π) = 1/d!, ∀π ∈ S ), denoted U(S ), coincides
D D D
with the Shapley values of the game (D,v), leading to the different, but equivalent, formulation
∀j ∈D, Shap (j)= 1 (cid:88) (cid:2) v (cid:0) πj(cid:1) −v (cid:0) πj \{j} (cid:1)(cid:3) =E (cid:2) v (cid:0) πj(cid:1) −v (cid:0) πj \{j} (cid:1)(cid:3) . (A.3)
v d! U(SD)
π∈SD
Example (Random order formulation for 3 players). We derive the computation of the Shapley values
according to Eq. (A.3) in the case where D ={1,2,3}. In this case, the 6 permutations are given by
S ={(1,2,3),(1,3,2),(2,3,1),(3,2,1),(2,1,3),(3,1,2)}.
D
For every permutation, the marginal contributions (MC) for each player are displayed in Table A.2.
The average of the marginal contributions of, e.g., player 1 with uniform weights (equal to 1/6 here)
Ordering Marg. Cont. player {1} Marg. Cont. player {2} Marg. Cont. player {3}
(1,2,3) v({1})−v(∅) v({1,2})−v({1}) v({1,2,3})−v({1,2})
(1,3,2) v({1})−v(∅) v({1,2,3})−v({1,3}) v({1,3})−v({1})
(2,3,1) v({1,2,3})−v({2,3}) v({2})−v(∅) v({2,3})−v({2})
(3,2,1) v({1,2,3})−v({2,3}) v({2,3})−v({3}) v({3})−v(∅)
(2,1,3) v({1,2})−v({2}) v({2})−v(∅) v({1,2,3})−v({1,2})
(3,1,2) v({1,3})−v({3}) v({1,2,3})−v({2,3}) v({3})−v(∅)
TableA.2: Marginalcontributionsofeachplayerforeverypossibleorderingof3players.
leads to
v({1})−v(∅) v({1,2,3})−v({2,3})
Shap (1)=2× +2×
v 6 6
v({1,2})−v({1}) v({1,3})−v({3})
+ +
6 6
which ends up being equivalent to the original formulation above. In general, with three players, one
has that, by letting D ={i,j,k},
v({j})−v(∅)+v(D)−v(D\{j}) v({j,k})−v({k})+v({j,i})−v({i})
Shap (j)=2× + ,
v 6 6
which remains equivalent to the original formula for the Shapley values.
19

Shapley Values as an Egalitarian Redistribution of Dividends: the Harsanyi Set. The allocations in the
Harsanyi set, as presented in Section 2.2, are efficient for any choice of valid weight system.
|             |          |     | (Efficiency |     | of  | the Harsanyi |     | set).       |        |          |         |
| ----------- | -------- | --- | ----------- | --- | --- | ------------ | --- | ----------- | ------ | -------- | ------- |
| Proposition | Appendix |     | A.2         |     |     |              |     | Allocations | in the | Harsanyi | set are |
efficient.
The efficiency of allocations in the Harsanyi set has been cataloged in the theory of cooperative
games (see e.g., [68]). However, for completeness’ sake, we provide a proof of this fact.
Proof of Proposition Appendix A.2. Recall that the Harsanyi dividends are defined as
|     |     |     |      |     |            | (cid:88) | (−1)|A|−|B|v(B), |     |     |     |     |
| --- | --- | --- | ---- | --- | ---------- | -------- | ---------------- | --- | --- | --- | --- |
|     |     |     | ∀A∈P | D   | , φ v (A)= |          |                  |     |     |     |     |
B∈PA
and from Rota’s generalization of the Möbius inversion formula on powersets [57], it implies that:
(cid:88)
|     |     |     |     | ∀A∈P | D , | v(A)= | φ   | v (A). |     |     |     |
| --- | --- | --- | --- | ---- | --- | ----- | --- | ------ | --- | --- | --- |
B∈PA
Notice that, by definition (∅)=v(∅), and thus, in particular, for in the previous equation, we
|     |     | φ   |     |     |     |     |     | A=D |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
v
have that
|     |     | (cid:88) |              |     |     | (cid:88) |     |                |     |     |     |
| --- | --- | -------- | ------------ | --- | --- | -------- | --- | -------------- | --- | --- | --- |
|     |     |          | φ v (A)=v(D) |     | ⇐⇒  |          | φ v | (A)=v(D)−v(∅). |     |     |     |
A∈PD
A∈PD:A̸=∅
| Next, notice | that: |          |     |          |        |          |     |      |          |     |     |
| ------------ | ----- | -------- | --- | -------- | ------ | -------- | --- | ---- | -------- | --- | --- |
|              |       | (cid:88) |     |          |        | (cid:88) |     | (A)1 |          |     |     |
|              |       |          |     | λ j (A)φ | v (A)= |          | λ   | j    | φ v (A). |     |     |
{j∈A}
|     |     | A∈PD | : j∈A |     |     | A∈PD | : A̸=∅ |     |     |     |     |
| --- | --- | ---- | ----- | --- | --- | ---- | ------ | --- | --- | --- | --- |
Thus,
| (cid:88) | (cid:88) |     |     |     | (cid:88) | (cid:88) |     |     |     |     |     |
| -------- | -------- | --- | --- | --- | -------- | -------- | --- | --- | --- | --- | --- |
(A)1
|         |     | λ j | (A)φ v | (A)= |         |      | λ j | {j∈A} φ | v (A) |     |     |
| ------- | --- | --- | ------ | ---- | ------- | ---- | --- | ------- | ----- | --- | --- |
| j∈DA∈PD | :   | j∈A |        |      | j∈DA∈PD | A̸=∅ |     |         |       |     |     |
:
|     |     |     |     |     |          |        |         |         |         |        |     |
| --- | --- | --- | --- | --- | -------- | ------ | -------- | ------- | -------- | ------ | --- |
|     |     |     |     |     | (cid:88) |        | (cid:88) |         |          |        |     |
|     |     |     |     | =   |          | φ (A) |          | λ (A)1  |          |        |     |
|     |     |     |     |     |          | v      |          | j       | {j∈A}   |        |     |
|     |     |     |     |     | A∈PD :   | A̸=∅   | j∈D      |         |          |        |     |
|     |     |     |     |     |          |        |         |         |         |        |     |
|     |     |     |     |     | (cid:88) |        | (cid:88) |         | (cid:88) |        |     |
|     |     |     |     | =   |          | φ (A) |          | λ (A)= |          | φ (A), |     |
|     |     |     |     |     |          | v      |          | j       |          | v      |     |
|     |     |     |     |     | A∈PD :   | A̸=∅   | j∈D      |         | A∈PD :   | A̸=∅   |     |
(cid:80)
| since the weight | system | must | respect |     | λ   | (A)=1. | Thus, |     |     |     |     |
| ---------------- | ------ | ---- | ------- | --- | --- | ------ | ----- | --- | --- | --- | --- |
j∈D j
|                |          |     | (cid:88) | (cid:88) |       |                     |     |     |     |     |     |
| -------------- | -------- | --- | -------- | -------- | ----- | ------------------- | --- | --- | --- | --- | --- |
|                |          |     |          |          | λ     | (A)φ (A)=v(D)−v(∅), |     |     |     |     |     |
|                |          |     |          |          | j     | v                   |     |     |     |     |     |
|                |          |     | j∈DA∈PD  |          | : j∈A |                     |     |     |     |     |     |
| and efficiency | follows. |     |          |          |       |                     |     |     |     |     |     |
Differentchoicesofweightsystemsleadtodifferentallocations. Inparticular,choosingtheegalitarian
redistribution of the dividends [22] (i.e., ∀j ∈D, ∀A∈P (A)=1/|A|) coincides with the Shapley
|     |     |     |     |     |     |     | D , λ | j   |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- |
values of (D,v), resulting in the different, but equivalent formulation of the Shapley values
|     |     |     |     |     |           |     | (cid:88) | φ (A) |     |     |       |
| --- | --- | --- | --- | --- | --------- | --- | -------- | ----- | --- | --- | ----- |
|     |     |     | ∀j  | ∈D, | Shap (j)= |     |          | v     | .   |     | (A.4) |
v
|A|
A∈PD : j∈A
The egalitarian nature of the redistribution of the dividends can be understood as follows: for any
, is divided in equal parts, which are redistributed to each of the players in A.
| A∈P φ(A) |     |     | |A| |     |     |     |     |     |     |     |     |
| -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
D
20

Coalition Harsanyi dividend λ λ λ
1 2 3
∅ v 0 0 0
∅
{1} v −v 1 0 0
1 ∅
{2} v −v 0 1 0
2 ∅
{3} v −v 0 0 1
3 ∅
{1,2} v −v −v +v 1/2 1/2 0
12 1 2 ∅
{1,3} v −v −v +v 1/2 0 1/2
13 1 3 ∅
{2,3} v −v −v +v 0 1/2 1/2
23 2 3 ∅
{1,2,3} v −v −v −v +v +v +v −v 1/3 1/3 1/3
123 12 13 23 1 2 3 ∅
TableA.3: HarsanyidividendsandweightsystemfortheShapleyvaluesofagamewith3players.
Example (Dividend-sharing formulation for 3 players). We derive the computation of the Shapley
values according to Eq. (A.4) in the case where D ={1,2,3}. The dividends are given in Table A.4.
Thus, for j =1, the reweighting of the dividends leads to:
(cid:18) (cid:19) (cid:18) (cid:19) (cid:18) (cid:19)
1 1 1 1 1 1 1 1
Shap (j)=v(∅) −1+ + − +v({1}) 1− − + +v({2}) − +
v 2 2 3 2 2 3 2 3
(cid:18) (cid:19) (cid:18) (cid:19) (cid:18) (cid:19)
1 1 1 1 1 1
+v({3}) − + +v({1,2}) − +v({1,3}) −
2 3 2 3 2 3
(cid:18) (cid:19)
1 v({1,2,3})
+v({2,3}) − +
3 3
v({1})−v(∅) v({1,2,3})−v({2,3}) v({1,2})−v({2}) v({1,3})−v({3})
= + + +
3 3 6 6
and we recover the original formula for the Shapley values.
General Relationship Between the Weber and Harsanyi Set. The Weber set of allocations is a subset of
the Harsanyi set. Thus, for a given random order distribution p(π), the corresponding weight system is
given by (see [16])
(cid:88)
∀j ∈D, ∀A∈P , λ (A)= p(π), (A.5)
D j
π∈SD : A∈πi
such that the resulting allocations are equivalent. Moreover, it is possible to find the corresponding
random order distribution for certain weight systems. More precisely, [17] (Theorem 5.5 and Corollary
5.7) showed that the Weber set coincides with the Harsanyi set restricted to weight systems satisfying:
(cid:88)
∀A∈P , ∀j ∈A, (−1)|A|−|B|λ (B)≥0,
D j
B∈PD
and consequently, for such weight systems, it is also possible to write their corresponding random order
distribution.
Appendix A.2. Faster Computations: Sampling Random Orders
Exact estimators of allocations from either the Weber or the Harsanyi set require estimating the
value function v on all the 2d−1 coalitions of players. This quickly becomes intractable in practice
for moderate to large numbers of features. We propose a Monte Carlo-type sampling strategy over
permutations of D to accommodate our methodological contributions, providing unbiased, consistent,
and approximately normal estimates for any allocation in the Weber set (provided p(π) is known).
First, let us recall the contributions of [78, 64] for the Shapley values. The authors leveraged Eq. (A.3)
by noticing that it is an expectation over uniformly distributed permutations. Hence, by uniformly
21

sampling a subset ⊆S of |Π |=m≪d! permutations, an approximation of this expectation is
|     |     |     | Π m | D   | m   |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
given by
|     |     |     |     |     | S(cid:92)hap | 1    | (cid:88) (cid:2) | (cid:0) πj(cid:1) | (cid:0) | (cid:1)(cid:3) |     |
| --- | --- | --- | --- | --- | ------------ | ---- | ---------------- | ----------------- | ------- | -------------- | --- |
|     |     |     | ∀j  | ∈D, |              | (j)= | v                | −v                | πj \{j} | ,              |     |
|     |     |     |     |     | v            | m    |                  |                   |         |                |     |
π∈Πm
and unbiasedness, consistency, and asymptotic normality of this estimator follows from basic Monte
| Carlo | sampling | arguments. |     |     |     |     |     |     |     |     |     |
| ----- | -------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Now, the same rationale can be applied for an arbitrary, but known, probability mass function p(π).
ItrequiresrandomlysamplingΠ accordingtop(π), andtheapproximationofthecorresponding
⊆S
m D
| Weber | allocation |     | is given by: |     |                 |            |                   |                         |      |                  |       |
| ----- | ---------- | --- | ------------ | --- | --------------- | ---------- | ----------------- | ----------------------- | ---- | ---------------- | ----- |
|       |            |     |              |     |                 | 1 (cid:88) |                   |                         |      |                  |       |
|       |            |     | ∀j           | ∈D, | ϕ(cid:99)v (j)= |            | (cid:2) v (cid:0) | πj(cid:1) −v (cid:0) πj | \{j} | (cid:1)(cid:3) , | (A.6) |
m
π∈Πm
yielding unbiased, consistent, and asymptotically normal estimators. This generalized Monte Carlo
approximation scheme for allocations in the Weber set is illustrated in Algorithm 2.
|           |     | Monte       | Carlo-type      |      | random        | order allocation |     | approximation |     |     |     |
| --------- | --- | ----------- | --------------- | ---- | ------------- | ---------------- | --- | ------------- | --- | --- | --- |
| Algorithm |     | 2           |                 |      |               |                  |     |               |     |     |     |
|           |     | Cooperative | game            |      |               |                  |     |               |     |     |     |
| Require:  |     |             |                 | (D   | ={1,...,d},v) |                  |     |               |     |     |     |
| Require:  |     | Number      | of permutations |      | m≪d!          |                  |     |               |     |     |     |
| Require:  |     | Probability | measure         | p(π) | over          | S                |     |               |     |     |     |
D
|     |     |     |     | (cid:16) |     |     | (cid:17) |     |     |     |     |
| --- | --- | --- | --- | -------- | --- | --- | -------- | --- | --- | --- | --- |
Ensure: Approximation ϕ(cid:99)v = ϕ(cid:99)v (1),...,ϕ(cid:99)v (d) of Weber allocation ϕ with p(π)
v
|     | for k ∈{1,...,m} |     | do  |     |     |     |     |     |     |     |     |
| --- | ---------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
1:
|     | Sample |            | according |          | to          |             |          |     |     |     |     |
| --- | ------ | ---------- | --------- | -------- | ----------- | ----------- | -------- | --- | --- | --- | --- |
| 2:  |        | π          | k ∈S D    |          | p(π)        |             |          |     |     |     |     |
| 3:  | for    | j =1,...,d | do        |          |             |             |          |     |     |     |     |
|     |        |            |           | (cid:16) | πj (cid:17) | (cid:16) πj | (cid:17) |     |     |     |     |
| 4:  |        | Compute    | MVj       | =v       | −v          | \{j}        |          |     |     |     |     |
|     |        |            | πk        |          | k           | k           |          |     |     |     |     |
| 5:  | end    | for        |           |          |             |             |          |     |     |     |     |
end for
6:
| 7:  | for j ∈D | do  |                   |           |      |     |     |     |     |     |     |
| --- | -------- | --- | ----------------- | --------- | ---- | --- | --- | --- | --- | --- | --- |
|     | Compute  |     | ϕ(cid:99)v (j)= 1 | (cid:80)m | MV j |     |     |     |     |     |     |
| 8:  |          |     | m                 | k=1       | π    |     |     |     |     |     |     |
k
9: end for
|          |        |              | (cid:16)                      |          | (cid:17) |         |              |         |        |     |     |
| -------- | ------ | ------------ | ----------------------------- | -------- | -------- | ------- | ------------ | ------- | ------ | --- | --- |
| 10:      | return | ϕ(cid:99)v = | ϕ(cid:99)v (1),...,ϕ(cid:99)v | (d)      |          |         |              |         |        |     |     |
| Appendix | A.3.   | Random       | Order                         | Sampling |          | for the | Proportional | Shapley | Values |     |     |
The proportional Shapley values are part of a broader set of allocations, called the weighted Shapley
values. They are characterized by weight systems having the form λWS(A)= w(i) for some arbitrary
|     |     |     |     |     |     |     |     |     | i   | w(A) |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- | --- |
weight function w :P →R. In [15], the author linked this set of allocations to their random order
D
| expression. |     | The | random order | distribution |     | is then      | given | by:      |              |           |       |
| ----------- | --- | --- | ------------ | ------------ | --- | ------------ | ----- | -------- | ------------ | --------- | ----- |
|             |     |     |              |              |     |              |       |         |              | (cid:33) |       |
|             |     |     |              |              |     |              |       | d        | (cid:32) d−1 |           |       |
|             |     |     |              |              |     |              |       | (cid:88) | (cid:88)     | w(π )     |       |
|             |     | ∀π  | =(π ,...,π   | )∈S          | ,   | WS(π):=exp− |       | log      | 1+           | j .      | (A.7) |
|             |     |     | 1            | d            | D   |              |       |          |              |           |       |
|             |     |     |              |              |     |              |       |          |              | w(π k )   |       |
|             |     |     |              |              |     |              |       | j=2      | k=1          |           |       |
The proportional Shapley values in Eq. (A.7) can be seen as the particular case of weight function
(cid:80) v({j}) for every ∈ P . Hence, the random order distribution for the proportional
| w(A)    | =      |        |          | A   | D   |     |     |     |     |           |     |
| ------- | ------ | ------ | -------- | --- | --- | --- | --- | --- | --- | --------- | --- |
| Shapley | values | j∈A is | given by |     |     |     |     |     |     |           |     |
|         |        |        |          |     |     |    |     |     |     | (cid:33) |     |
(cid:32)
|     |     |     |       |     |            |     | (cid:88) d | (cid:88) d−1 | | v ( π | ) | |       |
| --- | --- | --- | ----- | --- | ---------- | --- | ---------- | ------------ | ------- | --- | ----- |
|     |     |     |       |     | PS(π):=exp |     |            |              | j       |     | (A.8) |
|     |     |     | ∀π ∈S | D , |            | −  | log        | 1+           |         | ,  |       |
|     |     |     |       |     |            |     |            |              | | v ( π | ) | |       |
|     |     |     |       |     |            |     | j=2        | k=1          | k       |     |       |
22

provided the values are positive. In case of zero individual values, the probabilities can be computed
by considering the subgame with v({j})=0}. The interpretation of this result
|     |     |     |     | D\Z | Z   | ={j ∈D | :   |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- |
from [15] can be leveraged to construct a simple sampling scheme w.r.t. this random order distribution.
First, draw from where each has probability (cid:80) v({i}), then draw from
|     |     | π d | D   |     | j ∈D |          |     | v({j})/ |     |     | π d−1 |
| --- | --- | --- | --- | --- | ---- | -------- | --- | ------- | --- | --- | ----- |
|     |     |     |     |     |      | (cid:80) |     |         | i∈D |     |       |
D\π , where j ∈D has probability v({j})/ v({i}) and continue until every element of D
|     | d   |     |     |     |     |     | i∈D\πd |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- |
has been drawn. The resulting permutation will have been drawn with probability as
|           |            |             |              |                       |               | π =(π     | 1 ,...,π         | d )       |          |        |     |
| --------- | ---------- | ----------- | ------------ | --------------------- | ------------- | --------- | ---------------- | --------- | -------- | ------ | --- |
| in Eq.    | (A.8).     | This        | sampling     | scheme                | is further    | detailed  | in               | Algorithm | 3.       |        |     |
|           |            | Drawing     | permutations |                       | according     | to        | the proportional |           | Shapley  | values |     |
| Algorithm |            | 3           |              |                       |               |           |                  |           |          |        |     |
|           |            | Cooperative | game         |                       |               |           |                  |           |          |        |     |
| Require:  |            |             |              | (D                    | ={1,...,d},v) |           |                  |           |          |        |     |
| Require:  |            | Computed    | values       | for v({1}),...,v({d}) |               |           |                  |           |          |        |     |
| Ensure:   |            | Permutation | π =(π        | ,...,π                | ) drawn       | according |                  | to Eq.    | (A.8)    |        |     |
|           |            |             |              | 1                     | d             |           |                  |           |          |        |     |
|           | Initialize | D˜          |              |                       |               |           |                  |           |          |        |     |
| 1:        |            | ←D          |              |                       |               |           |                  |           |          |        |     |
| 2:        | for j      | =0,...,d−1  | do           |                       |               |           |                  |           |          |        |     |
|           | Draw       |             | from D˜      | where                 | each ∈D˜      | has       | probability      | v({k})/   | (cid:80) | v({i}) |     |
| 3:        |            | π d−j       |              |                       | k             |           |                  |           |          | i∈D˜   |     |
|           | Let        | D˜ ←D˜      | \π           |                       |               |           |                  |           |          |        |     |
| 4:        |            |             | d−j          |                       |               |           |                  |           |          |        |     |
5: end for
| 6:       | return | π =(π           | ,...,π | )         |             |     |          |        |     |     |     |
| -------- | ------ | --------------- | ------ | --------- | ----------- | --- | -------- | ------ | --- | --- | --- |
|          |        |                 | 1      | d         |             |     |          |        |     |     |     |
| Appendix |        | A.4. Importance |        | Sampling: | Reweighting |     | Computed | Values |     |     |     |
By applying a reweighting of the summands in Eq. (A.6), it is possible to produce estimates for a
differentrandomorderallocationwithoutre-samplingpermutationsorre-computingthevaluefunctions
on selected coalitions. More precisely, let and p′ be two different random order distributions, related
p
to the random order allocations ϕ and ϕ′. Let Π be a sample of S drawn according to p. Then, it
|     |          |      |     |                |      |            | m            |                      | D               |                |     |
| --- | -------- | ---- | --- | -------------- | ---- | ---------- | ------------ | -------------------- | --------------- | -------------- | --- |
| can | be shown | that |     |                |      |            |              |                      |                 |                |     |
|     |          |      |     |                | IS   | 1 (cid:88) | p′(π)(cid:2) |                      |                 |                |     |
|     |          |      | ∀j  | ∈D, ϕ(cid:99)′ |      |            |              | (cid:0) πj(cid:1) −v | (cid:0) πj \{j} | (cid:1)(cid:3) |     |
|     |          |      |     |                | v := |            | v            |                      |                 | ,              |     |
|     |          |      |     |                | m    |            | p(π)         |                      |                 |                |     |
π∈Πm
is an unbiased, consistent, and asymptotically normal estimate of ϕ′, from basic Monte Carlo-type
arguments. This entails that if a practitioner commits to approximation, e.g., the Shapley values by
drawing random permutations, the computations of the value function on the selected coalitions can
be recycled to approximate, e.g., the proportional Shapley values by reweighting the summands in
| Eq.      | (A.6) | by PS(π)×d!.    |     |          |             |     |     |     |     |     |     |
| -------- | ----- | --------------- | --- | -------- | ----------- | --- | --- | --- | --- | --- | --- |
| Appendix |       | B. Proofs       |     |          |             |     |     |     |     |     |     |
| Appendix |       | B.1. Efficiency |     | property | of CP-based | UA  |     |     |     |     |     |
ThisisadirectconsequenceofPropositionAppendixA.2asboththeShapley
| Proof    | of Proposition |                  | 3.1.      |            |            |               |        |              |     |     |     |
| -------- | -------------- | ---------------- | --------- | ---------- | ---------- | ------------- | ------ | ------------ | --- | --- | --- |
| and      | proportional   | Shapley          |           | values     | are in the | Harsanyi      | set of | allocations. |     |     |     |
| Appendix |                | B.2. Statistical |           | guarantees | of the     | approximation |        | scheme       |     |     |     |
|          |                | (Formal          | statement | of Theorem |            | 1).           |        |              |     |     |     |
Theorem Let p be any probability mass function over the permuta-
tions of S . For a positive integer m, let π ,...,π be an i.i.d. sample drawn from p. For π ∈S , let
|     |     | D   |     |     |     | 1   | m   |     |     |     | D   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
h(π)=v(πj)−v(πj \{j}). Assume that 0<E (cid:2) h(π)2 (cid:3) <∞. Then, for every j ∈D,
p
m
1 (cid:88)
|     |     |     |     |     | ϕ(cid:99)v | (j)= | h(π | i ), |     |     |     |
| --- | --- | --- | --- | --- | ---------- | ---- | --- | ---- | --- | --- | --- |
m
i=1
as computed by Algorithm 2, are unbiased, strongly consistent, and asymptotically normal estimators of
(j):=E
| ϕ   |     | [h(π)]. |     |     |     |     |     |     |     |     |     |
| --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| v   |     | p       |     |     |     |     |     |     |     |     |     |
23

This result specializes to the Shapley values when p=U(S ), and to the proportional Shapley values
D
when p=PS as in Eq. (A.8), corresponds to the distributions associated with the Shapley value or the
| proportional | Shapley |     | value.  |     |           |     |
| ------------ | ------- | --- | ------- | --- | --------- | --- |
| Proof of     | Theorem | 1.  | Let ∈D. | The | estimator |     |
j
m
1 (cid:88)
|     |     |     |     |     | ϕ(cid:99)v (j)= | h(π ), |
| --- | --- | --- | --- | --- | --------------- | ------ |
i
m
i=1
corresponds to the classical Monte Carlo integration estimator of E [h(π)]. Following [55, 65], provided
p
E [|h(π)|]<∞, the strong law of large numbers ensures that is strongly consistent. Moreover,
| p   |     |     |     |     |     | ϕ(cid:99)v (j) |
| --- | --- | --- | --- | --- | --- | -------------- |
assuming V ∞, the central limit Theorem guarantees asymptotic normality. Finally,
|     | 0 < | (f(π)) | <   |     |     |     |
| --- | --- | ------ | --- | --- | --- | --- |
p
| unbiasedness | comes | from | the | linearity | of the expectation | operator. |
| ------------ | ----- | ---- | --- | --------- | ------------------ | --------- |
Theorem (Formal statement of Theorem 2). Let p and p′ be two probability mass functions over
S , and for a positive integer m, let be an i.i.d. sample drawn from p. Assume that
| D                |     |           |          |     | π 1 ,...,π m |     |
| ---------------- | --- | --------- | -------- | --- | ------------ | --- |
| (cid:20)(cid:16) |     | (cid:17)2 | (cid:21) |     |              |     |
p′(π)h(π)
| 0<E |      |     | <∞. | Then, | for every j ∈D, |     |
| --- | ---- | --- | --- | ----- | --------------- | --- |
| p   | p(π) |     |     |       |                 |     |
m
|     |     |     |     |     | IS 1 (cid:88) | p ′ ( π ) |
| --- | --- | --- | --- | --- | ------------- | --------- |
|     |     |     |     |     | ϕ(cid:99)′ =  | i h(π )   |
|     |     |     |     |     | v             | i         |
|     |     |     |     |     | m             | p ( π i ) |
i=1
ϕ′ (j):=E
are unbiased, strongly consistent, and asymptotically normal estimators of p′[h(π)].
v
(cid:92) IS
For every j ∈ D, if p′ = PS and p = U(S ), this results particularizes to P -Shap (j), and if
D v
|          |         |       |                   |     | (cid:91) IS |      |
| -------- | ------- | ----- | ----------------- | --- | ----------- | ---- |
| p′ =U(S  | ) and   | p=PS, | it particularizes |     | to S hap    | (j). |
|          | D       |       |                   |     | v           |      |
| Proof of | Theorem | 2.    | Let ∈D.           | The | estimator   |      |
j
m
|     |     |     |     |     | IS 1 (cid:88) | p ′ ( π ) |
| --- | --- | --- | --- | --- | ------------- | --------- |
|     |     |     |     |     | ϕ(cid:99)′    | i         |
|     |     |     |     |     | v (j)=        | h(π i ),  |
|     |     |     |     |     | m             | p ( π )   |
i=1 i
(cid:104) (cid:105)
corresponds to the classical importance sampling estimator of E p′[h(π)]. Provided E p ′ ( π )h(π) <∞,
p
p ( π )
IS
the strong law of large numbers ensures that ϕ(cid:99)′ is strongly consistent [55]. Moreover, provided
|          |          |     |     |     | v (j) | 0<  |
| -------- | -------- | --- | --- | --- | ----- | --- |
| (cid:16) | (cid:17) |     |     |     |       |     |
V p′(π)h(π) <∞, the central limit theorem guarantees asymptotic normality. Finally, unbiasedness
p
p(π)
| comes from | the  | linearity  | of the        | expectation | operator.   |                    |
| ---------- | ---- | ---------- | ------------- | ----------- | ----------- | ------------------ |
| Appendix   | C.   | Additional |               | Details     | and Results | on the Experiments |
| Appendix   | C.1. | Modified   | Sobol’-Ativan |             | Benchmark   |                    |
Figure C.5 provides an alternative view of the convergence behavior of the Shapley value estimates
using the sampling-based approximation method. For each variable, the figure reports the estimates
obtained across 150 replications for each number of sampled permutations (y-axis). Shapley values
m
are shown along the x-axis. Dots represent the mean estimate across replications, and horizontal bars
indicate ±1 standard deviation. Vertical lines mark the Shapley values computed using the exact
method. As expected, the variance of the estimates—computed over 150 replications for each value of
| sampled  | permutations |            | m—decreases |     | with the number | of permutations. |
| -------- | ------------ | ---------- | ----------- | --- | --------------- | ---------------- |
| Appendix | C.2.         | Real-World | Datasets    |     |                 |                  |
We conduct a series of experiments on real-world datasets to estimate feature-wise allocations
that quantify each variable’s contribution to uncertainty. These allocations are based on Conformal
Prediction (CP) intervals, and the underlying models used to construct the CP intervals vary across
experiments. Depending on the CP method employed, different value functions are defined to reflect
the target quantity (e.g., interval width or predicted bound). We then estimate allocations using either
Shapley values or Proportional Shapley (P-Shapley) values. An overview is reported in Table (C.4).
24

|     | X 1 |     | X 2 |     | X 3 | X 4 |
| --- | --- | --- | --- | --- | --- | --- |
5000
4000
3000
2000
1000
0
.008 .010 .012 .014 .002 .003 .004 .005 .006 .080 .085 .090 .095 .100 -0.001 0.000 0.001
|     | X   |     | X   |     | X   | X   |
| --- | --- | --- | --- | --- | --- | --- |
|     | 5   |     | 6   |     | 7   | 8   |
5000
4000
3000
snoitatumrepforebmuN 2000
1000
0
.000 .003 .006 .009 .004 .006 .008 -0.010 -0.005 0.000 0.005 .160 .165 .170 .175 .180
|     | X   |     | X   |     | X   | X   |
| --- | --- | --- | --- | --- | --- | --- |
|     | 9   |     | 10  |     | 11  | 12  |
5000
4000
3000
2000
1000
0
.075 .080 .085 .090 -0.0025 0.0000 0.0025 .004 .006 .008 .105 .110 .115 .120
|     | X   |     | X   |     | X   | X   |
| --- | --- | --- | --- | --- | --- | --- |
|     | 13  |     | 14  |     | 15  | 16  |
5000
4000
3000
2000
1000
0
.195 .200 .205 .210 .215 .220 .060 .065 .070 .075 .080 .085 .090 .095 .145 .150 .155 .160
ShapleyValue
FigureC.5: EmpiricalconvergenceofMonteCarloShapleyvalueestimates. Foreachnumberofpermutations(y-axis),a
blackdotindicatesthemeanShapleyvalueover150replications,andhorizontalbarsindicate±1standarddeviation.
TheorangeverticallinemarkstheShapleyvaluecomputedwiththeexactestimationprocedure.
| Method | Predictive  | Models | Value    | Functions | Allocation | Methods |
| ------ | ----------- | ------ | -------- | --------- | ---------- | ------- |
| SMR    | LR,         |        | Width,   |           | Shap,      |         |
|        | RF,         |        | Upper    | Bound,    | P-Shap     |         |
|        | LGB         |        | Lower    | Bound,    |            |         |
|        |             |        | Cond.    | Mean      |            |         |
| LACP   | Cond. Mean: | LR,    | Width,   |           | Shap,      |         |
|        | RF,         |        | Upper    | Bound,    | P-Shap     |         |
|        | LGB         |        | Lower    | Bound,    |            |         |
|        | MAD: LGB    | (MAD)  | Cond.    | Mean and  | MAD        |         |
| CQR    | Q-LR,       |        | Interval | Width,    | Shap,      |         |
|        | Q-RF        |        | Upper    | Bound,    | P-Shap     |         |
|        |             |        | Lower    | Bound     |            |         |
TableC.4: Overviewofimplementedmethods,models,valuefunctions,andallocationapproaches.
25

Methods, models, value functions, and allocation approaches. We evaluate our methods on the seven
publicly available datasets described in Table (1). Depending on the number of features, we either
estimate the allocations using the exact method or the approximation method (Table (C.5)).
| Dataset   |                                                |     | Allocation      | # Models Trained |
| --------- | ---------------------------------------------- | --- | --------------- | ---------------- |
|           | n                                              | d   |                 |                  |
| bike      | 17,379                                         | 12  | Exact           | 4,096            |
|           |                                                |     | Approx. (m=50)  |                  |
| blog      | 52,397                                         | 238 |                 | 11,841           |
| casp      | 45,730                                         | 9   | Exact           | 512              |
| concrete  | 1,030                                          | 8   | Exact           | 256              |
|           |                                                |     | Approx. (m=200) |                  |
| facebook  | 79,788                                         | 37  |                 | 6,814            |
| UScrime   | 1,993                                          | 101 | Approx. (m=200) | 19,780           |
|           |                                                |     | Approx. (m=200) |                  |
| star      | 2,161                                          | 38  |                 | 7,220            |
| TableC.5: | Estimationofallocationsandmodelcountbydataset. |     |                 |                  |
Hyperparameter Settings. We rely on the default hyperparameters provided by the corresponding
packages, with a few targeted modifications. For Random Forests and Quantile Random Forests
R
(randomForest, quantregForest), we set the minimum node size to 20% of the training set, limit the
√
number of trees to 75, and define the number of variables considered at each split (mtry) as d. For
LightGBM (lightgbm), we fix the number of boosting rounds to 100, except for the USCrimes, star,
and blog datasets, where it is fixed to 25. In the case of Linear Quantile Regression (quantreg), we
use the Frisch–Newton algorithm for optimization and set the tolerance to the default precision,
R
.Machine$double.eps.
Visualization of Feature-Wise Contribution to Uncertainty. For each dataset, after estimating the
allocations corresponding to a given value function and CP method flavor, we visualize the results
using two complementary representations. The first is a matrix plot with variables as rows and ranks
as columns. The cell indicates the percentage of test-set observations for which the absolute value
[i,j]
of the allocation (Shapley or P-Shapley) for variable i ranks in position j among all variables. The cell
color encodes this percentage: the darker the green, the more frequently variable i appears in rank j
across the test set. The second visualization highlights the most frequently top-ranked variables. For
each rank from 1 to 5 (x-axis), we plot the proportion of test-set observations for which a given variable
appears most often in that position when allocations are ordered by absolute value. The y-axis reflects
the frequency of the dominant variable at each rank. This shows which features consistently contribute
the most to uncertainty. It is important to note that a single variable may appear as the most frequent
at multiple ranks, reflecting consistent influence across several positions in the allocation rankings.
26

| Appendix C.2.1. | bike Dataset |     |     |     |         |     |     |           |     |     |
| --------------- | ------------ | --- | --- | --- | ------- | --- | --- | --------- | --- | --- |
| A               |              |     | B   |     |         |     |     |           |     |     |
| Shapley         | P-Shapley    |     |     |     | Shapley |     |     | P-Shapley |     |     |
LinearReg.
| X12 |     |     |     | 1.00 |     |     |     |     |     |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0.75
0.50
|     |     |     | testsetniycneuqerfknaR | hr  | weekday holiday | workingday yr | hr  | mnth weathersit | temp | yr  |
| --- | --- | --- | ---------------------- | --- | --------------- | ------------- | --- | --------------- | ---- | --- |
0.25
| X1  |     |      |     | 0.00    |            |        |     |        |           |     |
| --- | --- | ---- | --- | ------- | ---------- | ------ | --- | ------ | --------- | --- |
| X12 |     |      |     | 1.00    |            |        |     |        |           |     |
|     |     |      |     | 0.75    |            |        |     |        |           | RF  |
|     |     | 100% |     | 0.50 hr | atemp temp | hum yr | hr  | season | hum atemp | yr  |
0.25
X1
0.00
LightGBM
| X12 |     |     |     | 1.00 |     |     |     |     |     |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0.75
|     |     |     |     | 0.50 hr | yr workingday | atemp temp | hr  | yr  | hum temp | mnth |
| --- | --- | --- | --- | ------- | ------------- | ---------- | --- | --- | -------- | ---- |
0.25
| X1      |           |     |     | 0.00 |         |            |     |           |     |            |
| ------- | --------- | --- | --- | ---- | ------- | ---------- | --- | --------- | --- | ---------- |
| 1       | 12 1 12   |     |     | 1    | 2       | 3 4 5      | 1   | 2         | 3 4 | 5          |
|         | Rank      |     |     |      |         | Rank(top5) |     |           |     |            |
| A       |           |     | B   |      |         |            |     |           |     |            |
| Shapley | P-Shapley |     |     |      | Shapley |            |     | P-Shapley |     |            |
| X12     |           |     |     | 1.00 |         |            |     |           |     | LinearReg. |
0.75
|     |     |     |                        | 0.50 hr |                    |             | hr  |              |      |     |
| --- | --- | --- | ---------------------- | ------- | ------------------ | ----------- | --- | ------------ | ---- | --- |
|     |     |     | testsetniycneuqerfknaR | 0.25    |                    |             |     |              |      |     |
|     |     |     |                        |         | workingday holiday | holiday hum |     | mnth weekday | temp | yr  |
| X1  |     |     |                        | 0.00    |                    |             |     |              |      |     |
75%
| X12 |     |     |     | 1.00 |     |     |     |     |     |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0.75
|     |     | 50% |     | 0.50 |          |            |     |     |             | RF   |
| --- | --- | --- | --- | ---- | -------- | ---------- | --- | --- | ----------- | ---- |
|     |     |     |     | hr   |          |            | hr  |     |             |      |
|     |     |     |     | 0.25 | yr atemp |            |     | yr  |             |      |
| X1  |     |     |     | 0.00 |          | atemp mnth |     |     | atemp atemp | temp |
25%
| X12 |     |     |     | 1.00 |     |     |     |     |     | LightGBM |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | -------- |
0.75
|     |     |     |     | 0.50 hr |     |     | hr  |     |     |     |
| --- | --- | --- | --- | ------- | --- | --- | --- | --- | --- | --- |
0.25
| X1  |     |     |     |     | yr atemp | atemp temp |     | yr  | hum temp | atemp |
| --- | --- | --- | --- | --- | -------- | ---------- | --- | --- | -------- | ----- |
0.00
| 1       | 12 1 12   |     |     | 1    | 2       | 3 4 5      | 1   | 2         | 3 4 | 5   |
| ------- | --------- | --- | --- | ---- | ------- | ---------- | --- | --------- | --- | --- |
|         | Rank      |     |     |      |         | Rank(top5) |     |           |     |     |
| A       |           |     | B   |      |         |            |     |           |     |     |
| Shapley | P-Shapley |     |     |      | Shapley |            |     | P-Shapley |     |     |
| X12     |           |     |     | 1.00 |         |            |     |           |     |     |
Quant.Reg.
0.75
100% testsetniycneuqerfknaR
0.50
|     |     |     |     |                        |         | season hr |     |                                |     |     |
| --- | --- | --- | --- | ---------------------- | ------- | --------- | --- | ------------------------------ | --- | --- |
|     |     | 75% |     | 0.25 weekdayworkingday | holiday |           | hr  | workingdayworkingdayworkingday |     |     |
mnth
| X1  |     |     |     | 0.00 |     |     |     |     |     |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
50%
| X12 |     |     |     | 1.00 |     |     |     |     |     |          |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | -------- |
|     |     | 25% |     | 0.75 |     |     |     |     |     | Quant.RF |
0.50
hr
|     |         |     |     | 0.25 | yr    |             | hr  |          |     |     |
| --- | ------- | --- | --- | ---- | ----- | ----------- | --- | -------- | --- | --- |
|     |         |     |     |      | atemp | atemp atemp |     | yr atemp | hum | hum |
| X1  |         |     |     | 0.00 |       |             |     |          |     |     |
| 1   | 12 1 12 |     |     | 1    | 2     | 3 4 5       | 1   | 2        | 3 4 | 5   |
|     | Rank    |     |     |      |       | Rank(top5)  |     |          |     |     |
FigureC.6: Heatmapofvariableranksacrosstestobservations(A),andtop-5rankfrequenciesofthemostdominant
variables(B).ToptobottomCPintervals: SMR,LACP,CQR.Valuefunction: width. bikedataset
27

| Appendix C.2.2. | blog Dataset |     |         |           |     |
| --------------- | ------------ | --- | ------- | --------- | --- |
| A               |              | B   |         |           |     |
| Shapley         | P-Shapley    |     | Shapley | P-Shapley |     |
X238
1.00
LinearReg.
0.75
testsetniycneuqerfknaR
|     |     | 0.50 X52 | X61 X11 X53 X4 | X1 X2 X3 | X4 X5 |
| --- | --- | -------- | -------------- | -------- | ----- |
0.25
| X1  |     | 0.00 |     |     |     |
| --- | --- | ---- | --- | --- | --- |
100%
| X238 |     | 1.00 |     |     |     |
| ---- | --- | ---- | --- | --- | --- |
LightGBM
0.75
|     |     | 0.50 X61 | X52 X11 X5 X37 | X1 X2 X3 | X4 X5 |
| --- | --- | -------- | -------------- | -------- | ----- |
0.25
| X1      |           | 0.00 |            |           |       |
| ------- | --------- | ---- | ---------- | --------- | ----- |
| 1       | 238 1 238 | 1    | 2 3 4 5    | 1 2       | 3 4 5 |
|         | Rank      |      | Rank(top5) |           |       |
| A       |           | B    |            |           |       |
| Shapley | P-Shapley |      | Shapley    | P-Shapley |       |
| X238    |           | 1.00 |            |           |       |
LinearReg.
testsetniycneuqerfknaR 0.75
100%
|     |     | 0.50     |     | X1 X2 X3 | X4 X5 |
| --- | --- | -------- | --- | -------- | ----- |
|     | 75% | 0.25 X61 | X52 |          |       |
X21
|     |     | 0.00 | X11 X10 |     |     |
| --- | --- | ---- | ------- | --- | --- |
X1 50%
X238
1.00
LightGBM
25% 0.75
|     |     | 0.50 |     | X1 X2 X3 | X4 X5 |
| --- | --- | ---- | --- | -------- | ----- |
0.25
|     |     | X61  | X21         |     |     |
| --- | --- | ---- | ----------- | --- | --- |
|     |     | 0.00 | X53 X11 X42 |     |     |
X1
| 1       | 238 1 238 | 1   | 2 3 4 5    | 1 2       | 3 4 5 |
| ------- | --------- | --- | ---------- | --------- | ----- |
|         | Rank      |     | Rank(top5) |           |       |
| A       |           | B   |            |           |       |
| Shapley | P-Shapley |     | Shapley    | P-Shapley |       |
X238
1.00
Quant.
0.75
100% testsetniycneuqerfknaR
|     |     | 0.50 |            | X1 X2 X3 | X4 X5 |
| --- | --- | ---- | ---------- | -------- | ----- |
|     |     | X36  |            |          | Reg.  |
|     | 75% | 0.25 | X11        |          |       |
|     |     |      | X6 X21 X46 |          |       |
| X1  |     | 0.00 |            |          |       |
50%
| X238 |     | 1.00 |     |     |     |
| ---- | --- | ---- | --- | --- | --- |
Quant.
25% 0.75
0.50
|     |     |     |     | X1 X2 X3 | X4 X5 |
| --- | --- | --- | --- | -------- | ----- |
RF
0.25
| X1  |           | 0.00 X29 | X29 X29 X29 X9 |     |       |
| --- | --------- | -------- | -------------- | --- | ----- |
| 1   | 238 1 238 | 1        | 2 3 4 5        | 1 2 | 3 4 5 |
|     | Rank      |          | Rank(top5)     |     |       |
FigureC.7: Heatmapofvariableranksacrosstestobservations(A),andtop-5rankfrequenciesofthemostdominant
variables(B).ToptobottomCPintervals: SMR,LACP,CQR.Valuefunction: width. blogdataset
28

| Appendix C.2.3. | casp Dataset |     |     |     |         |     |     |           |     |
| --------------- | ------------ | --- | --- | --- | ------- | --- | --- | --------- | --- |
| A               |              |     | B   |     |         |     |     |           |     |
| Shapley         | P-Shapley    |     |     |     | Shapley |     |     | P-Shapley |     |
LinearReg.
| X9  |     |     | 1.00 |     |     |     |     |     |     |
| --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0.75
|     |     |     | testsetniycneuqerfknaR 0.50 | F2  | F4 F3 | F6 F8 | F4  | F2 F3 | F6 F9 |
| --- | --- | --- | --------------------------- | --- | ----- | ----- | --- | ----- | ----- |
0.25
| X1  |     |      | 0.00 |     |       |       |     |       |       |
| --- | --- | ---- | ---- | --- | ----- | ----- | --- | ----- | ----- |
| X9  |     |      | 1.00 |     |       |       |     |       |       |
|     |     |      | 0.75 |     |       |       |     |       | RF    |
|     |     | 100% | 0.50 | F3  | F8 F2 | F4 F5 | F3  | F4 F8 | F9 F7 |
0.25
X1
0.00
LightGBM
| X9  |     |     | 1.00 |     |     |     |     |     |     |
| --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0.75
|     |     |     | 0.50 | F8  | F4 F2 | F6 F3 | F8  | F4 F3 | F9 F2 |
| --- | --- | --- | ---- | --- | ----- | ----- | --- | ----- | ----- |
0.25
| X1      |           |     | 0.00 |     |         |            |     |           |            |
| ------- | --------- | --- | ---- | --- | ------- | ---------- | --- | --------- | ---------- |
| 1       | 9 1 9     |     |      | 1   | 2       | 3 4 5      | 1   | 2         | 3 4 5      |
|         | Rank      |     |      |     |         | Rank(top5) |     |           |            |
| A       |           |     | B    |     |         |            |     |           |            |
| Shapley | P-Shapley |     |      |     | Shapley |            |     | P-Shapley |            |
| X9      |           |     |      |     |         |            |     |           | LinearReg. |
0.4
0.3
|     |     |     | 0.2                    |     |     |       | F4  |     |          |
| --- | --- | --- | ---------------------- | --- | --- | ----- | --- | --- | -------- |
|     |     |     | testsetniycneuqerfknaR | F4  | F2  |       |     |     |          |
|     |     |     | 0.1                    |     | F2  | F6 F6 |     | F2  | F3 F6 F6 |
| X1  |     | 40% | 0.0                    |     |     |       |     |     |          |
X9
|     |     | 30% | 0.4 |     |       |       |     |     |          |
| --- | --- | --- | --- | --- | ----- | ----- | --- | --- | -------- |
|     |     |     | 0.3 |     |       |       |     |     | RF       |
|     |     |     | 0.2 | F3  |       |       | F3  |     |          |
|     |     | 20% | 0.1 |     | F3 F2 | F2 F6 |     | F3  | F4 F6 F7 |
| X1  |     |     | 0.0 |     |       |       |     |     |          |
10%
| X9  |     |     |     |     |     |     |     |     | LightGBM |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- |
0.4
0.3
0.2
|     |     |     | 0.1 | F4  |       |       | F4  |     |          |
| --- | --- | --- | --- | --- | ----- | ----- | --- | --- | -------- |
| X1  |     |     |     |     | F8 F2 | F6 F6 |     | F4  | F6 F6 F7 |
0.0
| 1       | 9 1 9     |     |     | 1   | 2 3     | 4 5        | 1   | 2         | 3 4 5 |
| ------- | --------- | --- | --- | --- | ------- | ---------- | --- | --------- | ----- |
|         | Rank      |     |     |     |         | Rank(top5) |     |           |       |
| A       |           |     | B   |     |         |            |     |           |       |
| Shapley | P-Shapley |     |     |     | Shapley |            |     | P-Shapley |       |
X9
|     |     |     | 0.3                        |     |     |       |     |     | Quant.Reg. |
| --- | --- | --- | -------------------------- | --- | --- | ----- | --- | --- | ---------- |
|     |     |     | testsetniycneuqerfknaR 0.2 |     |     |       |     |     |            |
|     |     | 30% |                            |     | F2  |       | F4  |     |            |
|     |     |     | 0.1                        | F4  | F2  |       |     | F2  | F3 F3      |
|     |     |     |                            |     |     | F3 F1 |     |     | F1         |
| X1  |     | 20% | 0.0                        |     |     |       |     |     |            |
X9
|     |     | 10% | 0.3 |     |     |     |     |     | Quant.RF |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- |
0.2
F3
|     |       |     | 0.1 |     | F1 F1 | F1 F9      | F3  | F6  | F6 F9 F9 |
| --- | ----- | --- | --- | --- | ----- | ---------- | --- | --- | -------- |
| X1  |       |     | 0.0 |     |       |            |     |     |          |
| 1   | 9 1 9 |     |     | 1   | 2 3   | 4 5        | 1   | 2   | 3 4 5    |
|     | Rank  |     |     |     |       | Rank(top5) |     |     |          |
FigureC.8: Heatmapofvariableranksacrosstestobservations(A),andtop-5rankfrequenciesofthemostdominant
variables(B).ToptobottomCPintervals: SMR,LACP,CQR.Valuefunction: width. caspdataset
29

| Appendix C.2.4. | concrete Dataset |     |     |     |         |     |     |           |     |     |
| --------------- | ---------------- | --- | --- | --- | ------- | --- | --- | --------- | --- | --- |
| A               |                  |     | B   |     |         |     |     |           |     |     |
| Shapley         | P-Shapley        |     |     |     | Shapley |     |     | P-Shapley |     |     |
LinearReg.
| X8  |     |     |     | 1.00 |     |     |     |     |     |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0.75
0.50
|     |     |     | testsetniycneuqerfknaR | cement | h2o | spt bfs flash | cement | h2o | spt flash | bfs |
| --- | --- | --- | ---------------------- | ------ | --- | ------------- | ------ | --- | --------- | --- |
0.25
| X1  |     |      |     | 0.00        |     |               |        |     |           |     |
| --- | --- | ---- | --- | ----------- | --- | ------------- | ------ | --- | --------- | --- |
| X8  |     |      |     | 1.00        |     |               |        |     |           |     |
|     |     |      |     | 0.75        |     |               |        |     |           | RF  |
|     |     | 100% |     | 0.50 cement | age | fiag ca flash | cement | age | spt flash | bfs |
0.25
X1
0.00
LightGBM
| X8  |     |     |     | 1.00 |     |     |     |     |     |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0.75
|     |     |     |     | 0.50 age | cement | h2o ca bfs | age | cement | spt flash | ca  |
| --- | --- | --- | --- | -------- | ------ | ---------- | --- | ------ | --------- | --- |
0.25
| X1      |           |     |     | 0.00 |         |            |     |           |     |            |
| ------- | --------- | --- | --- | ---- | ------- | ---------- | --- | --------- | --- | ---------- |
| 1       | 8 1 8     |     |     | 1    | 2       | 3 4 5      | 1   | 2         | 3 4 | 5          |
|         | Rank      |     |     |      |         | Rank(top5) |     |           |     |            |
| A       |           |     | B   |      |         |            |     |           |     |            |
| Shapley | P-Shapley |     |     |      | Shapley |            |     | P-Shapley |     |            |
| X8      |           |     |     |      |         |            |     |           |     | LinearReg. |
0.6
0.4
|     |     |     | testsetniycneuqerfknaR | 0.2 cement | cement |         | cement |        |         |     |
| --- | --- | --- | ---------------------- | ---------- | ------ | ------- | ------ | ------ | ------- | --- |
| X1  |     |     |                        |            | h2o    | spt bfs |        | cement | h2o bfs | ca  |
|     |     | 60% |                        | 0.0        |        |         |        |        |         |     |
X8
0.6
|     |     | 40% |     | 0.4 |     |     |     |     |     | RF  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
age
|     |     |     |     | 0.2 age | cement h2o | spt spt |     | cement | spt |          |
| --- | --- | --- | --- | ------- | ---------- | ------- | --- | ------ | --- | -------- |
| X1  |     | 20% |     | 0.0     |            |         |     |        | bfs | bfs      |
| X8  |     |     |     |         |            |         |     |        |     | LightGBM |
0.6
|     |     |     |     | 0.4 age |        |         | age |     |            |        |
| --- | --- | --- | --- | ------- | ------ | ------- | --- | --- | ---------- | ------ |
|     |     |     |     | 0.2     | cement |         |     | spt |            |        |
| X1  |     |     |     |         | ca     | ca fiag |     |     | spt cement | cement |
0.0
| 1       | 8 1 8     |     |     | 1   | 2       | 3 4 5      | 1   | 2         | 3 4 | 5   |
| ------- | --------- | --- | --- | --- | ------- | ---------- | --- | --------- | --- | --- |
|         | Rank      |     |     |     |         | Rank(top5) |     |           |     |     |
| A       |           |     | B   |     |         |            |     |           |     |     |
| Shapley | P-Shapley |     |     |     | Shapley |            |     | P-Shapley |     |     |
X8
Quant.Reg.
0.6
testsetniycneuqerfknaR
0.4
cement
|     |     | 60% |     | 0.2 | age |         | cement |     |         |     |
| --- | --- | --- | --- | --- | --- | ------- | ------ | --- | ------- | --- |
|     |     |     |     |     |     | bfs h2o |        | age | spt bfs | bfs |
| X1  |     |     |     |     | spt |         |        |     |         |     |
0.0
40%
X8
|     |     | 20% |     | 0.6 |     |     |     |     |     | Quant.RF |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- |
0.4
|     |       |     |     | cement |         |            | age |     |         |       |
| --- | ----- | --- | --- | ------ | ------- | ---------- | --- | --- | ------- | ----- |
|     |       |     |     | 0.2    | age bfs | spt        |     |     |         |       |
|     |       |     |     |        |         | spt        |     | spt | h2o bfs | flash |
| X1  |       |     |     | 0.0    |         |            |     |     |         |       |
| 1   | 8 1 8 |     |     | 1      | 2       | 3 4 5      | 1   | 2   | 3 4     | 5     |
|     | Rank  |     |     |        |         | Rank(top5) |     |     |         |       |
FigureC.9: Heatmapofvariableranksacrosstestobservations(A),andtop-5rankfrequenciesofthemostdominant
variables(B).ToptobottomCPintervals: SMR,LACP,CQR.Valuefunction: width. concretedataset
30

| Appendix C.2.5. | facebook Dataset |     |     |     |         |     |     |           |     |
| --------------- | ---------------- | --- | --- | --- | ------- | --- | --- | --------- | --- |
| A               |                  |     | B   |     |         |     |     |           |     |
| Shapley         | P-Shapley        |     |     |     | Shapley |     |     | P-Shapley |     |
LinearReg.
| X37 |     |     | 1.00 |     |     |     |     |     |     |
| --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0.75
|     |     |     | testsetniycneuqerfknaR 0.50 | X40 | X17 X27 | X12 X47 | X12 | X9 X29 | X47 X40 |
| --- | --- | --- | --------------------------- | --- | ------- | ------- | --- | ------ | ------- |
0.25
| X1  |     |      | 0.00 |     |         |         |     |        |         |
| --- | --- | ---- | ---- | --- | ------- | ------- | --- | ------ | ------- |
| X37 |     |      | 1.00 |     |         |         |     |        |         |
|     |     |      | 0.75 |     |         |         |     |        | RF      |
|     |     | 100% | 0.50 | X31 | X33 X35 | X30 X32 | X31 | X8 X47 | X35 X26 |
0.25
| X1  |     |     | 0.00 |     |     |     |     |     |          |
| --- | --- | --- | ---- | --- | --- | --- | --- | --- | -------- |
| X37 |     |     |      |     |     |     |     |     | LightGBM |
1.00
0.75
|     |     |     | 0.50 | X35 | X31 X30 | X33 X32 | X35 | X33 X30 | X47 X28 |
| --- | --- | --- | ---- | --- | ------- | ------- | --- | ------- | ------- |
0.25
| X1      |           |     | 0.00 |     |         |            |     |           |       |
| ------- | --------- | --- | ---- | --- | ------- | ---------- | --- | --------- | ----- |
| 1       | 37 1 37   |     |      | 1   | 2       | 3 4 5      | 1   | 2         | 3 4 5 |
|         | Rank      |     |      |     |         | Rank(top5) |     |           |       |
| A       |           |     | B    |     |         |            |     |           |       |
| Shapley | P-Shapley |     |      |     | Shapley |            |     | P-Shapley |       |
X37 LinearReg.
0.75
0.50 X40
|     |     |     | testsetniycneuqerfknaR 0.25 |     |     |             |     |     |             |
| --- | --- | --- | --------------------------- | --- | --- | ----------- | --- | --- | ----------- |
|     |     |     |                             |     | X17 | X12 X27 X27 | X31 | X30 | X17 X13 X32 |
| X1  |     |     | 0.00                        |     |     |             |     |     |             |
75%
X37
0.75
|     |     | 50% | 0.50 |     |     |         |     |     | RF          |
| --- | --- | --- | ---- | --- | --- | ------- | --- | --- | ----------- |
|     |     |     | 0.25 | X35 | X31 | X32     | X28 |     |             |
| X1  |     |     | 0.00 |     |     | X37 X33 |     | X35 | X27 X19 X19 |
25%
X37 LightGBM
0.75
0.50
|         |           |     | 0.25 | X35 |         |             | X28 |           |             |
| ------- | --------- | --- | ---- | --- | ------- | ----------- | --- | --------- | ----------- |
|         |           |     |      |     | X32     | X32 X33 X30 |     | X27       | X27 X19 X19 |
| X1      |           |     | 0.00 |     |         |             |     |           |             |
| 1       | 37 1 37   |     |      | 1   | 2       | 3 4 5       | 1   | 2         | 3 4 5       |
|         | Rank      |     |      |     |         | Rank(top5)  |     |           |             |
| A       |           |     | B    |     |         |             |     |           |             |
| Shapley | P-Shapley |     |      |     | Shapley |             |     | P-Shapley |             |
| X37     |           |     | 1.00 |     |         |             |     |           |             |
Quant.Reg.
0.75
100%
|     |     |     | testsetniycneuqerfknaR 0.50 |     |     |           |     |        |     |
| --- | --- | --- | --------------------------- | --- | --- | --------- | --- | ------ | --- |
|     |     |     |                             | X12 | X27 | X1 X17 X3 |     |        |     |
|     |     |     |                             |     |     |           | X12 |        | X27 |
|     |     | 75% | 0.25                        |     |     |           |     | X1 X17 |     |
X2
| X1  |     |     | 0.00 |     |     |     |     |     |     |
| --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
50%
| X37 |     |     | 1.00 |     |     |     |     |     |          |
| --- | --- | --- | ---- | --- | --- | --- | --- | --- | -------- |
|     |     | 25% | 0.75 |     |     |     |     |     | Quant.RF |
0.50
0.25
|     |         |     |      | X35 | X35 |            | X28 | X17 |       |
| --- | ------- | --- | ---- | --- | --- | ---------- | --- | --- | ----- |
| X1  |         |     | 0.00 |     | X35 | X30 X30    |     | X28 | X4 X4 |
| 1   | 37 1 37 |     |      | 1   | 2   | 3 4 5      | 1   | 2   | 3 4 5 |
|     | Rank    |     |      |     |     | Rank(top5) |     |     |       |
FigureC.10: Heatmapofvariableranksacrosstestobservations(A),andtop-5rankfrequenciesofthemostdominant
variables(B).ToptobottomCPintervals: SMR,LACP,CQR.Valuefunction: width. facebookdataset
31

| Appendix C.2.6. | UScrime Dataset |     |     |         |     |     |           |     |            |
| --------------- | --------------- | --- | --- | ------- | --- | --- | --------- | --- | ---------- |
| A               |                 |     | B   |         |     |     |           |     |            |
| Shapley         | P-Shapley       |     |     | Shapley |     |     | P-Shapley |     |            |
| X101            |                 |     |     |         |     |     |           |     | LinearReg. |
1.00
0.75
0.50
|     |     |     | testsetniycneuqerfknaR PctIllegPctKids2ParPctFam2Par |     | state racePctWhite | PctPopUnderPopvctUrPbaenrsPerRentOcPccHtRoeucsentImmPicgtKids2Par |     |     |     |
| --- | --- | --- | ---------------------------------------------------- | --- | ------------------ | ----------------------------------------------------------------- | --- | --- | --- |
0.25
| X1   |     |      | 0.00                                                     |     |     |                                                                 |     |     |     |
| ---- | --- | ---- | -------------------------------------------------------- | --- | --- | --------------------------------------------------------------- | --- | --- | --- |
| X101 |     |      | 1.00                                                     |     |     |                                                                 |     |     |     |
|      |     |      | 0.75                                                     |     |     |                                                                 |     |     | RF  |
|      |     | 100% | 0.50 PctIllegracePctWhitePctKids2ParNumIllegracepctblack |     |     | PctLess9thGrardaecepctblackMedRentPctFamP2cPtNarotSpeakEnglWell |     |     |     |
0.25
| X1   |     |     | 0.00 |     |     |     |     |     |          |
| ---- | --- | --- | ---- | --- | --- | --- | --- | --- | -------- |
| X101 |     |     |      |     |     |     |     |     | LightGBM |
1.00
0.75
0.50 PctIlleg PctFam2ParPctKids2ParNumIllePgctVacantBoardedPctPopUndMeerPdoOvwnCostPPccttLInecss9thOGwrandOeccLowQPucatErtmplManu
0.25
| X1      |           |     | 0.00 |         |     |            |           |     |            |
| ------- | --------- | --- | ---- | ------- | --- | ---------- | --------- | --- | ---------- |
| 1       | 101 1 101 |     | 1    | 2       | 3 4 | 5 1        | 2 3       | 4   | 5          |
|         | Rank      |     |      |         |     | Rank(top5) |           |     |            |
| A       |           |     | B    |         |     |            |           |     |            |
| Shapley | P-Shapley |     |      | Shapley |     |            | P-Shapley |     |            |
| X101    |           |     |      |         |     |            |           |     | LinearReg. |
0.4
0.3
0.2
|     |     | 40% | testsetniycneuqerfknaR 0.1 PctIlleg |                                            |     |                      |                                   |     |     |
| --- | --- | --- | ----------------------------------- | ------------------------------------------ | --- | -------------------- | --------------------------------- | --- | --- |
|     |     |     |                                     | PctIllegracePctWhitePctKids2ParPctKids2Par |     | PctLess9thGradestate |                                   |     |     |
| X1  |     |     | 0.0                                 |                                            |     |                      | medFamIncLandArPeactYoungKids2Par |     |     |
X101
|     |     | 30% | 0.4 |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
0.3
RF
|     |     | 20% | 0.2 PctIlleg |     |     |     |     |     |     |
| --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- |
0.1 PctIllegracePctWhitePctKids2ParPctFam2Par
| X1  |     |     | 0.0 |     |     | PctLess9thGradLeandAreaPctRecImmig5LandArea |     | NumStreet |     |
| --- | --- | --- | --- | --- | --- | ------------------------------------------- | --- | --------- | --- |
10%
| X101 |     |     | 0.4 |     |     |     |     |     | LightGBM |
| ---- | --- | --- | --- | --- | --- | --- | --- | --- | -------- |
0.3
0.2 PctIlleg
|     |     |     | 0.1 | PctKids2Par PctKids2ParPctFam2ParPctFam2Par |     |     |     |     |     |
| --- | --- | --- | --- | ------------------------------------------- | --- | --- | --- | --- | --- |
racepctblackPctFam2ParracepctblacPkctRecImmigP5ctKids2Par
| X1      |           |     | 0.0 |         |     |            |           |     |     |
| ------- | --------- | --- | --- | ------- | --- | ---------- | --------- | --- | --- |
| 1       | 101 1 101 |     | 1   | 2 3     | 4   | 5 1        | 2         | 3 4 | 5   |
|         | Rank      |     |     |         |     | Rank(top5) |           |     |     |
| A       |           |     | B   |         |     |            |           |     |     |
| Shapley | P-Shapley |     |     | Shapley |     |            | P-Shapley |     |     |
X101
|     |     |     | 0.6                        |     |     |     |     |     | Quant.Reg. |
| --- | --- | --- | -------------------------- | --- | --- | --- | --- | --- | ---------- |
|     |     |     | testsetniycneuqerfknaR 0.4 |     |     |     |     |     |            |
state
|     |     | 75% | 0.2                                                   |     |     |             |             |       |       |
| --- | --- | --- | ----------------------------------------------------- | --- | --- | ----------- | ----------- | ----- | ----- |
|     |     |     | racePctWhPitcetPopUndPercPtYovoungKids2PPcatrKids2Par |     |     | PctTeen2Par |             |       |       |
|     |     |     |                                                       |     |     |             | state state | state |       |
| X1  |     |     | 0.0                                                   |     |     |             |             |       | state |
50%
X101
0.6
|     |     | 25% |     |     |     |     |     |     | Quant.RF |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- |
0.4
0.2
state
PctPopUnderPovstate
X1 0.0 racePctWhitePctKids2Par NumImmigHousVacaPncttLess9thGMraadleePctNevMarrPctIlleg
| 1   | 101 1 101 |     | 1   | 2 3 | 4   | 5 1        | 2   | 3 4 | 5   |
| --- | --------- | --- | --- | --- | --- | ---------- | --- | --- | --- |
|     | Rank      |     |     |     |     | Rank(top5) |     |     |     |
FigureC.11: Heatmapofvariableranksacrosstestobservations(A),andtop-5rankfrequenciesofthemostdominant
variables(B).ToptobottomCPintervals: SMR,LACP,CQR.Valuefunction: width. uscrimedataset
32

| Appendix C.2.7. | star Dataset |     |     |     |         |     |     |           |     |     |
| --------------- | ------------ | --- | --- | --- | ------- | --- | --- | --------- | --- | --- |
| A               |              |     | B   |     |         |     |     |           |     |     |
| Shapley         | P-Shapley    |     |     |     | Shapley |     |     | P-Shapley |     |     |
LinearReg.
| X39 |     |     |     | 1.00 |     |     |     |     |     |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0.75
0.50
|     |     |     | testsetniycneuqerfknaR | ethnicity | birth stark | star1 lunch1 | ethnicity | schoolk | gender school2 degree1 |     |
| --- | --- | --- | ---------------------- | --------- | ----------- | ------------ | --------- | ------- | ---------------------- | --- |
0.25
| X1  |     |      |     | 0.00        |                      |                 |        |        |                          |     |
| --- | --- | ---- | --- | ----------- | -------------------- | --------------- | ------ | ------ | ------------------------ | --- |
| X39 |     |      |     | 1.00        |                      |                 |        |        |                          |     |
|     |     |      |     | 0.75        |                      |                 |        |        |                          | RF  |
|     |     | 100% |     | 0.50 lunchk | experiencekschoolidk | system2 school2 | lunch3 | gender | lunchk ladder1 schoolid2 |     |
0.25
| X1  |     |     |     | 0.00 |     |     |     |     |     |          |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | -------- |
| X39 |     |     |     |      |     |     |     |     |     | LightGBM |
1.00
0.75
|     |     |     |     | 0.50 birth | experiencekexperience3schoolid2experience1 |     | degree2 | experiencekexperience3 | lunch3 tethnicityk |     |
| --- | --- | --- | --- | ---------- | ------------------------------------------ | --- | ------- | ---------------------- | ------------------ | --- |
0.25
| X1      |           |     |     | 0.00 |         |            |     |           |     |            |
| ------- | --------- | --- | --- | ---- | ------- | ---------- | --- | --------- | --- | ---------- |
| 1       | 39 1 39   |     |     | 1    | 2 3     | 4 5        | 1   | 2         | 3 4 | 5          |
|         | Rank      |     |     |      |         | Rank(top5) |     |           |     |            |
| A       |           |     | B   |      |         |            |     |           |     |            |
| Shapley | P-Shapley |     |     |      | Shapley |            |     | P-Shapley |     |            |
| X39     |           |     |     | 1.00 |         |            |     |           |     | LinearReg. |
0.75
|     |     |      |                        | 0.50 birth | ethnicity stark | lunch1 star1 |           |        |                     |     |
| --- | --- | ---- | ---------------------- | ---------- | --------------- | ------------ | --------- | ------ | ------------------- | --- |
|     |     | 100% | testsetniycneuqerfknaR | 0.25       |                 |              | ethnicity |        |                     |     |
|     |     |      |                        |            |                 |              |           | lunch1 | tethnicityk school1 |     |
| X1  |     |      |                        | 0.00       |                 |              |           |        | degree1             |     |
| X39 |     | 75%  |                        |            |                 |              |           |        |                     |     |
1.00
0.75
|     |     | 50% |     | 0.50 |     |     |     |     |     | RF  |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0.25
X1 0.00 birth experiencekexperiencekexperience2experience3 birth experience1experience1 birth experience1
25%
| X39 |     |     |     | 1.00 |     |     |     |     |     | LightGBM |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | -------- |
0.75
0.50
0.25
|         |           |     |     | birth | birth experience1experience2experiencek |            | birth | birth     | birth experiencekexperience3 |     |
| ------- | --------- | --- | --- | ----- | --------------------------------------- | ---------- | ----- | --------- | ---------------------------- | --- |
| X1      |           |     |     | 0.00  |                                         |            |       |           |                              |     |
| 1       | 39 1 39   |     |     | 1     | 2 3                                     | 4 5        | 1     | 2         | 3 4                          | 5   |
|         | Rank      |     |     |       |                                         | Rank(top5) |       |           |                              |     |
| A       |           |     | B   |       |                                         |            |       |           |                              |     |
| Shapley | P-Shapley |     |     |       | Shapley                                 |            |       | P-Shapley |                              |     |
| X39     |           |     |     | 1.00  |                                         |            |       |           |                              |     |
Quant.Reg.
0.75
100% testsetniycneuqerfknaR
0.50
|     |     |     |     | birth | ethnicity school2 | school3 schoolk |         |             |                           |     |
| --- | --- | --- | --- | ----- | ----------------- | --------------- | ------- | ----------- | ------------------------- | --- |
|     |     | 75% |     | 0.25  |                   |                 | schoolk | experience2 | school3 schoolidk system3 |     |
| X1  |     |     |     | 0.00  |                   |                 |         |             |                           |     |
50%
| X39 |     |     |     | 1.00 |     |     |     |     |     |          |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | -------- |
|     |     | 25% |     | 0.75 |     |     |     |     |     | Quant.RF |
0.50
0.25
X1 0.00 lunch1 lunchk lunch1 lunch1 ladder3 degree3 tethnicity3 lunch1 system1 system3
| 1   | 39 1 39 |     |     | 1   | 2 3 | 4 5        | 1   | 2   | 3 4 | 5   |
| --- | ------- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- |
|     | Rank    |     |     |     |     | Rank(top5) |     |     |     |     |
FigureC.12: Heatmapofvariableranksacrosstestobservations(A),andtop-5rankfrequenciesofthemostdominant
variables(B).ToptobottomCPintervals: SMR,LACP,CQR.Valuefunction: width. stardataset
33