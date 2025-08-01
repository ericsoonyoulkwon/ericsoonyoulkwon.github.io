# Windows 11 Startup Automation: Open Everything You Need Right When You Log In

#### Category: Productivity

---

Do you find yourself launching the same apps, folders, and websites every morning after starting your PC? What if you could skip all those clicks and let your computer handle it for you?

In this guide, you’ll learn how to automatically open files, folders, and websites when you log in to Windows 11 — no special coding ability required. Whether you’re aiming to set up a productive work environment or a smooth personal browsing routine, this simple automation will help you start your day smarter.

You’ll discover:

- How to launch any file or folder automatically at startup

- How to open multiple Chrome tabs in a single window

- What to avoid when using browser-based startup settings

- And how to do all this without moving your original files or cluttering your screen
  

## Step 1: Automatically Open Files and Folders at Startup

Windows has a special folder — the Startup folder — that launches everything inside it when you log in.

To automate file or folder opening, we’ll place shortcuts into this folder, rather than the original files. This ensures your documents and folders stay neatly in place.


### How to Open the Startup Folder

1. Press Win + R on your keyboard

2. Type:
   ```shell
   shell:startup
   ```

3. Press Enter
   
  ![name-split-formula](/images/shell-startup.png)

### How to Create a Shortcut to Any File or Folder

You need to move the shortcuts to the startup folder you just opened. If you’re unfamiliar with shortcut creation, don’t worry — here’s how to do it for both files and folders.

#### To create a shortcut to a folder:

1. Find the file or folder in File Explorer (e.g., C:\Program Files\Microsoft Office\root\Office16 if you want to create a shortcut for Outlook)

2. Right-click the file or folder

3. Click Show more options (if you're on Windows 11)

4. Select Create shortcut.
   
  ![name-split-formula](/images/create-shortcut.png)

A shortcut named “Outlook - Shortcut” will appear in the Desktop.

### Move Shortcuts to the Startup Folder

Once your shortcuts are created:

1. Open the folder containing the shortcut (Desktop if the shortcut icon was just created in the Desktop)

2. Right-click the shortcut → Cut

3. Navigate to the Startup folder (shell:startup)

4. Right-click inside the folder → Paste

  ![name-split-formula](/images/startup-folder.png)

That’s it! When you log in next time, these items will open automatically — no manual clicks required.

You can repeat the step 1 to place the shortcut icons of as many as files or folders you want to automatically launch when you log in to your machine.



## Step 2: Automatically Open Tabs in any internet browser — the Smart Way

Opening multiple browser tabs automatically is a huge time-saver. But if you add individual shortcuts for each website, Chrome will launch a separate window for every page — and that gets messy.

Let’s avoid that by opening all tabs in a single Chrome window.

You can do this in two ways:

### Option A: Create a Desktop Shortcut with Multiple URLs

This is the fastest method — great if you're only opening a few sites.

1. Right-click your desktop → New → Shortcut

2. In the location box, type:
   ```bash
   "C:\Program Files\Google\Chrome\Application\chrome.exe" https://site1.com https://site2.com https://site3.com
   ```

3. Click Next → Name the shortcut (e.g., “Chrome_open_tabs”) → Finish

4. Move the shortcut into the Startup folder (shell:startup)

   

Limitation: 

This method only works if the entire command (including all URLs) stays under 2048 characters — which includes spaces, quotes, and links. If you have long URLs or many sites, this limit is easily reached.


### Option B (Recommended): Use a Batch File for Unlimited URLs

A batch file avoids the character limit and gives you full control over how and when your browser launches.

1. Open Notepad

2. Paste this code (update with your links):
   ```bat
   @echo off
   set CHROME="C:\Program Files\Google\Chrome\Application\chrome.exe"
   start "" %CHROME% "https://site1.com" "https://site2.com" "https://site3.com"
  
3. Save it as Chrome_open_tabs.bat
- File → Save As
- Save as type: **All Files**
- File name: Chrome_open_tabs.**bat**
- Choose a safe location like Documents\Scripts

4. Right-click the .bat file → Create shortcut

5. Move the shortcut into the Startup folder

  ![name-split-formula](/images/chrome-tabs-shortcut.png)

Now Chrome will launch once, with all your sites opened as tabs in a single window — just the way it should.



### Chrome's Built-in Startup Settings (Optional)

Chrome also lets you configure certain pages to open whenever the browser itself starts.

1. Open Chrome and go to:

  ```arduino
  chrome://settings/onStartup
  ```
2. Choose “Open a specific page or set of pages”

3. Add your favorite websites

#### Why You Might Not Want This

While this seems convenient, there’s a catch:
These pages will open every single time you launch Chrome, not just at startup.

That means if you close Chrome during the day and reopen it just to check something, all those tabs will come flooding back again — even when you don’t want them.

For most users, it’s better to open key tabs only once — when you log in — and this is why the shortcut or batch file methods are more practical.

Also, if you are setting up this automation on your work computer, this Chrome setting may be managed by the organization and you may require admin access from your IT.


### Other browser than Chrome? No problem.

These steps also work for Microsoft Edge, Firefox, and most browsers — just change the path to the browser executable.

Simply replace some part of the bat file according to the browser you want to open the pages with.

```bat
@echo off
set EDGE="C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
start "" %EDGE% "https://site1.com" "https://site2.com" "https://site3.com"
```

Notes:
This uses the default installation path for Microsoft Edge on a 64-bit Windows system.

If you're unsure of the path, you can verify it by navigating to:
C:\Program Files (x86)\Microsoft\Edge\Application\ and checking for msedge.exe.


## Final Thoughts

Windows gives you powerful — but underused — tools for building a smooth, personalized startup routine.

By placing just a few shortcuts in the Startup folder or using a batch file for internet browsers, you can save precious time every morning and launch your day with everything in place.


### Bonus Tips

Use folders like Documents\Scripts or Documents\Shortcuts to keep things organized.


**Now you have everything ready the moment your computer boots up.**
