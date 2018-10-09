---
layout:     post
title:      Launcher3分析<一>
subtitle:   源码分析
date:       2018-09-19
author:     qkmin
header-img: img/post-bg-android.png
catalog: true
tags:
    - Android
    - Launcher
---
# 前言  
最近需要实现一个自己的Launcher，就借机学习下原生的Launcher源码。  
##多个Launcher
源码里有Launcher，Launcher2，Launcher3。那它们有什么区别呢。  
launcher不支持桌面小工具动画效果，launcher2添加了动画效果和3D初步效果支持。

Android 4.4 (KK)开始Launcher默认使用Launcher3，Launcher3较Launcher2 UI 有部分调整，主要包括:

*  状态栏透明，App List 透Wallpaper；
* 增加overview模式，可以调整workspace上页面的前后顺序；      
* 动态管理屏幕数量；       
* widget列表与app list分开显示；
*      默认不支持预置appwidget，需要用户指定权限；       
*       提供类似小米只有workspace的桌面机制 ；       
*       Wallpaper 的代码全部搬移到Launcher包；
*      类似Cling等细节的小变化；  
那我们就直接研究最新的Launcher3
#Launcher3  
## Launcher3介绍
Launcher是开机启动的第一个应用程序，用来展示应用列表和快捷方式、小部件等。
##Launcher3源码解析  
###AndroidManifest.xml  
一些标签属性
![123.png](https://upload-images.jianshu.io/upload_images/2607554-ee9755ecd74028f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### LauncherApplication   

	public class LauncherApplication extends Application {
	    @Override
	    public void onCreate() {
	        super.onCreate();
	        LauncherAppState.setApplicationContext(this);
	        LauncherAppState.getInstance();
	    }
	
	    @Override
	    public void onTerminate() {
	        super.onTerminate();
	        LauncherAppState.getInstance().onTerminate();
	    }
	}


初始化LauncherAppState类，在继续看LauncherAppState类。   

	private LauncherAppState() {
	      
	        // set sIsScreenXLarge and mScreenDensity *before* creating icon cache
	        mIsScreenLarge = isScreenLarge(sContext.getResources());
	        mScreenDensity = sContext.getResources().getDisplayMetrics().density;
	
	        mWidgetPreviewCacheDb = new WidgetPreviewLoader.CacheDb(sContext);
	        mIconCache = new IconCache(sContext);
	
	        mAppFilter = AppFilter.loadByName(sContext.getString(R.string.app_filter_class));
	        mModel = new LauncherModel(this, mIconCache, mAppFilter);
	
	        // Register intent receivers
	        IntentFilter filter = new IntentFilter(Intent.ACTION_PACKAGE_ADDED);
	        filter.addAction(Intent.ACTION_PACKAGE_REMOVED);
	        filter.addAction(Intent.ACTION_PACKAGE_CHANGED);
	        filter.addDataScheme("package");
	        sContext.registerReceiver(mModel, filter);
	        filter = new IntentFilter();
	        filter.addAction(Intent.ACTION_EXTERNAL_APPLICATIONS_AVAILABLE);
	        filter.addAction(Intent.ACTION_EXTERNAL_APPLICATIONS_UNAVAILABLE);
	        filter.addAction(Intent.ACTION_LOCALE_CHANGED);
	        filter.addAction(Intent.ACTION_CONFIGURATION_CHANGED);
	        sContext.registerReceiver(mModel, filter);
	        filter = new IntentFilter();
	        filter.addAction(SearchManager.INTENT_GLOBAL_SEARCH_ACTIVITY_CHANGED);
	        sContext.registerReceiver(mModel, filter);
	        filter = new IntentFilter();
	        filter.addAction(SearchManager.INTENT_ACTION_SEARCHABLES_CHANGED);
	        sContext.registerReceiver(mModel, filter);
	
	        // Register for changes to the favorites
	        ContentResolver resolver = sContext.getContentResolver();
	        resolver.registerContentObserver(LauncherSettings.Favorites.CONTENT_URI, true,
	                mFavoritesObserver);
	    }

   


* 初始化中读取配置，注册广播，实例化LauncherModel。Launcher2源码 LauncherModel创建是在LauncherApplication ,Launcher3移到这个类来了。
	
	
		  Maintains in-memory state of the Launcher. It is expected that there should be only one
	​	  LauncherModel object held in a static. Also provide APIs for updating the database state
	​	  for the Launcher.
	

单例，处理数据库。这个类继承BroadcastReceiver ，我们来重点看下这个类实现；
### LauncherModel  

	private static final HandlerThread sWorkerThread = new HandlerThread("launcher-loader");
	    static {
	        sWorkerThread.start();
	    }
	    private static final Handler sWorker = new Handler(sWorkerThread.getLooper());

创建一个线程，用来处理
LoaderTask  加载
*  workspace icons
* widgets
* all apps icons
 PackageUpdatedTask 用来更新applist.
看一下构造方法

	LauncherModel(LauncherAppState app, IconCache iconCache, AppFilter appFilter) {
	​        final Context context = app.getContext();
	
	        mAppsCanBeOnRemoveableStorage = Environment.isExternalStorageRemovable();
	        mApp = app;
	        mBgAllAppsList = new AllAppsList(iconCache, appFilter);
	        mIconCache = iconCache;
	    
	        mDefaultIcon = Utilities.createIconBitmap(
	                mIconCache.getFullResDefaultActivityIcon(), context);
	    
	        final Resources res = context.getResources();
	        Configuration config = res.getConfiguration();
	        mPreviousConfigMcc = config.mcc; //	SIM卡相关
	    }

传入LauncherAppState 来获取 context，WidgetPreviewLoader.CacheDb操作数据库；
IconCache类是用来缓存app图标
LauncherModle继承BroadcastReceiver 我们看下onReceive做了什么处理；

	 /**
	     * Call from the handler for ACTION_PACKAGE_ADDED, ACTION_PACKAGE_REMOVED and
	     * ACTION_PACKAGE_CHANGED.
	     */
	    @Override
	    public void onReceive(Context context, Intent intent) {
	        ...
	        if (Intent.ACTION_PACKAGE_CHANGED.equals(action)
	                || Intent.ACTION_PACKAGE_REMOVED.equals(action)
	                || Intent.ACTION_PACKAGE_ADDED.equals(action)) {
	            final String packageName = intent.getData().getSchemeSpecificPart();
	            final boolean replacing = intent.getBooleanExtra(Intent.EXTRA_REPLACING, false);
	
	            int op = PackageUpdatedTask.OP_NONE;
	
	            if (packageName == null || packageName.length() == 0) {
	                // they sent us a bad intent
	                return;
	            }
	
	            if (Intent.ACTION_PACKAGE_CHANGED.equals(action)) {
	                op = PackageUpdatedTask.OP_UPDATE;
	            } else if (Intent.ACTION_PACKAGE_REMOVED.equals(action)) {
	                if (!replacing) {
	                    op = PackageUpdatedTask.OP_REMOVE;
	                }
	                // else, we are replacing the package, so a PACKAGE_ADDED will be sent
	                // later, we will update the package at this time
	            } else if (Intent.ACTION_PACKAGE_ADDED.equals(action)) {
	                if (!replacing) {
	                    op = PackageUpdatedTask.OP_ADD;
	                } else {
	                    op = PackageUpdatedTask.OP_UPDATE;
	                }
	            }
	
	            if (op != PackageUpdatedTask.OP_NONE) {
	                enqueuePackageUpdated(new PackageUpdatedTask(op, new String[] { packageName }));
	            }
	          //移动APP完成之后，发出的广播
	        } else if (Intent.ACTION_EXTERNAL_APPLICATIONS_AVAILABLE.equals(action)) {
	            // First, schedule to add these apps back in.
	            String[] packages = intent.getStringArrayExtra(Intent.EXTRA_CHANGED_PACKAGE_LIST);
	            enqueuePackageUpdated(new PackageUpdatedTask(PackageUpdatedTask.OP_ADD, packages));
	            // Then, rebind everything.
	            startLoaderFromBackground();
	          //正在移动APP时，发出的广播
	        } else if (Intent.ACTION_EXTERNAL_APPLICATIONS_UNAVAILABLE.equals(action)) {
	            String[] packages = intent.getStringArrayExtra(Intent.EXTRA_CHANGED_PACKAGE_LIST);
	            enqueuePackageUpdated(new PackageUpdatedTask(
	                        PackageUpdatedTask.OP_UNAVAILABLE, packages));
	        } else if (Intent.ACTION_LOCALE_CHANGED.equals(action)) { //设备当前区域设置已更改时发出的广播
	            // If we have changed locale we need to clear out the labels in all apps/workspace.
	            forceReload();
	//设备当前设置被改变时发出的广播(包括的改变:界面语言，设备方向，等
	        } else if (Intent.ACTION_CONFIGURATION_CHANGED.equals(action)) {
	             // Check if configuration change was an mcc/mnc change which would affect app resources
	             // and we would need to clear out the labels in all apps/workspace. Same handling as
	             // above for ACTION_LOCALE_CHANGED
	             Configuration currentConfig = context.getResources().getConfiguration();
	             if (mPreviousConfigMcc != currentConfig.mcc) {
	                   Log.d(TAG, "Reload apps on config change. curr_mcc:"
	                       + currentConfig.mcc + " prevmcc:" + mPreviousConfigMcc);
	                   forceReload();
	             }
	             // Update previousConfig
	             mPreviousConfigMcc = currentConfig.mcc;
	        } else if (SearchManager.INTENT_GLOBAL_SEARCH_ACTIVITY_CHANGED.equals(action) ||
	                   SearchManager.INTENT_ACTION_SEARCHABLES_CHANGED.equals(action)) {
	            if (mCallbacks != null) {
	                Callbacks callbacks = mCallbacks.get();
	                if (callbacks != null) {
	                    callbacks.bindSearchablesChanged();
	                }
	            }
	        }
	    }

接收应用安装卸载后的广播，当接收到广播调用enqueuePackageUpdated来启动这个任务。

	public void run() {
	            final Context context = mApp.getContext();
	
	            final String[] packages = mPackages;
	            final int N = packages.length;
	            switch (mOp) {
	                case OP_ADD:
	                    for (int i=0; i<N; i++) {
	                        if (DEBUG_LOADERS) Log.d(TAG, "mAllAppsList.addPackage " + packages[i]);
	                        mBgAllAppsList.addPackage(context, packages[i]);
	                    }
	                    break;
	                case OP_UPDATE:
	                    for (int i=0; i<N; i++) {
	                        if (DEBUG_LOADERS) Log.d(TAG, "mAllAppsList.updatePackage " + packages[i]);
	                        mBgAllAppsList.updatePackage(context, packages[i]);
	                        WidgetPreviewLoader.removePackageFromDb(
	                                mApp.getWidgetPreviewCacheDb(), packages[i]);
	                    }
	                    break;
	                case OP_REMOVE:
	                case OP_UNAVAILABLE:
	                    for (int i=0; i<N; i++) {
	                        if (DEBUG_LOADERS) Log.d(TAG, "mAllAppsList.removePackage " + packages[i]);
	                        mBgAllAppsList.removePackage(packages[i]);
	                        WidgetPreviewLoader.removePackageFromDb(
	                                mApp.getWidgetPreviewCacheDb(), packages[i]);
	                    }
	                    break;
	            }
	            ....
	
	        if (added != null) {
	                // Ensure that we add all the workspace applications to the db
	                Callbacks cb = mCallbacks != null ? mCallbacks.get() : null;
	                if (!AppsCustomizePagedView.DISABLE_ALL_APPS) {
	                    addAndBindAddedApps(context, new ArrayList<ItemInfo>(), cb, added);
	                } else {
	                    final ArrayList<ItemInfo> addedInfos = new ArrayList<ItemInfo>(added);
	                    addAndBindAddedApps(context, addedInfos, cb, added);
	                }
	            }
	}

安装应用向AllAppsList添加应用信息；
卸载应用向AllAppsList 删除数据，并删除数据库的数据。
调用addAndBindAddedApps方法 :处理新添加的应用程序并首先将它们添加到数据库中;
```
callbacks.bindAppsAdded(addedWorkspaceScreensFinal,
                                        addNotAnimated, addAnimated, allAppsApps);
```
回调到Launcher 主Activity中，这个callback是Launcher初始化后调用的，我们后面在介绍。
定义
```
public interface Callbacks {
    //如果Launcher在加载完成之前被强制暂停，那么需要通过这个回调方法通知Launcher
    //在它再次显示的时候重新执行加载过程
    public boolean setLoadOnResume();
    //获取当前屏幕序号
    public int getCurrentWorkspaceScreen();
    //启动桌面数据绑定
    public void startBinding();
    //批量绑定桌面组件：快捷方式列表，列表的开始位置，列表结束的位置，是否使用动画
    public void bindItems(ArrayList<ItemInfo> shortcuts, int start, int end,
                          boolean forceAnimateIcons);
    //批量绑定桌面页，orderedScreenIds 序列化后的桌面页列表
    public void bindScreens(ArrayList<Long> orderedScreenIds);
    public void bindAddScreens(ArrayList<Long> orderedScreenIds);
    //批量绑定文件夹，folders 文件夹映射列表
    public void bindFolders(LongArrayMap<FolderInfo> folders);
    //绑定任务完成
    public void finishBindingItems();
    //批量绑定小部件，info 需要绑定到桌面上的小部件信息
    public void bindAppWidget(LauncherAppWidgetInfo info);
    //绑定应用程序列表界面的应用程序信息，apps 需要绑定到应用程序列表中的应用程序列表
    public void bindAllApplications(ArrayList<AppInfo> apps);
    // Add folders in all app list.
    public void bindAllApplications2Folder(ArrayList<AppInfo> apps, ArrayList<ItemInfo> items);
    //批量添加组件
    public void bindAppsAdded(ArrayList<Long> newScreens,
                              ArrayList<ItemInfo> addNotAnimated,
                              ArrayList<ItemInfo> addAnimated,
                              ArrayList<AppInfo> addedApps);
    //批量更新应用程序相关的快捷方式或者入口
    public void bindAppsUpdated(ArrayList<AppInfo> apps);
    public void bindShortcutsChanged(ArrayList<ShortcutInfo> updated,
            ArrayList<ShortcutInfo> removed, UserHandleCompat user);
    public void bindWidgetsRestored(ArrayList<LauncherAppWidgetInfo> widgets);
    public void bindRestoreItemsChange(HashSet<ItemInfo> updates);
    // 从桌面移除一些组件，当应用程序被移除或者禁用的时候调用安装应用向AllAppsList添加应用信息；
    public void bindComponentsRemoved(ArrayList<String> packageNames,
                    ArrayList<AppInfo> appInfos, UserHandleCompat user, int reason);
    public void bindAllPackages(WidgetsModel model);
    //全局搜索或者搜索属性更新
    public void bindSearchablesChanged();
    public boolean isAllAppsButtonRank(int rank);
    /**
     * 指示正在绑定的页面
     * @param page  桌面页序号
     */
    public void onPageBoundSynchronously(int page);
    //输出当前Launcher信息到本地文件中
    public void dumpLogsToLocalData();
}
```
接口都是在Launcher这个类实现的。

# 总结
* LauncherModel 是一个广播接收者BroadcastReceiver，监听应用的安装、卸载、移动等事件。
* 启动LoaderTask 初始化Launcher数据
# 赞助   
![](https://upload-images.jianshu.io/upload_images/2607554-4eb2f004840040e0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![TIM图片20180919152955.jpg](https://upload-images.jianshu.io/upload_images/2607554-421a527032b40030.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
