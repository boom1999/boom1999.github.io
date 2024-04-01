---
title: 腾讯2024暑期实习笔试
date: 2024-04-01
tags: 
    - LinkedList
    - UndirectedGraph
    - BackTrack
categories: 笔试
copyright: true
---

1. 打卡题目，无向图的邻接表，遍历判断
2. 判断分段拼接链表后，是否整段升序
3. 连通图，非连通的图增加一条边恰好连通的方案数
4. 异或前缀和+动态规划
5. 遍历+回溯判断矩阵构成字符串

<!--more-->

----------

## 1. 题目描述

小红拿到了一个无向图，其中一些边被染成了红色。
小红定义一个点是“好点”，当且仅当这个点的所有邻边都是红边。
现在请你求出这个无向图“好点”的数量。
注：如果一个节点没有任何邻边，那么它也是好点。

### 1.1 输入描述

第一行输入两个正整数n,m，代表节点的数量和边的数量。
接下来的m行，每行输入两个正整数u,v和一个字符chr，代表节点u和节点v有一条边连接
如果 chr 为'R'，代表这条边被染红；'W'代表未被染色。

### 1.2 输出描述

$
1\leq n,m \leq 10^5
1\leq u,v \leq n
$

### 1.3 输入样例

#### 1.3.1 输入

``` java
4 4
1 2 R
2 3 W
3 4 W
1 4 R
```

#### 1.3.2 输出

一个整数，代表“好点”的数量。

``` java
1 //说明：只有 1 号节点是好点。
```

### 1.4 解题思路

- 用一个无向图`UndirectedGraph`包含节点数量`V`和一个`Edge`泛型的`LinkedList`，遍历输入的边通过`addEdge(int src, int dest, boolean colored)`在无向图中同时分别存入起始点`src`和终点`dest`的边，并且标记是否`colored`
- 通过`countGood()`遍历无向图的`LinkedList`，判断以当前节点为起点或终点的边是否有颜色，同时默认值是`true`（即无邻边），都无颜色或无邻边则返回`true`, 即题意的好点
- Attention：`boolean colored = in.next().equals("R")?true:false;`，区分`==`和`.equals()`

``` java
import java.util.*;

// 注意类名必须为 Main, 不要有任何 package xxx 信息
public class Main {
    public static void main(String[] args) {
        try (Scanner in = new Scanner(System.in)) {
            // 注意 hasNext 和 hasNextLine 的区别
            // 节点数量
            int n = in.nextInt();
            // 边的数量
            int m = in.nextInt();
            //  创建无向图
            UndirectedGraph graph = new UndirectedGraph(n);
            for (int i = 0; i < m; i++) {
                int src = in.nextInt();
                int dest = in.nextInt();
                // 注意这里要用 equals 方法，不能用 == 比较，因为 next 返回的是 String 对象，== 比较的是引用
                boolean colored = in.next().equals("R")?true:false;
                // 添加边，注意节点编号从 1 开始，要减 1
                graph.addEdge(src-1, dest-1, colored);
            }
            int goodCount = graph.countGood();
            System.out.print(goodCount);
        }
    }
}

class Edge {
    int dest;
    boolean colored;
    public Edge(int dest, boolean colored) {
        this.dest = dest;
        this.colored = colored;
    }
}

class UndirectedGraph {
    int V;
    LinkedList<Edge>[] adjListArray;

    public UndirectedGraph(int V) {
        this.V = V;
        this.adjListArray = new LinkedList[V];
        for (int i = 0; i < V; i++) {
            adjListArray[i] = new LinkedList<Edge>();
        }
    }

    void addEdge(int src, int dest, boolean colored) {
        Edge edge1 = new Edge(dest, colored);
        adjListArray[src].add(edge1);

        Edge edge2 = new Edge(src, colored);
        adjListArray[dest].add(edge2);
    }

    int countGood(){
        int count = 0;
        for(int v = 0; v < V; v++){
            // 没有边的节点也是好节点
            boolean isGood = true;
            for(Edge edge: adjListArray[v]){
                if(!edge.colored){
                    isGood = false;
                    break;
                }
            }
            if(isGood) count++;
        }
        return count;
    }
}
```

## 2. 题目描述

> 小红拿到了一个链表。她准备将这个链表断裂成两个链表，再拼接到一起，使得链表从头节点到尾部升序。你能帮小红判断能否达成目的吗？
> 给定的为一个链表数组，你需要对于数组中每个链表进行一次“是”或者“否”的答案回答，并返回布尔数组。
> 每个链表的长度不小于 2，且每个链表中不包含两个相等的元素。所有链表的长度之和保证不超过10^5

### 2.1 输入样例

#### 2.1.1 输入

``` java
[{1,2,3},{2,3,1},{3,2,1}]
```

#### 2.1.2 输出

``` java
[true,true,false]  // 说明：第三个链表无论怎么操作都不满足条件。
```

### 2.2 解题思路

- 断裂成两端，重新拼接后从头到尾是升序。两种情况：本来就是一段单调增，不断开也升序；两段单调增区间，且不能有重叠区间例如[0, 4], [3, 6]，这样实际上有一段属于递减
- 用`ListNode findMaxNode(ListNode head)`找到最大节点`maxNode`，`ListNode rightStart = maxNode.next;`找到第二段起始节点，如果`maxNode`在末尾，则为第一种情况单段单调增，符合题意；判断从`rightStart`到结尾是否都是单调增，如果是单调增并且`end.val<=head.val`，则符合题意

``` java
import java.util.*;

/*
 * public class ListNode {
 *   int val;
 *   ListNode next = null;
 *   public ListNode(int val) {
 *     this.val = val;
 *   }
 * }
 */

public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     *
     * @param lists ListNode类一维数组
     * @return bool布尔型一维数组
     */
    public boolean[] canSorted (ListNode[] lists) {
        // 最大节点左边是升序，最大节点右边是升序；最大节点左边的值0
        int len = lists.length;
        boolean[] ans = new boolean[len];
        for (int i = 0; i < len; i++) {
            ans[i] = canConcat(lists[i]);
        }
        return ans;
    }

    private static ListNode findMaxNode(ListNode head) {
        ListNode maxNode = head;
        ListNode current = head;
        while (current != null) {
            if (current.val > maxNode.val) {
                maxNode = current;
            }
            current = current.next;
        }
        return maxNode;
    }

    private static ListNode findEndOfUp(ListNode start, ListNode end) {
        ListNode current = start;
        // 不用判断最后一个，因为已经没有了下一个节点了，此时的current已经是end
        while (current != null && current.next != end) {
            if (current.val > current.next.val)
                return current;
            current = current.next;
        }
        return current;
    }

    private static boolean isUpping(ListNode start, ListNode end) {
        ListNode current = start;
        while (current != end && current.next!= end) {
            if (current.val > current.next.val) {
                return false;
            }
            current = current.next;
        }
        return true;
    }

    private static boolean canConcat(ListNode head){
        //  找到最大节点，是第一个升序序列的最后一个节点
        ListNode maxNode = findMaxNode(head);
        ListNode rightStart = maxNode.next;
        // 如果最大节点是最后一个节点即rightStart是null，那么整个链表是升序的
        if(rightStart == null)
            return true;
        ListNode rightEnd = findEndOfUp(rightStart, null);
        // 确保右段序列是升序到结尾的；以及右段的最后一个小于左段的最小值
        // return isUpping(head,leftEnd) && isUpping(rightStart,null) && (rightStart.val<head.val);
        // return isUpping(rightStart,null) && (rightStart.val<head.val);3
        return isUpping(rightStart,null) && (rightEnd.val<head.val);

    }
}
```

## 3. 题目描述

小红拿到了一个有 n 个节点的无向图，这个图初始并不是连通图。
现在小红想知道，添加恰好一条边使得这个图连通，有多少种不同的加边方案？

### 3.1 输入描述

第一行输入两个正整数n，m，用空格隔开。
接下来的m行，每行输入两个正整数u，v，代表节点u和节点v之间有一条边连接。
$1\leq n,m \leq 10^5$
$1\leq u,v \leq n$
保证给出的图是不连通的。

### 3.2 输出描述

一个整数，代表加边的方案数。

### 3.3 输入样例1

#### 3.3.1 输入

``` java
4 2
1 2
3 4
```

#### 3.3.2 输出

``` java
4
```

> 说明：添加边 (1,3) 或者 (1,4) 或者 (2,3) 或者 (2,4) 都是可以的。

### 3.4 输入样例2

#### 3.4.1 输入

``` java
4 1
1 3
```

#### 3.4.2 输出

``` java
0
```

### 3.5 解题思路

- DFS深度优先搜索+连通图概念
- 先用`List<List<Integer>>`保存各邻边的节点信息
- 再用DFS从第一个结点开始搜索，遍历过的全用`boolean[] visited`标记，记录遍历过的节点数，直到DFS停止，记录当前连通块内含有的节点数；遍历下一个连通块
- *两个连通块，返回各块数量相乘；两个以上连通块，不能用增加一条边的方式连接，返回0*

> 说明：无论添加哪条边，都不可能使得该图变成连通图。

``` java
import java.util.*;

// 注意类名必须为 Main, 不要有任何 package xxx 信息
public class Main {
    static List<List<Integer>> graph  = new ArrayList<>();

    // 算有几个连通块：两个连通块，返回各块数量相乘；两个以上连通块，不能用增加一条边的方式连接，返回0
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        // 节点个数
        int n = in.nextInt();
        // 边的个数
        int m = in.nextInt();
        //  创建邻接表
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            // 初始化邻接表
            graph.add(new ArrayList<>());
        }
        for (int i = 0; i < m; i++) {
            int src = in.nextInt();
            int dest = in.nextInt();
            graph.get(src-1).add(dest-1);
            graph.get(dest-1).add(src-1);
        }
        // 计算连通块中节点的数量  
        List<Integer> connectedComponentSizes = calculateConnectedComponentSizes(graph);  
  
        // 结果  
        int result = 0;
        if (connectedComponentSizes.size() == 2) {  
            result=connectedComponentSizes.get(0) * connectedComponentSizes.get(1);  
        }   
        System.out.println(result); 
    }
    
    
    private static List<Integer> calculateConnectedComponentSizes(List<List<Integer>> graph) {
        // 记录各节点是否被访问过，默认为 false
        boolean[] visited = new boolean[graph.size()];  
        List<Integer> connectedComponentSizes = new ArrayList<>();  
        for (int i = 0; i < graph.size(); i++) {
            // 如果节点没有被访问过，进行深度优先搜索
            if (!visited[i]) {
                // 计算连通块中节点的数量，加入到结果中
                connectedComponentSizes.add(dfs(i, visited, graph));
            }  
        }  
        return connectedComponentSizes;  
    }  


    private static int dfs(int node, boolean[] visited, List<List<Integer>> graph) {
        // 标记当前节点已经被访问
        visited[node] = true;
        // 当前连通块的节点数量为1
        int size = 1;
        // 遍历当前节点的边
        for (int neighbor : graph.get(node)) {
            // 如果邻接节点没有被访问过，递归访问
            if (!visited[neighbor]) {  
                size += dfs(neighbor, visited, graph);  
            }  
        }  
        return size;  
    }
}
```

## 4. 题目描述

小红拿到了一个数组，她准备将数组分割成 k 段，使得每段内部做按位异或后，再全部求和。小红希望最终这个和尽可能大，你能帮帮她吗？

### 4.1 输入描述

输入包含两行。
第一行两个正整数 $n, k , (1\leq k \leq n \leq 400)$，分别表示数组的长度和要分的段数。
第二行 n 个整数 $a_i (0 \leq a_i \leq 10^9)$，表示数组 a 的元素。

### 4.2 输出描述

输出一个正整数，表示最终的最大和。

### 4.3 输入样例1

#### 4.3.1 输入

``` java
6 2
1 1 1 2 3 4
```

#### 4.3.2 输出

``` java
10
```

> 说明
> 小红将数组分为了：
> [1, 4] 和 [5, 6] 这两个区间，得分分别为：$1 \oplus 1 \oplus 1 \oplus 2 = 3 $和 $3 \oplus 4 = 7$。总得分为 3+7=10。
> 可以证明不存在比 10 更优的分割方案。
> 注：\oplus 符号表示异或操作。

### 4.4 输入样例2

#### 4.4.1 输入

``` java
5 3
1 0 1 1 0
```

#### 4.4.2 输出

``` java
3
```

### 4.5 输入样例3

#### 4.5.1 输入

``` java
3 3
1 1 2
```

#### 4.5.2 输出

``` java
4
```

### 4.6 解题思路

- 动态规划，递推公式：`dp[i][j] = max(dp[l-1][j-1] + preXor[l][i]) 1<=l<=i`
- 当i和j都已经确定的时候，遍历当前分段点，前`l`个点分`j-i`段+后续数组组成一段的最大值
- `preXor[i][j]`是从当前点`i`到点`j`一直进行异或运算
- DP初始化：每次都从最小开始，`dp[i][j] = Integer.MIN_VALUE;`

``` java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int[] nums = new int[n];
        int k = in.nextInt();
        for(int i = 0; i < n; i++) 
            nums[i] = in.nextInt();
        int[][] dp = new int[n+1][k+1];
        int[][] preXor = new int[n+1][n+1];
        // preXor[i][j]表示从i到j的异或结果
        for(int i = 1; i < n+1; i++){
            preXor[i][i] =nums[i-1];
            for(int j = i+1; j < n+1; j++){
                preXor[i][j] = preXor[i][j-1] ^ nums[j-1];
            }
        }
        // dp[i][j]表示前i个数分成j段的最大异或和
        // 递推公式：dp[i][j] = max(dp[l-1][j-1] + preXor[l][i]) 1<=l<=i
        //  l表示分段点，dp[l-1][j-1]表示前l-1个数分成j-1段的最大异或和
        //  preXor[l][i]表示从l到i的异或结果
        for(int i = 1; i < n+1; i++){
            for(int j = 1; j < k+1; j++){
                dp[i][j] = Integer.MIN_VALUE;
                // 遍历所有可能的分段点
                for(int l = 1; l < i+1; l++){
                    dp[i][j] = Math.max(dp[i][j], dp[l-1][j-1] + preXor[l][i]);
                }
            }
        }
        System.out.println(dp[n][k]);
    }
}
```

## 5. 题目描述

小红拿到了一个字符矩阵，她可以从任意一个地方出发，希望走 6 步后恰好形成"tencent"字符串。小红想知道，共有多少种不同的行走方案？
> 注：每一步可以选择上、下、左、右中任意一个方向进行行走。不可行走到矩阵外部。

### 5.1 输入描述

第一行输入两个正整数n,m，代表矩阵的行数和列数。
接下来的n行，每行输入一个长度为m的、仅由小写字母组成的字符串，代表小红拿到的矩阵。
$ 1\leq n,m \leq 1000 $

### 5.2 输出描述

一个整数，代表最终合法的方案数。

### 5.3 输入样例

#### 5.3.1 输入

``` java
3 3
ten
nec
ten
```

#### 5.3.2 输出

``` java
4
```

> 说明：
> 第一个方案，从左上角出发，右右下左左上。
> 第二个方案，从左上角出发，右右下左左下。
> 第三个方案，从左下角出发，右右上左左下。
> 第四个方案，从左上角出发，右右上左左上。

### 5.4 解题思路

- 先存到二维数组：`map[i] = in.next().toCharArray();`
- 静态变量`int[][] directions`用于定义上下左右移动
- 遍历二维数组，也就是从每个点开始，拼接当前字符，再对四个方向进行遍历回溯
- 达到长度判定是否等于`"tencent"`，是则加一不是则返回
- attention：要设置临时变量用于恢复字符串，每个方向返回后都需要恢复
- 考虑剪枝：第一个字符是`t`等

``` java
import java.util.Scanner;

// 注意类名必须为 Main, 不要有任何 package xxx 信息
public class Main {
    private static int[][] directions = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}};
    private static int result = 0;
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        char[][] map = new char[n][m];
        for(int i = 0; i < n; i++){
            map[i] = in.next().toCharArray();
        }
        for(int i = 0; i < n; i++){
            for(int j = 0; j < m; j++){
                backTrack(i, j, map, String.valueOf(map[i][j]));
            }
        }
        System.out.print(result);
    }

    private static void backTrack(int i, int j, char[][] map, String str){
        if (str.length()==7) {
            if (str.equals("tencent")) {
                result++;
            }
            return;
        }
        for(int[] direction : directions){
            int newI = i + direction[0];
            int newJ = j + direction[1];
            if(newI >= 0 && newI < map.length && newJ >= 0 && newJ < map[0].length){
                // 要暂存当前的 str, 递归返回后要恢复
                String temp = str;
                str += map[newI][newJ];
                backTrack(newI, newJ, map, str);
                str = temp;
            }
        }
    }
}
```
