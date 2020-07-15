[toc]
# Math

```
  
public class Demo{
    public static void main(String args[]){  
        /** 
         *Math.sqrt()//计算平方根
         *Math.cbrt()//计算立方根
         *Math.pow(a, b)//计算a的b次方
         *Math.max( , );//计算最大值
         *Math.min( , );//计算最小值
         */  

        System.out.println(Math.sqrt(16));   //4.0 
        System.out.println(Math.cbrt(8));    //2.0
        System.out.println(Math.pow(3,2));     //9.0
        System.out.println(Math.max(2.3,4.5));//4.5
        System.out.println(Math.min(2.3,4.5));//2.3

        /** 
         * abs求绝对值 
         */  
        System.out.println(Math.abs(-10.4));    //10.4  
        System.out.println(Math.abs(10.1));     //10.1  

        /** 
         * ceil天花板的意思，就是返回大的值
         */  
        System.out.println(Math.ceil(-10.1));   //-10.0  
        System.out.println(Math.ceil(10.7));    //11.0  
        System.out.println(Math.ceil(-0.7));    //-0.0  
        System.out.println(Math.ceil(0.0));     //0.0  
        System.out.println(Math.ceil(-0.0));    //-0.0  
        System.out.println(Math.ceil(-1.7));    //-1.0

        /** 
         * floor地板的意思，就是返回小的值 
         */  
        System.out.println(Math.floor(-10.1));  //-11.0  
        System.out.println(Math.floor(10.7));   //10.0  
        System.out.println(Math.floor(-0.7));   //-1.0  
        System.out.println(Math.floor(0.0));    //0.0  
        System.out.println(Math.floor(-0.0));   //-0.0  

        /** 
         * random 取得一个大于或者等于0.0小于不等于1.0的随机数 
         */  
        System.out.println(Math.random());  //小于1大于0的double类型的数
        System.out.println(Math.random()*2);//大于0小于1的double类型的数
        System.out.println(Math.random()*2+1);//大于1小于2的double类型的数

        /** 
         * rint 四舍五入，返回double值 
         * 注意.5的时候会取偶数    异常的尴尬=。=
         */  
        System.out.println(Math.rint(10.1));    //10.0  
        System.out.println(Math.rint(10.7));    //11.0  
        System.out.println(Math.rint(11.5));    //12.0  
        System.out.println(Math.rint(10.5));    //10.0  
        System.out.println(Math.rint(10.51));   //11.0  
        System.out.println(Math.rint(-10.5));   //-10.0  
        System.out.println(Math.rint(-11.5));   //-12.0  
        System.out.println(Math.rint(-10.51));  //-11.0  
        System.out.println(Math.rint(-10.6));   //-11.0  
        System.out.println(Math.rint(-10.2));   //-10.0  

        /** 
         * round 四舍五入，float时返回int值，double时返回long值 
         */  
        System.out.println(Math.round(10.1));   //10  
        System.out.println(Math.round(10.7));   //11  
        System.out.println(Math.round(10.5));   //11  
        System.out.println(Math.round(10.51));  //11  
        System.out.println(Math.round(-10.5));  //-10  
        System.out.println(Math.round(-10.51)); //-11  
        System.out.println(Math.round(-10.6));  //-11  
        System.out.println(Math.round(-10.2));  //-10  
    }  
}
        
```
# 随机数

###  利用System.currentTimeMillis()获取随机数
获取当前时间毫秒数，它是long类型。

```
final long l = System.currentTimeMillis();
final int i = (int)( l % 100 );
```


### 利用Math.random()获取随机数
它返回的是0(包含)到1(不包含)之间的double值

```
final double d = Math.random();
final int i = (int)(d*100);
```




### 利用Random类来获取随机数
 通过Random对象获取随机数。
 Random支持的随机值类型包括：boolean, byte, int, long, float, double。
 

```
Random random = new Random();//默认构造方法
 Random random = new Random(1000);//指定种子数字
 int i2 = random.nextInt(100);//获取[0, 100)之间的int整数
```


### 代码
```
/**
 * java 的随机数测试程序。共3种获取随机数的方法：
 *   (01)、通过System.currentTimeMillis()来获取一个当前时间毫秒数的long型数字。
 *   (02)、通过Math.random()返回一个0到1之间的double值。
 *   (03)、通过Random类来产生一个随机数，这个是专业的Random工具类，功能强大。
 *
 * @author skywang
 * @email kuiwu-wang@163.com
 */
public class RandomTest{

    public static void main(String args[]){

        // 通过System的currentTimeMillis()返回随机数
        testSystemTimeMillis();

        // 通过Math的random()返回随机数
        testMathRandom();

        // 新建“种子为1000”的Random对象，并通过该种子去测试Random的API
        testRandomAPIs(new Random(1000), " 1st Random(1000)");
        testRandomAPIs(new Random(1000), " 2nd Random(1000)");
        // 新建“默认种子”的Random对象，并通过该种子去测试Random的API
        testRandomAPIs(new Random(), " 1st Random()");
        testRandomAPIs(new Random(), " 2nd Random()");
    }

    /**
     * 返回随机数-01：测试System的currentTimeMillis()
     */
    private static void testSystemTimeMillis() {
        // 通过
        final long l = System.currentTimeMillis();
        // 通过l获取一个[0, 100)之间的整数
        final int i = (int)( l % 100 );

        System.out.printf("\n---- System.currentTimeMillis() ----\n l=%s i=%s\n", l, i);
    }


    /**
     * 返回随机数-02：测试Math的random()
     */
    private static void testMathRandom() {
        // 通过Math的random()函数返回一个double类型随机数，范围[0.0, 1.0)
        final double d = Math.random();
        // 通过d获取一个[0, 100)之间的整数
        final int i = (int)(d*100);

        System.out.printf("\n---- Math.random() ----\n d=%s i=%s\n", d, i);
    }


    /**
     * 返回随机数-03：测试Random的API
     */
    private static void testRandomAPIs(Random random, String title) {
        final int BUFFER_LEN = 5;

        // 获取随机的boolean值
        boolean b = random.nextBoolean();
        // 获取随机的数组buf[]
        byte[] buf = new byte[BUFFER_LEN];
        random.nextBytes(buf);
        // 获取随机的Double值，范围[0.0, 1.0)
        double d = random.nextDouble();
        // 获取随机的float值，范围[0.0, 1.0)
        float f = random.nextFloat();
        // 获取随机的int值
        int i1 = random.nextInt();
        // 获取随机的[0,100)之间的int值
        int i2 = random.nextInt(100);
        // 获取随机的高斯分布的double值
        double g = random.nextGaussian();
        // 获取随机的long值
        long l = random.nextLong();

        System.out.printf("\n---- %s ----\nb=%s, d=%s, f=%s, i1=%s, i2=%s, g=%s, l=%s, buf=[",
                title, b, d, f, i1, i2, g, l);
        for (byte bt:buf) 
            System.out.printf("%s, ", bt);
        System.out.println("]");
    }
}
```


