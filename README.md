# SImple-Browser-Extension
Simple Browser extension. It just automatically refreshes the browser and downloads some partial content of the webpage
-----------
To create a simple browser extension that automatically refreshes the browser and downloads partial content from the webpage, we can follow these steps. We'll break down the task into the following components:

    Manifest File - Required for defining the extension's structure.
    Background Script - This script will manage the refreshing process and content downloading.
    Content Script - This will be injected into the webpage to extract and download the partial content.
    Popup (Optional) - To control the extension (e.g., start/stop refreshing).

Step-by-Step Code Implementation
1. manifest.json

The manifest file defines the extension, its permissions, and background or content scripts. Here, we define a simple extension that requires the ability to access web pages and refresh them.

{
  "manifest_version": 3,
  "name": "Auto Refresh and Download Content",
  "description": "Automatically refreshes the page and downloads partial content.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "downloads"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html"
  },
  "host_permissions": [
    "http://*/*",
    "https://*/*"
  ]
}

2. background.js

This script controls the automatic refreshing. It will check the tab every few seconds and refresh the page.

let refreshInterval;
let refreshRate = 5000; // Refresh every 5 seconds

// Listen for commands from popup or content scripts
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "startRefreshing") {
    startRefreshing();
  }
  
  if (message.action === "stopRefreshing") {
    stopRefreshing();
  }
  
  sendResponse({ status: "received" });
});

// Function to start automatic refreshing
function startRefreshing() {
  refreshInterval = setInterval(() => {
    chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
      chrome.tabs.reload(tabs[0].id); // Refresh the active tab
    });
  }, refreshRate);
}

// Function to stop automatic refreshing
function stopRefreshing() {
  clearInterval(refreshInterval);
}

3. content.js

This script is injected into the webpage to extract specific content after each refresh. For demonstration purposes, we'll extract content from a specified HTML element (like a <div> with a particular class) and save it as a text file.

// Function to extract partial content from the page
function extractContent() {
  let content = document.querySelector('.target-content'); // Replace with actual selector
  if (content) {
    let contentText = content.innerText || content.textContent;
    
    // Download the content as a .txt file
    chrome.runtime.sendMessage({ action: "downloadContent", contentText });
  } else {
    console.log('No content found to download.');
  }
}

// Extract content on page load (after every refresh)
extractContent();

4. popup.html

This file is optional. It's used to control the extension via the popup UI, allowing the user to start and stop the auto-refreshing behavior.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Auto Refresh Extension</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 10px; }
    button { margin: 5px; padding: 10px; }
  </style>
</head>
<body>
  <h2>Auto Refresh and Download Content</h2>
  <button id="startBtn">Start Auto Refresh</button>
  <button id="stopBtn">Stop Auto Refresh</button>

  <script src="popup.js"></script>
</body>
</html>

5. popup.js

This script handles the logic to start and stop the refreshing process based on user interaction.

document.getElementById('startBtn').addEventListener('click', () => {
  chrome.runtime.sendMessage({ action: 'startRefreshing' });
});

document.getElementById('stopBtn').addEventListener('click', () => {
  chrome.runtime.sendMessage({ action: 'stopRefreshing' });
});

6. Download Content Feature

Modify the background.js file to handle the download of content when the message is received from the content script.

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "downloadContent") {
    const contentText = message.contentText;
    
    // Create a Blob and download it as a .txt file
    const blob = new Blob([contentText], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    
    chrome.downloads.download({
      url: url,
      filename: 'content.txt',
      saveAs: true
    });
  }
  sendResponse({ status: 'download started' });
});

Testing the Extension

    Load your extension into Chrome:
        Go to chrome://extensions/ and enable "Developer Mode".
        Click "Load unpacked" and select the folder containing your extension files.
    Navigate to a webpage with content you'd like to extract.
    Open the extension popup and click "Start Auto Refresh". The extension will automatically refresh the page and download content after each refresh.
    Click "Stop Auto Refresh" to halt the refreshing process.

Conclusion

This extension will automatically refresh the browser and extract content (e.g., from a specific div element) on every refresh. The content will then be downloaded as a .txt file. You can expand on this by adding more features like handling different content types, adding more complex filtering, or customizing the download options.
