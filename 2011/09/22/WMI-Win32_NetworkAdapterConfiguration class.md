# [WMI]Win32_NetworkAdapterConfiguration class

Win32_NetworkAdapterConfiguration class 这个类包含了相当丰富的内容.

资料来源:[TECHNET脚本库](http://promiseforever.com/redirect?url=http://www.microsoft.com/china/technet/community/scriptcenter/network/scrnet116.mspx&key=b947053e684d0794147f7f2c7a930848 "TECHNET脚本库")以及MSDN aa394217.

<!-- more -->

这个类能干嘛呢,获取DNS,IP,设置IP,DNS等各种属性,还有就是获得网卡等详细信息.
WMI获取ADAPTER信息脚本A.

```vbs
strComputer = "." 
Set objWMIService = GetObject("winmgmts:\\" strComputer & "\root\CIMV2") 
Set colItems = objWMIService.ExecQuery( "SELECT * FROM Win32_NetworkAdapterConfiguration",,48) 
For Each objItem in colItems 
    Wscript.Echo "-----------------------------------"
    Wscript.Echo "Win32_NetworkAdapterConfiguration instance"
    Wscript.Echo "-----------------------------------"
    Wscript.Echo "ArpAlwaysSourceRoute: " & objItem.ArpAlwaysSourceRoute
    Wscript.Echo "ArpUseEtherSNAP: " & objItem.ArpUseEtherSNAP
    Wscript.Echo "DatabasePath: " & objItem.DatabasePath
    Wscript.Echo "DeadGWDetectEnabled: " & objItem.DeadGWDetectEnabled
    Wscript.Echo "DefaultIPGateway: " & strDefaultIPGateway
    Wscript.Echo "DefaultTOS: " & objItem.DefaultTOS
    Wscript.Echo "DefaultTTL: " & objItem.DefaultTTL
    Wscript.Echo "Description: " & objItem.Description
    Wscript.Echo "DHCPEnabled: " & objItem.DHCPEnabled
    Wscript.Echo "DHCPLeaseExpires: " & objItem.DHCPLeaseExpires
    Wscript.Echo "DHCPLeaseObtained: " & objItem.DHCPLeaseObtained
    Wscript.Echo "DHCPServer: " & objItem.DHCPServer
    Wscript.Echo "DNSDomain: " & objItem.DNSDomain
    Wscript.Echo "DNSDomainSuffixSearchOrder: " & strDNSDomainSuffixSearchOrder
    Wscript.Echo "DNSEnabledForWINSResolution: " & objItem.DNSEnabledForWINSResolution
    Wscript.Echo "DNSHostName: " & objItem.DNSHostName
    Wscript.Echo "DNSServerSearchOrder: " & strDNSServerSearchOrder
    Wscript.Echo "DomainDNSRegistrationEnabled: " & objItem.DomainDNSRegistrationEnabled
    Wscript.Echo "ForwardBufferMemory: " & objItem.ForwardBufferMemory
    Wscript.Echo "FullDNSRegistrationEnabled: " & objItem.FullDNSRegistrationEnabled
    Wscript.Echo "GatewayCostMetric: " & strGatewayCostMetric
    Wscript.Echo "IGMPLevel: " & objItem.IGMPLevel
    Wscript.Echo "Index: " & objItem.Index
    Wscript.Echo "IPAddress: " & strIPAddress
    Wscript.Echo "IPConnectionMetric: " & objItem.IPConnectionMetric
    Wscript.Echo "IPEnabled: " & objItem.IPEnabled
    Wscript.Echo "IPFilterSecurityEnabled: " & objItem.IPFilterSecurityEnabled
    Wscript.Echo "IPPortSecurityEnabled: " & objItem.IPPortSecurityEnabled
    Wscript.Echo "IPSecPermitIPProtocols: " & strIPSecPermitIPProtocols
    Wscript.Echo "IPSecPermitTCPPorts: " & strIPSecPermitTCPPorts
    Wscript.Echo "IPSecPermitUDPPorts: " & strIPSecPermitUDPPorts
    Wscript.Echo "IPSubnet: " & strIPSubnet
    Wscript.Echo "IPUseZeroBroadcast: " & objItem.IPUseZeroBroadcast
    Wscript.Echo "IPXAddress: " & objItem.IPXAddress
    Wscript.Echo "IPXEnabled: " & objItem.IPXEnabled
    Wscript.Echo "IPXFrameType: " & strIPXFrameType
    Wscript.Echo "IPXNetworkNumber: " & strIPXNetworkNumber
    Wscript.Echo "IPXVirtualNetNumber: " & objItem.IPXVirtualNetNumber
    Wscript.Echo "KeepAliveInterval: " & objItem.KeepAliveInterval
    Wscript.Echo "KeepAliveTime: " & objItem.KeepAliveTime
    Wscript.Echo "MACAddress: " & objItem.MACAddress
    Wscript.Echo "MTU: " & objItem.MTU
    Wscript.Echo "NumForwardPackets: " & objItem.NumForwardPackets
    Wscript.Echo "PMTUBHDetectEnabled: " & objItem.PMTUBHDetectEnabled
    Wscript.Echo "PMTUDiscoveryEnabled: " & objItem.PMTUDiscoveryEnabled
    Wscript.Echo "ServiceName: " & objItem.ServiceName
    Wscript.Echo "SettingID: " & objItem.SettingID
    Wscript.Echo "TcpipNetbiosOptions: " & objItem.TcpipNetbiosOptions
    Wscript.Echo "TcpMaxConnectRetransmissions: " & objItem.TcpMaxConnectRetransmissions
    Wscript.Echo "TcpMaxDataRetransmissions: " & objItem.TcpMaxDataRetransmissions
    Wscript.Echo "TcpNumConnections: " & objItem.TcpNumConnections
    Wscript.Echo "TcpUseRFC1122UrgentPointer: " & objItem.TcpUseRFC1122UrgentPointer
    Wscript.Echo "TcpWindowSize: " & objItem.TcpWindowSize
    Wscript.Echo "WINSEnableLMHostsLookup: " & objItem.WINSEnableLMHostsLookup
    Wscript.Echo "WINSHostLookupFile: " & objItem.WINSHostLookupFile
    Wscript.Echo "WINSPrimaryServer: " & objItem.WINSPrimaryServer
    Wscript.Echo "WINSScopeID: " & objItem.WINSScopeID
    Wscript.Echo "WINSSecondaryServer: " & objItem.WINSSecondaryServer
Next
</pre>
脚本A升级版
<pre lang="c">
PrintAll_NICAdapter_information()
Function PrintAll_NICAdapter_information()

    strComputer = "."
    Set objWMIService = GetObject("winmgmts:\\" _
    & strComputer & "\root\CIMV2")

    Set colItems = objWMIService.ExecQuery( _
    "SELECT * FROM Win32_NetworkAdapterConfiguration",,48)

    i = 0
    For Each objItem in colItems
        i = i + 1
        Wscript.Echo "-----------------------------------"
        Wscript.Echo "Win32_NetworkAdapterConfiguration instance: " & i
        Wscript.Echo "-----------------------------------"
        
        strDefaultIPGateway = GetMultiString_FromArray(objitem.DefaultIPGateway, ", ")
        Wscript.Echo "MACAddress                  : " & vbtab & objItem.MACAddress
        Wscript.Echo "Description                 : " & vbtab & objItem.Description
        Wscript.Echo "DHCPEnabled                 : " & vbtab & objItem.DHCPEnabled

        strIPAddress=GetMultiString_FromArray(objitem.IPAddress, ", ")
        Wscript.Echo "IPAddress                   : " & vbtab & strIPAddress
        strIPSubnet = GetMultiString_FromArray(objitem.IPSubnet, ", ")
        Wscript.Echo "IPSubnet                    : " & vbtab & strIPSubnet
        Wscript.Echo "IPConnectionMetric          : " & vbtab & objItem.IPConnectionMetric
        Wscript.Echo "DHCPLeaseExpires            : " & vbtab & objItem.DHCPLeaseExpires
        Wscript.Echo "DHCPLeaseObtained           : " & vbtab & objItem.DHCPLeaseObtained
        Wscript.Echo "DHCPServer                  : " & vbtab & objItem.DHCPServer
        Wscript.Echo "DNSDomain                   : " & vbtab & objItem.DNSDomain
        Wscript.Echo "IPEnabled                   : " & vbtab & objItem.IPEnabled
        Wscript.Echo "DefaultIPGateway            : " & vbtab & strDefaultIPGateway
        Wscript.Echo "GatewayCostMetric           : " & vbtab & strGatewayCostMetric
        Wscript.Echo "IPFilterSecurityEnabled     : " & vbtab & objItem.IPFilterSecurityEnabled
        Wscript.Echo "IPPortSecurityEnabled       : " & vbtab & objItem.IPPortSecurityEnabled

        strDNSDomainSuffixSearchOrder = GetMultiString_FromArray(objitem.DNSDomainSuffixSearchOrder, ", ")
        Wscript.Echo "DNSDomainSuffixSearchOrder  : " & vbtab & strDNSDomainSuffixSearchOrder
        Wscript.Echo "DNSEnabledForWINSResolution : " & vbtab & objItem.DNSEnabledForWINSResolution
        Wscript.Echo "DNSHostName                 : " & vbtab & objItem.DNSHostName
        
        strDNSServerSearchOrder = GetMultiString_FromArray(objitem.DNSServerSearchOrder, ", ")
        Wscript.Echo "DNSServerSearchOrder        : " & vbtab & strDNSServerSearchOrder
        Wscript.Echo "DomainDNSRegistrationEnabled: " & vbtab & objItem.DomainDNSRegistrationEnabled
        Wscript.Echo "ForwardBufferMemory         : " & vbtab & objItem.ForwardBufferMemory
        Wscript.Echo "FullDNSRegistrationEnabled  : " & vbtab & objItem.FullDNSRegistrationEnabled

        strGatewayCostMetric = GetMultiString_FromArray(objitem.GatewayCostMetric, ", ")
        Wscript.Echo "IGMPLevel                   : " & vbtab & objItem.IGMPLevel
        Wscript.Echo "Index                       : " & vbtab & objItem.Index

        strIPSecPermitIPProtocols = GetMultiString_FromArray(objitem.IPSecPermitIPProtocols, ", ")
        Wscript.Echo "IPSecPermitIPProtocols      : " & vbtab & strIPSecPermitIPProtocols

        strIPSecPermitTCPPorts =GetMultiString_FromArray(objitem.IPSecPermitTCPPorts, ", ")
        Wscript.Echo "IPSecPermitTCPPorts         : " & vbtab & strIPSecPermitTCPPorts

        strIPSecPermitUDPPorts = GetMultiString_FromArray(objitem.IPSecPermitUDPPorts, ", ")
        Wscript.Echo "IPSecPermitUDPPorts         : " & vbtab & strIPSecPermitUDPPorts

        Wscript.Echo "IPUseZeroBroadcast          : " & vbtab & objItem.IPUseZeroBroadcast
        Wscript.Echo "IPXAddress                  : " & vbtab & objItem.IPXAddress
        Wscript.Echo "IPXEnabled                  : " & vbtab & objItem.IPXEnabled

        strIPXFrameType=GetMultiString_FromArray(objitem.IPXFrameType, ", ")
        Wscript.Echo "IPXFrameType                : " & vbtab & strIPXFrameType

        strIPXNetworkNumber=GetMultiString_FromArray(objitem.IPXNetworkNumber, ", ")
        Wscript.Echo "IPXNetworkNumber            : " & vbtab & strIPXNetworkNumber
        Wscript.Echo "IPXVirtualNetNumber         : " & vbtab _
                & objItem.IPXVirtualNetNumber
        Wscript.Echo "KeepAliveInterval           : " & vbtab _
                & objItem.KeepAliveInterval
        Wscript.Echo "KeepAliveTime               : " & vbtab & objItem.KeepAliveTime
        Wscript.Echo "MTU                         : " & vbtab & objItem.MTU
        Wscript.Echo "NumForwardPackets           : " & vbtab & objItem.NumForwardPackets
        Wscript.Echo "PMTUBHDetectEnabled         : " & vbtab & objItem.PMTUBHDetectEnabled
        Wscript.Echo "PMTUDiscoveryEnabled        : " & vbtab & objItem.PMTUDiscoveryEnabled
        Wscript.Echo "ServiceName                 : " & vbtab & objItem.ServiceName
        Wscript.Echo "SettingID                   : " & vbtab & objItem.SettingID
        Wscript.Echo "TcpipNetbiosOptions         : " & vbtab & objItem.TcpipNetbiosOptions
        Wscript.Echo "TcpMaxConnectRetransmissions: " & vbtab & objItem.TcpMaxConnectRetransmissions
        Wscript.Echo "TcpMaxDataRetransmissions   : " & vbtab & objItem.TcpMaxDataRetransmissions
        Wscript.Echo "TcpNumConnections           : " & vbtab & objItem.TcpNumConnections
        Wscript.Echo "TcpUseRFC1122UrgentPointer  : " & vbtab & objItem.TcpUseRFC1122UrgentPointer
        Wscript.Echo "TcpWindowSize               : " & vbtab & objItem.TcpWindowSize
        Wscript.Echo "WINSEnableLMHostsLookup     : " & vbtab & objItem.WINSEnableLMHostsLookup
        Wscript.Echo "WINSHostLookupFile          : " & vbtab & objItem.WINSHostLookupFile
        Wscript.Echo "WINSPrimaryServer           : " & vbtab & objItem.WINSPrimaryServer
        Wscript.Echo "WINSScopeID                 : " & vbtab & objItem.WINSScopeID
        Wscript.Echo "WINSSecondaryServer         : " & vbtab & objItem.WINSSecondaryServer
        Wscript.Echo "ArpAlwaysSourceRoute        : " & vbtab & objItem.ArpAlwaysSourceRoute
        Wscript.Echo "ArpUseEtherSNAP             : " & vbtab & objItem.ArpUseEtherSNAP
        Wscript.Echo "DatabasePath                : " & vbtab & objItem.DatabasePath
        Wscript.Echo "DeadGWDetectEnabled         : " & vbtab & objItem.DeadGWDetectEnabled
        Wscript.Echo "DefaultTOS                  : " & vbtab & objItem.DefaultTOS
        Wscript.Echo "DefaultTTL                  : " & vbtab & objItem.DefaultTTL
        
    Next
End Function

sub appendCollection(msg, colctn, nm)
    i=0
    for each t in colctn
        msg = msg & "nic." & nm & "["&i&"]: " & t & vbCRLF
        i = i + 1
    next
end sub

Function PrintOnlyEnabled_NICAdapter_information()
    strComputer = "."

    Set objWMIService = GetObject("winmgmts:{impersonationLevel=impersonate}!\\" & strComputer & "\root\cimv2")
    Set colNicConfigs = objWMIService.ExecQuery ("SELECT * FROM Win32_NetworkAdapterConfiguration WHERE IPEnabled = True")

    for each nic in colNicConfigs
        msg = "nic.ArpAlwaysSourceRoute: " & nic.ArpAlwaysSourceRoute & vbCRLF _
        & "nic.ArpUseEtherSNAP: " & nic.ArpUseEtherSNAP & vbCRLF _
        & "nic.Caption: " & nic.Caption & vbCRLF _
        & "nic.DatabasePath: " & nic.DatabasePath & vbCRLF _
        & "nic.DeadGWDetectEnabled: " & nic.DeadGWDetectEnabled & vbCRLF _
        & "nic.DefaultTOS: " & nic.DefaultTOS & vbCRLF _
        & "nic.DefaultTTL: " & nic.DefaultTTL & vbCRLF _
        & "nic.Description: " & nic.Description & vbCRLF _
        & "nic.DHCPEnabled: " & nic.DHCPEnabled & vbCRLF _
        & "nic.DHCPLeaseExpires: " & nic.DHCPLeaseExpires & vbCRLF _
        & "nic.DHCPLeaseObtained: " & nic.DHCPLeaseObtained & vbCRLF _
        & "nic.DHCPServer: " & nic.DHCPServer & vbCRLF _
        & "nic.DNSDomain: " & nic.DNSDomain & vbCRLF _
        & "nic.DNSEnabledForWINSResolution: " & nic.DNSEnabledForWINSResolution & vbCRLF _
        & "nic.DNSHostName: " & nic.DNSHostName & vbCRLF _
        & "nic.DomainDNSRegistrationEnabled: " & nic.DomainDNSRegistrationEnabled & vbCRLF _
        & "nic.DNSDomainSuffixSearchOrder: " & nic.DNSDomainSuffixSearchOrder & vbCRLF _
        & "nic.ForwardBufferMemory: " & nic.ForwardBufferMemory & vbCRLF _
        & "nic.FullDNSRegistrationEnabled: " & nic.FullDNSRegistrationEnabled & vbCRLF _
        & "nic.IGMPLevel: " & nic.IGMPLevel & vbCRLF _
        & "nic.Index: " & nic.Index & vbCRLF _
        & "nic.IPConnectionMetric: " & nic.IPConnectionMetric & vbCRLF _
        & "nic.IPEnabled: " & nic.IPEnabled & vbCRLF _
        & "nic.IPFilterSecurityEnabled: " & nic.IPFilterSecurityEnabled & vbCRLF _
        & "nic.IPPortSecurityEnabled: " & nic.IPPortSecurityEnabled & vbCRLF _
        & "nic.IPUseZeroBroadcast: " & nic.IPUseZeroBroadcast & vbCRLF _
        & "nic.IPXAddress: " & nic.IPXAddress & vbCRLF _
        & "nic.IPXEnabled: " & nic.IPXEnabled & vbCRLF _
        & "nic.IPXFrameType: " & nic.IPXFrameType & vbCRLF _
        & "nic.IPXMediaType: " & nic.IPXMediaType & vbCRLF _
        & "nic.IPXNetworkNumber: " & nic.IPXNetworkNumber & vbCRLF _
        & "nic.IPXVirtualNetNumber: " & nic.IPXVirtualNetNumber & vbCRLF _
        & "nic.KeepAliveInterval: " & nic.KeepAliveInterval & vbCRLF _
        & "nic.KeepAliveTime: " & nic.KeepAliveTime & vbCRLF _
        & "nic.MACAddress: " & nic.MACAddress & vbCRLF _
        & "nic.MTU: " & nic.MTU & vbCRLF _
        & "nic.NumForwardPackets: " & nic.NumForwardPackets & vbCRLF _
        & "nic.PMTUBHDetectEnabled: " & nic.PMTUBHDetectEnabled & vbCRLF _
        & "nic.PMTUDiscoveryEnabled: " & nic.PMTUDiscoveryEnabled & vbCRLF _
        & "nic.ServiceName: " & nic.ServiceName & vbCRLF _
        & "nic.SettingID: " & nic.SettingID & vbCRLF _
        & "nic.TcpipNetbiosOptions: " & nic.TcpipNetbiosOptions & vbCRLF _
        & "nic.TcpMaxConnectRetransmissions: " & nic.TcpMaxConnectRetransmissions & vbCRLF _
        & "nic.TcpMaxDataRetransmissions: " & nic.TcpMaxDataRetransmissions & vbCRLF _
        & "nic.TcpNumConnections: " & nic.TcpNumConnections & vbCRLF _
        & "nic.TcpUseRFC1122UrgentPointer: " & nic.TcpUseRFC1122UrgentPointer & vbCRLF _
        & "nic.TcpWindowSize: " & nic.TcpWindowSize & vbCRLF _
        & "nic.WINSEnableLMHostsLookup: " & nic.WINSEnableLMHostsLookup & vbCRLF _
        & "nic.WINSHostLookupFile: " & nic.WINSHostLookupFile & vbCRLF _
        & "nic.WINSPrimaryServer: " & nic.WINSPrimaryServer & vbCRLF _
        & "nic.WINSScopeID: " & nic.WINSScopeID & vbCRLF _
        & "nic.WINSSecondaryServer: " & nic.WINSSecondaryServer & vbCRLF _
        '& "nic.InterfaceIndex: " & "|"&nic.InterfaceIndex & vbCRLF _

        appendCollection msg, nic.DefaultIPGateway, "DefaultIPGateway"
        appendCollection msg, nic.DNSServerSearchOrder, "DNSServerSearchOrder"
        appendCollection msg, nic.GatewayCostMetric, "GatewayCostMetric"
        appendCollection msg, nic.IPAddress, "IPAddress"
        appendCollection msg, nic.IPSecPermitIPProtocols, "IPSecPermitIPProtocols"
        appendCollection msg, nic.IPSecPermitTCPPorts, "IPSecPermitTCPPorts"
        appendCollection msg, nic.IPSecPermitUDPPorts, "IPSecPermitUDPPorts"
        appendCollection msg, nic.IPSubnet, "IPSubnet"

        WScript.Echo msg
    next

End Function

Function GetMultiString_FromArray( ArrayString, Seprator)
    If IsNull ( ArrayString ) Then
        StrMultiArray = ArrayString
    else
        StrMultiArray = Join( ArrayString, Seprator )
   end if
   GetMultiString_FromArray = StrMultiArray
   
End Function
</pre>
如果需要列出有效的网卡相当简单。
网卡基本都有的常规属性是什么，网卡地址咯。
<pre lang="c">Set colItems = objWMIService.ExecQuery("SELECT * FROM Win32_NetworkAdapterConfiguration WHERE MACAddress!='' AND IPEnabled =TRUE ", , 48)</pre>

最后发一个获取DNS服务器搜索以及设置的脚本
<pre lang="c">
On Error Resume Next
strComputer = "."
Set objWMIService = GetObject("winmgmts:\\" & strComputer & "\root\cimv2")
Set colNetCards = objWMIService.ExecQuery _
    ("Select * From Win32_NetworkAdapterConfiguration Where IPEnabled = True")
For Each objNetCard in colNetCards
    arrDNSServers = Array("192.168.1.100", "192.168.1.200")
'这里如果调用SET类函数的话,就是设置,如果你要write出来那么就是获取.记得获取的数据是数据集合,那个集合的每个成员基本都是数组,调用的时候加下标或者判断或者转换下.
    objNetCard.SetDNSServerSearchOrder(arrDNSServers)
Next
```

最后贴一份MSDN保存起来,MSDN老变动..

原帖各种失效 [download id="75"]

