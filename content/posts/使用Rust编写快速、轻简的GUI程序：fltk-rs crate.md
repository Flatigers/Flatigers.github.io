---
title: 使用Rust编写快速、轻简的GUI程序：fltk-rs crate
date: 2022-10-16 23:51:12
tags:
    - Rust
    - 开发
    - fltk-rs
    - GUI
categories:
    - Rust
    - 开发
---



全文字数：3198词

阅读时长：大约5分钟（不包括所有代码）

## 使用Rust编写快速、轻简的GUI程序：fltk-rs crate

这是一篇推荐你使用Rust的文章......

“Rust，一门**赋**予每个人构建可靠且高效软件**能**力的语言。”

截至2021年，在Stack Overflow的年度 [Overview ](https://insights.stackoverflow.com/survey/2021#most-popular-technologies-language)上，**Rust** 已经**连续六年**成为最受开发者喜欢的编程语言！

![Stack Overflow 2021 Overview](https://pic1.imgdb.cn/item/634c136d16f2c2beb1b5df44.png)

Rust作为一门具有零开销抽象的系统编程语言，具有以生命周期和所有权借用的内存安全机制，使代码只要通过编译就没有内存安全问题。再加上rust具有极其活跃友好的社区，丰富的crate库，完备且大部分免费的参考资料，这让世界上的多数开发者喜欢上这门语言！正如Rust官网的介绍所说，作为一门编程语言，其最大特点在于“**赋能**”！

在学习编程语言时，尤其是在学习Rust这种学习曲线 **极其陡峭**（可能是目前最难学的）的语言时，保持自己对学习的兴趣很重要。而有什么能比编写出能看得到界面的程序**更简单**、能让人激动的呢？

![](https://pic1.imgdb.cn/item/634c19e216f2c2beb1c1dbdc.png)

于是，我利用业余时间将fltk-rs book翻译为中文（翻译水平不足，请大家谅解），方便大家学习fltk-rs，并使用Rust写出一个快速漂亮的程序！

由于国外网络有时不能访问，大家在公众号消息界面回复“**fltk-rs**”即可获得该电子书的中文和英文pdf文件，如果可以的话，请在GitHub上点个Star哦！



## FLTK for Rust?

在 Crates.io 上，Rust已经有很多跨平台的GUI框架可供使用了，比如 Iced，Frui，egui 等等。这里我们为什么要选择从FLTK开始学习呢？

最该放在前面的理由就是“**简单**”！

FLTK，全名 **Fast Light Toolkit**。而它正如它的名字一样，又小又快。FLTK库本身是使用 C++ 98开发的，fltk-rs则是使用Rust实现的，它通过FFI（Foreign Function Interface）调用一个FLTK封装器 **cfltk**（该库使用 C89 和 C++11编写），来达到使用rust编写fltk程序的目的。

**小知识：C++之父也在使用FLTK哦！**

现在说说我们使用fltk的好处：

- 构造简单，对习惯使用**面向对象**GUI库的开发者及其友好
- 文档齐全，通过查阅文档可以解决几乎所有问题
- 又轻又快，编译出的文件很小（iced一个测试文件占用100M以上，fltk则不到1M），运行时占用内存小，且快速
- 跨平台，无需动态链接，这让你一次程序可以在Window，Linux，macOS甚至Android上运行（Android还是写原生比较好）
- 组件丰富，具有图像支持，开发方式丰富，开源协议宽松（MIT License）

fltk也有些许缺点，比如对于复杂界面的编写没有很好的支持组件等等，但这些无伤大雅，对于入门rust GUI编写已经绰绰有余了。

下面，就让我们开始简单地学习下fltk-rs吧！

下面的内容可能有些**草率**，但这是让你看一下如何使用fltk-rs编写GUI程序。更全面地学习请参见fltk-rs官方文档和fltk-rs book。



## 配置

首先，确保你配置了 g++，CMake，git和make（linux上），同时可能还要安装一些必须的库。关于库的安装个其他平台工具链的配置请参阅fltk-rs book的Setup配置章节。

将以下代码添加到你的 Cargo.toml 文件：

```toml
[dependencies]
fltk = { version = "^1.3", features = ["fltk-bundled"] }
```

这基本上就可以了，Cargo会帮你下载和导入所有的库。

在这里我曾经踩到个坑，Cargo在下载库时可能会因为网络问题导致没有完全下载好所有需要的crate。但是，**它不会再重新下载了**，而且不会再给出提醒，这将导致一系列的**链接错误**！唯一的解决办法只有清楚cargo目录下缓存的crate文件，然后重新 `cargo build`下载，注意提示信息并保持网络通畅！

如果你也遇到了链接错误的问题，仔细想想`cargo build` 时有没有出错呢。



## 创建一个APP结构

fltk-rs crate在app模块中提供了一个App结构。初始化App结构可以初始化所有内部样式、字体和支持的图像类型。它还初始化了程序将要运行的多线程环境。

```rust
use fltk::*;

fn main() {
    let app = app::App::default();
    app.run().unwrap();
}
```

run方法运行gui应用程序的事件循环（event loop）。 要对事件进行精细的控制，可以使用wait()方法：

```rust
use fltk::*;

fn main() {
    let app = app::App::default();
    while app.wait() {
        // handle events
    }
}

```

此外，App结构允许你使用with_scheme()初始器来设置程序的全局主题：

```rust
use fltk::*;

fn main() {
    let app = app::App::default().with_scheme(app::Scheme::Gtk);
    app.run().unwrap();
}
```

这将使你的程序具有Gtk程序的样子。还有其他的内置方案，Basic、Plastic和Gleam。

App结构还负责在应用程序开始时使用load_system_fonts()方法加载系统字体。

一个典型的fltk-rs应用程序，将在创建任何部件和显示主窗口之前构建App结构。

任何在run()方法调用后添加的逻辑，将在事件循环结束后执行（通常是关闭应用程序的所有窗口时，或者调用quit()方法时）。该逻辑可能包括在必要时重启程序的逻辑。

除了App结构外，App模块本身还包含与你的程序的全局状态有关的结构和自由函数。其中包括设置背景和前景颜色、默认字体和大小等视觉效果、屏幕功能、剪贴板功能、全局处理器、应用事件、通道（channels）（发送器和接收器）和超时。



## 做出一个窗口

FLTK会在它支持的系统平台上调用原生窗口，然后基本上通过自己的方法来绘制。它会在windows上调用HWND，在MacOS上调用NSWindow，在X11系统（linux, BSD）上调用XWindow。

使用这样的代码可以创建一个window：

```rust
use fltk::{prelude::*, *};

fn main() {
    let app = app::App::default();
    let mut my_window = window::Window::new(100, 100, 400, 300, "My Window");
    my_window.end();
    my_window.show();
    app.run().unwrap();
}
```

调用new()函数需要五个参数：

- `x` 以电脑屏幕最左侧为原点的水平距离。
- `y` 以电脑屏幕最左侧为原点的垂直距离。
- `width` window的宽度。
- `height` window的高度。
- `title` window标题。

接下来注意对end()的调用。window，以及其他类型的widget，实现了GroupExt trait。实现该trait的这些部件将 持有 任何在call()和end()间创建的widget（通过new()创建串口时，隐式调用了begin()），或者作为其父widget。 下一个调用show()唤起了window，使其出现在显示屏上。



## 放几个组件

FLTK提供了大约80个窗口组件。这些组件都实现了WidgetBase和WidgetExt的基本trait集。 我们已经遇到了我们的第一个组件，Window。 正如我们在Window widget中所看到的，小组件也可以根据其功能实现其他trait。 在我们之前写的例子中添加一个按钮：

```rust
use fltk::{prelude::*, *};

fn main() {
    let app = app::App::default();
    let mut my_window = window::Window::new(100, 100, 400, 300, "My Window");
    let mut but = button::Button::new(160, 200, 80, 40, "Click me!");
    my_window.end();
    my_window.show();
    app.run().unwrap();
}
```

注意，这个按钮的父组件是my_window，因为它是在begin()和end()之间创建的。 另一种添加组件的方法是，使用实现了GroupExt trait的widget所提供的add(widget)方法。

```rust
use fltk::{prelude::*, *};

fn main() {
    let app = app::App::default();
    let mut my_window = window::Window::new(100, 100, 400, 300, "My Window");
    my_window.end();
    my_window.show();

    let mut but = button::Button::new(160, 200, 80, 40, "Click me!");
    my_window.add(&but);

    app.run().unwrap();
}
```

另一件要注意的事情是按钮的初始化，它的构造函数基本上与Window相同，这是因为它实现了WidgetBase trait。注意，虽然Window的x和y坐标是相对于屏幕的，但按钮的x和y坐标却是相对于包含按钮的窗口的。你可能已经注意到，这也适用于我们在前一页的嵌入式窗口。



下面我们来看几个其他的示例，具体编写方法和原理请参见官方文档或book！

### 菜单和按钮的示例

这里我们使用闭包设置回调编写这个示例，实际上，fltk还可以允许你使用函数对象来处理回调，或者直接使用sender和message来编写。

```rust
use fltk::{enums::*, prelude::*, *};

fn menu_cb(m: &mut impl MenuExt) {
    if let Some(choice) = m.choice() {
        match choice.as_str() {
            "New\t" => println!("New"),
            "Open\t" => println!("Open"),
            "Third" => println!("Third"),
            "Quit\t" => {
                println!("Quitting");
                app::quit();
            },
            _ => println!("{}", choice),
        }
    }
}

fn main() {
    let a = app::App::default();
    let mut win = window::Window::default().with_size(400, 300);
    let mut menubar = menu::MenuBar::new(0, 0, 400, 40, "rew");
    menubar.add("File/New\t", Shortcut::None, menu::MenuFlag::Normal, menu_cb);
    menubar.add(
        "File/Open\t",
        Shortcut::None,
        menu::MenuFlag::Normal,
        menu_cb,
    );
    let idx = menubar.add(
        "File/Recent",
        Shortcut::None,
        menu::MenuFlag::Submenu,
        menu_cb,
    );
    menubar.add(
        "File/Recent/First\t",
        Shortcut::None,
        menu::MenuFlag::Normal,
        menu_cb,
    );
    menubar.add(
        "File/Recent/Second\t",
        Shortcut::None,
        menu::MenuFlag::Normal,
        menu_cb,
    );
    menubar.add(
        "File/Quit\t",
        Shortcut::None,
        menu::MenuFlag::Normal,
        menu_cb,
    );
    let mut btn1 = button::Button::new(160, 150, 80, 30, "Modify 1");
    let mut btn2 = button::Button::new(160, 200, 80, 30, "Modify 2");
    let mut clear = button::Button::new(160, 250, 80, 30, "Clear");
    win.end();
    win.show();

    btn1.set_callback({
        let menubar = menubar.clone();
        move |_| {
            if let Some(mut item) = menubar.find_item("File/Recent") {
                item.add(
                    "Recent/Third",
                    Shortcut::None,
                    menu::MenuFlag::Normal,
                    menu_cb,
                );
                item.add(
                    "Recent/Fourth",
                    Shortcut::None,
                    menu::MenuFlag::Normal,
                    menu_cb,
                );
            }
        }
    });

    btn2.set_callback({
        let mut menubar = menubar.clone();
        move |_| {
            menubar.add(
                "File/Recent/Fifth\t",
                Shortcut::None,
                menu::MenuFlag::Normal,
                menu_cb,
            );
            menubar.add(
                "File/Recent/Sixth\t",
                Shortcut::None,
                menu::MenuFlag::Normal,
                menu_cb,
            );
        }
    });

    clear.set_callback(move |_| {
        menubar.clear_submenu(idx).unwrap();
    });

    a.run().unwrap();
}

```

### 一个逐渐消失的动画

我们使用了线程

```rust
use fltk::{enums::*, prelude::*, *};

fn main() {
    let a = app::App::default();
    let mut win = window::Window::default().with_size(400, 300);
    win.set_color(Color::White);
    // our button takes the whole left side of the window
    let mut sliding_btn = button::Button::new(0, 0, 100, 300, None);
    style_btn(&mut sliding_btn);
    win.end();
    win.show();

    sliding_btn.set_callback(|btn| {
        if btn.w() > 0 && btn.w() < 100 {
            return; // we're still animating
        }
        std::thread::spawn({
            let mut btn = btn.clone();
            move || {
                while btn.w() != 0 {
                    btn.set_size(btn.w() - 2, btn.h());
                    app::sleep(0.016);
                    app::awake(); // to awaken the ui thread
                    btn.parent().unwrap().redraw();
                }
            }
        });
    });
    a.run().unwrap();
}

fn style_btn(btn: &mut button::Button) {
    btn.set_color(Color::from_hex(0x42A5F5));
    btn.set_selection_color(Color::from_hex(0x42A5F5));
    btn.set_frame(FrameType::FlatBox);
}

```



### 一个绘画板程序

```rust
use fltk::{
    app,
    draw::{
        draw_line, draw_point, draw_rect_fill, set_draw_color, set_line_style, LineStyle, Offscreen,
    },
    enums::{Color, Event, FrameType},
    frame::Frame,
    prelude::*,
    window::Window,
};
use std::cell::RefCell;
use std::rc::Rc;

const WIDTH: i32 = 800;
const HEIGHT: i32 = 600;

fn main() {
    let app = app::App::default().with_scheme(app::Scheme::Gtk);

    let mut wind = Window::default()
        .with_size(WIDTH, HEIGHT)
        .with_label("RustyPainter");
    let mut frame = Frame::default()
        .with_size(WIDTH - 10, HEIGHT - 10)
        .center_of(&wind);
    frame.set_color(Color::White);
    frame.set_frame(FrameType::DownBox);

    wind.end();
    wind.show();

    // We fill our offscreen with white
    let offs = Offscreen::new(frame.width(), frame.height()).unwrap();
    #[cfg(not(target_os = "macos"))]
    {
        offs.begin();
        draw_rect_fill(0, 0, WIDTH - 10, HEIGHT - 10, Color::White);
        offs.end();
    }

    let offs = Rc::from(RefCell::from(offs));

    frame.draw({
        let offs = offs.clone();
        move |_| {
            let mut offs = offs.borrow_mut();
            if offs.is_valid() {
                offs.rescale();
                offs.copy(5, 5, WIDTH - 10, HEIGHT - 10, 0, 0);
            } else {
                offs.begin();
                draw_rect_fill(0, 0, WIDTH - 10, HEIGHT - 10, Color::White);
                offs.copy(5, 5, WIDTH - 10, HEIGHT - 10, 0, 0);
                offs.end();
            }
        }
    });

    frame.handle({
        let mut x = 0;
        let mut y = 0;
        move |f, ev| {
            // println!("{}", ev);
            // println!("coords {:?}", app::event_coords());
            // println!("get mouse {:?}", app::get_mouse());
            let offs = offs.borrow_mut();
            match ev {
                Event::Push => {
                    offs.begin();
                    set_draw_color(Color::Red);
                    set_line_style(LineStyle::Solid, 3);
                    let coords = app::event_coords();
                    x = coords.0;
                    y = coords.1;
                    draw_point(x, y);
                    offs.end();
                    f.redraw();
                    set_line_style(LineStyle::Solid, 0);
                    true
                }
                Event::Drag => {
                    offs.begin();
                    set_draw_color(Color::Red);
                    set_line_style(LineStyle::Solid, 3);
                    let coords = app::event_coords();
                    draw_line(x, y, coords.0, coords.1);
                    x = coords.0;
                    y = coords.1;
                    offs.end();
                    f.redraw();
                    set_line_style(LineStyle::Solid, 0);
                    true
                }
                _ => false,
            }
        }
    });

    app.run().unwrap();
}

```



### 一个flutter风格的计数器

FLTK在风格化应用方面提供了许多东西，这里通过WindowExt trait完成Styling：

```rust
use fltk::{
    enums::{Align, Color, Font, FrameType},
    prelude::*,
    *,
};

const BLUE: Color = Color::from_hex(0x42A5F5);
const SEL_BLUE: Color = Color::from_hex(0x2196F3);
const GRAY: Color = Color::from_hex(0x757575);
const WIDTH: i32 = 600;
const HEIGHT: i32 = 400;

fn main() {
    let app = app::App::default();
    let mut win = window::Window::default()
        .with_size(WIDTH, HEIGHT)
        .with_label("Flutter-like!");
    let mut bar =
        frame::Frame::new(0, 0, WIDTH, 60, "  FLTK App!").with_align(Align::Left | Align::Inside);
    let mut text = frame::Frame::default()
        .with_size(100, 40)
        .center_of(&win)
        .with_label("You have pushed the button this many times:");
    let mut count = frame::Frame::default()
        .size_of(&text)
        .below_of(&text, 0)
        .with_label("0");
    let mut but = button::Button::new(WIDTH - 100, HEIGHT - 100, 60, 60, "@+6plus");
    win.end();
    win.make_resizable(true);
    win.show();

    // Theming
    app::background(255, 255, 255);
    app::set_visible_focus(false);

    bar.set_frame(FrameType::FlatBox);
    bar.set_label_size(22);
    bar.set_label_color(Color::White);
    bar.set_color(BLUE);
    bar.draw(|b| {
        draw::set_draw_rgb_color(211, 211, 211);
        draw::draw_rectf(0, b.height(), b.width(), 3);
    });

    text.set_label_size(18);
    text.set_label_font(Font::Times);

    count.set_label_size(36);
    count.set_label_color(GRAY);

    but.set_color(BLUE);
    but.set_selection_color(SEL_BLUE);
    but.set_label_color(Color::White);
    but.set_frame(FrameType::OFlatFrame);
    // End theming

    but.set_callback(move |_| {
        let label = (count.label().parse::<i32>().unwrap() + 1).to_string();
        count.set_label(&label);
    });

    app.run().unwrap();
}

```

