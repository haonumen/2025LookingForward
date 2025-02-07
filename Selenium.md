# Selenium 使用定位器(Locator)的技巧
## 何时使用哪些定位器以及如何在代码中最好地管理它们
一般来说，如果HTML ID是可用的、唯一的并且始终可预测的，那么它们是在页面上定位元素的首选方法。它们往往工作得非常快，并且放弃了复杂DOM遍历带来的许多处理。
如果唯一id不可用，那么编写良好的CSS选择器是定位元素的首选方法。XPath和CSS选择器一样好用，但是语法很复杂，而且经常难以调试。虽然XPath选择器非常灵活，但它们通常没有经过浏览器供应商的性能测试，而且速度很慢。
基于linkText和partialLinkText的选择策略有缺点，因为它们只适用于链接元素。此外，它们在WebDriver内部调用querySelectorAll选择器。
标记名(Tag name)可能是定位元素的一种危险方法。页面上经常会出现相同标签的多个元素。这在调用返回元素集合的findElements（By）方法时非常有用。
我们的建议是让定位器尽可能简洁易读。要求WebDriver遍历DOM结构是一项代价高昂的操作，而且越能缩小搜索范围越好。
(1) ID / Name
(2) CSS Selector / XPath
(3) LinkText / ParialLinkText
(4) Tag Name

# Web元素信息
## isDisplayed()
此方法用于检查所连接的元素是否显示在网页上。返回一个布尔值，如果连接的元素显示在当前浏览上下文中则返回True，否则返回false
## isEnabled()
此方法用于检查在网页上是否启用或禁用了已连接的元素。返回一个布尔值，如果连接的元素在当前浏览上下文中启用，则返回True，否则返回false。
## isSelected()
此方法确定引用的元素是否被选中。该方法广泛用于复选框、单选按钮、输入元素和选项元素。
返回一个布尔值，如果引用的元素在当前浏览上下文中被选中，则返回True，否则返回false。
## getTagName()
它用于获取当前浏览上下文中具有焦点的引用元素的TagName。
## 尺寸和位置 getRect()
它用于获取被引用元素的尺寸和坐标。
获取的数据体包含以下详细信息：
- 从元素的左上角开始的x轴位置
- 从元素的左上角开始的y轴位置
- 元素的高度
- 元素的宽度
## 获取CSS值 getCssValue()
检索当前浏览上下文中元素的指定计算样式属性的值。
## 文本内容 getText()
检索指定元素的呈现文本。
## 获取属性 getAttribute()
获取与DOM属性关联的运行时值。它返回与元素的DOM属性或属性相关联的数据

# 定位器 （Locator）
定位器 （Locator）是在页面上识别元素的方法。它是传递给查找元素方法的参数。
Selenium为WebDriver中的这8种传统定位策略提供了支持。
| 定位器 Locator | 描述 | 
| --- | --- | 
| class name | 定位class属性与搜索值匹配的元素（不允许使用复合类名） |
| css selector | 定位 CSS 选择器匹配的元素 | 
| id | 定位 id 属性与搜索值匹配的元素 | 
| name | 定位 name 属性与搜索值匹配的元素 | 
| link text | 定位link text可视文本与搜索值完全匹配的锚元素 | 
| partial link text | 定位link text可视文本部分与搜索值部分匹配的锚点元素。如果匹配多个元素，则只选择第一个元素。 |
| tag name | 定位标签名称与搜索值匹配的元素 | 
| xpath | 定位与 XPath 表达式匹配的元素 | 

## CSS选择器比XPath快的主要原因包括以下几点‌：
- ‌遍历方式不同‌：CSS选择器是通过匹配DOM元素来定位，而XPath是通过遍历DOM树来查找元素。由于CSS选择器直接匹配DOM元素，不需要像XPath那样遍历整个DOM树，因此执行速度更快‌。
- 设计原理不同‌：CSS选择器设计时考虑了性能优化，而XPath则是为了灵活性和通用性设计。CSS选择器在匹配对象时更加高效，而XPath需要遍历HTML元素，这导致了性能上的差异‌。
- ‌浏览器支持‌：在不同的浏览器中，CSS选择器的性能表现通常优于XPath。例如，在Chrome和Firefox浏览器中，CSS选择器的查找速度更快，效率更高；而在IE浏览器中，XPath的效率相对较高。

[https://elementalselenium.com/](https://elementalselenium.com/)

# 相对定位器 （Relative Locators）
Selenium 4引入了相对定位器（以前称为友好定位器）。当不容易为所需的元素构造定位器，但容易在空间上描述元素相对于具有容易构造定位器的元素的位置时，这些定位器是有用的。
## 它是如何工作的
Selenium使用JavaScript函数getBoundingClientRect（）来确定页面上元素的大小和位置，并可以使用此信息来定位相邻元素。找出相关元素。
相对定位器方法可以将先前定位的元素引用或另一个定位器作为原点的参数。
- Above
- Below
- Left of
- Right of
- Near
```TypeScript
let emailLocator = locateWith(By.tagName('input')).above(By.id('password'));
let passwordLocator = locateWith(By.tagName('input')).below(By.id('email'));
let cancelLocator = locateWith(By.tagName('button')).toLeftOf(By.id('submit'));
let submitLocator = locateWith(By.tagName('button')).toRightOf(By.id('cancel'));
let emailLocator = locateWith(By.tagName('input')).near(By.id('lbl-email'));
let submitLocator = locateWith(By.tagName('button')).below(By.id('email')).toRightOf(By.id('cancel'));
```

# Selenium中的等待类型
Selenium提供了三种主要的等待方式：
- 隐式等待（Implicit Wait）
- 显式等待（Explicit Wait）
- 流式等待（Fluent Wait）

## 隐式等待（Implicit Wait）
隐式等待告诉Selenium WebDriver在查找一个元素时，如果元素没有立即出现，则等待一段时间。在等待的这段时间内，Selenium WebDriver会每隔一段时间重新尝试查找元素。
```PYTHON
from selenium import webdriver
driver = webdriver.Chrome()
driver.get("http://example.com")
# 设置隐式等待
driver.implicitly_wait(10)  # 等待最长10秒
# 尝试查找元素
element = driver.find_element_by_id("myElement")
```

## 显式等待（Explicit Wait）
显式等待是在代码中指定条件的等待，可以在指定的时间内等待某个条件成立。最常用的显式等待是WebDriverWait。
```PYTHON
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("http://example.com")

# 设置显式等待
wait = WebDriverWait(driver, 10)
element = wait.until(EC.presence_of_element_located((By.ID, "myElement")))

print(element.text)
```
## 流式等待（Fluent Wait）
流式等待是显式等待的一种扩展，允许我们定义等待的最大时长、轮询间隔以及忽略的异常类型。

```PYTHON
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import FluentWait
from selenium.common.exceptions import NoSuchElementException
import time

driver = webdriver.Chrome()
driver.get("http://example.com")

# 设置流式等待
wait = FluentWait(driver)
wait.with_timeout(10)  # 最长等待10秒
wait.polling_every(2)  # 每2秒检查一次
wait.ignoring(NoSuchElementException)

element = wait.until(lambda x: x.find_element_by_id("myElement"))

print(element.text)
```

# 使用选择列表元素
与其他元素相比，选择列表具有特殊的行为.Select对象现在将为您提供一系列命令, 用于允许您与 select 元素进行交互.
请注意，此类仅适用于 HTML 元素 select 和 option. 这个类将不适用于那些通过 div 或 li 并使用JavaScript遮罩层设计的下拉列表.
## 类型
选择方法的行为可能会有所不同， 具体取决于正在使用的 select 元素的类型.
### 单选
这是标准的下拉对象，其只能选定一个选项.
### 复选
此选择列表允许同时选定和取消选择多个选项. 这仅适用于具有 multiple 属性的 select元素.

## 构建类
首先定位一个 select 元素, 然后借助其初始化一个Select 对象. 请注意, 从 Selenium 4.5 开始, 您无法针对禁用的 select 元素构建 Select 对象.
```PYTHON
select_element = driver.find_element(By.NAME, 'selectomatic')
select = Select(select_element)
```
### 选项列表
共有两种列表可以被获取:

#### 全部选项
获取 select 元素中所有选项列表:
```PYTHON
option_list = select.options
```
#### 选中的选项
获取 select 元素中所选中的选项列表. 对于标准选择列表这将只是一个包含一个元素的列表, 对于复选列表则表示包含的零个或多个元素.
```PYTHON
selected_option_list = select.all_selected_options
```
### 选择选项
Select类提供了三种选择选项的方法. 请注意, 对于复选类型的选择列, 对于要选择的每个元素可以重复使用这些方法.

#### 文本
根据其可见文本选择选项
```PYTHON
select.select_by_visible_text('Four')
```
#### 值
根据其值属性选择选项
```PYTHON
select.select_by_value('two')
```
#### 序号
根据其在列表中的位置选择选项
```PYTHON
select.select_by_index(3)
```
#### 取消选择选项
只有复选类型的选择列表才能取消选择选项. 您可以对要选择的每个元素重复使用这些方法.
```PYTHON
select.deselect_by_value('eggs')
```

# Selenium 常见的异常
- 无效选择器的异常 (InvalidSelectorException)
某些时候难以获得正确的CSS以及XPath选择器。
- 没有这样元素的异常 (NoSuchElementException)
在您尝试找到该元素的当前时刻无法定位元素。
- 过时元素引用的异常 (StaleElementReferenceException)
当成功定位到元素时， WebDriver会为其设置一个引用ID作为标记， 如果由于上下文环境发生变化， 导致之前元素的位置发生了变化或者无法找到了， WebDriver并不会自动重新定位， 任何使用之前元素所做的操作将报错该异常。
- ElementClickInterceptedException
This exception occurs when Selenium tries to click an element, but the click would instead be received by a different element. Before Selenium will click an element, it checks if the element is visible, unobscured by any other elements, and enabled - if the element is obscured, it will raise this exception.
- 无效SessionId异常
有时您尝试访问的会话与当前可用的会话不同。
- SessionNotCreatedException
此异常发生在 WebDriver 无法为浏览器创建新会话时。通常由于版本不匹配、系统级限制或配置问题导致。
- ElementNotInteractableException
当 Selenium 尝试与当前状态下无法交互的元素进行交互时，会发生此异常。











