---
layout: post
title: Simit FFI 
---
The Simit Foreign Function Interface
====================================
___These pages are under construction___

The Simit Foreign Function Interface (ffi) lets Simit code call functions
written in C.  The `extern` keyword is used to call C functions. Extern
functions can accept or return any number of Simit scalar, vector and matrix
arguments. Passing elements or sets to extern functions is not supported. The
argument list of the C function must contain all the Simit arguments followed
by the result arguments, which are passed in using a pointer. A vector and
matrix argument passed into an extern function must be matched by one or more
arguments describing the vector or matrix type (e.g. it's size) followed by one
argument for its value.

The following declares a function `printscalar` written in C and linked to the
program:

```simit
extern func printInt(a : int);

export func main()
  i : int = 2;
  printInt(i);
end
```

The C function `printInt` can be defined as follows:

```c
int printInt(int a) {
  printf("%d\n", a);
  return 0;  // Return success error code
}
```

Since the argument is a scalar it is passed by value. The functions returns an
error code (0 for success, any other value for failure). A failure causes an
exception to be thrown from `simit::Function::run`.

The next example declares and calls a function that prints a vector.

```simit
extern func printVector(a : vector[3](int));

export func main()
  vec : vector[3](int) = [1, 2, 3];
  printVector(vec);
end
```

The C function is defined as follows

```c
int printVector(int n, int* a) {
  printf("[%d", a[0]);
  for (int i=1; i<n; i++) {
    printf(", %d", a[i]);
  }
  printf("]\n");
  return 0;  // Return success error code
}
```

Vector arguments in Simit becomes two arguments in C: the vector's size and a
pointer to its values.

The table lists the required C formal arguments for dense Simit vectors and
matrices. In the following all Simit floats are double precision, `V` refers to
a Simit set, and `3` can be replaced by any number.

| Simit Actual                  | C Formal                     |
|-------------------------------|------------------------------|
| `int`                         | `int`                        |
| `vector[3](float)`            | `int, double*`               |
| `vector[V](float)`            | `int, double*`               |
| `vector[V](vector[3](float))` | `int, int, double*`          |
| `matrix[3,3](float)`          | `int, int, double*`          |

The blocked types have two sets of type arguments. The first set describes the
number of blocks in each dimension (one for vector, two for matrices). The
second set describes the size of each block in each dimension.

The next table lists the C formal arguments for sparse Simit matrices:

| Simit Actual                      | C Formal                                  |
|-----------------------------------|-------------------------------------------|
| `matrix[V,V](float)`              | `int, int, int*, int*, double*`           |
| `matrix[V,V](matrix[3,3](float))` | `int, int, int*, int*, int, int, double*` |

[...]

This table lists the required C formal arguments for dense Simit
function results (return values):

| Simit                             | C                    |
|-----------------------------------|----------------------|
| `int`                             | `int*`               |
| `vector[3](float)`                | `int, double*`       |
| `vector[V](float)`                | `int, double*`       |
| `matrix[3,3](float)`              | `int, int, double*`  |
| `vector[V](vector[3](float))`     | `int, int, double*`  |

The final table lists the required C formal arguments for sparse Simit function
results:

| Simit                             | C                                            |
|-----------------------------------|----------------------------------------------|
| `matrix[V,V](float)`              | `int, int, int**, int**, double**`           |
| `matrix[V,V](matrix[3,3](float))` | `int, int, int**, int**, int, int, double**` |

The (rowPtr and colIdx) indices and value arrays of sparse matrices are passed
as pointer-pointers.  The reason for this is that Simit cannot know the number
of non-zeroes or the non-zero distribution of the matrix.  The memory of sparse
matrices must therefore be allocated by the user using `simit_malloc`.
However, Simit will free the arrays when they the scope they were allocated in
closes.

[TODO: Add code example]
