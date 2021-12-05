# 介绍

JBCrypt是一个密码加密框架，将salt加入到hash的密码中去，不需要单独保存salt

# Maven

```xml
<dependency>
    <groupId>org.mindrot</groupId>
    <artifactId>jbcrypt</artifactId>
    <version>0.4</version>
</dependency>
```

# 使用

- `BCrypt.gensalt()`：生成salt，一般不单独使用，配合下面这个方法一起使用。
- `BCrypt.hashpw(password, BCrypt.gensalt())`：密码加密，传入密码和salt。返回密码完成的密码。
- `BCrypt.checkpw(candidate, hashed)`：密码验证，传入密码候选值和加密的密码，返回布尔值。

