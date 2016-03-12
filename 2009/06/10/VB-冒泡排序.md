# [VB]冒泡排序

发一段以前写的冒泡排序，FOR Basic

```vb
Option Explicit
Private Sub Fir_Sort()

    Dim vArr As Variant, lngIndex As Long, lngMax As Long, lngTmp As Long

    vArr = Array(1999, 2000, 2008, 2009, 2007, 2006, 2003, 2004, 2005, 2001, 2002)
    lngMax = UBound(vArr)

    For lngIndex = 1 To lngMax
        For lngTmp = lngIndex - 1 To lngMax
            Fir_SwapB2S CLng(vArr(lngIndex - 1)), CLng(vArr(lngTmp))
        Next
    Next

    For lngIndex = 0 To lngMax
        Debug.Print vArr(lngIndex)
    Next

End Sub

Private Sub Fir_SwapB2S(lngA As Long, lngB As Long)

    If lngA < lngB Then

        Dim lngTmp As Long

        lngTmp = lngA: lngA = lngB: lngB = lngTmp
    End If

End Sub

Private Sub Fir_SwapS2B(lngA As Long, lngB As Long)

    If lngA > lngB Then

        Dim lngTmp As Long

        lngTmp = lngA: lngA = lngB: lngB = lngTmp
    End If

End Sub

Private Sub Form_Load()
    Call Fir_Sort
End Sub

```

