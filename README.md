# SAD-Mesh

*一个mesh应用的开发环境*

[github地址](https://github.com/xlcbingo1999/SAD-Mesh/tree/master)

## 项目参与人员

| 成员   | 学号     |
| ------ | -------- |
| 肖霖畅 | 18342105 |
| 秦裕胜 | 18342084 |
| 唐乾力 | 18342089 |
|柴百里|   18341004|

## SAD-Mesh 是什么

这是一个为科研工作者与学生群体提供的，帮助开发轻量级mesh应用辅助开发环境。

在这个环境里，我们提供了解析mesh文件和以mesh文件为基础进行仿真实验所必要的接口。妥善地利用这些接口，就能轻松地开发出轻量级地mesh应用，进行类似热传导的物理仿真实验。

## SAD-Mesh 的安装

直接将所有代码文件下载下来即可

由于我们直接用`C++`从底层开始实现的，实现过程中调用的也都是基本库中的函数，因此可以在几乎所有支持C语言的系统上使用。

## SAD-Mesh 的特点

1. 使用简单
2. 能在大多数平台上运行
3. 能满足大部分的mesh仿真实验的需要

## SAD-Mesh 提供的接口

1. 读取`msh`文件（使用Gmsh的官方文档中规定的msh格式：http://gmsh.info//doc/texinfo/gmsh.html#MSH-ASCII-file-format）
2. 基于mesh构造支持仿真实验的数据结构（比如以多叉树结构将所有element及其相邻关系保存起来）
3. 为element附加参数（以数组的形势保存，使用element的id作为索引）
4. 定义计算公式以及执行计算的接口（在后文会详细说明）

## SAD-Mesh 如何使用

### Windows 版本

1. 安装MinGW-w64编译器【著名的C/C++编译器GCC的Windows版本】[MinGW-w64安装教程](https://www.jianshu.com/p/d66c2f2e3537)；
2. 下载本软件代码 [github地址](https://github.com/xlcbingo1999/SAD-Mesh/tree/master)
3. 进入项目目录，在控制台输入：

```shell
mingw32-make clean
mingw32-make
```

4. 进入bin文件夹，控制台输入：

```shell
./main
```

5. 开始执行。

### Linux 版本

1. ubuntu安装gcc和g++编译器[ubuntu 安装 GCC 和 G++ C++ 开发环境](https://blog.csdn.net/ichsonx/article/details/14649291)；
2. 下载本软件代码 [github地址](https://github.com/xlcbingo1999/SAD-Mesh/tree/master)
3. 进入项目目录，在控制台输入：

```shell
make clean
make
```

4. 进入bin文件夹，控制台输入：

```shell
./main
```

5. 开始执行。

### 如何自定义计算公式并启动计算

想要使用这个工具来开发mesh应用你需要做的就是修改`Mesh.cpp`文件

调用我们提供的接口来实现能够满足你要求的kernel function。

一般而言，只需要修改main函数即可满足大部分我们能支持的需求

可参考如下热传导实验样例（`MeshTest.cpp`）：

```c++

#include "User.hpp"
/***********************************************************************
 * TO-DO: 请在本区域加入你想要加入的头文件，本程序不进行冲突确保
 * 
 */
#include <iostream>
#include <fstream>
using namespace std;
fstream infile2;
/***********************************************************************
 * TO-DO: 请在本区域自定义你的diyUser类
 * 
 */
class diyUser: public User {
private:
    /* data */
    bool outputToFile = false;
    bool odd = false;
public:
    diyUser();
    ~diyUser();
    meshHead* getMeshStructHead();
    size_t getMeshElementCount();

    ArguMap* initAllElementValue(double d);

    int add_coef(struct Coefficient * c);
    int add_attr(enum Attr a); 
    int set_form(Form f);

    double Execute(struct ArguMap** argus, int index); // virtual 函数，必须实现

    void displayResult(int ref, double ret); // virtual 函数，必须实现

    void beginSave(); // virtual 函数，必须实现
    void saveItem(double ret); // virtual 函数，必须实现
    void endSave(); // virtual 函数，必须实现

    void setOutputToFile();
    bool getOutputToFile();
};

diyUser::diyUser()
{
    
}

diyUser::~diyUser()
{

}

meshHead* diyUser::getMeshStructHead() {
    return &realMeshHead;
}

size_t diyUser::getMeshElementCount() {
    return getMeshStructHead()->elementsSet.numElements;
}

ArguMap* diyUser::initAllElementValue(double init) {
    return execManager.getNewArguMap(getMeshElementCount(), init);
}

int diyUser::add_coef(struct Coefficient * c) {
    return execManager.add_coef(c);
}

int diyUser::add_attr(enum Attr a) {
    return execManager.add_attr(a);
} 

int diyUser::set_form(Form f) {
    return execManager.set_form(f);
}

double diyUser::Execute(struct ArguMap** argus, int index) {
    return execManager.Execute(argus, index);
}

void diyUser::displayResult(int ref, double ret) {
    printf("%d %.2f\n",ref ,ret);
}

void diyUser::setOutputToFile() {
    outputToFile = true;
}

bool diyUser::getOutputToFile() {
    return outputToFile;
}

void diyUser::beginSave() {
    infile2.open("result.txt", ios::out);
}
    
void diyUser::saveItem(double ret) {
    infile2 << ret;
    if(odd)
        infile2 << "\n";
    else 
        infile2 << " ";
    odd = !odd;
}

    
void diyUser::endSave() {
    infile2.close();
}

/***********************************************************************
 * 以下主函数即是用户需要些的kernel_function
 * 以热传导仿真实验为例
 */
int main() {
    // -------------
    // 读入文件
    // -------------
    diyUser MAS;
    // 读入
    if(MAS.importApplication() == false) return -1;
    if(MAS.chooseDataStruct() == false) return -1;
    if(MAS.beginImporting() == false) return -1;
    // MAS.outputDataStruct(); // 输出
    MAS.diy();

/***********************************************************************
 * TO-DO: 请在本区域完成参数的定义和后台Mesh数据的拉取
 * 
 */
    meshHead *myMeshHead = MAS.getMeshStructHead();
    size_t elementCount = MAS.getMeshElementCount();

    printf("element count: %d\n", elementCount);

    struct ArguMap* maplist[1]; // 存放参数的结构体
    double init = 10;    // 设置初始热量
    maplist[0] = MAS.initAllElementValue(init);
    size_t origin = 1415; // 设置热源
    maplist[0]->value[origin] = 250;

/*********************************************************************
 * TO-DO: 请在本区域完成计算公式的设置
 * 示例：计算公式设置: x[n+1]= (x[n] + x1[n] + x2[n] + x3[n])/4
 * x[n]: n时刻编号为x的element的温度
 * x1,x2,x3为x的相邻节点，不一定全部存在
 */

    struct Coefficient* coef1 = new Coefficient[1];
    // 添加系数
    for (int i = 0; i < 4; i ++) coef1->value.push_back(1);
    MAS.add_coef(coef1);
    // 设置运算属性
    MAS.add_attr(ADDARGU);
    // 设置结果计算方法，即所有相邻原件的影响的平均值
    MAS.add_attr(END);
    MAS.set_form(AVG);

/*********************************************************************
 * TO-DO: 定义运算顺序(以后可以考虑独立出来成为我们系统提供的接口)
 * 
 */
    
/*********************************************************************
 * TO-DO: 请在这里使用Execute进行计算、显示和结果导出
 * 例子：计算出经过第一个时间周期后，温度产生变化的element变化后的温度
 */
    MAS.setOutputToFile();
    if(MAS.getOutputToFile() == true) {
        MAS.beginSave();
    }
    for (int i = 0; i < elementCount; i ++) {
        double res = MAS.Execute(maplist, i); 
        if (res != init) {
            MAS.displayResult(i+1,res);
        }
        MAS.saveItem(i+1);
        MAS.saveItem(res);
    }
    if(MAS.getOutputToFile() == true) {
        MAS.endSave();
    }
    // Execute(&express, maplist, myMeshHead, 0);

    return 0;
}
```

## SAD-Mesh 的细节

### 关于公式定义

此处由于还未提供UI支持，因此公式定义相对复杂。下面我将具体解释一下整个设计

首先，关于公式定义我们做了两个非常大的假设：

1. **所有需要使用到的公式都是多项式形式的**
2. **每个元素的状态都有其相邻元素的状态决定**

因此整个公式定义由四部分组成：

1. 参数 `ArguMap`
2. 系数 `Coefficient` 
3. 运算符：`Attr`（详细定义如下）
   * END 意味着达到末尾
   * WEI 使用一个参数结构，借此将所有相邻节点排序（可以用来改变参数的使用顺序）
   * ADDARGU 使用一个参数参数结构以及一个系数结构，将其相乘后加到原公式末尾
   * MULARGU 使用一个参数结构和一个，将其相乘后乘到原公式末尾
   * POW 使用一个系数结构，将其作为指数幂应用到原公式上
   * EXP 使用一个系数结构，将其作为e的指数幂乘到原公式末尾
   * LOG 使用一个系数结构，将系数作为对数的底数，对原公式求对数

```c++
enum Attr{END, WEI, ADDARGU, MULARGU, POW, EXP, LOG};
enum Form{SUM, AVG};
```

4. 结果计算方式`Form`：这主要是考虑到相邻element数目的不确定性
   * SUM 指全部求和
   * AVG 指求平均

#### 公式定义样例

目标公式：

假设一个element 周围有 `e1, e2, e3`三个相邻element，且该element的状态由如下表达式确定

参数的存储形形势为`{e1, e2, e3}`

$en = c1/(a1+b1) + c2/(a2+b2) +  c3/(a3+b3)$ 

```c++
argu: {
	{a1, a2, a3}
	{b1, b2, b3}
	{c1, c2, c3}
}
coef: {
	{1, 1, 1}
	{1, 1, 1} 
	{-1, -1, -1}
	{1, 1, 1}
}
Attr: [ADDARGU, ADDARGU, POW, MULARGU, END, LOG]
Form: SUM 
```

#### 程序使用样例

我们以上文的测试程序介绍一下如何设计一个公式

通过以下函数，你可以创建一个用于保存参数的数据结构，并进行需要的初始化赋值

```c++
struct ArguMap* maplist[1];
```

其后，创建一个用于保存数据结构

再然后，通过下列函数你可以实现添加运算符，定义公式如何使用

```c++
// 设置运算属性
MAS.add_attr(ADDARGU);
// 设置结果计算方法，即所有相邻原件的影响的平均值
MAS.add_attr(END);
MAS.set_form(AVG)
```

最后，以循环调用执行函数`Execute`的方式将公式应用到需要进行计算的公式上

```c++
for (int i = 0; i < elementCount; i ++) {
        double res = MAS.Execute(maplist, i); 
        if (res != init) {
            MAS.displayResult(i+1,res);
        }
        MAS.saveItem(i+1);
        MAS.saveItem(res);
    }
```

