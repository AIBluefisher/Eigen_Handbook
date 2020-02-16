# 2. TLDR
* Never pass an eigen-type by value.
* Always use [Eigen::aligned_allocator<T>](https://eigen.tuxfamily.org/dox/classEigen_1_1aligned__allocator.html) when using a fixed size eigen-type with a std::container.
* Use EIGEN_MAKE_ALIGNED_OPERATOR_NEW on classes/structs with fixed size eigen-type elements.
* Be very careful when assigning to memory you are operating on.
* Always check the matrices are the right size and their memory is correctly allocated.
* Don't mix Eigen versions


# 3. Possible symptoms

## Everything builds and runs correctly

Chances are you are here because someone saw that you had one of the issues mentioned below. So if everything is working, why should you care?

* **Chances are your code is currently very fragile:** Issues with alignment can be caused or solved simply by changing the order variables are declared in. Similarity issues with Eigen/StdVector can be caused by the ordering of includes. This means even the smallest cosmetic change can stop your code building or make it start segfaulting.

* **There is a good chance that your code won't work on a subtly different system:** Many alignment issues that do not cause problems on Ubuntu 14.04, immediately segfault things in Ubuntu 15.04.

* **Your code might be broken in a way that has so far gone unnoticed:**
For example, when two mismatched Eigen versions were used with manifold mapping the only symptom was that a Boolean initialized to true would change its value to false.

## Everything builds and runs correctly on one OS, but segfaults/crashes on another.

* **Most likely issue:** [memory misalignment](#memory-misalignment)
* **Possible issues:** [mismatched eigen versions](#mismatched-eigen-versions)

## Everything builds and runs correctly on one OS, but fails to build on another.

* **Possible issues:** [mixed use of StdVector](#mixed-use-of-StdVector)

## Segfaults/crashes

* **Most likely issue:** [memory misalignment](#memory-misalignment)
* **Possible issues:** [manual memory assignment](#manual-memory-assignment),  [incorrectly sized matrices](#incorrectly-sized-matrices), [mismatched Eigen versions](#mismatched-eigen-versions) or [mismatched build flags](#mismatched-build-flags)

## The contents of some/all of an Eigen Matrix is garbage

* **Most likely issue:** [incorrectly sized matrices](#incorrectly-sized-matrices) or [destructive eigen operations](#destructive-eigen-operations)
* **Possible issues:** everything

## Compiler error about partial specialization of std::vector:

* **Issue:** [mixed use of StdVector](#mixed-use-of-StdVector)

## There are issues in non-Eigen elements / the program appears to defy the core principles of the C++ language:

This covers everything from a non-Eigen variable changing its value between being read and written, to multiplying two variables causing an infinite loop. The behavior is usually highly dependent on the exact system.

* **Most likely issue:** [mismatched Eigen versions](#mismatched-eigen-versions)
* **Possible issues:** [memory misalignment](#memory-misalignment) or [mismatched build flags](#mismatched-build-flags)

# Possible underlying issues:

## Memory misalignment:

[See here for the official Eigen docs on this issue](https://eigen.tuxfamily.org/dox/group__TopicUnalignedArrayAssert.html)

This is the most common issue with Eigen. As a quick check for this issue try disabling all Eigen vectorization (add `#define EIGEN_DONT_VECTORIZE` and `#define EIGEN_DISABLE_UNALIGNED_ARRAY_ASSERT`) and recompile. 
If this fixes your problem, this was your issue. However, disabling vectorization will tank your codes efficiency so read on to find out how to fix it correctly.

Note: This issue appears very OS dependent, with OSX seeming the most robust to it and Ubuntu 16.04 the least. It will usually cause a segfault if encountered. The following changes are only actually required if you are using fixed Eigen types that have a number of bytes divisible by 16, however it doesn't hurt and it is easier just to do it everywhere an Eigen type shows up.

### You must use `Eigen::aligned_allocator` whenever you use a std container

* `std::vector<T>` -> `std::vector<T, Eigen::aligned_allocator<T>>`
* `std::map<A, B>` -> `std::map<std::pair<A,B>, Eigen::aligned_allocator<std::pair<A,B>>>`
* `std::queue<T>` -> `std::queue<T, std::deque<T, Eigen::aligned_allocator<T>>`
* etc...

### You cannot pass Eigen types by value:

Change to passing by reference and then make an internal copy if you need to modify things
`void func(Eigen::Vector3f vec){ ...` -> `void func(const Eigen::Vector3f& vec_in){ Eigen::Vector3f vec = vec_in; ...`

### Structs and Classes with fixed size Eigen variables need EIGEN_MAKE_ALIGNED_OPERATOR_NEW 
If you have a class or struct with a raw Eigen type as a variable (containers holding Eigen types don't count) you must add the EIGEN_MAKE_ALIGNED_OPERATOR_NEW macro in the public part.

`struct Foo{ Eigen::Vector3f vecA; Eigen::Vector3f vecB};` ->  `struct Foo{EIGEN_MAKE_ALIGNED_OPERATOR_NEW Eigen::Vector3f vecA; Eigen::Vector3f vecB}`;

## Mismatched Eigen versions:

The use of a combination of the system Eigen and eigen_catkin means it is possible for a program to be built against one version of Eigen and then linked with another. If this happens there is no limit to the number or nature of strange issues that can appear.

## Mixed use of StdVector:

Before C++11 how the memory alignment of `std::vector` was handled meant that it could not be used with Eigen types. To get around this Eigen defined its own `std::vector` in `Eigen/StdVector`. Since C++11 this is no longer needed and so most libraries do not use it.
However, this causes an error complaining about the partial specialization of `std::vector` if `std::vector` has been used to hold an Eigen type before `Eigen/StdVector` is included. This issue is only present in Eigen versions before 3.3. 

To solve it there are several options:
* upgrade to a newer Eigen
* add `#include<Eigen/StdVector>` before any `std::vector` is used (including inside other includes)
* remove `#include<Eigen/StdVector>` everywhere you find it (many libraries such as pcl use it internally, so this may not be possible)
* [patch Eigen](https://github.com/ethz-asl/eigen_catkin/blob/master/StdVector.patch) 

## Incorrectly sized matrices:

When compiled in release mode Eigen will happily perform such operations as

    Eigen::MatrixXd mat_a(Eigen::MatrixXd::Random(100,100));
    Eigen::MatrixXd mat_b(Eigen::MatrixXd::Random(5,5));
    mat_a = mat_a * mat_b;

Clearly the matrix sizes make this operation impossible and it is not clear what Eigen is actually doing, though for some matrix sizes this does not cause a segfault or any warnings / errors. This sort of thing can be difficult to spot when one matrix is a transpose of what it is meant to be.

## Destructive Eigen operations:

Some operations can cause Eigen to reallocate a matrix's memory, potentially changing the values of the matrix. [::resize](https://eigen.tuxfamily.org/dox/classEigen_1_1PlainObjectBase.html#a9fd0703bd7bfe89d6dc80e2ce87c312a) is an example of such a function.

Sometimes this issue can be subtle for example:

`Eigen::MatrixXf mat(Eigen::MatrixXf::Random(100,100));
mat = mat.block(0,0,50,50);`

In the above example some of the elements of `mat` may obtain erroneous values as the operation overwrites some values before it reads from them. 

## Manual memory assignment:
Behind the scenes Eigen stores all the matrix data in what is basically a raw c array. While this is usually handled internally, it is possible to access and manipulate this data directly. You can even create your own memory which an Eigen matrix can then use. However, as Eigen performs no checks that this memory is valid this can cause your program to segfault or have other ill defined behavior if the memory has not been allocated correctly.

As an example of this, the graph optimization software g2o uses Eigen matrices to store the hessian of each vertex. However, while the hessians are created with the vertex, the matrix's internal data pointer is set to NULL until after the first optimization. This means that while operations such as `.cols()` are valid, attempting to access an element will cause a segfault.

## Mismatched build flags:
Eigen supports different kinds of vectorization instructions to speed up computations. Different vectorization instructions expect different alignment: e.g. SSE instructions work on 16-byte-aligned data whereas AVX instructions (supported in Eigen >=3.3) expects 32-byte-aligned data. These instructions are enabled with compiler flags named `-m*`: `-msse` would enable SSE instructions for the current program and `-mavx` enables AVX instructions. With `-march=native`, all instructions that are supported by your CPU are enabled.

Linking programs together with different vectorization instructions enabled can lead to all kinds of weird segfaults. These segfaults are commonly caused by Eigen expecting a different alignment and using the wrong vectorization functions.

Example: You're using Eigen 3.3 package is compiled with only `-msse` and linked against a package that was compiled with `-msse -mavx`. Now somewhere in your code, the program may segfault because it tries to use AVX instructions (which need 32-byte-aligned data) on your 16-byte-aligned matrices.

To fix these issues, make sure that all your packages (including third-party libraries) are compiled with the same flags. Especially pay attention to the flags related to these vectorization instructions `-m*` or `-march`.