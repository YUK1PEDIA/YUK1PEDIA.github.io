---
layout: post
title: STL容器——string成员函数
description: 记录
tag: 算法
---

转载，原文链接：https://www.cnblogs.com/lynx-peng/p/16552710.html



## 1.构造与析构

**构造**

```c++
string()//构造空字符串
string(const char* s);//拷贝s所指向的字符串序列
string(const char* s, size_t n);//拷贝s所指向的字符串序列的第n个到结尾的字符
string(size_t n, char c);//将字符c复制n次
string(const string& str);//拷贝构造函数
string(const string& str, size_t pos, size_t len = npos);//拷贝s中从pos位置起的len个字符，若npos>字符串长度，就拷贝到字符串结尾结束
```



**析构**

```c++
~string();//删除字符串
```





## 2.迭代器

```c++
/*迭代器*/
iterator begin(); //返回指向字符串第一个字符的迭代器
iterator end(); //返回指向字符串最后一个字符的下一个位置的迭代器
reverse_iterator rbegin(); //返回字符串最后一个字符的反向迭代器
reverse_iterator rend(); //返回指向字符串第一个字符之前的反向迭代器
/*常量迭代器*/
iterator cbegin(); //返回指向字符串第一个字符的迭代器
iterator cend(); //返回指向字符串最后一个字符的下一个位置的迭代器
reverse_iterator rcbegin(); //返回字符串最后一个字符的反向迭代器
reverse_iterator rcend(); //返回指向字符串第一个字符之前的反向迭代器
```

注：迭代器（非常量迭代器）和常量迭代器的区别如下：

- 非常量迭代器允许修改迭代器所指向的元素，这类迭代器通常用于**需要修改容器中元素**的情况
- 常量迭代器不允许修改迭代器所指向的元素，这类迭代器通常用于**只需要读取容器中元素**的情况





## 3.访问

```c++
reference at(size_type pos);//同char& operator[]，返回pos位置字符的引用，字符串可读可写
char& back();//返回最后字符的引用
char& front();//返回第一个字符的引用
```





## 4.长度及容量

```c++
size_t size();//返回字符串字符个数
size_t length();//返回字符串字符个数
size_t max_size();//返回string对象中可存放的最大字符串的长度
size_t capacity();//返回string分配的存储容量。
void resize (size_t n);//调整源字符串的长度为n。
void resize (size_t n, char c);//调整字符串长度为n，并用字符c填充不足的部分
void reserve (size_t n = 0);//重新给源字符串分配存储最小为n的容量
void shrink_to_fit()//清理内存，使字符串的容量变得等于字符串的大小
void clear();//将字符串的内容清空，让源字符串成为一个空字符串（长度为0个字符）
bool empty();//判断字符串是否为空
```





## 5.修饰

### 5.1.append() 在结尾添加字符串

```c++
string & append(const string & str)//在结尾添加一个string字符串
string & append(const string & str, size_type subpos, size_type sublen)//追加str中从subpos开始的sublen个字符(子串)
string & append(const charT * s)//C语言字符串
string & append(const charT * s, size_type n)//C语言字符串(长度为n的子串)
string & append(size_type n, charT c)//n个字符c
string & append(InputIterator first, InputIterator last)//使用迭代器append
```



### 5.2.push_back() 将字符串添加到串尾

```c++
void push_back (charT c);//将字符C添加到结尾
```



### 5.3.assign() 赋值

```c++
string &assign(const char *s);///char*类型的字符串赋给当前字符串
string &assign(const char *s,int n);//从给定的 C 风格字符串 s 的前 n 个字符创建一个新的字符串，并将当前字符串设置为这个新字符串的内容
string &assign(const string &s);//把字符串s赋给当前字符串
string &assign(int n,char c);//用n个字符c赋值给当前字符串
string &assign(const string &s,int start,int n);//把字符串s中从start开始的n个字符赋给当前字符串
string &assign(const_iterator first,const_itertor last);//把first和last迭代器之间的部分赋给字符串
```



### 5.4.insert() 在串中间插入

```c++
string & insert(size_type pos, const string & str)//在pos位置插入字符串str
string & insert(size_type pos, const string & str,size_type subpos, size_type sublen)//从subpos开始的sublen的子串
string & insert(size_type pos, const charT * s)//C语言字符串
string & insert(size_type pos, const charT * s, size_type n)//C语言字符串(长度为n的子串)
string & insert(size_type pos,   size_type n, charT c)//n个字符c
iterator insert(const_iterator p, size_type n, charT c)//使用迭代器索引插入n和字符
iterator insert(const_iterator p, charT c)//单一字符
iterator insert(iterator p, InputIterator first, InputIterator last)//使用迭代器insert
```



### 5.5.erase() 删除字符串中的特定字符

```c++
string & erase(size_type pos=0, size_type len=npos)//从pos处删除len长度的字符
iterator erase(const_iterator p)//删除迭代器所指的单一字符
iterator erase(const_iterator first, const_iterator last)//删除2迭代器中间的字符
```



### 5.6.replace() 替换字符串的一部分

```c++
string & replace(size_type pos,size_type len,const string & str)//从pos位置开始，长度为len的字符替换为str
string & replace(const_iterator i1, const_iterator i2, const string & str)//替换两迭代器之间的字符
string & replace(size_type pos, size_type len, const string & str,size_type subpos, size_type sublen)//使用子串替换
string & replace(size_type pos,     size_type len,     const charT * s)//使用C语言字符串
string & replace(const_iterator i1, const_iterator i2, const charT * s)//迭代器方法
string & replace(size_type pos,     size_type len,     const charT * s, size_type n)//使用子串
string & replace(const_iterator i1, const_iterator i2, const charT * s, size_type n)//迭代器方法
string & replace(size_type pos,     size_type len,     size_type n, charT c)//用n个字符c替换
string & replace(const_iterator i1, const_iterator i2, size_type n, charT c)//迭代器方法
string & replace(const_iterator i1, const_iterator i2,
                       InputIterator first, InputIterator last)//迭代器方法
```



### 5.7.swap() 交换两字符串

```c++
void swap (& str);//交换self和str
```



### 5.8.pop_back() 删除最后一个字符

```c++
void pop_back();//删除串中最后一个字符
```





## 6.操作

### 6.1.find() 查找

```c++
size_type find(const string & str, size_type pos=0) const;//从母字符串的pos位置查找字串str.存在返回字串第一个字符的位置，不存在返回npos
size_type find(const charT * s, size_type pos=0) const;//C语言字符串作为子串
size_type find(const charT * s, size_type pos, size_type n) const;//C语言字符串的子串(长度为n)作为被查找子串
size_type find(charT c, size_type pos=0) const;//查找单个字符
//倒着找
size_type rfind(const string & str, size_type pos=npos) const;
size_type rfind(const charT * s, size_type pos=npos);
size_type rfind(const charT * s, size_type pos, size_type n);
size_type rfind(charT c, size_type pos=npos) const;
//查找字符串中与目标字符(串中任一个字符)相同的第一个字符
size_type find_first_of(const string & str, size_type pos=0) const;
size_type find_first_of(const charT * s, size_type pos=0) const;
size_type find_first_of(const charT * s, size_type pos, size_type n);
size_type find_first_of(charT c, size_type pos=0) const;
//倒着找 or 找最后一个
size_type find_last_of(const string & str, size_type pos=npos) const 
size_type find_last_of(const charT * s, size_type pos=npos) const
size_type find_last_of(const charT * s, size_type pos, size_type n) const
size_type find_last_of(charT c, size_type pos=npos) const
//查找字符串中与目标字符(串中任一个字符)不相同的第一个字符
size_type find_first_not_of(const string & str, size_type pos=0) const;
size_type find_first_not_of(const charT * s, size_type pos=0) const;
size_type find_first_not_of(const charT * s, size_type pos, size_type n) const;
size_type find_first_not_of(charT c, size_type pos=0) const;
//倒着找 or 找最后一个
size_type find_last_not_of(const string & str, size_type pos=npos) const ;
size_type find_last_not_of(const charT * s, size_type pos=npos) const;
size_type find_last_not_of(const charT * s, size_type pos, size_type n) const;
size_type find_last_not_of(charT c, size_type pos=npos) const ;
```



### 6.2.substr() 子串

```c++
string substr (size_type pos = 0, size_type len = npos) const;//返回一个从pos开始，len长度的string类型的子串
```



### 6.3.compare() 比较

```c++
int compare (const string& str) const ;//比较字符串大小，源串大于str返回值>0，相同=0,小于<0
int compare (size_type pos, size_type len, const string& str) const;//用自身的子串比较
int compare (size_type pos, size_type len, const string& str,
             size_type subpos, size_type sublen) const;//两字符串均为子串
int compare (const charT* s) const;//C语言字符串
int compare (size_type pos, size_type len, const charT* s) const;//C语言字符串子串
int compare (size_type pos, size_type len, const charT* s, size_type n) const;//双子串
```



### 6.4.copy() 复制到字符数组

```c++
size_type copy (charT* s, size_type len, size_type pos = 0) const;//从string类型对象中至多复制n个字符到字符指针p指向的空间中。不添加'\0'
```





## 7.符号重载

### 7.1.= 赋值

```c++
string &operator=(const string &s);//把字符串s赋给当前字符串
string &operator=(const char* s);//char*类型的字符串赋给当前字符串
string &operator=(char c);//单个字符赋给当前字符串
```



### 7.2.[] 访问

```c++
char& operator[] (size_t pos);//返回pos处字符的引用 越界导致未定义行为
```



### 7.3.+= 追加

```c++
string& operator+= (const string& str);//在结尾加如str字符串，等效于append
string& operator+= (const char* s);//C语言字符串
string& operator+= (char c);//单个字符
```





## 8.补充：cctype 字符处理库

C++从C语言继承了一个与字符相关的、非常方便的函数软件包，它可以简化诸如确定字符是否为大写字母、数字、标点符号等工作，这些函数的原型在头文件cctype中定义，使用 `#include <cctype>` 引入

```c++
isalnum()  // 如果参数是字母数字，即字母或者数字，函数返回true
isalpha()  // 如果参数是字母，函数返回true
iscntrl()  // 如果参数是控制字符，函数返回true
isdigit()  // 如果参数是数字（0－9），函数返回true
isgraph()  // 如果参数是除空格之外的打印字符，函数返回true
islower()  // 如果参数是小写字母，函数返回true
isprint()  // 如果参数是打印字符（包括空格），函数返回true
ispunct()  // 如果参数是标点符号，函数返回true
isspace()  // 如果参数是标准空白字符，如空格、换行符、水平或垂直制表符，函数返回true
isupper()  // 如果参数是大写字母，函数返回true
isxdigit() // 如果参数是十六进制数字，即0－9、a－f、A－F，函数返回true
tolower()  // 如果参数是大写字符，返回其小写，否则返回该参数
toupper()  // 如果参数是小写字符，返回其大写，否则返回该参数
```

