---
# C++ Object Oriented-Dynamic Matrix Class
## Overview
The program implements a Matrix class capable of dynamically creating and managing two-dimensional arrays using manual memory allocation through double pointers (double**). Each matrix object stores its dimensions (rows and cols) and the associated numerical elements in dynamically allocated memory.

The class provides constructors, a destructor, and deep-copy mechanisms to ensure safe initialization and deallocation of memory, preventing leaks or shallow copies. It also overloads fundamental arithmetic operators—+, -, and *—to enable intuitive mathematical operations between matrix objects. These operations include element-wise addition and subtraction, as well as matrix multiplication following dimensional compatibility rules.

Through its encapsulated methods (set, get, print, and various fill functions), the class abstracts away low-level pointer manipulation, offering a clean and user-friendly interface. The implementation reinforces key programming concepts such as pointers to pointers, dynamic allocation (new/delete), and operator overloading, which are essential for building complex object-oriented systems in C++.

## Engineering Relevance and Applications
This exercise simulates the core structure of how matrix operations are handled in scientific and engineering software. Matrices are a fundamental component in numerous computational fields, including:

* Structural and civil engineering – stiffness matrices and finite element analysis (FEA).
* Electrical and electronic engineering – circuit modeling, network analysis, and signal processing.
* Mechanical and aerospace engineering – dynamic system simulations and control algorithms.
* Data science and machine learning – representation of datasets and linear algebra computations.

By implementing the Matrix class manually (rather than relying on external libraries such as Eigen, Armadillo, or NumPy), the exercise deepens understanding of how memory is structured in 2D arrays, how to safely manage object lifetimes, and how to abstract mathematical behavior through operator overloading.

This foundational knowledge is directly applicable to developing simulation tools, numerical solvers, or embedded computational modules that require efficient and reliable matrix manipulation within larger engineering systems.
```C
/*-----------------------------------------------------------------------
 Name        : main.cpp
 Author      : Andy Bonilla
 Copyright   : Dynamic Matrix
 -----------------------------------------------------------------------*/

/* ---------------------------------------------------------------------
 	 Library inclusion
--------------------------------------------------------------------- */
#include <iostream>
using namespace std;

/* --------------------------------
    CLASS FOR ARBIRTRARY MATRIX SIZE
-------------------------------- */
class Matrix {
    private:
        int rows, cols;
        double** data;
    public:
        //constructor method
        Matrix(int r, int c) : rows(r), cols(c) 
        {
            if (r <= 0 || c <= 0)
                throw invalid_argument("Invalid Dimensions");
            data = new double*[rows];
            for (int i = 0; i < rows; ++i)
                data[i] = new double[cols];
            //zero init
            for (int i = 0; i < rows; ++i)
                for (int j = 0; j < cols; ++j)
                    data[i][j] = 0.0;
        }
        // --- Destructor ---
        ~Matrix() 
        {
            for (int i = 0; i < rows; ++i)
                delete[] data[i];
            delete[] data;
        }
        // --- Deep copy constructor
        Matrix(const Matrix& other) : rows(other.rows), cols(other.cols) 
        {
            data = new double*[rows];
            for (int i = 0; i < rows; ++i) 
            {
                data[i] = new double[cols];
                for (int j = 0; j < cols; ++j)
                    data[i][j] = other.data[i][j];
            }
        }
        // --- avoid memory leakage) ---
        Matrix& operator=(const Matrix& other) {
            if (this == &other) 
                return *this; // autoasignación

            //free last 
            for (int i = 0; i < rows; ++i)
                delete[] data[i];
            delete[] data;

            //reasging new dimensions
            rows = other.rows;
            cols = other.cols;

            //copy content
            data = new double*[rows];
            for (int i = 0; i < rows; ++i) {
                data[i] = new double[cols];
                for (int j = 0; j < cols; ++j)
                    data[i][j] = other.data[i][j];
            }
            return *this;
        }
        // --- Set and Get values ---
        void set(int i, int j, double val) {
            if (i < 0 || i >= rows || j < 0 || j >= cols)
                throw out_of_range("Index out of range!");
            data[i][j] = val;
        }
        double get(int i, int j) const {
            if (i < 0 || i >= rows || j < 0 || j >= cols)
                throw out_of_range("Index out of range!");
            return data[i][j];
        }
        // --- Mostrar matriX ---
        void print() 
        const {
            for (int i = 0; i < rows; ++i) {
                for (int j = 0; j < cols; ++j)
                    cout << data[i][j] << " ";
                cout << "\n";
            }
        }
        // --- Operador + 
        Matrix operator+(const Matrix& other) const {
            if (rows != other.rows || cols != other.cols)
                throw invalid_argument("Invalid Dimensions for addition!");
            Matrix result(rows, cols);
            for (int i = 0; i < rows; ++i)
                for (int j = 0; j < cols; ++j)
                    result.data[i][j] = data[i][j] + other.data[i][j];
            return result;
        }

        // --- Operador - 
        Matrix operator-(const Matrix& other) const {
            if (rows != other.rows || cols != other.cols)
                throw invalid_argument("Invalid Dimensions for substraction!");
            Matrix result(rows, cols);
            for (int i = 0; i < rows; ++i)
                for (int j = 0; j < cols; ++j)
                    result.data[i][j] = data[i][j] - other.data[i][j];
            return result;
        }

        // --- Operador * 
        Matrix operator*(const Matrix& other) const {
            if (cols != other.rows)
                throw invalid_argument("Invalid Dimensions for multiplication!");
            Matrix result(rows, other.cols);
            for (int i = 0; i < rows; ++i)
                for (int j = 0; j < other.cols; ++j)
                    for (int k = 0; k < cols; ++k)
                        result.data[i][j] += data[i][k] * other.data[k][j];
            return result;
        }
};
/* --------------------------------
    MAIN
-------------------------------- */
int main() {
    try 
    {
        //2x2 matrix example
        cout << "---2x2 MATRIX EXAMPLE\n";
        Matrix A_2b2(2,2);
        A_2b2.set(0,0,1); A_2b2.set(0,1,2);
        A_2b2.set(1,0,3); A_2b2.set(1,1,4);

        Matrix B_2b2(2,2);
        B_2b2.set(0,0,5); B_2b2.set(0,1,6);
        B_2b2.set(1,0,7); B_2b2.set(1,1,8);

        cout << "MatriX A:\n"; A_2b2.print();
        cout << "MatriX B:\n"; B_2b2.print();

        Matrix Sum_2b2 = A_2b2 + B_2b2;
        Matrix Subs_2b2 = A_2b2 - B_2b2;
        Matrix Mult_2b2 = A_2b2 * B_2b2;

        cout << "\nA + B:\n"; Sum_2b2.print();
        cout << "\nA - B:\n"; Subs_2b2.print();
        cout << "\nA * B:\n"; Mult_2b2.print();
        //3x3 matrix example
        cout << "---3x3 MATRIX EXAMPLE\n";
        Matrix A_3b3(3,3);
        A_3b3.set(0,0,1); A_3b3.set(0,1,0); A_3b3.set(0,2,2);
        A_3b3.set(1,0,-1); A_3b3.set(1,1,3); A_3b3.set(1,2,1);
        A_3b3.set(2,0,3); A_3b3.set(2,1,2); A_3b3.set(2,2,0);

        Matrix B_3b3(3,3);
        B_3b3.set(0,0,2); B_3b3.set(0,1,1); B_3b3.set(0,2,0);
        B_3b3.set(1,0,3); B_3b3.set(1,1,0); B_3b3.set(1,2,-1);
        B_3b3.set(2,0,4); B_3b3.set(2,1,1); B_3b3.set(2,2,2);
        
        cout << "MatriX A:\n"; A_3b3.print();
        cout << "MatriX B:\n"; B_3b3.print();

        Matrix Sum_3b3 = A_3b3 + B_3b3;
        Matrix Subs_3b3 = A_3b3 - B_3b3;
        Matrix Mult_3b3 = A_3b3 * B_3b3;

        cout << "\nA + B:\n"; Sum_3b3.print();
        cout << "\nA - B:\n"; Subs_3b3.print();
        cout << "\nA * B:\n"; Mult_3b3.print();
        //2x3 and 3x2 matrix example
        cout << "---2x3 and 3x2 MATRIX EXAMPLE\n";
        Matrix A_2b3(2,3);
        A_2b3.set(0,0,1); A_2b3.set(0,1,2); A_2b3.set(0,2,3);
        A_2b3.set(1,0,4); A_2b3.set(1,1,5); A_2b3.set(1,2,6);

        Matrix B_3b2(3,2);
        B_3b2.set(0,0,7);  B_3b2.set(0,1,8);
        B_3b2.set(1,0,9);  B_3b2.set(1,1,10);
        B_3b2.set(2,0,11); B_3b2.set(2,1,12);

        cout << "MatriX A:\n"; A_2b3.print();
        cout << "MatriX B:\n"; B_3b2.print();
        
        Matrix Mult_2b3_3b_2 = A_2b3*B_3b2;
        cout << "\nA * B:\n"; Mult_2b3_3b_2.print();

    } 
    catch (const exception& e) 
    {
        cerr << "Error: " << e.what() << "\n";
    }

    return 0;
}
```
After compiling

```bash
g++ -std=c++17 Main.cpp -o main
```
And running it 

```bash
./main
```
It shows the following in console:
```bash
---2x2 MATRIX EXAMPLE
MatriX A:
1 2 
3 4 
MatriX B:
5 6 
7 8 

A + B:
6 8 
10 12 

A - B:
-4 -4 
-4 -4 

A * B:
19 22 
43 50 
---3x3 MATRIX EXAMPLE
MatriX A:
1 0 2 
-1 3 1 
3 2 0 
MatriX B:
2 1 0 
3 0 -1 
4 1 2 

A + B:
3 1 2 
2 3 0 
7 3 2 

A - B:
-1 -1 2 
-4 3 2 
-1 1 -2 

A * B:
10 3 4 
11 0 -1 
12 3 -2 
---2x3 and 3x2 MATRIX EXAMPLE
MatriX A:
1 2 3 
4 5 6 
MatriX B:
7 8 
9 10 
11 12 

A * B:
58 64 
139 154 
```
