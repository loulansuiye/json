#### Description
This is a high-performance JSON library written in C++ 11 with the following features:
1. **header only**: no third-party dependencies;
2. **fast**: parsing speed is half of simdjson, much faster than rapidjson;
2. **simple api**: the common interface is similar to nlohmann / json;
3. **modularity**: You can customize various sub-modules to meet customized needs;
4. **json value type extension**: You can extend the new JSON basic type by yourself;
5. **support types serialization**: You can use JSON as an any class;
#### Supported compilers
+  GCC 4.8.5 or later 
+  Clang 3.4 or later
+  Microsoft Visual C++ 2015 or later
#### Performance 
+  Run the test program in the "./tests" directory to see the performance comparison between this library and rapidjson;
+  Direct manipulation of raw data can also improve performance

#### Examples
Read the test code in the tests directory for more usage. 
Basic usage:
```C++
#include "json.hpp"
#include <iostream>

int main(int argc, char** argv) {
    amo::json json_empty;                               // json_value_empty
    amo::json json_string = "ttt";                      // json_value_string
    amo::json json_number = 12.33;                      // json_value_float
    json_number = 3;                                    // json_value_signed
    json_number = 3u;                                   // json_value_unsigned
    amo::json json_boolean = true;                      // json_value_boolean
    amo::json json_null = nullptr;                      // json_value_null;
    amo::json json_object_{ amo::json_value_object };   // json_value_object;
    amo::json json_array_{ amo::json_value_array };     // json_value_array;
    
    amo::json json;                                     // empty json
    json["string"] = "amo::json";                       // {"string": "amo::json"}
    json["string"] = "amo.json";                        // {"string": "amo.json"}
    json["array"] = { 1, 0, 0, 0 };                     // {"string": "amo.json", "array": [1,0,0,0]};
    json["sub"] = { {"a", 1}, {"b", 1.1} };             // {"string": "amo.json", "array": [1,0,0,0], "sub":{"a": 1, "b": 1.1} };
    std::string str = json.to_string();                 // to string
    std::cout << str << std::endl;
    std::string s = json["string"];                     // get string from json;
    
    amo::json json2 = amo::string_reader(str);          // parse json from string
    bool has_string = json2.has("string");              // has key
    if (json2["sub"].is_object()) {                     // is json object
        amo::json sub = json2["sub"];                   // ref from json["sub"];
        amo::json sub2 = json2["sub"].get<amo::json>(); // copy from json["sub"];
        sub["a"] = 10.23;
        sub["c"] = true;
        
        std::cout << "[sub][a]=" << json2["sub"]["a"].get<int>() << std::endl;        // get<T>()
        std::cout << "[sub][a]=" << json2["sub"]["a"].get<double>() << std::endl;     // get<T>()
        std::cout << "[sub][b]=" << json2["sub"]["b"] << std::endl;                   // operator T()
        std::cout << "[sub][c]=" << json2["sub"]["c"] << std::endl;                   // operator T()
        if (sub2.has("c")) {
            // never execute
            std::cout << "[sub][c]=" << json2["sub"]["c"] << std::endl;               // operator <<
        }
    }
    
    if (json2["array"].is(amo::json_value_array)) {   // is json array
        amo::json array = json2["array"];             // ref from json2["array"]
        array[-1] = 3;                                // push back
        array.push_back(1);                           // push back
        array[0] = 2;                                 // modify value at index 0
    }
    
    std::cout << json2.to_string(4) << std::endl;     // prettify
    json2.to_file("./xxx.json");                      // write to file
    
    amo::json json3 = amo::file_reader("./xxx.json");           // parse json from file
    amo::json arr;                                              // empty json
    // arr.set_array();    // set to array
    arr.push_back(0);
    arr.push_back({1, 2, 3, 4, 5, false, true, "string txt" }); // push back with initializer_list
    std::vector<int> vec{9, 10, 11, 12 };
    arr.push_back(vec.begin(), vec.end());                      // push back with iterator
    
    // transfer json array
    for (const amo::json& p : arr) {
        std::cout << p << std::endl;
    }
    // transfer json array with array index
    arr.transfer<int>([](const int& index, amo::json & p) {
        std::cout << index << ": " << p << std::endl;
    });
    
    // transfer json object
    for (auto& p : json2) {
        std::cout << p << std::endl;   //  without key , only value
    }
    // transfer json object with key
    json2.transfer<amo::json::key_type>([](const amo::json::key_type & key, amo::json & p) {
        std::cout << key << ": " << p << std::endl;
    });

    
    return 0;
}

```

Custom json value type:
```C++
#include "json.hpp"

class custom_json_value {
public:
    const static int __json_type_value__ = 136;         // require  custom json_value_type must > 128
public:
    custom_json_value() {}
    custom_json_value(const std::string& str) : m_str(str) {}
    std::string to_string() const { return m_str; } // require
    static custom_json_value from_string(const std::string& str) { return custom_json_value(str); } // require
    std::string m_str;
};


int main(int argc, char** argv){
     amo::json json = custom_json_value("123");          // construct json with custom json value
    bool b = json.is(custom_json_value::__json_type_value__);     //
    std::cout << "json value type: " << b << std::endl;
    custom_json_value value = json;                     // get custom_json_value from json
    std::cout << value.m_str << std::endl;
    custom_json_value* value2 = json;                   // get custom_json_value* from json
    std::cout << value2->m_str << std::endl;
    delete value2;                                      // delete pointer
    json = "456";
    std::shared_ptr<custom_json_value> value3 = json;   // smart ptr
    std::cout << value3->m_str << std::endl;
    
    // use custom_json_value as a normal json value type
    json.set_object();
    json["custom_json_value"] = value;
    json["custom_json_value3"] = value3;
    std::cout << json.to_string() << std::endl;

    return 0;
}
```
Serialization / Deserialization:
```C++
#include "json.hpp"
#include <iostream>

// simplest class
class custom_json_object {
public:
    custom_json_object() {}
    custom_json_object(const std::string& str) : m_str(str) {}
    amo::json to_json() const {
        amo::json json;
        json["m_str"] = m_str;
        return (json);
    }
    
    static custom_json_object* from_json(const amo::json& json, custom_json_object* ptr) {
        if (ptr == nullptr) {
            ptr = new custom_json_object();
        }
        
        ptr->m_str = json["m_str"].get<std::string>();
        return ptr;
    }
    std::string m_str;
    
};

// use macro
class custom_json_object2 {
public:
    typedef amo::json JsonType;     // require,  json type
public:
    custom_json_object2() { m_int = 3; }
    
    AMO_ENTITY_ARGS_GET_BEGIN_WITH_JSON_CONSTRUCTOR(custom_json_object2)
    AMO_ENTITY_ARGS_GET(m_int)      // get value from json["m_int"] if json["m_int"] is not empty
    AMO_ENTITY_ARGS_GET(m_int2)
    AMO_ENTITY_ARGS_GET(m_vec)
    AMO_ENTITY_ARGS_GET(m_custom_json_object)
    AMO_ENTITY_ARGS_GET_END()
    
    AMO_ENTITY_ARGS_SET_BEGIN(custom_json_object2)
    AMO_ENTITY_ARGS_SET(m_int) // set value to json["m_int"]
    AMO_ENTITY_ARGS_SET(m_int2)
    AMO_ENTITY_ARGS_SET(m_vec)
    AMO_ENTITY_ARGS_SET(m_custom_json_object)
    AMO_ENTITY_ARGS_SET_END()
    
    int m_int;      // pod
    std::shared_ptr<int> m_int2;        // smart ptr
    std::vector<int> m_vec;             // stl container
    std::shared_ptr<custom_json_object>  m_custom_json_object;  // smart ptr  , class 
};



int main(int argc, char** argv){
     custom_json_object json_class;
    json_class.m_str = "3";
    amo::json simplejson = json_class;
    std::cout << simplejson << std::endl;       // {"m_str":"3"}
    
    custom_json_object2 c1;
    c1.m_vec.push_back(1);
    c1.m_vec.push_back(2);
    
    amo::json json = c1;
    std::cout << c1.to_string() << std::endl;   // {"m_int":3,"m_vec":[1,2]}
    std::cout << json.to_string() << std::endl; // {"m_int":3,"m_vec":[1,2]}
    json["m_int2"] = 3;
    
    custom_json_object2 c2 = json;
    std::cout << *c2.m_int2 << std::endl; // 3
    
    json["m_custom_json_object"] = custom_json_object("657");
    std::shared_ptr<custom_json_object2> c3 = json;
    std::cout << c3->m_custom_json_object->m_str << std::endl; // 567
    std::cout << c3->to_string() << std::endl;//{"m_int":3,"m_int2":3,"m_vec":[1,2],"m_custom_json_object":{"m_str":"657"}}
 
    return 0;
}


```
Use STL containers:
```C++
// construct from stl container
std::vector<int> vec{1,2,3,4,5,6,7};
amo::json json_vec = vec;  // json_value_array;
amo::json json_list = std::list<int>{1,2,3,4};  // json_value_array;
amo::json json_set = std::set<int>{1,2,3,4}; // json_value_array;
amo::json json_deque = std::deque<int>{1,2,3,4}; // json_value_array;
amo::json json_object_map = std::map<std::string, int>{{"a",1}, {"b", 2}}; // json_value_object
amo::json json_arr_map = std::map<int, int>{{1,1},{1,1}}; // json_value_array
amo::json json_vec2 = std::vector<amo::json::object_type>{"1",2,false, 33}

std::vector<int> my_vec = json_vec;    // ok
    
```
 
You can directly use native data to improve program performance:
```C++
amo::json json;
for (int i =0; i < 10000; ++i) {
    json.push_back(i);
}

// Use the find function to avoid temporary JSON objects
for(int i=0; i < 10000; ++i){
    json.find(i).set_value(i+1, json.get_allocator());  // 
}
// Use the get_data_object function to use native data directly.
for(int i=0; i < 10000; ++i){
    json.get_data_object().d.a.elements.at(i) = i+2;
}
```

#### Defects
1. The program uses pre-allocated memory to improve Performance, it will waste some memory; 
2. JSON will not release any memory during the life cycle, and it will also waste some memory; 
3. When parsing a JSON string, the keys in key-value pairs are not processed by default; 
```C++
std::string sb = R"({"\u4E2D\u6587": "\u4e2d\u6587\u5b57\u7b26\u4e32"})";   // u8 {"中文": "中文字符串"}
amo::json json = amo::string_reader(sb);
std::cout << json.to_string() << std::endl; // u8 {"\u4E2D\u6587":"中文字符串"},  key unchanged
```
4.  When you call the function operator ["key"], if the key-value does not exist, a empty value will be created. This feature will cause the return value of the size function to be inaccurate. You can use has ( "key") to determine whether the corresponding key exists; 
```C++
amo::json json;
json["a"];
json["b"] = 1;
std::cout << json.size() << std::endl;      //  2
std::cout << json.to_string() << std::endl; // {"b": 1}
std::cout << json.has("a") << std::endl;    // true
std::cout << json.has("c") << std::endl;    // false

```
5. The program uses the fast insert mode by default when parsing a JSON string. It may behave inconsistently when there are duplicate key values in the string. 16 key-values are inserted using vector.push_back, and later key-values are used map.insert; you can modify the configuration to filter duplicate key-value pairs;  
```C++
std::string str = R"({"a": 1, "a": 2, "a": 3, "a":4, "a": 5, "a": 6, "a": 7, "a": 8, "a": 9, "a": 10, "a": 11, "a": 12, "a": 13, "a": 14, "a": 15, "a": 16, "a": 17, "a": 18, "a": 19, "a": 20 , "b": 1, "b":2  })";
amo::json json = amo::string_reader(str);
std::cout << json["a"] << std::endl;         // 1  index < 16 use vector.push_back
std::cout << json["b"] << std::endl;         // 2  index >= 16 use map.insert()
std::cout << json.to_string() << std::endl;  // {"a": 1, "a": 2, "a": 3, "a":4, "a": 5, "a": 6, "a": 7, "a": 8, "a": 9, "a": 10, "a": 11, "a": 12, "a": 13, "a": 14, "a": 15, "a": 16, "a": 20, "b":2  }

```