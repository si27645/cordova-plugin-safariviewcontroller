SafariViewController Cordova Plugin
===================================
This is a branch from the plugin created by [Eddy Verbruggen](https://github.com/EddyVerbruggen/cordova-plugin-safariviewcontroller)

The only diffirence is that this plugin is forcing the use of chrome browser.

## 1. Installation
To install the plugin with the Cordova CLI from npm:

```
$ cordova plugin add https://github.com/MoemenMostafa/cordova-plugin-safariviewcontroller
```

### Graceful fallback to InAppBrowser
Since SafariViewController is new in iOS9 you need to have a fallback for older versions (and other platforms),
so if `available` returns false (see the snippet below) you want to open the URL in the InAppBrowser probably,
so be sure to include that plugin as well:

```
$ cordova plugin add cordova-plugin-inappbrowser
```

I'm not including it as a depency as not all folks may have this requirement.

## 2. Usage
Check the [demo code](demo/index.html) for an easy to drop in example, otherwise copy-paste this:

```js
function openUrl(url, readerMode) {
  SafariViewController.isAvailable(function (available) {
    if (available) {
      SafariViewController.show({
            url: url,
            hidden: false, // default false. You can use this to load cookies etc in the background (see issue #1 for details).
            animated: false, // default true, note that 'hide' will reuse this preference (the 'Done' button will always animate though)
            transition: 'curl', // (this only works in iOS 9.1/9.2 and lower) unless animated is false you can choose from: curl, flip, fade, slide (default)
            enterReaderModeIfAvailable: readerMode, // default false
            tintColor: "#00ffff", // default is ios blue
            barColor: "#0000ff", // on iOS 10+ you can change the background color as well
            controlTintColor: "#ffffff" // on iOS 10+ you can override the default tintColor
          },
          // this success handler will be invoked for the lifecycle events 'opened', 'loaded' and 'closed'
          function(result) {
            if (result.event === 'opened') {
              console.log('opened');
            } else if (result.event === 'loaded') {
              console.log('loaded');
            } else if (result.event === 'closed') {
              console.log('closed');
            }
          },
          function(msg) {
            console.log("KO: " + msg);
          })
    } else {
      // potentially powered by InAppBrowser because that (currently) clobbers window.open
      window.open(url, '_blank', 'location=yes');
    }
  })
}

function dismissSafari() {
  SafariViewController.hide()
}
```

## 3. Reading Safari Data and Cookies with Cordova

SFSafariViewController implements "real" Safari, meaning private data like cookies and Keychain passwords are available to the user. However, for security, this means that communication features such as javascript, CSS injection and some callbacks that are available in UIWebView are not available in SFSafariViewController.

To pass data from a web page loaded in SFSafariViewController back to your Cordova app, you can use a Custom URL Scheme such as _<mycoolapp://data?to=pass>_.  You will need to install an addition plugin to handle receiving data passed via URL Scheme in your Cordova app.

Combining the URL Scheme technique with the HIDDEN option in this plugin means you can effectively read data from Safari in the background of your Cordova app. This could be useful for automatically logging in a user to your app if they already have a user token saved as a cookie in Safari.

Do this:

1. Install the [Custom URL Scheme Plugin](https://github.com/EddyVerbruggen/Custom-URL-scheme)
2. Create a web page that reads Safari data on load and passes that data to the URL scheme:

    ```javascript
    <html>
      <head>
        <script type="javascript">
          function GetCookieData() {
            var app = "mycoolapp"; // Your Custom URL Scheme
            var data = document.cookie; // Change to be whatever data you want to read
            window.location = app + '://?data=' + encodeURIComponent(data); // Pass data to your app
          }
        </script>
      </head>
      <body onload="GetCookieData()">
      </body>
    </html>
    ```

3. Open the web page you created with a hidden Safari view:

    ```javascript
    SafariViewController.show({
      url: 'http://mycoolapp.com/hidden.html',
      hidden: true,
      animated: false
    });
    ```

4. Capture the data passed from the web page via the URL Scheme:

    ```javascript
    function handleOpenURL(url) {
      setTimeout(function() {
        SafariViewController.hide();
        var data = decodeURIComponenturl.substr(url.indexOf('=')+1));
        console.log('Browser data received: ' + data);
      }, 0);
    }
    ```
