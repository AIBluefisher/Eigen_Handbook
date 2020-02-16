# [Basic Operations](http://eigen.tuxfamily.org/dox-3.2/group__QuickRefPage.html#title5)

## 1. Initialization

We have studied how to declare a matrix/vector with eigen. Now we need to learn how to
initialize a matrix. There are many ways we can initialize a matrix.
```C++
#include <iostream>
#include <Eigen/Dense>

using namespace Eigen;

int main()
{
    MatrixXd A = MatrixXd::Random(3, 3);        // a 3-by-3 matrix initialized randomly with all values in [-1, 1].
    MatrixXd B = MatrixXd::Constant(3, 3, 1.0); // a 3-by-3 matrix with all coefficients=1.0.

    std::cout << "A: " << A << std::endl;
    std::cout << "B: " << B << std::endl;
}
```

### 1.1 The Comma Initializer
Eigen offers a comma initializer syntax which allows the user to easily set all the coefficients of a matrix, vector or array. Simply list the coefficients, starting at the top-left corner and moving from **left to right** and from the **top to the bottom**. **The size of the object needs to be specified beforehand.** If you list too few or too many coefficients, Eigen will complain. Comma initializer can be used to initialize a matrix with the overloaded C++ `<<` operator.
```C++
#include <iostream>
#include <Eigen/Dense>
using namespace Eigen;

int main()
{
    VectorXd v(3);
    v << 1.0 << 2.0 << 3.0;

    Matrix3f m;
    m << 1.0 << 2.0 << 3.0
      << 4.0 << 5.0 << 6.0
      << 7.0 << 8.0 << 9.0;

    std::cout << "v: " << v << std::endl;
    std::cout << "m: " << m << std::endl;
}
```

The comman initilizer is useful for join multiple vectors or matrices. i,e.
```C++
#include <iostream>
#include <Eigen/Dense>
using namespace Eigen;

int main()
{
    RowVectorXd vec1(3);
    vec1 << 1, 2, 3;
    std::cout << "vec1 = " << vec1 << std::endl;
    RowVectorXd vec2(4);
    vec2 << 1, 4, 9, 16;;
    std::cout << "vec2 = " << vec2 << std::endl;
    RowVectorXd joined(7);
    joined << vec1, vec2;
    std::cout << "joined = " << joined << std::endl;

    // Initialize matrices with block structure.
    MatrixXf A(2, 2);
    matA << 1, 2, 3, 4;
    MatrixXf B(4, 4);
    matB << A, A/10, A/10, A;
    std::cout << B << std::endl;
}
```

The comma initializer can also be used to fill block expressions, i,e.
```C++
#include <iostream>
#include <Eigen/Dense>
using namespace Eigen;

int main()
{
    Matrix3f m;
    m.row(0) << 1, 2, 3;
    m.block(1,0,2,2) << 4, 5, 7, 8;
    m.col(2).tail(2) << 6, 9;                   
    std::cout << m;
}
```

### 1.2 Special matrices and arrays
The `Matrix` and `Array` classes have static methods like `Zero()`, which can be used to initialize all coefficients to zero. There are three variants. The first variant takes no arguments and can only be used for fixed-size objects. If you want to initialize a dynamic-size object to zero, you need to specify the size. Thus, the second variant requires one argument and can be used for one-dimensional dynamic-size objects, while the third variant requires two arguments and can be used for two-dimensional objects.
```C++
#include <iostream>
#include <Eigen/Dense>
using namespace Eigen;

int main()
{
    Matrix3d zero_mat1 = Matrix3d::Zero();
    MatrixXd zero_mat2 = MatrixXd::Zero(3, 4);
    Matrix3d identity_mat1 = Matrix3d::Identity();
    MatrixXd identity_mat2 = MatrixXd::Identity(3, 4);
    // The same way for vectors.
    Vector3d zero_vec1 = Vector3d::Zero();
    // ...

    std::cout << "A fixed-size array:\n";
    Array33f a1 = Array33f::Zero();
    std::cout << a1 << "\n\n";
    std::cout << "A one-dimensional dynamic-size array:\n";
    ArrayXf a2 = ArrayXf::Zero(3);
    std::cout << a2 << "\n\n";
    std::cout << "A two-dimensional dynamic-size array:\n";
    ArrayXXf a3 = ArrayXXf::Zero(3, 4);
    std::cout << a3 << "\n";
}
```

## 2. Access matrices/vectors elements

Eigen also provides many functions we can utilize to access the elements of matrices/vectors
conveniently, like `topLeftCorner(row_index, col_index)`, `topRightCorner(row_index, col_index)`,
`bottomLeftCorner(row_index, col_index)`, `bottomRightCorner(row_index, col_index)` and we can directly access each element by `mat(row_index, col_index)`.
```C++
#include <iostream>
#include <Eigen/Dense>
using namespace Eigen;

template <typename T>
void OutputEigenMat(const T& mat)
{
    const int m = mat.rows();
    const int n = mat.cols();

    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            std::cout << mat(i, j) << " ";
        }
        std::cout << "\n";
    }
}

int main()
{
    const int size = 6;
    MatrixXd mat1(size, size);

    // Read-write access to a column or a row of a matrix (or array).
    mat1.row(i) = mat2.col(j);
    mat1.col(j1).swap(mat1.col(j2));

    // Read-write access to sub-vectors.
    // 1. Default versions.
    vec1.head(n);         // the first n coefficients.
    vec1.tail(n);         // the last n coefficients.
    vec1.segment(pos, n); // the n coefficients in the range [pos : pos + n - 1]
    // 2. Optimized versions when the size is known at compile time.
    vec1.head<n>();       // the first n coefficients.
    vec1.tail<n>();       // the last n coefficients.
    vec1.segment<n>(pos); // the n coefficients in the range [pos : pos + n - 1]

    // Read-write access to sub-matrices.
    mat1.block(i, j, rows, cols); // the rows x cols sub-matrix starting from position (i,j).
    mat1.block<rows,c ols>(i,j);

    // the rows x cols sub-matrix taken in one of the four corners.
    mat1.topLeftCorner(size/2, size/2)     = MatrixXd::Zero(size/2, size/2);
    mat1.topRightCorner(size/2, size/2)    = MatrixXd::Identity(size/2, size/2);
    mat1.bottomLeftCorner(size/2, size/2)  = MatrixXd::Identity(size/2, size/2);
    mat1.bottomRightCorner(size/2, size/2) = MatrixXd::Zero(size/2, size/2);
    // std::cout << mat1 << std::endl << std::endl;
    OutputEigenMat(mat1);

    // specialized versions of block() when the block fit two corners.
    mat1.topRows(rows);     mat1.topRows<rows>();
    mat1.bottomRows(rows);  mat1.bottomRows<rows>();
    mat1.leftCols(cols);    mat1.leftCols<cols>();
    mat1.rightCols(cols);   mat1.rightCols<cols>();

    MatrixXd mat2(size, size);
    mat2.topLeftCorner(size/2, size/2).setZero();
    mat2.topRightCorner(size/2, size/2).setIdentity();
    mat2.bottomLeftCorner(size/2, size/2).setIdentity();
    mat2.bottomRightCorner(size/2, size/2).setZero();
    // std::cout << mat2 << std::endl << std::endl;
    OutputEigenMat(mat2);
}
```

## 3. Arithmetic Operators

Eigen provides basic linear algebra arithmetic operation.

```C++
// add
mat3 = mat1 + mat2;  mat3 += mat1;
// subtract
mat3 = mat1 - mat2;  mat3 -= mat1;

// scalar product	
mat3 = mat1 * s1; mat3 *= s1; mat3 = s1 * mat1;
mat3 = mat1 / s1; mat3 /= s1;

// matrix/vector products
col2 = mat1 * col1;
row2 = row1 * mat1;           row1 *= mat1;
mat3 = mat1 * mat2;           mat3 *= mat1; 

// transposition
mat1 = mat2.transpose();      mat1.transposeInPlace();
// adjoint
mat1 = mat2.adjoint();        mat1.adjointInPlace();

// dot product
scalar = vec1.dot(vec2);

// inner product
scalar = col1.adjoint() * col2;
scalar = (col1.adjoint() * col2).value();

// outer product
mat = col1 * col2.transpose();

// norm
scalar = vec1.norm(); scalar = vec1.squaredNorm();

// normalization
vec2 = vec1.normalized(); vec1.normalize(); // inplace

// cross product	
#include <Eigen/Geometry>
vec3 = vec1.cross(vec2);

// trace
mat.trace();
```