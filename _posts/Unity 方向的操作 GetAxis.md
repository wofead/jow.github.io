# Unity 方向的操作 GetAxis

在Unity中，简单的操作可以通过GetAxis来获取输入，然后和游戏联动。

在游戏中，一般包括键盘操作和鼠标操作，所以我们把操作类型分为两个部分，分别是鼠标类（触屏类），这些都是通过传递参数给GetAxis来获取位置。



### 触屏类

1. Mouse X   鼠标沿着屏幕X移动时触发
2. Mouse Y 鼠标沿着屏幕Y移动时触发
3. Moses ScrollWheel 当鼠标滚动轮滚动时触发



### 键盘操作类

1. Vertical 对应键盘上面的上下箭头，当按下上或下（W或S）箭头时触发
2. Horizontal 对应键盘上面的左右箭头，当按下左或右（A或D）箭头时触发

---

这里还需要注意一点：

除了GetAxis之外还有一个函数GetAxisRaw函数，他们的参数一样，得到的返回值含义也近似，但是还是存在不同的。

**GetAxisRaw**返回值就是1.0或者-1.0，同时按下上下方向键的时候返回0.

**GetAxis**的返回值是从0变到1.0，如果再按下下方向键返回值从1.0变到-1.0，它是一个渐变的过程。

Input.GetAxis 当按下你设置的建则会返回一个类似加速度的值  0.1-->0.3 -->0.1然后将会依次减少。类似刹车和开车。