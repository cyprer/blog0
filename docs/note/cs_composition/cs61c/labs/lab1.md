- 学会了编译,gcc example.c ,这会生成一个a.out文件,然后./a.out就可以运行了
- 不同的是,gcc -o example example.c,这样就可以生成一个example可执行文件,然后./example就可以运行了,也就是说gcc -o 可以命名可执行文件的名字
- 还学了个重定向操作:
  就是   ./example < example.txt 
  就是将example.txt的内容作为example的输入