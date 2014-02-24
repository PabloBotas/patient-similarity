patient_similarity.py computes the similarities between one patient and a list of other patients. Pass --help for details.

Usage example:
./patient_similarity.py --log=INFO test/test.hpo data/hp.obo data/phenotype_annotation.tab data/en_product1.xml data/en_product2.xml > test/test.out

Algorithm Description (I'm sorry if this is a bit messy. If it's hard to understand, I can show you on the whiteboard.)

Notation
t is usually an HPO term
t.freq is the relative frequency of term t, computed from phenotype_annotation.tab. We have sum_t t.freq = 1.
t.prob is the total probability mass in the descendants of t, including t. We have sum_t t.prob > 1.
parents(t) is the set of direct parents of t
ancestors(t) is a sequence of all ancestors of t, including t. This sequence is always assumed to have an ordering consistent with the HPO tree. It must start with All and end with t. Typically, we will index elements of this set as [a_i for a_i in ancestors(t)].


Model:
In our model of patients, we make a distinction between "underlying phenotype" and "observed phenotype". The observed phenotype is the one reported by the doctor. The underlying phenotype can be any one of the descendants of the observed traits. In our model, we assume that traits are independent. That is, we assume that the probability of a patient with a set of underlying phenotypes U is proportional to

f(|U|) * prod_{t in U} t.freq    (1)

for some distribution f(|U|) which gives the probability that a patient has exactly |U| traits.

Now given a set of observed phenotypes O, there are many possible sets of underlying phenotypes U. The total probability of this set of underlying phenotypes is

f(|O|) * prod_{t in O} t.prob    (2)

Note the change from t.freq, to t.prob. We'll essentially disregard the term f(|O|) from here on.

Information Content:
The information contained in a set of observations can be found by taking a log. To prepare for taking the log of (2), let's first find the cost of describing a single observed phenotype t:

cost(t)=log(t.prob
= sum_{a_i in ancestors(t)} log(P(a_i|ancestors(t)[0:i]))
<= sum_{a_i in ancestors(t)} log(P(a_i|parents(a_i)))                             (3)
<= sum_{a_i in ancestors(t)} log(a_i.prob) - max_{p in parents(a_i)} log(p.prob)  (4)

The first equality is just writing t.prob as a chain of conditional probabilities involving all ancestors of t. The inequality comes from removing the dependence of the probability of a_i on its siblings. This dependence can arise if a_i and its siblings/cousins have common descendants. The last inequality is an equality in the case of a tree.

Now the information or cost of a set of observations U is

cost(U)=
log(equation (2)) = log(f(|O|)) + sum_{t in O} sum_{a_i in ancestors(t)} log(P(a_i|ancestors(t)[0:i]))    (5)

Shared Information:
For two sets of observations U_1 and U_2, we can compute cost(U_1) and cost(U_2) as in (5). To do this, we can use the approximations (3) or (4). The shared information is the sum of the common terms in the expressions for cost(U_1) and cost(U_2).

Normalization:
The similarity that we use is 2*shared_info/(cost(U_1) + cost(U_2)). This can be thought of as 1 minus the percent compression ratio achieved by describing both patients at once.
