#include "DataVector.h"

DataVector::DataVector(int dimension)
{
    v.resize(dimension);
}

DataVector::~DataVector()
{
    v.clear();
}

DataVector::DataVector(const DataVector &other)
{
    v = other.v;
}

DataVector &DataVector::operator=(const DataVector &other)
{
    v = other.v;
    return *this;
}

void DataVector::setDimension(int dimension)
{
    v.clear();
    v.resize(dimension);
}

int DataVector::size()
{
    return v.size();
}

double &DataVector::operator[](int index)
{
    return v[index];
}

DataVector DataVector::operator+(const DataVector &other)
{
    DataVector result(v.size());
    for (int i = 0; i < v.size(); i++)
    {
        result.v[i] = v[i] + other.v[i];
    }
    return result;
}

DataVector DataVector::operator-(const DataVector &other)
{
    DataVector result(v.size());
    for (int i = 0; i < v.size(); i++)
    {
        result.v[i] = v[i] - other.v[i];
    }
    return result;
}

double DataVector::operator*(const DataVector &other)
{
    double result = 0;
    for (int i = 0; i < v.size(); i++)
    {
        result += v[i] * other.v[i];
    }
    return result;
}

bool DataVector::operator==(const DataVector &other)
{
    for (int i = 0; i < v.size(); i++)
    {
        if (v[i] != other.v[i])
        {
            return false;
        }
    }
    return true;
}

double DataVector::norm()
{
    double result = 0;
    for (int i = 0; i < v.size(); i++)
    {
        result += v[i] * v[i];
    }
    return sqrt(result);
}

double DataVector::distance(const DataVector &other)
{
    DataVector diff = *this - other;
    return diff.norm();
}