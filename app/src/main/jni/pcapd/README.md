What's pcapd
------------

pcapd is an executable, run with root privileges, which can be used to capture network packets and process them into an Android app.

Its hot features include:

- Ability to capture from the internet-facing interface and automatically roam when it changes
- The executable lives within the normal app directories and does not alter the Android root file system
- Ability to capture from multiple interfaces
- Detect the direction of the packets
- All the dumped packets start with the IP headers, which relieves your app from doing datalink processing

The following datalinks are currently supported: `DLT_RAW`, `DLT_EN10MB` and `DLT_LINUX_SLL`.

Usage
-----

You can find pre-compiled binaries to integrate pcapd in other apps in the [pcapd-bin](https://github.com/emanuele-f/pcapd-bin) repository.

The pcapd executable can be run with the `-h` option to print its cli usage:

```
pcapd - root capture tool of PCAPdroid
Copyright 2021 Emanuele Faranda <black.silver@hotmail.it>

Usage: pcapd [-i ifname, -i ...] [-d] [-u uid] [-b bpf] [-l]
 -i [ifname]    capture packets on the specified interface. Can be specified
                multiple times. The '@inet' keyword can be used to capture from
                the internet interface
 -d             daemonize the process
 -t             dump the interface datalink header. By default, the interface
                datalink is skipped. This does not affect the @inet interface.
 -u [uid]       filter packets by uid
 -b [bpf]       filter packets by BPF filter
 -l [file]      log output to the specified file
```

If no option is provided, pcapd will start capturing on the internet interface.

The daemon logs messages to logcat, identified by the `pcapd` tag.

App integration
---------------

pcapd uses a UNIX socket to send the captured packets to the client app, so native code must be used to interface with it.

pcapd is compiled into the binary executable `libpcapd.so`, which is named as a shared library to force grandle to package it into the apk.
The app must set `android:extractNativeLibs="true"` in the `AndroidManifest.xml` in order to unpack the `libpcapd.so` to the Android filesystem once the app is installed.
The `libpcapd.so` is located into the app library path, which can be obtained in java via `Context.getApplicationInfo().nativeLibraryDir`.

Here are the steps to make it communicate with your app:

1. The app chdirs to a writable directory, which will be the working directory for the pcapd daemon.
2. The app should check if a `PCAPD_PID` file is present. If so, it can read it and kill the running deamon.
3. The app creates the `PCAPD_SOCKET_PATH` UNIX socket in listen mode.
4. The app spawns pcapd in deamon mode (with the `-d` option) and waits for its connection on the UNIX socket.
5. The app can now receive the packets on the UNIX socket.
6. When the app closes the UNIX socket, the pcapd daemon is automatically stopped.

Check out the [capture_root.c source](https://github.com/emanuele-f/PCAPdroid/blob/master/app/src/main/jni/vpnproxy-jni/capture_root.c) to see an example of integration.