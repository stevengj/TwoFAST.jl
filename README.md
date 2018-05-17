# TwoFAST

[![Build Status](https://travis-ci.org/hsgg/TwoFAST.jl.svg?branch=master)](https://travis-ci.org/hsgg/TwoFAST.jl)
[![Coverage Status](https://coveralls.io/repos/hsgg/TwoFAST.jl/badge.svg?branch=master&service=github)](https://coveralls.io/github/hsgg/TwoFAST.jl?branch=master)
[![codecov.io](http://codecov.io/github/hsgg/TwoFAST.jl/coverage.svg?branch=master)](http://codecov.io/github/hsgg/TwoFAST.jl?branch=master)
[![Build status](https://ci.appveyor.com/api/projects/status/7ew7w7qvo2m724yc?svg=true)](https://ci.appveyor.com/project/hsgg/twofast-jl)
[![Build status](https://ci.appveyor.com/api/projects/status/7ew7w7qvo2m724yc/branch/master?svg=true)](https://ci.appveyor.com/project/hsgg/twofast-jl/branch/master)

The 2-FAST (*2-point function from Fast and Accurate Spherical bessel
Transform*) algorithm is implemented here in the [Julia](https://julialang.org)
programming language.

The algorithm is documented in the paper [Fast and Accurate Computation of
Projected Two-point functions](https://arxiv.org/abs/1709.02401).


## Minimal example

Load the module:

```julia
    using TwoFAST
```

For both minimal examples we need a power spectrum. Get it like so:

```julia
    using Dierckx
    d = readdlm("test/data/planck_base_plikHM_TTTEEE_lowTEB_lensing_post_BAO_H070p6_JLA_matterpower.dat")
    pk = Spline1D(d[:,1], d[:,2])
```

To calculate the real-space correlation function, use

```julia
    N = 1024    # number of points to use in the Fourier transform
    kmax = 1e3  # maximum k-value
    kmin = 1e-5 # minimum k-value
    r0 = 1e-3   # minimum r-value (should be ~1/kmax)

    print("ξ(r), ℓ=0, ν=0: ")
    r00, xi00 = xicalc(pk, 0, 0; N=N, kmin=kmin, kmax=kmax, r0=r0)

    print("ξ(r), ℓ=0, ν=-2:")
    r, xi0m2 = xicalc(pk, 0, -2; N=N, kmin=kmin, kmax=kmax, r0=r0)
```

To calculate the wl terms, we first need to calculate the Fourier kernels at
the highest needed ℓ, and store the result in a file. The file can be specified
as the third parameter to `make_fell_lmax_cache()`. Then, we generate the
*Mll*-cache, which is also stored to a file, which can be specified with the
`outfile` option to `calcMljj()`. The input file generated by
`make_fell_lmax_cache()` can be specified by passing the `fell_lmax_file`
option to `calcMljj()`. Finally, to actually calculate the *wljj*-terms we call
the function `calcwljj()`. However, to store the *wljj*-terms, we need to
create the arrays, and write a function (`outfunc()`) that will store them in
the arrays. The function `outfunc()` will be called for each ℓ in the array
`ell`. Here's the example:

```julia
    N = 4096
    chi0 = 1e-3
    kmin = 1e-5
    kmax = 1e3
    q = 1.1
    ell = [42]  # only ell=42 for this run
    RR = [0.6, 0.7, 0.8, 0.9, 1.0]

    # calculate M_ll at high ell, result gets saved to a file:
    make_fell_lmax_cache(RR, maximum(ell); N=N, q=q, G=log(kmax / kmin), k0=kmin, r0=chi0)

    # calculate all M_ll, result gets saved to a file:
    tt = calcMljj(RR; ell=ell, kmin=kmin, kmax=kmax, N=N, r0=chi0, q=q)

    # calculate wljj:
    w00 = Array{Float64}(N, length(RR))
    w02 = Array{Float64}(N, length(RR))
    function outfunc(wjj, ell, rr, RR)
        if ell == 42
            w00[:,:] = wjj[1]
            w02[:,:] = wjj[2]
        end
    end
    rr = calcwljj(pk, RR; ell=ell, kmin=kmin, kmax=kmax, N=N, r0=chi0, q=q, outfunc=outfunc)
```
