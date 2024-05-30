Spring Security 提供了三种不同的安全注解：

1.Spring Security 自带的@Secured 注解；

2.JSR-250 的@RolesAllowed 注解；

3.表达式驱动的注解，包括@PreAuthorize、@PostAuthorize、@PreFilter 和 @PostFilter。

## [](https://mrbird.cc/Spring-Security%E4%BF%9D%E6%8A%A4%E6%96%B9%E6%B3%95.html#Secured "@Secured")@Secured

在 Spring-Security.xml 中启用@Secured 注解：

```

<global-method-security secured-annotations="enabled"/>
```

例如只有拥有权限“ROLE_ADMIN”的用户才能访问下面这个方法：

```
@Secured("ROLE_ADMIN")
public void test(){
    ...
}
```

权限不足时，方法抛出 Access Denied 异常。

@Secured 注解会使用一个 String 数组作为参数。每个 String 值是一个权限，调用这个方法至少需要具备其中的一个权限。如：
```
@Secured({"ROLE_ADMIN","ROLE_USER"})  
public void test(){  
 ...  
}
```

## [](https://mrbird.cc/Spring-Security%E4%BF%9D%E6%8A%A4%E6%96%B9%E6%B3%95.html#RolesAllowed "@RolesAllowed")@RolesAllowed

@RolesAllowed 注解和@Secured 注解在各个方面基本上都是一致的。启用@RolesAllowed 注解：

```
 <global-method-security jsr250-annotations="enabled"/> 
```

栗子：
```
@RolesAllowed("ROLE_ADMIN")  
public void test(){  
 ...  
}
```

## [](https://mrbird.cc/Spring-Security%E4%BF%9D%E6%8A%A4%E6%96%B9%E6%B3%95.html#SpEL%E6%B3%A8%E8%A7%A3 "SpEL注解")SpEL 注解

启用该注解：
```
<global-method-security pre-post-annotations="enabled"/> 
```
### [](https://mrbird.cc/Spring-Security%E4%BF%9D%E6%8A%A4%E6%96%B9%E6%B3%95.html#PreAuthorize "@PreAuthorize")@PreAuthorize

该注解用于方法前验证权限，比如限制非 VIP 用户提交 blog 的 note 字段字数不得超过 1000 字：
```
@PreAuthorize("hasRole('ROLE_ADMIN') and #form.note.length() <= 1000 or hasRole('ROLE_VIP')")  
public void writeBlog(Form form){  
 ...  
}
```
表达式中的#form 部分直接引用了方法中的同名参数。这使得 Spring Security 能够检查传入方法的参数，并将这些参数用于认证决策的制定。

### [](https://mrbird.cc/Spring-Security%E4%BF%9D%E6%8A%A4%E6%96%B9%E6%B3%95.html#PostAuthorize "@PostAuthorize")@PostAuthorize

方法后调用权限验证，比如校验方法返回值：
```
@PreAuthorize("hasRole(ROLE_USER)")  
@PostAuthorize("returnObject.user.userName == principal.username")  
public User getUserById(long id){  
 ...  
}
```

Spring Security 在 SpEL 中提供了名为 returnObject 的变量。在这里方法返回一个 User 对象，所以这个表达式可以直接访问 user 对象中的 userName 属性。
