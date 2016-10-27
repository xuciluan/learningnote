# MVP的P不需要绑定生命周期

参考自博客[Mosby的作者](http://hannesdorfmann.com/android/presenters-dont-need-lifecycle)

对于mvp来说，真正需要进行生命周期管理一般不是presenter，而是model和view，一般是view有生命周期，然后model中的逻辑在view不同的生命周期下有所改变。

举例，比如GPS轨迹模块，它有一个Activity是专门用于展示定位点，我们称他做`TrackerActvitiy`（`TrackerView`），之后需要一个`GpsTracker`用于定位检测，每次当有新的位置时反馈给`TrackingPresenter`，由`TrackingPresenter`来刷新界面。

那么当Activity处于`onPause`或者`onResume`状态的时候，需要调用`GPSTracker`的`onResume`或者`onPause`方法，那么是否应该调用`TrackingPresenter`的`onPause`或者`onResume`方法，然后由`TrackingPresenter`来调用`GPSTracker`的`onPause`和`onResume`方法呢？

其实，Mosby作者给出的解法是：

```java
class TrackingActivity extends MvpActivity implements TrackingView {
     private GpsTracker tracker;

     public onCreate(){
            tracker = new GpsTracker(this); // might need a context
            ...
     }

     public void onPause(){
         tracker.stop();
     }

    public void onResume(){
         tracker.start();
     }

    @Override
    public void createPresenter(){ // Called by Mosby
          return new TrackingPresenter(tracker);
     }
}
```

然后`TrackingPresenter`只有一个负责更新GPS定位点的职能：

```java
class TrackingPresenter extends MvpBasePresenter<TrackingView>
                        implements GpsUpdateListener{

  GpsTracker tracker;

  public TrackingPresenter(GpsTracker tracker){
    this.tracker = tracker;
    tracker.setGpsUpdateListener(this);
  }


  @Override
  public void onGpsLocationUpdated(GpsPosition position){
    view.showCurrentPosition(position.getLat(), position.getLng());
  }
}
```

在[LightCycle](https://github.com/soundcloud/lightcycle)开源项目中采用了依赖注入的方法来获取presenter：

```java
class TrackingActivity extends MvpActivity implements TrackingView {

    @Inject @LightCycle GpsTracker tracker;

    @Override
    public TrackingPresenter createPresenter(){
          return getObjectGraph.get(TrackingPresenter.class); // Dagger 1
          // or
          return getComponent().trackingPresenter(); // Dagger 2
     }
}
```

但是，`View`居然跟`Model`进行交互了，这显然是很危险的，对此`Mosby`的作者给出了这样的解决方案：

- 将View管理生命周期的职能交还给Activity

  简单来说，也就是Activity此时不再作为一个view了。

  举例：

  ```java
  class TrackingActivity extends MvpActivity<TrackignView, TrackingPresenter> {

     @Inject @LightCycle GpsTracker tracker;
     TrackingView view;

     public void onCreate(n){
       super.onCreate(b);
       setContentView(R.layout.activity_tracking);
       view = (TrackingView) findViewById(R.id.tracking_view);
     }

     @Override // called by Mosby, same as before
     public TrackingPresenter createPresenter(){ ... }

     @Override // called by Mosby to connect view with presenter
     public TrackingView getMvpView(){
       return view;
     }
  }
  ```

  这里是View可以是rootView，也可以是类似与`TrackingLayout extends FrameLayout implements TrackingView`的东西。

  这个方法的弊端是，多封装了一层，本来Activity就是View，但是现在可能就需要抽取出它的RootView然后进行封装。这可能就是`over-engineering`的开始。

  所以，这不是作者建议采纳的解决方案。

- ​