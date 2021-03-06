[![Build Status](https://travis-ci.org/nacholibre/cpos.svg?branch=master)](https://travis-ci.org/nacholibre/cpos)
##Introduction
With `cpos` you can start Chromium browser with socketIO server and emit messages from any socketIO client (browser for example) to open specific URL and return some data back like page load time or page screenshot.

`cpos` uses [node-webkit](https://github.com/rogerwang/node-webkit).

Install it with `npm install cpos`

##Usage
`cpos` requires libuv0 and xvfb-run. If you don't have libuv0, you can symlink libuv0 to libuv1. Example in Arch Linux: `sudo ln -s /usr/lib/libudev.so /usr/lib/libudev.so.0`

####Example
```javascript
'use strict';

var cpos = require('cpos');
var io = require('socket.io-client');

var chromiumServer = new cpos.Server();

chromiumServer.listen(3000, 'localhost', function() {
    console.log('Chromium browser socketIO server has started');

    var client = io.connect('http://localhost:3000');
    client.on('connect', function() {

        client.emit('openURL', {'url': 'http://google.com'}, function(data) {
            console.log('Google takes %s milliseconds to load', data.pageLoadTimeMS);

            client.close();
            chromiumServer.close();
        });
    });
});
```
##API
###Class: cpos.Server
####server.listen(port, hostname, [callback])
Starts the node-webkit Chromium app using xvfb-run. The app itself starts socket.io server listening for `openURL` messages.

`callback` is called when the applications is ready to receive openURL requests.
####server.close()
Terminates the server.
####socket.emit('openURL', options, callback);
available `options` are 
- url: url for opening
- width: window width (default: 1280)
- height: window height (default: 768)
- capture: return screenshot as base64 encoded string (default: false)
- timeout: max timeout in seconds when opening url (default: 15 seconds)

callback is called with
- pageLoadTimeMS: page load time in milliseconds
- screenshot: base64 encoded screenshot of the loaded page (only if capture is set to true)
- loaded: True or False, false can be if the timeout is reached

`openURL` opens new tab in Chromium and after page has loaded the tab is closed and the callback is executed, which means you can open pages in parallel.

You can use [node async](https://github.com/caolan/async) to limit max parallel tasks.
