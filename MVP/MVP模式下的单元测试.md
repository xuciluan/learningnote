# MVP模式下的单元测试

MVP各层所使用的单元测试框架如下：

 ![mvp](E:\学习笔记\图片\mvp.PNG)

- P层： 不需要任何Android环境，因此使用Junit测试即可。

- V层：使用Google的Espresso进行UI测试。

- M层：设计到数据库相关操作，因此需要依赖Android环境，使用AndroidJUnitRunner进行测试。

  以下以Google公司提供的todo-sample为例子：

  ​