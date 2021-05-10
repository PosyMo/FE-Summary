## markdown 语法整理

#### 1.1 标题

##### [h1~h6]
```
# ~ ######
```
一般#作为文章大标题，只有一个，### 作为段落标题。

##### 上下文标题
```
AAA
===
BBB
---
```

#### 1.2 强调
##### 引用
```
> 引用的内容
```

```
>如果引用内容需要换行，  
>可以在行尾添加两个空格
>
>或者在引用内容中加一个空行
```
>如果引用内容需要换行，  
>可以在行尾添加两个空格
>
>或者在引用内容中加一个空行

##### 斜体
```
*
_
```

##### 加粗
```
**
```

##### 删除线
```
~~
```

#### 1.3代码

##### 代码块标记
```
```code```
```

##### 代码块缩进表示法

```
Tab 或四个空格
```

##### 语法高亮显示

```
```javascript
```

##### 内联代码块
```
` 
```

#### 1.4表格

##### 表格
```
|     a     |        b        |      c       |
|:---------:|:--------------- | ------------:|
|   居中    | 左对齐           |       右对齐 |
| ========= | =============== | ============ |
```
##### 简约写法
```
a  | b | c  
:-:|:- |-:
    居中    |     左对齐      |   右对齐    
============|=================|=============
```

#### 1.5链接

##### 内链式
```
[百度1](http://www.baidu.com/"百度一下"){:target="_blank"} 
```

##### 引用式
```
[百度2][2]{:target="_blank"}
[2]: http://www.baidu.com/  "百度二下"
```

##### 邮箱链接
```
<xxx@outlook.com>
```

#### 1.6图片

##### 内链式
```
![name](./01.png '描述')
```

##### 引用式
```
![name][01]
[01]: ./01.png '描述'
```

##### 图片带有链接
```
[![name](./01.png '百度')](http://www.baidu.com){:target="_blank"}   

[![name](./01.png '百度')][5]{:target="_blank"} 
[5]: http://www.baidu.com
```

#### 1.7 分割线
```
---

***
```

#### 2.1序表

##### 无序
```
* one
* two
* three
+ - 可替代 *
```

##### 有序
```
1. one
2. two
3. three
```

##### 序表嵌套
```
* one
    * two
    * three


1. one
    2. two
    3. three
```

#### 2.2清单
```
- [x] 选项一 
- [ ] 选项二
```

#### 2.3锚点
```
[公式标题锚点](#1)

[需要跳转的目录] {#1}
```