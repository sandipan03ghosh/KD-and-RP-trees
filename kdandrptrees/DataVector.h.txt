#ifndef DATAVECTOR_H_
#define DATAVECTOR_H_
#include <bits/stdc++.h>


class DataVector
{

private:
    // a vector of double values
    std::vector<double> v;

public:
    // a constructor that takes an integer argument and initializes the vector to the specified dimension
    DataVector(int dimension = 0);
    // a destructor that clears the vector
    ~DataVector();
    // a copy constructor that takes a DataVector object as an argument
    // and initializes the vector to the same dimension and values
    DataVector(const DataVector &other);
    // an assignment operator that takes a DataVector object as an argument
    // and assigns the vector to the same dimension and values
    DataVector &operator=(const DataVector &other);
    // a member function that takes an integer argument and sets the vector to the specified dimension
    void setDimension(int dimension = 0);
    // a member function that returns the dimension of the vector
    int size();
    // a member function that takes an integer argument and returns the value at the specified index
    double &operator[](int index);
    // a member function that takes a DataVector object as an argument
    // and returns the sum of the two vectors
    DataVector operator+(const DataVector &other);
    // a member function that takes a DataVector object as an argument
    // and returns the difference of the two vectors
    DataVector operator-(const DataVector &other);
    // a member function that takes a DataVector object as an argument
    // and returns the dot product of the two vectors
    double operator*(const DataVector &other);
    // a member function that takes a DataVector object as an argument
    // and returns whether the two vectors are equal
    bool operator==(const DataVector &other);
    // a member function that returns the norm of the vector
    double norm();
    // a member function that takes a DataVector object as an argument
    // and returns the distance between the two vectors
    double distance(const DataVector &other);
};

#endif // DATAVECTOR_H_
