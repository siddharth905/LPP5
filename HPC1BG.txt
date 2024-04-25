#include <iostream>
#include <stack>
#include <omp.h>

using namespace std;

class Graph {
    int V; // Number of vertices
    int** adj; // Adjacency list represented using 2D array

public:
    Graph(int V) : V(V) {
        adj = new int*[V];
        for (int i = 0; i < V; ++i) {
            adj[i] = new int[V];
            // Initialize adjacency matrix with 0
            for (int j = 0; j < V; ++j)
                adj[i][j] = 0;
        }
    }

    // Function to add an edge to the graph
    void addEdge(int u, int v) {
        adj[u][v] = 1;
        adj[v][u] = 1; // Assuming undirected graph
    }

    // Parallel DFS traversal
    void parallelDFS(int source) {
        bool* visited = new bool[V];
        for (int i = 0; i < V; ++i)
            visited[i] = false;

        stack<int> st;
        st.push(source);
        visited[source] = true;

        // Start parallel region for DFS traversal
        #pragma omp parallel
        {
            while (!st.empty()) {
                int u;
                // Avoid contention for popping nodes from the stack
                #pragma omp critical
                {
                    u = st.top();
                    st.pop();
                }

                // Process node u
                cout << u << " ";

                // Explore adjacent nodes in parallel
                #pragma omp for
                for (int v = 0; v < V; ++v) {
                    if (adj[u][v] && !visited[v]) {
                        // Mark node v as visited
                        visited[v] = true;
                        // Push node v onto the stack
                        #pragma omp critical
                        st.push(v);
                    }
                }
            }
        }

        delete[] visited;
    }

    ~Graph() {
        for (int i = 0; i < V; ++i)
            delete[] adj[i];
        delete[] adj;
    }
};

int main() {
    int V = 6;
    Graph g(V);
    g.addEdge(0, 1);
    g.addEdge(0, 2);
    g.addEdge(1, 3);
    g.addEdge(2, 4);
    g.addEdge(2, 5);

    cout << "Parallel DFS starting from node 0:\n";
    g.parallelDFS(0);
    cout << endl;

    return 0;
}