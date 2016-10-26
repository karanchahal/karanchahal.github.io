---
layout:     post
title:      Make A Chrome Extension
comments: true
date:       2016-07-26  15:31:19
summary:    My Views On Chat-bots
categories: Chrome Extension Programming
---
<div style="text-align:center" markdown="1">

![desk](https://raw.githubusercontent.com/karanchahal/Torrent-Hunter/master/Screenshots/new.gif)

</div>
Greetings fellow human,

We are going to make a simple chrome extension today.

It is a very powerful concept that if used correctly can add a lot of improvements to web interaction on your browser.

# The Idea
We are going to make an extension that makes it easier to download torrent files. This extension aims to alleviate problems that a user encounters while downloading torrent files from various torrent hosting websites on the internet .
These websites generally have a lot of spam and advertisements across them, which makes it difficult to find the link to the file you actually want to download.

To solve this, we are going to make an extension that parses all magnet links on the page and open them on clicking the extension icon, as shown above. The extension should also go through a color change on detecting torrent magnet links on the current tab.



## Magnet Links

When you download a _.torrent file_, you're essentially downloading a small file that contains information on the larger files you want to download. The torrent file tells your torrent client the names of the files being shared, a URL for the tracker, and more. Your torrent client then calculates a hash code, which is a unique code that only that torrent has—kind of like an ISBN or catalog number. From there, it can use that code to find others uploading those files, so you can download from them.

A magnet link does away with the middleman. A magnet link is essentially a hyperlink containing the hash code for that torrent, which your torrent client can immediately use to start finding people sharing those files. Magnet links don't require a tracker (since it uses DHT, which you can read more about here), nor does it require you to download a separate file before starting the download, which is convenient.


Soon, popular torrent site [The Pirate Bay will no longer host torrent files](https://torrentfreak.com/the-pirate-bay-will-stop-serving-torrents-120112/). Instead, it will only offer magnet links.

# Implementation

To start, we first initialize a _manifest.json_ file.
This file defines your chrome extension and contains useful details like:

* The type of files that are allowed
* The links on which the extension is active
* The default icon and title
* Content and background scripts
* Permissions
* Description and version
The manifest file is as follows:

``` {
  "manifest_version": 2,

  "name": "Torrent Hunter",
  "description": "This extension aims to get the relevant magnet link from a sea of advertisements and spam.",
  "browser_action": {
    "default_icon":"icon-default.png",
    "default_title":"No torrents on this page"
  },

  "version": "1.0.0",
  "background": {
      "scripts":["background.js"]
  },
  "permissions": [
    "activeTab"
    ],
   "content_scripts":[
       {
         "matches": ["http://*/*","https://*/*"],
         "js": ["script.js"]
       }
   ]
}
```

## Background script

This Javascript file is the command center of your extension and directs flow of information to and fro from the HTML files to the content scripts.

In the discussion of this extension, the background.js file takes the links sent from the content scripts and figures out whether to change the icon color to indicate whether the website has magnet links in them .


The code for the background script that checks if the icon has been clicked and that saves the link if found is

```
chrome.runtime.onMessage.addListener(
  function(request, sender, sendResponse) {

      link = request.link;

      chrome.browserAction.setIcon({
              path: './icon-success.png',
              tabId: sender.tab.id
          });
      chrome.browserAction.setTitle({
              title: 'Click to download torrents',
              tabId: sender.tab.id
          });
  });

 ```

The background file also checks if the extension icon is clicked , if yes then it runs another script (_worker-script.js_) to open the magnet link found in the previous content script.

The code for the background script that checks this is:

```
  chrome.browserAction.onClicked.addListener(function() {
      chrome.tabs.executeScript(null,{
          code: 'var magnetLink=' + link + ';'
      },function() {
          chrome.tabs.executeScript(null,{file:'worker_script.js'});
      });
```

You'll observe that all these functions start with the
` chrome.runtime.` line.
These functions are all part of the chrome extension API's found here.


# Content Scripts

These scripts run on the User DOM _(Document Object Model)_ , that is, the website you are currently looking at.
Unlike the background.js script, these Javascript files directly interact with the websites.

The _script.js_ parses all the links found on the page and checks whether they're of the magnet variety.

Magnet links have a 'magnet:' prefix to them, and when a magnet link is encountered , it is packaged into a json object and sent to the background.js file. The background.js file saves these links in a variable `link`.

The code for _script.js_ is

```
// Get magnet links
var links = document.links;
var magnetLink = '';
var str = '';

for(var i=0; i<links.length; i++) {
    str = String(links[i]).slice(0,7);
    if(str == 'magnet:') {
        magnetLink = links[i].href;
        break;
    }
}

// send links to background if found
if(magnetLink != '') {
    chrome.runtime.sendMessage({
        link:magnetLink
    });
}
```

The contents of this variable are used whenever a user clicks on the chrome extension.

On clicking the chrome extension, through a chrome-tabs API which listens for whenever a user clicks on the extension.
So when the icon is clicked, a content script is activated, _worker-script.js_ which opens the link stored via the _script.js_.

The worker_script code is simply
```
window.open(magnetLink,'_self');
```
This triggers the magnet link, '_self' makes sure it opens in the current_tab.



Message passing is used to communicate between background and content scripts through the chrome.runtime API .

Chrome.tabs is used a lot and it is a Google extension API to facilitate client-background communication.

# Loading the extension

Now to complete this process drag and drop the extension folder to (here)[chrome://extensions/]. ENable/Load it and run.




THe entire github repo can be found [here](https://github.com/karanchahal/Torrent-Hunter).

# Conclusion

This has been just a slight taste of wehat extensions can do, had over to the [official chrome extension documentation](https://developer.chrome.com/extensions) that explores a variety of unique ways that an extension can be used.

Have a great day !
