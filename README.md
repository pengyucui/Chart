## 前言



因为目前看到公司设计图初稿有个饼图，一开始想用github上的一些开源的框架，但是感觉框架太全用到的只有一个饼图，并且自己撸一个也是一个学习进步的过程。如果认为还可以可否给个start。

先看下效果图：
![效果图](/img/效果图.gif)

## 绘制扇形

1、先确定圆的区域

```java
RectF mRect = new RectF(mCenterX - mOuterRadius, mCenterY - mOuterRadius, mCenterX + mOuterRadius, mCenterY + mOuterRadius);
```

2、创建画笔和路径

```java
Paint mPaint = new Paint();
Path mPath = new Path();
```

3、确定开始角度和当前当前路径所扫过的角度

```java
//开始角度
float mStartAngle = 0f;
//扫过的角度
float mSweepAngle = 0f;
```

4、开始绘制扇形

```java
//移动到中心点
mPath.moveTo(mCenterX, mCenterY);
//设置扇形路径
mPath.arcTo(mRect, mStartAngle, mSweepAngle);
//关闭路径
mPath.close();
//重新赋值起始位置
mStartAngle += mSweepAngle + 1f;
//绘制扇形
canvas.drawPath(mPath, mPaint);
```

## 绘制线段

先看下计算直线点的图，如下（画的有点丑，将就看吧-。-！）：

![画线的原理图](/img/画线的原理图.jpg)

我们要绘制直线需要起点和中点坐标，如图可以把他看做一个直角三角形，我们要计算stopX,stopY和startX,startY的点。

我们可以使用三角函数计算这四个点坐标(特意又百度重新学的，好几年不用忘脑后了-。-！)。`cosA=b/c`;我们要知道b的长度也就是`startx`的坐标，也就是`b=cosA*c；cosA`的角度就是`mStartAngle + mSweepAngle / 2`，同理个点坐标计算如下：

```java
//mOuterRadius大圆的半径
float startX = (float) ((mOuterRadius) * Math.cos(Math.toRadians(mStartAngle +mSweepAngle /2)));
float startY = (float) ((mOuterRadius) * Math.sin(Math.toRadians(mStartAngle + mSweepAngle / 2)));
//mPercentageLong先的长度
float stopX = (float) ((mOuterRadius+mPercentageLong)*Math.cos(Math.toRadians(mStartAngle +mSweepAngle / 2)));
float stopY = (float) ((mOuterRadius+mPercentageLong)*Math.sin(Math.toRadians(mStartAngle + mSweepAngle / 2)));
//绘制扇形中心点位置的直线
canvas.drawLine(mCenterX + startX, mCenterY + startY, mCenterX + stopX, mCenterY + stopY, mPaint);
```

当stopX大于0的时候是处于12象限，否则就是34象限。我们再绘制折线直线的方向如下：

```java
if (stopX > 0) {
//在 1   2象限
canvas.drawLine(mCenterX + stopX, mCenterY + stopY, mCenterX + stopX + mPercentageLong, mCenterY + stopY, mPaint);
canvas.drawText(mPercentageTv, mCenterX + stopX + mPercentageLong, mCenterY + stopY, mPaint);
} else {
//在  3  4象限
canvas.drawLine(mCenterX + stopX, mCenterY + stopY, mCenterX + stopX - mPercentageLong, mCenterY + stopY, mPaint);
canvas.drawText(mPercentageTv, mCenterX + stopX - mPercentageLong-mPaint.measureText(mPercentageTv), mCenterY + stopY, mPaint);}
```

## 点击事件

```java
private List<Region> mRegions;
//创建矩形
RectF rectF = new RectF();
//计算边界
mPath.computeBounds(rectF, true);
Region mRegion = new Region();
//设置区域
mRegion.set((int) rectF.left, (int) rectF.top, (int) rectF.right, (int) rectF.bottom);
//设置路径
mRegion.setPath(mPath, mRegion);
mRegions.add(mRegion);


@Override
public boolean onTouchEvent(MotionEvent event) {
      if (event.getAction() == MotionEvent.ACTION_DOWN ) {
        //获取点击位置
          float x = event.getX();
          float y = event.getY();
        //遍历
         for (int i = 0; i < mRegions.size(); i++) {
           //判断是否在点击区域
             if (mRegions.get(i).contains((int) x, (int) y)) {
                  //重绘
                  invalidate();
              } else {
                    mDatas.get(i).setCheck(false);
             }
           }
      }
        return super.onTouchEvent(event);
  }
```

**我简单封装了一下方便使用，如果你看完了，并且认为不错是否可以给个Start[项目地址：Chart](https://github.com/BryceCui/Chart)。如果有什么问题请联系我。**
