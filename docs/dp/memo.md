# 聊聊动态规划与记忆化搜索

by $\\color{Gray}InterestingLSY$ (菜到发灰)

## 1. 记忆化搜索是啥

好，就以 [这道题](https://www.luogu.org/problemnew/show/P1048) 为例，我不会动态规划，只会搜索，我就会直接写一个粗暴的 [DFS](/search/dfs) :

- 注: 为了方便食用, 本文中所有代码省略头文件 \*

```cpp
int n,t;
int tcost[103],mget[103];
int ans = 0;
void dfs( int pos , int tleft , int tans ){
    if( tleft < 0 ) return;
    if( pos == n+1 ){
		ans = max(ans,tans);
		return;
	}
	dfs(pos+1,tleft,tans);
    dfs(pos+1,tleft-tcost[pos],tans+mget[pos]);
}
int main(){
    cin >> t >> n;
    for(int i = 1;i <= n;i++)
        cin >> tcost[i] >> mget[i];
    dfs(1,t,0);
	cout << ans << endl;
    return 0;
}
```

这就是个十分智障的大暴搜是吧......

emmmmmm....... $\\color{Red}30$ 分

然后我心血来潮, 想不借助任何 "外部变量"(就是 dfs 函数外且 ** 值随 dfs 运行而改变的变量 **), 比如 ans

把 ans 删了之后就有一个问题: 我们拿什么来记录答案?

答案很简单:

** 返回值!**

此时 $dfs(pos,tleft)$ 返回在时间 $tleft$ 内采集 ** 后 **$pos$ 个草药, 能获得的最大收益

不理解就看看代码吧:

```cpp
int n,time;
int tcost[103],mget[103];
int dfs(int pos,int tleft){
    if(pos == n+1)
        return 0;
    int dfs1,dfs2 = -INF;
    dfs1 = dfs(pos+1,tleft);
	if( tleft >= tcost[pos] )
    	dfs2 = dfs(pos+1,tleft-tcost[pos]) + mget[pos];
    return max(dfs1,dfs2);
}
int main(){
    cin >> time >> n;
    for(int i = 1;i <= n;i++)
        cin >> tcost[i] >> mget[i];
    cout << dfs(1,time) << endl;
    return 0;
}
```

~~emmmmmm....... 还是 $\\color{Red}30$ 分~~

但这个时候, 我们的程序已经不依赖任何外部变量了.

然后我非常无聊, 将所有 dfs 的返回值都记录下来, 竟然发现......

** 震惊, 对于相同的 pos 和 tleft,dfs 的返回值总是相同的!**

想一想也不奇怪, 因为我们的 dfs 没有依赖任何外部变量.

旁白: 像 $tcost[103]$,$mget[103]$ 这种东西不算是外部变量, 因为她们在 dfs 过程中不变.

然后?

开个数组 $mem$ , 记录下来每个 $dfs(pos,tleft)$ 的返回值. 刚开始把 $mem$ 中每个值都设成 $-1$ (代表没访问过). 每次刚刚进入一个 dfs 前 (我们的 dfs 是递归调用的嘛), 都检测 $mem[pos][tleft]$ 是否为 $-1$ , 如果是就正常执行并把答案记录到 $mem$ 中, 否则?

** 直接返回 $mem$ 中的值!**

```cpp
int n,t;
int tcost[103],mget[103];
int mem[103][1003];
int dfs(int pos,int tleft){
	if( mem[pos][tleft] != -1 ) return mem[pos][tleft];
    if(pos == n+1)
        return mem[pos][tleft] = 0;
    int dfs1,dfs2 = -INF;
    dfs1 = dfs(pos+1,tleft);
	if( tleft >= tcost[pos] )
    	dfs2 = dfs(pos+1,tleft-tcost[pos]) + mget[pos];
    return mem[pos][tleft] = max(dfs1,dfs2);
}
int main(){
	memset(mem,-1,sizeof(mem));
    cin >> t >> n;
    for(int i = 1;i <= n;i++)
        cin >> tcost[i] >> mget[i];
    cout << dfs(1,t) << endl;
    return 0;
}
```

此时 $mem$ 的意义与 dfs 相同:

> 在时间 $tleft$ 内采集 **后** $pos$ 个草药, 能获得的最大收益

这能 ac?

能.** 这就是 "采药" 那题的 AC 代码 **

好我们 yy 出了记忆化搜索

#### 总结一下记忆化搜索是啥:

- 不依赖任何 ** 外部变量 **

- 答案以返回值的形式存在, 而不能以参数的形式存在 (就是不能将 dfs 定义成 $dfs(pos ,tleft , nowans )$, 这里面的 nowans 不符合要求.

- 对于相同一组参数, dfs 返回值总是相同的

## 2. 记忆化搜索与动态规划的关系:

\~~ 基本是朋 (ji) 友关系\~~

时间复杂度 / 空间复杂度与 ** 不加优化的 dp** 完全相同

不管定义咋扯, 反正我觉得

> 记忆化搜索就是动态规划,**(印象中) 任何一个 dp 方程都能转为记忆化搜索 **

比如:

$dp[i][j][k] = dp[i+1][j+1]\[k-a[j]] + dp[i+1][j][k]$

转为

```cpp
int dfs( int i , int j , int k ){
	边界条件
    if( mem[i][j][k] != -1 ) return mem[i][j][k];
    return mem[i][j][k] = dfs(i+1,j+1,k-a[j]) + dfs(i+1,j,k);
}
int main(){
	memset(mem,-1,sizeof(mem));
	读入
    cout << dfs(1,0,0) << endl;
}
```

$dp[i] = max{dp[j]+1}\\quad 1 \\leq j &lt; i \\text{且}a[j]&lt;a[i]$  (最长上升子序列)

转为

```cpp
int dfs( int i ){
	if( mem[i] != -1 ) return mem[i];
	int ret = 1;
	for( int j = 1 ; j < i ; j++ )
    	if( a[j] < a[i] )
    		ret = max(ret,dfs(j)+1);
    return mem[i] = ret;
}
int main(){
	memset(mem,-1,sizeof(mem));
	读入
    cout << dfs(n) << endl;
}
```

**当然, 以我的经验更多情况下记忆化搜索是写完暴力 dfs(本来想骗分) 后突然发现能改成记忆化搜索**

\~~ 然后 AC 了一道全场没几个人会的超难 dp\~~

感受以下那种发现自己写的暴力改改就是正解的快感吧!

啊哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈!

## 3. 记忆化搜索的优缺点

咳咳, 说正事

优点:

- 记忆化搜索可以避免搜到无用状态, 特别是在有状态压缩时

- 边界情况非常好处理, 且能有效防止数组访问越界

- \~~ 写起来简单易懂\~~ 至少我镇么认为 qwq

- 有些 dp(如区间 dp) 用记忆化搜索写很简单但正常 dp 很难

缺点:

- 致命伤: 不能滚动数组!(哪位 dalao 会记搜 + 滚动的请在评论区留名)

- 有些优化比较难加

- 由于递归, 有时效率较低但不至于 TLE

- 代码有点长\~~ 其实也不算太长\~~

## 4. 记忆化搜索的注意事项:

- 千万别忘了加记忆化! (别笑, 认真的

- 边界条件要加在检查当前数组值是否为 - 1 前 (防止越界)

- 数组不要开小了 (逃

## 如有疑问或质疑, 请留下评论或私信我

** questions are welcome **
