所有子问题规模相同的递归才能使用master公式：
$$T_n=a\times T_{\frac{n}{b}}+O(n^c)$$
其中a,b,c都是常数

1. 如果$\log_{b}{a}<c$，复杂度为$O(n^c)$
2. 如果$\log_{b}{a}>c$，复杂度为$O(n^{\log_{b}{a}})$
3. 如果$\log_{b}{a}=c$，复杂度为$O(n^c\times logn)$

若$T_n=2\times T_{\frac{n}{2}}+O(n\times logn)$，则复杂度为$O(n\times (logn)^2)$