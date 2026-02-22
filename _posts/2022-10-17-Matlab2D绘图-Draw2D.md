---
layout: post
title: 'Matlab2D绘图'
subtitle: 'Draw2D'
date: 2022-10-17
categories: tech
author: yucol
cover: 'https://images.unsplash.com/photo-1529322365446-6efd62aed02e?w=1600&q=900'
cover_author: 'inma santiago'
cover_author_link: 'https://unsplash.com/@inmasantiago'

tags: Matlab
music-id: 461544126
---

```
x = [0:0.01:10];
y = cos(x)
plot(x,y),xlabel('x'),ylabel('cos(x)');
```

```
fplot('exp(-2*t)*sin(t)',[0, 4]),xlabel('t'),ylabel('f(t)'),title('阻尼弹力');
```

```
t = [0:0.02:4];
f = exp(-2.*t).*sin(t);% 多个函数相乘时必须是.*，不能是*
plot(t,f)
```

```
% 绘制网格
x = [-6:0.01:6];
y = tanh(x);
plot(x,y),grid on,axis equal
```

```
% 同一图像绘制多个函数
% 添加图例
t = [0:0.01:5];
f = exp(-t);
g = exp(-2*t);
plot(t,f,':',t,g,'--'),xlabel('t'),ylabel('value'),legend('f(t)','g(t)')
```

```
% 设置颜色
x = [-5:0.01:5];
y = sinh(x);
z = cosh(x);
plot(x,y,'r:',x,z,'b--');
```

```
% 设置坐标比例
x = [0:0.01:5];
y = sin(2*x+3);
plot(x,y),axis([0 5 -1 1])% axis ( [xmin xmax ymin ymax] )
y = exp(-3/2*x).*sin(5*x+3);
plot(x,y),axis([0,5,-1,1])
y = sin(5*x).^2; % sin函数的平方需要用.^
plot(x,y),axis([0,5,0,1])
```

```
% 子图
x = [0:0.01:5];
y = exp(-1.2*x).*sin(20*x);
subplot(1,2,1)
plot(x,y,'r-.'),xlabel('x'),ylabel('exp(–2x)*sin(20x)'),axis([0,5,-1,1])
subplot(1,2,2)
plot(x,y,'b'),xlabel('x'),ylabel('exp(–2x)*sin(20x)'),axis([0 5 -1 1])
```

```
% 图像重叠和linspace命令
x = linspace(a,b)会在a到b间取出均匀分布的100个点;
x = linspace(a,b,n)会在a、b之间取出均匀分布的n个点。
x = linspace(0,2*pi);
plot(x,cos(x)),axis([0 2*pi -1 1])
hold on
plot(x,sin(x)),axis([0 2*pi -1 1])
```

```
% 离散数据绘图
% 使用set命令添加标签
x = [1:5];
y = [50,98,75,80,98];
% ['Adrian'; 'Jim';'Joe';'Sally';'Sue'] = ['001';'002';'003';'004';'005'];
plot(x,y,'o',x,y),set(gca,'XTicklabel',['001'; '002';'003';'004';'005']), ... 
set(gca,'XTick',[1:5]),axis([1 5 0 100]),xlabel('学生'),ylabel('期末成绩'),title('2005 年 12 月期末考试')
% 二维条形图
x = [1:5]; 
y = [50,98,75,80,98]; 
bar(x,y),xlabel('学生'),ylabel('分数'),title('期末测试')
% 针状图
t = [0:5:200];
f = exp(-0.01*t).*sin(t/4);
stem(t,f),xlabel('时间(秒)'),ylabel('弹簧响应')
```

```
% 等高线图
[x,y] = meshgrid(-5:0.1:5,-3:0.1:3);
z = x.^2 + y.^2;
[C,h] = contour(x,y,z);
% 为曲线添加标签
set(h,'ShowText','on','TextStep',get(h,'LevelStep')*2)
% 使用contour3绘制三维等高线图
[x,y] = meshgrid(-2:0.1:2);
z = y.*exp(-x.^2 - y.^2);
contour3(x, y, z, 30)
% 使用surface美观
surface(x,y,z,'EdgeColor',[.8 .8 .8],'FaceColor','none') 
grid off
view(-15,20)
```

![](https://lz.sinaimg.cn/nmw690/ebeef3aaly3h8bl9o4hqjj20w11kw7n0.jpg)