# Boost.Graph - 图算法库

## 概述

Boost.Graph (BGL) 提供通用的图数据结构和算法，支持有向图、无向图、加权图等。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/graph_utility.hpp>
#include <iostream>

int main() {
    using namespace boost;
    
    // 创建无向图
    typedef adjacency_list<vecS, vecS, undirectedS> Graph;
    Graph g(5);  // 5个顶点
    
    // 添加边
    add_edge(0, 1, g);
    add_edge(0, 2, g);
    add_edge(1, 3, g);
    add_edge(2, 3, g);
    add_edge(3, 4, g);
    
    std::cout << "顶点数: " << num_vertices(g) << std::endl;
    std::cout << "边数: " << num_edges(g) << std::endl;
    
    print_graph(g);
    
    return 0;
}
```

---

## 有向图

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <iostream>

int main() {
    using namespace boost;
    
    // 创建有向图
    typedef adjacency_list<vecS, vecS, directedS> Graph;
    Graph g(4);
    
    add_edge(0, 1, g);
    add_edge(0, 2, g);
    add_edge(1, 2, g);
    add_edge(2, 3, g);
    add_edge(3, 1, g);  // 循环边
    
    std::cout << "有向图边列表:\\n";
    typedef graph_traits<Graph>::edge_iterator edge_iter;
    std::pair<edge_iter, edge_iter> ep;
    
    for (ep = edges(g); ep.first != ep.second; ++ep.first) {
        std::cout << "  " << source(*ep.first, g) 
                  << " -> " << target(*ep.first, g) << std::endl;
    }
    
    return 0;
}
```

---

## 加权图

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <iostream>

struct EdgeWeight {
    double weight;
};

int main() {
    using namespace boost;
    
    typedef adjacency_list<vecS, vecS, undirectedS, no_property, EdgeWeight> Graph;
    Graph g(5);
    
    // 添加带权重的边
    EdgeWeight w1{1.5}, w2{2.3}, w3{0.7}, w4{3.1};
    add_edge(0, 1, w1, g);
    add_edge(0, 2, w2, g);
    add_edge(1, 3, w3, g);
    add_edge(2, 3, w4, g);
    
    std::cout << "加权边:\\n";
    typedef graph_traits<Graph>::edge_iterator edge_iter;
    
    for (auto ep = edges(g); ep.first != ep.second; ++ep.first) {
        std::cout << "  " << source(*ep.first, g) 
                  << " - " << target(*ep.first, g)
                  << " (权重: " << g[*ep.first].weight << ")" << std::endl;
    }
    
    return 0;
}
```

---

## 深度优先搜索

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/depth_first_search.hpp>
#include <iostream>

struct dfs_visitor : public boost::default_dfs_visitor {
    template <typename Vertex, typename Graph>
    void discover_vertex(Vertex v, const Graph&) const {
        std::cout << "发现顶点: " << v << std::endl;
    }
};

int main() {
    using namespace boost;
    
    typedef adjacency_list<vecS, vecS, directedS> Graph;
    Graph g(6);
    
    add_edge(0, 1, g);
    add_edge(0, 2, g);
    add_edge(1, 3, g);
    add_edge(2, 4, g);
    add_edge(3, 5, g);
    add_edge(4, 5, g);
    
    std::cout << "深度优先搜索:\\n";
    depth_first_search(g, visitor(dfs_visitor()));
    
    return 0;
}
```

---

## 广度优先搜索

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/breadth_first_search.hpp>
#include <iostream>

struct bfs_visitor : public boost::default_bfs_visitor {
    template <typename Vertex, typename Graph>
    void discover_vertex(Vertex v, const Graph&) const {
        std::cout << "发现顶点: " << v << std::endl;
    }
};

int main() {
    using namespace boost;
    
    typedef adjacency_list<vecS, vecS, undirectedS> Graph;
    Graph g(6);
    
    add_edge(0, 1, g);
    add_edge(0, 2, g);
    add_edge(1, 3, g);
    add_edge(1, 4, g);
    add_edge(2, 5, g);
    
    std::cout << "广度优先搜索（从顶点0开始）:\\n";
    breadth_first_search(g, 0, visitor(bfs_visitor()));
    
    return 0;
}
```

---

## Dijkstra 最短路径

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/dijkstra_shortest_paths.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost;
    
    typedef adjacency_list<vecS, vecS, directedS, no_property,
                          property<edge_weight_t, double>> Graph;
    typedef graph_traits<Graph>::vertex_descriptor Vertex;
    
    Graph g(5);
    
    add_edge(0, 1, 1.0, g);
    add_edge(0, 2, 5.0, g);
    add_edge(1, 2, 2.0, g);
    add_edge(1, 3, 6.0, g);
    add_edge(2, 3, 1.0, g);
    add_edge(2, 4, 3.0, g);
    add_edge(3, 4, 4.0, g);
    
    std::vector<Vertex> predecessors(num_vertices(g));
    std::vector<double> distances(num_vertices(g));
    
    dijkstra_shortest_paths(g, 0,
        predecessor_map(&predecessors[0])
        .distance_map(&distances[0]));
    
    std::cout << "从顶点0到各顶点的最短距离:\\n";
    for (size_t i = 0; i < distances.size(); ++i) {
        std::cout << "  到顶点" << i << ": " << distances[i] << std::endl;
    }
    
    return 0;
}
```

---

## 拓扑排序

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/topological_sort.hpp>
#include <iostream>
#include <list>

int main() {
    using namespace boost;
    
    typedef adjacency_list<vecS, vecS, directedS> Graph;
    Graph g(6);
    
    // 任务依赖关系
    add_edge(0, 1, g);  // 任务0必须在任务1之前
    add_edge(0, 2, g);
    add_edge(1, 3, g);
    add_edge(2, 3, g);
    add_edge(3, 4, g);
    add_edge(3, 5, g);
    
    std::list<int> sorted;
    topological_sort(g, std::front_inserter(sorted));
    
    std::cout << "拓扑排序（任务执行顺序）:\\n";
    for (int task : sorted) {
        std::cout << "  任务" << task << std::endl;
    }
    
    return 0;
}
```

---

## 最小生成树 (Kruskal)

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/kruskal_min_spanning_tree.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost;
    
    typedef adjacency_list<vecS, vecS, undirectedS, no_property,
                          property<edge_weight_t, double>> Graph;
    typedef graph_traits<Graph>::edge_descriptor Edge;
    
    Graph g(5);
    
    add_edge(0, 1, 2.0, g);
    add_edge(0, 2, 3.0, g);
    add_edge(1, 2, 1.0, g);
    add_edge(1, 3, 4.0, g);
    add_edge(2, 3, 5.0, g);
    add_edge(2, 4, 6.0, g);
    add_edge(3, 4, 2.0, g);
    
    std::vector<Edge> mst;
    kruskal_minimum_spanning_tree(g, std::back_inserter(mst));
    
    std::cout << "最小生成树的边:\\n";
    double total_weight = 0;
    for (const Edge& e : mst) {
        double w = get(edge_weight, g, e);
        std::cout << "  " << source(e, g) << " - " << target(e, g)
                  << " (权重: " << w << ")" << std::endl;
        total_weight += w;
    }
    std::cout << "总权重: " << total_weight << std::endl;
    
    return 0;
}
```

---

## 连通分量

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/connected_components.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost;
    
    typedef adjacency_list<vecS, vecS, undirectedS> Graph;
    Graph g(8);
    
    // 第一个连通分量
    add_edge(0, 1, g);
    add_edge(1, 2, g);
    add_edge(2, 0, g);
    
    // 第二个连通分量
    add_edge(3, 4, g);
    add_edge(4, 5, g);
    
    // 第三个连通分量
    add_edge(6, 7, g);
    
    std::vector<int> component(num_vertices(g));
    int num = connected_components(g, &component[0]);
    
    std::cout << "连通分量数: " << num << std::endl;
    std::cout << "顶点所属分量:\\n";
    for (size_t i = 0; i < component.size(); ++i) {
        std::cout << "  顶点" << i << ": 分量" << component[i] << std::endl;
    }
    
    return 0;
}
```

---

## PageRank 算法

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/page_rank.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost;
    
    typedef adjacency_list<vecS, vecS, directedS> Graph;
    Graph g(5);
    
    // 构建网页链接图
    add_edge(0, 1, g);
    add_edge(0, 2, g);
    add_edge(1, 2, g);
    add_edge(2, 0, g);
    add_edge(2, 3, g);
    add_edge(3, 4, g);
    add_edge(4, 2, g);
    
    std::vector<double> ranks(num_vertices(g));
    
    page_rank(g, make_iterator_property_map(
        ranks.begin(), get(vertex_index, g)));
    
    std::cout << "PageRank 值:\\n";
    for (size_t i = 0; i < ranks.size(); ++i) {
        std::cout << "  页面" << i << ": " << ranks[i] << std::endl;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Graph 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/graph/doc/index.html)
- [图算法教程](https://www.boost.org/doc/libs/1_90_0/libs/graph/doc/quick_tour.html)
