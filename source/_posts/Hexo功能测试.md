---
title: Hexo功能测试
date: 2024-05-07 13:55:07
description: 这个是一篇用来测试Hexo博客功能的文章
categories:
- Blog
tags:
- Hexo
- blog
mathjax: false
katex: true
---

# Hexo功能测试

## 引用

> 引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用
> 引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用
> 引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用引用

## 代码块

```c++ test.cpp
#include <iostream>
 
using namespace std;
 
// 基类
class Shape 
{
public:
   // 提供接口框架的纯虚函数
   virtual int getArea() = 0;
   void setWidth(int w)
   {
      width = w;
   }
   void setHeight(int h)
   {
      height = h;
   }
protected:
   int width;
   int height;
};
 
// 派生类
class Rectangle: public Shape
{
public:
   int getArea()
   { 
      return (width * height); 
   }
};
class Triangle: public Shape
{
public:
   int getArea()
   { 
      return (width * height)/2; 
   }
};
 
int main(void)
{
   Rectangle Rect;
   Triangle  Tri;
 
   Rect.setWidth(5);
   Rect.setHeight(7);
   // 输出对象的面积
   cout << "Total Rectangle area: " << Rect.getArea() << endl;
 
   Tri.setWidth(5);
   Tri.setHeight(7);
   // 输出对象的面积
   cout << "Total Triangle area: " << Tri.getArea() << endl; 
 
   return 0;
}
```

## 数学公式

$$
\begin{aligned}
    &f(x) = \int_{-\infty}^{\infty} e^{-x^2} dx \\
    &\sum_{n=1}^{\infty} \frac{1}{n^2} = \frac{\pi^2}{6} \\
    &\lim_{x \to \infty} \left(1 + \frac{1}{x}\right)^x = e
\end{aligned}
$$

## 图片测试

<img src="测试图片.png" style="max_width:50%">

