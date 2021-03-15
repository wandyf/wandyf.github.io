---
layout: post
title: "Thymeleaf 基础语法"
# background-image: ""
date: 2020-10-12
published: true
category: Language
tags:
        - Thymeleaf
---

# 引用命名空间

```html
<html lang="zh" xmlns:th="http://www.thymeleaf.org"
        xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
```

# 常用th标签

- th:text
    - `<div th:text="${name}?:'default-name'">name</div>`
    - 用于显示控制器传入的name值，不存在则显示默认值
- th:object
    - `th:object="${user}"`
    - 用于接收后台传来的对象
- th:action
    - `<form th:action="@{/article/}+${article.id}" method="post"></form>`
    - 用于指定表单提交地址
- th:value
    - `<input type="text" th:value="${article.id}" name="id"/>`
    - 用对象将id的值替换为value的属性
- th:field
    - `<input type="text" id="title" name="title" th:field="${article.title}"/>`
    - `<input type="text" id="title" name="title" th:field="*{title}"/>`
    - 用于绑定后台对象和表单数据

# URL写法

- 通过语法`@{...}`来处理URL，需要使用`th:href`和`th:src`等属性
    - `<a th:href="@"http:/>/eg.com"">绝对路径</a>`
    - `<a th:href="@{/}">相对路径</a>`
    - `<a th:href="@{css/bootstrap.min.css}">默认访问static下的css文件夹</a>`

# 用Thymeleaf进行条件求值

 - `th:if`，条件成立时
     - `<a th:href="@{/login}" th:if${session.user == null}>Login</a>`
 - `th:unless`，条件不成立时
    - `<a th:href="@{/login}" th:unless=${session.user == null}Login</a>`

# Switch

```html
<div th:switch=${user.role}>
    <p th:case="admin">管理员</p>
    <p th:case="vip">VIP</p>
    <p th:case="*">普通会员</p>
</div>
```

# 字符串拼接替换

- `<span th:text="'欢迎，' + ${name} + '!'"></span>`
- `<span th:text="|欢迎，${name}!|">`

# 运算符

- 算数运算符
    - `<span th:text="1+3">1 + 3 </span><br/>`
    - `<span th:text="9%2">9 % 2 </span><>br/`
- 条件运算符
    - `<div th:if="${role} eq admin"></div>`
    - `gt`:大于
    - `ge`:大于或等于
    - `eq`:等于
    - `lt`:小于
    - `le`:大于或等于
    - `ne`:不等于
- 空值判断
    - `<span th:if="${name} == null">空</span>`
    - `<span th:if="${name} != null">非空</span>`

# 公用对象

- 格式化时间
    - `<td th:text="${#dates.format(item.createTime, 'yyyy-MM-dd HH:mm:ss')}">格式化时间</td>`
- 判断是否为空字符串
    - `<span th:if="${#strings.isEmpty(name)}">空字符串</span>`
- 判断是否包含字符串，区分大小写
    - `<span th:if="${#strings.contains(name, 'Bob')}">包含Bob</span>`

# 循环遍历

- 遍历对象
    - ```html
        <div th:each="article:${articles}">
            <li th:text="${article.title}">title</li>
            <li th:text="${article.body}">body</li>
        </div>
        ```
- 遍历分页
    - ```html
        <div th:each="item:${page.content}">
            <li th:text="${item.id}">id</li>
            <li th:text="${item.title}">title</li>
        </div>
        ```
- 遍历列表
    - ```html       
        <div th:each="item:${list}">
            <li th:text="${item.id}">id</li>
            <li th:text="${item.name}">name</li>
        </div>
        ```
- 遍历数组
    - ```html
         <div th:each="item:${arrays}">
            <li th:text="${item}">item</li>
        </div>
        ```
- 遍历集合
    - ```html
         <div th:each="item:${map}">
            <li th:text="${item.key}">key</li>
            <li th:text="${item.value}">value</li>
            <li th:text="${item}">key-value</li>
        </div>
        ```

