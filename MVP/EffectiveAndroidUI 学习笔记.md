# EffectiveAndroidUI 学习笔记

## 1. 引入插件checkstyle、findBugs、pmd

## 2. 关于Fragment的静态初始化

之前一直使用的是动态初始化的Fragment，只知道可以通过`beginTransition()`方法来动态添加和删除Fragment，这次看项目的时候发现，原来可以在代码中直接指定Fragment的，只需要在xml文件中加入`class`属性指定即可，这样当activity初始化的时候，fragment也会自动被初始化。

布局文件：

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="MergeRootFrame"
    tools:context="com.github.pedrovgs.effectiveandroidui.ui.activity.MainActivity">

  <!-- TvShowCatalogFragment -->

  <fragment
      android:id="@+id/f_tv_show_catalog"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      class="com.github.pedrovgs.effectiveandroidui.ui.fragment.TvShowCatalogFragment"
      tools:context="com.github.pedrovgs.effectiveandroidui.ui.activity.MainActivity"/>

</FrameLayout>
```

然后在`TvShowCatalogFragment`的`onViewCreated`方法中加入布局：

```java
 @Override public View onCreateView(LayoutInflater inflater, ViewGroup container,
      Bundle savedInstanceState) {
    return inflater.inflate(R.layout.fr_tvshow, container, false);
  }
```

这样就完成了一个Fragment的动态加载。

## 3. 两个Fragment之间是如何进行交互的

在这个项目中，是使用`Navigator`来进行交互的。

关于点击事件的实现，是放在`TvShowCatalogPresenter`中的：

```java
 public void onTvShowThumbnailClicked(final TvShow tvShow) {
      navigator.openTvShowDetails(tvShow);
  }
```

关于`Navigator`的定义简单来说就是：

在这个类中保存了`TvShowDrawggableFragment`（也就是之后详情界面的Fragment）的实例如图：

```java
 private TvShowDraggableFragment tvShowDraggableFragment;
```

之后`openTvShowDetails()`方法如下：

```java
   public void openTvShowDetails(TvShow tvShow) {

        if (canInteractWithFragments()) {
            showTvShowOnTvShowDraggableFragment(tvShow);
            showTvShowOnTvShowFragment(tvShow);
        } else {
            openTvShowActivity(tvShow.getTitle());
        }
    }
```

这里有一个很好的要借鉴的地方，就是在这里通过代码，实现了平板和手机的不同实现：

```java
private boolean canInteractWithFragments() {
        //这是平板界面上的fragment的id是f_tv_show
        tvShowFragment = (TvShowFragment) getFragmentManager().findFragmentById(R.id.f_tv_show);
        //这是手机界面上的fragment的id，叫f_tv_show_draggable
        tvShowDraggableFragment =
                (TvShowDraggableFragment) getFragmentManager().findFragmentById(R.id.f_tv_show_draggable);
        //只要两者有其中一个存在，就可以显示影片详情
        return tvShowDraggableFragment != null || tvShowFragment != null;
    }
```

然后：

```java
   //手机详情界面
private void showTvShowOnTvShowDraggableFragment(TvShow tvShow) {
        if (isFragmentAvailable(tvShowDraggableFragment)) {
            tvShowDraggableFragment.showTvShow(tvShow.getTitle());
        }
    }

//平板
    private void showTvShowOnTvShowFragment(TvShow tvShow) {
        if (isFragmentAvailable(tvShowFragment)) {
            tvShowFragment.showTvShow(tvShow.getTitle());
        }
    }

 private boolean isFragmentAvailable(Fragment fragment) {
        return fragment != null && fragment.isAdded();
    }
```

## 4. 后台任务的执行

首先，执行后台程序的接口是这样定义的：

```java
public interface GetTvShows {

  interface Callback {
    void onTvShowsLoaded(final Collection<TvShow> tvShows);

    void onConnectionError();
  }

  void execute(Callback callback);
}
```

然后在程序中需要进行网络请求数据的地方这样写：

```java
 private GetTvShows getTvShowsInteractor;
private void loadTvShows() {
    if (view.isReady()) {
      view.showLoading();
    }
    getTvShowsInteractor.execute(new GetTvShows.Callback() {
      @Override public void onTvShowsLoaded(final Collection<TvShow> tvShows) {
        //请求成功要做的事
      }

      @Override public void onConnectionError() {
        //网络连接失败要做的事
      }
    });
  }
```

那么，它是怎么一个`execute`就完成请求的呢？

看他的实现类:

```java
class GetTvShowsInteractor implements Interactor, GetTvShows {

  private static final int PERCENTAGE_OF_FAILS = 50;
  public static final int WAIT_TIME = 1500;

  private final Catalog catalog;
  private final Executor executor;
  private final MainThread mainThread;

  private Callback callback;

  @Inject GetTvShowsInteractor(Catalog catalog, Executor executor, MainThread mainThread) {
    this.catalog = catalog;
    this.executor = executor;
    this.mainThread = mainThread;
  }

  @Override public void execute(final Callback callback) {
    if (callback == null) {
      throw new IllegalArgumentException(
          "Callback can't be null, the client of this interactor needs to get the response "
              + "in the callback");
    }
    this.callback = callback;
    this.executor.run(this);
  }

  @Override public void run() {
    waitToDoThisSampleMoreInteresting();

    if (haveToShowError()) {
      notifyError();
    } else {
      Collection<TvShow> tvShows = catalog.getTvShows();
      nofityTvShowsLoaded(tvShows);
    }
  }

  /**
   * To simulate a we are getting the TvShows data from internet we are going to force a 1.5
   * seconds
   * delay using Thread.sleep.
   */
  private void waitToDoThisSampleMoreInteresting() {
    try {
      Thread.sleep(WAIT_TIME);
    } catch (InterruptedException e) {
      //Empty
    }
  }

  private boolean haveToShowError() {
    return RandomUtils.percent(PERCENTAGE_OF_FAILS);
  }

  private void notifyError() {
    mainThread.post(new Runnable() {
      @Override public void run() {
        callback.onConnectionError();
      }
    });
  }

  private void nofityTvShowsLoaded(final Collection<TvShow> tvShows) {
    mainThread.post(new Runnable() {
      @Override public void run() {
        callback.onTvShowsLoaded(tvShows);
      }
    });
  }
}
```

`Interactor`：

```java
public interface Interactor {
  void run();
}
```

`Executor`:

```java
public interface Executor {

  void run(final Interactor interactor);
}
```

它的默认实现类：

```java
class ThreadExecutor implements Executor {

  private static final int CORE_POOL_SIZE = 3;
  private static final int MAX_POOL_SIZE = 5;
  private static final int KEEP_ALIVE_TIME = 120;
  private static final TimeUnit TIME_UNIT = TimeUnit.SECONDS;
  private static final BlockingQueue<Runnable> WORK_QUEUE = new LinkedBlockingQueue<Runnable>();

  private ThreadPoolExecutor threadPoolExecutor;

  @Inject ThreadExecutor() {
    int corePoolSize = CORE_POOL_SIZE;
    int maxPoolSize = MAX_POOL_SIZE;
    int keepAliveTime = KEEP_ALIVE_TIME;
    TimeUnit timeUnit = TIME_UNIT;
    BlockingQueue<Runnable> workQueue = WORK_QUEUE;
    threadPoolExecutor =
        new ThreadPoolExecutor(corePoolSize, maxPoolSize, keepAliveTime, timeUnit, workQueue);
  }

  @Override
  public void run(final Interactor interactor) {
    if (interactor == null) {
      throw new IllegalArgumentException("Interactor to execute can't be null");
    }
    threadPoolExecutor.submit(new Runnable() {
      @Override public void run() {
        interactor.run();
      }
    });
  }
}
```

`MainThread`：

```java
public interface MainThread {

  void post(final Runnable runnable);
}
```

它的默认实现类`MainThreadImpl`：

```java
class MainThreadImpl implements MainThread {

  private Handler handler;

  @Inject MainThreadImpl() {
    this.handler = new Handler(Looper.getMainLooper());
  }

  public void post(Runnable runnable) {
    handler.post(runnable);
  }
}
```

> 思考如何跟RxJava结合，共用一个接口

## 5.如何实现平板和手机的适配工作





## 6. 如何使用Dagger动态注入presenter、context等对象

