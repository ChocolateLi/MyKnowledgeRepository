# python爬虫

## bs4



打开BeautifulSoup

```python
from bs4 import BeautifulSoup

with open('home.html','r') as html_file:
    content = html_file.read()
    //把内容丢进汤里,使用lxml的解析方式
    soup = BeautifulSoup(content,'lxml')
    //搜索到第一个标签并停止
    tags = soup.find('h5')
```



## 正则表达式

### 7个level

**level1-固定字符**

要求：确定字符串中是否有123456

```python
import re
text = '麦叔身高:178，体重：168，学号：123456，密码:9527'
print(re.findall(r'123456', text))
```



**level2-某一类字符**

要求：找出所有的单个的数字

```python
text = '麦叔身高:178，体重：168，学号：123456，密码:9527'
print(re.findall(r'\d', text))
```

这个表达式`\d`表示所有的数字，所以1,7,8,1,6,8等都可以匹配到。这是很简单的模式，只匹配1个单独的数字。



**level3-重复某一类字符**

要求：找所有的数字，比如178，168，123456，9527等。

```python
text = '麦叔身高:178，体重：168，学号：123456，密码:9527'
print(re.findall(r'\d+', text))
```

这个模式`\d+`在\d的后面增加了+号，表示数字可以出现1到多次，所以178等都符合它的要求。



**level4-组合level2**

要求：找出座机号码

```python
text = '麦叔电话是18812345678，他还有一个电话号码是18887654321，他爱好的数字是01234567891，他的座机是：0571-52152166'
print(re.findall(r'\d{4}-\d{8}', text))
```

`\d{4}-\d{8}`这是一个组合的模式，表示前面4个数字，中间一个横杠，后面8个数字。

这里会匹配一个结果：0571-52152166



**level5-多种情况**

要求：找出手机号码或者座机号码

```python
text = '麦叔电话是18812345678，他还有一个电话号码是18887654321，他爱好的数字是01234567891，他的座机是：0571-52152166'
print(re.findall(r'\d{4}-\d{8}|1\d{10}', text))
```

比上面有复杂了点，因为使用竖线(|)来表示”或“的关系，就是手机号码和电话号码都可以。

这里会匹配三个结果：18812345678，18887654321，0571-52152166。



**level6-限定位置**

要求：在句子开头的手机号码,或座机

```python
text = '18812345678，他还有一个电话号码是18887654321，他爱好的数字是01234567891，他的座机是：0571-52152166'
print(re.findall(r'^1\d{10}|^\d{4}-\d{8}', text))
```

在表达式的最开始使用了^符号，表示一定要在句子的开头才行。这时候只有18812345678能匹配上。



**level7-内部约束**

要求：找出形如barbar, dardar的前后三个字母重复的字符串

```python
text = 'barbar carcar harhel'
print(re.findall(r'(\w{3})(\1)', text))
```

这里barbar和carcar会匹配上，但harhel不会被匹配，因为它不是3个字符重复的。

\w{3}表示3个字符，放在小括号中(\w{3})就成为一个分组，而后面的(\1)表示它里面的内容和第1个括号里的内容必须相同，其中的1就表示第1个括号，也就是说3个字符要重复出现两次。



### 写正则表达式的步骤

以包含分机号码的座机电话号码为例，比如0571-88776655-9527

1. 确定包含几个子模式

   它包含3个子模式：**0571**-**88776655**-**9527**。这3个子模式用固定字符连接。

2. 各个部分字符分类是什么

   这3个子模式都是数字类型，可以用\d。现在可以写出模式为：`\d-\d-\d`

3. 各个子模式如何重复

   第1个子模式重复3到4次，因为有010和021等直辖市

   第2个子模式重复7到8次，有的地区只有7位电话号码

   第3个子模式重复3-4次

   加上次数限制后，模式成为：`\d{3,4}-\d{7,8}-\d{3,4}`

   但有的座机没有分机号，所以我们用或运算符让它支持两者：

   ```
   \d{3,4}-\d{7,8}-\d{3,4}|\d{3,4}-\d{7,8}
   ```

4. 是否有外部限制

5. 是否内部制约



### 语法规则

![](D:\github\MyKnowledgeRepository\img\picture\正则表达式.png)



```python
import re
text = '麦叔身高:178，体重：168，学号：123456，密码:9527'
print(re.findall(r'(?<=密码.)(?#这部分是注释，正则不会匹配它。表示必须要在密码之后)\d+', text))
```



### python正则模块 re

#### 常用方法

- re.search()：查找符合模式的字符，只返回第一个，返回Match对象
- re.match()：和search一样，但要求必须从字符串开头匹配
- re.findall()：返回所有匹配的字符串列表
- re.finditer()：返回一个迭代器，其中包含所有的匹配，也就是Match对象
- re.sub()：替换匹配的字符串，返回替换完成的文本
- re.subn()：替换匹配的字符串，返回替换完成的文本和替换的次数
- re.split()：用匹配表达式的字符串做分隔符分割原字符串
- re.compile()：把正则表达式编译成一个对象，方便后面使用





## xpath

### 基本语法

xpath的思想：通过路径找节点（元素属性、内容）

固定格式：`//*[]`



**语法规则**

`/` 根节点，节点分隔符

`//` 任意位置

`*` 任意元素

`@` 属性

`.` 当前节点

`..` 父级节点，也可以使用parent::来获取父节点

`text` 文本值



**常用两种格式**

通过属性查找：`//*[@属性='属性值']` 

通过文本值查找：`//*[text()='文本值']` 



可以使用逻辑关系词如and

`//*[text()='文本值' and @属性='属性值']` 



返回多个元素节点

`//*[] | //*[]`



**高级用法**

contains模糊查询：`contains(@属性/text()，值)`

starts-with 匹配一个属性开始位置的关键字：`starts-with(@属性/text(),值)`



**节点轴选择**

XPath提供了很多节点选择方法，包括获取子元素、兄弟元素、父元素、祖先元素等

```python
from lxml import etree

text1='''
<div>
    <ul>
         <li class="aaa" name="item"><a href="link1.html">第一个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第二个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第三个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第四个</a></li> 
     </ul>
 </div>
'''

html=etree.HTML(text1,etree.HTMLParser())
result=html.xpath('//li[1]/ancestor::*')  #获取所有祖先节点
result1=html.xpath('//li[1]/ancestor::div')  #获取div祖先节点
result2=html.xpath('//li[1]/attribute::*')  #获取所有属性值
result3=html.xpath('//li[1]/child::*')  #获取所有直接子节点
result4=html.xpath('//li[1]/descendant::a')  #获取所有子孙节点的a节点
result5=html.xpath('//li[1]/following::*')  #获取当前子节之后的所有节点
result6=html.xpath('//li[1]/following-sibling::*')  #获取当前节点的所有同级节点


#
[<Element html at 0x3ca6b960c8>, <Element body at 0x3ca6b96088>, <Element div at 0x3ca6b96188>, <Element ul at 0x3ca6b961c8>]
[<Element div at 0x3ca6b96188>]
['aaa', 'item']
[<Element a at 0x3ca6b96248>]
[<Element a at 0x3ca6b96248>]
[<Element li at 0x3ca6b96308>, <Element a at 0x3ca6b96348>, <Element li at 0x3ca6b96388>, <Element a at 0x3ca6b963c8>, <Element li at 0x3ca6b96408>, <Element a at 0x3ca6b96488>]
[<Element li at 0x3ca6b96308>, <Element li at 0x3ca6b96388>, <Element li at 0x3ca6b96408>]
```





**特殊**

svg：`//*[name()='svg']`



## lxml库

lxml的基本用法

```python
from lxml import etree
# 将字符串转化为Element对象
# Element对象具有xpath的方法，可以接收bytes和str的数据类型
# lxml可以自动修正html代码
html = etree.HTML(text) # Element对象
print(etree.tostring(html).decode('utf-8')) # 转化为字符对象
# 使用xpath
html.xpath('//*') # 它是一个列表元素
```



### 使用入门

1.导入lxml的etree库`from lxml import etree`

2.利用`etree.HTML()`方法，将字符串转化为Element对象

```python
html = etree.HTML(text)
```

3.Element对象具有xpath()方法

```
element = html.xpath()
```



### 常用方法

`etree.HTML(text)` :转化为xpath对象

`etree.tostring(html,enconding="utf-8"，pretty_print=True).decode("utf-8")` :解析成二进制对象，进一步解析成字符串对象

`etree.parse(html文件)`:用这个方法来读取文件

```python
parser = etree.HTMLParser(encoding='utf-8')
html_element = etree.parse('1.html',parser)
```



















