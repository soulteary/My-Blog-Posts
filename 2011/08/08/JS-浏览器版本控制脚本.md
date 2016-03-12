# [JS]浏览器版本控制脚本

收藏两个微软开发小组做的JS,相当精干。实现的功能，和实现的细节，相当舒服。

```js
// ----------------------------------------------------------------------------------
// Microsoft Developer & Platform Evangelism
//
// Copyright (c) Microsoft Corporation. All rights reserved.
//
// THIS CODE AND INFORMATION ARE PROVIDED "AS IS" WITHOUT WARRANTY OF ANY KIND,
// EITHER EXPRESSED OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES
// OF MERCHANTABILITY AND/OR FITNESS FOR A PARTICULAR PURPOSE.
// ----------------------------------------------------------------------------------
// The example companies, organizations, products, domain names,
// e-mail addresses, logos, people, places, and events depicted
// herein are fictitious.  No association with any real company,
// organization, product, domain name, email address, logo, person,
// places, or events is intended or should be inferred.
// ----------------------------------------------------------------------------------
 
// Enumerators
var BUH_positionType = {
  "Fixed" : 0,
  "Absolute" : 1
  };
// ---

// Settings
var BUH_showAllBrowsers = true;
var BUH_cookieName = 'IE8Detector';
var BUH_rememberUserSettings = false;
var BUH_position = BUH_positionType.Fixed;
var BUH_scriptPath = "script/";
// ---

// Default Text
var BUH_IE8_MessageTitle = "You may want to enhance your browser";
var BUH_IE8_MessageDownloadLink = "Make it yours";
var BUH_OldIE_MessageTitle = "Upgrade for free your browser";
var BUH_OldIE_MessageDownloadLink = "Download now";
var BUH_OtherBrowser_MessageTitle = "You may want to try the new Internet Explorer 8";
var BUH_OtherBrowser_MessageDownloadLink = "Download now";
// ---

// URLs
var BUH_IE8_DownloadPage_Url = "http://go.microsoft.com/?linkid=9701257";
var BUH_WindowsXP_32Bit_InstallerUrl = "http://go.microsoft.com/?linkid=9701258";
var BUH_WindowsServer2003_32Bit_InstallerUrl = "http://go.microsoft.com/?linkid=9701262";
var BUH_WindowsVistaWindowsServer2008_32Bit_InstallerUrl = "http://go.microsoft.com/?linkid=9701260";
var BUH_WindowsXPWindowsServer2003_64bit_InstallerUrl = "http://go.microsoft.com/?linkid=9701263";
var BUH_WindowsVistaWindowsServer2008_64bit_InstallerUrl = "http://go.microsoft.com/?linkid=9701261";
// ---

// Preliminary operations
if (typeof IsRunningIE8 == 'function') {
    
    if (BUH_isUpgradeControlToDisplay(BUH_cookieName)) {      
     BUH_addCssReference();
     BUH_showHeader();
  }
} else { alert("Some JavaScript functions are missing.\nPlease verify all external files and references."); }
// ---

// Preliminary operations
if (typeof IsRunningIE9 == 'function') {

    if (BUH_isUpgradeControlToDisplay(BUH_cookieName)) {
        BUH_addCssReference();
       // BUH_showHeader();
    }
} else { alert("Some JavaScript functions are missing.\nPlease verify all external files and references."); }
// ---

// Functions
function BUH_isUpgradeControlToDisplay(cookieName) {
  if (BUH_rememberUserSettings && document.cookie.indexOf(cookieName + "=") != -1) { return false; }
  if (GetBrowser() != BrowserType.MSInternetExplorer) { return BUH_showAllBrowsers; }
  if (IsRunningIE8() || IsRunningIE9()) { return false; }
  
  return true;
}

function BUH_addCssReference() {
  var fileref = document.createElement("link");
  var fileName;

  if (BUH_position == BUH_positionType.Absolute) {
     fileName = "headerAbsolute.css";
  } else if (GetIEVersion() == IEVersion.IE6) {
            fileName = "headerFixedIE6.css";
         } else { fileName = "headerFixed.css"; }

  fileref.setAttribute("rel", "stylesheet");
  fileref.setAttribute("type", "text/css");
  fileref.setAttribute("href", BUH_scriptPath + "CSS/" + fileName);
  document.getElementsByTagName("head")[0].appendChild(fileref);
}

function BUH_closeUpgradeBox(containerID1, containerID2, cookieName) {
  var exdate = new Date();

  exdate.setDate(exdate.getDate() + 356);
  document.cookie = cookieName + '=NoDisplay; expires=' + exdate.toGMTString();
  document.getElementById(containerID1).style.display = 'none';
  document.getElementById(containerID2).style.display = 'none';
  return false;
}

function BUH_getDownloadLink()
{
  if (GetBrowser() == BrowserType.MSInternetExplorer) {
     var currentOS = GetOperatingSystem();
     var currentArch = GetArchitecture();

     if (currentOS == OperatingSystem.WinXP) {
	if (currentArch == Architecture.x32) { return BUH_WindowsXP_32Bit_InstallerUrl; }
	if (currentArch == Architecture.x64) { return BUH_WindowsXPWindowsServer2003_64bit_InstallerUrl; }
     } else if (currentOS == OperatingSystem.WinVistaOrWinServer2008) {
   	       if (currentArch == Architecture.x32) { return BUH_WindowsVistaWindowsServer2008_32Bit_InstallerUrl; }
               if (currentArch == Architecture.x64) { return BUH_WindowsVistaWindowsServer2008_64bit_InstallerUrl; }
            } else if (currentOS == OperatingSystem.WinServer2003OrWinXPx64) {
	              if (currentArch == Architecture.x32) { return BUH_WindowsServer2003_32Bit_InstallerUrl; }
		      if (currentArch == Architecture.x64) { return BUH_WindowsXPWindowsServer2003_64bit_InstallerUrl; }
                   } else if (currentOS == OperatingSystem.Win7OrWinServer2008R2) {
	                     if (currentArch == Architecture.x32) { return BUH_IE8_DownloadPage_Url; }
		             if (currentArch == Architecture.x64) { return BUH_IE8_DownloadPage_Url; }
                          }
  }
		     return BUH_IE8_DownloadPage_Url;
		      
}

function BUH_showMessageTitle()
{
//  if (GetBrowser() != BrowserType.MSInternetExplorer) { return BUH_OtherBrowser_MessageTitle; }
////  if (GetIEVersion() == IEVersion.IE8) { return BUH_IE8_MessageTitle; }
//  return BUH_OldIE_MessageTitle;
}

function BUH_showMessageDownloadLink()
{ 
//  if (GetBrowser() != BrowserType.MSInternetExplorer) { return BUH_OtherBrowser_MessageDownloadLink; }
////  if (GetIEVersion() == IEVersion.IE8) { return BUH_IE8_MessageDownloadLink; }
 // return BUH_OldIE_MessageDownloadLink;
}
// ---

// Control rendering
function BUH_showHeader() {

    if (GetBrowser() == BrowserType.MSInternetExplorer) {

        document.write("<div style=\"clear:both; height:59px; position:relative;\">");
        document.write("<a href=\"http://112.125.57.5/tracking_ie8_dowload.html" + "\"target=\"_blank\">");
        document.write("<img src=\"images/warning_bar_0027_Simplified Chinese.jpg\"" + "\"border=\"0\" height=\"42\" width=\"820\" border=\"0\"");
        document.write("alt=\"You are using an outdated browser. For a faster, safer browsing experience, upgrade for free today.\"");
        document.write("/></a><\/div>");

    }
}
</pre>

<pre lang="javascript">
// ----------------------------------------------------------------------------------
// Microsoft Developer & Platform Evangelism
//
// Copyright (c) Microsoft Corporation. All rights reserved.
//
// THIS CODE AND INFORMATION ARE PROVIDED "AS IS" WITHOUT WARRANTY OF ANY KIND,
// EITHER EXPRESSED OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES
// OF MERCHANTABILITY AND/OR FITNESS FOR A PARTICULAR PURPOSE.
// ----------------------------------------------------------------------------------
// The example companies, organizations, products, domain names,
// e-mail addresses, logos, people, places, and events depicted
// herein are fictitious.  No association with any real company,
// organization, product, domain name, email address, logo, person,
// places, or events is intended or should be inferred.
// ----------------------------------------------------------------------------------

// Regex Settings
var IERegularExpression = new RegExp("MSIE ([0-9]{1,})");
var FFRegularExpression = new RegExp("Firefox");
var ChromeRegularExpression = new RegExp("Chrome");
var OperaRegularExpression = new RegExp("Opera");
var SafariRegularExpression = new RegExp("Safari");
var TridentRegularExpression = new RegExp("Trident\/([0-9]{1,}[\.0-9]{0,})");
 
// ---

// Enumerators
var Architecture = {
  "x32" : "x32",
  "x64" : "x64",
  "InformationNotAvailable" : "InformationNotAvailable"
  };
var BrowserType = {
  "MSInternetExplorer" : "MSInternetExplorer",
  "Firefox" : "Firefox",
  "Chrome" : "Chrome",
  "Safari" : "Safari",
  "Opera" : "Opera",
  "Other" : "Other"
  };
var IEVersion = {
  "IE5orBelow" : "IE5orBelow",
  "IE6" : "IE6",
  "IE7" : "IE7",
  "IE8": "IE8",
  "IE9": "IE9",
  "InformationNotAvailable" : "InformationNotAvailable"
  };
var OperatingSystem = {
  "WinXP" : "WinXP",
  "Win2000" : "Win2000",
  "WinServer2003OrWinXPx64" : "WinServer2003OrWinXPx64",
  "WinVistaOrWinServer2008" : "WinVistaOrWinServer2008",
  "Win7OrWinServer2008R2" : "Win7OrWinServer2008R2",
  "Win98" : "Win98",
  "Win95" : "Win95",
  "SunOS" : "SunOS",
  "Macintosh" : "Macintosh",
  "Linux" : "Linux",
  "Other" : "Other"
  };
// ---

var currentUserAgent = navigator.userAgent;

function GetBrowser() {
  if (IERegularExpression.test(currentUserAgent))   { return BrowserType.MSInternetExplorer; }
  if (FFRegularExpression.test(currentUserAgent))   { return BrowserType.Firefox; }
  if (ChromeRegularExpression.test(currentUserAgent)) { return BrowserType.Chrome; }
  if (OperaRegularExpression.test(currentUserAgent)) { return BrowserType.Opera; }
  if (SafariRegularExpression.test(currentUserAgent)) { return BrowserType.Safari; }
  return BrowserType.Other; 
}

function GetIEVersion() { ;
  if (GetBrowser() == BrowserType.MSInternetExplorer)
  { 
      if (IsRunningIE8()) { return IEVersion.IE8; }

      if (IsRunningIE9()) { return IEVersion.IE8; }
      
    var version = IERegularExpression.exec(currentUserAgent)[0];

    if (version == "MSIE 7") { return IEVersion.IE7; }
    if (version == "MSIE 6") { return IEVersion.IE6; }
    return IEVersion.IE5orBelow;
  }
  return IEVersion.InformationNotAvailable;
}

function IsRunningIE8() {
   
  if (GetBrowser() == BrowserType.MSInternetExplorer && TridentRegularExpression.test(currentUserAgent)) {
     return (TridentRegularExpression.exec(currentUserAgent)[0] == "Trident/4.0");
  }
  return false;
}

function IsRunningIE9() {  
    if (GetBrowser() == BrowserType.MSInternetExplorer && TridentRegularExpression.test(currentUserAgent)) {
        return (TridentRegularExpression.exec(currentUserAgent)[0] == "Trident/5.0");
    }
    return false;
} 

function GetLanguage() {
  var language = navigator.language ? navigator.language : navigator.userLanguage;
 
  return language.substr(0, 2).toLowerCase();
}

function GetOperatingSystem() {
  var agent = currentUserAgent.toUpperCase();

  if (agent.indexOf("WINDOWS NT 6.1") != -1) {
    return OperatingSystem.Win7OrWinServer2008R2;
  }
  if (agent.indexOf("WINDOWS NT 6.0") != -1) {
    return OperatingSystem.WinVistaOrWinServer2008;
  }
  if (agent.indexOf("WINDOWS NT 5.2") != -1) {
    return OperatingSystem.WinServer2003OrWinXPx64;
  }
  if (agent.indexOf("WINDOWS NT 5.1") != -1 || agent.indexOf("WINDOWS XP") != -1) {
    return OperatingSystem.WinXP;
  }
  if (agent.indexOf("WINDOWS NT 5.0") != -1 || agent.indexOf("WINDOWS 2000") != -1) {
    return OperatingSystem.Win2000;
  }
  if (agent.indexOf("WINDOWS 98") != -1 || agent.indexOf("WIN98") != -1) {
    return OperatingSystem.Win98;
  }
  if (agent.indexOf("WINDOWS 95") != -1 || agent.indexOf("WIN95") != -1 || agent.indexOf("WINDOWS_95") != -1) {
    return OperatingSystem.Win95;
  }
  if (agent.indexOf("SUNOS") != -1) {
    return OperatingSystem.SunOS;
  }
  if (agent.indexOf("MAC_POWERPC") != -1 || agent.indexOf("MACINTOSH") != -1) {
    return OperatingSystem.Macintosh;
  }
  if (agent.indexOf("X11") != -1) {
  	return OperatingSystem.Linux;
  }
  return OperatingSystem.Other;
}

function GetArchitecture() {
  var agent = currentUserAgent.toUpperCase();

  if (GetBrowser() == BrowserType.MSInternetExplorer) {
     if (agent.indexOf("WOW64") != -1 || agent.indexOf("X64") != -1 || agent.indexOf("WIN64") != -1 || agent.indexOf("IA64") != -1) {
        return Architecture.x64;
     }
     return Architecture.x32;
  }
  return Architecture.InformationNotAvailable;
}
```

