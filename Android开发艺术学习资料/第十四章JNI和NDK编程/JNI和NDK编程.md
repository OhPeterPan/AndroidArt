JNI Java Native Interface java本地接口，方便java调用C、C++等本地代码所封装的一层接口。通过JNI，用户可以调用C、C++所编写的本地代码

NDK是Android创建的一个工具集合，通过NDK可以在Android中更加方便的通过JNI来访问本地代码，比如C或者C++，提供交叉编译器，简单的修改mk文件就可以生成特定CPU平台的动态库。

使用NDK的好处
	提高代码的安全性、方便使用已有的c/c++开源库、便于平台间的移植、提高程序在某些特定情形下的执行效率，但是并不能明显提升Android程序的性能