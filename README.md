# CliqueRT
R package for randomization tests of causal effects under general interference.  The full package is under construction, but the main functions are up and running!  

This is an implementation of the clique-based randomization test developed in the paper [A Graph-Theoretic Approach to Randomization Tests of Causal Effects Under General Interference](https://arxiv.org/pdf/1910.10862.pdf). If used, please cite the paper.

To use, first install devtools:
```R
install.packages("devtools")
```
and then, install the package:
```R
library(devtools)
install_github("dpuelz/CliqueRT")
```

The following example runs a small simulation on a synthetic network 
with 500 nodes:

```R
# generated network - 3 clusters of 2D Gaussians
# loads in the 500x500 matrix Dmat
# Dmat just encodes all pairwise Euclidean distances between network nodes, and
# this is used to define the spillover hypothesis below.
library(CliqueRT)
set.seed(1)
thenetwork = out_example_network(500)
D = thenetwork$D

# simulation parameters
num_randomizations = 5000
radius = 0.01

# First, construct Z, Z_a, Z_b.
# Here, a unit receives exposure a if it is untreated, but within radius of a 
# treated unit; it receives exposure b if it is untreated and at least radius 
# distance away from all treated units.
# Experimental design is Bernoulli with prob=0.2.
# a_threshold is a scalar denoting the threshold that triggers an exposure to a.  
# If exposure a is simply binary, i.e. whether or not unit j is exposed to a, then 
# this value should be set to 1.
# b_threshold is a scalar denoting the threshold that triggers an exposure to b.  If exposure b
# is simply binary, i.e. whether or not unit j is exposed to b, then this value should be set to 1.
Z = out_Z(pi=rep(0.2,dim(D)[1]),num_randomizations)
D_a = sparsify((D<radius)); a_threshold = 1
D_b = sparsify((D<radius)); b_threshold = 1

Z_a = D_a%*%Z
Z_a = sparsify((Z_a>=a_threshold))
Z_b = D_b%*%Z
Z_b = sparsify((Z_b<b_threshold))

# simulating an outcome vector
Y_a = rnorm(dim(Z)[1])
Y_b = Y_a + 0.2
Y = out_Yobs(Z_a[,1],Z_b[,1],Y_a,Y_b)

# run the test
CRT = clique_test(Y,Z,Z_a,Z_b,Zobs_id=1,minr=15,minc=15)
```
