# Boost.Graph - 图算法库

## 概述

Boost.Graph (BGL) 提供通用的图数据结构和算法，支持多种图表示方法和丰富的图算法。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/graph_traits.hpp>
#include <iostream>

using namespace boost;

int main() {
    // 创建无向图
    typedef adjacency_list<vecS, vecS, undirectedS> Graph;

    Graph g(5);  // 5 个顶点

    // 添加边
    add_edge(0, 1, g);
    add_edge(0, 2, g);
    add_edge(1, 3, g);
    add_edge(2, 3, g);
    add_edge(3, 4, g);

    // 输出图信息
    std::cout << "顶点数: " << num_vertices(g) << std::endl;
    std::cout << "边数: " << num_edges(g) << std::endl;

    // 遍历所有边
    graph_traits<Graph>::edge_iterator ei, ei_end;
    std::cout << "边列表:\n";
    for (tie(ei, ei_end) = edges(g); ei != ei_end; ++ei) {
        std::cout << "  " << source(*ei, g) << " -- "
                  << target(*ei, g) << std::endl;
    }

    return 0;
}
```

---

## 有向图和带权重的边

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <iostream>

using namespace boost;

int main() {
    // 有向图，边带权重
    typedef adjacency_list<vecS, vecS, directedS,
                          no_property,  // 顶点属性
                          property<edge_weight_t, double>  // 边权重
                          > Graph;
    typedef graph_traits<Graph>::edge_descriptor Edge;

    Graph g(4);

    // 添加带权重的边
    Edge e;
    bool inserted;

    tie(e, inserted) = add_edge(0, 1, 2.5, g);
    tie(e, inserted) = add_edge(0, 2, 1.0, g);
    tie(e, inserted) = add_edge(1, 3, 3.5, g);
    tie(e, inserted) = add_edge(2, 3, 0.5, g);

    // 获取权重属性映射
    property_map<Graph, edge_weight_t>::type weight_map = get(edge_weight, g);

    // 输出边和权重
    graph_traits<Graph>::edge_iterator ei, ei_end;
    std::cout << "边和权重:\n";
    for (tie(ei, ei_end) = edges(g); ei != ei_end; ++ei) {
        std::cout << source(*ei, g) << " -> " << target(*ei, g)
                  << " [" << weight_map[*ei] << "]" << std::endl;
    }

    return 0;
}
```

---

## 深度优先搜索 (DFS)

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/depth_first_search.hpp>
#include <iostream>
#include <vector>

using namespace boost;

// DFS 访问器
class DFSVisitor : public default_dfs_visitor {
public:
    template <typename Vertex, typename Graph>
    void discover_vertex(Vertex v, const Graph&) const {
        std::cout << "发现顶点: " << v << std::endl;
    }

    template <typename Edge, typename Graph>
    void examine_edge(Edge e, const Graph& g) const {
        std::cout << "检查边: " << source(e, g) << " -> "
                  << target(e, g) << std::endl;
    }
};

int main() {
    typedef adjacency_list<vecS, vecS, directedS> Graph;

    Graph g(6);
    add_edge(0, 1, g);
    add_edge(0, 2, g);
    add_edge(1, 3, g);
    add_edge(2, 4, g);
    add_edge(3, 5, g);
    add_edge(4, 5, g);

    std::cout << "深度优先搜索:\n";
    DFSVisitor vis;
    depth_first_search(g, visitor(vis));

    return 0;
}
```

---

## 广度优先搜索 (BFS)

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/breadth_first_search.hpp>
#include <iostream>

using namespace boost;

// BFS 访问器
class BFSVisitor : public default_bfs_visitor {
public:
    template <typename Vertex, typename Graph>
    void discover_vertex(Vertex v, const Graph&) const {
        std::cout << "访问顶点: " << v << std::endl;
    }
};

int main() {
    typedef adjacency_list<vecS, vecS, undirectedS> Graph;

    Graph g(6);
    add_edge(0, 1, g);
    add_edge(0, 2, g);
    add_edge(1, 3, g);
    add_edge(2, 4, g);
    add_edge(3, 5, g);
    add_edge(4, 5, g);

    std::cout << "广度优先搜索（从顶点 0 开始）:\n";
    BFSVisitor vis;
    breadth_first_search(g, 0, visitor(vis));

    return 0;
}
```

---

## Dijkstra 最短路径算法

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/dijkstra_shortest_paths.hpp>
#include <iostream>
#include <vector>

using namespace boost;

int main() {
    typedef adjacency_list<vecS, vecS, directedS,
                          no_property,
                          property<edge_weight_t, double>> Graph;
    typedef graph_traits<Graph>::vertex_descriptor Vertex;

    Graph g(5);

    // 添加带权重的边
    add_edge(0, 1, 1.0, g);
    add_edge(0, 2, 5.0, g);
    add_edge(1, 2, 2.0, g);
    add_edge(1, 3, 6.0, g);
    add_edge(2, 3, 1.0, g);
    add_edge(2, 4, 3.0, g);
    add_edge(3, 4, 1.0, g);

    // 存储前驱和距离
    std::vector<Vertex> predecessors(num_vertices(g));
    std::vector<double> distances(num_vertices(g));

    Vertex start = 0;

    // 计算最短路径
    dijkstra_shortest_paths(
        g, start,
        predecessor_map(&predecessors[0]).distance_map(&distances[0])
    );

    // 输出从起点到所有顶点的最短距离
    std::cout << "从顶点 " << start << " 到各顶点的最短距离:\n";
    for (size_t i = 0; i < distances.size(); ++i) {
        std::cout << "  到顶点 " << i << ": " << distances[i] << std::endl;
    }

    // 输出到特定顶点的路径
    Vertex target = 4;
    std::cout << "\n从 " << start << " 到 " << target << " 的路径: ";

    std::vector<Vertex> path;
    for (Vertex v = target; v != start; v = predecessors[v]) {
        path.push_back(v);
    }
    path.push_back(start);

    for (auto it = path.rbegin(); it != path.rend(); ++it) {
        std::cout << *it;
        if (it + 1 != path.rend()) std::cout << " -> ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 拓扑排序

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/topological_sort.hpp>
#include <iostream>
#include <vector>

using namespace boost;

int main() {
    typedef adjacency_list<vecS, vecS, directedS> Graph;
    typedef graph_traits<Graph>::vertex_descriptor Vertex;

    Graph g(6);

    // 构建有向无环图 (DAG)
    add_edge(0, 1, g);
    add_edge(0, 2, g);
    add_edge(1, 3, g);
    add_edge(2, 3, g);
    add_edge(2, 4, g);
    add_edge(3, 5, g);
    add_edge(4, 5, g);

    std::vector<Vertex> sorted;

    // 拓扑排序
    topological_sort(g, std::back_inserter(sorted));

    std::cout << "拓扑排序结果: ";
    for (auto it = sorted.rbegin(); it != sorted.rend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 最小生成树 (Kruskal 算法)

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/kruskal_min_spanning_tree.hpp>
#include <iostream>
#include <vector>

using namespace boost;

int main() {
    typedef adjacency_list<vecS, vecS, undirectedS,
                          no_property,
                          property<edge_weight_t, double>> Graph;
    typedef graph_traits<Graph>::edge_descriptor Edge;

    Graph g(6);

    // 添加带权重的边
    add_edge(0, 1, 4.0, g);
    add_edge(0, 2, 3.0, g);
    add_edge(1, 2, 1.0, g);
    add_edge(1, 3, 2.0, g);
    add_edge(2, 3, 4.0, g);
    add_edge(2, 4, 3.0, g);
    add_edge(3, 4, 2.0, g);
    add_edge(3, 5, 6.0, g);
    add_edge(4, 5, 1.0, g);

    // 计算最小生成树
    std::vector<Edge> spanning_tree;
    kruskal_minimum_spanning_tree(g, std::back_inserter(spanning_tree));

    // 获取权重映射
    property_map<Graph, edge_weight_t>::type weight_map = get(edge_weight, g);

    // 输出最小生成树
    double total_weight = 0;
    std::cout << "最小生成树的边:\n";
    for (const auto& e : spanning_tree) {
        double weight = weight_map[e];
        total_weight += weight;
        std::cout << "  " << source(e, g) << " -- " << target(e, g)
                  << " [" << weight << "]" << std::endl;
    }

    std::cout << "总权重: " << total_weight << std::endl;

    return 0;
}
```

---

## 顶点和边属性

```cpp
#include <boost/graph/adjacency_list.hpp>
#include <iostream>
#include <string>

using namespace boost;

// 顶点属性
struct VertexProperties {
    std::string name;
    int value;
};

// 边属性
struct EdgeProperties {
    double weight;
    std::string label;
};

int main() {
    typedef adjacency_list<vecS, vecS, directedS,
                          VertexProperties,
                          EdgeProperties> Graph;
    typedef graph_traits<Graph>::vertex_descriptor Vertex;

    Graph g;

    // 添加顶点并设置属性
    Vertex v0 = add_vertex(g);
    g[v0].name = "Start";
    g[v0].value = 100;

    Vertex v1 = add_vertex(g);
    g[v1].name = "Middle";
    g[v1].value = 200;

    Vertex v2 = add_vertex(g);
    g[v2].name = "End";
    g[v2].value = 300;

    // 添加边并设置属性
    auto e01 = add_edge(v0, v1, g).first;
    g[e01].weight = 5.5;
    g[e01].label = "路径 A";

    auto e12 = add_edge(v1, v2, g).first;
    g[e12].weight = 3.2;
    g[e12].label = "路径 B";

    // 访问属性
    std::cout << "顶点信息:\n";
    graph_traits<Graph>::vertex_iterator vi, vi_end;
    for (tie(vi, vi_end) = vertices(g); vi != vi_end; ++vi) {
        std::cout << "  " << g[*vi].name << " (值: "
                  << g[*vi].value << ")\n";
    }

    std::cout << "\n边信息:\n";
    graph_traits<Graph>::edge_iterator ei, ei_end;
    for (tie(ei, ei_end) = edges(g); ei != ei_end; ++ei) {
        std::cout << "  " << g[*ei].label << ": "
                  << g[source(*ei, g)].name << " -> "
                  << g[target(*ei, g)].name
                  << " [权重: " << g[*ei].weight << "]\n";
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Graph 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/graph/doc/index.html)
- [BGL 用户指南](https://www.boost.org/doc/libs/1_90_0/libs/graph/doc/using_adjacency_list.html)
