# dCpp
Infinitely-differentiable C++; conditionals, loops, recursion and all things C++

###Abstract
We provide an illustrative implementation of an analytic, infinitely-differentiable machine, implementing infinitely-differentiable programming spaces and operators acting upon them, as constructed in the paper _Operational calculus on programming spaces and generalized tensor networks_. Implementation closely follows theorems and derivations of the paper, intended as an educational guide.

This is the openSource version.

###Theory

From _abstract_ of the paper  _Operational calculus on programming spaces and generalized tensor networks_ in which the theory is derived by [Žiga Sajovic](https://www.linkedin.com/in/zigasajovic):

In this paper, we develop the theory of analytic virtual machines, that
implement analytic programming spaces and operators acting upon them. Such a machine fully integrates control structures, reataining the expressive freedom of algorithmic control flow.

A programming space is a subspace of the function space of maps on the virtual
memory. We can construct a differential operator on programming spaces as we 
extend the virtual memory to a tensor product of a virtual space with tensor algebra
of its dual. Extended virtual memory serves by itself as an algebra of programs, giving the expansion of the original program as an infinite tensor series at
program's input values. 

A paper explaining implementation of this theory is avaliable [/paper/dCpp.pdf](https://zigasajovic.github.io/dCpp/paper/dCpp.pdf).

The paper _Operational calculus on programming spaces and generalized tensor networks_ , containing construction of the theory will soon be available on arXiv.


###Usage
By employing analytic virtual machines, we can construct analytic procedures, viewing algorithms in a new light. One can start incorporating variable parameters into algorithm design, revealing the true nature of hyper-parameters often used in practice.

###Tutorial
As most programmers face the need of differentiability through machine learning, we use the concept a [Recurrent neural network](https://en.wikipedia.org/wiki/Recurrent_neural_network) employing [logistic regression](https://en.wikipedia.org/wiki/Logistic_regression) with [softmax normalization](https://en.wikipedia.org/wiki/Softmax_function#Softmax_Normalization) as a vessel for this tutorial. We demostrate, how it is simply constructed using algorithmic control flow and reccursion.

First we include the necessities

```c++
#include <iostream>
#include <dCpp.h>
#include <vector>
```

We initialize a n-differentiable programming space (order is arbitrary here)

```c++
using namespace dCpp;
int n_differentiable=2;
initSpace(n_differentiable);
```

We will need the folowing functions
* [sigmoid(x)](https://en.wikipedia.org/wiki/Sigmoid_function)
* [softmax(vec)](https://en.wikipedia.org/wiki/Softmax_function)
* [dotProduct(vec1,vec2)](https://en.wikipedia.org/wiki/Dot_product)
* [matVecProduct(mat,vec)](https://en.wikipedia.org/wiki/Matrix_multiplication)

By coding sigmoid(x), we will learn about creating differentiable maps, constructable using the differentiable programming space _dCpp_ and the algebra of the virtual memory _var_.
First we create maps double->double, for e(x) and its' derivative.
```c++
var sigmoidMap(const var&v){return 1/(1+exp(-1*v));};

```

We test it out and display all first and second derivatives.

```c++
//  set inputs
    double x=4;
    double y=2;
//  set weights
    var w_1(0.4);
    var w_2(0.6);
//  initialize weights as twice differentiable variables
    dCpp::init(w_1);
    dCpp::init(w_2);
//  now we use sigmoid map as a differentiable map
    var f=sigmoidMap(w_1*x+w_2*y);
//  df/dx
    std::cout<<"df/dw_1 = "<<f.d(&w_1).id<<std::endl;
//  df/dw_2
    std::cout<<"df/dw_2 = "<<f.d(&w_2).id<<std::endl;
//  df/dw_1dw_1
    std::cout<<"df/dw_1dw_1 = "<<f.d(&w_1).d(&w_1).id<<std::endl;
//  df/dw_1dw_2
    std::cout<<"df/dw_1dw_2 = "<<f.d(&w_1).d(&w_2).id<<std::endl;
//  df/dw_2dw_1
    std::cout<<"df/dw_2dw_1 = "<<f.d(&w_2).d(&w_1).id<<std::endl;
//  df/dw_2dw_2
    std::cout<<"df/dw_2dw_2 = "<<f.d(&w_2).d(&w_2).id<<std::endl;
```

 Similarly, we could have used the operator [tau](include/tau.h) by coding , which allows one to create it's own elements of the differentiable programming space _dCpp_, returning a differentiable variable [var](/include/var.h).
```
By coding the softmax normalization, we reveal how analytic differentiable machines fully integrate control structures.
```c++
//simply code the map existing in the programming space dCpp
//and the belonging algebra
std::vector<var> softmax(const std::vector<var>& V){
    std::vector<var> out;
    var sum(0);
    init(sum);
    for(var v:V){
        sum=sum+exp(v);
    }
    for(var v:V){
        out.push_back(exp(v)/sum);
    }
    return out;
}

```
We test it, by inititalizing a four-differentiable programming space and displaying all derivatives.

```c++
//  initiaize Virtual memory of fourth order
    initSpace(4);
//get a vector of variables
    int size=2;
    std::vector<var> vars;
    for(int i=1;i<=size;i++){
        var tmp=var(i);
        init(tmp);
        vars.push_back(tmp);
    }
//  use the softmax function
    std::vector<var> f=softmax(vars);
//  display derivatives of all four orders
//   of one of the components
    f[1].print();

```

Assume existence of functions _vecSum_, _matVecProd_, _genRandVec_ and _forAll_ written in a similar fashion. Thus, we have all the tools needed to build a recursive layer. It will consist of two layers, mapping a 2-vector to a 2-vector. Output of the second layer will be recursively connected to the input of the next recursive layer.

For brevity, we denote _std::vector<std::vector<var> >_ by _mat_ and _std::vector<var>_ by _vec_.

```c++
vec recursionNet(vec input, mat weights[2],vec bias[], int depth){
    if(depth==0){
        return softmax(input);
    }
    else{
        vec firstOut;
        //matrix vector multiplication
        firstOut=matVecProd(weights[0],input);
        firstOut=vecSum(firstOut,bias[0]);
        forAll(firstOut,sigmoid);
        vec secondOut;
        secondOut=matVecProd(weights[1],firstOut);
        secondOut=vecSum(secondOut,bias[1]);
        forAll(secondOut,sigmoid);
        return recursionNet(secondOut,weights, bias,depth-1);
    }
}
```

Now only some initialization of weights is needed and the network can be used, exactly like any other function would, with the exception, that this function is _n-differentiable_.

```c++
vec output = recursionNet(input,weights[],bias[], depth);
for(var v:output)v.print();
```

to display derivatives of all orders, upt to _n_ by which the space has been initialized.

###External libraries

Usage with external libraries written in generic paradigm is demonstrated on the example of [Eigen](http://eigen.tuxfamily.org/). 
We will code a perceptron with sigmoid activations, followed by softmax normalization, taking 28x28 image as an input and outputting a 10 class classifier. We will use dCpp provided mappings in the _dEigen_ header.

```c++
#include <iostream>
#include <dCpp.h>
#include <dEigen.h>
using namespace std;
using namespace dC;

//create a softmax function
template <typename Derived>
    void softmax(Eigen::MatrixBase<Derived>& matrix){
            //maps each element of the matrix by y=e^x;
            dC::map_by_element(matrix,&dC::exp);
            //sums the elements of the matrix using Eigens function
            var tmp=matrix.sum();
            //divides each element by the sum
            for (size_t i=0, nRows=matrix.rows(), nCols=matrix.cols(); i<nCols; ++i)
                for (size_t j=0; j<nRows; ++j){
                    matrix(j,i)/=tmp;
                }
}

int main(){
    //    Matrix holding the inputs (imgSizeX1 vector)
    const int imgSize=28*28;
    const Eigen::Matrix<var,1,imgSize>input=Eigen::Matrix<var,1,imgSize>::Random(1,imgSize);
    //    number of outputs of the layer
    const int numOfOutOnFirstLevel=10;
    //    matrix of weights on the first level (imgSizeXnumOfOutOnFirstLevel)
    Eigen::Matrix<var,imgSize,numOfOutOnFirstLevel>firstLayerVars=Eigen::Matrix<var,imgSize,numOfOutOnFirstLevel>::Random(imgSize,numOfOutOnFirstLevel);
    //    initializing weights
    dC::init(firstLayerVars);
    //    mapping of the first layer --> resulting in 10x1 vector
    Eigen::Matrix<var,numOfOutOnFirstLevel,1>firstLayerOutput=input*firstLayerVars;
    //    apply sigmoid layer --> resulting in 10x1 vector
    dC::map_by_element(firstLayerOutput,&dC::sigmoid);
    //    apply sofmax layer --> resulting in 10x1 vector
    softmax(firstLayerOutput);
    //    display the first output layer and its Jaccobian
    //    Jacobian is a 10x7840 matrix of derivatives
    for (size_t i=0, nRows=firstLayerOutput.rows(), nCols=firstLayerOutput.cols(); i<nCols; ++i){
                for (size_t j=0; j<nRows; ++j) {
                    dC::print(firstLayerOutput(j,i));
                }
                cout<<endl;
    }
}

```

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">dC++</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="https://si.linkedin.com/in/zigasajovic" property="cc:attributionName" rel="cc:attributionURL">Žiga Sajovic</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
