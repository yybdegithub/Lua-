__Closure__  ~~***一般习惯翻译为闭包***~~  __是lua中非常重要的机制，lua中的函数也只是Closure的一种特殊形式，lua有很多基于Closure的高级用法，下面我将列举其中几种：__

__一.用作匿名函数__
```lua
--看一个重载排序的例子
local classInfo = {
    {name = "Kobe", score = 80},
    {name = "James", score = 50},
    {name = "O'Neal", score = 100},
}

function sortByName(x)  --按名字排序
    table.sort(x, 
        function(lf, rf) return lf.name < rf.name end)
end

function sortByScore(x) --按分数排序
    table.sort(x,
        function(lf, rf) return lf.score < rf.score end)
end

sortByName(classInfo)
for i,v in pairs(classInfo) do
    print(i, v.name, v.score)
end

sortByScore(classInfo)
for i,v in pairs(classInfo) do
    print(i, v.name, v.score)
end
--上述例子中table.sort方法可以访问其上级函数的参数，这里涉及到一个词法域的问题