# 2. Basic Operations

## 2.1 Using Eigen with CMake
### Install
```sh
sudo apt-get install libeigen3-dev
```
### Finding Eigen by CMake
```cmake
find_package(eigen3 REQUIRED)
```

## 2.2 [Eigen Types](http://eigen.tuxfamily.org/dox-3.2/group__QuickRefPage.html#QuickRef_Types)

The Eigen library is divided in a Core module and several additional modules. Each module has a corresponding header file which has to be included in order to use the module. The Dense and Eigen header files are provided to conveniently gain access to several modules at once.

| Module | Header file | Contents |
| :-: | :-: | :- |
| Core | `#include <Eigen/Core>` | Matrix and Array classes, basic linear algebra (including triangular and selfadjoint products), array manipulation |
| Geometry | `#include <Eigen/Geometry>` | Transform, Translation, Scaling, Rotation2D and 3D rotations (Quaternion, AngleAxis) |
| LU | `#include <Eigen/LU>` | Inverse, determinant, LU decompositions with solver (FullPivLU, PartialPivLU) |
| Cholesky | `#include <Eigen/Cholesky>` | 	LLT and LDLT Cholesky factorization with solver |
| Householder | `#include <Eigen/Householder>` | 	Householder transformations; this module is used by several linear algebra modules |
| SVD | `#include <Eigen/SVD>` | SVD decomposition with least-squares solver (JacobiSVD) |
| QR | `#include <Eigen/QR>` | QR decomposition with solver (HouseholderQR, ColPivHouseholderQR, FullPivHouseholderQR) |
| Eigenvalues | `#include <Eigen/Eigenvalues>` | Eigenvalue, eigenvector decompositions (EigenSolver, SelfAdjointEigenSolver, ComplexEigenSolver) |
| Sparse | `include <Eigen/Sparse>` | Sparse matrix storage and related basic linear algebra (SparseMatrix, DynamicSparseMatrix, SparseVector) |
| | `#include <Eigen/Dense>` | Includes Core, Geometry, LU, Cholesky, SVD, QR, and Eigenvalues header files|
| | `#include <Eigen/Eigen>` | Includes Dense and Sparse header files (the whole Eigen library) |


### Array, matrix and vector types

Eigen provides two kinds of dense objects: mathematical matrices and vectors which are both represented by the template class Matrix, and general 1D and 2D arrays represented by the template class Array:

```C++
typedef Matrix<Scalar, RowsAtCompileTime, ColsAtCompileTime, Options> MyMatrixType;
typedef Array<Scalar, RowsAtCompileTime, ColsAtCompileTime, Options> MyArrayType;
```

- `Scalar` is the scalar type of the coefficients (e.g., float, double, bool, int, etc.).
- `RowsAtCompileTime` and `ColsAtCompileTime` are the number of rows and columns of the matrix as known at compile-time or Dynamic.
- `Options` can be ColMajor or RowMajor, default is ColMajor. (see class Matrix for more options)
All combinations are allowed: you can have a matrix with a fixed number of rows and a dynamic number of columns, etc. The following are all valid:

```C++
Matrix<double, 6, Dynamic>                  // Dynamic number of columns (heap allocation)
Matrix<double, Dynamic, 2>                  // Dynamic number of rows (heap allocation)
Matrix<double, Dynamic, Dynamic, RowMajor>  // Fully dynamic, row major (heap allocation)
Matrix<double, 13, 3>                       // Fully fixed (usually allocated on stack)
```

**Note:** *The use of fixed-size matrices and vectors has two advantages. The compiler emits better (faster) code because it knows the size of the matrices and vectors. Specifying the size in the type also allows for more rigorous checking at compile-time. For instance, the compiler will complain if you try to multiply a Matrix4d (a 4-by-4 matrix) with a Vector3d (a vector of size 3). However, the use of many types increases compilation time and the size of the executable. The size of the matrix may also not be known at compile-time. A rule of thumb is to use fixed-size matrices for size 4-by-4 and smaller.*

Eigen has predefined some simplified types for matrices and vectors, for example
```C++
using Matrix<double, 2, 1> = Vector2d;
using Matrix<double, 3, 1> = Vector3d;
using Matrix<double, 3, 3> = Matrix3d;
using Matrix<double, Dynamic, Dynamic> = MatrixXd;
using Matrix<double, Dynamic, 1> = VectorXd;
...
```


