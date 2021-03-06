---
layout: post
title: "Flutter State中的生命周期小结"
date: 2021-4-21
description: "Flutter State的一些实践总结"
tag: Flutter 
--- 


1.initState，初始化调用，一个生命周期只调用一次.

2.didChangeDependencies，挂载到渲染树上时会调用，会在initState之后执行，和deactivate相对，这个didChangeDependencies需要与element里面的didChangeDependencies区分，element里面的didChangeDependencies会调用    markNeedsBuild();进而调用owner.scheduleBuildFor(this);导致渲染流水线的启动，也就是界面更新。此处的didChangeDependencies表示State的生命周期。在InheritedWidget中调用dependOnInheritedWidgetOfExactType的时候会把自身加载到找到的InheritedWidget的_dependents里面，在InheritedWidget刷新的时候，会循环调用_dependents的didChangeDependencies方法，此时调用的就是element的didChangeDependencies方法，而不是state的didChangeDependencies方法，但是在element的didChangeDependencies方法调用之后会触发渲染流水线，然后调用rebuild进而调用performRebuild，进而调用_state.didChangeDependencies();所以state的didChangeDependencies()方法最终还是会触发。

3.setstate后，如果widget被移除之后会先调用deactivate，然后调用dispose,等到再次被加进渲染树会调用initState和didChangeDependencies，

4.setstate后，如果widget被移除，不只自己会调用deactivate和dispose，自己子类的所有statefulWidget的state都会调用deactivate，调用顺序为，deactivate是从上到下调用，dispose是从下到上调用

5.setstate后，如果widget被重新挂载，不只自己会调用initState和didChangeDependencies，自己子类的所有statefulWidget的state都会调用initState和didChangeDependencies，调用顺序为从上到下

6.树的结构改变之后，比如A被移除，B被重新挂载，会先调用A以及A 所有child的deactivate，然后调用B以及B child的所有的initState和didChangeDependencies方法，然后调用A以及A 所有child的dispose

![](/images/flutter_state/flutter_state_1.png)

7.flutter在维护树的时候认准的是widget的runtimeType和key，runtimeType和key一样的话会复用state，所以这个时候就算widget变了，树的结构也没变，比如

![](/images/flutter_state/flutter_state_2.png)
这种写法，setstate后树的结构是没有变化的，也不会造成initstate和didChangeDependencies的调用，只会造成didUpdateWidget的调用deactivate，就是setstate之后如果widget被移除，就会造成deactivate的调用，并且很快会调用dispose

8.didUpdateWidget是在state之后，如果树的结构没变，此时state被复用了，但是widget重新构建了，此时会回调didUpdateWidget

9.Offstage的offstage如果为true或false，不会造成树的结构改变，此时还是只会回调didUpdateWidget，比如

![](/images/flutter_state/flutter_state_3.png)

![](/images/flutter_state/flutter_state_4.png)

这个时候setstate，isChangeView的值改变了，不会回调didChangeDependencies，只会回调didUpdateWidget


