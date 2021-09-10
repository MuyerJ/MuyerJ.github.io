---
layout: post
title: HashMap位运算
tags: HashMap
categories: Java
---

### 一、HashMap的长度为啥是2的N次方

1、hash%length==hash&(length-1)的前提是length是2的n次方

2、hash%length，计算机中直接求余效率不如位移运算，源码中做了优化hash&(length-1)

3、两个性能差别对比
```

public static void main(String[] args) {
	
	long currentTimeMillis = System.currentTimeMillis();
	int a=0;
	int times = 10000*10000;
	for (long i = 0; i < times; i++) {
		 a=9999%1024;
	}
	long currentTimeMillis2 = System.currentTimeMillis();
	
	int b=0;
	for (long i = 0; i < times; i++) {
		 b=9999&(1024-1);
	}
	
	long currentTimeMillis3 = System.currentTimeMillis();
	System.out.println(a+","+b);
	System.out.println("%: "+(currentTimeMillis2-currentTimeMillis));
	System.out.println("&: "+(currentTimeMillis3-currentTimeMillis2));
}


结果：

783,783
%: 359
&: 93

```

### 二、hash(Object key)的原理

```

    /**
     * Computes key.hashCode() and spreads (XORs) higher bits of hash
     * to lower.  Because the table uses power-of-two masking, sets of
     * hashes that vary only in bits above the current mask will
     * always collide. (Among known examples are sets of Float keys
     * holding consecutive whole numbers in small tables.)  So we
     * apply a transform that spreads the impact of higher bits
     * downward. There is a tradeoff between speed, utility, and
     * quality of bit-spreading. Because many common sets of hashes
     * are already reasonably distributed (so don't benefit from
     * spreading), and because we use trees to handle large sets of
     * collisions in bins, we just XOR some shifted bits in the
     * cheapest possible way to reduce systematic lossage, as well as
     * to incorporate impact of the highest bits that would otherwise
     * never be used in index calculations because of table bounds.
     */
   
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

```

1、对注释的解释

```
1) 通过hashCode()方法，计算得到key的hash值，并将近右移16位使其高16位与原hashcode进行异或（hash值本为32位的int类型）
    
2) 之所以需要右移16位，是因为散列表的大小都是2的阶乘，如果不将高位与低位进行异或将会造成持续的hash碰撞。

3) 原因在于当散列表足够小的时候，在计算存放位置的时候，参与比较的将只有低位，那么高位的hash码将会失去意义，导致大量碰撞产生

```

2、>>>:无符号右移，忽略符号位，空位都以0补齐

```

生成的新的hash值，其高位部分(左边16位蓝紫色部分)保留了原hashcode的高位，低位部分(红色部分)保留了原来的高位和低位的“特征”——如果原来高位部分某一位发生改变，则影响到结果的对应位；如果原来低位某一位发生改变，也同样影响到结果相应的位。

```


3、为什么要用异或操作？

```
使用“与”操作时，当一个数为“0”，则结果必然为“0”，不必考虑另一个操作数；
使用“或”操作时，当一个操作数为“1”，则结果必然为“1”；
使用“异或”操作时，需要知道两个操作数才能决定结果。

```

4、用>>>和^生成的新hashcode的每一位都对最终的取模都产生影响，这使得生产的余数更加均衡，更加散列，碰撞几率更小