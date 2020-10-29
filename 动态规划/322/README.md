## 零钱兑换(coin-change)
> https://leetcode-cn.com/problems/coin-change/

核心不变, 优先理清思路, 然后解决问题, 实现 暴力递归 -> 带备忘录的递归 -> `dp` 数组 这样一个解题过程。

当熟练之后可以跳过带存储的递归, 直接使用 `dp` 数组解决。

### 思路
先看下题目: 给你 `k` 种面值的硬币, 面值分别为 `c1`, `c2` ... `ck`, 每种硬币的数量无限, 再给一个总金额 `amount`, 问你最少需要几枚硬币凑出这个金额, 如果不可能凑出, 算法返回 `-1` 。算法的函数签名如下: 
```js
/**
 * @param {number[]} coins
 * @param {number} amount
 * @return {number}
 */
var coinChange = function (coins, amount) {}
```

通过审题发现, 最基础的实现思路就是通过穷举所有可能性(用所有硬币不同的组合分别尝试)。

然后又由于问题有要求`最少需要几枚硬币`, 所以需要得到`最优子结构`, 最优性原理实际上是要求问题的最优策略的子策略也是最优。

然后通过绘制二叉树发现, 依旧存在`重叠子问题`。

由此判断, 这是一个典型的`动态规划问题`(注意与分治法的区别)。

### 暴力递归
既然知道了问题类型, 最核心的就是找到这个暴力递归的运算方法, 也就是`状态转移方程`。

#### 状态
状态表示每个阶段开始面临的自然状况或客观条件。这里是每次需要计算的总金额 `amount` 与每个分节点上的总金额。

假设完成 `k` 目标的硬币序列为 `a1`, `a2`, ..., `ak`。  
原问题: `k`。  
子问题: `k - a1`, 以此类推。

#### 转移方程(决策)
原题硬币数组为 `[1, 2, 5]`, 由于需要最优解, 所以有 `f(k) = Math.min(f(k - 1) + 1, f(k - 2) + 1, f(k - 5) + 1)`。

#### 边界
原题已给出, 无答案返回 `-1`, 不需要答案时返回 `0`。

#### 代码
```js
var coinChange = function (coins, amount) {
    // 状态转移方程
    function f(amount) {
        if(amount < 0 ) return -1; // 需要的结果 < 0 时, 不存在结果, 直接返回
        if(amount === 0) return 0; // 当需要计算的结果恰好为0时, 返回0

        let res = Infinity; // 初始化结果为 Infinity

        // 递归所有给定的硬币
        for(const coin of coins) {
            // 必须使用当前目标金额 - 硬币面值的结果
            // 不能使用目标金额 - 硬币面值的形式进行判断, 因为也有概率返回 -1, 不能用于计算
            const temp = f(amount - coin); // <-- 开始套娃, 也就是优化点
            // 当不存在结果时, 不进行后续计算, 跳过
            if(temp === -1) continue;

            // 获取当前 res 与计算结果的最小值
            // 理解为什么 +1
            res = Math.min(res, temp + 1);
        }
        // 返回最终结果
        return isFinite(res) ? res : -1;
    }

    return f(amount);
}
```

### 带备忘录实现
```js
var coinChange = function (coins, amount) {
    // 创建并填充所有数组项
    const memorandum = new Array(amount + 1).fill(undefined);
    memorandum[0] = 0;

    // 状态转移方程
    function f(amount) {
        // 存在结果直接返回, 初步优化, 在备忘录中查找
        if(memorandum[amount] !== undefined) return memorandum[amount];
        if(amount < 0 ) return -1; // 需要的结果 < 0 时, 不存在结果, 直接返回
        if(amount === 0) return 0; // 当需要计算的结果恰好为0时, 返回0

        let res = Infinity; // 初始化结果为 Infinity

        // 递归所有给定的硬币
        for(const coin of coins) {
            // 必须使用当前目标金额 - 硬币面值的结果
            // 不能使用目标金额 - 硬币面值的形式进行判断, 因为也有概率返回 -1, 不能用于计算
            const temp = f(amount - coin); // <-- 经过初步优化, 可以继续优化
            // 当不存在结果时, 不进行后续计算, 跳过
            if(temp === -1) continue;

            // 获取当前 res 与计算结果的最小值
            // 理解为什么 +1
            res = Math.min(res, temp + 1);
        }
        // 结果存入备忘录
        memorandum[amount] = isFinite(res) ? res : -1;
        // 返回最终结果
        return memorandum[amount];
    }

    return f(amount);
}
```

### dp 数组
```js
var coinChange = function (coins, amount) {
    // 创建并填充所有数组项
    const dp = new Array(amount + 1).fill(Infinity);
    dp[0] = 0;

    if(amount < 0 ) return -1; // 需要的结果 < 0 时, 不存在结果, 直接返回

    for(let i = 1; i <= amount; i++) {
        for(const coin of coins) {
            // i 为 当前目标金额
            // coin 为 当前拾取的零钱面值
            // 当两项差值小于 0, 不进行后续计算
            if(i - coin < 0) continue;

            // 将较小的数值放入 dp 数组
            dp[i] = Math.min(dp[i], dp[i - coin] + 1);
        }
    }

    // 从 dp 数组中拾取结果
    const res = dp[amount];
    // 返回最终结果
    return isFinite(res) ? res : -1;
}
```