---
layout: wiki
title: CodeTemplate
categories: Java
description: CodeTemplate
keywords: Java
---

## 输入问题

&emsp;&emsp;Codeforces著名世界级选手Petr大爷写的Java输入内部类:  
```
class InputReader {
    public BufferedReader reader;
    public StringTokenizer tokenizer;

    public InputReader(InputStream stream) {
        reader = new BufferedReader(new InputStreamReader(stream), 32768);
        tokenizer = null;
    }

    public String next() {
        while (tokenizer == null || !tokenizer.hasMoreTokens()) {
            try {
                tokenizer = new StringTokenizer(reader.readLine());
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
        return tokenizer.nextToken();
    }

    public int nextInt() {
        return Integer.parseInt(next());
    }

}
```

## 输出问题

&emsp;&emsp;文档：https://www.cs.colostate.edu/~cs160/.Summer16/resources/Java_printf_method_quick_reference.pdf  

&emsp;&emsp;常见输出格式：  
```
    // 定义一些变量，用来格式化输出。
    double d = 345.678;
    String s = "你好！";
    int i = 1234;
    // "%"表示进行格式化输出，"%"之后的内容为格式的定义。
    System.out.printf("%f", d);// "f"表示格式化输出浮点数。
    System.out.println();
    System.out.printf("%9.2f", d);// "9.2"中的9表示输出的长度，2表示小数点后的位数。
    System.out.println();
    System.out.printf("%+9.2f", d);// "+"表示输出的数带正负号。
    System.out.println();
    System.out.printf("%-9.4f", d);// "-"表示输出的数左对齐（默认为右对齐）。
    System.out.println();
    System.out.printf("%+-9.3f", d);// "+-"表示输出的数带正负号且左对齐。
    System.out.println();
    System.out.printf("%d", i);// "d"表示输出十进制整数。
    System.out.println();
    System.out.printf("%o", i);// "o"表示输出八进制整数。
    System.out.println();
    System.out.printf("%x", i);// "d"表示输出十六进制整数。
    System.out.println();
    System.out.printf("%#x", i);// "d"表示输出带有十六进制标志的整数。
    System.out.println();
    System.out.printf("%s", s);// "d"表示输出字符串。
    System.out.println();
    System.out.printf("输出一个浮点数：%f，一个整数：%d，一个字符串：%s", d, i, s);
    // 可以输出多个变量，注意顺序。
    System.out.println();
    System.out.printf("字符串：%2$s，%1$d的十六进制数：%1$#x", i, s);
```
```
    345.678000
       345.68
      +345.68
    345.6780 
    +345.678 
    1234
    2322
    4d2
    0x4d2
    你好！
    输出一个浮点数：345.678000，一个整数：1234，一个字符串：你好！
    字符串：你好！，1234的十六进制数：0x4d2
```

## 精度问题

#### 问题分析  

&emsp;&emsp;丫的，前两天做题发现了这么个奇葩的问题，代码如下：  

```
    public class Main {

        public static void main(String[] args) throws Exception {
            System.out.println(0.2 + 0.1);
            System.out.println(0.4 - 0.3);
            System.out.println(0.1 * 0.2);
            System.out.println(0.6 / 0.2);
            System.out.println(4.1 - 1.1);
        }
        //    0.30000000000000004
        //    0.10000000000000003
        //    0.020000000000000004
        //    2.9999999999999996
        //    2.9999999999999996

    }
```  
&emsp;&emsp;尤其是最下面那组4.1-1.1，解题的一个步骤是要求两个数(double类型)差的绝对值，然后向下取整，我直接用Math.floor(Math.abs(4.1-1.1));
发现结果竟然等于2，后来经过网上搜集资料，发现Java在精度上是有问题的！  

理论解释  

&emsp;&emsp;借用《Effactive Java》书中的话，float和double类型的主要设计目标是为了科学计算和工程计算。他们执行二进制浮点运算，这是为了在广域数值范围上提供较为精确的快速近似计算而精心设计的。然而，它们没有提供完全精确的结果，所以不应该被用于要求精确结果的场合。但是，商业计算往往要求结果精确，这时候BigDecimal就派上大用场了。  

&emsp;&emsp;在学习计算机组成原理的时候我们应该都对浮点数有一定的了解，看下这个演示的例子就知道为啥会出现丢失精度的问题：https://www.zhihu.com/question/42024389  

解决办法  

&emsp;&emsp;使用Java中的BigDecimal，主要有如下几种构造方式：  
```
    1.public BigDecimal(double val)    //将double表示形式转换为BigDecimal, 不建议使用

    2.public BigDecimal(int val)　　   //将int表示形式转换成BigDecimal

    3.public BigDecimal(String val)　　//将String表示形式转换成BigDecimal
```  

&emsp;&emsp;为什么不推荐使用double构造BigDecimal对象呢，看下面的实例：  

```
    import java.math.BigDecimal;
    public class Main {

        public static void main(String[] args) throws Exception {
            BigDecimal bigDecimal = new BigDecimal(2);
            BigDecimal bDouble = new BigDecimal(2.3);
            BigDecimal bString = new BigDecimal("2.3");
            System.out.println("bigDecimal=" + bigDecimal);
            System.out.println("bDouble=" + bDouble);
            System.out.println("bString=" + bString);
        }
        //    bigDecimal=2
        //    bDouble=2.29999999999999982236431605997495353221893310546875
        //    bString=2.3

    }
```

&emsp;&emsp;为什么会出现这种情况呢？  

- 1.参数类型为double的构造方法的结果有一定的不可预知性。有人可能认为在Java中写入newBigDecimal(0.1)所创建的BigDecimal正好等于 0.1（非标度值 1，其标度为 1），但是它实际上等于0.1000000000000000055511151231257827021181583404541015625。这是因为0.1无法准确地表示为 double（或者说对于该情况，不能表示为任何有限长度的二进制小数）。这样，传入到构造方法的值不会正好等于 0.1（虽然表面上等于该值）。  

- 2.String 构造方法是完全可预知的：写入 newBigDecimal("0.1") 将创建一个 BigDecimal，它正好等于预期的 0.1。因此，比较而言，通常建议优先使用String构造方法。  

&emsp;&emsp;当必须使用double构造时，可以使用String进行过桥，避免丢失精度，例如：BigDecimal b1 = new BigDecimal(Double.toString(v1));
  
加减乘除  

```
    public BigDecimal add(BigDecimal value);                        //加法

    public BigDecimal subtract(BigDecimal value);                   //减法 

    public BigDecimal multiply(BigDecimal value);                   //乘法

    public BigDecimal divide(BigDecimal value);                     //除法
```  

&emsp;&emsp;值得注意的是对于除法，如果不能整除，运行时会出现java.lang.ArithmeticException异常，解决办法是添加额外的参数：public BigDecimal divide(BigDecimal divisor, int scale, int roundingMode)。其中第一参数表示除数，第二个参数表示小数点后保留位数，第三个参数表示舍入模式，只有在作除法运算或四舍五入时才用到舍入模式，有下面这几种：  

- ROUND_CEILING        //向正无穷方向舍入
- ROUND_DOWN           //向零方向舍入
- ROUND_FLOOR          //向负无穷方向舍入
- ROUND_HALF_DOWN      //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，向下舍入, 例如1.55 保留一位小数结果为1.5
- ROUND_HALF_EVEN      //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，如果保留位数是奇数，使用ROUND_HALF_UP，如果是偶数，使用ROUND_HALF_DOWN
- ROUND_HALF_UP        //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，向上舍入, 1.55保留一位小数结果为1.6
- ROUND_UNNECESSARY    //计算结果是精确的，不需要舍入模式
- ROUND_UP             //向远离0的方向舍入  

&emsp;&emsp;例如，常见的四舍五入为：System.out.println("a / b =" + a.divide(b, 10, BigDecimal.ROUND_HALF_UP));  

&emsp;&emsp;此外，四则运算后会产生新的对象，即BigDecimal对象不可变！  

截断操作  

&emsp;&emsp;需要对BigDecimal进行截断和四舍五入可用setScale方法，例如：  

```
    import java.math.*;
    import java.math.BigDecimal;
    public class Main {

        public static void main(String[] args) throws Exception {
            BigDecimal a = new BigDecimal("4.5635");
            a = a.setScale(3, RoundingMode.HALF_UP);    //保留3位小数，且四舍五入
            System.out.println(a);
        }

    }
```
常见变量  

```
    BigDecimal zero = BigDecimal.ZERO;  
    BigDecimal one = BigDecimal.ONE;  
    BigDecimal ten = BigDecimal.TEN; 
```  

比较操作  

```
    BigDecimal one = BigDecimal.valueOf(1);  
    BigDecimal two = BigDecimal.valueOf(2);  
    BigDecimal three = one.add(two);  
    int i1 = one.compareTo(two);//-1  
    int i2 = two.compareTo(two);//0  
    int i3 = three.compareTo(two);//1  
```  

完整API  

&emsp;&emsp;地址：http://www.javaweb.cc/help/JavaAPI1.6/java/math/class-use/BigDecimal.html  
  
&emsp;&emsp;地址：https://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html

源码分析  

&emsp;&emsp;有待补充  

#### 解题模板  

```
class Arith{
    private static final int DEF_DIV_SCALE = 10; //这个类不能实例化
    private Arith(){
    }
    /**
     * 提供精确的加法运算。
     * @param v1 被加数
     * @param v2 加数
     * @return 两个参数的和
     */
    public static double add(double v1,double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.add(b2).doubleValue();
    }
    /**
     * 提供精确的减法运算。
     * @param v1 被减数
     * @param v2 减数
     * @return 两个参数的差
     */
    public static double sub(double v1,double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.subtract(b2).doubleValue();
    }
    /**
     * 提供精确的乘法运算。
     * @param v1 被乘数
     * @param v2 乘数
     * @return 两个参数的积
     */
    public static double mul(double v1,double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.multiply(b2).doubleValue();
    }
    /**
     * 提供（相对）精确的除法运算，当发生除不尽的情况时，精确到
     * 小数点以后10位，以后的数字四舍五入。
     * @param v1 被除数
     * @param v2 除数
     * @return 两个参数的商
     */
    public static double div(double v1,double v2){
        return div(v1,v2,DEF_DIV_SCALE);
    }
    /**
     * 提供（相对）精确的除法运算。当发生除不尽的情况时，由scale参数指
     * 定精度，以后的数字四舍五入。
     * @param v1 被除数
     * @param v2 除数
     * @param scale 表示表示需要精确到小数点以后几位。
     * @return 两个参数的商
     */
    public static double div(double v1,double v2,int scale){
        if(scale<0){
            throw new IllegalArgumentException(
                    "The scale must be a positive integer or zero");
        }
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.divide(b2,scale,BigDecimal.ROUND_HALF_UP).doubleValue();
    }
    /**
     * 提供精确的小数位四舍五入处理。
     * @param v 需要四舍五入的数字
     * @param scale 小数点后保留几位
     * @return 四舍五入后的结果
     */
    public static double round(double v,int scale){
        if(scale<0){
            throw new IllegalArgumentException("The scale must be a positive integer or zero");
        }
        BigDecimal b = new BigDecimal(Double.toString(v));
        BigDecimal one = new BigDecimal("1");
        return b.divide(one,scale,BigDecimal.ROUND_HALF_UP).doubleValue();
    }
}
```

## 类型转换

#### String转double
```
public static boolean isDouble(String data) {
        try{
            double tmp = Double.parseDouble(data);
            return true;
        }
        catch (Exception e) {

        }
        return false;
    }
```

#### ArrayList转int[]
```
class Transformation {

    // ArrayList转int[]
    public static int[] list2Ints(ArrayList<Integer> arrayList){
        return arrayList.stream().mapToInt(i->i).toArray();
    }

}
```

&emsp;&emsp;持续更新中

## 进制转换

&emsp;&emsp;任意进制间的转换，使用10进制进行桥接即可。利用了StringBuilder、取余等操作即可，实现过程如下：
```
import java.util.*;
import java.io.*;
import java.math.*;
public class Main{

    private static char[] array = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();
    private static String numStr = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

    public static void main(String[] args) throws Exception{
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
        Scanner in = new Scanner(System.in);
        while(in.hasNext()){
            int from = in.nextInt();
            int to = in.nextInt();
            String input = in.next();
            // 任意进制转任意进制 使用10进制进行桥接
            System.out.println(getData(from, to, input));
        }

    }
    // 获取结果
    public static String getData(int from, int to, String data) {
        Long tmp_10= Hexadecimal_From_Random_To_10(data, from);
        String tmp_random = Hexadecimal_From_10_To_Random(tmp_10, to);
        String ans = tmp_random.replaceFirst("^0*", "");
        return ans;
    }
    // 10进制转任意进制
    public static String Hexadecimal_From_10_To_Random(long data, int N) {
        Long tmp = data;
        Stack<Character> stack = new Stack<Character>();
        StringBuilder result = new StringBuilder(0);
        while (tmp != 0) {
            stack.add(array[new Long((tmp % N)).intValue()]);
            tmp = tmp / N;
        }
        for (; !stack.isEmpty();) {
            result.append(stack.pop());
        }
        return result.length() == 0 ? "0":result.toString();
    }
    // 任意进制转10进制
    public static long Hexadecimal_From_Random_To_10(String number, int N) {
        char ch[] = number.toCharArray();int len = ch.length;
        long result = 0;
        if (N == 10){
            return Long.parseLong(number);
        }
        long base = 1;
        for (int i = len - 1; i >= 0; i--) {
            int index = numStr.indexOf(ch[i]);
            result += index * base;
            base *= N;
        }
        return result;
    }

}
```

## 字典Map的使用

- 1.HashMap
- 2.LinkedHashMap
- 3.TreeMap

&emsp;&emsp;HashMap最基本的Map，遍历时的顺序与插入顺序无关  

&emsp;&emsp;LinkedHashMap是特殊的一种Map，遍历时的顺序与插入顺序相同  

&emsp;&emsp;TreeMap默认是按照key的字典序升序排列，如果想降序的话按照如下操作:  
```
    Map<Integer, Integer> map = new TreeMap<Integer, Integer>(new Comparator<Integer>(){
        /*
         * int compare(Object o1, Object o2) 返回一个基本类型的整型，
         * 返回负数表示：o1 小于o2，
         * 返回0 表示：o1和o2相等，
         * 返回正数表示：o1大于o2。
         */
        public int compare(Integer a,Integer b){
            return b-a;
        }
    });
```

&emsp;&emsp;黑科技：Java8中引入了getOrDefault(key, 找不大时返回的值)方法。  

## 数学问题

#### 数列片段和

&emsp;&emsp;例如，给定数列{0.1, 0.2, 0.3, 0.4}，我们有(0.1) (0.1, 0.2) (0.1, 0.2, 0.3) (0.1, 0.2, 0.3, 0.4) (0.2) (0.2, 0.3) (0.2, 0.3, 0.4) (0.3) (0.3, 0.4) (0.4) 这10个片段。 给定正整数数列，求出全部片段包含的所有的数之和。如本例中10个片段总和是0.1 + 0.3 + 0.6 + 1.0 + 0.2 + 0.5 + 0.9 + 0.3 + 0.7 + 0.4 = 5.0。  

&emsp;&emsp;套路：Sum+=(double)(N-i)*(double)(i+1)*a[i];  

#### 简单约瑟夫问题

&emsp;&emsp;出列问题：令res = 0;for(int i=2;i<=n;i++){res = (res+偏移量)%i};System.out.println((res+1));  

#### 快速幂

&emsp;&emsp;给出三种求出快速幂的方案，原题为：https://leetcode-cn.com/problems/powx-n/description/  

```
class Solution {
    public double myPow(double x, int n) {
        
        // 暴力法，不解释
        //return Math.pow(x, n);
        
        if(n==0)
            return 1.0;
        if(x==0)
            return 0.0;
        
        // 迭代法
        // double res = 1.0;
        // double base = x;
        // int tmp = n;
        // while (tmp!=0) {
        //     if(tmp%2!=0)
        //         res *= base;
        //     base *= base;
        //     tmp /= 2;
        // }
        // return n <0 ? 1.0 / res : res;
        
        // 递归法
        if(n==0)
            return 1;
        double half = myPow(x, n/2);
        if(n%2==0)
            return half * half;
        else if(n>0)
            return half * half * x;
        else
            return half * half / x;
        
    }
}
```

#### 快速幂取模

&emsp;&emsp;计算a的b次方对c取模，算法正确性未探知  

```
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int a = in.nextInt(), b = in.nextInt(), c = in.nextInt();
        int res = 1;
        a %= c;
        for (; b != 0; b /= 2) {
            if (b % 2 == 1)
                res = (res * a) % c;
            a = (a * a) % c;
        }
        System.out.println(res);
    }
```

#### 递推公式对某个数是否能整除

&emsp;&emsp;多半是有套路的，比如x%y==z?  

#### 完全平方数
&emsp;&emsp;给定正整数 n，找到若干个完全平方数（比如 1, 4, 9, 16, ...）使得它们的和等于 n。你需要让组成和的完全平方数的个数最少。  
&emsp;&emsp;涉及四平方和定理  
```
class Solution {
    public int numSquares(int n) {
        while(n%4 == 0)
            n/=4;
        if(n%8 == 7)
            return 4;
        for(int i=0; i*i<=n; i++) {
            int j = (int)Math.sqrt(n- i*i);
            if(i*i + j*j==n){
                if(i>0&&j>0)
                    return 2;
                if(i>0||j>0)
                    return 1;
            }
        }
        return 3;
    }
}
```

#### 平方数之和
&emsp;&emsp;给定一个非负整数c，你要判断是否存在两个整数a和b，使得 a*a + b*b = c  
&emsp;&emsp;单循环，左右坐标试探加减即可  

```
class Solution {
    public boolean judgeSquareSum(int c) {
        int max = (int)Math.sqrt(c);
        if(max*max==c)
            return true;
        int min = 0;
        while(min<=max) {
            int tmp = min*min+max*max;
            if(tmp==c)
                return true;
            else if(tmp>c)
                max--;
            else
                min++;
        }
        return false;
    }
}
```

## 日期问题

#### 给定年月日判断是星期几  
```
1.公元一年一月一日为星期一(现在世界各国通用一星期七天的制度。这个制度最早由君士坦丁大帝[Constantine the Great]制定。他在公元321年3月7日正式宣布7天为1周，这个制度一直沿用至今)。
2.算今天到公元一年一月一日有多少天，%7，一周7天，周而复始。
3.每年365天，365=52*7+1，所以，过一年，在算星期的时候，就相当于多了一天。
4.闰年多一天。过一个闰年，在3月及以后就要多加一天。
5.公元一年各月一日的星期：t0[] = {1, 4, 4, 0, 2, 5, 0, 3, 6, 1, 4, 6}。
6.本质：日期+月修正+年修正+闰年修正，模7，得到星期几。
7.我们看上面的常数组t[]，其实就是将t0对应的1、2月-1，其后的-2。这个-2，因为是相对公元一年一月一日，所以y-1、m-1和d-1，其中m-1体现在下标中，y和d合起来就是-2了。前两个月-1，是因为在算闰年的时候，将1、2月的y多减了1，在t中补上。即，t[]不仅仅是月份修正常数，而是一个年月日的综合修正常数。
8.以上这个代码，用常数组隐藏了一些算法细节，使得代码变得相当的帅。
```
```
// 由Tomohiko Sakamoto提供
    public static int dayofweek(int y, int m, int d) {
        int[] t = {0, 3, 2, 5, 0, 3, 5, 1, 4, 6, 2, 4};
        if(m < 3)
            y--;
        return (y + y/4 - y/100 + y/400 + t[m-1] + d) % 7;
    }
```

## 递推问题

#### 全错排公式

```
F(1) = 0;F(2) = 1; F(n) = (n-1)*(F(n-2) + F(n-1))//错排公式为F(n)=n!(1/2!-1/3!+…..+(-1)^n/n!)
```

#### 部分错排公式(新郎问题：n个人中m个错误的情况数量)

```
D(m, n) = F(m) * C(m, n);其中F(m)是全错排数量,C(m, n)是组合数格式为C(up, low)
```

## 数论问题

#### 组合数、排列数求解模板及基本公式变形

![](https://ws1.sinaimg.cn/large/005L0VzSgy1ftwvodb6eej30sg0hzwf6.jpg)  

&emsp;&emsp;其他性质、定理传送门

- 1.https://blog.csdn.net/litble/article/details/75913032
- 2.https://blog.csdn.net/w1y2s312138/article/details/70478078
- 3.https://blog.csdn.net/littlewhite520/article/details/71551123

```
    // 排列数
    public static int A(int up,int bellow) {
        int result=1;
        for(int i=up;i>0;i--) {
            result*=bellow;
            bellow--;
        }
        return result;
    }
    // 阶乘
    public static long jie(long n) {
        long ans = 1;
        for(int i=1; i<=n; i++)
            ans *= i;
        return ans;
    }
    // 组合数 C(up, low)
    public static long C(long m,long n) {
        return jie(n) / (jie(n-m)*jie(m));
    }
```

## 字符串问题

#### 一些基本操作

```
    去除空格:s.replaceAll(" ","");
    大小写转换:s.toLowerCase();s.toUpperCase();
    提取字母&数字:s.replaceAll("[^a-z^A-Z^0-9]", "");
    字符串逆序:new StringBuffer(s).reverse().toString();
    字符大小写转换:大->小 (char)(c+32);小->大 (char)(c-32);
```

#### 无重复字符的最长子串长度

&emsp;&emsp;算法运行一遍即可了解过程  

```
public class Main {

    public static void main(String[] args) throws Exception{
        Scanner scanner = new Scanner(System.in);
        String d = scanner.nextLine();
        System.out.println(fun(d));
    }

    public static int fun(String s){
        if(s == null || s.length() < 1)
            return 0;
        HashSet<Character> set = new HashSet<Character>();
        int left = 0;
        int max_str = Integer.MIN_VALUE;
        int right = 0;
        int len = s.length();
        while(right < len) {
            if(set.contains(s.charAt(right))) {
                if(right - left > max_str)
                    max_str = right - left;
                while(s.charAt(left) != s.charAt(right)) {
                    set.remove(s.charAt(left));
                    left++;
                }
                for(Character c: set){
                    System.out.print(c+" ");
                }
                System.out.println();
                left++;
                System.out.println(left + " -> " + right);
            }
            else {
                set.add(s.charAt(right));
                for(Character c: set){
                    System.out.print(c+" ");
                }
                System.out.println();
                System.out.println(left + " -> " + right);
            }
            right++;
        }
        max_str = Math.max(max_str, right - left);
        return max_str;
    }
}
```

#### 子串(indexOf)手动实现

&emsp;&emsp;https://leetcode-cn.com/problems/implement-strstr/description/  

```
class Solution {
    public int strStr(String haystack, String needle) {
        if(needle==null||needle.length()==0)
            return 0;    
        for(int i=0; i<haystack.length()-needle.length()+1; i++) {
            boolean flag = true;
            for(int j=0; j<needle.length(); j++) {
                if(haystack.charAt(i+j)!=needle.charAt(j)) {
                    flag = false;
                    break;
                }
            }
            if(flag==true)
                return i;
        }
        return -1;
    }
}
```

## 图论问题

#### 遍历矩阵的每个点的路径长度

&emsp;&emsp;https://www.nowcoder.com/questionTerminal/2c9e3a1f8a2a487ba399af97781bd0cb  

![](https://ws1.sinaimg.cn/large/005L0VzSgy1fty0an21p0j30ix0kg779.jpg)  


## 朋友圈、观众分类、水坑、迷宫搜索类问题

&emsp;&emsp;https://leetcode-cn.com/problems/friend-circles/description/  

![](https://ws1.sinaimg.cn/large/005L0VzSgy1fv4r3gha09j30m00fhdgh.jpg)  

&emsp;&emsp;这种问题本质上可以用DFS/BFS解决，通常给定一个二维矩阵，我们可以利用一个一维矩阵visited[]进行记录。  

```
// 朋友圈问题的两种解法,观众、球迷分类本质上也是朋友圈的变种
class Solution {
    public int findCircleNum(int[][] M) {
        int[] visited = new int[M.length];
        int res = 0;
        for(int i=0; i<M.length; i++) {
            if(visited[i]==0) {
                opt(M, visited, i);
                res++;
            }
        }
        return res;
    }
    
    public static void opt(int[][] M, int[] visited, int i) {
        for(int j=0; j<M.length; j++) {
            if(M[i][j]==1 && visited[j]==0) {
                visited[j] = 1;
                opt(M, visited, j);
            }
        }
        return ;
    }
}

class Solution {
    public static Queue<Integer> q = new LinkedList<>();
    public int findCircleNum(int[][] M) {
        
        int[] visited = new int[M.length];
        int res = 0;
        for(int i=0; i<M.length; i++) {
            if(visited[i]==0) {
                opt(M, visited, i);
                res++;
            }
        }
        return res;
    }
    
    public static void opt(int[][] M, int[] visited, int i) {
        q.offer(i);
        visited[i] = 1;
        while (!q.isEmpty()) {
            int node = q.poll();
            for(int j=0; j<M.length; j++) {
                if(visited[j]==0 && M[node][j]==1) {
                    q.offer(j);
                    visited[j] = 1;
                }
            }
        }
    }
}
```

&emsp;&emsp;水坑问题(DFS)：

![](https://ws1.sinaimg.cn/large/005L0VzSgy1fv4rg2bzhzj30k60fwwfs.jpg)  

&emsp;&emsp;迷宫最短路径问题(BFS):  

![](https://ws1.sinaimg.cn/large/005L0VzSgy1fv4rkf6nehj30jx09g3zf.jpg)  
![](https://ws1.sinaimg.cn/large/005L0VzSgy1fv4rkz5x11j30jw0hbq5a.jpg)  

&emsp;&emsp;岛屿问题(DFS):https://leetcode-cn.com/problems/number-of-islands/description/  
```
class Solution {
    char[][] data;
    public int numIslands(char[][] grid) {
        int res = 0;
        data = grid;
        for(int i=0; i<grid.length; i++) {
            for(int j=0; j<grid[0].length; j++) {
                if(grid[i][j]=='1') {
                    dfs(i, j);
                    res++;
                }
            }
        }
        return res;
    }
    public void dfs(int x, int y) {
        
        data[x][y] = '0';
        for(int i=0; i<2; i++) {
            for(int j=-1; j<3; j+=2) {
                int nx = x;
                int ny = y;
                if(i==0) {
                    nx += j;
                }
                else{
                    ny += j;
                }
                
                if(nx>=0 && ny>=0 && nx<data.length && ny<data[0].length && data[nx][ny]=='1'){
                    dfs(nx, ny);
                }
            }
        }
        return;
    }
}
```
&emsp;&emsp;岛屿问题改(求最大岛屿面积)(DFS):https://leetcode-cn.com/problems/max-area-of-island/description/  

```
class Solution {
    int opt = 0;
    public int maxAreaOfIsland(int[][] grid) {
        int res = 0;
        
        for(int i=0; i<grid.length; i++) {
            for(int j=0; j<grid[0].length; j++) {
                if(grid[i][j]==1) {
                    dfs(grid, i, j);
                    res = Math.max(opt, res);
                    opt = 0;
                }
            }
        }
        return res;
    }
    
    public void dfs(int[][] grid, int x, int y) {
        grid[x][y] = 0;
        opt++;
        for(int i=0; i<2; i++) {
            for(int j=-1; j<3; j+=2) {
                int nx = x;
                int ny = y;
                if(i==0) {
                    nx += j;
                }
                else{
                    ny += j;
                }
                if(nx>=0 && ny>=0 && nx<grid.length && ny<grid[0].length && grid[nx][ny]==1) {
                    dfs(grid, nx, ny);
                }
            }
        }
    }
}
```
## DP问题

&emsp;&emsp;移步参考链接：https://github.com/zx950519/zx950519.github.io/blob/master/_posts/2018-05-22-%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E6%80%BB%E7%BB%93.md