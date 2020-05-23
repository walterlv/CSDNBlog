

在 XAML 中写绑定是 WPF 学习的必修课，进阶一点的，是用 C# 代码来写绑定。然而一旦绑定的属性是附加属性，好多小伙伴就会开始遇到坑了。

本文将介绍如何在 XAML 和 C# 代码中绑定附加属性。

---

@[TOC](本文内容)

## 背景代码

开始遇到这个问题的背景是我定义了一个附加属性，然后试图通过绑定的方式完成一些业务。

用附加属性来完成的很大一个好处在于不需要改动原有的代码破坏原来的类。例如我只需要在任何一个类中定义 `IsDraggable` 附加属性，就可以让我其他地方的 `Grid` `Button` 等支持拖拽。

```csharp
public class DraggableElement : FrameworkElement
{
    static TabViewItem()
    {
        DefaultStyleKeyProperty.OverrideMetadata(typeof(DraggableElement),
            new FrameworkPropertyMetadata(typeof(DraggableElement)));
    }

    public static readonly DependencyProperty IsDraggableProperty =
        DependencyProperty.RegisterAttached(
            "IsDraggable", typeof(bool), typeof(TabViewItem),
            new PropertyMetadata(true));

    public static bool GetIsDraggable(DependencyObject item)
        => (bool) item.GetValue(IsDraggableProperty);

    public static void SetIsDraggable(DependencyObject obj, bool value)
        => obj.SetValue(IsDraggableProperty, value);
}
```

## 在 XAML 中绑定附加属性

在 XAML 中绑定附加属性的时候需要加上括号和类型的命名空间前缀：

```xml
<ListViewItem Content="{Binding (local:DraggableElement.IsDraggable), RelativeSource={RelativeSource Self}}"
              local:DraggableElement.IsDraggable="True" />
```

对于 WPF 内置的命名空间（`http://schemas.microsoft.com/winfx/2006/xaml/presentation` 命名空间下），是不需要加前缀的。

```xml
<TextBlock x:Name="DemoTextBlock" Grid.Row="1"
           Text="{Binding (Grid.Row), RelativeSource={RelativeSource Self}}" />
```

跟其他的绑定一样，这里并不需要在 `Binding` 后面写 `Path=`，因为 `Binding` 的构造函数中传入的参数就是赋值给 `Path` 的。

## 在 C# 代码中绑定附加属性

上面在说明附加属性绑定的时候我特地额外写了一个不需要写命名空间的 XAML 绑定附加属性的代码，这是为了说明接下来写 C# 代码时的注意事项。

是这样写吗？

```csharp
// 给不看全文的小伙伴：这段代码是无法工作的！正常工作的在后文。
Binding binding = new Binding("(Grid.Row)")
{
    Source = DemoTextBlock,
}
BindingOperations.SetBinding(DemoTextBlock, TextBox.TextProperty, binding);
```

设想应该不是，因为 C# 代码中是没有命名空间前缀的，于是对于前面 XAML 中 `(local:DraggableElement.IsDraggable)` 的 `local` 部分就很不好处理。

实际上，这里的字符串即便是写成 `System.Windows.Grid.Row` 和 `Walterlv.BindingDemo.DraggableElement.IsDraggable` 也依然会绑定失败。

在 C# 代码中绑定附加属性，需要 **使用依赖项属性，而不能使用字符串**！

```csharp
Binding binding = new Binding
{
    Source = DemoTextBlock,
    Path = new PropertyPath(Grid.RowProperty),
}
BindingOperations.SetBinding(DemoTextBlock, TextBox.TextProperty, binding);
```

```csharp
Binding binding = new Binding
{
    Source = DemoDraggableElement,
    Path = new PropertyPath(DraggableElement.IsDraggableProperty),
}
BindingOperations.SetBinding(DemoDraggableElement, TextBox.TextProperty, binding);
```

因此需要特别注意，附加属性的绑定不再能使用字符串，需要使用依赖项属性。

---

**参考资料**

- [Binding to an Attached Property](http://geekswithblogs.net/NewThingsILearned/archive/2008/01/15/binding-to-an-attached-property.aspx)

---

我的博客会首发于 <https://blog.walterlv.com/>，而 CSDN 会从其中精选发布，但是一旦发布了就很少更新。

如果在博客看到有任何不懂的内容，欢迎交流。我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入。

![知识共享许可协议](https://img-blog.csdnimg.cn/20190406094629787.png)

本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。欢迎转载、使用、重新发布，但务必保留文章署名吕毅（包含链接：<https://walterlv.blog.csdn.net/>），不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请[与我联系](mailto:walter.lv@qq.com)。
