# 作用域链和闭包 ：代码中出现相同的变量，JavaScript 引擎是如何选择的？

## 作用域链

每个执行上下文的变量环境中,都包含一个外部引用,指向外部的执行上下文,这个外部引用成为`outer`

JavaScript 引擎查找变量时,会现在当前执行上下文的词法环境和变量环境中查找,如果查找不到,则在变量环境中的外部引用执行上下文中查找,直到得出最终结果,这个查找链条称为`作用域链`

## 词法作用域

词法作用域是静态作用域,作用域由代码中函数声明的位置决定,函数执行上下文环境变量的外部引用指向词法作用域;

## 闭包

根据词法作用域的规则,内部函数总是可以访问外部函数中声明的变量;

当通过调用外部函数,并返回一个内部函数后,但内部函数引用的外部函数变量依然存在,外部函数和被引用的变量称为闭包;

闭包查找变量: 当前执行上下文->函数闭包-> 外部引用上下文-> 全局执行上下文

内存泄漏（Memory leak）: 由于疏忽或错误造成程序未能释放已经不再使用的内存。

如果引用闭包的函数是一个全局变量，那么闭包会一直存在直到页面关闭；但如果这个闭包以后不再使用的话，就会造成内存泄漏。

如果引用闭包的函数是个局部变量，等函数销毁后，在下次 JavaScript 引擎执行垃圾回收时，判断闭包这块内容如果已经不再被使用了，那么 JavaScript 引擎的垃圾回收器就会回收这块内存。

所以在使用闭包的时候，你要尽量注意一个原则：如果该闭包会一直使用，那么它可以作为全局变量而存在；但如果使用频率不高，而且占用内存又比较大的话，那就尽量让它成为一个局部变量。
