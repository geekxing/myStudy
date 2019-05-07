# Swift Optional

## Introduce
Swift 是一个强类型语言，它希望在编译期做更多的安全检查，所以引入了类型推断。
Optional 在 Swift 语言中其实是一个枚举类型：

```
public enum Optional<Wrapped> {
    case none
    case some(Wrapped)
}
```
## Optional 的嵌套
看下面的例子

```
let a: Int? = 1
let b: Int?? = a
let c: Int??? = b
```
使用`fr v -R`观察每个变量内存构成

```
(lldb) frame variable -R a
(Swift.Optional<Swift.Int>) a = some {
  some = {
    _value = 1
  }
}

(lldb) frame variable -R b
(Swift.Optional<Swift.Optional<Swift.Int>>) b = some {
  some = some {
    some = {
      _value = 1
    }
  }
}

(lldb) frame variable -R c
(Swift.Optional<Swift.Optional<Swift.Optional<Swift.Int>>>) c = some {
  some = some {
    some = some {
      some = {
        _value = 1
      }
    }
  }
}
```
这个内存结构其实是一个类似二叉树一样的形状
![多层嵌套optional树结构](https://static001.infoq.cn/resource/image/a7/a8/a7abf52ecdbe4e7cdb8a80a5cd780ea8.jpg)
Q：Optional.None 可以出现在每一层，那么在每一层的效果一样吗？   
A：不一样

```
let a: Int? = nil
let b: Int?? = a
let c: Int??? = b
let d: Int??? = nil
```
上述每个变量值都是nil，来看下他们的内存布局

```
(lldb) fr v -R a
(Swift.Optional<Swift.Int>) a = None {
  Some = {
    value = 0
  }
}

(lldb) fr v -R b
(Swift.Optional<Swift.Optional<Swift.Int>>) b = Some {
  Some = None {
    Some = {
      value = 0
    }
  }
}

(lldb) fr v -R c
(Swift.Optional<Swift.Optional<Swift.Optional<Swift.Int>>>) c = Some {
  Some = Some {
    Some = None {
      Some = {
        value = 0
      }
    }
  }
}

(lldb) fr v -R d
(Swift.Optional<Swift.Optional<Swift.Optional<Swift.Int>>>) d = None {
  Some = Some {
    Some = Some {
      Some = {
        value = 0
      }
    }
  }
}

```

* 变量 c 因为是多层嵌套的 nil，所以它在顶层的二叉树上的值，是一个Optional\<Optional\<Int>>。
* 变量 d 因为是直接赋值成 nil，所以它在顶层的二叉树上的值，是一个 Optional.None。

麻烦：Optional 类型的变量，在使用时，大多需要用if let的方式来解包，但如果用 if let 来判断变量 c 是否为 nil，会发现失效了。
## Optional 的构造函数
Swift 是强类型，但在上节中，optional变量赋值左右两边类不一样，却能成功赋值不报错。   
比如，变量 b（类型为 Int??），它可以接受以下几种类型的赋值：

1、nil 类型   
2、Int? 类型   
3、Int?? 类型   

看Optional的[源码](https://github.com/apple/swift/blob/master/stdlib/public/core/Optional.swift)

```
public enum Optional<Wrapped> : _Reflectable, NilLiteralConvertible {
  case None
  case Some(Wrapped)

  @available(*, unavailable, renamed="Wrapped")
  public typealias T = Wrapped

  /// Construct a `nil` instance.
  @_transparent
  public init() { self = .None }

  /// Construct a non-`nil` instance that stores `some`.
  @_transparent
  public init(_ some: Wrapped) { self = .Some(some) }

}
```
以上代码中，Optional 提供了两种构造函数，完成了刚刚提到的类型转换工作。  
**这种机制支持了Optional的多层嵌套，也导致使用 `if let` 判断多层嵌套的nil失效的问题。**
## 看个🌰
```
var dict :[String:String?] = [:]
dict = ["key": "value"]
func justReturnNil() -> String? {
    return nil
}
dict["key"] = justReturnNil()
dict
```
运行结果
![result](https://github.com/geekxing/myStudy/blob/master/resources/swifty_optional.png?raw=true)
意图是通过给这个 Dictionary 设置一个 nil，来删除掉这个 key-value 对。但是从 playground 的执行结果上看，key 并没有被删掉。
下面是个实验，测试设置什么值可以正常删除key-value

```
var dict :[String:String?] = [:]
// first try
dict = ["key": "value"]
dict["key"] = Optional<Optional<String>>.None
dict

// second try
dict = ["key": "value"]
dict["key"] = Optional<String>.None
dict

// third try
dict = ["key": "value"]
dict["key"] = nil
dict

// forth try
dict = ["key": "value"]
let nilValue:String? = nil
dict["key"] = nilValue
dict

// fifth try
dict = ["key": "value"]
let nilValue2:String?? = nil
dict["key"] = nilValue2
dict

```
运行结果
![result2](https://github.com/geekxing/myStudy/blob/master/resources/swifty_optional2.png?raw=true)
一个 [String: String?] 的 Dictionary，可以接受以下类型的赋值：

* nil
* String
* String?
* String??

由运行结果可知，如果要删除这个 Dictionary 中的元素，必须传入 nil 或 Optional\<Optional\<String>>.None ，而如果传入Optional\<String>.None，则不能正常删除元素。  
这是因为Dictionary的下标设值实现

```
  public subscript(key: Key) -> Value? {
    get {
      return _variantStorage.maybeGet(key)
    }
    set(newValue) {
      if let x = newValue {
        // FIXME(performance): this loads and discards the old value.
        _variantStorage.updateValue(x, forKey: key)
      }
      else {
        // FIXME(performance): this loads and discards the old value.
        removeValueForKey(key)
      }
    }
  }

```
此例中， Dictionary 的 value 类型为 String?，那么设置它的值需要传入一个String??类型参数。
如果传入Optional\<String>.None，是一个String?类型，

* 会执行一次Optional构造函数 转成Optional.Some(\<Optional\<String>.None>)
* 这时 if let 对它解包判断不为空，于是执行到 `_variantStorage.updateValue(x, forKey: key)`

### 内容摘自：[Swift 烧脑体操（一） - Optional 的嵌套](https://www.infoq.cn/article/swift-brain-gym-optional)


