# Frida Quickstart

Frida is a dynamic instrumentation toolkit. Use it on apps and devices you own or are
authorized to test.

## 1. Install

```bash
pip install frida-tools
frida --version
```

On a rooted Android device, match the `frida-server` architecture:

```bash
adb shell getprop ro.product.cpu.abi
# arm64-v8a, armeabi-v7a, x86, x86_64
```

Download the matching `frida-server` release, push it, and start it:

```bash
adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"
```

For a non-rooted device, use a repackaged app with Frida Gadget, or a debug build.

## 2. List processes and apps

```bash
frida-ps -U
frida-ps -Uai
frida-ps -Ua
```

- `-U` — USB device
- `-a` — running applications
- `-i` — installed applications

## 3. Attach and spawn

```bash
frida -U -n com.example.app
frida -U -f com.example.app
frida -U -f com.example.app --no-pause
```

`-f` spawns the app and pauses it so you can load your script before the app initializes.
`--no-pause` resumes immediately.

## 4. Basic JavaScript API

```js
Java.perform(function () {
  console.log("Java loaded");

  var Activity = Java.use("android.app.Activity");
  Activity.onCreate.overload("android.os.Bundle").implementation = function (bundle) {
    console.log("onCreate called");
    this.onCreate(bundle);
  };
});
```

Hook a method and print arguments:

```js
Java.perform(function () {
  var Login = Java.use("com.example.app.Login");
  Login.checkPassword.implementation = function (password) {
    console.log("password =", password);
    var result = this.checkPassword(password);
    console.log("result =", result);
    return result;
  };
});
```

Call a method directly:

```js
Java.perform(function () {
  var Utils = Java.use("com.example.app.Utils");
  console.log(Utils.getSecret());
});
```

## 5. Bypass a simple root check

```js
Java.perform(function () {
  var RootCheck = Java.use("com.example.app.RootCheck");
  RootCheck.isRooted.implementation = function () {
    console.log("isRooted() -> false");
    return false;
  };
});
```

This is a lab technique. Real apps use multiple checks, native code, and server-side signals.

## 6. SSL pinning bypass skeleton

```js
Java.perform(function () {
  try {
    var TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    var SSLContext = Java.use("javax.net.ssl.SSLContext");

    var TrustAll = Java.registerClass({
      name: "com.example.TrustAll",
      implements: [TrustManager],
      methods: {
        checkClientTrusted: function (chain, authType) {},
        checkServerTrusted: function (chain, authType) {},
        getAcceptedIssuers: function () { return []; }
      }
    });

    var ctx = SSLContext.getInstance("TLS");
    ctx.init(null, [TrustAll.$new()], null);
    SSLContext.init.overload(
      "[Ljavax.net.ssl.KeyManager;",
      "[Ljavax.net.ssl.TrustManager;",
      "java.security.SecureRandom"
    ).implementation = function (km, tm, sr) {
      this.init(km, [TrustAll.$new()], sr);
    };
  } catch (e) {
    console.log("Java trust manager hook failed:", e);
  }
});
```

Also check OkHttp `CertificatePinner`, TrustKit, and native pinning. Hook each layer
separately and verify with a proxy.

## 7. Native hooks

```js
var open = Module.findExportByName(null, "open");
if (open) {
  Interceptor.attach(open, {
    onEnter: function (args) {
      console.log("open:", args[0].readCString());
    }
  });
}
```

List modules and exports:

```js
Process.enumerateModules().forEach(function (m) {
  console.log(m.name, m.base, m.size);
});
```

## 8. Running scripts

```bash
frida -U -f com.example.app -l hook.js --no-pause
frida -U -n com.example.app -l hook.js
```

Load a script from a file and keep the REPL open. Use `%reload` to reload.

## 9. Useful one-liners

```bash
# All loaded classes matching a name
frida -U -n com.example.app -e 'Java.perform(function(){ Java.enumerateLoadedClasses({ onMatch: function(c){ if(c.indexOf("example")>=0) console.log(c); } }); });'

# Current activity
adb shell dumpsys activity activities | grep mResumedActivity

# Check frida-server is running
adb shell ps -A | grep frida
```

## 10. Troubleshooting

- `unable to connect`: check `adb devices`, restart `frida-server`, and match versions.
- `Failed to spawn: unable to find process`: verify the package name with `frida-ps -Uai`.
- App crashes on spawn: use `--no-pause` or hook earlier with a gadget.
- Java bridge missing: the app may be Flutter/React Native; hook native code instead.
- Anti-Frida checks: look for `frida` strings, port 27042, and thread names; use a renamed
  server or gadget in a lab.
