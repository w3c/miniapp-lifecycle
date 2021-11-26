# MiniApp Lifecycle explainer

> Note: This document serves as a supplementary explanation of the [MiniApp Lifecycle](https://w3c.github.io/miniapp/specs/lifecycle/) 
spec. If there is any inconsistency with the spec, you should consider the spec to be authoritative.

## Authors

Qing An (Alibaba)

## What is this

This is a proposal for a new Web API for MiniApp lifecycle to manage the lifecycle events of both global MiniApp application 
lifecycle and each MiniApp page’s lifecycle.

MiniApp is composed of two layers, app layer and page layer. Therefore, MiniApp lifecycle contains application lifecycle and 
page lifecycle.

To better understand MiniApp lifecycle, here is the description of MiniApp running mechanism:

* Download

MiniApp is install-free. When user opens a MiniApp for the first time, the hosted native App will download MiniApp resource 
from backend server. The downloaded MiniApp resource will be cached in the hosted native App for some duration. Afterwards, 
when user reopens the cached MiniApp, the downloading procedure can be skipped, therefore the MiniApp can be opened more quickly.

* Cold Launch and Hot Launch

Cold launch means the procedure of opening a MiniApp that has never been launched locally, or has been destroyed. During this 
procedure, MiniApp will implement initialization.

Hot launch means the procedure of opening a MiniApp that is running in the background. In this case, MiniApp is only switching 
from running in the background to running in the foreground

* Foreground and Background

Running in the foreground: when user firstly opens a MiniApp, it is in the state of running in the foreground.

Running in the background: when user closes the MiniApp, or leaves the hosted Native App, the MiniApp will not be destroyed 
directly. Instead, it will switch to run in the background.

Switching from running in the background to foreground: when the has-not-been-destroyed MiniApp is reopened, it will switch 
from running in the background to running in the foreground.

* Destroy

When MiniApp is closed, it just switches to be running in the background. Only after the MiniApp have been running in the 
background for a specific duration, or it has occupied too many system resources, then it will be destroyed.

## MiniApp application lifecycle states

*	MiniApp initialization: after MiniApp initialization is completed, MiniApp application enters the state of “Launched”. 
At this moment, the path and query of MiniApp URI can be obtained.
*	MiniApp running in foreground: once the MiniApp launch is completed, or once the MiniApp switches to be running in 
foreground from background, MiniApp application enters the state of “Shown”
*	MiniApp running in background: once the MiniApp switches to be running from foreground to background, MiniApp application 
enters the state of “Hidden”
*	MiniApp error: once the MiniApp is confronted with script error, MiniApp application enters the state of “Error”

## MiniApp page lifecycle states

*	MiniApp page loading: once MiniApp page loading is completed, MiniApp page enters the state of “Loaded”
*	MiniApp page first rendering ready: once the MiniApp page first rendering is completed, MiniApp page enters the state of "Ready"
*	MiniApp page running in foreground: once the page switches to be running in foreground from background, MiniApp page enters 
the state of “Shown”
*	MiniApp page running in background: once the MiniApp page switches to be running from foreground to background, MiniApp page 
enters the state of “Hidden”
*	MiniApp page unloading: once the MiniApp page is destroyed, MiniApp page enters the state of “Unloaded”

## MiniApp origin

When a MiniApp is launched, its hosted [Super App](https://w3c.github.io/miniapp/white-paper/#dfn-super-app) or OS will check the origin of the MiniApp, to guarantee the MiniApp's security.

## Sample code

*	MiniApp application lifecycle: 

Assume MiniApp URI is: `miniapp://foo;version=1.0.1-trial@example.com:8080/pages/index?k=v#bar`

```js
App({ 
  onLaunch(options) { // MiniApp is launched for the first time
    console.log(options.query); // k=v#bar
    console.log(options.path);  // pages/index
  },
  onShow(options) { // MiniApp launch is completed, or MiniApp switches to be running in foreground from background
    console.log(options.query); // k=v#bar
    console.log(options.path);  // pages/index
  },
  onHide() { // MiniApp switches to be running from foreground to background
    console.log('app hide');
  },
  onError(error) { // MiniApp is confronted with script error
    console.log(error);
  },
});
```

* MiniApp page lifecycle: 

```js
Page({
  onLoad(query) {
    // MiniApp page loading is completed
  },
  onShow() {
    // MiniApp page switches to be running in foreground from background
  },
  onReady() {
    // MiniApp page first rendering is completed
  },
  onHide() {
    // MiniApp page switches to be running from foreground to background
  },
  onUnload() {
    // MiniApp page is destroyed
  },
});
```


## Comparison with some related work in W3C (such as [Service Worker](https://www.w3.org/TR/service-workers/), [Page Visibility](https://w3c.github.io/page-visibility/) and [Page Lifecycle](https://wicg.github.io/page-lifecycle/))

<table>
    <thead>
        <tr class="thead-first-child">
          <th align="left"> MiniApp Lifecycle States</th>
          <th align="left"> Lifecycle States defined by existing W3C specs </th>
        </tr>
    </thead>
        <tr class="tbody-first-child">
          <td align="left"> Application Launched </td>
          <td align="left"> Service Worker Installed (https://www.w3.org/TR/service-workers-1/#dom-serviceworkerstate-installed) </td>
        </tr>
        <tr class="tbody-first-child">
          <td align="left"> Application Shown </td>
          <td align="left"> Service Worker Activating (https://www.w3.org/TR/service-workers-1/#dom-serviceworkerstate-activating) or Activated (https://www.w3.org/TR/service-workers-1/#dom-serviceworkerstate-activated) </td>
        </tr>
        <tr class="tbody-first-child">
          <td align="left"> Application Hidden </td>
          <td align="left"> N/A </td>
        </tr>
        <tr class="tbody-first-child">
          <td align="left"> Application Error </td>
          <td align="left"> <a href="https://w3c.github.io/uievents/#event-type-error">error</a> event on <code>Element</code>/<code>Window</code> </td>
        </tr>
        <tr class="tbody-first-child">
          <td align="left"> Page Loaded </td>
          <td align="left"> load UI Event (https://www.w3.org/TR/uievents/#event-type-load), or window.onload event handler (https://html.spec.whatwg.org/multipage/webappapis.html#handler-onload)</td>
        </tr>
        <tr class="tbody-first-child">
          <td align="left"> Page Ready </td>
          <td align="left"> N/A </td>
        </tr>
        <tr class="tbody-first-child">
          <td align="left"> Page Shown </td>
          <td align="left"> Visible (https://w3c.github.io/page-visibility/#visibility-states) </td>
        </tr>
        <tr class="tbody-first-child">
          <td align="left"> Page Hidden </td>
          <td align="left"> Hidden (https://w3c.github.io/page-visibility/#visibility-states) </td>
        </tr>
        <tr class="tbody-first-child">
          <td align="left"> Page Unloaded </td>
          <td align="left"> Discarded (https://wicg.github.io/page-lifecycle/#sec-lifecycle-states) </td>
        </tr>
        
</table>
