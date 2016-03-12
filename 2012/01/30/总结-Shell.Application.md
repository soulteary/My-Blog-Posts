# [总结]Shell.Application

09年收集的一篇资料,看到草稿中放着,就发出来吧. 1.对象的创建 ASP.NET中创建Shell对象

```vb
var Shell = new ActiveXObject(Shell.Application);
```

VB中创建Shell对象

```vb
Set Shell = CreateObject("Shell.Application")
```

2.Shell对象的属性和方法[是不是和FSO很相似呢] 至此是不是获取Windows常用对象的方法又多了一种呢

```vb
Shell.Application
 Shell.Parent
 Shell.CascadeWindows()
 Shell.TileHorizontally()
 Shell.TileVertically()
 Shell.ControlPanelItem(sDir)
 Shell.EjectPC()
 Shell.Explore(vDir)
 Shell.Open(vDir)
 Shell.FileRun()
 Shell.FindComputer()
 Shell.FindFiles()
 Shell.Help()
 Shell.MinimizeAll()
 Shell.UndoMinimizeALL()
 Shell.RefreshMenu()
 Shell.SetTime()
 Shell.TrayProperties()
 Shell.ShutdownWindows()
 Shell.Suspend()
 Shell.Windows() 
 Shell.NameSpace(vDir) 
 Shell.BrowseForFolder(Hwnd, sTitle, iOptions [, vRootFolder]) 

  function BrowseFolder()
  {
   var Message = 清选择文件夹;

   var Shell  = new ActiveXObject( Shell.Application );
   var Folder = Shell.BrowseForFolder(0,Message,0x0040,0x11);
   if(Folder != null)
   {
    Folder = Folder.items(); '返回 FolderItems 对象
    Folder = Folder.item();  '返回 Folderitem 对象
    Folder = Folder.Path;    '返回路径
    if(Folder.charAt(varFolder.length-1) != \\){
     Folder = varFolder + \\;
    }
    return Folder;
   }
  }

  var Folder = Shell.NameSpace(C:\\); ' 返回 Folder对象
```

3、使用 Folder 对象

```vb
 [ oApplication = ] Folder.Application   // Contains the Application object.
 [ oParentFolder= ] Folder.ParentFolder   // Contains the parent Folder object.
 [    oTitle    = ] Folder.Title    // Contains the title of the folder.

 Folder.CopyHere(vItem [, vOptions])   // Copies an item or items to a folder.
 Folder.MoveHere(vItem [, vOptions])   // Moves an item or items to this folder.
'  vItem:  Required. Specifies the item or items to move. This can be a string that represents a file name, a FolderItem object, or a FolderItems object. 
'    vOptions Optional. Specifies options for the move operation. This value can be zero or a combination of the following values. These values are based upon flags defined for use with the fFlags member of the C++ SHFILEOPSTRUCT structure. These flags are not defined as such for Microsoft? Visual Basic?, Visual Basic Scripting Edition (VBScript), or Microsoft JScript?, so you must define them yourself or use their numeric equivalents.
'   4  Do not display a progress dialog box.  
'   8  Give the file being operated on a new name in a move, copy, or rename operation if a file with the target name already exists.  
'   16  Respond with Yes to All for any dialog box that is displayed.  
'   64  Preserve undo information, if possible. 
'   128 Perform the operation on files only if a wildcard file name (*.*) is specified.  
'   256  Display a progress dialog box but do not show the file names.  
'   512  Do not confirm the creation of a new directory if the operation requires one to be created.  
'   1024 Do not display a user interface if an error occurs.  
'   2048  Version 4.71\. Do not copy the security attributes of the file. 
'   4096  Only operate in the local directory. Dont operate recursively into subdirectories. 
'   9182 Version 5.0\. Do not move connected files as a group. Only move the specified files.  

 Folder.NewFolder(bName)     // Creates a new folder.
 ppid = Folder.ParseName(bName)    // Creates and returns a FolderItem object that represents a specified item.
' bName:  Required. A string that specifies the name of the item. 

 oFolderItems = Folder.Items()    // Retrieves a FolderItems object that represents the collection of items in the folder.
 sDetail = Folder.GetDetailsOf(vItem, iColumn)  // Retrieves details about an item in a folder. For example, its size, type, or the time of its last modification.
'  vItem:  Required. Specifies the item for which to retrieve the information. This must be a FolderItem object. 
'  iColumn: Required. An Integer value that specifies the information to be retrieved. The information available for an item depends on the folder in which it is displayed. This value corresponds to the zero-based column number that is displayed in a Shell view. For an item in the file system, this can be one of the following values:0 Retrieves the name of the item. 
'   1  Retrieves the size of the item. 
'   2  Retrieves the type of the item. 
'   3  Retrieves the date and time that the item was last modified. 
'   4  Retrieves the attributes of the item. 
'   -1 Retrieves the info tip information for the item. 
```

4、使用 FolderItems 对象

```vb
var FolderItems = Shell.NameSpace(C:\\).Items(); ' 返回 FolderItems 对象

 [ oApplication = ] FolderItems.Application
 [    iCount    = ] FolderItems.Count
 [    oParent   = ] FolderItems.Parent

 oFolderItem = FolderItems.Item([iIndex])  '返回 FolderItem 对象
```

5、使用 FolderItem 对象

```vb
var FolderItem = Shell.NameSpace(C:\\).Items().Item(iIndex); // 返回 FolderItems 对象

 [ oApplication = ] FolderItem.Application
 [    oParent   = ] FolderItem.Parent
 [ sName = ] FolderItem.Name(sName) [ = sName ]
 [ sPath = ] FolderItem.Path
 [ iSize = ] FolderItem.Size
 [ sType = ] FolderItem.Type
 [ bIsLink = ] FolderItem.IsLink
 [ bIsFolder = ] FolderItem.IsFolder
 [ bIsFileSystem = ] FolderItem.IsFileSystem
 [ bIsBrowsable = ] FolderItem.IsBrowsable
 [  oGetLink  = ] FolderItem.GetLink     ' 返回 ShellLinkObject 对象
 [ oGetFolder = ] FolderItem.GetFolder   ' 返回 Folder 对象
 [ oModifyDate= ] FolderItem.ModifyDate(oModifyDate) [ = oModifyDate ] 'Sets or retrieves the date and time that the item was last modified. 

 vVerb = FolderItem.Verbs()        '返回 FolderItemVerbs 对象. This object is the collection of verbs that can be executed on the item.
 FolderItem.InvokeVerb( [vVerb])  'Executes a verb on the item.
```

6、使用 FolderItemVerbs 对象

```vb
var FolderItem = Shell.NameSpace(C:\\).Items().Item(iIndex).Verbs(); '返回 FolderItems 对象

 [ oApplication = ] FolderItemVerbs.Application
 [ oParent = ] FolderItemVerbs.Parent
 [ iCount = ] FolderItemVerbs.Count

 oVerb = FolderItemVerbs.Item( [iIndex])' 返回 FolderItemVerb 对象.
```

7、使用 FolderItemVerb 对象

```vb
var FolderItem = Shell.NameSpace(C:\\).Items().Item(iIndex).Verbs().Item(iIndex); '返回 FolderItems 对象

 [ oApplication = ] FolderItemVerbs.Application
 [ oParent = ] FolderItemVerbs.Parent
 [ oName = ] FolderItemVerbs.Name

 FolderItemVerb.DoIt() 'Executes a verb on the FolderItem associated with the verb.
```

8、使用 ShellLinkObject 对象

```vb
[ sWorkingDirectory = ]ShellLinkObject.WorkingDirectory(sWorkingDirectory) [ = sWorkingDirectory ]
 [ intShowCommand = ]ShellLinkObject.ShowCommand(intShowCommand) [ = intShowCommand ]
'  intShowCommand  Integer that specifies or receives the links show state. This can be one of the following values.
'    1  Activates and displays a window. If the window is minimized or maximized, the system restores it to its original size and position. 
'    2  Activates the window and displays it as a minimized window. 
'    3  Activates the window and displays it as a maximized window. 
 [ sArguments = ] ShellLinkObject.Arguments(sArguments) [ = sArguments ]
 [ sDescription = ] ShellLinkObject.Description(sDescription) [ = sDescription ]
 [ iHotkey = ] ShellLinkObject.Hotkey(iHotkey) [ = iHotkey ]
'  iHotkey   Integer that specifies or receives the links hot key code. The virtual key code is in the low-order byte, and the modifier flags are in the high-order byte. The modifier flags can be a combination of the following values.
'   1 SHIFT key  2 CTRL key
'   4 ALT key    8 Extended key
 [ sPath = ] ShellLinkObject.Path(sPath) [ = sPath ]

 iIcon = ShellLinkObject.GetIconLocation(sPath)
 ShellLinkObject.Resolve(fFlags)
'  fFlags   Required. Flags that specify the action to be taken. This can be a combination of the following values.
'   1  Do not display a dialog box if the link cannot be resolved. When this flag is set, the high-order word of fFlags specifies a time-out duration, in milliseconds. The method returns if the link cannot be resolved within the time-out duration. If the high-order word is set to zero, the time-out duration defaults to 3000 milliseconds (3 seconds). 
'   4  If the link has changed, update its path and list of identifiers. 
'   8  Do not update the link information. 
'   16  Do not execute the search heuristics. 
'   32 Do not use distributed link tracking. 
'   64  Disable distributed link tracking. By default, distributed link tracking tracks removable media across multiple devices based on the volume name. It also uses the Universal Naming Convention (UNC) path to track remote file systems whose drive letter has changed. Setting this flag disables both types of tracking. 
'   128  Call the Microsoft? Windows? Installer.
 ShellLinkObject.Save( [sFile])
 ShellLinkObject.SetIconLocation(sPath, iIndex)
'  sPath   Required. String value that contains the fully qualified path of the file that contains the icon. 
'  iIndex   Required. Integer that is set to the index of the icon in the file specified by sPath. 
```

9、使用 ShellWindows 对象

```vb
[ intCount = ] ShellWindows.Count
oShellWindows = ShellWindows._NewEnum() 'Creates and returns a new ShellWindows object that is a copy of this ShellWindows object.
oFolder = ShellWindows.Item( [iIndex])  'Retrieves an InternetExplorer object that represents the Shell window.
```

10、说明 通过第一步创建 Shell 对象，并进行相关函数调用，就可以返回以上各种对象，并进行相关操作。另外，在学习的过程中，发现了两个在msdn中提及却没相关的函数：

```vb
ShellApp.ShellExecute(cmd.exe);
  ShellApp.NameSpace(vDir).Items().InvokeVerbEx(vVerb); 'vVerb:如delete
```

还有些特殊的用法：

```vb
var myprinterfolder = Shell.NameSpace(shell:PrintersFolder);
var mydocsfolder = Shell.NameSpace(shell:personal);
var mycompfolder = Shell.NameSpace(shell:drivefolder);

Shell.ShellExecute( wiaacmgr.exe,/SelectDevice );
Shell.ShellExecute( rundll32.exe, shell32.dll,Control_RunDLL sysdm.cpl,,1 )
Shell.ShellExecute( rundll32.exe, shell32.dll,Control_RunDLL netcpl.cpl,,1 );
Shell.ShellExecute( rundll32.exe, shell32.dll,Control_RunDLL sysdm.cpl,,1 );

'The following command will run Rundll32.exe.
Rundll32.exe <dllname>,<entrypoint>, <optional arguments="">'The following code sample shows how to use the command.
Rundll32.exe Setupx.dll,InstallHinfSection 132 C:\Windows\Inf\Shell.inf

Shell.ShowBrowserBar({C4EE31F3-4768-11D2-BE5C-00A0C9A83DA1}, true);</optional> </entrypoint></dllname>
```

11、使用 Shell.UIHelper.1 对象

```vb
ShellUI = new ActiveXObject(Shell.UIHelper.1);

ShellUI.AddChannel(sURL)
ShellUI.AddFavorite(sURL [, vTitle])
bBool = ShellUI.IsSubscribed(sURL)  'Indicates whether or not a URL is subscribed to.
ShellUI.AddDesktopComponent(sURL, sType [, Left] [, Top] [, Width] [, Height])

  sURL   'Required. A String value that specifies the URL of the new favorite item.
  sType  'Required. A String value that specifies the type of item being added. This can be one of the following values:
    image   'The component is an image.
    website 'The component is a web site.
    Left 'Optional. Specifies the position of the left edge of the component, in screen coordinates.
    Top  'Optional. Specifies the position of the top edge of the component, in screen coordinates.
```

