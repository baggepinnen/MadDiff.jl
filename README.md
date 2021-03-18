[![build](https://github.com/sshin23/SimpleNLModels.jl/actions/workflows/test.yml/badge.svg)](https://github.com/sshin23/SimpleNLModels.jl/actions/workflows/test.yml)
[![codecov](https://codecov.io/gh/sshin23/SimpleNLModels.jl/branch/main/graph/badge.svg?token=U6NMMW0IT5)](https://codecov.io/gh/sshin23/SimpleNLModels.jl)
# SimpleNLModels.jl

SimpleNLModels.jl is a simple algebraic modeling/differentiation package.

## Installation
```julia
pkg> add https://github.com/sshin23/SimpleNLModels.jl.git
```

## Usage
### Nonlinear Expressions
SimpleNLModels provides a flexible user-interface for writing nonlinear expressions and evaluating the expressions and functions. For example,
```julia
using SimpleNLModels

x = Variable("x")
p = Parameter("p")
expr = x[1]^2 + exp(x[2]^p[1])/2 + log(x[3]+p[2])
print(expr) # x[1]^2 + exp(x[2]^2)*0.5 + log(x[3] + 0.5)

x0 = [0.,0.5,1.5]
p0 = [2,0.5]

f = func(expr)
print("f(x0,p0) = $(f(x0,p0))") # f(x0,p0) = 1.3351598889038159

d_2 = deriv(expr)[2]
print("d_2(x0,p0) = $(d_2(x0,p0))") # d_2(x0,p0) = 0.6420127083438707
```

### Nonlinear Programming
SimpleNLModels provide a simple user-interface for creating nonlinear prgogramming models and allows solving the created models with optimization solvers (Ipopt and MadNLP.jl). The syntax is as follows:
```julia
using SimpleNLModels, Ipopt

m = SimpleNLModels.Model(Ipopt.Optimizer; print_level=3) # SimpleModel( ... ) works as well

x = [variable(m; start=mod(i,2)==1 ? -1.2 : 1.) for i=1:1000]   
objective(m, sum(100(x[i-1]^2-x[i])^2+(x[i-1]-1)^2 for i=2:1000))
for i=1:998
    constraint(m, 3x[i+1]^3+2*x[i+2]-5+sin(x[i+1]-x[i+2])sin(x[i+1]+x[i+2])+4x[i+1]-x[i]exp(x[i]-x[i+1])-3 == 0)
end

instantiate!(m) # optional; this will make the model ready to be solved
optimize!(m)
```

## How it works?
SimpleNLModels constructs first and second derivative functions off-line (i.e., prior to calling the optimization solver) by applying operator overloading-based automatic differentiation on _functions_ rather than dual numbers. The exact _derivative functions_ can be obtained as results. This off-line procedure is typically more expensive than the model creation procedure in other AD-based algebraic modeling languages such as `JuMP` and `Ampl`, but enables faster first and second order derivative evaluations (thus is suitable for problems that require repeated solution of the same problem, such as control). A benchmark code is included in [here](https://github.com/sshin23/SimpleNLModels.jl/blob/main/benchmark/benchmark.jl).

<img src="/benchmark/output/luksanvlcek.png" width="400"/><img src="/benchmark/output/hehnandrea.png" width="400"/>

## Bug reports and support
Please report issues and feature requests via the [Github issue tracker](https://github.com/sshin23/SimpleNLModels.jl/issues).
