# var、let、const
var  
① 声明的变量往往会越域  
② 可以声明多次  

let  
① 声明的变量有严格局部作用域  
② 只能声明一次  

const  
① 声明之后不允许改变  
② 并且一旦声明，必须初始化，否则会报错  

# 解构表达式
①数组解构  

②对象解构  
```
// 变换变量名称
const {name:nameAlias, age, sex} = person;
console.log(nameAlias, age, sex)
```

# 对象的优化
① Object.keys(person)  
② Object.values(person)  
③ Object.entries(person)  
④ Object.assign(person)  

# 箭头函数
箭头函数不能使用this  
错误示例```param => this.name```  
正确示例```param => obj.name```

# 事件修饰符
```
<a href="www.baidu.com" @click.prevent.stop="hello">去百度</a>
```

# 按键修饰符
```
<input type="text" v-mode="num" @keyup.up="num+=2" @click.ctrl="num=10"/>
```

# v-for遍历
遍历数组  
```
<li v-for="(item,index) in items" :key="index"></li>
```

遍历对象  
```
<li v-for="(value,key,index) in obj" :key="index"></li>
```




