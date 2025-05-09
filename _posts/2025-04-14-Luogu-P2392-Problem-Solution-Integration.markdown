---
layout:     post
title:      "洛谷P2392题解整合"
date:       2025-04-25
author:     "ZZJ"
header-style: text
catalog: true
tags:
    - C++
    - 洛谷
    - 题解
---

## 题目

[点击跳题](https://www.luogu.com.cn/problem/P2392)

## 详细题解

### 方法一：递归回溯法
```cpp
/**
 * 核心思想：将每个科目的题目分配到左/右脑，求最小化最大耗时
 * 时间复杂度：O(2^n) 每个科目独立处理
 */
#include <bits/stdc++.h>
using namespace std;

int s[5], hw[5][61];  // s[1]-s[4]：四科知识点数，hw[i][j]：第i科第j知识点耗时
int minn, ans;        // minn：单科最优解，ans：四科最优解总和

// DFS枚举分组：l左组耗时，r右组耗时，id当前知识点，sub当前科目
void dfs(int l, int r, int id, int sub) {
    if (id > s[sub]) {                // 终止条件：所有知识点分配完毕
        minn = min(minn, max(l, r));  // 关键策略：取两组的最大耗时，更新最小最大耗时
        return;
    }
    // 核心决策：将当前知识点加入左组或右组，生成两种分支（类似01背包决策树）
    dfs(l + hw[sub][id], r, id + 1, sub);  // 左分支：当前知识点归左组
    dfs(l, r + hw[sub][id], id + 1, sub);  // 右分支：当前知识点归右组
}

int main() {
    // 输入四科知识点数量（注意题目输入顺序）
    for (int i = 1; i <= 4; i++) cin >> s[i];
    
    // 分层处理四科数据（各科独立处理）
    for (int i = 1; i <= 4; i++) {
        // 输入当前科目知识点耗时（按s[i]数量读取）
        for (int j = 1; j <= s[i]; j++) cin >> hw[i][j];
        
        minn = INT_MAX;        // 初始化极值（寻找最小值）
        dfs(0, 0, 1, i);       // 启动DFS（初始两组均为0耗时，从第一个知识点开始）
        ans += minn;           // 核心逻辑：各科最优解累加得总答案
    }
    
    cout << ans;  // 输出：四科最小总耗时之和
    return 0;
}
```

----------------------------

### 方法二：动态规划法（01背包变种）

```txt
思路解释：
想象你要把一堆书装进两个行李箱，要求最重的箱子尽可能轻。这里的：
- 每个科目 ➜ 一个行李箱套装
- 每道题 ➜ 一本书
- 做题耗时 ➜ 书的重量
```

```cpp
/*-- 核心思路 --*
1. 对每个科目单独处理，就像整理一套行李箱
2. 计算总重量，尝试将一半重量的书装入第一个箱子
3. 剩余的书自动进入第二个箱子
4. 取两个箱子中较重的作为当前科目耗时*/

#include<bits/stdc++.h>
using namespace std;

// 工具准备
int subject[5];     // 四科的题目数（下标1-4）
int totalTime;      // 四科总耗时（最终结果）
int problems[21];   // 当前科目各题耗时（最多20题）
int bag[2501];      // 模拟行李箱（容量2500分钟）

int main() {
    // 步骤1：读取各科题目数量
    for(int i=1; i<=4; i++) cin>>subject[i];
    
    // 分别处理四科目
    for(int i=1; i<=4; i++) {
        int sum = 0;  // 当前科目总耗时
        
        // 步骤2：读取题目耗时并计算总耗时
        for(int j=1; j<=subject[i]; j++) {
            cin >> problems[j];
            sum += problems[j]; // 累加所有书的总重量
        }
        
        // 步骤3：装箱模拟（核心逻辑）
        for(int j=1; j<=subject[i]; j++) 
            // 逆向装入防止重复（类似从行李箱底部开始放书）
            for(int k=sum/2; k>=problems[j]; k--) 
                // 决策：放入当前书后是否比原有装法更接近半重
                bag[k] = max(bag[k], bag[k-problems[j]] + problems[j]);
        
        // 步骤4：计算当前科目耗时
        totalTime += sum - bag[sum/2]; // 总重 - 较轻箱 = 较重箱耗时
        
        // 清空行李箱准备下一科目
        memset(bag, 0, sizeof(bag));
    }
    
    // 输出最终总耗时
    cout << totalTime;
    return 0;
}
```
```txt
生活案例说明:
假设处理数学科目（3道题耗时10、20、30分钟）：
1. 总耗时sum=60分钟，目标装30分钟的书到左脑
2. 最佳装法：10+20=30分钟
3. 剩余30分钟自动分到右脑
4. 取最大值30作为数学科目耗时
```
### 方法二：动态规划法（01背包变种）——假代码
```cpp
/*-- 核心思路 --*
用智能装箱策略找到最优分配*/
#include<bits/stdc++.h>
using namespace std;

int 题量[5];         // 各科题数
int 总耗时;           // 最终结果
int 单科题目[21];     // 当前科题目耗时
int 智能装箱表[2501]; // 装箱策略记录

int main() {
    // 第一步：获取各科题量
    for(int i=1; i<=4; i++) cin>>题量[i];
    
    // 分科处理
    for(int 当前科=1; 当前科<=4; 当前科++) {
        int 总重量 = 0;
        
        // 第二步：读题并计算总耗时
        for(int 题号=1; 题号<=题量[当前科]; 题号++) {
            cin >> 单科题目[题号];
            总重量 += 单科题目[题号]; // 累计所有书的重量
        }
        
        // 第三步：智能装箱（核心）
        for(int 题号=1; 题号<=题量[当前科]; 题号++) 
            // 从半重开始逆向装填（确保每本书只装一次）
            for(int 当前容量=总重量/2; 当前容量>=单科题目[题号]; 当前容量--) 
                // 比较两种选择：装 vs 不装当前书
                智能装箱表[当前容量] = max(
                    智能装箱表[当前容量], // 保持现状
                    智能装箱表[当前容量-单科题目[题号]] + 单科题目[题号] // 装入当前书
                );
        
        // 第四步：计算本科耗时
        总耗时 += 总重量 - 智能装箱表[总重量/2]; // 总重 - 轻背包 = 重背包耗时
        
        // 重置记录表准备下一科
        memset(智能装箱表, 0, sizeof(智能装箱表));
    }
    
    cout << 总耗时;
    return 0;
}
```
---------------------

## 方法对比

| 特征 | 递归回溯法  |  动态规划法  |
| ---- | ---------  |  --------   |
| 时间复杂度 | O(2ⁿ) 每科目 | O(n*sum) 每科目 |
| 空间复杂度 | O(n) 递归栈 | O(sum) 数组空间 |
| 适用数据规模 | n ≤ 20 (单科题目数) | sum ≤ 2500 (单科总耗时) |
| 核心思想 | 暴力枚举所有分组组合 | 转化为背包问题求最优解 |
| 代码复杂度 | 需要回溯逻辑 | 需要背包问题理解 |
| 实际运行效率 | 20题需处理约百万次操作 | 2500容量仅需数万次操作 |
| 优势场景 | 小规模数据直观实现 | 大规模数据高效处理 |
