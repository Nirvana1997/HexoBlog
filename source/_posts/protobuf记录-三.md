---
title: protobuf记录(三)——c++中的使用
date: 2018-06-22 12:45:43
category: protobuf
tags:
- protobuf 
- C++ 
- 序列化
---
## 前言
没有前言了

<!-- more -->

## 1.提供接口
### 属性接口
在编译完成后的.h文件中，有非常多的方法供调用。
上一次定义了addressbook.proto，这里再放上来：

```
syntax = "proto2";

package tutorial;

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```
通过这份.proto文件编译成的.h文件中有非常多的接口和方法。先来看关于数据项的一些接口：

``` c++
  // name,主要以他为例,下面说明每个方法的作用
  // has_xxx(),用来判断该值是否已经赋过值
  inline bool has_name() const;
  // clear_xxx(),用来判断该值是否已经赋过值
  inline void clear_name();
  // xxx(),获取对应属性的值,即我们一般使用的get方法
  inline const ::std::string& name() const;
  // set_xxx(),修改对应属性的值,此处参数是string,c++11中会多一个这个方法
  inline void set_name(const ::std::string& value);
  // set_xxx(),修改对应属性的值,此处参数是char*
  inline void set_name(const char* value);
  // mutable_xxx(),返回一个直接指向对象对应属性的指针,可以通过这个指针改变属性值
  inline ::std::string* mutable_name();

  // id,int型的只有4个基本方法
  inline bool has_id() const;
  inline void clear_id();
  inline int32_t id() const;
  inline void set_id(int32_t value);

  // email,同name
  inline bool has_email() const;
  inline void clear_email();
  inline const ::std::string& email() const;
  inline void set_email(const ::std::string& value);
  inline void set_email(const char* value);
  inline ::std::string* mutable_email();

  // phones,phones是一个repeated的数据项,所有会有一些特有的方法
  // xxx_size(),数据条目数
  inline int phones_size() const;
  inline void clear_phones();
  inline const ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >& phones() const;
  inline ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >* mutable_phones();
  // xxx(int index)和mutable_xxx(),通过index去访问某一个repeated的数据项
  inline const ::tutorial::Person_PhoneNumber& phones(int index) const;
  inline ::tutorial::Person_PhoneNumber* mutable_phones(int index);
  // add_xxx(),添加一条数据
  inline ::tutorial::Person_PhoneNumber* add_phones();
```
### message接口
每个message生成类中会包含以下接口:

``` c++
// 判断是否所有required的数据项都赋值过了
bool IsInitialized() const;
// 返回一个包含各个属性的可读字符串,用于调试
string DebugString() const;
// 用另一个message类给自己赋值
void CopyFrom(const Person& from);
// 清空所有值
void Clear();
```

### 序列化接口
``` c++
// 序列化至output字符串
bool SerializeToString(string* output) const;
// 使用字符串给自己赋值
bool ParseFromString(const string& data);
// 序列化至输出流
bool SerializeToOstream(ostream* output) const;: writes the message to the given C++ ostream.
// 使用输入流给自己赋值
bool ParseFromIstream(istream* input);: parses a message from the given C++ istream.
```

***

以上接口是一些常用接口，但并不是全部接口。全部接口的介绍可以查看[message API文档](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.message#Message)

## 2.通过接口去读写对象
以下是两段样例代码：

### 写

``` c++
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// This function fills in a Person message based on user input.
void PromptForAddress(tutorial::Person* person) {
  cout << "Enter person ID number: ";
  int id;
  cin >> id;
  person->set_id(id);
  cin.ignore(256, '\n');

  cout << "Enter name: ";
  getline(cin, *person->mutable_name());

  cout << "Enter email address (blank for none): ";
  string email;
  getline(cin, email);
  if (!email.empty()) {
    person->set_email(email);
  }

  while (true) {
    cout << "Enter a phone number (or leave blank to finish): ";
    string number;
    getline(cin, number);
    if (number.empty()) {
      break;
    }

    tutorial::Person::PhoneNumber* phone_number = person->add_phones();
    phone_number->set_number(number);

    cout << "Is this a mobile, home, or work phone? ";
    string type;
    getline(cin, type);
    if (type == "mobile") {
      phone_number->set_type(tutorial::Person::MOBILE);
    } else if (type == "home") {
      phone_number->set_type(tutorial::Person::HOME);
    } else if (type == "work") {
      phone_number->set_type(tutorial::Person::WORK);
    } else {
      cout << "Unknown phone type.  Using default." << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file,
//   adds one person based on user input, then writes it back out to the same
//   file.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!input) {
      cout << argv[1] << ": File not found.  Creating a new file." << endl;
    } else if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  // Add an address.
  PromptForAddress(address_book.add_people());

  {
    // Write the new address book back to disk.
    fstream output(argv[1], ios::out | ios::trunc | ios::binary);
    if (!address_book.SerializeToOstream(&output)) {
      cerr << "Failed to write address book." << endl;
      return -1;
    }
  }

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```

### 读

``` c++
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// Iterates though all people in the AddressBook and prints info about them.
void ListPeople(const tutorial::AddressBook& address_book) {
  for (int i = 0; i < address_book.people_size(); i++) {
    const tutorial::Person& person = address_book.people(i);

    cout << "Person ID: " << person.id() << endl;
    cout << "  Name: " << person.name() << endl;
    if (person.has_email()) {
      cout << "  E-mail address: " << person.email() << endl;
    }

    for (int j = 0; j < person.phones_size(); j++) {
      const tutorial::Person::PhoneNumber& phone_number = person.phones(j);

      switch (phone_number.type()) {
        case tutorial::Person::MOBILE:
          cout << "  Mobile phone #: ";
          break;
        case tutorial::Person::HOME:
          cout << "  Home phone #: ";
          break;
        case tutorial::Person::WORK:
          cout << "  Work phone #: ";
          break;
      }
      cout << phone_number.number() << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file and prints all
//   the information inside.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  ListPeople(address_book);

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```
