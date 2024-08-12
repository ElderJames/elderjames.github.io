---
title: 'Blazor 标签页简化版实现'
slug: 'blazor-simple-reuse-tabs'
date: 2024-08-12
tags: [.NET, Blazor]
toc: true
description: AntDesign Blazor 中 ReuseTabs 通过获取页面实例来控制渲染，本文将介绍另一种方法，不需要传递路由数据，只用一个劫持ChildContetn渲染的方法。
---

核心组件，只渲染一次ChildContent，子内容不再渲染，则这个组件实例就可以达到劫持子内容的效果。

```cs
    public class KeepAlive : IComponent
    {
        private RenderHandle _renderHandle;
        private bool _rendered = false;

        [Parameter] public RenderFragment? ChildContent { get; set; }

        public void Attach(RenderHandle renderHandle)
        {
            _renderHandle = renderHandle;
        }

        public Task SetParametersAsync(ParameterView parameters)
        {
            parameters.SetParameterProperties(this);

            if (!_rendered && ChildContent != null)
            {
                _rendered = true;
                _renderHandle.Render(ChildContent);
            }

            return Task.CompletedTask;
        }
    }
```

然后实现页签和页面内容，分别用循环，然后用 css 来控制页面的切换即可。

```
<!-- Tabs -->

<div>
    @foreach (var item in urls)
    {
        <a href="@item" style="margin:10px;">@item</a>
    }
</div>


@foreach (var item in urls)
{
    <div style="@(item == NavigationManager.Uri ? "display: block" : "display: none") ">
        <KeepAlive>
            @ChildContent
        </KeepAlive>
    </div>
}


@code {
    [Inject] private NavigationManager NavigationManager { get; set; } = default!;

    [Parameter] public RenderFragment? ChildContent { get; set; }

    private HashSet<string> urls = new HashSet<string>();

    protected override void OnInitialized()
    {
        base.OnInitialized();
        urls.Add(NavigationManager.Uri);

        NavigationManager.LocationChanged += OnLocationChanged;
    }

    private void OnLocationChanged(object? target, LocationChangedEventArgs args)
    {
        urls.Add(args.Location);
        InvokeAsync(StateHasChanged);
    }

    public void Dispose()
    {
        NavigationManager.LocationChanged -= OnLocationChanged;
    }
}
```

在布局组件中使用：

```
@inherits LayoutComponentBase

<div class="page">
    <div class="sidebar">
        <NavMenu />
    </div>

    <main>
        <div class="top-row px-4">
            <a href="https://learn.microsoft.com/aspnet/core/" target="_blank">About</a>
        </div>

        <article class="content px-4">
            <Tabs>
                @Body
            </Tabs>
        </article>
    </main>
</div>
```

这样就可以实现页签的简单复用了。