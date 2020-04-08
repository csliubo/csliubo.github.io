---
title: Java语言规范中的 Example 17.5-2
date: 2020-04-02 10:38:45
tags: [java]
---

最近开始学习java中的多线程，在翻看jls(Java Language Specification)的时候发现了这个例子。
```java
//thread 1
Global.s = "/tmp/usr".substring(4); 
```

```java
// thread2
String myS = Global.s;
if (myS.equals("/tmp"))System.out.println(myS);
```

<!-- more -->

thread2 有可能输出 /usr ，具体的解释就是String的构造函数有个offset字段，而这个字段不是final，导致线程2有可能会看到构造到一半的myS。
```java
myS.equals("/tmp") //此时myS构造了一半,offset = 0，count=4，所以 myS 的值为 /tmp
System.out.println(myS); //myS构造完成，offset = 4，count = 4，myS的值为 /usr
```

如果你现在用的是jdk8或者jdk7的高版本，你看一下substring的实现，你会发现对应的String构造函数的成员变量只涉及到value，而且value是final修饰的。所以根本不会出现规范中所说的问题。
```java
public String substring(int beginIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        int subLen = value.length - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
    }

private final char value[];

public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= value.length) {
                this.value = "".value;
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }
```

搜索一下，[stackoverflow](https://stackoverflow.com/questions/6638819/why-the-following-program-can-work-under-jls-spec-on-jvm-memory-model/6638890) 上也有人提了这个问题。
里面回答里贴出来的[String代码](http://www.docjar.com/html/api/java/lang/String.java.html)有一个内部构造函数是有offset跟count的。
```java
  644       // Package private constructor which shares value array for speed.
  645       String(int offset, int count, char value[]) {
  646           this.value = value;
  647           this.offset = offset;
  648           this.count = count;
  649       }
```
而且这里的substring实现确实依赖了这个构造函数
```java
 1924       public String substring(int beginIndex) {
 1925           return substring(beginIndex, count);
 1926       }
 1927   
 1928       /**
 1929        * Returns a new string that is a substring of this string. The
 1930        * substring begins at the specified <code>beginIndex</code> and
 1931        * extends to the character at index <code>endIndex - 1</code>.
 1932        * Thus the length of the substring is <code>endIndex-beginIndex</code>.
 1933        * <p>
 1934        * Examples:
 1935        * <blockquote><pre>
 1936        * "hamburger".substring(4, 8) returns "urge"
 1937        * "smiles".substring(1, 5) returns "mile"
 1938        * </pre></blockquote>
 1939        *
 1940        * @param      beginIndex   the beginning index, inclusive.
 1941        * @param      endIndex     the ending index, exclusive.
 1942        * @return     the specified substring.
 1943        * @exception  IndexOutOfBoundsException  if the
 1944        *             <code>beginIndex</code> is negative, or
 1945        *             <code>endIndex</code> is larger than the length of
 1946        *             this <code>String</code> object, or
 1947        *             <code>beginIndex</code> is larger than
 1948        *             <code>endIndex</code>.
 1949        */
 1950       public String substring(int beginIndex, int endIndex) {
 1951           if (beginIndex < 0) {
 1952               throw new StringIndexOutOfBoundsException(beginIndex);
 1953           }
 1954           if (endIndex > count) {
 1955               throw new StringIndexOutOfBoundsException(endIndex);
 1956           }
 1957           if (beginIndex > endIndex) {
 1958               throw new StringIndexOutOfBoundsException(endIndex - beginIndex);
 1959           }
 1960           return ((beginIndex == 0) && (endIndex == count)) ? this :
 1961               new String(offset + beginIndex, endIndex - beginIndex, value);
 1962       }
```
再进一步看的话，你会发现offset跟count其实都是final修饰的，也就是这一段代码在多线程环境下也是不会出现规范里说的问题。
```java
  114       private final char value[];
  115   
  116       /** The offset is the first index of the storage that is used. */
  117       private final int offset;
  118   
  119       /** The count is the number of characters in the String. */
  120       private final int count;
```

当然，我认为jls这么举例，应该是之前的实现确实出现过这个问题，而最大的可能应该是1.5之前的版本。因为jsr133就是针对这些乱七八糟的情况提出来的嘛，并且在jdk1.5实现了。

于是，我去搜了一下jdk1.4的代码，我在github上找到[这个仓库](https://github.com/eagle518/jdk-source-code/)。
这个仓库里的jdk1.4里的[String实现](https://github.com/eagle518/jdk-source-code/blob/master/jdk1.4.2_src/java/lang/String.java)里有这个构造方法，并且offset,count和value都不是final修饰的。也就是说在jdk1.4下，确实会存在jls所说的这个问题。


```java

/** The value is used for character storage. */
    private char value[];

    /** The offset is the first index of the storage that is used. */
    private int offset;

    /** The count is the number of characters in the String. */
    private int count;

// Package private constructor which shares value array for speed.
    String(int offset, int count, char value[]) {
	this.value = value;
	this.offset = offset;
	this.count = count;
    }

```

ps. openjdk7里这个substring和String实现的修改是[这次提交](http://hg.openjdk.java.net/jdk7u/jdk7u/jdk/rev/e1c679a00712)变更的
