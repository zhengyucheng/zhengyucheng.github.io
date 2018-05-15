---
layout:     post
title:      "Miller Rabin"
subtitle:   "大质数判定"
date:       2018-05-13
author:     "JU"
header-img: "img/post-bg-2015.jpg"
mathjax:    false
catalog:    true
tags:
    - 质数
    - 数论
---

要判定一个数是不是质数，最妇孺皆知的方法莫过于O（sqrt(n)）的方法了（相信大家看见时间复杂度已经知道方法了……）。然而还有基于随机的算法：  
### 费马小定理
首先要知道的是费马小定理：  
当p是质数时，**a^(p-1)≡1(mod p)　(gcd(a,p)=1)**  
## 运用
实际上呢，这条东东可以写成：**a^(p-1)%p=1**  
不幸的是，当上面成立时，**p并不一定是质数**……  
然而，令人欣慰的是，如果上面**不成立，p一定是合数**。  
于是，那么每次要判断`p`是否为质数，只需随机一个`a`（可能是为了方便，`1<a<p`），判断`p`是否为合数。每次判断错误地将合数判断为质数的概率约为25%，但只要判断多几次（如20次），错判的概率就会很小很小，小到看不见了……  
	
	typedef long long ll;
	bool miller_rabin (ll p)
	{
		if (p<=1) return 0;
		if (p==2) return 1;
		int s=20;
		while (s--)
		{
			ll a=rand()%(p-2)+2;
			if (!pow_mod (a,p-1,p)) return 0;
		}
		return 1;
	}
	
### 快速幂
标题都说了，是硕质数，因此可能超过`int`，因此在求多次方时，要使用快速幂。  
同时，快速幂中两个数相乘时，也要使用类似的分治思想，避免溢出。

	ll mul (ll a,ll b,ll m)  //a*b%m
	{
		ll ans=0;
		while (b)
		{
			if (b&1) ans=(ans+a)%m;
			a=(a+a)%m;
			b>>=1;
		}
		return ans;
	}
	bool pow_mod (ll a,ll b,ll m)  //a^b%m
	{
		ll ans=1;
		while (b)
		{
			if (b&1) ans=mul (ans,a,m);
			a=mul (a,a,m);
			b>>=1;
		}
		if (ans!=1) return 0;
		return 1;
	}
	
#### 时间复杂度
O(k*log³n)  
(k为判断轮数)
