# 字符串的不可变性

一旦一个string对象在内存（堆）中被创建出来，他就无法被改变。所以如果需要一个可修改的字符串，应该使用StringBuffer或StringBuilder。

效率比较（用时）：`StringBuilder`<`StringBuffer`<`concat`<`+`<`StringUtils.join`