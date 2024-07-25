### Javascript的数据类型
#### 基本类型
* string：字符串由单引号和双引号定义（与其他语言不同，不区分单双引号）
* number：表示任何数字类型（与其他语言不同，不区分浮点数与整形）
* boolean：布尔值，表示true或者false的类型
* null：表示变量为空
* undefined：表示变量未定义，通常用于比较不存在的属性或值

#### 引用类型
* Object：以`key-value`键值对的形式存在，`key`是字符串类型，而`value`可以是任意类型
* Array：有序排列的集合，数组的值可以是任意类型

#### ES6新增类型
Map和Set类型是ES6中新增的类型，以解决ES5中Object和Array不足
* Map：键值对关系的集合，不同的是`key`可以是任意类型
* Set：不重复数值的数组

### Object对象的深浅拷贝
* 浅拷贝：只复制对象的内存地址，类似于指针
* 深拷贝：完全克隆，生成一个新的对象

### 特殊对象
* JSON
    * JSON.stringify 序列化
    * JSON.parse 反序列化
* Date
