# Learning Algebraic Varieties
Welcome to the LearningAlgebraicVarieties package associated to the article

              Learning Algebraic Varieties from Samples

by P. Breiding, S. Kalisnik, B. Sturmfels and M. Weinstein.

To install the package, open a new `Julia` session and type
```julia
Pkg.clone("https://github.com/PBrdng/LearningAlgebraicVarieties.git")
```
After the installation is completed the command
```julia
using LearningAlgebraicVarieties
```
loads all the functions into the current session.

All functions accept m data points in ℝ^n or ℙ^(n-1) as an m×n matrix Ω; i.e., as `Array{T,2}` where `T<:Number`.

We provide some datasets in the [JLD](https://github.com/JuliaIO/JLD.jl.git) data format. They can be loaded into the session by typing
```julia
import JLD: load
datasets = load(string(Pkg.dir("LearningAlgebraicVarieties"), "/datasets.jld"))
```

## How to make dimension diagrams
Here is an example:
```julia
DimensionDiagrams(Ω, true)
```
plots the all dimension diagrams for the data in projective space. On the other hand,
```julia
DimensionDiagrams(Ω, false, methods = [:CorrSum, :BoxCounting], eps_ticks = 10)
```
plots the dimension diagrams CorrSum and BoxCounting for data in euclidean space. The estimates are computed for 10 values of ϵ between 0 and 1.

The complete syntax of the ``DimensionDiagrams`` function is as follows.
```julia
DimensionDiagrams(
    Ω::Array{T,2},
    projective::Bool;
    methods  = [:CorrSum, :BoxCounting, :PHCurve, :NPCA, :MLE, :ANOVA],
    eps_ticks = 25,
    fontsize = 16,
    lw = 4,
    log_log = false
    ) where {T <: Number}
```
Here:
* `Ω` is a matrix whose columns are the data points.
* `projective = false`: makes diagrams in euclidean space.
* `projective = true`: makes diagrams in projective space.
There are some optional arguments.
* `methods`: lists the dimension estimators to be plotted.
* `eps_ticks = k`: puts k evenly spaced ϵ into [0,1]. At those ϵs the dimensions are computed.
* `fontsize`: sets the font size of the axes.
* `lw`: sets the line width.
* `log_log = true`: makes a plot in the log-log scale.


## How to compute multivariate Vandermonde matrices
Here is an example.
```julia
MultivariateVandermondeMatrix(Ω, 2, true)
```
computes the multivariate Vandermonde matrix for the sample Ω and all monomials of degree  2. The `true` value determines homogeneous equations. On the other hand, the Vandermonde matrix with all monomials of degree at most 2 is computed by
```julia
MultivariateVandermondeMatrix(Ω, 2, false)
```
It is also possible to define the exponents involved. For example,
```julia
exponents = [[1,0,0], [1,1,1]]
MultivariateVandermondeMatrix(Ω, exponents)
```
computes the multivariate Vandermonde matrix for Ω ⊂ ℝ^3 and the monomials `x_1` and `x_1 x_2 x_3`.

Here is the full syntax
```julia
MultivariateVandermondeMatrix(Ω::Array{T},
                              d::Int64,
                              homogeneous_equations::Bool)

MultivariateVandermondeMatrix(data::Array{T},
                              exponents::Vector)
```
where
* `Ω` is a matrix whose colums are the data.
* `d` is the degree of the monomials.
* `homogeneous_equations = true` restricts the space of monomials to monomials of degree d.
* `homogeneous_equations = false` computes all monomials of degree at most d.
* `exponents` is an array of exponents vectors.

## How to find equations
Here is an example.
```julia
FindEquations(Ω, :with_svd, 2, true)
```
finds homogeneous equations of degree 2 using SVD to compute the kernel of the Vandermonde matrix, while
```julia
FindEquations(Ω, :with_qr, 3, false)
```
finds all polynomials of degree at most 3 and uses QR to compute the kernel of the Vandermonde matrix.

To find all equations with support `x_1 x_2` and `x_1^2` using the reduced row echelon form to compute the kernel, type
```julia
exponents = [[1,1], [2,0]]
FindEquations(Ω, :with_rref, exponents)
```

A multivariate Vandermonde matrix  may be passed to FindEquations:
```julia
M = MultivariateVandermondeMatrix(Ω, 2, false)
FindEquations(M, :with_svd, τ)
```
where τ is a tolerance value.


The full syntax of ``FindEquations`` is as follows.
```julia
FindEquations(Ω::Array{T,2},
              alg::Symbol,
              d::Int64,
              homogeneous_equations::Bool)
              where  {T<:Number}

FindEquations(Ω::Array{T,2},
              alg::Symbol,
              exponents::Array{Array{Int64,1},1})
              where  {T<:Number}

FindEquations(M::MultivariateVandermondeMatrix,
              alg::Symbol,
              τ::Float64)
```
Here:
* `Ω` is a matrix whose colums are the data points.
* `alg` is the algorithm that should be used (one of `:with_svd`, `:with_qr`, `:with_rref`).
* `d` is the degree of the equations.
* `homogeneous_equations = true` restricts the search space to homogeneous polynomials.
* `homogeneous_equations = false` computes all polynomials of degree at most d.
* `exponents` is an array of exponent vectors.
* `τ` is the tolerance value.

## Persistent homology
For computing barcodes in persistent homology we use the package [Eirene](https://github.com/Eetion/Eirene.jl). Until it has become an official package `LearningAlgebraicVarieties` contains the full source of `Eirene`. In future versions we will import the official `Eirene` package. We provide the `barcode_plot` functions that takes as input a dictionary `C` computed with the `eirene()` function. For details, see `Eirene`'s [online documentation](http://gregoryhenselman.org/eirene/documentation.html).

For example,
```julia
barcode_plot(C, [0,1,2], [5,5,5])
```
plots barcodes in dimensions 0, 1 and 2 and plots 5 bars in each dimension. The full syntax is as follows.
```julia
barcode_plot(C::Dict{String,Any},
             dims::Array{Int64,1},
             how_many_bars::Array{Int64,1}; sorted_by::Symbol=:lower_limit,
             lw=10,
             upper_limit = Inf,
             fontsize = 16)

```
Here
* `C` is the dictionary computes with the `eirene()` function.
* `dims` is an array that determins which dimensions should be displayed.
* `how_many_bars` is an array that specifies how many bars in each dimension should be plotted. The length of this array must coincide with the length of `dims`.
* `sorted_by` is either `:lower_limit` or `:length`. In the first case, the bars are sorted by their date of appearance, in the second case the bars are sorted by length.
* `lw` defines the width of the bars.
* `upper_limit` sets the upper limit of the x-axis in the plot. If `upper_limif = Inf`, the upper limit is set as twice the length of the second longest barcode.
* `fontsize` sets the font size of the axes.

## Distances
Computing distances is a key aspect in both dimension estimation and persistent homology. Here are the functions with which we compute distances.

To compute the scaled Fubini Study distances between the data points in Ω type
```julia
ScaledEuclidean(Ω)
```

On the other hand,to compute the scaled Fubini Study distances between the data points in Ω type
```julia
ScaledFubiniStudy(Ω)
```
Finally, the ellipsoid-driven complex is encoded in a distance matrix. It is computed by typing
```julia
EllipsoidDistances(Ω, f, λ)
```
where `f` is a vector of polynomials of the type provided by [DynamicPolynomials.jl](https://github.com/JuliaAlgebra/DynamicPolynomials.jl) and λ sets the ratio between the principal axes of the ellipsoids
