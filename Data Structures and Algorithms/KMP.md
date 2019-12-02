本蒟蒻经过一下午的挣扎终于能略微窥探到KMP算法的精妙，也是弥补了以前学数据结构的遗憾，这里记录一下蒟蒻自己理解的KMP算法，博客简略，主要是大概过一下，让自己以后能回忆起来，具体参考**《大话数据结构》**。

---

KMP的精妙之处就在于它的next数组，**next[i]**保存的是第$i$个数之前的字符串**前后缀相似度**，也可以理解为**上一个可匹配的前缀字符串下一个字符**。求next数组的算法核心就在于$j$的回溯，他可以回溯到之前的**最长可匹配前缀字串**，因为这样后缀字串就为对应的**最长可匹配后缀字串**，至于为什么手动模拟一遍$ababcabababc$就知道了（滑稽）。

上代码：

```c++
/*求子串T的next数组*/
void get_next(string T, int *next)
{
	int i = 1, j = 0;
	next[1] = 0;//守字母next赋值0
	while (i < T[0])//T[0]储存子串T长度
	{
		if (j == 0 || T[i] == T[j])
		{
			i++;
			j++;
			next[i] = j;
		}
		else
			j = next[j];//字符不相同，回溯
	}
}
```

求出了next数组，接下来的KMP就比较好理解了，只回溯$j$，不回溯$i$

```c++
int index_KMP(string S, string T, int pos)//返回匹配到的下标
{
	int i = pos;//pos为S主串开始匹配的位置
	int j = 1;
	int next[255];
	get_next(T, next);
	while (i <= S[0] && j <= T[0])//i，j小于两个字符串的大小
	{
		if (j == 0 || S[i] == T[j])//和传统匹配算法相比增加特判
		{
			i++;
			j++;
		}
		else
			j = next[j];
	}
	if (j > T[0])
		return i - T[0];//返回S中匹配字符串的开头位置
	else
		return 0;//匹配失败
}
```

