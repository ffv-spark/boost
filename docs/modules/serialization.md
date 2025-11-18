# Boost.Serialization - 序列化库

## 概述

Boost.Serialization 提供对象序列化和反序列化功能，支持多种格式。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <iostream>
#include <fstream>
#include <string>

class Person {
private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & name;
        ar & age;
    }

    std::string name;
    int age;

public:
    Person() {}
    Person(const std::string& n, int a) : name(n), age(a) {}

    void print() const {
        std::cout << name << ", " << age << " 岁" << std::endl;
    }
};

int main() {
    // 序列化
    {
        std::ofstream ofs("person.txt");
        boost::archive::text_oarchive oa(ofs);

        Person p("Alice", 30);
        oa << p;
    }

    // 反序列化
    {
        std::ifstream ifs("person.txt");
        boost::archive::text_iarchive ia(ifs);

        Person p;
        ia >> p;
        p.print();
    }

    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_serialization`

---

## 基本类型序列化

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <iostream>
#include <sstream>

int main() {
    // 序列化
    std::stringstream ss;
    {
        boost::archive::text_oarchive oa(ss);

        int i = 42;
        double d = 3.14;
        std::string s = "Hello";

        oa << i << d << s;
    }

    std::cout << "序列化数据:\n" << ss.str() << std::endl;

    // 反序列化
    {
        boost::archive::text_iarchive ia(ss);

        int i;
        double d;
        std::string s;

        ia >> i >> d >> s;

        std::cout << "\n反序列化:\n";
        std::cout << "i = " << i << std::endl;
        std::cout << "d = " << d << std::endl;
        std::cout << "s = " << s << std::endl;
    }

    return 0;
}
```

---

## 容器序列化

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/serialization/vector.hpp>
#include <boost/serialization/map.hpp>
#include <iostream>
#include <sstream>
#include <vector>
#include <map>

int main() {
    std::stringstream ss;

    // 序列化
    {
        boost::archive::text_oarchive oa(ss);

        std::vector<int> vec = {1, 2, 3, 4, 5};
        std::map<std::string, int> m = {{"one", 1}, {"two", 2}};

        oa << vec << m;
    }

    // 反序列化
    {
        boost::archive::text_iarchive ia(ss);

        std::vector<int> vec;
        std::map<std::string, int> m;

        ia >> vec >> m;

        std::cout << "vector: ";
        for (int x : vec) {
            std::cout << x << " ";
        }
        std::cout << "\nmap:\n";
        for (const auto& p : m) {
            std::cout << "  " << p.first << " = " << p.second << std::endl;
        }
    }

    return 0;
}
```

---

## 二进制归档

```cpp
#include <boost/archive/binary_oarchive.hpp>
#include <boost/archive/binary_iarchive.hpp>
#include <iostream>
#include <fstream>
#include <string>

class Data {
private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & value;
        ar & text;
    }

public:
    int value;
    std::string text;

    Data() : value(0) {}
    Data(int v, const std::string& t) : value(v), text(t) {}
};

int main() {
    // 二进制序列化
    {
        std::ofstream ofs("data.bin", std::ios::binary);
        boost::archive::binary_oarchive oa(ofs);

        Data d(42, "Binary data");
        oa << d;
    }

    // 二进制反序列化
    {
        std::ifstream ifs("data.bin", std::ios::binary);
        boost::archive::binary_iarchive ia(ifs);

        Data d;
        ia >> d;

        std::cout << "value = " << d.value << std::endl;
        std::cout << "text = " << d.text << std::endl;
    }

    return 0;
}
```

---

## XML 归档

```cpp
#include <boost/archive/xml_oarchive.hpp>
#include <boost/archive/xml_iarchive.hpp>
#include <iostream>
#include <fstream>
#include <string>

class Config {
private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & BOOST_SERIALIZATION_NVP(host);
        ar & BOOST_SERIALIZATION_NVP(port);
        ar & BOOST_SERIALIZATION_NVP(timeout);
    }

public:
    std::string host;
    int port;
    int timeout;

    Config() : port(0), timeout(0) {}
    Config(const std::string& h, int p, int t) 
        : host(h), port(p), timeout(t) {}
};

int main() {
    // XML 序列化
    {
        std::ofstream ofs("config.xml");
        boost::archive::xml_oarchive oa(ofs);

        Config cfg("localhost", 8080, 30);
        oa << BOOST_SERIALIZATION_NVP(cfg);
    }

    // XML 反序列化
    {
        std::ifstream ifs("config.xml");
        boost::archive::xml_iarchive ia(ifs);

        Config cfg;
        ia >> BOOST_SERIALIZATION_NVP(cfg);

        std::cout << "Host: " << cfg.host << std::endl;
        std::cout << "Port: " << cfg.port << std::endl;
        std::cout << "Timeout: " << cfg.timeout << std::endl;
    }

    return 0;
}
```

---

## 继承类序列化

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/serialization/base_object.hpp>
#include <iostream>
#include <sstream>
#include <string>

class Animal {
private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & name;
    }

protected:
    std::string name;

public:
    Animal() {}
    Animal(const std::string& n) : name(n) {}
    virtual ~Animal() {}

    std::string get_name() const { return name; }
};

class Dog : public Animal {
private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & boost::serialization::base_object<Animal>(*this);
        ar & breed;
    }

    std::string breed;

public:
    Dog() {}
    Dog(const std::string& n, const std::string& b) 
        : Animal(n), breed(b) {}

    std::string get_breed() const { return breed; }
};

int main() {
    std::stringstream ss;

    // 序列化
    {
        boost::archive::text_oarchive oa(ss);
        Dog dog("Buddy", "Golden Retriever");
        oa << dog;
    }

    // 反序列化
    {
        boost::archive::text_iarchive ia(ss);
        Dog dog;
        ia >> dog;

        std::cout << "Name: " << dog.get_name() << std::endl;
        std::cout << "Breed: " << dog.get_breed() << std::endl;
    }

    return 0;
}
```

---

## 指针序列化

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/serialization/shared_ptr.hpp>
#include <iostream>
#include <sstream>
#include <memory>
#include <string>

class Node {
private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & value;
        ar & next;
    }

public:
    int value;
    std::shared_ptr<Node> next;

    Node() : value(0) {}
    Node(int v) : value(v) {}
};

int main() {
    std::stringstream ss;

    // 序列化链表
    {
        boost::archive::text_oarchive oa(ss);

        auto n1 = std::make_shared<Node>(1);
        auto n2 = std::make_shared<Node>(2);
        auto n3 = std::make_shared<Node>(3);

        n1->next = n2;
        n2->next = n3;

        oa << n1;
    }

    // 反序列化
    {
        boost::archive::text_iarchive ia(ss);

        std::shared_ptr<Node> n1;
        ia >> n1;

        std::cout << "链表: ";
        for (auto p = n1; p; p = p->next) {
            std::cout << p->value << " ";
        }
        std::cout << std::endl;
    }

    return 0;
}
```

---

## 版本控制

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/serialization/version.hpp>
#include <iostream>
#include <sstream>
#include <string>

class Document {
private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & title;
        ar & content;

        // 版本2添加了作者字段
        if (version >= 2) {
            ar & author;
        }
    }

public:
    std::string title;
    std::string content;
    std::string author;

    Document() {}
    Document(const std::string& t, const std::string& c, const std::string& a)
        : title(t), content(c), author(a) {}
};

BOOST_CLASS_VERSION(Document, 2)

int main() {
    std::stringstream ss;

    // 序列化版本2
    {
        boost::archive::text_oarchive oa(ss);
        Document doc("Title", "Content", "Alice");
        oa << doc;
    }

    // 反序列化
    {
        boost::archive::text_iarchive ia(ss);
        Document doc;
        ia >> doc;

        std::cout << "标题: " << doc.title << std::endl;
        std::cout << "内容: " << doc.content << std::endl;
        std::cout << "作者: " << doc.author << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Serialization 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/serialization/doc/index.html)
