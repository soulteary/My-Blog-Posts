# [API]字体、语言相关函数原型

感谢ChinaAVG shane007的辛苦整理哈~！ 

【汉化资料】字体、语言相关的API函数原型 字体、语言相关的API函数原型：

```text
createFont 用指定的属性创建一种逻辑字体
createFontIndirect 用指定的属性创建一种逻辑字体
GetStockObject 取得一个固有对象（笔、刷子、字体等）
GetACP 检取ANSI系统代码页的标识符
GetOEMCP 检取系统的OEM代码页标识符
GetTextExtentPoint 判断一个字串的大小（范围）(用于限制输入)
EnumFontFamilies 列举指定设备可用的字体
EnumFontFamiliesEx 列举指定设备可用的字体
EnumFonts 列举指定设备可用的字体
GetSystemDefaultLangID 取得系统的默认语言ID
GetUserDefaultLangID 为当前用户取得默认语言ID
GetStringTypeA 获取ANSI字符串类型 ,返回指定字符串的字符类型信息
GetStringTypeW 返回一个Unicode串的字符类型信息
GetSystemDefaultLangID 检取系统缺省语言标识符
WideCharToMultiByte 把一个宽字符串映射为一个新字符串
```

```c
HFONT CreateFont( 
int nHeight, // logical height of font 
int nWidth, // logical average character width 
int nEscapement, // angle of escapement 
int nOrientation, // base-line orientation angle 
int fnWeight, // font weight 
DWORD fdwItalic, // italic attribute flag 
DWORD fdwUnderline, // underline attribute flag 
DWORD fdwStrikeOut, // strikeout attribute flag 
DWORD fdwCharSet, // character set identifier 
DWORD fdwOutputPrecision, // output precision 
DWORD fdwClipPrecision, // clipping precision 
DWORD fdwQuality, // output quality 
DWORD fdwPitchAndFamily, // pitch and family 
LPCTSTR lpszFace // pointer to typeface name string 
);

HFONT CreateFontIndirect( 
CONST LOGFONT *lplf // pointer to logical font structure 
);

HGDIOBJ GetStockObject( 
int fnObject // type of stock object 
);
```

fnObject Specifies the type of stock object. This parameter can be any one of the following values: Value 4.BLACK_BRUSH Black brush. 3.DKGRAY_BRUSH Dark gray brush. 2.GRAY_BRUSH Gray brush. 5.HOLLOW_BRUSH Hollow brush (equivalent to NULL_BRUSH). 1.LTGRAY_BRUSH Light gray brush. 5.NULL_BRUSH Null brush (equivalent to HOLLOW_BRUSH). 0.WHITE_BRUSH White brush. 7.BLACK_PEN Black pen. 8.NULL_PEN Null pen. 6.WHITE_PEN White pen. 11.（0B）ANSI_FIXED_FONT Windows fixed-pitch (monospace) system font. 12.（0C）ANSI_VAR_FONT Windows variable-pitch (proportional space) system font. 14.（0E）DEVICE_DEFAULT_FONT Windows NT only: Device-dependent font. 17.（11）DEFAULT_GUI_FONT Windows 95 only: Default font for user interface objects such as menus and dialog boxes.（一般选这种） 10.（0A）OEM_FIXED_FONT Original equipment manufacturer (OEM) dependent fixed-pitch (monospace) font. 13.（0D）SYSTEM_FONT System font. By default, Windows uses the system font to draw menus, dialog box controls, and text. In Windows versions 3.0 and later, the system font is a proportionally spaced font; earlier versions of Windows used a monospace system font. 16.（10）SYSTEM_FIXED_FONT Fixed-pitch (monospace) system font used in Windows versions earlier than 3.0\. This stock object is provided for compatibility with earlier versions of Windows. 15.DEFAULT_PALETTE Default palette. This palette consists of the static colors in the system palette. If the function succeeds, the return value identifies the logical object requested. If the function fails, the return value is NULL. ================================================================================= The GetACP function retrieves the current ANSI code-page identifier for the system.

```c
UINT GetACP(VOID)
```

Parameters This function has no parameters. Return Values If the function succeeds, the return value is the current ANSI code-page identifier for the system, or a default identifier if no code page is current. Remarks Following are the ANSI code-page identifiers: Identifier Meaning 874 Thai 932 Japan 936 Chinese (PRC, Singapore) 949 Korean 950 Chinese (Taiwan, Hong Kong) 1200 Unicode (BMP of ISO 10646) 1250 Windows 3.1 Eastern European 1251 Windows 3.1 Cyrillic 1252 Windows 3.1 Latin 1 (US, Western Europe) 1253 Windows 3.1 Greek 1254 Windows 3.1 Turkish 1255 Hebrew 1256 Arabic 1257 Baltic The GetOEMCP function retrieves the current OEM code-page identifier for the system. (OEM stands for original equipment manufacturer.)

```c
UINT GetOEMCP(VOID)
```

Parameters This function has no parameters. Return Values If the function succeeds, the return value is the current OEM code-page identifier for the system or a default identifier if no code page is current. Remarks Following are the OEM code-page identifiers: Identifier Meaning 437 MS-DOS United States 708 Arabic (ASMO 708) 709 Arabic (ASMO 449+, BCON V4) 710 Arabic (Transparent Arabic) 720 Arabic (Transparent ASMO) 737 Greek (formerly 437G) 775 Baltic 850 MS-DOS Multilingual (Latin I) 852 MS-DOS Slavic (Latin II) 855 IBM Cyrillic (primarily Russian) 857 IBM Turkish 860 MS-DOS Portuguese 861 MS-DOS Icelandic 862 Hebrew 863 MS-DOS Canadian-French 864 Arabic 865 MS-DOS Nordic 866 MS-DOS Russian (former USSR) 869 IBM Modern Greek 874 Thai 932 Japan 936 Chinese (PRC, Singapore) 949 Korean 950 Chinese (Taiwan, Hong Kong) 1361 Korean (Johab)

```c
BOOL GetTextExtentPoint(
HDC hdc, // handle of device context
LPCTSTR lpString, // address of text string
int cbString, // number of characters in string
LPSIZE lpSize // address of structure for string size
);
```

The EnumFontFamilies function enumerates the fonts in a specified font family that are available on a specified device. This function supersedes the EnumFonts function.

```c
int EnumFontFamilies(
HDC hdc, // handle to device control
LPCTSTR lpszFamily, // pointer to family-name string
FONTENUMPROC lpEnumFontFamProc, // pointer to callback function
LPARAM lParam // address of application-supplied data
);
```

Parameters hdc Identifies the device context. lpszFamily Points to a null-terminated string that specifies the family name of the desired fonts. If lpszFamily is NULL, EnumFontFamilies randomly selects and enumerates one font of each available type family. lpEnumFontFamProc Specifies the procedure-instance address of the application-defined callback function. For information about the callback function, see the EnumFontFamProc function. lParam Points to application-supplied data. The data is passed to the callback function along with the font information. Return Values If the function succeeds, the return value is the last value returned by the callback function. Its meaning is implementation specific. ============================================================================= The EnumFontFamiliesEx function enumerates all fonts in the system that match the font characteristics specified by the LOGFONT structure. EnumFontFamiliesEx enumerates fonts based on typeface name, character set, or both. It is recommended that Win32-based applications use this function rather than EnumFontFamilies to enumerate fonts. int EnumFontFamiliesEx( HDC hdc, // handle to device context LPLOGFONT lpLogfont, // pointer to logical font information FONTENUMPROC lpEnumFontFamExProc, // pointer to callback function LPARAM lParam, // application-supplied data DWORD dwFlags // reserved; must be zero ); ================================================================================= The EnumFonts function enumerates the fonts available on a specified device. For each font with the specified typeface name, the EnumFonts function retrieves information about that font and passes it to the application-defined callback function. This callback function can process the font information as desired. Enumeration continues until there are no more fonts or the callback function returns zero. This function is provided for compatibility with earlier versions of Microsoft Windows. Win32-based applications should use the EnumFontFamilies function. int EnumFonts( HDC hdc, // handle to device context LPCTSTR lpFaceName, // pointer to font typeface name string FONTENUMPROC lpFontFunc, // pointer to callback function LPARAM lParam // address of application-supplied data ); Parameters hdc Identifies the device context. lpFaceName Points to a null-terminated character string that specifies the typeface name of the desired fonts. If lpFaceName is NULL, EnumFonts randomly selects and enumerates one font of each available typeface. lpFontFunc Points to the application-defined callback function. For more information about the callback function, see the EnumFontsProc function. lParam Points to any application-defined data. The data is passed to the callback function along with the font information. Return Values If the function succeeds, the return value is the last value returned by the callback function. Its meaning is defined by the application. ====================================================================== The GetSystemDefaultLangID function retrieves the system default language identifier. LANGID GetSystemDefaultLangID(VOID) Parameters This function has no parameters. Return Values If the function succeeds, the return value is the system default language identifier. ================================================================================ The GetUserDefaultLangID function retrieves the user default language identifier. LANGID GetUserDefaultLangID(VOID) Parameters This function has no parameters. Return Values If the function succeeds, the return value is the user default language identifier. ================================================================================ BOOL GetStringTypeA( LCID Locale, // locale identifer DWORD dwInfoType, // information-type options LPCSTR lpSrcStr, // pointer to the source string int cchSrc, // size, in bytes, of the source string LPWORD lpCharType // pointer to the buffer for output ); Parameters Locale LOCALE_SYSTEM_DEFAULT Default system locale LOCALE_USER_DEFAULT Default user locale dwInfoType CT_CTYPE1 Retrieve character type information. CT_CTYPE2 Retrieve bidirectional layout information. CT_CTYPE3 Retrieve text processing information. lpSrcStr Points to the string for which character types are requested. If cchSrc is -1, the string is assumed to be null terminated. This must be an ANSI string. Note that this can be a double-byte character set (DBCS) string if the locale is appropriate for DBCS. cchSrc Specifies the size, in bytes, of the string pointed to by the lpSrcStr parameter. If this count includes a null terminator, the function returns character type information for the null terminator. If this value is -1, the string is assumed to be null terminated and the length is calculated automatically. lpCharType Points to an array of 16-bit values. The length of this array must be large enough to receive one 16-bit value for each character in the source string. When the function returns, this array contains one word corresponding to each character in the source string. Return Values If the function succeeds, the return value is nonzero. ================================================================= BOOL GetStringTypeW( DWORD dwInfoType, // information-type options LPCWSTR lpSrcStr, // address of source string int cchSrc, // number of characters in string LPWORD lpCharType // address of buffer for output ); ========================================================================== The GetSystemDefaultLangID function retrieves the system default language identifier. LANGID GetSystemDefaultLangID(VOID) Parameters This function has no parameters. Return Values If the function succeeds, the return value is the system default language identifier. Remarks For more information about language identifiers, see Language Identifiers and Locales.

```c
int WideCharToMultiByte(
UINT CodePage, // code page
DWORD dwFlags, // performance and mapping flags
LPCWSTR lpWideCharStr, // address of wide-character string
int cchWideChar, // number of characters in string
LPSTR lpMultiByteStr, // address of buffer for new string
int cchMultiByte, // size of buffer
LPCSTR lpDefaultChar, // address of default for unmappable characters
LPBOOL lpUsedDefaultChar // address of flag set when default char. used
);
```

Parameters CodePage Specifies the code page used to perform the conversion. This parameter can be given the value of any codepage that is installed or available in the system. The following values may be used to specify one of the system default code pages: Value Meaning CP_ACP ANSI code page CP_MACCP Macintosh code page CP_OEMCP OEM code page dwFlags A set of bit flags that specify the handling of unmapped characters. The function performs more quickly when none of these flags is set. The following flag constants are defined: Value Meaning WC_COMPOSITECHECK Convert composite characters to precomposed characters. WC_DISCARDNS Discard nonspacing characters during conversion. WC_SEPCHARS Generate separate characters during conversion. This is the default conversion behavior. WC_DEFAULTCHAR Replace exceptions with the default character during conversion. When WC_COMPOSITECHECK is specified, the function converts composite characters to precomposed characters. A composite character consists of a base character and a nonspacing character, each having different character values. A precomposed character has a single character value for a base/nonspacing character combination. In the character the e is the base character, and the accent grave mark is the nonspacing character. When an application specifies WC_COMPOSITECHECK, it can use the last 3 flags in this list (WC_DISCARDNS, WC_SEPCHARS, and WC_DEFAULTCHAR) to customize the conversion to precomposed characters. These flags determine the functions behavior when there is no precomposed mapping for a base/nonspace character combination in a wide-character string. These last 3 flags can only be used if the WC_COMPOSITECHECK flag is set. The functions default behavior is to generate separate characters (WC_SEPCHARS) for unmapped composite characters. lpWideCharStr Points to the wide-character string to be converted. cchWideChar Specifies the number of characters in the string pointed to by the lpWideCharStr parameter. If this value is -1, the string is assumed to be null-terminated and the length is calculated automatically. lpMultiByteStr Points to the buffer to receive the translated string. cchMultiByte Specifies the size in characters of the buffer pointed to by the lpMultiByteStr parameter. If this value is zero, the function returns the number of bytes required for the buffer. (In this case, the lpMultiByteStr buffer is not used.) lpDefaultChar Points to the character used if a wide character cannot be represented in the specified code page. If this parameter is NULL, a system default value is used. The function is faster when both lpDefaultChar and lpUsedDefaultChar are NULL. lpUsedDefaultChar Points to a flag that indicates whether a default character was used. The flag is set to TRUE if one or more wide characters in the source string cannot be represented in the specified code page. Otherwise, the flag is set to FALSE. This parameter may be NULL. The function is faster when both lpDefaultChar and lpUsedDefaultChar are NULL. Return Values If the function succeeds, and cchMultiByte is nonzero, the return value is the number of bytes written to the buffer pointed to by lpMultiByteStr. If the function succeeds, and cchMultiByte is zero, the return value is the required size, in bytes, for a buffer that can receive the translated string. If the function fails, the return value is zero. To get extended error information, call GetLastError. GetLastError may return one of the following error codes: ERROR_INSUFFICIENT_BUFFER ERROR_INVALID_FLAGS ERROR_INVALID_PARAMETER

