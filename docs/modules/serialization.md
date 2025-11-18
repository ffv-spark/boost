# Boost.Serialization - 对象序列化库

## 概述

Boost.Serialization 提供对象序列化和反序列化功能，支持多种存档格式。

**类型**: 需要编译链接的库

**链接库**: `-lboost_serialization`

---

## 快速开始

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/serialization/string.hpp>
#include <iostream>
#include <fstream>
#include <string>

class Person {
public:
    Person() = default;
    Person(const std::string& name, int age) : name_(name), age_(age) {}

    void print() const {
        std::cout << "Name: " << name_ << ", Age: " << age_ << std::endl;
    }

private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & name_;
        ar & age_;
    }

    std::string name_;
    int age_;
};

int main() {
    // 序列化
    {
        std::ofstream ofs("person.txt");
        boost::archive::text_oarchive oa(ofs);

        Person p("Alice", 25);
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

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_serialization -o example
```

---

## 存档格式

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/archive/binary_oarchive.hpp>
#include <boost/archive/binary_iarchive.hpp>
#include <boost/archive/xml_oarchive.hpp>
#include <boost/archive/xml_iarchive.hpp>
#include <fstream>

struct Data {
    int value;
    std::string text;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & BOOST_SERIALIZATION_NVP(value);
        ar & BOOST_SERIALIZATION_NVP(text);
    }
};

int main() {
    Data data{42, "Hello"};

    // 1. 文本格式
    {
        std::ofstream ofs("data.txt");
        boost::archive::text_oarchive oa(ofs);
        oa << data;
    }

    // 2. 二进制格式
    {
        std::ofstream ofs("data.bin", std::ios::binary);
        boost::archive::binary_oarchive oa(ofs);
        oa << data;
    }

    // 3. XML 格式
    {
        std::ofstream ofs("data.xml");
        boost::archive::xml_oarchive oa(ofs);
        oa << BOOST_SERIALIZATION_NVP(data);
    }

    return 0;
}
```

---

## STL 容器序列化

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/serialization/vector.hpp>
#include <boost/serialization/map.hpp>
#include <boost/serialization/set.hpp>
#include <boost/serialization/list.hpp>
#include <fstream>

int main() {
    // 序列化
    {
        std::ofstream ofs("containers.txt");
        boost::archive::text_oarchive oa(ofs);

        std::vector<int> vec = {1, 2, 3, 4, 5};
        std::map<std::string, int> map = {{"a", 1}, {"b", 2}};

        oa << vec;
        oa << map;
    }

    // 反序列化
    {
        std::ifstream ifs("containers.txt");
        boost::archive::text_iarchive ia(ifs);

        std::vector<int> vec;
        std::map<std::string, int> map;

        ia >> vec;
        ia >> map;

        std::cout << "Vector: ";
        for (int x : vec) std::cout << x << " ";
        std::cout << "\nMap: ";
        for (const auto& [k, v] : map) {
            std::cout << k << "=" << v << " ";
        }
        std::cout << std::endl;
    }

    return 0;
}
```

---

## 继承关系序列化

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/serialization/base_object.hpp>
#include <boost/serialization/export.hpp>
#include <fstream>

class Animal {
public:
    virtual ~Animal() = default;
    virtual void speak() const = 0;

protected:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & name_;
    }

    std::string name_;
};

class Dog : public Animal {
public:
    Dog() = default;
    Dog(const std::string& name) { name_ = name; }

    void speak() const override {
        std::cout << name_ << " says: Woof!" << std::endl;
    }

private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & boost::serialization::base_object<Animal>(*this);
    }
};

BOOST_CLASS_EXPORT(Dog)

int main() {
    // 序列化
    {
        std::ofstream ofs("animal.txt");
        boost::archive::text_oarchive oa(ofs);

        Animal* animal = new Dog("Buddy");
        oa << animal;
        delete animal;
    }

    // 反序列化
    {
        std::ifstream ifs("animal.txt");
        boost::archive::text_iarchive ia(ifs);

        Animal* animal = nullptr;
        ia >> animal;
        animal->speak();
        delete animal;
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
#include <fstream>

class VersionedClass {
public:
    VersionedClass() : value_(0), new_field_(0.0) {}

private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & value_;

        // 版本 1 添加的字段
        if (version >= 1) {
            ar & new_field_;
        }
    }

    int value_;
    double new_field_;  // 版本 1 新增
};

BOOST_CLASS_VERSION(VersionedClass, 1)

int main() {
    // 使用新版本序列化
    {
        std::ofstream ofs("versioned.txt");
        boost::archive::text_oarchive oa(ofs);

        VersionedClass obj;
        oa << obj;
    }

    return 0;
}
```

---

## 智能指针序列化

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/serialization/shared_ptr.hpp>
#include <boost/serialization/unique_ptr.hpp>
#include <memory>
#include <fstream>

class Resource {
public:
    Resource(int id = 0) : id_(id) {}

private:
    friend class boost::serialization::access;

    template<class Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & id_;
    }

    int id_;
};

int main() {
    // 序列化
    {
        std::ofstream ofs("pointers.txt");
        boost::archive::text_oarchive oa(ofs);

        auto shared = std::make_shared<Resource>(42);
        auto unique = std::make_unique<Resource>(99);

        oa << shared;
        oa << unique;
    }

    // 反序列化
    {
        std::ifstream ifs("pointers.txt");
        boost::archive::text_iarchive ia(ifs);

        std::shared_ptr<Resource> shared;
        std::unique_ptr<Resource> unique;

        ia >> shared;
        ia >> unique;
    }

    return 0;
}
```

---

## 最佳实践

1. **私有序列化**: 使用 friend access
2. **版本管理**: 为可能变化的类添加版本
3. **NVP**: XML 格式使用命名值对
4. **性能**: 二进制格式最快
5. **可读性**: 文本格式便于调试

---

## 参考资源

- [Boost.Serialization 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/serialization/doc/index.html)
