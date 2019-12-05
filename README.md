# FLOWMath

<!--
[![Stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://byuflowlab.github.io/FLOWMath.jl/stable)
[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://byuflowlab.github.io/FLOWMath.jl/dev)
-->
![](https://github.com/byuflowlab/FLOWMath.jl/workflows/Run%20tests/badge.svg)

*A collection of mathematical functions and convenience methods*

Examples of the available methods are shown below.  More examples are available in the test suite (/test/runtests.jl)

## Quadrature

### Trapezoidal Integration

```julia
x = range(0.0, pi+1e-15, step=pi/100)
y = sin.(x)

z = trapz(x, y)
```

## Root Finding 

### Brent's Method (1D functions)

```julia
f(x) = x^2 - 1.0

xstar = brent(f, -2.0, 0)
```

## Interpolation

### Akima Spline

```julia

x = 0:pi/4:2*pi
y = sin.(x)
xpt = 0:pi/16:2*pi

ypt = akima(x, y, xpt)
```

or if you plan to evaluate the spline repeatedly

```julia

spline = akima_setup(x, y)
ypt1 = akima_interp(xpt1, spline)
ypt2 = akima_interp(xpt2, spline)
```

### 2D/3D/4D Interpolation using Recursive 1D Interpolation

The functions `interp2d`, `interp3d`, and `interp4d` are generic and will accept any method that performs 1D interpolation as the first argument.  In the below examples, akima is used.  These examples are based off of examples from Matlab's interpn documentation.

2D:
```
x = -5.0:5.0
y = -5.0:5.0
z = zeros(11, 11)
for i = 1:11
    for j = 1:11
        v = sqrt(x[i]^2 + y[j]^2) + 1e-15
        z[i, j] = sin(v) / v
    end
end

xpt = range(-5.0, 5.0, length=100)
ypt = range(-5.0, 5.0, length=100)

zpt = interp2d(akima, x, y, z, xpt, ypt)

```

4D:
```
x = -1:0.2:1
y = -1:0.2:1
z = -1:0.2:1
t = 0:2:10.0

nx = length(x)
ny = length(y)
nz = length(z)
nt = length(t)

f = Array{typeof(x[1])}(undef, nx, ny, nz, nt)

for i = 1:nx
    for j = 1:ny
        for k = 1:nz
            for l = 1:nt
                f[i, j, k, l] = t[l]*exp(-x[i]^2 - y[j]^2 - z[k]^2)
            end
        end
    end
end

xpt = -1:0.05:1
ypt = -1:0.08:1
zpt = -1:0.05:1
tpt = 0:0.5:10.0

fpt = interp4d(akima, x, y, z, t, f, xpt, ypt, zpt, tpt)
```

## Derivate Estimation

We highly recommend using algorithmic differentiation (AD) and analytic sensitivity methods.  The below methods are useful for quick and dirty prototyping and/or to check derivatives (particularly w/ complex step).

```julia
# 2 inputs, 3 outputs
g(x) = [exp(x[1]) / sqrt(sin(x[1])^3 + cos(x[2])^3); sin(x[2]); cos(x[1])]

x = [1.5, 0.5]
dff = forwarddiff(g, x)
dfb = backwarddiff(g, x)
dfc = centraldiff(g, x)
dfcplx = complexstep(g, x)
```

## Smoothing

### Absolute value

The absolute value function is not differentiable at x = 0.  The below function smoothly adds a small quadratic function in place of the cusp with a half-width given by `delta_x`.  This small rounding at the bottom can prevent numerical issues with gradient-based optimization.

```julia
y = abs_smooth(x, delta_x)
```