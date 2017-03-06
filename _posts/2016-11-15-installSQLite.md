---
layout: post
title:  "Install SQLite"
date:   2016-11-15 06:11:47 +0800
categories: sqlite
---

## `Windows`

访问[SQLite下载页面](http://www.sqlite.org/download.html)
下载`sqlite-dll-win32-*.zip`和`sqlite-tools-win32-*.zip`

### 安装

解压`sqlite-tools-win32-*.zip`，得到其中的`sqlite3.exe`文件
保存`sqlite3.exe`到一个你喜欢的文件夹，如：`C:\sqlite`
然后将`C:\sqlite`添加到`PATH`环境变量中

命令行中
```Bash
$ sqlite3 --version
3.15.1 2016-11-04 12:08:49 1136863c76576110e710dd5d69ab6bf347c65e36
```

> 之前`sqlite3.exe`文件是被放在`sqlite-shell-win32-*.zip`中的

### 动态链接库(dll)

解压`sqlite-dll-win32-*.zip`，得到`sqlite3.dll`和`sqlite3.def`
这里提供的SQLite DLL是线程安全的，也就是说，它是定义了THREADSAFE预处理符后编译的。
这样就可以在多线程中使用SQLite，也可以在多线程中使用DLL，安全地在多线程中执行同步操作
该DLL需要与程序`sqlite3.exe`处于同一文件夹下或者在系统的路径中`C:\Windows\System32`，
或者遵循WINDOWS动态链接库加载时寻找DLL的规则

### 问题

以上安装可是使`SQLite`中场在`cmd`和`powershell`中运行
但是在`babun`中无法正常工作。`--help`和`--version`可以执行，`sqlite3`一直卡死

使用`g++ sqlite101.cpp -l sqlite3`编译无法关联到相关的库文件

参考
[Using SQLite3 with Cygwin](http://superuser.com/questions/253059/using-sqlite3-with-cygwin)
> Cygwin uses Windows pipes to emulate ptys,
so native console program see a pipe where they expect to see a console.
Among other issues, that often causes them to enter non-interactive mode.
> Cygwin currently doesn't work well with interactive native programs

解决
* 使用`-interactive`可以运行`sqlite3`进行交互
* 通过`Cygwin`的`setup.exe`安装`sqlite`组件，成功使用`g++ sqlite101.cpp -l sqlite3`编译

### 使用VS编译

[Using SQLite in a Native Visual C++ Application](https://dcravey.wordpress.com/2011/03/21/using-sqlite-in-a-visual-c-application/)

下载`sqlite-amalgamation-*.zip`文件，其中包含`sqlite3.h`, `sqlite3.c`等文件,将这些文件放入你的项目中即可

```c++
#include "stdafx.h"
#include <ios>
#include <iostream>
#include "sqlite3.h"

using namespace std;

int _tmain(int argc, _TCHAR* argv[])
{
   int rc;
   char *error;

   // Open Database
   cout << "Opening MyDb.db ..." << endl;
   sqlite3 *db;
   rc = sqlite3_open("MyDb.db", &db);
   if (rc)
   {
      cerr << "Error opening SQLite3 database: "
           << sqlite3_errmsg(db) << endl << endl;
      sqlite3_close(db);
      return 1;
   }
   else
   {
      cout << "Opened MyDb.db." << endl << endl;
   }

   // Execute SQL
   cout << "Creating MyTable ..." << endl;
   const char *sqlCreateTable = "CREATE TABLE MyTable \
        (id INTEGER PRIMARY KEY, value STRING);";
   rc = sqlite3_exec(db, sqlCreateTable, NULL, NULL, &error);
   if (rc)
   {
      cerr << "Error executing SQLite3 statement: "
           << sqlite3_errmsg(db) << endl << endl;
      sqlite3_free(error);
   }
   else
   {
      cout << "Created MyTable." << endl << endl;
   }

   // Execute SQL
   cout << "Inserting a value into MyTable ..." << endl;
   const char *sqlInsert = "INSERT INTO MyTable VALUES(NULL, 'A Value');";
   rc = sqlite3_exec(db, sqlInsert, NULL, NULL, &error);
   if (rc)
   {
      cerr << "Error executing SQLite3 statement: "
           << sqlite3_errmsg(db) << endl << endl;
      sqlite3_free(error);
   }
   else
   {
      cout << "Inserted a value into MyTable." << endl << endl;
   }

   // Display MyTable
   cout << "Retrieving values in MyTable ..." << endl;
   const char *sqlSelect = "SELECT * FROM MyTable;";
   char **results = NULL;
   int rows, columns;
   sqlite3_get_table(db, sqlSelect, &results, &rows, &columns, &error);
   if (rc)
   {
      cerr << "Error executing SQLite3 query: "
           << sqlite3_errmsg(db) << endl << endl;
      sqlite3_free(error);
   }
   else
   {
      // Display Table
      for (int rowCtr = 0; rowCtr <= rows; ++rowCtr)
      {
         for (int colCtr = 0; colCtr < columns; ++colCtr)
         {
            // Determine Cell Position
            int cellPosition = (rowCtr * columns) + colCtr;

            // Display Cell Value
            cout.width(12);
            cout.setf(ios::left);
            cout << results[cellPosition] << " ";
         }

         // End Line
         cout << endl;

         // Display Separator For Header
         if (0 == rowCtr)
         {
            for (int colCtr = 0; colCtr < columns; ++colCtr)
            {
               cout.width(12);
               cout.setf(ios::left);
               cout << "~~~~~~~~~~~~ ";
            }
            cout << endl;
         }
      }
   }
   sqlite3_free_table(results);

   // Close Database
   cout << "Closing MyDb.db ..." << endl;
   sqlite3_close(db);
   cout << "Closed MyDb.db" << endl << endl;

   // Wait For User To Close Program
   cout << "Please press any key to exit the program ..." << endl;
   cin.get();

   return 0;
}
```

`c++`操作接口，其使用可以参考[这篇文章](http://www.runoob.com/sqlite/sqlite-c-cpp.html)  
详细文档可以查阅[官方文档](https://www.sqlite.org/cintro.html)

| API  | 描述  |
| :--- | :--- |
| `sqlite3_open(const char *filename, sqlite3 **ppDb)` | 该例程打开一个指向 SQLite 数据库文件的连接，返回一个用于其他 SQLite 程序的数据库连接对象。如果 filename 参数是 NULL 或 ':memory:'，那么 sqlite3_open() 将会在 RAM 中创建一个内存数据库，这只会在 session 的有效时间内持续。如果文件名 filename 不为 NULL，那么 sqlite3_open() 将使用这个参数值尝试打开数据库文件。如果该名称的文件不存在，sqlite3_open() 将创建一个新的命名为该名称的数据库文件并打开。 |
| `sqlite3_exec(sqlite3*, const char *sql, sqlite_callback, void *data, char **errmsg)` | 该例程提供了一个执行 SQL 命令的快捷方式，SQL 命令由 sql 参数提供，可以由多个 SQL 命令组成。在这里，第一个参数 sqlite3 是打开的数据库对象，sqlite_callback 是一个回调，data 作为其第一个参数，errmsg 将被返回用来获取程序生成的任何错误。sqlite3_exec() 程序解析并执行由 sql 参数所给的每个命令，直到字符串结束或者遇到错误为止。 |
| `sqlite3_close(sqlite3*)` | 该例程关闭之前调用 sqlite3_open() 打开的数据库连接。所有与连接相关的语句都应在连接关闭之前完成。如果还有查询没有完成，sqlite3_close() 将返回 SQLITE_BUSY 禁止关闭的错误消息。|
