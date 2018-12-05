#**学习Lua元表总结**

***

**之前浅显的接触了Lua的元表知识是在学习使用Lua创建一个简单的类的时候。当时大多使用setmetatable()方法进行一些继承之类的操作，更深入的学习之后才发现元表的强大所在。**

***
###**元方法**
* lua中的表对象在执行一些操作时，会默认调用一些函数，如执行'+'操作符运算时会调用'__add'函数。这些方法就称为元方法，而我们可以通过对这些元方法赋值来进行重载。

**1.算术类元方法**
```lua
Set = {}

function Set.new(x)
    local set = {}
    for _, v in pairs(x) do set[v] = true end
    return set
end

--定义集合的并运算
function Set.union(lfSet, rtSet)
    local res = Set.new{}
    for element in pairs(lfSet) do res[element] = true end
    for element in pairs(rtSet) do res[element] = true end
    return res
end

--定义集合的交运算
function Set.intersection(lfSet, rtSet)
    local res = Set.new{}
    --按照左集合来逐个查看右集合中是否有相同元素
    for element in pairs(lfSet) do
        res[element] = rtSet[element]
    end
    return res
end

-- 定义打印集合的函数
function Set.tostring(set)
    local l = {}
    -- 此处对l[#l + 1]赋值使用了一个取table长度的技巧省了一个变量，写法上更简洁了。（但是相比使用变量，运行性能上有一点影响）
    for element in pairs(set) do
        l[#l + 1] = element
    end
    -- 此处table.concat函数据说比直接使用字符串拼接速度快。（但是经我测验发现速度差不多，可能是版本差异）
    return "{" .. table.concat(l, ", ") .. "}" 
end

function Set.print(set)
    print(Set.tostring(set))
end

-----------------------------------------------
-- 接下来要请出今天的C位元表出场【-->加特效<--】--
-----------------------------------------------

-- 定义一个表作为元表
local laowang = {} -- 就叫他老王吧

-- 此时产生一个新表的new函数就发生了一些变化
function Set.new(x)
    local son = {}
    setmetatable(son, laowang) -- 将老王的基因传授给他的儿子
    for _, v in pairs(x) do son[v] = true end
    return son
end

xiaozhao = Set.new{1, 2, 3, 4, 5}
xiaowang = Set.new{1, 3, 5, 7, 9}

--此处查看xiaozhao和dazhao的元表
print(getmetatable(xiaozhao)) 
--输出： table: 00E3B4D0
print(getmetatable(xiaowang)) 
--输出： table: 00E384D0
-- (⊙ˍ⊙)!! 没错，小明和小王有着相同的Y染色体。。

laowang.__add = Set.union()     -- 此处使用元方法改变老王的能力
Set.print(xiaozhao + xiaowang)  -- 哦吼！这种操作都可以了
-- 输出： 1, 2, 3, 4, 5, 7, 9
laowang.__mul = Set.intersection
Set.print(xiaozhao * xiaowang)
-- 输出： 1， 5， 3
```

**上面的例子体现了元表的一类作用，使用元表的元方法产生类似重载运算符的作用，使得小赵和小王可以违背常理的结合（滑稽）。像是重载这些运算符的方法是算术类的元方法，类似的还有：**
```
__add 加法
__mul 乘法
__sub 减法
__div 除法
__unm 相反数（别问我我也没用过。。）
__mod 取模
__pow 乘幂
__concat 连接字段
```
**2.关系类元方法**
**lua中关系类元方法较少，仅有：**
```
__eq 等于
__lt 小于
__le 小于等于
```
**其余关系类元方法均是由它们转换而来， 关系类元方法和算术类元方法的使用方法类似，此处就不再赘述**
**3.库函数定义的元方法**
**主要是一个tostring方法用于格式化输出，和一个metatable方法用于保护元表**
```lua
laowang.__tostring = Set.tostring()
print(xiaowang)

laowang.__metatable = "Warning!!!"
print(getmetatable(xiaowang))
setmetatable(xiaowang, {})
--注意此处保护元表是指如果有一个表对象继承了这个元表，那么我们不能获取和修改这个表的元表指向，体现再此处为xiaowang的元表只能是laowang
```
**4.table访问的元方法**
**前面是是一些类似运算符重载的操作，接下来的两个方法则直接修改table的行为（不清楚没关系，下面会详细解释）**
```lua
--index字段，当访问一个table中不存在的字段时，实际上解释器会去查找一个叫__index的元方法。如果没有该元方法，则访问结果如前述为nil，否则就由这个元方法提供最终结果。
Window = {}
--定义窗口原型，用于设置默认值
Window.prototype = {x = 0, y = 0, width = 1024, height = 768}
--声明一个元表给新建的表对象继承
Window.mt = {}
--声明窗口构造函数
function Window.new(o)
    setmetatable(o, Window.mt)
    return o
end


--定义__index元方法，在其中定义找不到表内元素的行为。此处即：如果找不到元素则返回上述定义的原型中的值，实现了默认值的设置
Window.mt.__index = function(table, key) --table访问元素时传入的参数，属于常规操作
    return Window.prototype[key]
end
--此处也可以用
--Window.mt.__index = Window.prototype
--代替，也就是说__index元方法不仅可以定义为函数，还可以定义为一个表，效果是一样的


--创建一个新窗口，并且访问一个它没有的字段
window = Window.new{x=10, y=20}
print(window.width)
-- 输出：1024
```
**虽然此处将函数和表赋值给__index变量是等效的，但是实际上函数更为灵活，可以通过函数来实现类似于多重继承和缓存的功能**
```lua
--newindex字段和index字段类似，不同的是前者用于给不存在的值赋值时访问，后者在查询时访问

--定义赋值缓存
Window.cache = {name = "WINDOW", size = 1024}
Window.mt.__newindex = function(table, key, value)
    Window.cache[key] = value
end

window.name = "WINDOW_NEW"
-- 赋值成功，但是改变的是元表内cache的值
print(window.name)
-- 输出： nil
```
**这里需要注意一点，当我们使用了__index和__newindex元方法对表的行为进行篡改了之后，我们是不能按原来的方法随意增加和删除表的字段了。**
```lua
--试图给表新建一个字段
window.color = "red"
print(window.color)
-- 输出:nil

--试图删除表中已有字段
window.y = nil
print(window.nil)
-- 输出：0
```
**未完待续**