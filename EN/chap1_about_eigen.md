
# 1. What is Eigen
[Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page) is a highly efficient linear algebra library for C++.

Eigen's focus on efficient operation has two main downsides

1. When compiled in release mode it will blindly assume that all of the memory needed for any operation has been correctly allocated in the right location, without any form of checks. It will also not check that any inputs meet the required prerequisites (for example that matrices are of the correct size for multiplication to be a meaningful operation).
2. To allow it to use SIMD instructions it assumes all fixed size Eigen types have been allocated at 16-byte-aligned location.

These two downsides combine to give a large range of subtle hard to track down bugs.



