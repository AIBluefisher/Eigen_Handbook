# 3. Advanced Operations

## [EigenValue and Eigen Vectors](http://eigen.tuxfamily.org/dox-3.2/classEigen_1_1EigenSolver.html)

```C++
#include <Eigen/Core>
#include <Eigen/Eigenvalues>

int main()
{
    Eigen::MatrixXd V;
    // Initialize V in your code here.
    // ...

    // Using EigenSolver.
    Eigen::EigenSolver<Eigen::MatrixXd> eigen_solver(V);

    // The result of EigenSolver is actually complex number.
    MatrixXcd complex_eigenvalues = es.eigenvalues();
    MatrixXcd complex_eigenvectors = es.eigenvectors();

    // Retrieve the real value of eigen values and eigen vectors.
    Eigen::MatrixXd real_eigenvalues = eigen_solver.eigenvalues().real();
    Eigen::MatrixXd real_eigenvectors = eigen_solver.eigenvectors().real();
}

```

## [Singular Value Decomposition](http://eigen.tuxfamily.org/dox-3.2/classEigen_1_1JacobiSVD.html#title0)

Eigen 3.2 uses `JacobiSVD` to compute SVD.  `JacobiSVD` decomposition computes only the singular values by default. If you want `U` or `V`, you need to ask for them explicitly. Singular values are always sorted in decreasing order.

```C++
#include <iostream>

#include <Eigen/Core>
#include <Eigen/SVD>

int main()
{
    Eigen::MatrixXd A;
    // Initialize A in your code here.
    // ...

    Eigen::JacobiSVD<Eigen::MatrixXd> jacobi_svd(A, 
                                                Eigen::ComputeThinU | Eigen::ComputeThinV);
    
    // Get singular values.
    std::cout << "Sigular value: " << jacobi_svd.singularValues() << std::endl;
    // Get U or V.
    std::cout << "U: " << jacobi_svd.matrixU() << "\n"
              << "V: " << jacobi_svd.matrixV() << std::endl; 
}

```

## [Sparse Matrix](http://eigen.tuxfamily.org/dox-3.2/group__TutorialSparse.html)
Also see this [page](http://eigen.tuxfamily.org/dox-3.2/group__SparseQuickRefPage.html) for quick reference.

Manipulating and solving sparse problems in eigen includes six modules:
- `SparseCore`. SparseMatrix and SparseVector classes, matrix assembly, basic sparse linear algebra (including sparse triangular solvers).

- `SparseCholesky`. Direct sparse LLT and LDLT Cholesky factorization to solve sparse self-adjoint positive definite problems.

- `SparseLU`. Sparse LU factorization to solve general square sparse systems.

- `SparseQR`. Sparse QR factorization for solving sparse linear least-squares problems.

- `IterativeLinearSolvers`. Iterative solvers to solve large general linear square problems (including self-adjoint positive definite problems).

- `Sparse`. Includes all the above modules.

`SparseMatrix` implements a more versatile variant of the widely-used Compressed Column (or Row) Storage scheme. It consists of four compact arrays:

- `Values`: stores the coefficient values of the non-zeros.
- `InnerIndices`: stores the row (resp. column) indices of the non-zeros.
- `OuterStarts`: stores for each column (resp. row) the index of the first non-zero in the previous two arrays.
- `InnerNNZs`: stores the number of non-zeros of each column (resp. row). The word inner refers to an inner vector that is a column for a column-major matrix, or a row for a row-major matrix. The word outer refers to the other direction.

The case where no empty space is available is a special case, and is refered as the compressed mode. It corresponds to the widely used Compressed Column (or Row) Storage schemes (CCS or CRS). Any SparseMatrix can be turned to this form by calling the `SparseMatrix::makeCompressed()` function. In this case, one can remark that the InnerNNZs array is redundant with OuterStarts because we the equality: InnerNNZs[j] = OuterStarts[j+1]-OuterStarts[j]. Therefore, in practice a call to SparseMatrix::makeCompressed() frees this buffer.

The results of Eigen's operations always produces compressed sparse matrices. On the other hand, the insertion of a new element into a SparseMatrix converts this later to the uncompressed mode.

Refer to the [page](http://eigen.tuxfamily.org/dox-3.2/group__SparseQuickRefPage.html) for more supported SparseMatrix operations.

```C++
#include <Eigen/Core>
#include <Eigen/SparseCore>

int main()
{
    // array size.
    const int m = 100;
    const int n = 50;

    // Declare a column-major compressed sparse matrix.
    Eigen::SparseMatrix<double> sparse_mat(m, n);
    // Or row-major compressed sparse matrix.
    // Eigen::SparseMatrix<double, RowMajor> sparse_mat(m, n);

    // Filling sparse matrix.
    std::vector<Eigen::Triplet<double>> triplets;
    std::vector<double> data;
    // assign data in your code here.
    // ...
    int k = 0;
    for (int i = 0; i < ...; i++) {
        for (int j = 0; j < ...; j++) {
            triplets.push_back(Eigen::Triplet(i, j, data[k++]));
            // [alternative]: or instead using:
            // sparse_mat.insert(i, j) = data[k++];
        }
    }

    // If using the alternative code, comment the code below.
    sparse_mat.setFromTriplets(triplets.begin(), triplets.end());

    // optional operation.
    sparse_mat.makeCompressed();
}
```

## Optimizing Eigen Operations

// TODO.