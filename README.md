# blog

```
graph TD;
a(start) -->b;
b[发起请求] -->c;
c[参数处理]-->d;
d[获取用户信息]-->e;
e{绑定手机?}-->| 是 |f;
e{绑定手机?}-->| 否 |f1;
f1[return error: 未绑定手机]-->z;
f{是否已报名}-->| 否 |g;
f{是否已报名}-->| 是 |h;
g[插入新记录]-->h;
h[retrun ok: ok]-->z(end);
```
