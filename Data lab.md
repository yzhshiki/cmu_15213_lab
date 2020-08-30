### datalab

#### 1. bitXor

用到德摩根定律将非门、与门转为异或门

（遇到./dlc bits.c 提示dlc permission denied，用chmod指令修改权限即可（加上可执行））

```c
int bitXor(int x, int y) {
  int m = ~x & y;
  int n = x & ~y;
  return ~(~m & ~n);
}
```

#### 2. tmin

```c
int isTmax(int x) {
  return 2;
}
```

#### 3. isTmax

第四五行排除掉x=-1的情况。

```c
int isTmax(int x) {
  int i=x+1;
  x = x+i;
  x = ~x;
  i = !i;//如果输入x是Tmax，这里i从Tmin变成0，下一步不会影响x。
  x = x+i;//如果输入x是-1，上面的i从0变成1，使这个x从0到1。
  return !x;
}
```

#### 4. allOddBits

要求奇数位全为1，偶数位不作要求

``` c
int allOddBits(int x) {
  int m=0xAA;
  m = (m<<8)|m;
  m = (m<<16)|m;
  return !((x & m) ^ m);
}
```

#### 5. negate

```c
int negate(int x) {
  return ~x+1;
}
```

#### 6. isAsciiDigit

设置upperbound使得大于0x39的数加上之后会从Tmax到负数，设置lowerbound使得小于0x30的数加上之后小于-1。这两种情况的最高位都是1，所以要求就是在上下界的测试下最高位都是0。

```c
int isAsciiDigit(int x) {
  int sign = 0x1<<31;
  int upperbound = ~(sign|0x39);
  int lowerbound = ~0x30;
  int lessthan40 = !((upperbound+x)>>31);
  int morethan29 = !((lowerbound+x+1)>>31);

  return lessthan40 & morethan29;
}
```

#### 7. conditional

```c
int conditional(int x, int y, int z) {
  x=!!x;	//非零变成1
  x = ~x+1;	//0的补码全是0，1的相反数-1的补码全是一
  return (x&y)|(~x&z);
}
```

#### 8. isLessOrEqual

```c
int isLessOrEqual(int x, int y) {
  int topx = !!(x>>31);	//x的正负性
  int topy = !!(y>>31);	//y的正负性
  int negx = ~x + 1;
  int m = y + negx;
  m = m>>31;
  m = !!m;	//y-x的正负性
  return (!m & !(topy & !topx))|(m & (!topy & topx));
}
```

- m=0时，y>x或（y<x && y-x<Tmin）
- m=1时，(y>x && y-x>Tmax) 或y<x

这两种情况的后一半我们都要排除，排除方法就是同时限制x和y的符号位，见程序最后一行。

#### 9. logicalNeg（实现“!”）

``` c
int logicalNeg(int x) {
  return ((x|(~x+1))>>31)+0x1;
}
```

利用0的相反数还是0，其他数与相反数位或后符号位为1加以区分。

#### 10. howManyBits

一个数的补码表示法最少需要多少位

```c
int howManyBits(int x) {
  int b16,b8,b4,b2,b1,b0;
  int sign = x>>31;
  x = (~sign & x)|(sign & ~x);

  b16 = !!(x>>16)<<4;	//如果16个高位有非零，b16=16，否则b16=0
  x = x>>b16;
  b8 = !!(x>>8)<<3;
  x = x>>b8;
  b4 = !!(x>>4)<<2;
  x = x>>b4;
  b2 = !!(x>>2)<<1;
  x = x>>b2;
  b1 = !!(x>>1);	//剩余需要考虑的x只剩两位，只可能是01或00。
  x = x>>b1;
  b0 = !!x;	
  return b16+b8+b4+b2+b1+b0+1;
}
```

#### 11. floatScale2

```c
unsigned floatScale2(unsigned uf) {
  int exp = (0x7f800000 & uf)>>23;	//取exp（30~23bit）
  int sign = (1<<31)&uf;	//取标志位
  if(exp==0) return (uf<<1)|sign;	//阶域码全0，后面是0.xxxxx，所以直接左移
  if(exp==255) return uf;
  return ((exp+1)<<23)|(0x807fffff & uf);	//规格化，exp部分加1就表示乘2
}
```

#### 12. floatFloatInt

```c
int floatFloat2Int(unsigned uf) {
  int s_ = uf>>31;
  int exp_ = ((0x7f800000 & uf)>>23)-127;	//注意-127
  int frac_ = (uf & 0x007fffff)|0x00800000;	//注意这里符号位默认为0，所以之后要判断
  if(!(uf&0x7fffffff)) return 0;  //uf为0，返回0
  
  if(exp_ > 31) return 0x80000000u;	//frac部分是大于1的，所以这里不能大于31
  if(exp_ < 0) return 0;

  if(exp_ > 23) frac_<<=(exp_-23);	//把小数部分转化为整数
  else frac_>>=(23-exp_);	//如果exp不够就舍弃后面的小数

  if(!((frac_>>31)^s_)) return frac_;	//若符号相同，直接返回
  else if(frac_>>31) return 0x80000000;	//原正现负说明溢出了
  else return ~frac_+1;	//原负现正，去相反数即可。
}
```

#### 13. floatPower2

模仿浮点数表示2^x

```c
unsigned floatPower2(int x) {
    int exp = x+127;
    if(exp<0) return 0;
    if(exp>=255) return 0x7f800000;
    return exp<<23;
}
```

![1594661407030](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1594661407030.png)