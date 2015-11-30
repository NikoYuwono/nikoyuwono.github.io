---
layout: post
title: Toolbar Panel
---

Recently I've been working on a Library that inspired from Android DrawerLayout, but instead of showing drawer from either left or right this library will provide you function to pull down the toolbar to show a panel which you can customize by yourself.

#Demo

You can watch the demo video here

[![Toolbar Panel Demo](http://img.youtube.com/vi/KTPwKYvP6OI/0.jpg)](http://www.youtube.com/watch?v=KTPwKYvP6OI)

#Usage
To use it you need to implement `ToolbarPanelLayout` as your root layout :

```
<com.nikoyuwono.toolbarpanel.ToolbarPanelLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/sliding_down_toolbar_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:panelId="@+id/panel"
    app:toolbarId="@+id/toolbar">

    <RelativeLayout
        android:id="@+id/content_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        //Your content here
    </RelativeLayout>

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/color_primary" />

    <RelativeLayout
        android:id="@+id/panel"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        //Your panel content here
    </RelativeLayout>
</com.nikoyuwono.toolbarpanel.ToolbarPanelLayout>
```

The key is to set `panelId` and `toolbarId` to `ToolbarPanelLayout` so it can decide where to draw the panel

To open and close panel programatically you can call

```
ToolbarPanelLayout.openPanel(); // Open the panel
ToolbarPanelLayout.closePanel(); // Close the panel
```

And to lock the panel you can use `ToolbarPanelLayout.setLockMode(int lockMode);`
which support 3 Mode :

```
/**
 * The drawer is unlocked.
 */
public static final int LOCK_MODE_UNLOCKED = 0;

/**
 * The drawer is locked closed. The user may not open it, though the app may open it
 * programmatically.
 */
public static final int LOCK_MODE_LOCKED_CLOSED = 1;

/**
 * The drawer is locked open. The user may not close it, though the app may close it
 * programmatically.
 */
public static final int LOCK_MODE_LOCKED_OPEN = 2;
```

You can also set listener to the `ToolbarPanelLayout` with `ToolbarPanelListener` which supported 3 events for now :

```
/**
 * Called when a panel's position changes.
 */
public void onPanelSlide(Toolbar toolbar, View panelView, float slideOffset);

/**
 * Called when a panel has settled in a completely open state.
 * The panel is interactive at this point.
 */
public void onPanelOpened(Toolbar toolbar, View panelView);

/**
 * Called when a panel has settled in a completely closed state.
 */
public void onPanelClosed(Toolbar toolbar, View panelView);
```

#Download

You can grab it via Gradle :

```
compile 'com.nikoyuwono:toolbar-panel:0.1'
```

Or you can check the sourcode in [Github](https://github.com/NikoYuwono/ToolbarPanel)
Issues and pull requests are welcome!