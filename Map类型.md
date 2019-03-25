# MAP类
> ref:http://eigen.tuxfamily.org/dox/group__TutorialMapClass.html

本章介绍怎么使用C/C++中的数据。该技术适用于很多场景，特别是从其他库里面导入vector或matrices进入到Eigen。

## 介绍
有时候，想要预定义一个数组，希望把这个数组转化为Eigen里的vector或者matrix。一种方法是重新拷贝一份数据，通常我们更希望能够把原来的内存复用为一个Eigen的类型，Eigen里面的map类就能够轻易做到。

## Map类型与Map变量的声明
Map的对象等价于
`Map<Matrix<typename Scalar, int RowsAtCompileTime, int ColsAtCompileTime> >`

在默认情况下，Map只需要一个模板参数即可。
要创建一个Map的变量，需要另外两部分信息：一个指向定义数组的内存的指针，以及matrix或者vector的形状
例如，定义一个指定大小的float的matrix，操作如下

` Map<MatrixXf> mf(pf,rows,columns);`

pf是一个float*，指向数组的内存空间。
一个固定大小并且只读的int型的vector可以声明如下：

`Map<const Vector4i> mi(pi);`

pi是一个int*，在这种情况下，不需要在构造函数里面传入size，因为已经在Matrix/Array里面确定了。

Map类型没有默认构造函数，必须要传入一个指针去初始化。

Map类型很灵活，能够使用不同的数据表达方式。下面就有两种模板类型（可选）：

`Map<typename MatrixType,
   int MapOptions,
    typename StrideType>`
	
* MapOptions 确定指针是否是对齐的。默认是不对齐的。
* StrideType 可以通过Stride类型确定内存的布局，见下面的例子
```c++
int array[8];
for(int i = 0; i < 8; ++i) array[i] = i;
cout << "Column-major:\n" << Map<Matrix<int,2,4> >(array) << endl;
cout << "Row-major:\n" << Map<Matrix<int,2,4,RowMajor> >(array) << endl;
cout << "Row-major using stride:\n" <<
  Map<Matrix<int,2,4>, Unaligned, Stride<1,4> >(array) << endl;
```
输出如下
```
Column-major:
0 2 4 6
1 3 5 7
Row-major:
0 1 2 3
4 5 6 7
Row-major using stride:
0 1 2 3
4 5 6 7
```
当然Stride类的使用远比例子中所述的更灵活，具体的详见Stride类的说明

## 使用Map变量
你可以像其他Eigen类型一样去使用Map对象
```c++
typedef Matrix<float,1,Dynamic> MatrixType;
typedef Map<MatrixType> MapType;
typedef Map<const MatrixType> MapTypeConst;   // a read-only map
const int n_dims = 5;
  
MatrixType m1(n_dims), m2(n_dims);
m1.setRandom();
m2.setRandom();
float *p = &m2(0);  // get the address storing the data for m2
MapType m2map(p,m2.size());   // m2map shares data with m2
MapTypeConst m2mapconst(p,m2.size());  // a read-only accessor for m2
cout << "m1: " << m1 << endl;
cout << "m2: " << m2 << endl;
cout << "Squared euclidean distance: " << (m1-m2).squaredNorm() << endl;
cout << "Squared euclidean distance, using map: " <<
  (m1-m2map).squaredNorm() << endl;
m2map(3) = 7;   // this will change m2, since they share the same array
cout << "Updated m2: " << m2 << endl;
cout << "m2 coefficient 2, constant accessor: " << m2mapconst(2) << endl;
/* m2mapconst(2) = 5; */   // this yields a compile-time error
```
输出如下：
```
m1:   0.68 -0.211  0.566  0.597  0.823
m2: -0.605  -0.33  0.536 -0.444  0.108
Squared euclidean distance: 3.26
Squared euclidean distance, using map: 3.26
Updated m2: -0.605  -0.33  0.536      7  0.108
m2 coefficient 2, constant accessor: 0.536
```
所有的Eigen函数都能够接收Map对象的参数。当需要些一个参数为Eigen类型的函数时，会有“a Map type is not identical to its Dense equivalent”的问题。详见以Eigen类型作为参数的函数的章节。

## 改变数组
我们可以通过new在声明之后修改Map对象
```C++
int data[] = {1,2,3,4,5,6,7,8,9};
Map<RowVectorXi> v(data,4);
cout << "The mapped vector v is: " << v << "\n";
new (&v) Map<RowVectorXi>(data+4,5);
cout << "Now v is: " << v << "\n";
```
输出
```
The mapped vector v is: 1 2 3 4
Now v is: 5 6 7 8 9
```
上述操作不会有分配内存的操作，因为已经指定了内存位置。

我们可以在不知道数组内存空间的情况下去声明Map对象
```c++
Map<Matrix3f> A(NULL);  // don't try to use this matrix yet!
VectorXf b(n_matrices);
for (int i = 0; i < n_matrices; i++)
{
  new (&A) Map<Matrix3f>(get_matrix_pointer(i));
  b(i) = A.trace();
}
```
