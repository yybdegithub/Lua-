***
__1.对于table, userdata和函数，Lua是作引用比较的。只有当它们引用同一个对象时，才认为它们相等。__

__2.尤其注意数字和字符串在比较大小时的差异 如2 < 15 而'2' > '15'并且Lua中只能对数字和字符串作比较大小操作__

__3.lua中在循环体内的局部变量可以用作条件测试（只限于repeat util循环，while循环不行）__
```lua
repeat
    local tmp = 5
until tmp > 10                                --tmp为循环体内局部变量，但可用于条件测试
---------------------------------------
while tmp > 10 do
    local tmp = 5
    --do something
end                                           --若在循环体内定义tmp则报错:
                                              --(attempt to compare number with nil)
---------------------------------------
```
__4.如果一个函数调用不是一系列表达式的最后一个元素，那么将只产生一个值__
```lua
--假设函数foo()返回a,b两个值

local x, y, z = 10, foo()                    --foo()是最后一个元素，x = 10, y = a, z = b

local x, y, z = foo(),10                     --foo()不是最后一个元素，将只产生一个值, x = a, y = 10, z = nil
```
__5.使用具名实参以确保函数调用的一些参数检查__
```lua
function window(argv)                        --函数原型
window( {titile = , name = } )               --函数调用使用具名实参,实际上是传入一个table

function window(argv)
    if argv.title ==  then .....
    if argv.name ==  then .....
end                                          --在函数中接受具名实参并检查
```
***