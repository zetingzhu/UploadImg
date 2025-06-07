Android ExoPlayer 文档

官网文档地址

https://developer.android.google.cn/guide/topics/media/exoplayer/hello-world?hl=zh-cn

项目使用地址 xtrend speed

https://codes.51jiawawa.com/FX/XtrendSF/-/tree/v1.8.0-ExoPlayer


# 添加依赖
```Java
// 主要
implementation "androidx.media3:media3-exoplayer:1.2.0"
implementation "androidx.media3:media3-exoplayer-dash:1.2.0"
implementation "androidx.media3:media3-ui:1.2.0"
// hls 协议
implementation "androidx.media3:media3-exoplayer-hls:1.2.0"
// trmp 协议
implementation "androidx.media3:media3-datasource-rtmp:1.2.0"
```

# 编译 SDK 最低34

```java
android {
    compileSdk 34
}
```

# PlayerView 描述
下方自定义的播放器视图控件

id | 描述 | 使用方法
--|--|--
exo_ad_overlay，exo_overlay | 广告图层 |
exo_shutter | 播放器未初始化背景颜色 | app:shutter_background_color="#D31313"
exo_artwork | 没有视频时候默认图片 | 未显示过，不知到如何使用，
exo_buffering | 缓冲进度条|app:show_buffering="always"
exo_error_message | 错误信息| setCustomErrorMessage()


exo_artwork 使用

<font color=red>未测试成功，展示失败</font>
```java
app:artwork_display_mode="fit"
app:default_artwork="@drawable/exo_styled_controls_default"

playerView.setDefaultArtwork(ContextCompat.getDrawable(this, R.drawable.exo_styled_controls_default));
playerView.setArtworkDisplayMode(PlayerView.ARTWORK_DISPLAY_MODE_FILL);

```

show_buffering 默认进度条展示

```
app:show_buffering="always"
<enum name="never" value="0"/>
<enum name="when_playing" value="1"/>
<enum name="always" value="2"/>
```


# PlayerControlView 视频控制器描述
下方自定义控制器需要对应视图id

id | 内容描述 | 使用方法
-- | -- | --
exo_position | 当前视频播放时间 |
exo_duration | 当前视频总时长 |
exo_vr | 是否显示 vr 按钮 | app:show_vr_button="true"
exo_shuffle | 随机播放按钮 | app:show_shuffle_button="true"
exo_repeat_toggle | 单一播放，重复播放按钮 | app:repeat_toggle_modes="one"，app:repeat_toggle_modes="all"，app:repeat_toggle_modes="none"
exo_subtitle | 字幕按钮 | app:show_subtitle_button="true"
exo_settings | 设置按钮，速度设置，音频设置 |
exo_fullscreen | 全屏切换按钮 |
exo_minimal_fullscreen | 最小全屏 |
exo_next | 下一个视频 |
exo_prev | 上一个视频 |
exo_ffwd_with_amount | 快进按钮 |
exo_rew_with_amount | 快退按钮 |

- exo_fullscreen，exo_minimal_fullscreen | 全屏切换按钮

设置显示全屏切换按钮状态
全屏两个按钮id,新图片替换这两个id

> exo_styled_controls_fullscreen_enter.png
>
> exo_styled_controls_fullscreen_exit.png

```java
playerView.setFullscreenButtonClickListener(new PlayerView.FullscreenButtonClickListener() {
    @Override
    public void onFullscreenButtonClick(boolean isFullScreen) {

    }
});
```

播放暂停按钮ui替换

> exo_styled_controls_pause.png
>
> exo_styled_controls_play.png

# 自定义视频页面基本设置

1. 自定义播放器ui exo_player_view_v2.xml

```HTML
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <androidx.media3.ui.AspectRatioFrameLayout
        android:id="@id/exo_content_frame"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="center">
        <!--视频展示-->
        <androidx.media3.ui.SubtitleView
            android:id="@id/exo_subtitles"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
        <!--缓冲进度-->
        <ProgressBar
            android:id="@id/exo_buffering"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:indeterminate="true" />
        <!--视频绑定控制器-->
        <androidx.media3.ui.PlayerControlView
            android:id="@id/exo_controller"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:controller_layout_id="@layout/exo_player_control_view_v2" />
    </androidx.media3.ui.AspectRatioFrameLayout>
</merge>

```

2. 自定义控制器ui exo_player_control_view_v2.xml

```HTML
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 0dp dimensions are used to prevent this view from influencing the size of
         the parent view if it uses "wrap_content". It is expanded to occupy the
         entirety of the parent in code, after the parent's size has been
         determined. See: https://github.com/google/ExoPlayer/issues/8726.
    -->
    <View
        android:id="@id/exo_controls_background"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:background="@color/exo_black_opacity_60" />

    <FrameLayout
        android:id="@id/exo_bottom_bar"
        android:layout_width="match_parent"
        android:layout_height="@dimen/exo_styled_bottom_bar_height"
        android:layout_gravity="bottom"
        android:layout_marginTop="@dimen/exo_styled_bottom_bar_margin_top"
        android:background="@color/exo_bottom_bar_background"
        android:layoutDirection="ltr">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical|start"
            android:gravity="center_vertical"
            android:layoutDirection="ltr"
            android:paddingStart="@dimen/exo_styled_bottom_bar_time_padding"
            android:paddingLeft="@dimen/exo_styled_bottom_bar_time_padding"
            android:paddingEnd="@dimen/exo_styled_bottom_bar_time_padding"
            android:paddingRight="@dimen/exo_styled_bottom_bar_time_padding">

            <!--播放暂停按钮-->
            <ImageButton
                android:id="@id/exo_play_pause"
                style="@style/ExoStyledControls.Button.Center.PlayPause"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <!--播放时间-->
            <TextView
                android:id="@id/exo_position"
                style="@style/ExoStyledControls.TimeText.Position"
                android:layout_width="wrap_content"
                android:layout_height="match_parent"
                android:layout_weight="0" />
            <!--拖拽进度条-->
            <androidx.media3.ui.DefaultTimeBar
                android:id="@id/exo_progress"
                android:layout_width="0dp"
                android:layout_height="match_parent"
                android:layout_weight="1" />
            <!--视频时长-->
            <TextView
                android:id="@id/exo_duration"
                style="@style/ExoStyledControls.TimeText.Duration"
                android:layout_width="wrap_content"
                android:layout_height="match_parent"
                android:layout_weight="0" />

            <!--全屏按钮-->
            <ImageButton
                android:id="@id/exo_fullscreen"
                style="@style/ExoStyledControls.Button.Bottom.FullScreen" />

        </LinearLayout>
    </FrameLayout>
</merge>
```

# 视频控制播放器工具类

1. 简单使用,直接绑定声明周期使用

```java
// 对象创建
exoPlayerUtil = new ExoPlayerUtil(getActivity(), playerView);
// 声明周期绑定
getLifecycle().addObserver(exoPlayerUtil);
// 播放媒体创建
MediaItem mediaItem3 = MediaItem.fromUri("视频地址");
// 设置播放源，
exoPlayerUtil.setCurrentMediaItem(mediaItem3);

// 播放修改播放地址方法
setCurrentMediaItemToPlayer(String url)
setCurrentMediaItemToPlayer(Uri url) {
setCurrentMediaItemToPlayer(MediaItem currentMediaItem)

```

2. 不绑定生命周期

```java
exoPlayerUtil = new ExoPlayerUtil(getActivity(), playerView);
MediaItem mediaItem3 = new MediaItem.Builder().build();
exoPlayerUtil.setCurrentMediaItem(mediaItem3);
分别实现下方四个声明周期方法
exoPlayerUtil.onStart();
exoPlayerUtil.onResume();
exoPlayerUtil.onPause();
exoPlayerUtil.onStop();

下面用于直接控制视图，并且不绑定声明周期方法
// 释放资源
exoPlayerUtil.releaseResource()
// 重新播放
exoPlayerUtil.replay（）
```

3. 控制视频的播放和暂停

```java
// 播放
exoPlayerUtil.setPlayAndPause(true);
// 暂停
exoPlayerUtil.setPlayAndPause(false);

```
4. 播放本地文件

```java
public void onCreate(@Nullable Bundle savedInstanceState) {
......
// 1.定义 ResultLauncher
videoLocalFilePickerLauncher = registerForActivityResult(new ActivityResultContracts.StartActivityForResult(), this::videoLocalFilePickerLauncherResult);
.......
}

// 2. 权限判断
private void selectLocalFile(ActivityResultLauncher<Intent> localFilePickerLauncher, String mimeType) {
        String permission = Build.VERSION.SDK_INT >= 33 ? READ_MEDIA_VIDEO : READ_EXTERNAL_STORAGE;
        if (ActivityCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, new String[]{permission}, FILE_PERMISSION_REQUEST_CODE);
        } else {
            launchLocalFilePicker(localFilePickerLauncher, mimeType);
        }
    }
// 3. 权限处理，授权成功打开文件选择器
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
super.onRequestPermissionsResult(requestCode, permissions, grantResults);
  if (requestCode == FILE_PERMISSION_REQUEST_CODE && grantResults.length == 1 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
      launchLocalFilePicker(videoLocalFilePickerLauncher, "video/*");
  }
}
// 3. 打开文件选择器
private void launchLocalFilePicker(ActivityResultLauncher<Intent> localFilePickerLauncher, String mimeType) {
    Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
    intent.setType(mimeType);
    localFilePickerLauncher.launch(intent);
}
// 4 处理文件返回结果，播放
private void videoLocalFilePickerLauncherResult(ActivityResult result) {
    Intent data = result.getData();
    if (data != null) {
        localFileUri = data.getData();
        Log.d(TAG, "本地url:" + localFileUri);
        MediaItem mediaItem = MediaItem.fromUri(localFileUri);
        exoPlayerUtil.setCurrentMediaItemToPlayer(mediaItem);
    }
}

```

5. 直播播放

```java
exoPlayerUtil = new ExoPlayerUtil(getActivity(), playerView);
getLifecycle().addObserver(exoPlayerUtil);
MediaItem mediaItem5 = MediaItem.fromUri("直播地址");
exoPlayerUtil.setCurrentMediaItemToPlayer(mediaItem);
```


ExoPlayerUtil 工具代码

```Java
package com.exoplayer.util;

import android.content.Context;
import android.net.Uri;
import android.os.Build;
import android.text.TextUtils;
import android.util.Log;
import android.view.KeyEvent;
import android.view.View;
import android.widget.ImageView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.annotation.OptIn;
import androidx.lifecycle.DefaultLifecycleObserver;
import androidx.lifecycle.LifecycleOwner;
import androidx.media3.common.AudioAttributes;
import androidx.media3.common.C;
import androidx.media3.common.MediaItem;
import androidx.media3.common.PlaybackException;
import androidx.media3.common.Player;
import androidx.media3.common.Timeline;
import androidx.media3.common.util.UnstableApi;
import androidx.media3.common.util.Util;
import androidx.media3.exoplayer.DefaultLivePlaybackSpeedControl;
import androidx.media3.exoplayer.ExoPlayer;
import androidx.media3.exoplayer.drm.FrameworkMediaDrm;
import androidx.media3.exoplayer.util.EventLogger;
import androidx.media3.ui.PlayerView;

import com.exoplayer.R;
import com.exoplayer.imp.PlayStateChangedListener;

/**
 * @author: zeting
 * @date: 2023/12/4
 * 视频播放器工具
 */
public class ExoPlayerUtil implements DefaultLifecycleObserver {
    private static final String TAG = "ExoPlayer-Util";

    private Context mContext;
    /**
     * 播放视图
     */
    protected PlayerView playerView;
    /**
     * 播放器
     */
    protected @Nullable ExoPlayer player;

    /**
     * 临时媒体信息
     */
    private MediaItem currentMediaItem;

    /**
     * 是否自动播放
     */
    private boolean startAutoPlay = true;

    /**
     * 媒体列表索引
     */
    private int startItemIndex;
    /**
     * 播放时间
     */
    private long startPosition;

    /**
     * 全屏按钮
     */
    private ImageView fullScreenButton;

    /**
     * 加载对话框
     */
    private View loadingView;

    /**
     * 播放状态监听
     */
    private PlayStateChangedListener playStateChangedListener;

    public ExoPlayerUtil(Context mContext, PlayerView playerView) {
        this.mContext = mContext;
        this.playerView = playerView;
    }

    public MediaItem getCurrentMediaItem() {
        return currentMediaItem;
    }


    /**
     * 设置播放源
     *
     * @param currentMediaItem
     */
    public void setCurrentMediaItem(MediaItem currentMediaItem) {
        this.currentMediaItem = checkCreateMediaItems(currentMediaItem);
    }

    /**
     * 设置播放地址
     *
     * @param url
     */
    public void setCurrentMediaItemToPlayer(String url) {
        if (!TextUtils.isEmpty(url)) {
            MediaItem mediaItem = MediaItem.fromUri(url);
            setCurrentMediaItemToPlayer(mediaItem);
        }
    }

    /**
     * 设置播放地址
     *
     * @param url
     */
    public void setCurrentMediaItemToPlayer(Uri url) {
        if (url != null) {
            MediaItem mediaItem = MediaItem.fromUri(url);
            setCurrentMediaItemToPlayer(mediaItem);
        }
    }

    /**
     * 设置视频，并且播放
     *
     * @param currentMediaItem
     */
    public void setCurrentMediaItemToPlayer(MediaItem currentMediaItem) {
        this.currentMediaItem = checkCreateMediaItems(currentMediaItem);
        clearStartPosition();
        initializePlayer();
        if (playerView != null) {
            playerView.onResume();
        }
    }

    @Override
    public void onCreate(@NonNull LifecycleOwner owner) {
        onCreate();
    }

    @Override
    public void onStart(@NonNull LifecycleOwner owner) {
        onStart();
    }

    @Override
    public void onResume(@NonNull LifecycleOwner owner) {
        onResume();
    }

    @Override
    public void onPause(@NonNull LifecycleOwner owner) {
        onPause();
    }

    @Override
    public void onStop(@NonNull LifecycleOwner owner) {
        onStop();
    }

    public void onCreate() {
        clearStartPosition();
    }

    public void onNewIntent() {
        releasePlayer();
        clearStartPosition();
    }

    public void onStart() {
        if (Build.VERSION.SDK_INT > 23) {
            initializePlayer();
            if (playerView != null) {
                playerView.onResume();
            }
        }
    }

    public void onResume() {
        if (Build.VERSION.SDK_INT <= 23 || player == null) {
            initializePlayer();
            if (playerView != null) {
                playerView.onResume();
            }
        }
    }

    public void onPause() {
        if (Build.VERSION.SDK_INT <= 23) {
            if (playerView != null) {
                playerView.onPause();
            }
            releasePlayer();
        }
    }

    public void onStop() {
        if (Build.VERSION.SDK_INT > 23) {
            if (playerView != null) {
                playerView.onPause();
            }
            releasePlayer();
        }
    }

    @Nullable
    public ExoPlayer getPlayer() {
        return player;
    }

    /**
     * 获取播放状态
     *
     * @return
     */
    public boolean isPlaying() {
        if (player != null) {
            return player.isPlaying();
        }
        return false;
    }

    /**
     * 关闭释放资源
     */
    public void releaseResource() {
        if (playerView != null) {
            playerView.onPause();
        }
        releasePlayer();
    }

    /**
     * 重播
     */
    public void replay() {
        clearStartPosition();
        initializePlayer();
        if (playerView != null) {
            playerView.onResume();
        }
    }

    /**
     * 播放还是暂停
     */
    @OptIn(markerClass = UnstableApi.class)
    public void setPlayAndPause(boolean playIfSuppressed) {
        if (player != null) {
            Util.handlePlayPauseButtonAction(player, playIfSuppressed);
        }
    }

    public boolean dispatchKeyEvent(KeyEvent event) {
        if (playerView != null) {
            return playerView.dispatchKeyEvent(event);
        }
        return false;
    }

    /**
     * 检测当前所有媒体项目是否可用
     */
    public MediaItem checkCreateMediaItems(MediaItem mediaItem) {
        if (!Util.checkCleartextTrafficPermitted(mediaItem)) {
            Log.e(TAG, "Cleartext HTTP traffic not permitted. See https://developer.android.com/guide/topics/media/issues/cleartext-not-permitted");
            return null;
        }
        if (mediaItem.localConfiguration != null) {
            MediaItem.DrmConfiguration drmConfiguration = mediaItem.localConfiguration.drmConfiguration;
            if (drmConfiguration != null) {
                if (Build.VERSION.SDK_INT < 18) {
                    Log.e(TAG, "DRM content not supported on API levels below 18");
                    return null;
                } else if (!FrameworkMediaDrm.isCryptoSchemeSupported(drmConfiguration.scheme)) {
                    Log.e(TAG, "This device does not support the required DRM scheme");
                    return null;
                }
            }
        }

        return mediaItem;
    }

    /**
     * 初始化播放器
     */
    @OptIn(markerClass = UnstableApi.class)
    public void initializePlayer() {
        if (currentMediaItem == null) {
            return;
        }
        if (playerView == null) {
            return;
        }
        if (mContext == null) {
            return;
        }
        fullScreenButton = playerView.findViewById(R.id.exo_fullscreen);
        if (player == null) {
            ExoPlayer.Builder playerBuilder = new ExoPlayer.Builder(mContext);
            // 直播自定义播放速度调整算法,
            playerBuilder.setLivePlaybackSpeedControl(new DefaultLivePlaybackSpeedControl.Builder()
                    .setFallbackMaxPlaybackSpeed(1.04f)// 最大播放速度
                    .build());
            player = playerBuilder.build();
            // 添加播放监听
            player.addListener(new PlayerEventListener());
            // 添加日志监听
            player.addAnalyticsListener(new EventLogger());
            // 设置音频
            player.setAudioAttributes(AudioAttributes.DEFAULT, true);
            // 准备就绪自动播放
            player.setPlayWhenReady(startAutoPlay);
            // 设置重复模式，单曲循环
//            player.setRepeatMode(Player.REPEAT_MODE_ONE);
            // 绑定视图和播放器
            playerView.setPlayer(player);
        }
        if (player != null) {
            boolean haveStartPosition = startItemIndex != C.INDEX_UNSET;
            if (haveStartPosition) {
                // 设置播放进度
                player.seekTo(startItemIndex, startPosition);
            }
            // 添加播放源
            player.setMediaItem(currentMediaItem, !haveStartPosition);
            // 播放器准备
            player.prepare();
        }
    }

    /**
     * 释放播放器
     */
    private void releasePlayer() {
        if (player != null) {
            updateStartPosition();
            player.release();
            player = null;
            playerView.setPlayer(null);
        }
    }

    /**
     * 更新当前视频起始位置
     */
    private void updateStartPosition() {
        if (player != null) {
            startAutoPlay = player.getPlayWhenReady();
            startItemIndex = player.getCurrentMediaItemIndex();
            startPosition = Math.max(0, player.getContentPosition());
        }
    }

    /**
     * 初始化其实位置
     */
    private void clearStartPosition() {
        startItemIndex = C.INDEX_UNSET;
        startPosition = C.TIME_UNSET;
    }

    /**
     * 各种监听
     */
    private class PlayerEventListener implements Player.Listener {
        @OptIn(markerClass = UnstableApi.class)
        @Override
        public void onEvents(Player player, Player.Events events) {
            Player.Listener.super.onEvents(player, events);
            Log.d(TAG, ">Player.Listener onEvents player:" + player + " events:" + events);
        }

        @OptIn(markerClass = UnstableApi.class)
        @Override
        public void onTimelineChanged(Timeline timeline, int reason) {
//            Player.Listener.super.onTimelineChanged(timeline, reason);
//            if (player != null) {
//                Log.d(TAG, ">Player.Listener onTimelineChanged timeline:" + timeline + " reason:" + reason
//                        + "\n "
//                        + "\n Live:" + player.isCurrentMediaItemLive()  // 指示当前正在播放的媒体项是否为直播流。即使直播已结束，此值仍然为 true。
//                        + "\n Dynamic:" + player.isCurrentMediaItemDynamic()   // 指示当前播放的媒体项是否仍在更新。对于尚未结束的直播，通常也是如此。请注意，在某些情况下，此标志也适用于非直播。
//                        + "\n offset:" + player.getCurrentLiveOffset()  //  返回当前实时与播放位置（如果有）之间的偏移量。
//                        + "\n Duration:" + player.getDuration()  //  会返回当前实时窗口的长度。
//                        + "\n Pos:" + player.getCurrentPosition()  //  返回相对于直播窗口起始位置的播放位置。
//                        + "\n MediaItem:" + player.getCurrentMediaItem().toBundleIncludeLocalConfiguration()   //  返回当前媒体内容，其中 MediaItem.liveConfiguration 包含应用为目标实时偏移量和实时偏移量调整参数提供的替换项。
//                        + "\n timeline:" + player.getCurrentTimeline().toBundle()  //  用于在 Timeline 中返回当前媒体结构。
//                        + "\n ItemIndex:" + player.getCurrentMediaItemIndex()  // 可以使用 Player.getCurrentWindowIndex 和 Timeline.getWindow 从 Timeline 检索当前 Timeline.Window。在 Window 中：
//                );
//
////                Timeline.Window window = timeline.getWindow(player.getCurrentMediaItemIndex(), );
////                Log.d(TAG, ">Player.Listener onTimelineChanged window "
////                        + "\n " + window.liveConfiguration()  //  包含目标实时偏移量和实时偏移量调整参数。这些值基于媒体中的信息以及在 MediaItem.liveConfiguration 中设置的任何应用提供的替换项。
////                        + "\n " + window.windowStartTimeMs()  //  是自 Unix Epoch 以来的时间，即实时窗口的开始时间。
////                        + "\n " + window.getCurrentUnixTimeMs()  //  是自当前实时的 Unix 纪元以来经过的时间。可以根据服务器和客户端之间的已知时钟差值来更正此值。
////                        + "\n " + window.getDefaultPositionMs()  //  是直播窗口中默认播放器开始播放的位置。
////                );
//            }
        }

        @Override
        public void onPlaybackStateChanged(int playbackState) {
            Log.d(TAG, ">Player.Listener onPlaybackStateChanged:" + playbackState);
            switch (playbackState) {
                case Player.STATE_IDLE:       // 加载播放
                case Player.STATE_BUFFERING:       // 加载播放
                    if (playStateChangedListener != null) {
                        playStateChangedListener.onStateBuffering();
                    }
                    if (loadingView != null) {
                        loadingView.setVisibility(View.VISIBLE);
                    }
                    break;
                case Player.STATE_READY:   // 加载准备播放
                    if (playStateChangedListener != null) {
                        playStateChangedListener.onStateReady();
                    }
                    if (loadingView != null) {
                        loadingView.setVisibility(View.GONE);
                    }
                    break;
                case Player.STATE_ENDED:  // 播放结束
                    if (playStateChangedListener != null) {
                        playStateChangedListener.onStateEnded();
                    }
                    break;

            }
        }

        @Override
        public void onPlayerError(PlaybackException error) {
            Log.d(TAG, ">Player.Listener onPlayerError:" + error.errorCode);
            if (playStateChangedListener != null) {
                playStateChangedListener.onStateError();
            }
            if (error.errorCode == PlaybackException.ERROR_CODE_BEHIND_LIVE_WINDOW) {
                // 直播状态异常，恢复直播位置重新准备
                if (player != null) {
                    player.seekToDefaultPosition();
                    player.prepare();
                }
            }
        }

        @Override
        public void onIsPlayingChanged(boolean isPlaying) {
            Log.d(TAG, ">Player.Listener onIsPlayingChanged:" + isPlaying);
            if (playStateChangedListener != null) {
                playStateChangedListener.onIsPlayingChanged(isPlaying);
            }
        }

        @Override
        public void onIsLoadingChanged(boolean isLoading) {
            Log.d(TAG, ">Player.Listener onIsLoadingChanged:" + isLoading);
        }
    }


    /**
     * 添加监听
     *
     * @param mPlayerListener
     */
    public void addListener(Player.Listener mPlayerListener) {
        if (player != null) {
            player.addListener(mPlayerListener);
        }
    }

    /**
     * 设置是否自动播放
     *
     * @param startAutoPlay
     */
    public void setStartAutoPlay(boolean startAutoPlay) {
        this.startAutoPlay = startAutoPlay;
    }

    public void setPlayStateChangedListener(PlayStateChangedListener playStateChangedListener) {
        this.playStateChangedListener = playStateChangedListener;
    }

    /**
     * 全部状态点击
     */
    public void setFullScreenButtonClicked() {
        if (fullScreenButton != null) {
            fullScreenButton.performClick();
        }
    }

    public void setLoadingView(View loadingView) {
        this.loadingView = loadingView;
    }
}
```


# 遗留问题
> 1. 播放视图不设置高度默认就全屏，然后才恢复视频高度，

解决方案
```java
/**
* 设置绑定之后就设置视图默认横宽比，不让视图显示最高视图
*/
AspectRatioFrameLayout aspectRatioFrameLayout = playerView.findViewById(androidx.media3.ui.R.id.exo_content_frame);
if (aspectRatioFrameLayout != null) {
   aspectRatioFrameLayout.setAspectRatio(1.778F);
}
```
