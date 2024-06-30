#ifndef VECTOR_DATASET_H
#define VECTOR_DATASET_H
using namespace std;
#include "DataVector.cpp"

class VectorDataset
{
    private:
    // a pointer to the entire dataset
    vector<DataVector> dataset;

    public:
        // a constructor that initializes the pointer to NULL
        VectorDataset();
        // a destructor that deletes the dataset
        ~VectorDataset();
        // an operator that takes an integer argument and returns the DataVector object at the specified index
        DataVector &operator[](int index);
        // a member function that takes a filename, the number of DataVector objects, and the dimension of each DataVector object
        // and reads the dataset from the csv file
        void ReadDataset(const char *filename, int n, int d);
        void PrintDataset();
        void AddDataVector(DataVector &dv);
        void DeleteDataVector(int index);
        void DeleteDataVector(DataVector &dv);
        int size();
};

#endif // VECTOR_DATASET_H