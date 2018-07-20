[![autodiff](art/autodiff-header.svg)][autodiff]


# Overview

You most likely arrived here because:

- you have complicated functions for which you would like to compute derivatives;
- you understand that manually deriving and implementing the analytical expressions of these derivatives is a tedious and error-prone task;
- you are aware that numerical derivatives using finite differences can be inaccurate and inefficient; and
- you are looking for a straightforward solution to calculate derivatives in C++ using **automatic differentiation**.

Here is how [autodiff][autodiff] can help you. Let's assume you have the following code:

```c++
double x = 0.0;
double y = f(x);
```

for which you want to calculate `dydx`, that is, the derivative of output variable `y` with respect to input variable `x` when `x = 0.0`. Instead of using type `double` for `x` and `y`, as well as all for all intermediate variables inside `f`, if applicable, use `autodiff::var` instead:

```c++
var x = 0.0;
var y = f(x);
```

and calculate your derivative `dydx` using the `autodiff::grad` function as follows:

```cpp
double dydx = grad(y, x);
```

That's it -- simple and straightforward!

# Examples

The previous demonstration might be too simple for your needs. Below you find many complete C++ examples for different types of functions and circumstances.

## Example 1: Derivative of a single-variable function

The example below demonstrates how to use `autodiff` for calculating derivatives of a single-variable function.

~~~c++
// C++ includes
#include <iostream>
using namespace std;

// autodiff include
#include <autodiff.hpp>
using namespace autodiff;

// The single-variable function for which derivatives are needed
var f(var x)
{
    return 1 + x + x*x + 1/x + log(x);
}

int main()
{
    var x = 2.0;                         // x - input variable
    var y = f(x);                        // y - output variable

    double dydx = grad(y, x);            // evaluate the derivative dy/dx

    cout << "y = " << y << endl;         // print the evaluated output y
    cout << "dy/dx = " << dydx << endl;  // print the evaluated derivative dy/dx
}
~~~

After compiling and executing this example, you should see the following output:

~~~
y = 8.19315
dy/dx = 5.25
~~~

## Example 2: Derivatives of a multi-variable function

~~~c++
// C++ includes
#include <iostream>
using namespace std;

// autodiff include
#include <autodiff.hpp>
using namespace autodiff;

// The multi-variable function for which derivatives are needed
var f(var x, var y, var z)
{
    return 1 + x + y + z + x*y + y*z + x*z + x*y*z + exp(x/y + y/z);
}

int main()
{
    var x = 1.0;                         // x - input variable
    var y = 2.0;                         // y - input variable
    var z = 3.0;                         // z - input variable
    var u = f(x, y, z);                  // u - output variable

    double dudx = grad(u, x);            // evaluate the derivative du/dx
    double dudy = grad(u, y);            // evaluate the derivative du/dy
    double dudz = grad(u, z);            // evaluate the derivative du/dz

    cout << "u = " << u << endl;         // print the evaluated output u
    cout << "du/dx = " << dudx << endl;  // print the evaluated derivative du/dx
    cout << "du/dy = " << dudy << endl;  // print the evaluated derivative du/dy
    cout << "du/dz = " << dudz << endl;  // print the evaluated derivative du/dz
}
~~~

Executing this example produces:

~~~
u = 27.2113
du/dx = 13.6056
du/dy = 8.26761
du/dz = 5.28638
~~~

## Example 3: Derivatives with respect to function parameters

Sometimes, it is necessary to understand how sensitive an output is with respect to some parameters. For example, a mathematical model that computes the density of a substance at different temperatures and pressures will depend on some parameters. One might want to optimize the values of such parameters so that the model is accurate relative to some new experimental measurements. In such cases, the parameter optimization calculation can greatly benefit from available derivatives of the density with respect to every parameter in the model. 

~~~c++
// C++ includes
#include <iostream>
using namespace std;

// autodiff include
#include <autodiff.hpp>
using namespace autodiff;

// A type defining parameters for a function of interest
struct Params
{
    var a;
    var b;
    var c;
};

// The function that depends on parameters for which derivatives are needed
var f(var x, const Params& params)
{
    return params.a * sin(x) + params.b * cos(x) + params.c * sin(x)*cos(x);
}

int main()
{
    Params params;                       // initialize the parameter variables
    params.a = 1.0;                      // a - parameter variable
    params.b = 2.0;                      // b - parameter variable
    params.c = 3.0;                      // c - parameter variable

    var x = 0.5;                         // x - input variable
    var y = f(x, params);                // y - output variable

    double dydx = grad(y, x);            // evaluate derivative du/dx
    double dyda = grad(y, params.a);     // evaluate derivative du/da
    double dydb = grad(y, params.b);     // evaluate derivative du/db
    double dydc = grad(y, params.c);     // evaluate derivative du/dc

    cout << "y = " << y << endl;         // print evaluated output y
    cout << "dy/dx = " << dydx << endl;  // print evaluated derivative dy/dx
    cout << "dy/da = " << dyda << endl;  // print evaluated derivative dy/da
    cout << "dy/db = " << dydb << endl;  // print evaluated derivative dy/db
    cout << "dy/dc = " << dydc << endl;  // print evaluated derivative dy/dc
}

~~~

The example above introduces a new type `Params` with three data members of type `autodiff::var`: `a`, `b`, and `c`. These can be seen as parameters for the mathematical model function `f`. The example then demonstrates how to calculate not only the derivative of the output variable `y` with respect to input variable `x`, given by `dydx`, but also with respect to every parameter in `Params`: `dyda`, `dydb`, and `dydx`.

This example outputs the following results:
~~~
y = 3.4968
dy/dx = 1.53964
dy/da = 0.479426
dy/db = 0.877583
dy/dc = 0.420735
~~~

# What is missing?

1. Combine autodiff with C++ linear algebra library [Eigen][Eigen].

2. Evaluate the performance of autodiff for many functions with different complexity.

# How does it work?

`autodiff` works its magic in calculating *a posteriori* derivatives of **any `var` variable with respect to any `var` variable** by generating an **expression tree** of the entire evaluation procedure. For example, in the code below:

~~~c++
var x = 0.5;
var y = x * log(x)
~~~

the `var` variable `y` not only knows its value, given by `0.5 * log(0.5)`, but also how this value was computed step by step. This expression tree information that defines how a `var` variables is evaluated is stored in the data member `var::expr`. Every time a mathematical function (e.g., `sin`, `cos`, `log`, etc.) is applied to a `var` variable, or two or more such variables interact with each other arithmetically (e.g., operators `+`, `-`, `*`, `/`, etc.) a new expression tree is formed as a result by combining the expression trees of the participating intermediate variables. At some point, the output variable of interest has been fully evaluated, producing not only its actual value but alto the corresponding expression tree to calculate that value. 

The following features of C++ are extensively used in `autodiff:

- operator overloading
- inheritance; and 
- polymorphism.

Operator overloading is used to permit the use of arithmetic operators for variables of type `var`.

# License

MIT License

Copyright (c) 2018 Allan Leal

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

[autodiff]: https://github.com/autodiff/autodiff/releases "autodiff"
[Eigen]: http://eigen.tuxfamily.org "Eigen"
