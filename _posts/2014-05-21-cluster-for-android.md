---
layout: post
title: Cluster for Android
---

Say hello to [Cluster](https://github.com/DevBytes/cluster), a collection of helper classes and utility methods for Android. I find myself routinely writing code to do basic things from project to project. Cluster is meant to be an expansive library that minimizes how much utility code needs to be written for each project.

## Project Page

https://github.com/DevBytes/cluster

## Components

Cluster is currently composed of the following:

### Service Connection Manager
There are two major types of services in Android: the ones you start, and the ones you bind. In the latter case, you cannot communicate with the service until you have successfully connected to it. Service connections are asynchronous and require code to keep your activity/fragment in sync with this state. Favor contains a `ServiceConnectionManager` that you can use to connect to your service. You can also feed the manager commands that will be executed once the service is successfully connected.

For each service that you want to manage, create an extension of the service manager:

{% highlight java %}
class MyServiceConnectionManager extends ServiceConnectionManager<MyService, MyServiceBinder> {

  @Override
  protected Class<MyService> getServiceClass() {
    return MyService.class;
  }

}
{% endhighlight %}

Then use it in your Activity or Fragment as follows:

{% highlight java %}
class MainActivity extends Activity {

  private MyServiceConnectionManager manager;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    // Create the manager
    manager = new MyServiceConnectionManager();

    // Start the manager and connect to the service
    manager.start();

    // Run a command as soon as the service is bound
    manager.runCommand(new ServiceCommand<MyServiceBinder>() {

    @Override
    public void run(MyServiceBinder service) {
      // get something from the service
    }

  }
}
{% endhighlight %}

### Screen
Screen is a utility class holding methods related to screen size `isTablet()` and orientation `isLandscape()`
