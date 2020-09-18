#### 给定一个数组data，是否可以从中取出n个数，使他们的和为sum？如果是，请求出这n个数

```javascript
/**
 * 动态规划适用范围：最优子结构 && 无后效性 && 重复子问题
 */

/**
 * 1.基本的背包问题：
 * 在满足背包最大重量限制的前提下，求背包中物品总重量的最大值
 */

/**
 * 
 * @param {物品重量}      weight 
 * @param {物品个数}      n 
 * @param {背包可承载重量} w 
 * states[i][j] (0 < i < n, 0 < j < w + 1)
 * 表示第i个物品决策完之后 背包内的总重量为j时 该种情况发生的可能性为 states[i][j]
 */
// function knapsack(weight, n, w){
//   states = new Array(n);
//   for(let i = 0; i < states.length; ++i){
//     states[i] = new Array(w + 1).fill(false);
//   }
//   states[0][0] = true;  // 初始化第一行，第 0 个物品不放入背包
//   states[0][weight[0]] = true;  // 初始化第一行，第 0 个物品放入背包
//   for(let i = 1; i < n; ++i){  // 依次遍历每个物品
//     for(let j = 0; j <= w; ++j){  // 第 i 个物品不放入背包
//       if(states[i-1][j]) states[i][j] = states[i-1][j];
//     }
//     // w - weight[i],避免放入 i 后背包超重
//     for(let j = 0; j <= w - weight[i]; ++j){  // 第 i 个物品放入背包
//       if(states[i - 1][j]) states[i][j + weight[i]] = true;
//     }
//   }
//   console.log (states);
//   for(let i = w; i >= 0; --i){  // 找出满足条件的最大值 
//     if(states[n-1][i]) return i
//   }
  
//   return 0;
// }

// let wei = [2,2,4,6,3]
// console.log (knapsack(wei, wei.length, 9));

// 2.优化版，一维数组，降低空间复杂度
// function knapsack2(weight, n, w){
//   states = new Array(w + 1).fill(false);
//   states[0] = true;  // 初始化第一行，第 0 个物品不放入背包
//   states[weight[0]] = true;  // 初始化第一行，第 0 个物品放入背包
//   for(let i = 1; i < n; ++i){  // 依次遍历每个物品
//     // w - weight[i],避免放入 i 后背包超重
//     // 仅用一维数组即可，因为不放入的情况不必考虑，第i个物品不放入背包就是处理完第i-1个物品后的情况
//     for(let j = w - weight[i]; j >= 0; --j){  // 第 i 个物品放入背包
//       if(states[j]) states[j + weight[i]] = true;
//     }
//   }

//   for(let i = w; i >= 0; --i){  // 找出满足条件的最大值 
//     if(states[i]) return i;
//   }
//   return 0;
// }
// let wei2 = [2,2,4,6,3]
// console.log (knapsack2(wei2, wei2.length, 9));

// 3.背包问题延伸，求无序数组中是否有n个数的和等于sum
function knapsack3(weight, n, sum){
  states = new Array(n);
  for(let i = 0; i < states.length; ++i){
    states[i] = new Array(sum + 1).fill(false);
  }
  states[0][0] = true;  // 初始化第一行，第 0 个物品不放入背包
  states[0][weight[0]] = true;  // 初始化第一行，第 0 个物品放入背包
  for(let i = 1; i < n; ++i){  // 依次遍历每个物品
    for(let j = 0; j <= sum; ++j){  // 第 i 个物品不放入背包
      if(states[i-1][j]) states[i][j] = states[i-1][j];
    }
    // w - weight[i],避免放入 i 后背包超重
    for(let j = 0; j <= sum - weight[i]; ++j){  // 第 i 个物品放入背包
      if(states[i - 1][j]) states[i][j + weight[i]] = true;
    }
  }

  // 打印所选物品
  let j = sum;
  if(states[n - 1][sum]){
    for(let i = n - 1; i >= 0; --i){
      if(states[i][j - weight[i]]){
        console.log (i + " is in the knapsack");
        j -= weight[i];
      }
    }
  }else{
    console.log ("not exists!");
  }
}

let wei3 = [1,5,4,3];
console.log (knapsack3(wei3, wei3.length, 8));
```