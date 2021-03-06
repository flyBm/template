## 动态规划

[TOC]
### 枚举子集
C(n, k) k = 0 特殊处理
```cpp
void GospersHack(int k, int n) {
    int cur = (1 << k) - 1;
    int limit = (1 << n);
    while (cur < limit) {
        int lb = cur & -cur;
        int r = cur + lb;
        cur = (((r ^ cur) >> 2) / lb) | r;
    }
}
```
### 树型$DP$基础应用

#### 树上距离之和

```cpp
ll siz[N], son[N]; 
ll ans[N];
int n;
// son 子树的答案 ans 其他顶点到i的答案
void predfs(int node, int fa) {
    siz[node] = 1;
    son[node] = 0;
    for (auto to : v[node]) {
        if (to == fa) continue;
        predfs(to, node);
        siz[node] += siz[to];
        son[node] += siz[to] + son[to];
    }
}

void dfs(int node, int fa) {
    if (node == fa) {
        ans[node] = son[node];
    } else {
        ans[node] = (ans[fa] - son[node] - siz[node]) + (n - siz[node]) + son[node];
    }
    for (auto to : v[node]) {
        if (to == fa) continue;
        dfs(to, node);
    }
}
```

#### 最小支配集

求覆盖一个树所有的点，需要覆盖最小的次数，覆盖一个点之后其相邻的点也被覆盖。

```cpp
int dp[10050][3];
//0 表示被儿子覆盖 1 表示被自己覆盖 2 表示被父亲覆盖
//dp[node][1] = min(dp[to][2], dp[to][1], dp[to][0])
//dp[node][2] = min(dp[to][1], dp[to][0])
//dp[node][0] = min(dp[to][1], dp[to][0]) + res
vector<int> v[10050];
void dfs (int node, int fa) {
    dp[node][1] = 1;
    int res = 0x3f3f3f3f;
    bool flag = false;
    for (int to : v[node]) {
        if(to==fa)continue;
        dfs(to, node);
        dp[node][1] += min(dp[to][0], min(dp[to][1], dp[to][2]));
        dp[node][2] += min(dp[to][1], dp[to][0]);
        if(dp[to][1] <= dp[to][0] and not flag) res = 0, flag = true;
        if(dp[to][1] > dp[to][0] and not flag) res = min(res, dp[to][1] - dp[to][0]);
    }
    dp[node][0] = dp[node][2] + res;
}
```

#### 最大独立子集
最大独立子集: 对于一个树形结构 所有的孩子和他们的父亲存在排斥

也就是如果选取了某个节点 则不能选取这个节点的所有孩子节点。

```cpp
int f[N][2]; // 0 不选 1 选
void dfs (int node, int fa) {
    f[node][1] = h[node];
    for (int k : v[node]) {
        if(k == fa)continue;
        dfs(k, node);
        f[node][1] += f[k][0];
        f[node][0] += max(f[k][1], f[k][0]);
    }
}
```

#### 最小点覆盖

所有的边被看住 即对于一条边我选了父亲就不用选儿子

```cpp
int dp[1600][2]; // 0 不选 1 选
vector<int> v[1600];
void dfs(int node, int fa) {
    dp[node][1] = 1;
    for(auto to : v[node]){
        if(to == fa)continue;
        dfs(to, node);
        dp[node][1] += min(dp[to][0], dp[to][1]);
        dp[node][0] += dp[to][1];
    }
}
```
### 树上背包
```cpp
ll dp[N][N][2], w[N], siz[N];

void dfs (int node, int fa) {
    siz[node] = 1;
    dp[node][1][1] = w[node], dp[node][0][0] = 0;

    for (auto to : v[node]) {
        if (to == fa) continue;
        dfs(to, node);
        for (int j = siz[node]; j >= 0; --j) {
            for (int k = siz[to]; k >= 0; --k) {
                dp[node][j + k][0] = max(dp[node][j][0] + dp[to][k][0], dp[node][j + k][0]);
                if (k >= 1) dp[node][j + k][0] = max(dp[node][j + k][0], dp[node][j][0] + dp[to][k][1] + w[to]);
                if (k + j >= 1 and j >= 1) dp[node][j + k][1] = max(dp[node][j + k][1], dp[node][j][1] + dp[to][k][0] + w[to]);
                if (k + j >= 1 and j >= 1 and  k >= 1) dp[node][j + k][1] = max(dp[node][j + k][1], dp[node][j][1] + dp[to][k][1] + w[to]);
            }
        }
        siz[node] += siz[to];
    }
}
```
### 分组背包

```cpp
struct EulerSieve{
    static const int N = 3e4 + 50;
    int vis[N], prime[N];
    int cnt = 0;

    void getPrime () {
        vis[1] = 1;
        for(int i = 2; i < N; ++i) {
            if(!vis[i]) prime[++cnt] = i;
            for(int j = 1; j <= cnt; ++j) {
                if(i * prime[j] > N) break;
                vis[i * prime[j]] = 1;
                if(i % prime[j] == 0) break;
            }
        }
    }

} phi;

struct Groupbackpack {
    static const int N = 3e4 + 50;
    double dp[N];
    int num;

    void init() {
        phi.getPrime();
        for(int i = 1; i < N; ++i) ln[i] = log(i);
        for(int i = 0; i < N; ++i) dp[i] = 0;
        num = phi.cnt;
        getans();
    }

    // dp[i][j] = dp[i - 1][j - w] + log(k)
    void getans() {
        dp[1] = 0.0;
        for(int i = 1; i <= num; ++i) {
            for(int j = N - 1; j >= 1; --j) {// 先枚举j 再枚举k
                for(int k = phi.prime[i]; k < N; k *= phi.prime[i]) {
                    if(j < k) break;
                    dp[j] = max(dp[j], dp[j - k] + ln[k]);
                }
            }
        }
    }
}dp;
```

### 数位dp

```cpp
//接下来n行 每行一个数字x 接下来一个数len表示数字x在数字串中连续出现的次数不能大于len
const int N = 2e5 + 100;
const int mod = 20020219;
int f[20][15][20][2];
int limit[20];
int a[20];
ll dp(int pos, int num, int cnt, bool flag){
//pos 还剩下几位 num 当前数位上是什么数字 cnt 当前数字出现次数 flag 当前位是否有限制
    if(pos == 0) return cnt <= limit[num];
    if(cnt > limit[num]) return 0;
    if(flag && f[pos][num][cnt][flag] != -1) return f[pos][num][cnt][flag];
    int x = flag ? 9 : a[pos];
    ll ans = 0;
    for(int i = 0; i <= x; ++i) {
        if(i == num)ans = (ans + dp(pos - 1, num, cnt + 1, flag || i < x)) % mod;
        else ans = (ans + dp(pos - 1, i, 1, flag || i < x)) % mod;
    }
    ans = ans % mod;
    if(flag) f[pos][num][cnt][flag] = ans;
    return ans;
}
//计算每一位数的限制
ll calc(ll num){
    memset(f, -1, sizeof(f));
    int pos = 0;
    while(num){
        a[ ++pos ] = num % 10;
        num /= 10;
    }
    return dp(pos, 0, 0, 0);
}

void solve(){
    memset(limit, 0x3f, sizeof(limit));
    ll l = gl(), r = gl(), n = gl();
    for(int i = 1; i <= n; ++i) {
        int x = gn(), y = gn();
        limit[x] = min(limit[x], y);
    }
}
```

### 区间dp

```cpp
//区间dp
void solve(){
    memset(dp, 0x3f, sizeof(dp))；
    for(int i = 1; i <= n; ++i) {
        dp[i][i] = 0;
    }
    for(int l = 2; l <= n; ++l) {
        for(int i = 1; i + l - 1 <= n; ++i) {
            int j = i + l - 1;
            for(int k = i; k < j; ++k) {
                dp[i][j] = min(dp[i][j], dp[i][k] + dp[k + 1][j] + sum[j]-sum[i - 1]);
            }
        }
    }
}
//优化
int m[2005][2005] = {0};
int sum[1005] = {0};

void solver() {
    memset(dp, 0x7f, sizeof(dp));
    int n = gn();
    for(int i = 1; i <= n; ++i) {
        a[i] = gn();
        sum[i] = sum[i-1] + a[i];
    }
    for(int i = 1; i <= 2 * n; ++i) {
        dp[i][i] = 0;
        m[i][i] = i;
    }
    for(int l = 2;l <= n; ++l) {
        for(int i = 1; i + l - 1 <= 2 * n; ++i) {
            int j = i + l - 1;
            for(int k = m[i][j-1]; k <= m[i + 1][j]; ++k) {
                int a, b;
                if(i - 1 > n) {
                    a = sum[n] + sum[i - 1 - n];
                }
                else a = sum[i - 1];
                if(j > n) {
                    b = sum[n] + sum[j - n];
                } else b = sum[j];
                int t = b - a;
                if(dp[i][j] > dp[i][k] + dp[k + 1][j] + t) {
                    dp[i][j] = dp[i][k] + dp[k + 1][j] + t;
                    m[i][j] = k;
                }
            }
        }
    }
    int ans = 1e9 + 7;
    for(int i = 1; i <= n; ++i) {
        ans=min(ans, dp[i][i + n - 1]);
    }
    cout << ans << endl;
}
```

### 背包问题

```cpp
int n, m;
int N[555], V[555];
int dp[100000 + 1024];
//01背包
void ZeroOnePack(int cost, int weight){
    for(int i = m; i >= cost; --i) {//反向更新
        dp[i] = max(dp[i], dp[i - cost] + weight);
        if(dp[i - cost] + weight == dp[i])//统计方案数
            sum[i] += sum[i-cost];
    }
}
//完全背包
void CompletePack(int cost, int weight){
    for(int i = cost; i <= m; ++i) {
        dp[i] = max(dp[i], dp[i - cost] + weight);
    }
}
//多重背包
void MultiplePack(int cost, int weight, int amount) {
    if(cost * amount > m) {
        CompletePack(cost, weight);
        return ;
    }
    int k = 1;
    while(k < amount) {
        ZeroOnePack(k * cost, k * weight);
        amount -= k;
        k *= 2;
    }
    ZeroOnePack(amount * cost, amount * weight);
}

for(all group k)
    for(v = V....0)
        for(all i belong to group k)
            f[v] = max{f[v], f[v-c[i]]+w[i]}
```

### 最长公共子序列(O(nlogn))

```cpp
int dp[N];
int a[N], b[N];

void solve() {
    int n = gn();
    int k;
    for(int i = 1; i <= n; ++i) k = gn(),a[k] = i;
    for(int i = 1; i <= n; ++i) k = gn(),b[i] = a[k];
    int len = 0;
    fill(dp, dp + N, 0);
    for(int i = 1; i <= n; ++i) {
        if(b[i] > dp[len]){
            dp[++len]=b[i];
        }
        else{
            int l = 1,r = len;
            while(l <= r){
                int mid = (l + r) / 2;
                if(b[i] > dp[mid])l = mid + 1;
                else r = mid - 1;
            }
            dp[l]=min(b[i], dp[l]);
        }
    }
    cout<<len<<endl;
}
```
