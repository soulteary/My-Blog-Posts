# [MFC]消息机制

MFC剖析：消息机制 出处：白云黄鹤 转载的一篇老文，有的放矢的说明了MFC消息流程。 首先,让我们看一下MFC的消息循环部分：（程序取自MFC源程序，由于篇幅，我删去了一些非重要的部分。） MFC的WinMain函数：

```c
extern "C" int WINAPI  
_tWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,  
LPTSTR lpCmdLine, int nCmdShow)  
{  
// call shared/exported WinMain  
return AfxWinMain(hInstance, hPrevInstance, lpCmdLine, nCmdShow);  
}  

int AFXAPI AfxWinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance,  

LPTSTR lpCmdLine, int nCmdShow)  
{  
int nReturnCode = -1;  
CWinApp* pApp = AfxGetApp();  

// ......  

// Perform specific initializations  
if (!pApp->InitInstance())  
{  
// ......  

//初始化实例不成功，通常一个Dialog Based MFC程序必须返回FALSE  
//这样就可以跳过消息循环。  
nReturnCode = pApp->ExitInstance();  
goto InitFailure;  
}  

nReturnCode = pApp->Run(); //进入消息循环部分  

InitFailure:  
// ......  

// 程序结束  
AfxWinTerm();  
return nReturnCode;  
}  

int CWinApp::Run()  
{  
// ......  
return CWinThread::Run(); // 消息循环被封装在CWinThread类里。  
}  

int CWinThread::Run()  
{  
BOOL bIdle = TRUE;  
LONG lIdleCount = 0;  

// 死循环，只有收到 WM_QUIT 消息后才会退出。  
for (;;)  
{  
while (bIdle &&  
!::PeekMessage(&m_msgCur, NULL, NULL, NULL, PM_NOREMOVE))  
{  
if (!OnIdle(lIdleCount++))  
bIdle = FALSE;  
}  
// 如果消息队列中没有消息，那么就调用OnIdle函数  
// 否则，发送消息  
do  
{  
if (!PumpMessage())  
// PumpMessage函数仅在收到WM_QUIT消息才返回FALSE  
return ExitInstance(); // 退出死循环  

if (IsIdleMessage(&m_msgCur))  
{  
bIdle = TRUE;  
lIdleCount = 0;  
}  
} while (::PeekMessage(&m_msgCur, NULL, NULL, NULL, PM_NOREMOVE));  
// 这段程序不仅完成了消息的发送，还实现了Idle功能。  
// GetMessage函数在消息队列中没有消息时，将不会返回，  
// 而是将控制权交给操作系统，直到消息队列中有消息为止。  
// 这段程序在一开始就调用PeekMessage函数来检测消息队列中  
// 是否有消息存在，如果存在就发送消息，  
// 否则就意味着空闲，那么就调用OnIdle函数，  
// 这样做，控制权永远不会交给操作系统。  
// 由于Windows 95, NT都是抢占式的操作系统，  
// 系统会自动进行任务切换。  
// 所以不用担心别的程序不会被运行。  
}  
}  

BOOL CWinThread::PumpMessage()  
{  
if (!::GetMessage(&m_msgCur, NULL, NULL, NULL))  
{// 收到 WM_QUIT 消息，就返回 FALSE。  
return FALSE;  
}  

// 否则就发送消息  
if (m_msgCur.message != WM_KICKIDLE && !PreTranslateMessage(&m_msgCur)
)  
{  
::TranslateMessage(&m_msgCur);  
::DispatchMessage(&m_msgCur);  
}  
return TRUE;  
}  
</pre>

主程序的流程：

<pre lang="text">(程序开始)  
|  
|  
v  
WinMain  
|  
|  
v  
AfxWinMain  
|  
|  
v FALSE  
CWinApp::InitInstance-------> 退出程序  
|  
|TRUE  
|  
v  
CWinApp::Run  
|  
|  
v  
CWinThread::Run  
|  
|<----------------------------+ 
v FALSE | 
PeekMessage------->OnIdle--------+  
|TRUE |  
|<-------------------------+ | 
v | | 
GetMessage | | 
| | | 
| | | 
YES v | | 
+-----WM_QUIT 消息？ | | 
| | | | 
| |NO | | 
| v | | 
| TranslateMessage | | 
| | | | 
| | | | 
| v | | 
| DispatchMessage | | 
| | | | 
| | | | 
| v TRUE | | 
| PeekMessage------------------>+ |  
| | |  
| |FALSE |  
| +-----------------------------+  
|  
v  
(程序结束)
</pre>

现在，再让我们来看一下MFC的窗口是如何响应消息的。 我们先来看一段建立一个窗口的代码.

<pre lang="c">class CMsgWnd : public CWnd  
{  
public:  
CMsgWnd() {}  

// ClassWizard generated virtual function overrides  
//{{AFX_VIRTUAL(CMsgWnd)  
virtual BOOL Create(CWnd* pParentWnd);  
//}}AFX_VIRTUAL  

virtual ~CMsgWnd() {}  

protected:  
//{{AFX_MSG(CMsgWnd)  
afx_msg void OnPaint();  
afx_msg void OnLButtonDown(UINT nFlags, CPoint point);  
//}}AFX_MSG  
DECLARE_MESSAGE_MAP()  
};  

BEGIN_MESSAGE_MAP(CMsgWnd, CWnd)  
//{{AFX_MSG_MAP(CMsgWnd)  
ON_WM_PAINT()  
ON_WM_LBUTTONDOWN()  
//}}AFX_MSG_MAP  
END_MESSAGE_MAP()  

BOOL CMsgWnd::Create(CWnd *pParentWnd)  
{  
return CWnd::Create(AfxRegisterWndClass(CS_DBLCLKS),  
"Message Window", WS_VISIBLE|WS_CHILD, CRect(0,0,100,100),  
pParentWnd, 12345);  
}  

void CMsgWnd::OnPaint()  
{  
CPaintDC dc(this); // device context for painting  

dc.TextOut(0,0,"hello");  
}  

void CMsgWnd::OnLButtonDown(UINT nFlags, CPoint point)  
{  
MessageBox("OnLButtonDown");  

CWnd::OnLButtonDown(nFlags, point);  
}  
</pre>

以上是一段标准的CWnd窗口类的子类实现代码. 我想大家应该是可以看的懂的. 注意到CMsgWnd类中有一句代码 DECLARE_MESSAGE_MAP() 我们来看看这个宏是如何定义的:

<pre lang="c">typedef void (AFX_MSG_CALL CCmdTarget::*AFX_PMSG)(void);  
// 这是一个指向 CCmdTarget 类成员函数的指针类型.  

struct AFX_MSGMAP_ENTRY // 消息映射表之消息入口  
{  
UINT nMessage; // 消息  
UINT nCode; // 控制码或者是 WM_NOTIFY 消息的通知码  
UINT nID; // 控件的ID,如果是窗口消息则为0  
UINT nLastID; // 如果是一个范围的消息,那么这是最后一个控件的ID  
UINT nSig; // 消息处理类型  
AFX_PMSG pfn; // 消息处理函数  
};  

struct AFX_MSGMAP // 消息映射表  
{  
#ifdef _AFXDLL // 如果MFC是动态连接的,  
// 就是编译时选择 Use MFC in a shared DLL  
const AFX_MSGMAP* (PASCAL* pfnGetBaseMap)();  
#else // MFC是静态连接的,编译时选择 Use MFC in a static library.  
const AFX_MSGMAP* pBaseMap;  
#endif  
// 如果MFC是动态连接的, 就用pfnGetBaseMap函数返回基类的消息映射表  
// 否则 pBaseBap 指向基类的消息映射表  

const AFX_MSGMAP_ENTRY* lpEntries; // 指向消息入口的指针  
};  

#ifdef _AFXDLL // 如果MFC是动态连接的  
#define DECLARE_MESSAGE_MAP() \  
private: \  
static const AFX_MSGMAP_ENTRY _messageEntries[]; \ // 消息入口  
protected: \  
static AFX_DATA const AFX_MSGMAP messageMap; \ // 消息映射表  
static const AFX_MSGMAP* PASCAL _GetBaseMessageMap(); \  
// 该函数返回基类的消息映射表  
virtual const AFX_MSGMAP* GetMessageMap() const; \  
// 该函数返回当前类的消息映射表  

#else // 静态连接的  
#define DECLARE_MESSAGE_MAP() \  
private: \  
static const AFX_MSGMAP_ENTRY _messageEntries[]; \  
protected: \  
static AFX_DATA const AFX_MSGMAP messageMap; \  
virtual const AFX_MSGMAP* GetMessageMap() const; \  
#endif  
//^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
// 以上这段代码实际上是嵌入在你的类中.  

在 .CPP 文件中我们看到还有这段宏,  
BEGIN_MESSAGE_MAP(CMsgWnd, CWnd)  
//{{AFX_MSG_MAP(CMsgWnd)  
ON_WM_PAINT()  
ON_WM_LBUTTONDOWN()  
//}}AFX_MSG_MAP  
END_MESSAGE_MAP()  
现在我们再来看看它是被如何定义的.  

#ifdef _AFXDLL  
#define BEGIN_MESSAGE_MAP(theClass, baseClass) \  
const AFX_MSGMAP* PASCAL theClass::_GetBaseMessageMap() \  
{ return &baseClass::messageMap; } \ // 返回基类的消息映射表  
^^^^^^^^^^^^^^^^^^^^^^  
const AFX_MSGMAP* theClass::GetMessageMap() const \  
{ return &theClass::messageMap; } \ // 返回当前类的消息映射表  

AFX_DATADEF const AFX_MSGMAP theClass::messageMap = \ // 消息映射表  
{ &theClass::_GetBaseMessageMap, &theClass::_messageEntries[0] }; \  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
const AFX_MSGMAP_ENTRY theClass::_messageEntries[] = \  
{ \  

#else  
#define BEGIN_MESSAGE_MAP(theClass, baseClass) \  
const AFX_MSGMAP* theClass::GetMessageMap() const \  
{ return &theClass::messageMap; } \ // 返回当前类的消息映射表  

AFX_DATADEF const AFX_MSGMAP theClass::messageMap = \ // 消息映射表  
{ &baseClass::messageMap, &theClass::_messageEntries[0] }; \  
^^^^^^^^^^^^^^^^^^^^^^ 可以看到这是 MFC 动态连接与静态连接的区别所在  

动态连接时使用函数 _GetBaseMessageMap 返回 &baseClass::messageMap  
而静态连接是直接使用.至于Microsoft为什么要这样做,  
好像没有什么很好的理由.  
当然这个并不重要,我们暂且不用理会.  

const AFX_MSGMAP_ENTRY theClass::_messageEntries[] = \  
//开始初始化消息入口  
{ \  
#endif  

#define END_MESSAGE_MAP() \  
{0, 0, 0, 0, AfxSig_end, (AFX_PMSG)0 } \  
}; \ ^^^^^^^^^^^^^^^^^^^^^^ 指示消息映射结束.  

在这两个宏之间还有  
ON_WM_PAINT()  
ON_WM_LBUTTONDOWN()  
它们是我们在ClassWizard中选择了WM_PAINT和WM_LBUTTONDOWN消息后,  
MFC自动加入的,那么这两个又是如何定义的呢?  

......  

#define ON_WM_PAINT() \  
{ WM_PAINT, 0, 0, 0, AfxSig_vv, \  
^^^^^^^^ ^ ^ ^ ^  
| | | | +------------- 消息处理类型  
| | | +------------------- LastID  
| | +---------------------- ID=0, 窗口消息  
| +------------------------- 控制码  
+-------------------------------- WM_PAINT 消息  
(AFX_PMSG)(AFX_PMSGW)(void (AFX_MSG_CALL CWnd::*)  
(void))&OnPaint },  
^^^^^^^^^^^^^^^ 消息处理函数  
.....  

#define ON_WM_LBUTTONDOWN() \  
{ WM_LBUTTONDOWN, 0, 0, 0, AfxSig_vwp, \  
^^^^^^^^^^^^^^ WM_LBUTTON 消息  
(AFX_PMSG)(AFX_PMSGW)(void (AFX_MSG_CALL CWnd::*)  
(UINT, CPoint))&OnLButtonDown },  
^^^^^^^^^^^^^ 消息处理函数  
......  

现在我们差不多可以看得出来了,消息处理函数可以  
靠_messageEntries来找到每个消息的处理函数.  
我们可以再看看CWnd类来验证我们的想法.  
先看一下窗口的建立过程:  
BOOL CWnd::Create(LPCTSTR lpszClassName,  
LPCTSTR lpszWindowName, DWORD dwStyle,  
const RECT& rect,  
CWnd* pParentWnd, UINT nID,  
CCreateContext* pContext)  
{  
// can't use for desktop or pop-up windows (use CreateEx instead)  
ASSERT(pParentWnd != NULL);  
ASSERT((dwStyle & WS_POPUP) == 0);  

return CreateEx(0, lpszClassName, lpszWindowName,  
dwStyle | WS_CHILD,  
rect.left, rect.top,  
rect.right - rect.left, rect.bottom - rect.top,  
pParentWnd->GetSafeHwnd(), (HMENU)nID, (LPVOID)pContext);  
}  

BOOL CWnd::CreateEx(DWORD dwExStyle, LPCTSTR lpszClassName,  
LPCTSTR lpszWindowName, DWORD dwStyle,  
int x, int y, int nWidth, int nHeight,  
HWND hWndParent, HMENU nIDorHMenu, LPVOID lpParam)  
{  
// allow modification of several common create parameters  
CREATESTRUCT cs;  
cs.dwExStyle = dwExStyle;  
cs.lpszClass = lpszClassName;  
cs.lpszName = lpszWindowName;  
cs.style = dwStyle;  
cs.x = x;  
cs.y = y;  
cs.cx = nWidth;  
cs.cy = nHeight;  
cs.hwndParent = hWndParent;  
cs.hMenu = nIDorHMenu;  
cs.hInstance = AfxGetInstanceHandle();  
cs.lpCreateParams = lpParam;  

if (!PreCreateWindow(cs))  
{  
PostNcDestroy();  
return FALSE;  
}  

AfxHookWindowCreate(this);  
^^^^^^^^^^^^^^^^^^^^^^^^^^ 函数见后  
HWND hWnd = ::CreateWindowEx(cs.dwExStyle, cs.lpszClass,  
cs.lpszName, cs.style, cs.x, cs.y, cs.cx, cs.cy,  
cs.hwndParent, cs.hMenu, cs.hInstance, cs.lpCreateParams);  

if (!AfxUnhookWindowCreate())  
PostNcDestroy(); // cleanup if CreateWindowEx fails too soon  

if (hWnd == NULL)  
return FALSE;  
ASSERT(hWnd == m_hWnd); // should have been set in send msg hook  
return TRUE;  
}  

// for child windows  
BOOL CWnd::PreCreateWindow(CREATESTRUCT& cs)  
{  
if (cs.lpszClass == NULL)  
{  
// make sure the default window class is registered  
if (!AfxDeferRegisterClass(AFX_WND_REG))  
return FALSE;  

// no WNDCLASS provided - use child window default  
ASSERT(cs.style & WS_CHILD);  
cs.lpszClass = _afxWnd;  
}  
return TRUE;  
}  

void AFXAPI AfxHookWindowCreate(CWnd* pWnd)  
{  
_AFX_THREAD_STATE* pThreadState = _afxThreadState.GetData();  
if (pThreadState->m_pWndInit == pWnd)  
return;  

if (pThreadState->m_hHookOldCbtFilter == NULL)  
{  
pThreadState->m_hHookOldCbtFilter = ::SetWindowsHookEx(WH_CBT,  
^^^^^^  
Computer-based Training,当建立、删除、移动、最大化、最小化  
窗口时，将会调用钩子函数。  
_AfxCbtFilterHook, NULL, ::GetCurrentThreadId());  
^^^^^^^^^^^^^^^^^ 钩子函数  
if (pThreadState->m_hHookOldCbtFilter == NULL)  
AfxThrowMemoryException();  
}  
ASSERT(pThreadState->m_hHookOldCbtFilter != NULL);  
ASSERT(pWnd != NULL);  
ASSERT(pWnd->m_hWnd == NULL); // only do once  

ASSERT(pThreadState->m_pWndInit == NULL); // hook not already in progr
ess  
pThreadState->m_pWndInit = pWnd;  
}  

BOOL AFXAPI AfxUnhookWindowCreate()  
{  
_AFX_THREAD_STATE* pThreadState = _afxThreadState.GetData();  
#ifndef _AFXDLL  
if (afxContextIsDLL && pThreadState->m_hHookOldCbtFilter != NULL)  
{  
::UnhookWindowsHookEx(pThreadState->m_hHookOldCbtFilter);  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
删除钩子函数  
pThreadState->m_hHookOldCbtFilter = NULL;  
}  
#endif  
if (pThreadState->m_pWndInit != NULL)  
{  
pThreadState->m_pWndInit = NULL;  
return FALSE; // was not successfully hooked  
}  
return TRUE;  
}  

// Window creation hooks  
LRESULT CALLBACK  
_AfxCbtFilterHook(int code, WPARAM wParam, LPARAM lParam)  
{  
_AFX_THREAD_STATE* pThreadState = _afxThreadState.GetData();  
if (code != HCBT_CREATEWND)  
{ ^^^^^^^^^^^^^^^^^^^^^^ 是要建立窗口吗？  
// wait for HCBT_CREATEWND just pass others on...  
return CallNextHookEx(pThreadState->m_hHookOldCbtFilter, code,  
wParam, lParam);  
}  

ASSERT(lParam != NULL);  
LPCREATESTRUCT lpcs = ((LPCBT_CREATEWND)lParam)->lpcs;  
ASSERT(lpcs != NULL);  

// this hook exists to set the SendMessage hook on window creations  
// (but this is only done for MFC windows or non-child windows)  
// the subclassing cannot be done at this point because on Win32s  
// the window does not have the WNDPROC set yet  
CWnd* pWndInit = pThreadState->m_pWndInit;  
if (pWndInit != NULL || (!(lpcs->style & WS_CHILD) && !afxContextIsDLL
))  
{  
ASSERT(wParam != NULL); // should be non-NULL HWND  
HWND hWnd = (HWND)wParam;  
WNDPROC oldWndProc;  
if (pWndInit != NULL)  
{ ^^^^^^^^^^^^^^^^ 窗口建立来自一个CWnd?  
// the window should not be in the permanent map at this time  
ASSERT(CWnd::FromHandlePermanent(hWnd) == NULL);  

// connect the HWND to pWndInit...  
pWndInit->Attach(hWnd);  
// allow other subclassing to occur first  
pWndInit->PreSubclassWindow();  

WNDPROC *pOldWndProc = pWndInit->GetSuperWndProcAddr();  
ASSERT(pOldWndProc != NULL);  

#ifndef _MAC  
_AFX_CTL3D_STATE* pCtl3dState;  
DWORD dwFlags;  
if (!afxData.bWin4 && !afxContextIsDLL &&  
(pCtl3dState = _afxCtl3dState.GetDataNA()) != NULL &&  
pCtl3dState->m_pfnSubclassDlgEx != NULL &&  
(dwFlags = AfxCallWndProc(pWndInit, hWnd, WM_QUERY3DCONTROLS)) != 0)  

{  
// was the class registered with AfxWndProc?  
WNDPROC afxWndProc = AfxGetAfxWndProc();  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
函数见后  
BOOL bAfxWndProc = ((WNDPROC)  
GetWindowLong(hWnd, GWL_WNDPROC) == afxWndProc);  

pCtl3dState->m_pfnSubclassDlgEx(hWnd, dwFlags);  

// subclass the window if not already wired to AfxWndProc  
if (!bAfxWndProc)  
{  
// subclass the window with standard AfxWndProc  
oldWndProc = (WNDPROC)SetWindowLong(hWnd, GWL_WNDPROC,  
(DWORD)afxWndProc); // 修改窗口的WndProc!!!  

ASSERT(oldWndProc != NULL);  
*pOldWndProc = oldWndProc;  
}  
}  
else  
#endif  
{  
// subclass the window with standard AfxWndProc  
WNDPROC afxWndProc = AfxGetAfxWndProc();  
oldWndProc = (WNDPROC)SetWindowLong(hWnd, GWL_WNDPROC,  
(DWORD)afxWndProc);  
ASSERT(oldWndProc != NULL);  
if (oldWndProc != afxWndProc)  
*pOldWndProc = oldWndProc;  
}  
pThreadState->m_pWndInit = NULL;  
}  
else  
{  
ASSERT(!afxContextIsDLL); // should never get here  

// subclass the window with the proc which does gray backgrounds  
oldWndProc = (WNDPROC)GetWindowLong(hWnd, GWL_WNDPROC);  
if (oldWndProc != NULL)  
{  
ASSERT(GetProp(hWnd, szAfxOldWndProc) == NULL);  
SetProp(hWnd, szAfxOldWndProc, oldWndProc);  
if ((WNDPROC)GetProp(hWnd, szAfxOldWndProc) == oldWndProc)  
{  
SetWindowLong(hWnd, GWL_WNDPROC,  
(DWORD)(pThreadState->m_bDlgCreate ?  
_AfxGrayBackgroundWndProc : _AfxActivationWndProc));  
ASSERT(oldWndProc != NULL);  
}  
}  
}  
}  

LRESULT lResult = CallNextHookEx(pThreadState->m_hHookOldCbtFilter, co
de,  
wParam, lParam);  

#ifndef _AFXDLL  
if (afxContextIsDLL)  
{  
::UnhookWindowsHookEx(pThreadState->m_hHookOldCbtFilter);  
pThreadState->m_hHookOldCbtFilter = NULL;  
}  
#endif  
return lResult;  
}  

// always indirectly accessed via AfxGetAfxWndProc  
WNDPROC AFXAPI AfxGetAfxWndProc()  
{  
#ifdef _AFXDLL  
return AfxGetModuleState()->m_pfnAfxWndProc;  
// 如果MFC是动态连入，返回的地址是AfxWndProcBase  
#else  
return &AfxWndProc; // 否则返回AfxWndProc  
#endif  
}  

// The WndProc for all CWnd's and derived classes  

LRESULT CALLBACK  
AfxWndProc(HWND hWnd, UINT nMsg, WPARAM wParam, LPARAM lParam)  
{  
// special message which identifies the window as using AfxWndProc  
if (nMsg == WM_QUERYAFXWNDPROC)  
^^^^^^^^^^^^^^^^^^ 查询是否为AFX的窗口过程  
return 1;  

// all other messages route through message map  
CWnd* pWnd = CWnd::FromHandlePermanent(hWnd);  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
FromHandlePermanent函数用于返回一个和hWnd句柄对应的  
永久的CWnd类，所谓永久的，就是CWnd调用了Attach函数  
连上了一个窗口句柄。  

ASSERT(pWnd != NULL);  
ASSERT(pWnd->m_hWnd == hWnd);  
return AfxCallWndProc(pWnd, hWnd, nMsg, wParam, lParam);  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ 函数见后  
}  

#ifdef _AFXDLL  
#undef AfxWndProc  
LRESULT CALLBACK  
AfxWndProcBase(HWND hWnd, UINT nMsg, WPARAM wParam, LPARAM lParam)  
{  
AFX_MANAGE_STATE(_afxBaseModuleState.GetData());  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
// 由于MFC是动态连入的，所以必须要这条语句！  
return AfxWndProc(hWnd, nMsg, wParam, lParam);  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
调用MFC静态连入时使用的函数  
}  
#endif  

// 微软官方规定的发送消息给CWnd的函数  
LRESULT AFXAPI AfxCallWndProc(CWnd* pWnd, HWND hWnd, UINT nMsg,  
WPARAM wParam = 0, LPARAM lParam = 0)  
{  
_AFX_THREAD_STATE* pThreadState = _afxThreadState.GetData();  
MSG oldState = pThreadState->m_lastSentMsg; // save for nesting  
pThreadState->m_lastSentMsg.hwnd = hWnd;  
pThreadState->m_lastSentMsg.message = nMsg;  
pThreadState->m_lastSentMsg.wParam = wParam;  
pThreadState->m_lastSentMsg.lParam = lParam;  

#ifdef _DEBUG  
if (afxTraceFlags & traceWinMsg)  
_AfxTraceMsg(_T("WndProc"), &pThreadState->m_lastSentMsg);  
#endif  

// Catch exceptions thrown outside the scope of a callback  
// in debug builds and warn the user.  
LRESULT lResult;  
TRY  
{  
#ifndef _AFX_NO_OCC_SUPPORT  
// special case for WM_DESTROY  
if ((nMsg == WM_DESTROY) && (pWnd->m_pCtrlCont != NULL))  
pWnd->m_pCtrlCont->OnUIActivate(NULL);  
#endif  

// special case for WM_INITDIALOG  
CRect rectOld;  
DWORD dwStyle;  
if (nMsg == WM_INITDIALOG)  
_AfxPreInitDialog(pWnd, &rectOld, &dwStyle);  

// delegate to object's WindowProc  
lResult = pWnd->WindowProc(nMsg, wParam, lParam);  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
调用CWnd类的WindowProc函数,函数见后  

// more special case for WM_INITDIALOG  
if (nMsg == WM_INITDIALOG)  
_AfxPostInitDialog(pWnd, rectOld, dwStyle);  
}  
CATCH_ALL(e)  
{  
lResult = AfxGetThread()->ProcessWndProcException(e, &pThreadState->m_
lastSentMsg);  
TRACE1("Warning: Uncaught exception in WindowProc (returning %ld).\n",

lResult);  
DELETE_EXCEPTION(e);  
}  
END_CATCH_ALL  

pThreadState->m_lastSentMsg = oldState;  
return lResult;  
}  
消息处理转入CWnd类了,  

LRESULT CWnd::WindowProc(UINT message, WPARAM wParam, LPARAM lParam)  

{  
// OnWndMsg does most of the work, except for DefWindowProc call  
LRESULT lResult = 0;  
if (!OnWndMsg(message, wParam, lParam, &lResult))  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
处理窗口消息  

lResult = DefWindowProc(message, wParam, lParam);  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
如果没有处理，就调用DefWindowProc，  
呵呵，和我们以前用C写Windows程序一样！！  
return lResult;  
}  

BOOL CWnd::OnWndMsg(UINT message, WPARAM wParam, LPARAM lParam, LRESUL
T* pResult)  
{  
LRESULT lResult = 0;  

// special case for commands  
if (message == WM_COMMAND)  
{ ^^^^^^^^^^ WM_COMMAND消息？  
if (OnCommand(wParam, lParam))  
{ ^^^^^^^^^^^^^^^^^^^^^^^^^  
调用OnCommand函数，这是一个虚函数  
lResult = 1;  
goto LReturnTrue;  
}  
return FALSE;  
}  

// special case for notifies  
if (message == WM_NOTIFY)  
{ ^^^^^^^^^ 通告消息？  
NMHDR* pNMHDR = (NMHDR*)lParam;  
if (pNMHDR->hwndFrom != NULL &&  
OnNotify(wParam, lParam, &lResult))  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
调用OnNotify函数，也是一个虚函数  
goto LReturnTrue;  
return FALSE;  
}  

// special case for activation  
if (message == WM_ACTIVATE)  
_AfxHandleActivate(this, wParam, CWnd::FromHandle((HWND)lParam));  

// special case for set cursor HTERROR  
if (message == WM_SETCURSOR &&  
_AfxHandleSetCursor(this, (short)LOWORD(lParam), HIWORD(lParam)))  
{  
lResult = 1;  
goto LReturnTrue;  
}  

const AFX_MSGMAP* pMessageMap; pMessageMap = GetMessageMap();  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
GetMessageMap返回当前类的消息映射表  

UINT iHash; iHash = (LOWORD((DWORD)pMessageMap) ^ message) & (iHashMax
-1);  
AfxLockGlobals(CRIT_WINMSGCACHE);  
AFX_MSG_CACHE* pMsgCache; pMsgCache = &_afxMsgCache[iHash];  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
为了查找消息更快，微软实现了一个消息cache！  
const AFX_MSGMAP_ENTRY* lpEntry;  
if (message == pMsgCache->nMsg && pMessageMap == pMsgCache->pMessageMa
p)  
{  
// 消息在cache里，命中  
lpEntry = pMsgCache->lpEntry;  
AfxUnlockGlobals(CRIT_WINMSGCACHE);  
if (lpEntry == NULL)  
return FALSE;  

// cache hit, and it needs to be handled  
if (message < 0xC000) 
goto LDispatch; 
else 
goto LDispatchRegistered; 
} 
else 
{ 
// 不在cache里面，只好查找整个消息映射表 
pMsgCache->nMsg = message;  
pMsgCache->pMessageMap = pMessageMap;  

#ifdef _AFXDLL  
for (/* pMessageMap already init'ed */; pMessageMap != NULL;  
pMessageMap = (*pMessageMap->pfnGetBaseMap)())  
#else  
for (/* pMessageMap already init'ed */; pMessageMap != NULL;  
pMessageMap = pMessageMap->pBaseMap)  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
从最高层类一层一层往下找  
#endif  
{  
// Note: catch not so common but fatal mistake!!  
// BEGIN_MESSAGE_MAP(CMyWnd, CMyWnd)  
#ifdef _AFXDLL  
ASSERT(pMessageMap != (*pMessageMap->pfnGetBaseMap)());  
#else  
ASSERT(pMessageMap != pMessageMap->pBaseMap);  
#endif  

if (message < 0xC000) 
{ 
// constant window message 
if ((lpEntry = AfxFindMessageEntry(pMessageMap->lpEntries,  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
这是一个汇编写的查找消息处理入口的函数  
message, 0, 0)) != NULL)  
{  
pMsgCache->lpEntry = lpEntry;  
AfxUnlockGlobals(CRIT_WINMSGCACHE);  
goto LDispatch;  
}  
}  
else  
{  
// registered windows message  
lpEntry = pMessageMap->lpEntries;  
while ((lpEntry = AfxFindMessageEntry(lpEntry, 0xC000, 0, 0)) != NULL)

{  
UINT* pnID = (UINT*)(lpEntry->nSig);  
ASSERT(*pnID >= 0xC000 || *pnID == 0);  
// must be successfully registered  
if (*pnID == message)  
{  
pMsgCache->lpEntry = lpEntry;  
AfxUnlockGlobals(CRIT_WINMSGCACHE);  
goto LDispatchRegistered;  
}  
lpEntry++; // keep looking past this one  
}  
}  
}  

pMsgCache->lpEntry = NULL;  
AfxUnlockGlobals(CRIT_WINMSGCACHE);  
return FALSE;  
}  
ASSERT(FALSE); // not reached  

LDispatch: //消息处理函数找到了就转到此处了  
ASSERT(message < 0xC000); 
union MessageMapFunctions mmf; 
mmf.pfn = lpEntry->pfn;  

// if we've got WM_SETTINGCHANGE / WM_WININICHANGE, we need to  
// decide if we're going to call OnWinIniChange() or OnSettingChange()

int nSig;  
nSig = lpEntry->nSig;  
^^^^^^^^^^^^^^^^^^^^  
if (lpEntry->nID == WM_SETTINGCHANGE)  
{  
DWORD dwVersion = GetVersion();  
if (LOBYTE(LOWORD(dwVersion)) >= 4)  
nSig = AfxSig_vws;  
else  
nSig = AfxSig_vs;  
}  

switch (nSig)  
{// nSig为消息参数类型,详细解释见后  
default:  
ASSERT(FALSE);  
break;  

case AfxSig_bD:  
lResult = (this->*mmf.pfn_bD)(CDC::FromHandle((HDC)wParam));  
^^^ mmf就是我们在Message Map里面设的消息处理函数  
它是一个共用体（union），其中定义了  
带有各种不同参数形式的函数。  
break;  

case AfxSig_bb: // AfxSig_bb, AfxSig_bw, AfxSig_bh  
lResult = (this->*mmf.pfn_bb)((BOOL)wParam);  
break;  

case AfxSig_bWww: // really AfxSig_bWiw  
lResult = (this->*mmf.pfn_bWww)(CWnd::FromHandle((HWND)wParam),  
(short)LOWORD(lParam), HIWORD(lParam));  
break;  

case AfxSig_bWCDS:  
lResult = (this->*mmf.pfn_bWCDS)(CWnd::FromHandle((HWND)lParam),  
(COPYDATASTRUCT*) lParam);  
break;  

case AfxSig_bHELPINFO:  
lResult = (this->*mmf.pfn_bHELPINFO)((HELPINFO*)lParam);  
break;  

case AfxSig_hDWw:  
{  
// special case for OnCtlColor to avoid too many temporary objects  
ASSERT(message == WM_CTLCOLOR);  
AFX_CTLCOLOR* pCtl = (AFX_CTLCOLOR*)lParam;  
CDC dcTemp; dcTemp.m_hDC = pCtl->hDC;  
CWnd wndTemp; wndTemp.m_hWnd = pCtl->hWnd;  
UINT nCtlType = pCtl->nCtlType;  
// if not coming from a permanent window, use stack temporary  
CWnd* pWnd = CWnd::FromHandlePermanent(wndTemp.m_hWnd);  
if (pWnd == NULL)  
{  
#ifndef _AFX_NO_OCC_SUPPORT  
// determine the site of the OLE control if it is one  
COleControlSite* pSite;  
if (m_pCtrlCont != NULL && (pSite = (COleControlSite*)  
m_pCtrlCont->m_siteMap.GetValueAt(wndTemp.m_hWnd)) != NULL)  
{  
wndTemp.m_pCtrlSite = pSite;  
}  
#endif  
pWnd = &wndTemp;  
}  
HBRUSH hbr = (this->*mmf.pfn_hDWw)(&dcTemp, pWnd, nCtlType);  
// fast detach of temporary objects  
dcTemp.m_hDC = NULL;  
wndTemp.m_hWnd = NULL;  
lResult = (LRESULT)hbr;  
}  
break;  

case AfxSig_hDw:  
{  
// special case for CtlColor to avoid too many temporary objects  
ASSERT(message == WM_REFLECT_BASE+WM_CTLCOLOR);  
AFX_CTLCOLOR* pCtl = (AFX_CTLCOLOR*)lParam;  
CDC dcTemp; dcTemp.m_hDC = pCtl->hDC;  
UINT nCtlType = pCtl->nCtlType;  
HBRUSH hbr = (this->*mmf.pfn_hDw)(&dcTemp, nCtlType);  
// fast detach of temporary objects  
dcTemp.m_hDC = NULL;  
lResult = (LRESULT)hbr;  
}  
break;  

case AfxSig_iwWw:  
lResult = (this->*mmf.pfn_iwWw)(LOWORD(wParam),  
CWnd::FromHandle((HWND)lParam), HIWORD(wParam));  
break;  

case AfxSig_iww:  
lResult = (this->*mmf.pfn_iww)(LOWORD(wParam), HIWORD(wParam));  
break;  

case AfxSig_iWww: // really AfxSig_iWiw  
lResult = (this->*mmf.pfn_iWww)(CWnd::FromHandle((HWND)wParam),  
(short)LOWORD(lParam), HIWORD(lParam));  
break;  

case AfxSig_is:  
lResult = (this->*mmf.pfn_is)((LPTSTR)lParam);  
break;  

case AfxSig_lwl:  
lResult = (this->*mmf.pfn_lwl)(wParam, lParam);  
break;  

case AfxSig_lwwM:  
lResult = (this->*mmf.pfn_lwwM)((UINT)LOWORD(wParam),  
(UINT)HIWORD(wParam), (CMenu*)CMenu::FromHandle((HMENU)lParam));  
break;  

case AfxSig_vv:  
(this->*mmf.pfn_vv)();  
break;  

case AfxSig_vw: // AfxSig_vb, AfxSig_vh  
(this->*mmf.pfn_vw)(wParam);  
break;  

case AfxSig_vww:  
(this->*mmf.pfn_vww)((UINT)wParam, (UINT)lParam);  
break;  

case AfxSig_vvii:  
(this->*mmf.pfn_vvii)((short)LOWORD(lParam), (short)HIWORD(lParam));  

break;  

case AfxSig_vwww:  
(this->*mmf.pfn_vwww)(wParam, LOWORD(lParam), HIWORD(lParam));  
break;  

case AfxSig_vwii:  
(this->*mmf.pfn_vwii)(wParam, LOWORD(lParam), HIWORD(lParam));  
break;  

case AfxSig_vwl:  
(this->*mmf.pfn_vwl)(wParam, lParam);  
break;  

case AfxSig_vbWW:  
(this->*mmf.pfn_vbWW)(m_hWnd == (HWND)lParam,  
CWnd::FromHandle((HWND)lParam),  
CWnd::FromHandle((HWND)wParam));  
break;  

case AfxSig_vD:  
(this->*mmf.pfn_vD)(CDC::FromHandle((HDC)wParam));  
break;  

case AfxSig_vM:  
(this->*mmf.pfn_vM)(CMenu::FromHandle((HMENU)wParam));  
break;  

case AfxSig_vMwb:  
(this->*mmf.pfn_vMwb)(CMenu::FromHandle((HMENU)wParam),  
LOWORD(lParam), (BOOL)HIWORD(lParam));  
break;  

case AfxSig_vW:  
(this->*mmf.pfn_vW)(CWnd::FromHandle((HWND)wParam));  
break;  

case AfxSig_vW2:  
(this->*mmf.pfn_vW)(CWnd::FromHandle((HWND)lParam));  
break;  

case AfxSig_vWww:  
(this->*mmf.pfn_vWww)(CWnd::FromHandle((HWND)wParam), LOWORD(lParam), 

HIWORD(lParam));  
break;  

case AfxSig_vWp:  
{  
CPoint point((DWORD)lParam);  
(this->*mmf.pfn_vWp)(CWnd::FromHandle((HWND)wParam), point);  
}  
break;  

case AfxSig_vWh:  
(this->*mmf.pfn_vWh)(CWnd::FromHandle((HWND)wParam),  
(HANDLE)lParam);  
break;  

case AfxSig_vwW:  
(this->*mmf.pfn_vwW)(wParam, CWnd::FromHandle((HWND)lParam));  
break;  

case AfxSig_vwWb:  
(this->*mmf.pfn_vwWb)((UINT)(LOWORD(wParam)),  
CWnd::FromHandle((HWND)lParam), (BOOL)HIWORD(wParam));  
break;  

case AfxSig_vwwW:  
case AfxSig_vwwx:  
{  
// special case for WM_VSCROLL and WM_HSCROLL  
ASSERT(message == WM_VSCROLL || message == WM_HSCROLL ||  
message == WM_VSCROLL+WM_REFLECT_BASE || message == WM_HSCROLL+WM_REFL
ECT_BASE);  
int nScrollCode = (short)LOWORD(wParam);  
int nPos = (short)HIWORD(wParam);  
if (lpEntry->nSig == AfxSig_vwwW)  
(this->*mmf.pfn_vwwW)(nScrollCode, nPos,  
CWnd::FromHandle((HWND)lParam));  
else  
(this->*mmf.pfn_vwwx)(nScrollCode, nPos);  
}  
break;  

case AfxSig_vs:  
(this->*mmf.pfn_vs)((LPTSTR)lParam);  
break;  

case AfxSig_vws:  
(this->*mmf.pfn_vws)((UINT) wParam, (LPCTSTR)lParam);  
break;  

case AfxSig_vOWNER:  
(this->*mmf.pfn_vOWNER)((int)wParam, (LPTSTR)lParam);  
lResult = TRUE;  
break;  

case AfxSig_iis:  
lResult = (this->*mmf.pfn_iis)((int)wParam, (LPTSTR)lParam);  
break;  

case AfxSig_wp:  
{  
CPoint point((DWORD)lParam);  
lResult = (this->*mmf.pfn_wp)(point);  
}  
break;  

case AfxSig_wv: // AfxSig_bv, AfxSig_wv  
lResult = (this->*mmf.pfn_wv)();  
break;  

case AfxSig_vCALC:  
(this->*mmf.pfn_vCALC)((BOOL)wParam, (NCCALCSIZE_PARAMS*)lParam);  
break;  

case AfxSig_vPOS:  
(this->*mmf.pfn_vPOS)((WINDOWPOS*)lParam);  
break;  

case AfxSig_vwwh:  
(this->*mmf.pfn_vwwh)(LOWORD(wParam), HIWORD(wParam), (HANDLE)lParam);

break;  

case AfxSig_vwp:  
{  
CPoint point((DWORD)lParam);  
(this->*mmf.pfn_vwp)(wParam, point);  
break;  
}  
case AfxSig_vwSIZING:  
(this->*mmf.pfn_vwl)(wParam, lParam);  
lResult = TRUE;  
break;  

case AfxSig_bwsp:  
lResult = (this->*mmf.pfn_bwsp)(LOWORD(wParam), (short) HIWORD(wParam)
,  
CPoint(LOWORD(lParam), HIWORD(lParam)));  
if (!lResult)  
return FALSE;  
}  
goto LReturnTrue;  

LDispatchRegistered: // for registered windows messages  
ASSERT(message >= 0xC000);  
mmf.pfn = lpEntry->pfn;  
lResult = (this->*mmf.pfn_lwl)(wParam, lParam);  

LReturnTrue:  
if (pResult != NULL)  
*pResult = lResult;  
return TRUE;  
}  

LRESULT CWnd::DefWindowProc(UINT nMsg, WPARAM wParam, LPARAM lParam)  

{  
if (m_pfnSuper != NULL)  
return ::CallWindowProc(m_pfnSuper, m_hWnd, nMsg, wParam, lParam);  

WNDPROC pfnWndProc;  
if ((pfnWndProc = *GetSuperWndProcAddr()) == NULL)  
return ::DefWindowProc(m_hWnd, nMsg, wParam, lParam);  
else  
return ::CallWindowProc(pfnWndProc, m_hWnd, nMsg, wParam, lParam);  
}  

enum AfxSig // 消息处理函数的参数类型  
{  
AfxSig_end = 0, // [marks end of message map]  

AfxSig_bD, // BOOL (CDC*)  
^^  
|+--------------- D表示CDC*  
+---------------- b表示返回值为bool型  
AfxSig_bb, // BOOL (BOOL)  
AfxSig_bWww, // BOOL (CWnd*, UINT, UINT)  
AfxSig_hDWw, // HBRUSH (CDC*, CWnd*, UINT)  
AfxSig_hDw, // HBRUSH (CDC*, UINT)  
AfxSig_iwWw, // int (UINT, CWnd*, UINT)  
AfxSig_iww, // int (UINT, UINT)  
AfxSig_iWww, // int (CWnd*, UINT, UINT)  
AfxSig_is, // int (LPTSTR)  
AfxSig_lwl, // LRESULT (WPARAM, LPARAM)  
AfxSig_lwwM, // LRESULT (UINT, UINT, CMenu*)  
AfxSig_vv, // void (void)  
^^^^^^^^^ 举个例子，如WM_PAINT消息，  
在BEGIN_MESSAGE_MAP里面我们定义的消息处理函数为  
ON_WM_PAINT()  
察看我们前面剖析过程，知道  
#define ON_WM_PAINT() \  
{ WM_PAINT, 0, 0, 0, AfxSig_vv, \  

所以处理函数的形式即为  
OnPaint()  
即没有参数，没有返回值。  
采用这种方法，就可以避免把没用的wParam和lParam参数传给用户  

AfxSig_vw, // void (UINT)  
AfxSig_vww, // void (UINT, UINT)  
AfxSig_vvii, // void (int, int) // wParam is ignored  
AfxSig_vwww, // void (UINT, UINT, UINT)  
AfxSig_vwii, // void (UINT, int, int)  
AfxSig_vwl, // void (UINT, LPARAM)  
AfxSig_vbWW, // void (BOOL, CWnd*, CWnd*)  
AfxSig_vD, // void (CDC*)  

AfxSig_vM, // void (CMenu*)  
AfxSig_vMwb, // void (CMenu*, UINT, BOOL)  

AfxSig_vW, // void (CWnd*)  
AfxSig_vWww, // void (CWnd*, UINT, UINT)  
AfxSig_vWp, // void (CWnd*, CPoint)  
AfxSig_vWh, // void (CWnd*, HANDLE)  
AfxSig_vwW, // void (UINT, CWnd*)  
AfxSig_vwWb, // void (UINT, CWnd*, BOOL)  
AfxSig_vwwW, // void (UINT, UINT, CWnd*)  
AfxSig_vwwx, // void (UINT, UINT)  
AfxSig_vs, // void (LPTSTR)  
AfxSig_vOWNER, // void (int, LPTSTR), force return TRUE  
AfxSig_iis, // int (int, LPTSTR)  
AfxSig_wp, // UINT (CPoint)  
AfxSig_wv, // UINT (void)  
AfxSig_vPOS, // void (WINDOWPOS*)  
AfxSig_vCALC, // void (BOOL, NCCALCSIZE_PARAMS*)  
AfxSig_vNMHDRpl, // void (NMHDR*, LRESULT*)  
AfxSig_bNMHDRpl, // BOOL (NMHDR*, LRESULT*)  
AfxSig_vwNMHDRpl, // void (UINT, NMHDR*, LRESULT*)  
AfxSig_bwNMHDRpl, // BOOL (UINT, NMHDR*, LRESULT*)  
AfxSig_bHELPINFO, // BOOL (HELPINFO*)  
AfxSig_vwSIZING, // void (UINT, LPRECT) -- return TRUE  

// signatures specific to CCmdTarget  
AfxSig_cmdui, // void (CCmdUI*)  
AfxSig_cmduiw, // void (CCmdUI*, UINT)  
AfxSig_vpv, // void (void*)  
AfxSig_bpv, // BOOL (void*)  

// Other aliases (based on implementation)  
AfxSig_vwwh, // void (UINT, UINT, HANDLE)  
AfxSig_vwp, // void (UINT, CPoint)  
AfxSig_bw = AfxSig_bb, // BOOL (UINT)  
AfxSig_bh = AfxSig_bb, // BOOL (HANDLE)  
AfxSig_iw = AfxSig_bb, // int (UINT)  
AfxSig_ww = AfxSig_bb, // UINT (UINT)  
AfxSig_bv = AfxSig_wv, // BOOL (void)  
AfxSig_hv = AfxSig_wv, // HANDLE (void)  
AfxSig_vb = AfxSig_vw, // void (BOOL)  
AfxSig_vbh = AfxSig_vww, // void (BOOL, HANDLE)  
AfxSig_vbw = AfxSig_vww, // void (BOOL, UINT)  
AfxSig_vhh = AfxSig_vww, // void (HANDLE, HANDLE)  
AfxSig_vh = AfxSig_vw, // void (HANDLE)  
AfxSig_viSS = AfxSig_vwl, // void (int, STYLESTRUCT*)  
AfxSig_bwl = AfxSig_lwl,  
AfxSig_vwMOVING = AfxSig_vwSIZING, // void (UINT, LPRECT) -- return TR
UE  

AfxSig_vW2, // void (CWnd*) (CWnd* comes from lParam)  
AfxSig_bWCDS, // BOOL (CWnd*, COPYDATASTRUCT*)  
AfxSig_bwsp, // BOOL (UINT, short, CPoint)  
AfxSig_vws,  
};  
```

