# Lab05
[![Coverage Status](https://coveralls.io/repos/github/HECCYLLIujTbmy/lab05-to4no/badge.svg?branch=master)](https://coveralls.io/github/HECCYLLIujTbmy/lab05-to4no?branch=master)
### Задание
1. Создайте `CMakeList.txt` для библиотеки *banking*.
2. Создайте модульные тесты на классы `Transaction` и `Account`.
    * Используйте mock-объекты.
    * Покрытие кода должно составлять 100%.
3. Настройте сборочную процедуру на **TravisCI**.
4. Настройте [Coveralls.io](https://coveralls.io/).

```sh
┌──(kali㉿kali)-[~]
└─$ cd /home/kali/HECCYLLIujTbmy/workspace/projects/              
                                                                                        
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects]
└─$ git clone https://github.com/tp-labs/lab05 lab05-to4no
Cloning into 'lab05-to4no'...
remote: Enumerating objects: 137, done.
remote: Counting objects: 100% (25/25), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 137 (delta 18), reused 16 (delta 16), pack-reused 112
Receiving objects: 100% (137/137), 918.92 KiB | 2.84 MiB/s, done.
Resolving deltas: 100% (60/60), done.
 ```                                                                                       
```sh
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects]
└─$ cd lab05-to4no                                  
 ```                                                                                                                                
 ```sh                                                                                       
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects/lab05-to4no]
└─$ git init                
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint:   git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint:   git branch -m <name>
Initialized empty Git repository in /home/kali/HECCYLLIujTbmy/workspace/projects/lab05-to4no/.git/
                                                                                        
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects/lab05-to4no]
└─$ git remote add origin https://github.com/HECCYLLIujTbmy/lab05-to4no
                                                                                        
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects/lab05-to4no]
└─$ git submodule add  https://github.com/google/googletest.git
Cloning into '/home/kali/HECCYLLIujTbmy/workspace/projects/lab05-to4no/googletest'...
remote: Enumerating objects: 26510, done.
remote: Counting objects: 100% (87/87), done.
remote: Compressing objects: 100% (64/64), done.
remote: Total 26510 (delta 42), reused 49 (delta 22), pack-reused 26423
Receiving objects: 100% (26510/26510), 12.32 MiB | 4.46 MiB/s, done.
Resolving deltas: 100% (19677/19677), done.
```
```sh
┌──(kali㉿kali)-[~/…/workspace/projects/lab05-to4no/banking]
└─$ nano CMakeLists.txt             
cmake_minimum_required(VERSION 3.4)
project(bank_lib)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_library(banking STATIC Account.cpp Account.h Transaction.cpp Transaction.h)
```
```sh
(kali㉿kali)-[~/…/workspace/projects/lab05-to4no/banking]
└─$ cd ..     
                                                                                        
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects/lab05-to4no]
└─$ mkdir .github && cd .github && mkdir workflows && cd workflows
```
```sh                                                                                        
┌──(kali㉿kali)-[~/…/projects/lab05-to4no/.github/workflows]
```

<details><summary>$ nano cmake.yml</summary>

```sh
name: CMake
on:
 push:
  branches: [master]
 pull_request:
  branches: [master]
jobs:
 build_Linux:
  runs-on: ubuntu-latest
  steps:
  - uses: actions/checkout@v3
  - name: Adding gtest
    run: git clone https://github.com/google/googletest.git third-party/gtest -b release-1.11.0
  - name: Install lcov
    run: sudo apt-get install -y lcov
  - name: Config banking with tests
    run: cmake -H. -B ${{github.workspace}}/build -DBUILD_TESTS=ON
  - name: Build banking
    run: cmake --build ${{github.workspace}}/build
  - name: Run tests
    run: build/check
  - name: Do lcov stuff
    run: lcov -c -d build/CMakeFiles/banking.dir/banking/ --include *.cpp --output-file ./coverage/lcov.info
  - name: Publish to coveralls.io
    uses: coverallsapp/github-action@v1.1.2
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

</details>

```sh
┌──(kali㉿kali)-[~/…/projects/lab05-to4no/.github/workflows]
└─$ cd ../..
                                                                                        
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects/lab05-to4no]
└─$ mkdir tests && cd tests                                       
```

```sh                                                                                        
┌──(kali㉿kali)-[~/…/workspace/projects/lab05-to4no/tests]
```
<details><summary>$ nano test_Account.cpp</summary>

```sh
#include "Account.h"
#include "Transaction.h"

#include <gtest/gtest.h>
#include <gmock/gmock.h>

class AccountMock : public Account {
public:
    AccountMock(int id, int balance) : Account(id, balance) {}
    MOCK_CONST_METHOD0(GetBalance, int());
    MOCK_METHOD1(ChangeBalance, void(int diff)); 
    MOCK_METHOD0(Lock, void());
    MOCK_METHOD0(Unlock, void());
};

class TransactionMock : public Transaction {
public:
    MOCK_METHOD3(Make, bool(Account& from, Account& to, int sum));
};

TEST(Account, Mock) {
    AccountMock acc(1, 666);
    EXPECT_CALL(acc, GetBalance()).Times(1);
    EXPECT_CALL(acc, ChangeBalance(testing::_)).Times(2);
    EXPECT_CALL(acc, Lock()).Times(2);
    EXPECT_CALL(acc, Unlock()).Times(1);
    acc.GetBalance();
    acc.ChangeBalance(100); 
    acc.Lock();
    acc.ChangeBalance(100);
    acc.Lock(); 
    acc.Unlock();
}

TEST(Account, SimpleTest) {
    Account acc(1, 666);
    EXPECT_EQ(acc.id(), 1);
    EXPECT_EQ(acc.GetBalance(), 666);
    EXPECT_THROW(acc.ChangeBalance(200), std::runtime_error);
    EXPECT_NO_THROW(acc.Lock());
    acc.ChangeBalance(200);
    EXPECT_EQ(acc.GetBalance(), 866);
    EXPECT_THROW(acc.Lock(), std::runtime_error);
    EXPECT_NO_THROW(acc.Unlock());
}

TEST(Transaction, Mock) {
    TransactionMock tr;
    Account ac1(1, 100);
    Account ac2(2, 300);
    EXPECT_CALL(tr, Make(testing::_, testing::_, testing::_))
    .Times(5);
    tr.set_fee(200);
    tr.Make(ac1, ac2, 200);
    tr.Make(ac2, ac1, 300);
    tr.Make(ac1, ac1, 0); 
    tr.Make(ac1, ac2, -5); 
    tr.Make(ac2, ac1, 50); 
}

TEST(Transaction, SimpleTest) {
    Transaction tr;
    Account ac1(1, 100);
    Account ac2(2, 300);
    tr.set_fee(10);
    EXPECT_EQ(tr.fee(), 10);
    EXPECT_THROW(tr.Make(ac1, ac2, 40), std::logic_error);
    EXPECT_THROW(tr.Make(ac1, ac2, -5), std::invalid_argument);
    EXPECT_THROW(tr.Make(ac1, ac1, 100), std::logic_error);
    EXPECT_FALSE(tr.Make(ac1, ac2, 400));
    EXPECT_FALSE(tr.Make(ac2, ac1, 300));
    EXPECT_FALSE(tr.Make(ac2, ac1, 290));
    EXPECT_TRUE(tr.Make(ac2, ac1, 150));
}

```

<details>

```sh
┌──(kali㉿kali)-[~/…/workspace/projects/lab05-to4no/tests]
```
<details><summary>$ nano test_Transaction.cpp</summary>

```sh
#include <Account.h>
#include <Transaction.h>
#include <gtest/gtest.h>

TEST(Transaction, Banking){
	const int base_A = 5000, base_B = 5000, base_fee = 100;
	Account Alice(0,base_A), Bob(1,base_B);
	Transaction test_tran;
	ASSERT_EQ(test_tran.fee(), 1);
	test_tran.set_fee(base_fee);
	ASSERT_EQ(test_tran.fee(), base_fee);
	ASSERT_THROW(test_tran.Make(Alice, Alice, 1000), std::logic_error);
	ASSERT_THROW(test_tran.Make(Alice, Bob, -50), std::invalid_argument);
	ASSERT_THROW(test_tran.Make(Alice, Bob, 50), std::logic_error);
	if (test_tran.fee()*2-1 >= 100)
		ASSERT_EQ(test_tran.Make(Alice, Bob, test_tran.fee()*2-1), false);
	Alice.Lock();
	ASSERT_THROW(test_tran.Make(Alice, Bob, 1000), std::runtime_error);
	Alice.Unlock();
	ASSERT_EQ(test_tran.Make(Alice, Bob, 1000), true);
	ASSERT_EQ(Bob.GetBalance(), base_B+1000);
	ASSERT_EQ(Alice.GetBalance(), base_A-1000-base_fee);
	ASSERT_EQ(test_tran.Make(Alice, Bob, 3900), false);
	ASSERT_EQ(Bob.GetBalance(), base_B+1000);
	ASSERT_EQ(Alice.GetBalance(), base_A-1000-base_fee);
}
```

</details>

```sh
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects/lab05-to4no]
└─$ mkdir coverage && cd coverage
                                                                                        
┌──(kali㉿kali)-[~/…/workspace/projects/lab05-to4no/coverage]
└─$ touch temp.txt
                                                                                        
┌──(kali㉿kali)-[~/…/workspace/projects/lab05-to4no/coverage]
└─$ cd ..         
                                                                                        
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects/lab05-to4no]
└─$ git add -A
                                                                                        
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects/lab05-to4no]
└─$ git commit -m"push"                                   
[master (root-commit) 56f092c] push
 15 files changed, 631 insertions(+)
 create mode 100644 .github/workflows/cmake.yml
 create mode 100644 .gitmodules
 create mode 100644 LICENSE
 create mode 100644 README.md
 create mode 100644 banking/Account.cpp
 create mode 100644 banking/Account.h
 create mode 100644 banking/CMakeList.txt
 create mode 100644 banking/CMakeLists.txt
 create mode 100644 banking/Transaction.cpp
 create mode 100644 banking/Transaction.h
 create mode 100644 coverage/temp.txt
 create mode 160000 googletest
 create mode 100644 preview.png
 create mode 100644 tests/test_Account.cpp
 create mode 100644 tests/test_Transaction.cpp
                                                                                        
┌──(kali㉿kali)-[~/HECCYLLIujTbmy/workspace/projects/lab05-to4no]
└─$ git push origin master
Username for 'https://github.com': HECCYLLIujTbmy
Password for 'https://HECCYLLIujTbmy@github.com': 
Enumerating objects: 21, done.
Counting objects: 100% (21/21), done.
Delta compression using up to 2 threads
Compressing objects: 100% (16/16), done.
Writing objects: 100% (21/21), 889.16 KiB | 23.40 MiB/s, done.
Total 21 (delta 0), reused 0 (delta 0), pack-reused 0
remote: 
remote: Create a pull request for 'master' on GitHub by visiting:
remote:      https://github.com/HECCYLLIujTbmy/lab05-to4no/pull/new/master
remote: 
To https://github.com/HECCYLLIujTbmy/lab05-to4no
 * [new branch]      master -> master
 ```

## Links

- [C++ CI: Travis, CMake, GTest, Coveralls & Appveyor](http://david-grs.github.io/cpp-clang-travis-cmake-gtest-coveralls-appveyor/)
- [Boost.Tests](http://www.boost.org/doc/libs/1_63_0/libs/test/doc/html/)
- [Catch](https://github.com/catchorg/Catch2)

```
Copyright (c) 2015-2021 The ISC Authors
```

