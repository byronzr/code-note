## RBAC with resource roles

### config rule

```toml
# 定义请求？
[request_definition]
r = sub, obj, act

# 定义规则
[policy_definition]
p = sub, obj, act

# 定义角色
[role_definition]
g = _, _
g2 = _, _

# 定义多条匹配处理规则
[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && g2(r.obj, p.obj) && r.act == p.act
```

### config data

```toml
# 描述角色与资源(分类的UUID可有效去重)的关系
p, m1, cate_uuid1, read
p, m2, cate_uuid2, read
p, m3, cate_uuid3, read
#p, m1, cate_uuid1, write
p, m2, cate_uuid2, write
p, m3, cate_uuid3, write

# 描述角色与操作路径（使用join后的project+job可减少数据查询，也方便管理）的关系
p, m1, user_default, read
# m3 继承 m1 的 read
p, m3, user_default, write

# 描述角色继承关系
g, m2,m1
g, m3,m1
g, m3,m2

# 描述人物身份对应角色
g, byron, m1
g, aaron, m3


#g2, article1, m1
#g2, article3, m3
```

### request

```
byron, cate_uuid1, read				// true
byron, cate_uuid1, write			// false
byron, cate_uuid2, read				// false
byron, cate_uuid2, write 			// false
byron, cate_uuid3, read				// false
byron, cate_uuid3, write 			// false

aaron, cate_uuid1, read				// true
aaron, cate_uuid1, write			// false
aaron, cate_uuid2, read				// true
aaron, cate_uuid2, write 			// true
aaron, cate_uuid3, read				// true
aaron, cate_uuid3, write 			// true

byron, user_default, read			// true
byron, user_default, write		// false

aaron, user_default, read			// true
aaron, user_default, write		// false
```

