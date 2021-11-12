---
title: "DFS求集合所有子集"
date: 2021-11-10T11:27:21+08:00
draft: false
---
对应代码中的数组b，空集为空行

![dfs集合子集](https://img-blog.csdnimg.cn/20200215023541807.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYxOTA4NQ==,size_16,color_FFFFFF,t_70)

```java
import java.util.ArrayList;
/**
 * DFS求数组的全部子集
 */
public class Try001 {
    //a为原数组，b为子集，n计数
    public static void dfs(ArrayList<Integer> a, ArrayList<Integer> b, int n){
        if(n == a.size()) { //终止条件
            //输出子集的值，空行代表空集
            for (Integer integer : b)
                System.out.print(integer + " ");
            System.out.println();//换行
            return;
        }
        //选择添加
        b.add(a.get(n));
        dfs(a,b,n+1);
        //不选择添加
        b.remove(a.get(n));
        dfs(a,b,n+1);
    }
    public static void main(String[] args) {
        ArrayList<Integer> a = new ArrayList<>();
        ArrayList<Integer> b = new ArrayList<>();
        for(int i = 1; i <= 3; i++){
            a.add(i);//初始化集合
        }
        dfs(a, b, 0);
    }
}
```
