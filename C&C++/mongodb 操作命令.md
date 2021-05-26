http://mongocxx.org/mongocxx-v3/tutorial/

>[root@dwj ~]# c++ --std=c++11 mongo.cpp -o testomongo $(pkg-config --cflags --libs libmongocxx)

```c++
#include <iostream>

#include <bsoncxx/builder/stream/document.hpp>
#include <bsoncxx/json.hpp>

#include <mongocxx/client.hpp>
#include <mongocxx/instance.hpp>

int main(int, char**)
{
    mongocxx::instance inst{};

    //mongodb://user:password@ip:port
    mongocxx::client conn{mongocxx::uri{"mongodb://cvap:Gsafety.@10.3.9.106:27017"}};

    //创建数据记录行/文档
    bsoncxx::builder::stream::document document{};

    // 一种获取方式
    // auto collection = conn["cvap"];
    // auto cursor = collection["user_info"].find({});

    //数据库/表
    auto collection = conn["cvap"]["user_info"];

    //查询所有数据，cursor是一行记录
    auto cursor = collection.find({});

    //查询第一条记录
    bsoncxx::stdx::optional<bsoncxx::document::value> maybe_result = collection.find_one({});
    if(maybe_result) {
        // Do something with *maybe_result;
        std::cout << bsoncxx::to_json(*maybe_result) << std::endl;
    }

    decltype(collection) col=collection;
    std::cout<<"type: --->"<<typeid(collection).name()<<std::endl;

    //罗列全部数据库
    auto dbcur = conn.list_databases();
    for(auto && db:dbcur){ std::cout<<bsoncxx::to_json(db)<<std::endl; }

    //按照条件查询数据
    cursor = collection.find(bsoncxx::builder::basic::make_document(bsoncxx::builder::basic::kvp("name", "test")));

    //获取表中记录总数量，新版函数
    std::cout<<"\n count: --->"<<collection.count_documents({})<<std::endl;

    //doc 是表中一条记录,循环输出所有记录
    for (auto && doc : cursor)
    {
        std::cout << bsoncxx::to_json(doc) << std::endl;
    }

    //插入数据
    document <<"name" <<"gjsy"
    <<"account" << "gsafety"
    <<"password" << "xxabcd,c123"
    <<"time" << "1007"
    <<"_class" << bsoncxx::builder::stream::open_array
    << "com" << "gsafety" << "cvap" << bsoncxx::builder::stream::close_array
    <<"address" <<bsoncxx::builder::stream::open_document
    <<"city"<<"shenzhen"
    <<"nation"<<"china"
    <<"phone"<<13524531211
    <<bsoncxx::builder::stream::close_document;

    //插入文档
    //collection.insert_one(document.view());

    //更新文档,需要document.clear()
    bsoncxx::stdx::optional<mongocxx::result::update> result =
    collection.update_one(document << "name" << "gjsy" << bsoncxx::builder::stream::finalize,
        document << "$set" << bsoncxx::builder::stream::open_document
        << "name" << "chenan"
        << bsoncxx::builder::stream::close_document
        << bsoncxx::builder::stream::finalize);
    std::cout<<"update count:" << result->modified_count() << std::endl;

    //删除文档
    collection.delete_one((bsoncxx::builder::basic::make_document(bsoncxx::builder::basic::kvp("name", "gjsy"))));
    //collection.delete_one(document.view());
    // collection.delete_many(document.view());

    std::cout<<"the document:"<<bsoncxx::to_json(document) << std::endl;

    //按完整数组查找
    bsoncxx::builder::stream::document finddocument;
    finddocument <<"_class" << bsoncxx::builder::stream::open_array
    << "com" << "gsafety" << "cvap"
    << bsoncxx::builder::stream::close_array;
    //按数组的一部分内容查找
    //finddocument<<"3D" <<48;

    auto arr_cursor = collection.find(finddocument.view());

    //doc 是表中一条记录,循环输出所有记录
    for (auto && doc : arr_cursor)
    {
        std::cout <<  "array find: "  << bsoncxx::to_json(doc) << std::endl;
    }

    finddocument.clear();
    //嵌套文档查找
    finddocument <<"address" <<bsoncxx::builder::stream::open_document
    <<"city"<<"shenzhen"
    <<"nation"<<"china"
    <<"phone"<<13524531211
    <<bsoncxx::builder::stream::close_document;

    auto doc_cursor = collection.find(finddocument.view());

    for (auto && doc : doc_cursor)
    {
        std::cout <<  "nesting doc find: "  << bsoncxx::to_json(doc) << std::endl;
    }

    //获取数据记录行字段attribute和value
    bsoncxx::document::view view = document.view();
    bsoncxx::document::element element = view["name"];
    if(element.type() == bsoncxx::type::k_string) {
        std::string name = element.get_string().value.to_string();
        std::cout <<  "element value : "  << name << std::endl;
    }
}
```mongo.cpp
