# Bridge

[![Release](https://jitpack.io/v/Livefront/bridge.svg)](https://jitpack.io/#Livefront/bridge)

A library for avoiding TransactionTooLargeException during state saving and restoration.

## Contents

* [Motivation](#motivation)
* [Setup](#setup)
* [Clearing Data](#clear)
* [Install](#install)
* [How Does It Work](#how)
* [Limitations](#limitations)
* [Testing](#testing)
* [License](#license)

<a name="motivation"></a>
## Motivation

In spite of warnings from the Android development team stating that the state restoration framework should only be used for small amounts of view-related data, many developers have found it very useful to save large amounts of network-related data to avoid unnecessary network requests across configuration changes and when restoring from a background state. There was always a limitation to this, but that limit resulted in a silent failure to save state. In Android Nougat, that was upgraded to a crash via a [TransactionTooLargeException](https://developer.android.com/reference/android/os/TransactionTooLargeException.html).

At Google I/O 2017, the Android development team gave a series of recommendations about app architecture that made clear that---rather than relying on the state restoration framework---the preferred method to save network data involves:

- saving and restoring data from memory across configuration changes
- saving and restoring data from disk when restoring from a background state

While new tools are available to achieve these goals, many developers will not be able to quickly update their apps to take advantage of them and will still face the dreaded `TransactionTooLargeException`. `Bridge` was created as a way to keep your existing app architecture in place while avoiding crashes related to state saving and restoration by following those two main principles.

<a name="setup"></a>
## Setup

`Bridge` is intended as a simple wrapper for annotation-based state saving libraries like [Icepick](https://github.com/frankiesardo/icepick), [Android-State](https://github.com/evernote/android-state), and [Icekick](https://github.com/tinsukE/icekick). For example, if you are already using `Icepick`, simply replace all calls to `Icepick.restoreInstanceState()` and `Icepick.saveInstanceState()` with their `Bridge` equivalents:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Bridge.restoreInstanceState(this, savedInstanceState);
}

@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    Bridge.saveInstanceState(this, outState);
}
```

The only additional change to make is to initialize `Bridge` in your `Application.onCreate()` and specify `Icepick` as your "saved state handler":

```java
Bridge.initialize(getApplicationContext(), new SavedStateHandler() {
    @Override
    public void saveInstanceState(@NonNull Object target, @NonNull Bundle state) {
        Icepick.saveInstanceState(target, state);
    }

    @Override
    public void restoreInstanceState(@NonNull Object target, @Nullable Bundle state) {
        Icepick.restoreInstanceState(target, state);
    }
});
```

That's it! You don't have to change any of your other code. If you are using any other `Icepick`-like library, simply swap out the library referred to in the `SavedStateHandler`.

<a name="clear"></a>
## Clearing Data

Bridge will clear all data written to disk each time the app is loaded and it detects that there is no saved state the system is trying to restore. It will also try to automatically clear state while the app is actively used to avoid holding onto data that is no longer relevant. However, in order to guarantee that no data is retained on disk for screens that have been left, the `Bridge.clear()` method may be used:

```java
    @Override
    public void onDestroy() {
        super.onDestroy();
        Bridge.clear(this);
    }
```

In the event that you might like to migrate away from the use of `Bridge` but ensure that all associated data is cleared, `Bridge.clearAll` may be called at any time.

<a name="install"></a>
## Install

`Bridge` can be installed via gradle:

```gradle
repositories {
    maven { url "https://jitpack.io" }
}

dependencies {
    compile 'com.github.livefront:bridge:v1.1.0'
}
```

<a name="how"></a>
## How Does It Work

`Bridge` uses the `SavedStateHandler` to load your object's state into the given `Bundle`, but rather than send that `Bundle` to the main process of the OS (where it is subject to the `TransactionTooLargeException`) it saves it to memory and disk in a way that can restored to the same objects later.

There is one main caveat here : in order to ensure that as little of your app's code needs to change as possible, `Bridge` will read its data from disk on the main thread. This is currently done in a way that may add a small amount of time to your app's startup process. Fortunately, `Bridge` leverages the compact nature of `Bundle` to store data as efficiently as possible, and even extremely large amounts of data well in excess of the `1MB` limit leading to `TransactionTooLargeException` should only add something on the order of 100ms to the startup time.

<a name="testing"></a>
## Testing

Typically state saving and restoration may be tested by simply testing device rotation. It is recommended that you also use tools that actually test full state restoration however, such as [Process Killer](https://github.com/livefront/process-killer-android) or the "Don't Keep Activities" developer option.

<a name="license"></a>
## License

    Copyright 2017 Livefront

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.