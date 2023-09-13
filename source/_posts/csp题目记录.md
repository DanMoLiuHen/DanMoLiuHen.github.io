---
title: csp做题记录
categories: learning-notes
excerpt: 由于csp题目答案比较难找到，所以特此记录拿到过满分的题目
hide: false
math: true
tag: C++
---
是我蠢了，这玩意官方有[解析](https://dl.ccf.org.cn/index.html?_ack=1)

## 注意事项
以下注意事项均针对csp中选择Dev-cpp(C++语言)条件
1. 使用`pow()`函数需要加上`cmath`库否则编译错误，与在vs中仅添加`iostream`有区别
2. 二维数组声明时注意空格如`vector<vector<int> >`，写为`>>`出现编译错误
3. 选择C++环境不支持auto智能指针

## 202209-1如此编码
根据提示完成即可，比较简单
```C++
#include<iostream>
#include<vector>
#include <fstream>
using namespace std;

int main() {
	ifstream cin("aaa.txt");
	int n, m;
	cin >> n >> m;
	vector<int>a(n+1);
	for (int i = 1; i <= n; i++)
		cin >> a[i];
	vector<int>c(n+1);
	vector<int>b(n+1);
	c[0] = 1;
	for (int i = 1; i <= n; i++) {
		c[i] = c[i - 1] * a[i];
		b[i] = (m % c[i] - m % c[i - 1]) / c[i - 1];
	}
	for (int i = 1; i <= n; i++)
		cout << b[i] << " ";
	return 0;
}
```

## 202209-2何以包邮
如果遍历所有的可能的取法，会有$2^{n}$种，运算量比较大，会有超时，只能拿下70分
```C++
#include<iostream>
#include<vector>
#include<cmath>
using namespace std;

int main() {
	int n, x;
	cin >> n >> x;
	vector<int> a(n);
	for (int i = 0; i < n; i++)
		cin >> a[i];
	//遍历
	int res = INT_MAX;
	for (int i = pow(2, n); i > 0; i--) {
		int sum = 0;
		for (int j = 0; j < n && (i >> j); j++) {
			if ((i >> j) & 1) {
				sum += a[j];
			}
		}
		res = sum >= x ? min(sum, res) : res;
	}
	cout << res;
}
```
考虑dfs+剪枝优化，同样是遍历，但通过计算节点后续能拿到的最大金额是否大于x，能够剪枝
```C++

```
但感觉还能用dp快速求解，类似于01背包，相当于把01背包问题的容量看做价格，用dp[j]表示价格不超过j的这n本书能组成的最大值，那么`dp[j]=max(dp[j],dp[j-a[i]]+a[i])`，该解法满分
```C++
#include<vector>
#include <iostream>
using namespace std; 

int main() {
	int n, x,sum = 0;//sum记录书本总价
	cin >> n >> x;
	vector<int>a(n);//记录书本价格
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		sum += a[i];
	}
	//构建sum+1个空间，如果x<sum那么符合条件的价格会出现在dp[x]之后所以不能仅仅构建x+1个空间，如果x=sum那么dp[x]就是符合条件的答案
	vector<int>dp(sum+1);
	//对于每本书都需要重新修正dp的每一项
	for (int i = 0; i < n; i++) {
		for (int j = sum; j >= a[i]; j--) {
			if (j - a[i] >= 0)
				dp[j] = max(dp[j], dp[j - a[i]] + a[i]);
		}
	}
	// 不必从开始查找，分析见上述注释
	for (int i = x; i <sum+1; i++)
	{
		if (dp[i] > x) {
			cout << dp[i];
			return 0;
		}
	}
}
```