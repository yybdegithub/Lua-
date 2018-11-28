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
-- (⊙ˍ⊙)!! 他俩居然都是老王的后代

laowang.__add = Set.union()     -- 此处使用元方法改变老王的能力
Set.print(xiaozhao + xiaowang)  -- 小赵和小王居然能结合了。。。
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
**其余关系类元方法均是由它们转换而来**
**未完待续。。。。。。