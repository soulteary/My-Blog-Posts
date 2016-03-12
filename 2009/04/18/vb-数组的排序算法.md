# [vb]数组的排序算法

```vb
Option Explicit   

Dim startTime As Double  
Dim endTime As Double  
Dim timeToSort As Double  
Dim timeTaken As String  

Sub Command1_Click()   
  Dim lMyArray(0 To 2000) As Long  
  Dim vTemp1 As Variant  
  Dim vTemp2 As Variant  
  Dim vTemp3 As Variant  
  Dim iLoop As Integer  

  Randomize   
  For iLoop = LBound(lMyArray) To UBound(lMyArray)   
    lMyArray(iLoop) = Int(Rnd * 100) + 1   
  Next iLoop   
  vTemp1 = lMyArray   
  vTemp2 = lMyArray   
  vTemp3 = lMyArray   
  Screen.MousePointer = vbHourglass   
  Call BubbleSortNumbers(vTemp1)   
  Call SelectionSortNumbers(vTemp2)   
  Call ShellSortNumbers(vTemp3)   
  Screen.MousePointer = vbDefault   
  MsgBox timeTaken   
End Sub  

Sub BubbleSortNumbers(iArray As Variant)   
  Dim lLoop1 As Long  
  Dim lLoop2 As Long  
  Dim lTemp As Long  

  startTime = Time()   
  For lLoop1 = UBound(iArray) To LBound(iArray) Step -1   
    For lLoop2 = LBound(iArray) + 1 To lLoop1   
      If iArray(lLoop2 - 1) > iArray(lLoop2) Then  
        lTemp = iArray(lLoop2 - 1)   
        iArray(lLoop2 - 1) = iArray(lLoop2)   
        iArray(lLoop2) = lTemp   
      End If  
    Next lLoop2   
  Next lLoop1   
  endTime = Time()   
  timeToSort = endTime - startTime   
  timeTaken = "Bubble Sort: " & timeToSort   
End Sub  

Sub SelectionSortNumbers(vArray As Variant)   
  Dim lLoop1 As Long  
  Dim lLoop2 As Long  
  Dim lMin As Long  
  Dim lTemp As Long  

  startTime = Time()   
  For lLoop1 = LBound(vArray) To UBound(vArray) - 1   
    lMin = lLoop1   
      For lLoop2 = lLoop1 + 1 To UBound(vArray)   
        If vArray(lLoop2) < vArray(lMin) Then lMin = lLoop2   
      Next lLoop2   
      lTemp = vArray(lMin)   
      vArray(lMin) = vArray(lLoop1)   
      vArray(lLoop1) = lTemp   
  Next lLoop1   
  endTime = Time()   
  timeToSort = endTime - startTime   
  timeTaken = timeTaken & ";   Selection Sort: " & timeToSort   
End Sub  

Sub ShellSortNumbers(vArray As Variant)   
  Dim lLoop1 As Long  
  Dim lHold As Long  
  Dim lHValue As Long  
  Dim lTemp As Long  

  startTime = Time()   
  lHValue = LBound(vArray)   
  Do  
    lHValue = 3 * lHValue + 1   
  Loop Until lHValue > UBound(vArray)   
  Do  
    lHValue = lHValue / 3   
    For lLoop1 = lHValue + LBound(vArray) To UBound(vArray)   
      lTemp = vArray(lLoop1)   
      lHold = lLoop1   
      Do While vArray(lHold - lHValue) > lTemp   
        vArray(lHold) = vArray(lHold - lHValue)   
        lHold = lHold - lHValue   
        If lHold < lHValue Then Exit Do  
      Loop  
      vArray(lHold) = lTemp   
    Next lLoop1   
  Loop Until lHValue = LBound(vArray)   
  endTime = Time()   
  timeToSort = endTime - startTime   
  timeTaken = timeTaken & ";   Shell Sort: " & timeToSort   
End Sub

```

