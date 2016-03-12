# [vb]获得汉字的区位码

获得汉字的区位码 CSDN:Hassle原创

```vb
Option   Explicit      
       
  Private   Declare   Sub   CopyMemory   Lib   "kernel32"   Alias   "RtlMoveMemory"   (Destination   As   Any,   Source   As   Any,   ByVal   Length   As   Long)      
       
  Private   Sub   Command1_Click()      
          MsgBox   Convert("保")      
  End   Sub      
       
  Private   Function   Convert(ByVal   sChar   As   String)   As   String      
          Dim   aBuf(0   To   1)   As   Byte      
          Dim   nCode   As   Integer      
          Dim   s   As   String      
          Dim   i   As   Long      
       
          Convert   =   ""      
          nCode   =   Asc(sChar)      
          CopyMemory   aBuf(0),   nCode,   2      
          For   i   =   0   To   1      
                  aBuf(i)   =   aBuf(i)   -   &HA0      
                  s   =   aBuf(i)      
                  If   Len(s)   =   1   Then   s   =   0   &   s      
                  Convert   =   s   &   Convert      
          Next      
  End   Function      
```


