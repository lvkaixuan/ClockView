# ClockView
Android自定义View之会走动的时钟
效果图如下: 

<img src="https://user-gold-cdn.xitu.io/2019/10/25/16e01db9a70b32fc?w=311&h=640&f=gif&s=1472292" width="155" height="320">

## 绘制流程: 

* 绘制表盘
* 绘制刻度
* 绘制数字
* 旋转数字
* 绘制时分秒指针
* 使用Handler定时刷新指针

## 接下来一步一步实现:

* ### 绘制表盘

``` java
canvas.drawCircle(mWidth / 2, mHeight / 2, mHeight / 2 - dp2px(borderPadding), mPaint);
```
<img src="https://user-gold-cdn.xitu.io/2019/10/25/16e01e2727b886d4?w=414&h=364&f=png&s=22110" width="207" height="182">

* ### 绘制刻度
``` java
        //绘制刻度,每次绘制完需要旋转一定的角度,然后继续绘制
        for (int i = 0; i < 60; i++) {
            if (i % 5 == 0) { //大刻度
                mPaint.setStrokeWidth(dp2px(3));
                canvas.drawLine(mWidth / 2, dp2px(borderPadding), mWidth / 2, dp2px(23), mPaint);
            } else { //小刻度
                mPaint.setStrokeWidth(dp2px(1));
                canvas.drawLine(mWidth / 2, dp2px(borderPadding), mWidth / 2, dp2px(20), mPaint);
            }
            //一共绘制60个刻度,每次旋转360°/60
            canvas.rotate(360 / 60, mWidth / 2, mHeight / 2);
        }
```

<img src="https://user-gold-cdn.xitu.io/2019/10/25/16e01e639353af5e" width="207" height="207">

* ### 绘制数字
```java
        //绘制刻度数字
        for (int i = 0; i < 12; i++) {
            //绘制数字,一共12个
            canvas.drawText(String.valueOf(i==0?"12":i), mWidth / 2, dp2px(textPadding), mPaint);
        }
```
<img src="https://user-gold-cdn.xitu.io/2019/10/25/16e01ea1fa03bd8d?w=370&h=370&f=png&s=31537" width="207" height="207">

* ### 旋转数字
```!
这里为什么要旋转数字呢?有人可能会注意到数字随着角度旋转了,这样很不友好,所以我们需要把数字都"正"过来
```

<img src="https://user-gold-cdn.xitu.io/2019/10/25/16e01f32b6a6d3bb?w=880&h=736&f=png&s=124075" width="440" height="368">
```java
        //绘制刻度数字
        for (int i = 0; i < 12; i++) {
            //保存当前画布状态
            canvas.save();
            //将要旋转的中心点定位到数字上,旋转的角度取决于当前数字倾斜的角度
            canvas.rotate(-360/12*i, mWidth / 2, dp2px(textPadding)- mPaint.measureText(String.valueOf(i==0?"12":i)) / 2);
            //绘制数字
            canvas.drawText(String.valueOf(i==0?"12":i), mWidth / 2, dp2px(textPadding), mPaint);
            //恢复被旋转的画布
            canvas.restore();
            //绕表盘中心旋转,绘制下一个刻度
            canvas.rotate(360 / 12, mWidth / 2, mHeight / 2);
        }
```

#### 图中可以看到,刻度上的数字已经都"正"过来了

![](https://user-gold-cdn.xitu.io/2019/10/25/16e01fcd177e3258?w=1202&h=630&f=png&s=234248)

* ### 绘制时分秒指针
```java
        //绘制秒针
        mPaint.setStrokeWidth(dp2px(1));
        mPaint.setColor(Color.BLACK);
        canvas.save();
        canvas.rotate(second * (365 / 60), mWidth / 2, mHeight / 2);
        canvas.drawLine(mWidth / 2, dp2px(textPadding) + dp2px(10), mWidth / 2, mHeight / 2 + dp2px(20), mPaint);
        canvas.restore();
        //绘制分针
        mPaint.setStrokeWidth(dp2px(3));
        mPaint.setColor(Color.BLACK);
        canvas.save();
        canvas.rotate((minute + (float) second / 60) * 360 / 60, mWidth / 2, mHeight / 2);
        canvas.drawLine(mWidth / 2, dp2px(textPadding) + dp2px(20), mWidth / 2, mHeight / 2 + dp2px(20), mPaint);
        canvas.restore();
        //绘制时针
        mPaint.setStrokeWidth(dp2px(5));
        mPaint.setColor(Color.BLACK);
        canvas.save();
        canvas.rotate((hour + (float) minute / 60) * 360 / 12, mWidth / 2, mHeight / 2);
        canvas.drawLine(mWidth / 2, dp2px(textPadding) + dp2px(50), mWidth / 2, mHeight / 2 + dp2px(20), mPaint);
        canvas.restore();
        //绘制中心圆点
        mPaint.setColor(Color.RED);
        mPaint.setStyle(Paint.Style.FILL);
        canvas.drawCircle(mWidth / 2, mHeight / 2, dp2px(5), mPaint);
```

<img src="https://user-gold-cdn.xitu.io/2019/10/25/16e0202852485bcc?w=560&h=472&f=png&s=49241" width="280" height="236">

* ### 使用Handler定时刷新指针
```!
上面绘制时分秒指针的时候已经在代码逻辑中加入根据时间来绘制不同的角度,这里我们只需要每隔一秒重新获取一次系统时间,并调用invalidate()方法重新绘制界面,时钟就可以"动"起来了
```
```java
    private Handler mHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                mCalendar = Calendar.getInstance();
                //重新绘制页面
                invalidate();
                //1000ms后再刷新一次页面
                sendEmptyMessageDelayed(0, 1000);
            }
        };
```
```java
        //获取当前时间
        int second = mCalendar.get(Calendar.SECOND);
        int hour = mCalendar.get(Calendar.HOUR);
        int minute = mCalendar.get(Calendar.MINUTE);
        ...
        //绘制时分秒指针
        ...
```

<img src="https://user-gold-cdn.xitu.io/2019/10/25/16e021f9edf54844?w=640&h=571&f=gif&s=3625146" width="320" height="285.5">

* ### 扫码体验Demo

<img src="https://user-gold-cdn.xitu.io/2019/10/25/16e02258b2b4d8b0?w=384&h=372&f=png&s=19903" width="250" height="250">

* ### 欢迎star
