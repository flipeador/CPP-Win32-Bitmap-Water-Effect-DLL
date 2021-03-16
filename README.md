# CPP-Win32-Bitmap-Water-Effect-DLL

#### AutoHotkey v2 Example (https://www.autohotkey.com/)

> ⚠ A 32bpp device-independent bitmap (DIB) is required.


```autohotkey
; https://www.autohotkey.com/
#Requires AutoHotkey v2.0-a129-78d2aa15
#DllLoad waterfx.dll

global wfx := WaterFx()

global WM_MOUSEMOVE   := 0x0200
global WM_LBUTTONDOWN := 0x0201
global WM_LBUTTONUP   := 0x0202
global WM_RBUTTONDOWN := 0x0204
global WM_RBUTTONUP   := 0x0205

OnMessage(WM_MOUSEMOVE  , WindowProc)
OnMessage(WM_LBUTTONDOWN, WindowProc)
OnMessage(WM_LBUTTONUP  , WindowProc)
OnMessage(WM_RBUTTONDOWN, WindowProc)
OnMessage(WM_RBUTTONUP  , WindowProc)

Wnd := Gui(, Format("WaterFx DEMO | {} bits", 8*A_PtrSize))
Wnd.OnEvent("Escape", ExitApp)
Wnd.OnEvent("Close", ExitApp)

if !wfx.Load(LoadPicture(FileSelect())) || !wfx.Start(Wnd)
    MsgBox("ERROR!"), ExitApp()

Wnd.Show(Format("w{} h{}", wfx.Width, wfx.Height))

WindowProc(wParam, lParam, Message, hWnd)
{
    if !wfx.status
        return

    x := lParam & 0xFFFF
    y := (lParam >> 16) & 0xFFFF

    switch Message
    {
    case WM_MOUSEMOVE:
        wfx.Blob(x, y)
    case WM_LBUTTONDOWN, WM_LBUTTONUP:
        wfx.Blob(x, y, 3, 300)
    case WM_RBUTTONDOWN, WM_RBUTTONUP:
        wfx.Blob(x, y, 4, 900)
    }
}

class WaterFx
{
    __New()
    {
        this.id := DllCall("waterfx\create", "Cdecl Ptr")
        this.status := false
    }

    __Delete()
    {
        DllCall("waterfx\destroy", "Ptr", this.id, "Cdecl")
    }

    SetDensity(Density)
    {
        DllCall("waterfx\set_density", "Ptr", this.id, "Int", Density, "Cdecl")
    }

    SetRenderingDelay(RenderingDelay)
    {
        DllCall("waterfx\set_delay", "Ptr", this.id, "UInt", RenderingDelay, "Cdecl")
    }

    Load(hBitmap)  ; DIB 32bpp
    {
        return DllCall("waterfx\load", "Ptr", this.id, "Ptr", hBitmap, "Cdecl")
    }

    Clear()
    {
        DllCall("waterfx\clear", "Ptr", this.id, "Cdecl")
    }

    Blob(X, Y, Radius := 2, Height := 30)
    {
        DllCall("waterfx\blob", "Ptr", this.id, "Int", X, "Int", Y, "Int", Radius, "Int", Height, "Cdecl")
    }

    Render(hWnd)
    {
        DllCall("waterfx\render_hwnd", "Ptr", this.id
            , "Ptr", IsObject(hWnd) ? hWnd.hWnd : hWnd, "Cdecl")
    }

    Start(hWnd)
    {
        return this.status := DllCall("waterfx\start", "Ptr", this.id
            , "Ptr", IsObject(hWnd) ? hWnd.hWnd : hWnd, "Cdecl Ptr")  ; hThread
    }

    Stop()
    {
        DllCall("waterfx\stop", "Ptr", this.id, "Cdecl")
        this.status := false
    }

    Width[]
    {
        get => DllCall("waterfx\get_width", "Ptr", this.id, "Cdecl")
    }

    Height[]
    {
        get => DllCall("waterfx\get_height", "Ptr", this.id, "Cdecl")
    }
}
```
