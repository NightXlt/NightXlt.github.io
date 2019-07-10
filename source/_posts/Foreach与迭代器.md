title: Foreach与迭代器
date: 2016-01-04 00:00:00
tags: [Java]
categories: Thinking in Java
description: 神器的迭代器
---
## Foreach与迭代器

  emmmmmm，孤陋寡闻的博主在看TIJ之前只以为Foreach语句只可以用在数组，结果没想到可以应用到任意Collection对象。/捂脸

  瞅瞅Collection对象的Foreach肿么写哈......

```java
public class IterableClass implements Iterable<String> {  
  
    String[] words = ("Nice to meet you").split(" ");  
  
    public Iterator<String> iterator() {  
        return new Iterator<String>(){  
            private int index = 0;  
            public boolean hasNext() {  
                return index < words.length;  
            }  
  
            public String next() {  
                return words[index++];  
            }  
              
        };  
    }  
  public static void main(String[] args) {  
        for(String s : new IterableClass()){  
            System.out.print(s + " ");  
        }  
    }  
}  

```

  输出请自行脑补，不做另行说明....刚才是谁说我菜的站出来，说的就是你，死肥宅！！！


  从上述代码可以看出一个类要使用ForEach需要实现Iterable这个接口，实现iterator()方法，在方法中可以看到我返回了一个匿名内部类，操作的具体实现都在内部类里面了。它也就重写了hasNext()、next()方法在遍历时进行回调。


  emmmmm还有一个ForEach的非常规操作，一般人我都不告诉，看在阁下看你的骨骼精奇，是万中无一的武学奇才，见与你有缘。就告诉你罢。


  众所周知，ForEach都是顺序访问，那要逆序访问呢？show you the code......


```java
class ReverseArrayList<T> extends ArrayList<T> {

    public ReverseArrayList(@NotNull Collection<? extends T> c) {
        super(c);
    }

    public Iterable<T> reversed() {
        return new Iterable<T>() {
                      public Iterator<T> iterator() {
                return new Iterator<T>() {
                    int current = size() - 1;

                    public boolean hasNext () {
                        return current > -1;
                    }

                    public T next () {
                        return get(current--);
                    }
                };
            }
        };
    }

    public static void main(String[] args) {
        for (String s : new ReverseArrayList<String>(Arrays.asList("Nice to meet you !".split(" "))).reversed()) {
            System.out.println(s+"");
        }
    }
}
```


```java
output:
!
you
meet
to
Nice
```


  系不系很神奇呀，如果你细心的话，可以注意到我在reverse返回的是Iterable而不是Iterator,Iterable可以在默认迭代器的基础上，添加产生反向迭代器的能力，而Iterator只能实现现有的方法。在上述代码中，我自定义了一个reversed()逆序访问。那随机访问也类似......


```java
class IterClass extends IterableClass {
    public Iterable<String> randomize() {
        return new Iterable<String>() {
            @NotNull
            @Override
            public Iterator<String> iterator() {
                List<String> shuffle = new ArrayList<String>(Arrays.asList(words));
                Collections.shuffle(shuffle, new Random(47));
                return shuffle.iterator();
            }
        };
    }
    public static void main(String[] args){
        for (String s : new IterClass().randomize()) {
            System.out.println(s + "");
        }
    }
}
```

```java
output:
Nice
to
you
meet
```

  值得注意的是Collections.shuffle()并没有影响到原数组，只是打乱了shuffle的引用，所以在上述代码中new ArrayList<String>(Arrays.asList(words));通过创建一个list用以保存新的引用，防止更改原先的数组引用。