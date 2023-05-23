---
title: Android监听Home键
category: android
tags: [home key]
---

Android的Home比较特殊，不能像其他键(如返回键)那样直接用onKeyDown或onKeyUp来监听，需要用其他的方法来实现监听。
<!-- more -->

在Home键按下时，系统会发出一个广播，我们只需要注册一个Receiver来接收这个广播即可：
```java
public class HomeKeyReceiver extends BroadcastReceiver {
	private static final String TAG = HomeKeyReceiver.class.getSimpleName();
	
	/* extra from home key broadcase receiver */
	private static final String SYSTEM_DIALOG_REASON_EXTRA = "reason";
    /* press home key to go back to home */
    private static final String SYSTEM_DIALOG_REASON_HOME_KEY = "homekey";
	/* long press home key to show recent apps */
    private static final String SYSTEM_DIALOG_REASON_RECENT_APPS = "recentapps";
    /* (long) press home key to show assistant */
    private static final String SYSTEM_DIALOG_REASON_ASSIST = "assist";
    /* (long) press home key to lock */
    private static final String SYSTEM_DIALOG_REASON_LOCK = "lock";
	
	private Context mContext;
	/* listeners used in this broadcast receiver */
	private OnHomeKeyListener mOnHomeKeyListener;
	private OnRecentAppListener mOnRecentAppListener;
	private OnAssistListener mOnAssistListener;
	private OnLockListener mOnLockListener;
	
	public HomeKeyReceiver(Context context) {
		mContext = context;
	}
	
	/**
	 * Interface definition for a callback to be invoked when the reason is "homekey".
	 */
	public interface OnHomeKeyListener {
		void onKeypressed();
	}
	
	/**
	 * Interface definition for a callback to be invoked when the reason is "recentapps".
	 */
	public interface OnRecentAppListener {
		void onKeypressed();
	}
	
	/**
	 * Interface definition for a callback to be invoked when the reason is "assist".
	 */
	public interface OnAssistListener {
		void onKeypressed();
	}
	
	/**
	 * Interface definition for a callback to be invoked when the reason is "lock".
	 */
	public interface OnLockListener {
		void onKeypressed();
	}
	
	@Override
	public void onReceive(Context context, Intent intent) {
		if (intent.getAction().equals(Intent.ACTION_CLOSE_SYSTEM_DIALOGS)) {
			String reason = intent.getStringExtra(SYSTEM_DIALOG_REASON_EXTRA);
			if (TextUtils.isEmpty(reason)) {
				return;
			}
			
			switch (reason) {
			case SYSTEM_DIALOG_REASON_HOME_KEY:
				if (mOnHomeKeyListener != null) {
					mOnHomeKeyListener.onKeypressed();
				}
				break;
				
			case SYSTEM_DIALOG_REASON_RECENT_APPS:
				if (mOnRecentAppListener != null) {
					mOnRecentAppListener.onKeypressed();
				}
				break;
				
			case SYSTEM_DIALOG_REASON_ASSIST:
				if (mOnAssistListener != null) {
					mOnAssistListener.onKeypressed();
				}
				break;
				
			case SYSTEM_DIALOG_REASON_LOCK:
				if (mOnLockListener != null) {
					mOnLockListener.onKeypressed();
				}
				break;

			default:
				Log.d(TAG, "other reason: " + reason);
				break;
			}
		}
	}
	
	/**
	 * Register a callback to be invoked when system dialog reason is "homekey".
	 */
	public void setOnHomeKeyListener(OnHomeKeyListener l) {
		mOnHomeKeyListener = l;
	}
	
	/**
	 * Register a callback to be invoked when system dialog reason is "recentapps".
	 */
	public void setOnRecentAppListener(OnRecentAppListener l) {
		mOnRecentAppListener = l;
	}
	
	/**
	 * Register a callback to be invoked when system dialog reason is "assist".
	 */
	public void setOnAssistListener(OnAssistListener l) {
		mOnAssistListener = l;
	}
	
	/**
	 * Register a callback to be invoked when system dialog reason is "lock".
	 */
	public void setOnLockListener(OnLockListener l) {
		mOnLockListener = l;
	}

	/**
	 * Register the home key boradcasr receiver.
	 */
	public void register() {
		IntentFilter homeFilter = new IntentFilter(Intent.ACTION_CLOSE_SYSTEM_DIALOGS);
		mContext.registerReceiver(this, homeFilter);
	}
	
	/**
	 * Unregister the home key boradcasr receiver.
	 */
	public void unregister() {
		mContext.unregisterReceiver(this);
	}
```

由于home键在不同手机上，长/短按有不同的效果，代码列举出常用的几种，其中`homekey`表示短按home键退到桌面。

简单使用：
```java
private HomeKeyReceiver mHomeKeyReceiver;

@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);

	mHomeKeyReceiver = new HomeKeyReceiver(this);
	mHomeKeyReceiver.setOnHomeKeyListener(new OnHomeKeyListener() {
		@Override
		public void onKeypressed() {
			Log.d(TAG, "home key pressed");
		}
	});
}

@Override
protected void onResume() {
	super.onResume();
	mHomeKeyReceiver.register();
}

@Override
protected void onPause() {
	super.onPause();
	mHomeKeyReceiver.unregister();
}

```

注意：这个Receiver需要在`onResume`中注册，在`onPause`中注销。