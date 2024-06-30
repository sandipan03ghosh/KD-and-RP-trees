#include "VectorDataset.cpp"

class TreeIndex
{
protected:
    VectorDataset dataset;
    TreeIndex() : dataset() {}
    ~TreeIndex() {}

public:
    static TreeIndex &GetInstance()
    {
        static TreeIndex instance;
        return instance;
    }
    void ReadData(const char *filename, int n, int d)
    {
        dataset.ReadDataset(filename, n, d);
    }
};

class KDNode
{
private:
    bool leaf;
    int axis;
    double median;
    KDNode *left, *right, *parent;
    vector<int> indices;

public:
    KDNode() : leaf(false), axis(0), median(0), left(NULL), right(NULL), parent(NULL) {}
    ~KDNode() {}
    void setLeaf(bool leaf)
    {
        this->leaf = leaf;
    }
    bool isLeaf()
    {
        return leaf;
    }
    void setAxis(int axis)
    {
        this->axis = axis;
    }
    int getAxis()
    {
        return axis;
    }
    void setMedian(double median)
    {
        this->median = median;
    }
    double getMedian()
    {
        return median;
    }
    void setLeft(KDNode *left)
    {
        this->left = left;
    }
    KDNode *getLeft()
    {
        return left;
    }
    void setRight(KDNode *right)
    {
        this->right = right;
    }
    KDNode *getRight()
    {
        return right;
    }
    void setParent(KDNode *parent)
    {
        this->parent = parent;
    }
    KDNode *getParent()
    {
        return parent;
    }
    void setIndices(vector<int> indices)
    {
        this->indices = indices;
    }
    vector<int> getIndices()
    {
        return indices;
    }
};

// Get median of a vector of doubles in O(n) time
double GetMedian(vector<double> &values)
{
    nth_element(values.begin(), values.begin() + values.size() / 2, values.end());
    return values[values.size() / 2];
}    

class KDTreeIndex : public TreeIndex
{
private:
    KDNode *root;
    KDTreeIndex() : TreeIndex(), root(NULL) {}
    ~KDTreeIndex() {}
    KDNode *searchnode(KDNode *node, DataVector &dv)
    {
        if (node == NULL)
        {
            return NULL;
        }
        if (node->isLeaf())
        {
            return node;
        }
        int axis = node->getAxis();
        if (dv[axis] < node->getMedian())
        {
            return searchnode(node->getLeft(), dv);
        }
        else
        {
            return searchnode(node->getRight(), dv);
        }
    }

public:
    static KDTreeIndex &GetInstance()
    {
        static KDTreeIndex instance;
        return instance;
    }
    double ChooseRule(vector<int> &indices, int axis)
    {
        vector<double> values;
        for (int i = 0; i < indices.size(); i++)
        {
            values.push_back(dataset[indices[i]][axis]);
        }
        return GetMedian(values);
    }
    KDNode *BuildKDTree(vector<int> &indices, int depth)
    {
        KDNode *node = new KDNode();
        if (indices.size() == 0)
        {
            node->setLeaf(true);
            return node;
        }
        if (indices.size() == 1)
        {
            node->setLeaf(true);
            node->setIndices(indices);
            return node;
        }
        int axis = depth % dataset[0].size();
        node->setAxis(axis);
        double median = ChooseRule(indices, axis);
        node->setMedian(median);
        vector<int> leftIndices, rightIndices;
        for (int i = 0; i < indices.size(); i++)
        {
            if (dataset[indices[i]][axis] < median)
            {
                leftIndices.push_back(indices[i]);
            }
            else
            {
                rightIndices.push_back(indices[i]);
            }
        }
        // prevent infinite recursion
        if (leftIndices.size() == indices.size() || rightIndices.size() == indices.size())
        {
            node->setLeaf(true);
            node->setIndices(indices);
            return node;
        }
        node->setLeft(BuildKDTree(leftIndices, depth + 1));
        node->setRight(BuildKDTree(rightIndices, depth + 1));
        node->getLeft()->setParent(node);
        node->getRight()->setParent(node);
        return node;
    }
    void BuildKDTree()
    {
        vector<int> indices;
        for (int i = 0; i < dataset.size(); i++)
        {
            indices.push_back(i);
        }
        root = BuildKDTree(indices, 0);
    }
    void ReadData(const char *filename, int n, int d)
    {
        TreeIndex::ReadData(filename, n, d);
    }
    void PrintData()
    {
        dataset.PrintDataset();
    }
    void PrintKDTree(KDNode *node)
    {
        if (node == NULL)
        {
            return;
        }
        if (node->isLeaf())
        {
            vector<int> indices = node->getIndices();
            for (int i = 0; i < indices.size(); i++)
            {
                cout << indices[i] << " ";
            }
            cout << endl;
            return;
        }
        cout << "Axis: " << node->getAxis() << " Median: " << node->getMedian() << endl;
        PrintKDTree(node->getLeft());
        PrintKDTree(node->getRight());
    }
    void PrintKDTree()
    {
        PrintKDTree(root);
    }
    void KDAddDataVector(DataVector &dv)
    {
        dataset.AddDataVector(dv);
        KDNode *node = searchnode(root, dv);
        if (node == NULL)
        {
            cerr << "Error: could not find node" << endl;
            exit(1);
        }
        vector<int> indices = node->getIndices();
        indices.push_back(dataset.size() - 1);
        node->setIndices(indices);
    }
    void KDDeleteDataVector(DataVector &dv)
    {
        dataset.DeleteDataVector(dv);
        KDNode *node = searchnode(root, dv);
        if (node == NULL)
        {
            cerr << "Error: could not find node" << endl;
            exit(1);
        }
        vector<int> indices = node->getIndices();
        for (int i = 0; i < indices.size(); i++)
        {
            if (dataset[indices[i]] == dv)
            {
                indices.erase(indices.begin() + i);
                break;
            }
        }
        node->setIndices(indices);
    }
    vector<DataVector> getDataVectors(vector<int> indices)
    {
        vector<DataVector> result;
        for (int i = 0; i < indices.size(); i++)
        {
            result.push_back(dataset[indices[i]]);
        }
        return result;
    }
    // A recursive backtracking function to find the k nearest neighbors
    // goes to the leaf node that contains the query point
    // backtracks to the parent node and checks the other child node if necessary
    KDNode *searchKNN(KDNode *node, DataVector &query, int k, std::priority_queue<std::pair<double, int>> &neighbors)
    {
        if (node == NULL)
        {
            return NULL;
        }

        int axis = node->getAxis();

        // Handle leaves (store data) and non-leaves (null data) separately
        if (node->isLeaf())
        {
            for(auto index: node->getIndices())
            {
                double dist = query.distance(dataset[index]); // Access data for leaf nodes
                if (neighbors.size() < k || dist < neighbors.top().first)
                {
                    neighbors.push({dist, index});
                    if (neighbors.size() > k)
                    {
                        neighbors.pop(); // Keep only k closest neighbors
                    }
                }
            }
        }
        else
        {
            // Calculate a hypothetical distance for non-leaf nodes using median
            double dist = std::abs(query[axis] - node->getMedian());
            if (neighbors.size() < k || dist < neighbors.top().first)
            {
                // Potentially explore both subtrees
                if (query[axis] < node->getMedian())
                {
                    searchKNN(node->getLeft(), query, k, neighbors);
                    searchKNN(node->getRight(), query, k, neighbors);
                }
                else
                {
                    searchKNN(node->getRight(), query, k, neighbors);
                    searchKNN(node->getLeft(), query, k, neighbors);
                }
            }
        }

        return NULL; // No further node to explore
    }

    std::vector<std::pair<double, int>> knnSearch(DataVector &query, int k)
    {
        std::priority_queue<std::pair<double, int>> neighbors; // Max-heap for nearest neighbors
        searchKNN(root, query, k, neighbors);

        std::vector<std::pair<double, int>> results;
        while (!neighbors.empty())
        {
            results.push_back(neighbors.top());
            neighbors.pop();
        }
        std::reverse(results.begin(), results.end()); // Reverse for ascending order
        return results;
    }
};

class RPNode
{
private:
    bool leaf;
    DataVector axis;
    double threshold;
    RPNode *left, *right, *parent;
    vector<int> indices;

public:
    RPNode() : leaf(false), axis(0), threshold(0), left(NULL), right(NULL), parent(NULL) {}
    ~RPNode() {}
    void setLeaf(bool leaf)
    {
        this->leaf = leaf;
    }
    bool isLeaf()
    {
        return leaf;
    }
    void setAxis(DataVector axis)
    {
        this->axis = axis;
    }
    DataVector getAxis()
    {
        return axis;
    }
    void setThreshold(double threshold)
    {
        this->threshold = threshold;
    }
    double getThreshold()
    {
        return threshold;
    }
    void setLeft(RPNode *left)
    {
        this->left = left;
    }
    RPNode *getLeft()
    {
        return left;
    }
    void setRight(RPNode *right)
    {
        this->right = right;
    }
    RPNode *getRight()
    {
        return right;
    }
    void setParent(RPNode *parent)
    {
        this->parent = parent;
    }
    RPNode *getParent()
    {
        return parent;
    }
    void setIndices(vector<int> indices)
    {
        this->indices = indices;
    }
    vector<int> getIndices()
    {
        return indices;
    }
};

class RPTreeIndex : public TreeIndex
{
private:
    RPNode *root;
    RPTreeIndex() : TreeIndex(), root(NULL) {}
    ~RPTreeIndex() {}
    RPNode *searchnode(RPNode *node, DataVector &dv)
    {
        if (node == NULL)
        {
            return NULL;
        }
        if (node->isLeaf())
        {
            return node;
        }
        DataVector axis = node->getAxis();
        double dot = dv * axis;
        if (dot < node->getThreshold())
        {
            return searchnode(node->getLeft(), dv);
        }
        else
        {
            return searchnode(node->getRight(), dv);
        }
    }

public:
    static RPTreeIndex &GetInstance()
    {
        static RPTreeIndex instance;
        return instance;
    }
    double ChooseRule(vector<int> &indices, DataVector &axis)
    {
        vector<double> values;
        for (int i = 0; i < indices.size(); i++)
        {
            values.push_back(dataset[indices[i]] * axis);
        }
        return GetMedian(values);
    }
    RPNode *BuildRPTree(vector<int> &indices, int depth)
    {
        RPNode *node = new RPNode();
        if (indices.size() == 0)
        {
            node->setLeaf(true);
            return node;
        }
        if (indices.size() == 1)
        {
            node->setLeaf(true);
            node->setIndices(indices);
            return node;
        }
        DataVector axis;
        axis.setDimension(dataset[0].size());
        for (int i = 0; i < axis.size(); i++)
            axis[i] = (double)(rand()%1000);
        int norm = axis.norm();
        for (int i = 0; i < axis.size(); i++)
            axis[i] /= norm;
        node->setAxis(axis);
        double threshold = ChooseRule(indices, axis);
        node->setThreshold(threshold);
        vector<int> leftIndices, rightIndices;
        for (int i = 0; i < indices.size(); i++)
        {
            if (dataset[indices[i]] * axis < threshold)
            {
                leftIndices.push_back(indices[i]);
            }
            else
            {
                rightIndices.push_back(indices[i]);
            }
        }
        // prevent infinite recursion
        if (leftIndices.size() == indices.size() || rightIndices.size() == indices.size())
        {
            node->setLeaf(true);
            node->setIndices(indices);
            return node;
        }
        node->setLeft(BuildRPTree(leftIndices, depth + 1));
        node->setRight(BuildRPTree(rightIndices, depth + 1));
        node->getLeft()->setParent(node);
        node->getRight()->setParent(node);
        return node;
    }
    void BuildRPTree()
    {
        vector<int> indices;
        for (int i = 0; i < dataset.size(); i++)
        {
            indices.push_back(i);
        }
        root = BuildRPTree(indices, 0);
    }
    void ReadData(const char *filename, int n, int d)
    {
        TreeIndex::ReadData(filename, n, d);
    }
    void PrintData()
    {
        dataset.PrintDataset();
    }
    void PrintRPTree(RPNode *node)
    {
        if (node == NULL)
        {
            return;
        }
        if (node->isLeaf())
        {
            vector<int> indices = node->getIndices();
            for (int i = 0; i < indices.size(); i++)
            {
                cout << indices[i] << " ";
            }
            cout << endl;
            return;
        }
        cout << "Axis: ";
        for (int i = 0; i < node->getAxis().size(); i++)
        {
            cout << node->getAxis()[i] << " ";
        }
        cout << "Threshold: " << node->getThreshold() << endl;
        PrintRPTree(node->getLeft());
        PrintRPTree(node->getRight());
    }
    void PrintRPTree()
    {
        PrintRPTree(root);
    }
    void RPAddDataVector(DataVector &dv)
    {
        dataset.AddDataVector(dv);
        RPNode *node = searchnode(root, dv);
        if (node == NULL)
        {
            cerr << "Error: could not find node" << endl;
            exit(1);
        }
        vector<int> indices = node->getIndices();
        indices.push_back(dataset.size() - 1);
        node->setIndices(indices);
    }
    void RPDeleteDataVector(DataVector &dv)
    {
        dataset.DeleteDataVector(dv);
        RPNode *node = searchnode(root, dv);
        if (node == NULL)
        {
            cerr << "Error: could not find node" << endl;
            exit(1);
        }
        vector<int> indices = node->getIndices();
        for (int i = 0; i < indices.size(); i++)
        {
            if (dataset[indices[i]] == dv)
            {
                indices.erase(indices.begin() + i);
                break;
            }
        }
        node->setIndices(indices);
    }
    vector<DataVector> getDataVectors(vector<int> indices)
    {
        vector<DataVector> result;
        for (int i = 0; i < indices.size(); i++)
        {
            result.push_back(dataset[indices[i]]);
        }
        return result;
    }
    // A recursive backtracking function to find the k nearest neighbors
    // goes to the leaf node that contains the query point
    // backtracks to the parent node and checks the other child node if necessary
    RPNode *searchKNN(RPNode *node, DataVector &query, int k, std::priority_queue<std::pair<double, int>> &neighbors)
    {
        if (node == NULL)
        {
            return NULL;
        }

        // Handle leaves (store data) and non-leaves (null data) separately
        if (node->isLeaf())
        {
            for(auto index: node->getIndices())
            {
                double dist = query.distance(dataset[index]); // Access data for leaf nodes
                if (neighbors.size() < k || dist < neighbors.top().first)
                {
                    neighbors.push({dist, index});
                    if (neighbors.size() > k)
                    {
                        neighbors.pop(); // Keep only k closest neighbors
                    }
                }
            }
        }
        else
        {
            // Calculate a hypothetical distance for non-leaf nodes using axis and threshold
            double dist = std::abs(query * node->getAxis() - node->getThreshold());
            if (neighbors.size() < k || dist < neighbors.top().first)
            {
                // Potentially explore both subtrees
                if (query * node->getAxis() < node->getThreshold())
                {
                    searchKNN(node->getLeft(), query, k, neighbors);
                    searchKNN(node->getRight(), query, k, neighbors);
                }
                else
                {
                    searchKNN(node->getRight(), query, k, neighbors);
                    searchKNN(node->getLeft(), query, k, neighbors);
                }
            }
        }

        return NULL; // No further node to explore
    }
    std::vector<std::pair<double, int>> knnSearch(DataVector &query, int k)
    {
        std::priority_queue<std::pair<double, int>> neighbors; // Max-heap for nearest neighbors
        searchKNN(root, query, k, neighbors);

        std::vector<std::pair<double, int>> results;
        while (!neighbors.empty())
        {
            results.push_back(neighbors.top());
            neighbors.pop();
        }
        std::reverse(results.begin(), results.end()); // Reverse for ascending order
        return results;
    }
};