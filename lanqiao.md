## 2022

### 填空

#### 2186 2022

思路：将选择 10 个数抽象成背包问题：`dp[i][j][k]` 表示前 i 个数中选择 j 个总和为 k 的选法。

#### 2218 数数

统计质因数个数方法：

```cpp
#include <bits/stdc++.h>
using namespace std;
const int L = 2333333;
const int R = 23333333;
int f[R + 10], ans;
vector<int> vec;
int main()
{
  for(int i = 2; i <= R; ++i) {
    if(!f[i]) {
      vec.push_back(i);
      f[i] = 1;
    }
    if(i >= L && f[i] == 12) ans++;
    for(int x : vec) {
      if(x * i > R) break;
      f[x * i] = f[i] + 1;
    }
  }
  cout << ans << endl;
  return 0;
}
```

