# normestm
MATLAB code to estimate the largest element of a matrix using only matrix-vector products.

## Introduction
The functions in this repository are designed to estimate the quantity max(max(abs(A))) where the matrix A
is not known explicitly.
For example we may have A = B*C, A = expm(B), or A = inv(B) where forming A explicitly is impractical
(maybe even impossible if B and C are large and sparse).
However in each case forming matrix-vector products with A and its conjugate transpose is efficient.

The two algorithms in this repository: normestm and normestm_multi are designed to find the largest and the top-p largest
elements of A, respectively.
In particular normestm_multi **has a dependency** on maxk, authored by Bruno Luong. This free software can be downloaded from the [MATLAB File Exchange.](http://uk.mathworks.com/matlabcentral/fileexchange/23576-min-max-selection)

Our algorithms are a heavily modified version of the mixed subordinate norm estimators by Boyd (1974)
to estimate the (1,infinity)-norm which is equal to max(max(abs(A))).
The full details, along with thorough numerical experiments of the accuracy, can be found in the
(open access) paper (ADD LINK).

To check the code is functioning properly you can run [the testcode](normestm_testcode.m) in MATLAB.

## Details
Our algorithms can be applied to matrices known explicitly, which is usually less efficient than just calling max(max(abs(A))) direclty, or to matrices known only implicitly. Here is an example of each case.

```matlab
A = inv(randn(500));
opts.t = 3;
opts.abs = true;
[nrmest, nrmrow, nrmcol, iters] = normestm(A, opts);

clear all
A = inv(randn(500));
opts.alpha = 5;
opts.abs = true;
p = 5;
[nrmests, nrmrows, nrmcols, iters] = normestm_multi(A, p, opts);
```
#### Normest inputs and outputs
The inputs to normestm are:
* A    - Matrix with real/complex entries or a function handle (see advanced examples below). Can be rectangular.
* opts - Struct where opts.t is an integer controlling the accuracy and opts.abs determines whether we find max(max(abs(A))) or max(max(A)).

In addition normestm_multi has one extra input:
* p - An integer stating how many of the top-p largest elements are required.

The outputs of normestm and normestm_multi are:
* nrmest    - The estimate(s) of max(max(abs(A))).
* nrmestrow - The rows where the estimates appear.
* nrmestcol - The columns where the estimates appear.
* iter      - The number of iterations required.

#### Using implicitly defined matrices
The main power of this algorithm is that it can be applied to matrices where only the matrix-vector products are known.
To make use of this functionality you will need to write a short wrapper function to perform the matrix-vector products
in the follwing form (only the first two arguments are required).

```matlab
function b = mywrapper(flag, x, A)
% MYWRAPPER Computes matrix-vector products for use with normestm and normestm_multi.

switch flag
    case 'dim'
        % Compute size of the implicit matrix (number of rows and columns).
        b = ...;
    case 'notransp'
        % Compute A*x
        b = ...
    case 'transp'
        % Compute A'*x
        b = ...
    otherwise
        error('Undefined flag');
end
end
```

A fully functioning wrapper for computing the matrix exponential (using [expmv](http://www.mathworks.com/matlabcentral/fileexchange/29576-matrix-exponential-times-a-vector/content/expmv.m))
is included in the repository as [expmv_wrapper.m.](expmv_wrapper.m)
We can find the p = 10 largest entries of the matrix exponential as follows 
(the matrix is taken from the University of Florida Sparse Matrix Collection).

```matlab
p = 10;
opts.alpha = 3;
opts.abs = true;
A = UFget('SNAP/ca-AstroPh');
A = A.A;
[nrmests, nrmows, nrmcols] = normestm_multi_deflate(@expmv_wrapper, p, opts, A);
```

## Referencing
To cite this code please use the following.

Nicholas J. Higham and Samuel D. Relton,
**How to Estimate the Largest Entries of a Matrix.**
MIMS EPrint..., 2015.
