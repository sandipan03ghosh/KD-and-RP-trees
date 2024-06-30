#include "TreeIndex.h"
#include <chrono>

int main()
{
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    cout.tie(NULL);
    
    auto begin = std::chrono::high_resolution_clock::now();
    RPTreeIndex::GetInstance().ReadData("fmnist-train.csv", 60000, 784);
    // KDTreeIndex::GetInstance().PrintData();
    RPTreeIndex::GetInstance().BuildRPTree();
    // KDTreeIndex::GetInstance().PrintKDTree();
    VectorDataset test_fmnist;
    test_fmnist.ReadDataset("fmnist-test.csv", 10000, 784);
    auto end = std::chrono::high_resolution_clock::now();
    auto elapsed = std::chrono::duration_cast<std::chrono::nanoseconds>(end - begin);
    cerr << "Time measured: " << elapsed.count() * 1e-9 << " seconds.\n"; 

    begin = std::chrono::high_resolution_clock::now();
    vector<pair<double, int>> result = RPTreeIndex::GetInstance().knnSearch(test_fmnist[0], 5);
    for(int i = 0; i < result.size(); i++)
    {
        cout << "Distance: " << result[i].first << ", Index: " << result[i].second << endl;
    }
    end = std::chrono::high_resolution_clock::now();
    elapsed = std::chrono::duration_cast<std::chrono::nanoseconds>(end - begin);
    cerr << "Time measured: " << elapsed.count() * 1e-9 << " seconds.\n"; 
    return 0;
}