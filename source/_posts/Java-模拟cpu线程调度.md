---
title: Java 模拟cpu线程调度
date: 2016-03-16 17:41:53
tags: [Java]
categories: coding
---

### 写在前面
这篇是我早前写在csdn上的操作系统作业，如果你也有这个作业，可以作为参考，原来的博客这里就只字不改了，这也是种不错的感觉。这里贴出优先数算法的代码，在源码中附带了时间片轮转算法和段作业优先算法的算法。

### 博客原文
这本来是我的操作系统作业，斑竹本来想偷懒在网上找一篇交上去，但无奈没有找到符合的，只好自己写了。言归正传，在这个例子中，我实现了进程调度的三种算法，分别是优先级算法，时间片算法，和段作业优先算法（fcfs算法比较简单，这里就不做说明了，读者可以根据斑竹的思路自己写）。在写这个程序的时候，斑竹感觉也挺麻烦的，没有具体的思路，但第二天就要交了，没办法，斑竹在前一天晚上终于想通了关键。那就是，不论是哪一种算法，只要能列出在个时间段当前在运行的进程就好了，剩下的求响应时间，周转时间，甘特图什么的就一样了。
废话少说，这里贴出代码，如果心急的朋友，可以直接去下载[源码](http://download.csdn.net/detail/xiaowei1118/8261711)。

1. 读文件，从txt文件中读取json格式的进程。我的json数据如下：

```json
 [
        {
            "name": "P1",
            "startTime": 0,
            "runTime": 7,
            "priority": 5,
            "isOver":false
        },
        {
            "name": "P2",
            "startTime": 1,
            "runTime": 1,
            "priority": 1,
             "isOver":false
        },
        {
            "name": "P3",
            "startTime": 1,
            "runTime": 3,
            "priority": 4,
            "is_over":false
        },
        {
            "name": "P4",
            "startTime": 2,
            "runTime": 5,
            "priority": 3,
             "isOver":false
        },
        {
            "name": "P5",
            "startTime": 4,
            "runTime": 4,
            "priority": 2,
            "isOver":false
        }
    ]
```
主函数中读取文件的方法如下：
```java
private  static String loadProcess() {
  URL  xmlpath=MainRun.class.getClassLoader().getResource("");
  String encoding="utf-8";

  try {
    File file=new File(xmlpath.toString().replace("file:/", "")+"com/box/process/JOB1.txt");
    if(file.isFile() && file.exists()){
    InputStreamReader read = new InputStreamReader(
    new FileInputStream(file),encoding);
    BufferedReader bufferedReader = new BufferedReader(read);
    StringBuffer buffer=new StringBuffer();
    String lineTxt = null;
    while((lineTxt = bufferedReader.readLine()) != null){
    buffer.append(lineTxt);
    }
    String message=buffer.toString();
    read.close();
    return message;
    }else{
    System.out.println("打开文件失败");
    }
  } catch (Exception e) {
    System.out.println("加载文件失败");
    e.printStackTrace();
  }
  return null;
}
```
2. 优先数算法：
```java
private static void priority(List<PCB> pcbs) {
  List<String> queueList=new ArrayList<>();
  System.out.println("调度队列中每秒运行的进程如下：");
  for(int i=0;i<allTime;i++){
  PCB current=null;
  int k=0;
  for(int j=0;j<pcbs.size();j++){
    PCB pcb=pcbs.get(j);
    if(pcb.getIsOver()==false){
      if(pcb.getStartTime()<=i){
        if(current==null){
           current=pcb;
        }else{
        if (current.getPriority()>pcb.getPriority()){
          current=pcb;
          k=j;
        }
        }
      }
    }
  }
  pcbs.get(k).decrease();
  queueList.add(current.getName());
  System.out.print(current.getName());
  }

  int j=0;
  System.out.println("\n 优先算法甘特图如下：");
  System.out.print("0 "+queueList.get(j)+" ");
  for(int i=0;i<queueList.size();i++){
    if(!queueList.get(i).equals(queueList.get(j))){
      System.out.print(i+" "+queueList.get(i)+" ");
      j=i;
    }
  }

  System.out.print(" "+(queueList.size()));

  System.out.println("\n进程名:\t"+"等待时间:\t"+"周转时间:\t");
  for(int i=0;i<pcbs.size();i++){
    System.out.print(pcbs.get(i).getName()+"\t");
    for(int k=0;k<queueList.size();k++){
    if(pcbs.get(i).getName().equals(queueList.get(k))){
      System.out.print(k-pcbs.get(i).getStartTime());
      System.out.print("\t");
      break;
    }
  }

  for(int k=queueList.size()-1;k>=0;k--){
  if(pcbs.get(i).getName().equals(queueList.get(k))){
  System.out.print((k-pcbs.get(i).getStartTime())+1);
  System.out.print("\t");
  break;
  }
  }
  System.out.println();
  }
}
```

3. 进程的javabean
```java
import com.alibaba.fastjson.JSONObject;
public class PCB {
private int runTime;  //运行时间
    private  int priority;   //优先级
    private  String name; //进程名臣
    private boolean isOver; //是否运行结束
    private int startTime;  //开始运行时间
   //get,set 方法省略
    public void Json2Object(JSONObject jsonObject) {
        setName(jsonObject.getString("name"));
        setStartTime(jsonObject.getInteger("startTime"));
        setIsOver(jsonObject.getBooleanValue("isOver"));
        setPriority(jsonObject.getIntValue("priority"));
        setRunTime(jsonObject.getIntValue("runTime"));
    }
    public void decrease() {
        runTime--;
        if(runTime<=0){
        isOver=true;
        }
    }
```
我在工程中将json转化成object的时候用到了阿里巴巴的fastjson jar包，当然你们也可以用别的jar包。斑竹自知这个工程还有改进的地方，比如在得到所有时刻点的队列之后计算响应时间和周转时间，画甘特图等为了重用性封装成类，还有进程的list用队列可能更好一点（不过斑竹习惯了用arraylist），有需要的同学可以自己改进。希望批评指正。
