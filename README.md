# HyatteRegencyMUSIC
silight description for Android MusicMediaPlayer，as for its playing,pausing,next,prev,and playmode just as recyclelist,recycleOne,random.if you eager to setting your owener skins relativly,you must choose it!
package com.liteplayer.activity;


import com.liteplayer.service.DownloadService;
import com.liteplayer.service.PlayService;
import com.liteplayer.utils.L;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.support.v4.app.FragmentActivity;

/**
 * 服务的开启与中止
 * 服务的中止
 */
public abstract class BaseActivity extends FragmentActivity {
	
	protected PlayService mPlayService;
	protected DownloadService mDownloadService;
	private final String TAG = BaseActivity.class.getSimpleName();
	
	private ServiceConnection mPlayServiceConnection = new ServiceConnection() {
		@Override
		public void onServiceDisconnected(ComponentName name) {
			L.l(TAG, "play--->onServiceDisconnected");
			mPlayService = null;
		}
		
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			mPlayService = ((PlayService.PlayBinder) service).getService();
			mPlayService.setOnMusicEventListener(mMusicEventListener);
			onChange(mPlayService.getPlayingPosition());
		}
	};
	
	private ServiceConnection mDownloadServiceConnection = new ServiceConnection() {
		@Override
		public void onServiceDisconnected(ComponentName name) {
			L.l(TAG, "download--->onServiceDisconnected");
			mDownloadService = null;
		}
		
		
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			mDownloadService = ((DownloadService.DownloadBinder) service).getService();
		}
	};
	
	/**
	 * 音乐播放服务回调接口的实现类
	 */
	private PlayService.OnMusicEventListener mMusicEventListener = 
			new PlayService.OnMusicEventListener() {
		@Override
		public void onPublish(int progress) {
			BaseActivity.this.onPublish(progress);
		}

		@Override
		public void onChange(int position) {
			BaseActivity.this.onChange(position);
		}
	};
	
	/**
	 * Fragment的view加载完成后回调
	 * allowBindService()使用绑定的方式启动歌曲播放的服务
	 * allowUnbindService()方法解除绑定
	 * 
	 * 在SplashActivity.java启动音乐播放
	 * 
	 */
	public void allowBindService() {
		getApplicationContext().bindService(new Intent(this, PlayService.class),
				mPlayServiceConnection,
				Context.BIND_AUTO_CREATE);
	}
	
	/**
	 * 服务的中止项
	 */
	public void allowUnbindService() {
		getApplicationContext().unbindService(mPlayServiceConnection);
	}
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		//绑定下载服务
		bindService(new Intent(this, DownloadService.class),
				mDownloadServiceConnection,
				Context.BIND_AUTO_CREATE);
	}
	

	@Override
	protected void onDestroy() {
		unbindService(mDownloadServiceConnection);
		super.onDestroy();
	}

	
	public DownloadService getDownloadService() {
		return mDownloadService;
		
	}
	
	
	/**
	 * 实现service与主界面通信
	 * @param progress 进度
	 */
	public abstract void onPublish(int progress);
	/**
	 * 切换歌曲
	 * 实现service与主界面通信
	 * @param position 歌曲在list中的位置
	 */
	public abstract void onChange(int position);
}
