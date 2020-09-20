# go语言学习记录
demo
## 程序结构
### 名称
    Go中函数、变量、常量、类型、语句标签和包的名称遵循一个简单的规则：名称的开头是一个字母或下划线，后面可以跟任意数量的字符、数字和下划线、并区分大小写。
    Go中有25个像if和switch这样的关键字，只能用在语法允许的地方，他们不能作为名称：
    break   default func    interface   select  case    defer   go  map struct  chan    else    goto    package switch  const   fallthrough if  range   type    continue    for import  return  var
    另外还有35个预声明的内置的常量、类型和函数：
    常量：true  false   iota    nil
    类型：int int8  int16   int32   int64   uint uint8  uint16  uint32  uint64  float32 float64 complex128  complex64   uintptr bool    byte    rune    string  error
    函数：make len cap  new append  copy    close   delete  complex real    imag    panic   recover

    风格上，当遇到单词组合的名称时，Go程序员使用“驼峰式”的风格-更喜欢使用大写字母而不是下划线。像ASCII和HTML这种的首字母缩写词通常使用相同的大小写，所以一个函数可以叫做htmlEscape、HTMLEscape或者escapeHTML,但不会是escapeHtml
### 变量
    var声明创建一个具体类型的变量，然后给它附加一个名字，设置它的初始值。每一个声明有一个通用的形式：
    var name type = expression
    类型和表达式部分可以省略一个，但是不能都省略。如果类型省略，它的类型将由初始化表达式决定，如果表达式省略，它的初始值应于类型的零值--对于数字时0，对于布尔值时false，对于字符串是"",对于借口和引用类型（slice，map，指针，函数，通道）是nil，对于一个像数组或结构体这样的复合类型，零值是其所有元素或成员的零值
    零值机制保证所有的变量是良好定义的，Go里面不存在未初始化的变量。这种机制简化了代码，并且不需要额外工作就能感知便捷条件的行为
    var s string //""
    var i,j,k int //int int int 
    var b, f, s = true, 2.3, "four" //bool float64 string
    初始值设定可以是字面量值或者任意的表达式。包级别的初始化在main开始之前进行，局部变量初始化和声明一样在函数执行期间进行
    变量可以通过调用返回多个值的函数进行初始化：
    var f, err = os.Open(name) //os.Open返回一个文件和一个错误
### 短变量声明
    在函数中，一种称作短变量声明的可选形式可以用来声明和初始化局部变量。
    name := expression
    anim := gif.GIF{LoopCount: nframes}
    freq := rand.Float64() * 3.0
    t := 0
    因其短小灵活，故在局部变量的声明和初始化中主要使用短变量声明。var声明通常是为那些跟初始化表达式类型不一致的局部变量保留的，或者用于后面才对变量赋值以及变量初始值不重要的情况。
    i := 100 //一个int类型的变量
    var boiling float64 = 100 // 一个float64类型的变量
    var names []string
    var err error
    var p Point
    与var声明一样，多个变量可以以短变量声明的方式声明和初始化
    i, j := 0, 1 //声明i,j
    :=表示声明，=表示赋值。
    i, j = j, i // 交换i和j的值
    短变量声明最少声明一个新变量，否则，代码编译不通过
### 指针
    指针的值是一个变量的地址。一个指针指示值所保存的位置。不是所有的值都有地址，但是所有的变量都有。使用指针，可以在
无须知道变量名字的情况下，简介读取或更新变量的值，var x int,表达式&x(x的地址)获取一个指向整型变量的指针，它的类型为*int,p=&x表示p指向x,p指向的变量写成*p，即*p表示变量x的值。指针类型的零值为nil，且可以比较
### new函数
    表达式new(T)创建一个未命名的T类型的变量，初始化为T类型的零值,并返回其地址
    p := new(int)//*int类型的p，执行未命名的int变量
    fmt.Println(*p)//输出0
    使用new创建的变量和取其地址的普通局部变量没有什么不同，只是不需要引入一个虚拟的名字，通过new(T)就可以直接在表达
因此new指示语法上的便利，不是一个基础概念，它是一个预声明的函数，不是一个关键字，因此可以重定义为其他类型。
### 变量的生命周期
    生命周期是指在程序执行过程中，变量存在的时间段，包级别的变量的生命周期是整个程序的执行时间，相反，局部变量有一个动态的生命周期，垃圾回收器苏荷知道一个变量是否应该被回收？基本思路是每一个包级别的变量，以及每一个当前执行函数的局部变量，可以作为追溯该变量的路径的源头，通过指针和其他方式引用可以找到变量。如果变量的路径不存在，那么变量变得不可访问，因此他不会影响任何其他的计算过程。
    var global *int
    func f(){
        var x int
        x = 1
        global = &x
    }
    func g(){
        y := new(int)
        *y = 1
    }
    x使用堆空间，因为他在f()返回的时候还可以从global变量访问，尽管他被声明为一个局部变量。这种情况我们说x从f逃逸。相反，当g函数返回时，变量*y变得不可访问，可回收。因为*y没有从g中逃逸，所以编译器可以安全的在栈上分配*y,即便使用new函数创建它。
### 赋值
    x = 1 //有名称的变量
    *p = true //间接变量
    person.name = "bob" // 结构体成员
    count[x] = count[x] * scale //数组或slice或map的元素
    v := 1
    v++ // 等同与v = v + 1
    v-- // 等同与v = v - 1
### 多重赋值
    指的是，允许将几个变量一次性被复制。在实际更新变量前，右边的表达式被推演，当变量同时出现在赋值符连个的时候，这种形式特别好用，例如当交换两个变量的值时：
    x, y = y, x
    a[i], a[j] = a[j], a[i]
### 包和文件
    


