---
title: '接口comparable和comparator的区别'
date: "2017/10/19"
tags: [java]
categories: ['java']
copyright: true
---
comparable只提供了compareTo方法，可以当做是一个内比较器。
comparator提供了compare和equals方法，外比较器。
如果开发者add进入一个Collection的对象想要Collections的sort方法帮你自动进行排序的话，那么这个对象必须实现Comparable接口。
下面通过讲解最常见的sort：
对一个自己编写的对象ChatLog实现sort的步骤（以前都是套用的这个流程并没有完全理解）：
（1）ChatLog实现Comparable并Override compareTo方法，在compareTo中具体实现比较过程。

（2）为ChatLog实现接口Comparator，在compare方法简单比较大小。

（3）执行Collections.sort(List<ChatLog>, comparator)完成排序。

这样的一个过程，我们还可以去掉第（3）中的第二个参数，进而不要第二步。
实现Comparator接口后，如果要修改排序的实现就只需要改Comparator不用该ChatLog对象了，如需要升序改成降序，把Comparator的compare中实现的前后对象换一下位置就可以实现了。
其实实现Comparator接口只是为了在外面包一层排序的外衣，可以省略。
第一步：
```java
public class ChatLog implements Comparable{
    String name = null;
    String time = null;
    String chat = null;

    public String getTime() {
        return time;
    }

    public void setTime(String time) {
        this.time = time;
    }

    public String getChat() {
        return chat;
    }

    public void setChat(String chat) {
        this.chat = chat;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public int compareTo(Object o){
        if(this == o){
            return 0;
        }else if(o!=null && o instanceof ChatLog){
            ChatLog chatLog = (ChatLog) o;
            if((time + name).compareTo(chatLog.time+chatLog.name) < 0){
                return -1;
            }else{
                return 1;
            }
        }else{
            return -1;
        }

    }
}
```
第二步：
```java
public static final Comparator<ChatLog> COMPARATOR = new Comparator<ChatLog>() {
    @Override
    public int compare(ChatLog o1, ChatLog o2) {
        return o1.compareTo(o2);
    }
};
```

第三步：
```java
//对chatlog列表进行排序
public List<ChatLog> sortMsg(List<ChatLog> allChat){
    Collections.sort(allChat, COMPARATOR);
    return allChat;
}
```
