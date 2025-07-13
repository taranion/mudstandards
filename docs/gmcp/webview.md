---
sidebar_label: webview
---
# The ``WebView`` package

Source: [BeipMu](https://github.com/BeipDev/BeipMU/blob/master/Documentation/WebViews.md#gmcp)

## webview.open 

Example for opening an URL
````json
webview.open { 
    "url":"https://our_cool_website.html" 
}
````

Example for having an URL auto dock, plus providing http-request-header to auto-login to your website. Useful for doing online player editing through the website with a simple in-game command!
````json
webview.open { 
    "id":"Character editor", 
    "dock":"right", 
    "url":"value", 
    "http-request-headers":{ 
        "name1":"value1", 
        "name2":"value2" 
    } 
}
````
- **url**
  (*Mandatory*) URL to open
- **id**
  (*Optional*) A way to refer to a webview. If a later webview.open comes in, it will replace the original one with the same id
- **dock**
  (*Optional*) If given, the new view will dock on the given side of the terminal window, instead of opening a new window.
- **http-request-headers**
  (*Optional*) A list of name/value pairs to be added to the http request.
