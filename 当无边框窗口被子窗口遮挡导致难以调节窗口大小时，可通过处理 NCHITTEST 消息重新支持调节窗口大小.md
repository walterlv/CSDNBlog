

做无边框窗口之后，我们有方法可以让窗口的标题栏区域和边缘调大小的区域继续正常工作，直到——这个窗口上面覆盖了其他的子窗口。这个子窗口会吃掉消息导致父窗口的边缘无法再继续处理这些消息。

---

@[TOC](本文内容)

## 子窗口遮挡了父窗口

看一下下面的动画，这个窗口的下半部分放了一个子窗口。

![被子窗口遮挡了边缘的父窗口](/static/posts/2020-04-11-two-windows.gif)

然后尝试在边缘调节窗口尺寸，会发现被子窗口覆盖的部分是无法完成窗口大小调节的。

![子窗口区域无法调节窗口大小](/static/posts/2020-04-11-resize-with-child-windows.gif)

究其原因，是子窗口处理掉了与调窗口大小相关的消息，导致父窗口完全不知道应该如何处理这个时候的操作。

## 在子窗口处理消息循环

在我的[另一篇博客](https://blog.walterlv.com/post/handle-nchittest-message-to-support-resize)中，我有提到通过处理 `WM_NCHITTEST` 消息，返回 `HT_RIGHT` 等来实现支持 Windows 原生窗口功能的效果。然而那种方法是不适用于本文的场景的，如果你试试就会发现，那种方法会使得你只能调子窗口的大小，对父窗口无济于事。

正确的处理方法是当鼠标划过原本应该处在非客户区部分的时候，将消息交给父窗口处理。于是，我们需要在消息循环的处理中返回 `HTTRANSPARENT` 来告诉操作系统这个区域子窗口不处理消息，请交给父窗口。

这里，我以 WPF 的消息循环来写代码。因为只要是 Windows 平台的 UI 框架都有消息循环的处理，所以可以很容易迁移到其他框架甚至是其他语言。

```csharp
public partial class ChildWindow : Window
{
    public ChildWindow()
    {
        InitializeComponent();
        SourceInitialized += ChildWindow_SourceInitialized;
    }

    private async void ChildWindow_SourceInitialized(object sender, EventArgs e)
    {
        var helper = new WindowInteropHelper(this);
        var source = HwndSource.FromHwnd(helper.Handle);
        source.AddHook(WndProc);
    }

    private IntPtr WndProc(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
    {
        const int WM_NCHITTEST = 0x0084;
        const int HTTRANSPARENT = -1;
        switch (msg)
        {
            case WM_NCHITTEST:
                // 这里，我强行让所有区域返回 HTTRANSPARENT，于是整个子窗口都交给父窗口处理消息。
                // 正常，你应该在这里计算窗口边缘。
                handled = true;
                return new IntPtr(HTTRANSPARENT);
            default:
                break;
        }
        return IntPtr.Zero;
    }
}
```

上面的代码会比较简化，因为我让子窗口的所有区域都返回 `HTTRANSPARENT`，这会让整个子窗口区域的消息都不由子窗口处理。如果需要使用这段代码的话，你需要自己判断窗口的边缘。

![子窗口区域可以调节窗口大小](/static/posts/2020-04-11-resize-with-child-windows-2.gif)

如果需要得到当前坐标的话，可以把下面的方法加入到你的项目中：

```csharp
public static (int lowOrder, int highOrder) GetOrderWord(IntPtr value)
{
    int low = unchecked((short) (long) value);
    int high = unchecked((short) ((long) value >> 16));
    return (low, high);
}
```

于是将消息循环中的 `lParam` 传入可以获得当前的坐标（屏幕坐标系）：

```csharp
// 获得当前基于屏幕坐标系的当前鼠标光标位置。
var (x, y) = GetOrderWord(lParam);
```

## 需要注意一些坑

当你准备使用返回 `HTTRANSPARENT` 时，一定要保证你坐标所在的父子窗口在同一个线程！

返回 `HTTRANSPARENT` 时，操作系统只会查找同线程的其他窗口，如果你的父窗口非同一个线程，那么操作系统处理消息循环时是找不到下一个处理消息的窗口的。

如果你一定要在父窗口非同一个线程时返回 `HTTRANSPARENT` 那么你的整个窗口（顶层窗口和子窗口）将无法再操作！你可以阅读 [HTTRANSPARENT is evil - virtualdub.org](http://virtualdub.org/blog/pivot/entry.php?id=147) 了解相关的坑。

---

**参考资料**

- [WM_NCHITTEST message (Winuser.h) - Win32 apps - Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/inputdev/wm-nchittest?source=docs)
- [multithreading - WM_NCHITTEST and HTTRANSPARENT blocks input from message loop - Stack Overflow](https://stackoverflow.com/questions/33628963/wm-nchittest-and-httransparent-blocks-input-from-message-loop)
- [WM_NCHITTEST and HTTRANSPARENT blocks input from message loop - multithreading](https://php.developreference.com/article/17428986/WM_NCHITTEST+and+HTTRANSPARENT+blocks+input+from+message+loop)
- [Click through window with image (WPF) issues (HTTRANSPARENT isn't working)](https://social.msdn.microsoft.com/Forums/Windowsdesktop/en-US/a5e3cbbb-fd07-4343-9b60-6903cdfeca76/click-through-window-with-image-wpf-issues-httransparent-isnt-working?forum=csharplanguage)
- [HTTRANSPARENT is evil - virtualdub.org](http://virtualdub.org/blog/pivot/entry.php?id=147)
- [c++ - how to move parent window without border from child using WM_NCHITTEST - Stack Overflow](https://stackoverflow.com/questions/8969852/how-to-move-parent-window-without-border-from-child-using-wm-nchittest)
- [winapi - Win api in C#. Get Hi and low word from IntPtr - Stack Overflow](https://stackoverflow.com/a/7913393/6233938)

---

我的博客会首发于 <https://blog.walterlv.com/>，而 CSDN 会从其中精选发布，但是一旦发布了就很少更新。

如果在博客看到有任何不懂的内容，欢迎交流。我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入。

![知识共享许可协议](https://img-blog.csdnimg.cn/20190406094629787.png)

本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。欢迎转载、使用、重新发布，但务必保留文章署名吕毅（包含链接：<https://walterlv.blog.csdn.net/>），不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请[与我联系](mailto:walter.lv@qq.com)。
