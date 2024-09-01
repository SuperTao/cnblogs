不能在对一个List进行遍历的时候将其中的元素删除掉

会报错java.util.ConcurrentModificationException

出错代码如下,在for循环中删除动态数组中的内容:
```
        for (moveBall x : ballShadow) {
            Log.e("51", "X ref:" + x.ballX + " ,Y ref:" + x.ballY + " ,alpha: " + x.alpha);
            x.alpha -= 30;
            if (x.alpha < 0) {
                ballShadow.remove(x);    //这条语句出错
            }
        }
```
解决方法：

1.使用index进行删除
```
        for (int i = 0; i < ballShadow.size(); i++) {
            ballShadow.get(i).alpha -= 30;
            if (ballShadow.get(i).alpha < 0) {
                ballShadow.remove(i);
            }
        }
```
2.使用迭代器删除
```
        ListIterator<moveBall> moveBallListIterator = ballShadow.listIterator();
        while (moveBallListIterator.hasNext()) {
            moveBall ball = moveBallListIterator.next();
            ball.alpha -= alphaLevel;
            if (ball.alpha < 0)
                moveBallListIterator.remove();
        }
```
3.在添加一个链表记录要删除的元素，不会出错，但最后结果和上面2种方法不同。
```
        List delList = new ArrayList();
        for (moveBall x : ballShadow) {
            x.alpha -= 30;
            if (x.alpha < 0) {
                delList.add(x);
            }
        }
        ballShadow.remove(delList);
```