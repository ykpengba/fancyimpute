[![Build Status](https://travis-ci.org/hammerlab/fancyimpute.svg?branch=master)](https://travis-ci.org/hammerlab/fancyimpute) [![Coverage Status](https://coveralls.io/repos/hammerlab/fancyimpute/badge.svg?branch=master&service=github)](https://coveralls.io/github/hammerlab/fancyimpute?branch=master)

# fancyimpute

A variety of matrix completion and imputation algorithms implemented in Python.

## Usage

```python
from fancyimpute import BiScaler, KNN, NuclearNormMinimization, SoftImpute

# X is a data matrix which we're going to randomly drop entries from
missing_mask = np.random.randn(*X.shape) > 0
X_incomplete = X.copy()
# missing entries indicated with NaN
X_incomplete[missing_mask] = np.nan


# Use 3 nearest rows which have a feature to fill in each row's missing features
knnImpute = KNN(k=3)
X_filled_knn = knnImpute.complete(X_incomplete)

# matrix completion using convex optimization to find low-rank solution
# that still matches observed values. Slow!
X_filled_nnm = NuclearNormMinimization().complete(X_incomplete)

# Instead of solving the nuclear norm objective directly, instead
# induce sparsity using singular value thresholding
softImpute = SoftImpute()

# simultaneously normalizes the rows and columns of your observed data,
# sometimes useful for low-rank imputation methods
biscaler = BiScaler()

# rescale both rows and columns to have zero mean and unit variance
X_incomplete_normalized = biscaler.fit_transform(X_incomplete)

X_filled_softimpute_normalized = softImpute.complete(X_incomplete_normalized)
X_filled_softimpute = biscaler.inverse_transform(X_filled_softimpute_normalized)

# print mean squared error for the three imputation methods above
nnm_mse = ((X_filled_knn[missing_mask] - X[missing_mask]) ** 2).mean()
print("Nuclear norm minimization MSE: %f" % nnm_mse)

softImpute_mse = ((X_filled_softimpute[missing_mask] - X[missing_mask]) ** 2).mean()
print("SoftImpute MSE: %f" % softImpute_mse)

knn_mse = ((X_filled_knn[missing_mask] - X[missing_mask]) ** 2).mean()
print("knnImpute MSE: %f" % knn_mse)
```

## Algorithms

* `SimpleFill`: Replaces missing entries with the mean or median of each column.

* `KNN`: Nearest neighbor imputations which weights samples using the mean squared difference
on features for which two rows both have observed data.

* `SoftImpute`: Matrix completion by iterative soft thresholding of SVD decompositions. Inspired by the [softImpute](https://web.stanford.edu/~hastie/swData/softImpute/vignette.html) package for R, which is based on [Spectral Regularization Algorithms for Learning Large Incomplete Matrices](http://web.stanford.edu/~hastie/Papers/mazumder10a.pdf) by Mazumder et. al.

* `IterativeSVD`: Matrix completion by iterative low-rank SVD decomposition. Should be similar to SVDimpute from [Missing value estimation methods for DNA microarrays](http://www.ncbi.nlm.nih.gov/pubmed/11395428) by Troyanskaya et. al.

* `MICE`: Reimplementation of [Multiple Imputation by Chained Equations](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3074241/).

* `MatrixFactorization`: Direct factorization of the incomplete matrix into low-rank `U` and `V`, with an L1 sparsity penalty on the elements of `U` and an L2 penalty on the elements of `V`. Solved by gradient descent.

* `NuclearNormMinimization`: Simple implementation of [Exact Matrix Completion via Convex Optimization](http://statweb.stanford.edu/~candes/papers/MatrixCompletion.pdf
) by Emmanuel Candes and Benjamin Recht using [cvxpy](http://www.cvxpy.org/en/latest/). Too slow for large matrices.

* `BiScaler`: Iterative estimation of row/column means and standard deviations to get doubly normalized
matrix. Not guaranteed to converge but works well in practice. Taken from [Matrix Completion and Low-Rank SVD via Fast Alternating Least Squares](http://arxiv.org/abs/1410.2596).

