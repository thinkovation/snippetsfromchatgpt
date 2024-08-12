# Creating a chrome extension that copies the functionality of ModHeader

Creating a Chrome extension that clones the functionality of ModHeader involves several steps. Below is a step-by-step guide to help you get started:

### Step 1: Set Up Your Project

1. **Create a project folder**: Create a new directory for your Chrome extension project. For example, `modheader-clone`.

2. **Create essential files**:
   - `manifest.json`: This file describes your extension and its permissions.
   - `background.js`: This will handle the background tasks.
   - `popup.html`, `popup.js`, and `popup.css`: These files will create a popup interface for user interaction.

### Step 2: Create `manifest.json`

The `manifest.json` file is crucial as it provides metadata about your extension and specifies the permissions it needs.

```json
{
  "manifest_version": 3,
  "name": "ModHeader Clone",
  "version": "1.0",
  "description": "A simple Chrome extension to modify HTTP headers",
  "permissions": [
    "webRequest",
    "webRequestBlocking",
    "storage",
    "activeTab",
    "tabs"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icon16.png",
      "48": "icon48.png",
      "128": "icon128.png"
    }
  },
  "host_permissions": [
    "<all_urls>"
  ],
  "icons": {
    "16": "icon16.png",
    "48": "icon48.png",
    "128": "icon128.png"
  }
}
```

### Step 3: Create `background.js`

This file will handle modifying the headers of HTTP requests.

```javascript
chrome.webRequest.onBeforeSendHeaders.addListener(
  function(details) {
    let headers = details.requestHeaders;

    // Retrieve the headers from storage
    chrome.storage.sync.get(['headers'], function(result) {
      if (result.headers) {
        result.headers.forEach(header => {
          headers.push({name: header.name, value: header.value});
        });
      }
    });

    return {requestHeaders: headers};
  },
  {urls: ["<all_urls>"]},
  ["blocking", "requestHeaders"]
);
```

### Step 4: Create `popup.html`

This file creates the UI for adding new headers.

```html
<!DOCTYPE html>
<html>
<head>
  <title>ModHeader Clone</title>
  <link rel="stylesheet" type="text/css" href="popup.css">
</head>
<body>
  <h1>ModHeader Clone</h1>
  <form id="header-form">
    <input type="text" id="header-name" placeholder="Header Name" required>
    <input type="text" id="header-value" placeholder="Header Value" required>
    <button type="submit">Add Header</button>
  </form>
  <ul id="headers-list"></ul>
  <script src="popup.js"></script>
</body>
</html>
```

### Step 5: Create `popup.js`

This file handles the logic for adding and displaying headers.

```javascript
document.getElementById('header-form').addEventListener('submit', function(event) {
  event.preventDefault();

  let headerName = document.getElementById('header-name').value;
  let headerValue = document.getElementById('header-value').value;

  chrome.storage.sync.get(['headers'], function(result) {
    let headers = result.headers || [];
    headers.push({name: headerName, value: headerValue});
    chrome.storage.sync.set({headers: headers}, function() {
      displayHeaders(headers);
    });
  });
});

function displayHeaders(headers) {
  let headersList = document.getElementById('headers-list');
  headersList.innerHTML = '';
  headers.forEach(header => {
    let listItem = document.createElement('li');
    listItem.textContent = `${header.name}: ${header.value}`;
    headersList.appendChild(listItem);
  });
}

// Load headers on popup open
chrome.storage.sync.get(['headers'], function(result) {
  displayHeaders(result.headers || []);
});
```

### Step 6: Create `popup.css`

This file contains the styles for your popup.

```css
body {
  font-family: Arial, sans-serif;
  width: 300px;
  padding: 10px;
}

h1 {
  font-size: 16px;
}

form {
  margin-bottom: 10px;
}

input {
  width: 100%;
  padding: 5px;
  margin: 5px 0;
}

button {
  width: 100%;
  padding: 5px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  background: #f0f0f0;
  padding: 5px;
  margin: 5px 0;
}
```

### Step 7: Add Icons

Add icons of sizes 16x16, 48x48, and 128x128 to your project folder and name them `icon16.png`, `icon48.png`, and `icon128.png`.

### Step 8: Load Your Extension

1. Open Chrome and go to `chrome://extensions/`.
2. Enable "Developer mode" using the toggle in the top right.
3. Click on the "Load unpacked" button and select your project folder.

Your extension should now be loaded and ready to use. You can test it by clicking on the extension icon and adding headers.

### Final Notes

This is a basic example to get you started. Depending on your needs, you might want to add more features such as the ability to delete headers, edit existing headers, or more sophisticated error handling. You can refer to the [Chrome Extension Documentation](https://developer.chrome.com/docs/extensions/mv3/getstarted/) for more details and advanced features.

# Will this add headers to every request? How can I modify it so it only adds the header to specofied urls?

Yes, the current implementation will add headers to every request. To modify it so that headers are added only to specified URLs, you can allow users to specify which URLs they want the headers to be added to and store this information in the `chrome.storage.sync`. 

Here's how you can modify the code to include this functionality:

### Step 1: Modify `popup.html` to include URL input

Add an input field for specifying URLs:

```html
<!DOCTYPE html>
<html>
<head>
  <title>ModHeader Clone</title>
  <link rel="stylesheet" type="text/css" href="popup.css">
</head>
<body>
  <h1>ModHeader Clone</h1>
  <form id="header-form">
    <input type="text" id="url-pattern" placeholder="URL Pattern" required>
    <input type="text" id="header-name" placeholder="Header Name" required>
    <input type="text" id="header-value" placeholder="Header Value" required>
    <button type="submit">Add Header</button>
  </form>
  <ul id="headers-list"></ul>
  <script src="popup.js"></script>
</body>
</html>
```

### Step 2: Modify `popup.js` to handle URL patterns

Update the script to store and display URL patterns along with headers:

```javascript
document.getElementById('header-form').addEventListener('submit', function(event) {
  event.preventDefault();

  let urlPattern = document.getElementById('url-pattern').value;
  let headerName = document.getElementById('header-name').value;
  let headerValue = document.getElementById('header-value').value;

  chrome.storage.sync.get(['headers'], function(result) {
    let headers = result.headers || [];
    headers.push({urlPattern: urlPattern, name: headerName, value: headerValue});
    chrome.storage.sync.set({headers: headers}, function() {
      displayHeaders(headers);
    });
  });
});

function displayHeaders(headers) {
  let headersList = document.getElementById('headers-list');
  headersList.innerHTML = '';
  headers.forEach(header => {
    let listItem = document.createElement('li');
    listItem.textContent = `${header.urlPattern} - ${header.name}: ${header.value}`;
    headersList.appendChild(listItem);
  });
}

// Load headers on popup open
chrome.storage.sync.get(['headers'], function(result) {
  displayHeaders(result.headers || []);
});
```

### Step 3: Modify `background.js` to apply headers to specified URLs

Update the background script to add headers only to the specified URLs:

```javascript
chrome.webRequest.onBeforeSendHeaders.addListener(
  function(details) {
    let headers = details.requestHeaders;

    // Retrieve the headers from storage
    return new Promise((resolve) => {
      chrome.storage.sync.get(['headers'], function(result) {
        if (result.headers) {
          result.headers.forEach(header => {
            let urlPattern = new RegExp(header.urlPattern);
            if (urlPattern.test(details.url)) {
              headers.push({name: header.name, value: header.value});
            }
          });
        }
        resolve({requestHeaders: headers});
      });
    });
  },
  {urls: ["<all_urls>"]},
  ["blocking", "requestHeaders"]
);
```

### Final Steps

1. **Reload your extension**: After modifying the code, reload your extension from `chrome://extensions/`.

2. **Test your extension**: Click on the extension icon, add URL patterns, headers, and test if the headers are applied only to the specified URLs.

### Summary

This updated code allows users to specify URL patterns for which the headers should be added. The `background.js` script uses these patterns to conditionally add headers to requests matching the specified patterns. This ensures that headers are only added to requests for the specified URLs.