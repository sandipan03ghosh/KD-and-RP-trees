#include <bits/stdc++.h>
#include "VectorDataset.h"

using namespace std;
VectorDataset::VectorDataset()
{
    dataset.clear();
}

VectorDataset::~VectorDataset()
{
    if (dataset.size() > 0)
    {
        dataset.clear();
    }
}

DataVector &VectorDataset::operator[](int index)
{
    return dataset[index];
}

void VectorDataset::ReadDataset(const char *filename, int n, int d)
{
    ifstream file(filename);
    if (!file.is_open())
    {
        cerr << "Error: could not open file " << filename << endl;
        exit(1);
    }

    DataVector dv(d);
    for (int i = 0; i < n; i++)
    {
        
        for (int j = 0; j < d; j++)
        {
            if (!(file >> dv[j]))
            {
                cerr << "Error reading value at line " << i + 1 << ", position " << j + 1 << endl;
                exit(1);
            }

            if (j < d - 1)
            {
                if (file.peek() == ',')
                {
                    file.ignore();
                }
                else
                {
                    cerr << "Expected comma at line " << i + 1 << ", position " << j + 1 << endl;
                    exit(1);
                }
            }
        }
        dataset.push_back(dv);
    }

    file.close();
}

void VectorDataset::PrintDataset()
{
    for (int i = 0; i < dataset.size(); i++)
    {
        cout << "DataVector " << i << ": ";
        for (int j = 0; j < dataset[i].size(); j++)
        {
            cout << dataset[i][j] << " ";
        }
        cout << endl;
    }
}

void VectorDataset::AddDataVector(DataVector &dv)
{
    dataset.push_back(dv);
}

void VectorDataset::DeleteDataVector(int index)
{
    dataset.erase(dataset.begin() + index);
}

void VectorDataset::DeleteDataVector(DataVector &dv)
{
    for (int i = 0; i < dataset.size(); i++)
    {
        if (dataset[i] == dv)
        {
            dataset.erase(dataset.begin() + i);
            break;
        }
    }
    cout << "DataVector not found" << endl;
}

int VectorDataset::size()
{
    return dataset.size();
}