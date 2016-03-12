# [VB]WMI 脚本入门：第一部分，获取进程，内存，服务信息

WMI是一个强大的工具，一直有一个想法，就是把MS的一些东西用VB重写一下，最后聚合在一起，做一个实用的小工具。 文末有微软WMI页面。

```vb
Private Sub cmdCommand1_Click()

    Dim strComputerName As String

    strComputerName = VBA.Environ("ComputerName")

    Set wbemServices = GetObject("winmgmts:\\" & strComputerName)
    Set wbemObjectSet = wbemServices.InstancesOf("Win32_LogicalMemoryConfiguration")

    For Each wbemObject In wbemObjectSet

        MsgBox "Total Physical Memory (kb): " & wbemObject.TotalPhysicalMemory

    Next

End Sub
Private Sub cmdCommand2_Click()

    Dim strComputer As String

    strComputer = VBA.Environ("ComputerName")

    Set wbemServices = GetObject("winmgmts:\\" & strComputer)
    Set wbemObjectSet = wbemServices.InstancesOf("Win32_Service")

    For Each wbemObject In wbemObjectSet

        List1.AddItem "Display Name:  " & wbemObject.DisplayName & vbCrLf & "   State:      " & wbemObject.State & vbCrLf & "   Start Mode: " & wbemObject.StartMode
    Next

End Sub
Private Sub cmdCommand3_Click()

    Dim strComputer As String

    strComputer = VBA.Environ("ComputerName")

    Set wbemServices = GetObject("winmgmts:\\" & strComputer)
    Set wbemObjectSet = wbemServices.InstancesOf("Win32_NTLogEvent")

    For Each wbemObject In wbemObjectSet

        List1.AddItem "Log File:        " & wbemObject.LogFile & vbCrLf & "Record Number:   " & wbemObject.RecordNumber & vbCrLf & "Type:            " & wbemObject.Type & vbCrLf & "Time Generated:  " & wbemObject.TimeGenerated & vbCrLf & "Source:          " & wbemObject.SourceName & vbCrLf & "Category:        " & wbemObject.Category & vbCrLf & "Category String: " & wbemObject.CategoryString & vbCrLf & "Event:           " & wbemObject.EventCode & vbCrLf & "User:            " & wbemObject.User & vbCrLf & "Computer:        " & wbemObject.ComputerName & vbCrLf & "Message:         " & wbemObject.Message & vbCrLf
    Next

End Sub
Private Sub cmdCommand4_Click()

    Dim strComputer As String

    strComputer = VBA.Environ("ComputerName")

    strComputer = "."   ' Dot (.) equals local computer in WMI

    Set wbemServices = GetObject("winmgmts:\\" & strComputer)
    Set wbemObjectSet = wbemServices.InstancesOf("Win32_Process")

    For Each wbemObject In wbemObjectSet

        List1.AddItem "Name:          " & wbemObject.Name & vbCrLf & "   Handle:     " & wbemObject.Handle & vbCrLf & "   Process ID: " & wbemObject.ProcessID
    Next

End Sub

```

<!-- more -->

微软WMI入门教程离线版下载：[download id="13"]

