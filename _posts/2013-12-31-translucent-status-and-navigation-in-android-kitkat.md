---
layout: post
title: Translucent Status and Navigation in Android KitKat
---

A recent project I was working on required using a new feature added in KitKat (API 19): translucent status and navigation. Out of the box integration is easy, but if you need full bleed content or are looking for a way to blend the status bar and Action Bar together, the solution requires a few extra lines of code.

## Using Translucent Decor

Reading the Android docs lead me to a very basic way of achieving translucent decor, but it was not the full solution I was seeking. Here were my options.

### Option 1: Inherit a Theme

KitKat provides two themes you can use to quickly get translucent decor:

    Theme.Holo.NoActionBar.TranslucentDecor
    Theme.Holo.Light.NoActionBar.TranslucentDecor

You can use these themes either by setting one as the theme for your application or activity, or you can inheriting one of them in your custom theme.

Set the theme on the entire application

{% highlight xml %}
<application android:theme="@style/AppBaseTheme" >
{% endhighlight %}

Set the theme on an individual activity

{% highlight xml %}
<activity android:theme="@style/AppBaseTheme" />
{% endhighlight %}

Or inherit one in your custom theme

{% highlight xml %}
<style name="AppBaseTheme" parent="android:Theme.Holo.NoActionBar.TranslucentDecor">
{% endhighlight %}

### Option 2: Set Style Properties

If you are seeking more control over translucent decor, consider using a custom theme that extends something other than `*.TranslucentDecor` and use the following style properties in your theme:

{% highlight xml %}
<item name="android:windowTranslucentStatus">true</item>
<item name="android:windowTranslucentNavigation">true</item>
{% endhighlight %}

## Caveats

Getting translucent decor is easy, but getting an interface set up properly requires additional work thanks to these caveats:

1. The navigation bar is not always translucent.
2. The navigation bar can appear in different locations depending on the device (handset / tablet).
3. Using a translucent theme or setting the style property `windowTranslucentNavigation=true` will cause your layout to fill the screen even if it means content is hidden behind the navigation bar.
4. Using the predefined themes disables the Action Bar

### Workaround 1: Use *fitsSystemWindows* for content layouts

If your content does not need to be full bleed you can structure your layouts in such a way that the content will reside in an area of the screen that is not blocked by the status or navigation bar. To do this you would set `android:fitsSystemWindows="true"` on the content layout/view in your layout xml.

{% highlight xml %}
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/i_am_full_bleed"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <LinearLayout
        android:id="@+id/i_stay_in_the_content_area"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true" >
    </LinearLayout>

</FrameLayout>
{% endhighlight %}

### Workaround 2: Leverage Resource Qualifiers

In my case I needed the content to be full bleed. This meant I had to find a way around the opaque navigation bar and its inconsistent placement. I used resource qualifiers/rules to control when the nav bar would be translucent (see [Android: Providing Resources](http://developer.android.com/guide/topics/resources/providing-resources.html)).

*Note: In my tests the navigation bar was always at the bottom on tablets.*

**Rules:**

1. Base case: set the nav bar to translucent
2. Disable translucent nav bar for landscape
3. Enable translucent nav for landscape on tablets

This looks something like:

**res/values-v19/styles.xml** *(these values are in my theme)*

{% highlight xml %}
<item name="android:windowTranslucentStatus">true</item>
<item name="android:windowTranslucentNavigation">@bool/translucentNavBar</item>
{% endhighlight %}

**res/values-v19/bools.xml** *(Rule 1. enable translucent nav)*

{% highlight xml %}
<bool name="translucentNavBar">true</bool>
{% endhighlight %}

**res/values-land-v19/styles.xml** *(Rule 2. disable translucent nav in landscape)*

{% highlight xml %}
<bool name="translucentNavBar">false</bool>
{% endhighlight %}

**res/values-sw600dp-land-v19.xml** *(Rule 3. enable translucent nav for tablets in landscape)*

{% highlight xml %}
<bool name="translucentNavBar">true</bool>
{% endhighlight %}

## Blending in the Action Bar

Everything is great, except that I would like to be able to control the background of the status and navigation.

Look no further than [SystemBarTint](https://github.com/jgilfelt/SystemBarTint).

This library gives you everything you need to easily set a background on the status bar or navigation bar, and also comes with a few helpful methods that allow you to easily compute the screen space used by these two components. SystemBarTint's documentation is great, so have a look!

I used the following drawable for the Action Bar background:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle"
    android:useLevel="false" >

    <gradient
        android:angle="-90"
        android:endColor="#00000000"
        android:startColor="#BB000000"
        android:type="linear" />

</shape>
{% endhighlight %}

And the following code in my base `Activity` in order to blend the status bar and Action Bar:

{% highlight java %}
SystemBarTintManager tintManager = new SystemBarTintManager(this);
tintManager.setStatusBarTintColor(Color.parseColor("#BB000000"));
tintManager.setStatusBarTintEnabled(true);
{% endhighlight %}