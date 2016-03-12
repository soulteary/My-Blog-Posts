# [vb]如何制作弹性小球

```vb
Option Explicit  
 
Dim MoveX, MoveY As Integer 
 
Private Sub Form_Load()  
 
    With Me 
 
        .Height = 6810  
        .Width = 7365  
 
    End With 
 
    With shpRect  
        .Top = 120  
        .Left = 120  
        .Width = 6975  
        .Height = 6135  
    End With 
 
    With shpCircle  
      
        .Top = 120  
        .Left = 120  
        .Height = 615  
        .Width = 735  
      
    End With 
 
    MoveX = 1: MoveY = 1  
      
End Sub 
 
Private Sub Timer1_Timer()  
 
    With shpCircle  
 
        .Left = .Left + (MoveX * 80)  
      
        .Top = .Top + (MoveY * 80)  
 
        Select Case .Left  
 
            Case Is <= 120: MoveX = MoveX * -1  
      
            Case Is >= 6360: MoveX = MoveX * -1  
      
        End Select 
 
        Select Case .Top  
 
            Case Is <= 120: MoveY = MoveY * -1  
      
            Case Is >= 5640: MoveY = MoveY * -1  
 
        End Select 
 
    End With 
 
End Sub 
```

