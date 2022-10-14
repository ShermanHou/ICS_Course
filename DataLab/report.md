# DataLab report
## all pass!(22/22)
![pass](/datalabscore.png)

## Problem1 bitXor

从可用出发，可用的是~和&，

直接使用&可以得到(有0的pairs)->0,也即(全1的pairs)->1

与~结合，还可以得到(有1的pairs)->0,也即(全0的pairs)->1

我们需要的是(全0的或全1的)->0,也即(不全为0且不全为1的)->1
故为~全1&~全0

```c
int bitXor(int x, int y) {
  int a = x & y;//全为1的位
  int b = ~x & ~y;//全为0的位
  return  ~a & ~b;//不全为1
}
```

## Problem2 thirdBits

题目即从LSB开始记为第1位，1，4，7...这样每三位赋1，返回32位的结果。
由于题目限制给定常数最长可以是8bits,我们需要通过对"位段"移位来拓展。

初值赋为ox49,移位使得恰好错位拓展2倍，<<9，获得18长度的，

再<<18，获得36长度的,return int截取32位即可。

```c
int thirdBits(void) {
  int x = 0x49;
  x = (x<<9) + x;
  x = (x<<18) + x;
  return x;
}
```

## Problem3 fitsShort

题目即给一个32位的数，判断16位是否足够表示它。

若可以，最左边17位需要全为1或者0，但是，

这个想法有点绕，回归本质：即最左边16位由前一位拓展，不妨借助算数移位的特性，构造补位数，抑或原来的，判断是否一样。

```c
int fitsShort(int x) {
  int y = (x<<16)>>16;
  int z = y^x;
  return !z;
}
```

## Problem4 isTmax

判断x是否是0x7FFFFFFF这个数，一开始的思路是直接刨析这个数表示的本身，第一位是0，剩下的是1,不能用移位，不好办;

想一下它的特性，特殊点在于"最大"，只有它再大点会出"问题":加了1，变为负数?负数的判断本质还是要用移位才比较好，实际上我们不应该考虑负数不负数的，要考虑的是位本身的特点——加1后的再左移1位成了0!
这里的左移1位是容易实现的，只需要加上自己就行了。

这似乎是其他数都做不到的，真的是这样吗？很不幸，-1也是如此，因此要排除-1。
| x | 0x7FFFFFFF | 0xFFFFFFFF |
| :-----:| ----: | ----: |
| x+1 | 0x00 80000000 | 0x01 00000000 |
| (x+1)+(x+1) | 0x08 00000000 | 0x10 00000000 |

其实区别很简单...-1加1是0，而0x7FFFFFFF+1不是。不能是0，即取非后要为0，把所需为0的部分或！(x+1)即可。

```c
int isTmax(int x) {
  int y = x + 1;
  int z = y + y;
  return !(z|(!y));
}
```

## Problem5 fitsBits

输入n和x,判断x是否能只用n位表示。若能，同第3题，则第n位前的都由第n位拓展，左移32-n位，再右移这么多位。判断与原数是否相等即可。

```c
int fitsBits(int x, int n) {
  int m = 33 + ~n;
  int nbitX = (x << m) >> m;
  return !(x^nbitX);
}
```

利用移位特性的另一种方法：移位是先对32取模然后再移位，
+31相当于对>0的n减1，将n=0变为31 记为`m`
这样右移m位获得的**符号扩展段**记为`highbits`，需要满足nbitX是0或-1；

采用再移n位与`highbits`亦或的方式来判断。原理如下：

- n>1,`highbits`再右移n位（即右移正数位），仅0或-1才会满足移动后与`highbits`相等；

- n=1,再右移1位，当且仅当`highbits`是0时才相等（-1不满足要求）

```c
int fitsBits(int x, int n) {
  int m = 31 + n;
  int highbits = x >> m;
  return !((highbits>>n)^highbits);
}
```

## Problem6 upperBits

应该还是需要利用算数移位的特性，构造100...0，右移n-1位即可。但n=0时会是右移-1位，取模后为31，会移位为-1，在答案加!n即可实现分类讨论。

```c
int upperBits(int n) {
    return ((1<<(31))>>(n+~0))+!n;
}
```

利用移位特性的另一种方法：移n加31位，这样32归为31，1~31全实现了减一；

n=0单独谈论，可以在`1`的获取过程中分类。

```c
int upperBits(int n) {
  int one = (!!n)<<31;
  return one >>(31+n);
}
```

## Problem7 anyOddBit

构造掩码，设为奇数位全为1，偶数位全为0的；与原数作&操作，奇数位有1的返回非0数，无1的返回0，取两次!即满足题设。

```c
int anyOddBit(int x) {
    int mask = 0xAA;
    mask = mask+(mask<<8);
    mask = mask+(mask<<16);
    return !!((x&mask));
}
```

## Problem8 byteSwap

直观上，所需作的操作是把位段作交换，需要用一定的掩码构造一下加一下；但是，稍微仔细想一下就不难知道，这样的操作在位变化角度看来效率很低，有些位实际上并没有变化。

我们需要关注的是两段在相同相对位置处相异的位。n和m<<3位乘8倍后，x往右分别推这么多位，求^,保留不同的位为1，再用0xFF过滤一下即得**描述差异的段**。再移动到两个相应位置，用^作用于x，把**有差异、需要改变的位**0，1置反即可。

```c
int byteSwap(int x, int n, int m) {
  int y = 0;
  n = n<<3;
  m = m<<3;
  y = 0xff & ((x>>n)^(x>>m));
  x = x ^ (y<<n); 
  x = x ^ (y<<m);
  return x;
}
```

## Problem9 absVal

需要先对正负数作分类讨论，先取符号位，右移后实际上得到：负数为-1，正数为0；

与上一个题中最后^符号的利用类似，a^一个数会使**这个数为1的位上**a的位置置反；

极端情况即为，^0无变化，^(-1)等价于取~,潜在实现了是否~的分类操作。
$A+(-A)=0$
$A+(\~A)=-1$
$-A=\~A+1$

因为取相反数，+-1，取~均为互逆运算，$\~A+1=\~(A-1)$
直接在-1和^处利用上面得到的-1(和0)操作即可。

```c
int absVal(int x) {
  int mark = x>>31;
  return (x+mark)^mark;
}
```

## Problem10 divpwr2

简单尝试后发现，对于正数而言，x>>n即扔掉不够2^n的余数，得到向更小(也即向0)取整的商。对于负数，这样求得的商会是扔掉一部分正数，其余位降低n个权重，为向$-\infty$取整，而非向0取整。要想实现对负数的向0取整，需要加个偏置。

细化一下这个想法：我们想让那些余数在[1, 2^n)之间的负数除后的商加1；加2^n-1才可恰好满足要求；-1可通过符号位算数右移得到，以此为基计算偏置，便自动实现了是否加偏置的分类。

```c
int divpwr2(int x, int n) {
  int mark = x>>31;
  int add = ((mark & 1) << n) + mark; //符号0 1左移
      //加0或-1 给负数的x补充2^n-1,实现选择性加1
  return (x + add) >> n;
}
```

## Problem11 float_neg

对浮点数取负数需要单独考虑NaN;而该题由于是对unsigned操作，可以使用长度任意的常量，左移1位若大于exp位全为1的，则为NaN

```c
unsigned float_neg(unsigned uf) {
  if((uf<<1) > 0xFF000000)
        return uf;
  return uf^0x80000000;
}
```

## Problem12 logicalNeg

只有0的相反数是其本身，用虽为整体作用，但只考虑x和-x的符号位间的关系的操作单独区分出0；结果加1正好符合要求。

```c
int logicalNeg(int x) {
  int negX = ~x + 1;
  return ((x | negX) >> 31) + 1;
}
```

## Problem13 bitMask

所求mask可溯源拆分为两个mask作用后的结果：因为有high和low的规定顺序，我们需要一种错位:用lowbit左移`~0`来创造高位为1低位为0的highmask；相似地，我们也想用highbit来创造低位为1，高位为0的lowmask,如何做呢？对`~0`加“只有特定位置上为1的数”即可实现。

```c
int bitMask(int highbit, int lowbit) {
  int all1 = ~0;
  int highMask = all1 << lowbit;
  int lowMask = ((2 << highbit) + all1);
  return highMask & lowMask;
}
```

## Problem14 isGreater

直接想到的是去判断是否满足x-y>0，

即`x+~y+1>0`，也即`x+~y>-1`；即`x+~y>=0`,取符号位判断即可。

若两数同号，必然不会overflow；
但若x和y异号，有时候x-y会受限于int32位的限制而overflow；
这种情况具体实例有：最小值0x80000000再减一个正数；以及一个正数减去0x80000000。

这种情况需要单独判断。我们需要构造仅对应上述两种情况的区分值。前述x-y的判断是针对符号位的，上述越界情况也是针对符号位，我们不妨最后一起再>>31位。

简单尝试发现，直接对x和y做位操作不能实现针对性区分;再结合前述x-y的实现，发现借助x和`~y`的|操作可以区分出x为正数，y为负数，对应0；x和`~y`的&操作可以区分出x为负数，y为正数，对应1。最后组合构造即可。

```c
  int invy = ~y;
  int pn = x|invy;//0
  int np = x&invy;//1
  return !((np|((pn)&(x+invy)))>>31);
```

另一种方法：
通过对`x+~y>=0`中的y的改动，将对x和y异号的判断合并进去，

- 异号且x符号位扩展为-1,加1，return0

- 非异号，因为|, 不会对y产生影响；`x+~y`的符号位再加1,return 1

这样符号位拓展结果和+1连用可以把结果限制在1或0中。

```c
int isGreater(int x, int y) {
  return (((x + ~(((x ^ y) >> 31) | y))) >> 31) + 1;
}
```

## Problem15 logicalShift

消除算数移位误拓展出来的1即可。取符号位，模拟出这一部分，最后^操作即可。

```c
int logicalShift(int x, int n) {
  int aShift = x >> n;
  int sign = x >> 31;
  int mask = sign << 31 >> n << 1; //多次移位取代先操作后移位
  return aShift ^ mask;
```

利用一种trick和13题提到的+特定位上的1来实现高段取反的另一种方法：

**trick**!

由于31的特殊性，`31^`相当于最低5位范围内取反，作用于x，得到的31^x与x的和是31，即得31-x; 1左移这么多位后在（31-n）处；

x原来的符号位(31)>>n后也在(31-n)处。

而后利用13题提到的+特定位上的1来实现包含(31-n)位在内的高段取反；但（31-n）位是不该动的，最后^回来这一位即可。

```c
int logicalShift(int x, int n) {
  int maskbit = 1 << (31 ^ n);
  return ((x >> n) + maskbit) ^ maskbit;
}
```

## Problem16 satMul2

x<<1后符号改变，则意味着overflow，用flow记录下来，-1对应flow, 0对应正常；

用与flow的|过滤x, 用与flow的&过滤（sat的~），最后^连结两部分即可。

```c
int satMul2(int x) {
  int tmin = 1 << 31; // 0x80000000

  int x2 = x << 1;           // 2*x
  int flow = (x ^ x2) >> 31; // all1

  int brush_x2 = x2 | flow; //若flow了，brush to all1
  int xneg = x >> 31;
  return (brush_x2) ^ (flow & (tmin ^ (xneg)));
}
```
他山之石 as following...
改进方法1 :
```c
int satMul2(int x) {
  int x2 = x<<1;
  int flow = (x^x2)>>31;
  int res1 = x2>>(flow&31);//通过是否移位而非^来分类：若未溢出则仍为x2；若溢出则x2为负数对应x为正数res1=-1，x2为正数对应x为负数res1=0。
  int res2 = flow<<31;//若未溢出则为0，若溢出则为Tmin。
  int res = res1+res2;
  return res;
}
```
改进方法2:
```c
int satMul2(int x)
{
  int x2 = x << 1;
  int mask = (x ^ x2) >> 31;
  int mul2OrItssign = x2 >> mask;//巧妙利用>>-1位的特性，若flow,获得的将会是2*x符号位的拓展,加1后恰好是所需sat
  int maskmask = mask << mask;//-1to1,0to0 mask叠mask,很精炼ww
  return mul2OrItssign + maskmask;
}
```

## Problem17 subOK

当x>0 y<0 x-y<0或x<0 y>0 x-y>0时flow, 概括为x和y异号，且与（x-y）异号。

```c
int subOK(int x, int y){
  //当x>0 y<0 x-y<0或x<0 y>0 x-y>0时flow
  int dif = x + ~y + 1;
  return !(((x^y) & (x^dif)) >> 31);
}
```

x-y转为-(y-x)的想法类似14 isGreater，这样的好处在于：

x-y>0即x+~y+1>0，x+~y>=0；x-y<0即x+~y+1<0，x+~y<-1，只用x+~y是不够的。

-(y-x) 即~(y+~x)，取(y+~x)，与y^, x和y异号的前提下, y和(y-x)符号相同即x和(x-y)符号相反! 这样无共同元素x的两两的比较可以实现不等的传递！使用y+~x即可，无需再加1!

```c
    int subOK(int x, int y){
    int dif = y+~x;
    return !(((x^y) & (y^dif)) >> 31);
    }
```

## Problem18 trueThreeFourths

经过尝试，先乘3容易overflow，于是考虑先除4再乘3

先对x右移两位实现除4，获得商，与3取&获得对4的余数；

左移一位再加本身实现乘3，获得积，乘3后对4的余数也需作相应操作获取；

因为负数要round toward 0, 需要单独对负数的余数加3（4-1），再统一右移两位除4

```c
int trueThreeFourths(int x)
{
  int xdiv4 = x >> 2;
  int rem = x & 0x3;

  int xmul3div4 =(xdiv4 << 1)+ xdiv4;

  int rem3 = (rem << 1) + rem;

  int xneg = x >> 31;  
  int left = (rem3 + (xneg & 0x3)) >> 2;

  return xmul3div4 + left;
}
```

## Problem19 isPower2

特殊之处在于减1后由于退位，没有任何一位和原来的数一样；

但需要排除0，根据题目限制还需排除负数。利用y可以把0归入负数，用x|排除符号位为1的即可。之所以不能只用y，是因为需要考虑0x80000000减一后越界的情况。

```c
int isPower2(int x) {
  int y = x + ~0;
  return !((x & y) | ((x|y) >> 31));
}
```

小改进：不可能有2的正数幂-1后30位处仍为1，符合条件的数的30位应该全为0；x不为正和x不为Tmin可以进一步合并为判断最高两位是不是0，即通过右移30位实现。

```c
int isPower2(int x)
{
  int y = x + ~0;
  return !((x & y) | (y >> 30));
}
```

## Problem20 float_i2f

若x为0直接返回0。若x为非0，进行如下操作:

- sign保存x的符号位

- absx保存x的绝对值

- while循环实现小数点的移动，x的绝对值恰好小于1时退出循环；期间，shift记录阶码

- 上述while循环退出时的absx最后9位为x的小数部分

- 对于进位，需要对两种情况单独处理:
  
  - absx最后9位大于0x100，向上舍入
  
  - absx的9位处出于中间值状态，向偶数舍入

最后将符号位，指数位，小数位和舍入偏置bias相加即为所求。

```c
unsigned float_i2f(int x)
{
  unsigned sign = 0, shift = 159, bias = 0, tmp;
  unsigned absx = x;
  if (x == 0)
    return 0;
  if (x < 0)
  {
    sign = 0x80000000;
    absx = -x;
  }
  while (1)
  {
    tmp = absx;
    absx <<= 1;
    shift--;

    if (tmp & 0x80000000)
      break;
  }
  if ((absx & 0x01ff) > 0x0100)
    bias = 1; //向上舍入
  if ((absx & 0x03ff) == 0x0300)
    bias = 1; //需要向偶数舍入

  return sign + (absx >> 9) + ((shift) << 23) + bias;
}
```
他山之石 15op...
```c
unsigned float_i2f(int x)
{
unsigned ux = x;
	unsigned signexp = 0x4E800000;
	int flag = 0;
	int round = 0;
	if (x) {
		while (1) {
			if (ux & 0x80000000) {
				if (flag)
					break;
				else {
					ux = -x;
					signexp = 0xCE800000;
				}
			} else {
				ux <<= 1;
				signexp -= 0x800000;
			}
			flag = 1;
		}
		if (ux & 0x80)
			if (ux & 0x17F)
				round = 1;
		return signexp + (ux >> 8) + round;
	}
	return 0;
}
```
## Problem21 howManyBits

要给出确切数值，故考虑采用二分法逼近。

负数取非，其至少需要取非后的最左非0位所在数位+1位来表示(左边全1作为符号段)，基数`n`设置为0，因为正数也许+1位的符号位，最后一齐加1即可。

每次判断左半段是否为0：

- 为0，则可以无视这半段了，接下来的二分对右半段做；

- 不为0，则至少到用右半段所在位来表示，无视右半段，接下来继续对左半段二分；

对左右半段的选择通过每次二分对n的选择性加半段的位数来实现。

过程中，n的含义可以说是最左边非0位的当前已知最小位置。
```c
 int howManyBits(int x)
  {
    int n = 0; 
    //负数取反
    x = x ^ (x >> 31);

    n = ((!!(x >> 16)) << 4);          // 如果x右移16位后是0，说明x的高16位是0。如果不是，n累加 1<<4 = 16
    n = n + ((!!(x >> (8 + n))) << 3); // n基础上再右移8位
    n = n + ((!!(x >> (4 + n))) << 2); // n基础上再右移4
    n = n + ((!!(x >> (2 + n))) << 1); // n基础上再右移2
    //接下来是最后要解决的2位，所需映射关系为：3to3, 2to3, 1to2, 0to1
    //法1
    // n = n + (x >> (1 + n));
    // n = n + (x >> n);
    // n = n + 1;
    //法2
    x = x >> n;
    n+=x;
    n+=((x+1)>>2)^1;
    //法3
    //x = x >> n;
    // n+=x|1;
    // n+=!(x^1);
    return n;
  }
```
另一种方法:x<<1而后^原来的x从而固定最左边为1的是符号位，以此更好更精简地合并正负数
```c
int howManyBits(int x)
{
  int b16, b8, b4, b2, b1;
  x ^= (x << 1);
  b16 = ((!!(x >> 16)) << 4);
  x >>= b16;
  b8 = (!!(x >> 8)) << 3;
  x >>= b8;
  b4 = (!!(x >> 4)) << 2;
  x >>= b4;
  b2 = ((!!(x >> 2)) << 1) + 1;
  x >>= b2;
  b1 = x & 1;
  return b16 + b8 + b4 + b2 + b1;
}
```
他山之石
再改进：
`b2 = ((!!(x >> 2)) << 1) + 1;`一行中的b2要么为1要么为3，^3可再省略一个操作符，
变为`b2 = (!(x >> 2) << 1) ^ 3;`

else
26op solution:
```c
int howManyBits(int x) {
	int t = x ^ (x << 1);
	int sum = 31;
	int temp;
	temp = !(t >> 16) << 4;//! ^ rather than !! +
	sum ^= temp; t <<= temp; 
	temp = !(t >> 24) << 3;
	sum ^= temp; t <<= temp; 
	temp = !(t >> 28) << 2;
	sum ^= temp; t <<= temp;
	temp = !(t >> 30) << 1;
	sum ^= temp; t <<= temp;
	sum ^= !(t >> 31);
	return sum + 1;
}
```
22op solution...
```c
int howManyBits(int x) {
	// int ans;
	// x = x ^ (x << 1);
	// ans = (!(x >> 16)) << 4;
	// ans ^= 24;
	// ans ^= (!(x >> ans)) << 3;
	// ans ^= 4;
	// ans ^= (!(x >> ans)) << 2;
	// ans += ((~0x5B) >> ((x >> ans) & 30)) & 3;
	// return ans + 1;
  int sign = x >> 0x1F;
  int ans = 0x1F, cur;
  x ^= sign;
  cur = (!(x >> 0x0F)) << 4;
  ans ^= cur;
  x <<= cur;
  cur = (!(x >> 0x17)) << 3;
  ans ^= cur;
  x <<= cur;
  cur = (!(x >> 0x1B)) << 2;
  ans ^= cur;
  x <<= cur;
  cur = (!(x >> 0x1D)) << 1;
  ans ^= cur;
  x <<= cur;
  cur = (!(x >> 0x1E));
  ans ^= cur;
  return ans + 1;
}
```

## Problem22 float_half
首先记录uf的指数位，并获得uf的绝对值；

而后判断是否为>=NaN的特殊数，是否为非规范数。
最后，对于规范数，右移1位并加round实现向偶数取整的除2。
```c
unsigned float_half(unsigned uf)
{
  unsigned NaN = 0x7f800000;
  unsigned sign = (uf & 0x80000000);
  unsigned absuf = uf - sign;
  unsigned shift;
  unsigned round = (absuf & 3) == 3;
  int add;
  if (absuf >= NaN)
  {
    shift = 0;
    add = 0;
  }
  else if (absuf >= 0x900000)
  {
    shift = 0;
    add = -0x800000;
  }
  else
  {
    shift = 1;
    add = round;
  }
  return sign + (absuf >> shift) + add;
}
```
另外的写法：
```c
unsigned float_half(unsigned uf) {
	int round=!((uf & 3) ^ 3);
	int temp=uf&0x7fffffff;
	int d=uf&0x7f800000;
	if(d<=0x800000)
	{
		int p=temp>>1;
		return p+(uf&0x80000000)+round;
	}
	if(temp>=0x7f800000)
		return uf;
	uf=uf-0x800000;
	return uf;
}
```