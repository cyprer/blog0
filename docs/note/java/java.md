# java基础

## java基本类型
- byte 1B
- short 2B
- int 4B
- long 8B
- float 4B
- double 8B
- char 2B
- boolean ?
### 布尔值的大小
1. 作为局部变量时:  4B
  由于jvm没有专门的1bit/Byte字节指令操作boolean,因此会直接用int代替,1是true,0是false  
2. 作为布尔数组的元素时: 1B  
  jvm对布尔数组进行了优化,用byte存储,每个元素一字节  
3. 类成员变量: 1字节  
   属于对象内存布局,按字节对齐,最小一字节
4. Boolean包装类: 16B  
   对象头12字节,按8字节对齐(8的倍数),因此是16B.
> 像long和double的包装类就是24字节了,本身就是8字节了,加上对象头再对齐就是24字节