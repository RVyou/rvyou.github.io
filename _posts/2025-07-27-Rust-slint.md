---
title: slint 入门(持续更新)
author: rvyou
date: 2025-07-27 20:28:00
categories: [rust,slint]
tags: [rust,slint]
---
一直会更新一些使用的问题和解决方案。

不会很全，详细请看官方文档

## slint 声明式语言

### 基本语法

- 参考一 基本认识

```text
//引入的组件 AboutSlint 自定义组件
import { AboutSlint, Button, VerticalBox } from "std-widgets.slint";

// demo 是导出组件名字 ，export可选导出，还可以追加 inherits  Window(主窗口)
export component Demo {
  
    property <int> name :1;
    
    //标签以及属性
    VerticalBox {
        alignment: start;
        Text {
            text: "Hello World!"+name;
            font-size: 24px;
            horizontal-alignment: center;
        }
        AboutSlint {
            preferred-height: 150px;
        }
        HorizontalLayout { alignment: center; Button { text: "OK!"; } }
    }
}

```
- 参考二 变量相关

```text
import { AboutSlint, Button, VerticalBox } from "std-widgets.slint";
//全局变量 导出export可以给其他文件访问
export global Logic  {
    in-out property <int> the-value;
}
export component Demo {
    //private（默认）：该属性只能从组件内部访问。
    //in：该属性是一个输入。它可以由组件的用户设置和修改，例如通过绑定或在回调中赋值。组件可以提供默认绑定，但不能通过赋值覆盖它。
    //out：只能由组件设置的输出属性。对于组件的用户而言，它是只读的。
    //in-out：该属性可以被所有人读取和修改。
    in property <int> name1;
    property <int> name :111;//内部访问的变更了
    //定义变量
    a:=VerticalBox {
        //change callbacks  当height 发送变化时候，做出相应内容
        changed height => {
            root.name =root.name+1;;
        }
        b:=Text {
            text: "Hello World!"+root.name;//root self parent 限定关键字
        }
    }
   Text {
        text: b.text; //定义变量可以在component 访问
   }
}
```
- 参考三 绑定

### 组件
#### 主题和渲染
主题在 build.rs 可以指定
```rust
//native			这是根据平台而定的其他样式的别名。
// 它适用cupertino于 macOS、fluentWindows、materialAndroid、qtLinux（如果 Qt 可用）
// 或fluent其他平台。
fn main() {
    let config =
        slint_build::CompilerConfiguration::new()
            .with_style("material".into());
    slint_build::compile_with_config("ui/app-window.slint", config).unwrap();
}
```
- 环境变量选择渲染器

SLINT_BACKEND=Qt 选择 Qt 后端

SLINT_BACKEND=winit 选择 winit 后端

SLINT_BACKEND=winit-femtovg 选择带有 FemtoVG 渲染器的 winit 后端

SLINT_BACKEND=winit-skia 选择带有 skia 渲染器的 winit 后端

SLINT_BACKEND=winit-software 选择带有软件渲染器的 winit 后端

如果所选后端不可用，则将使用默认值。 也可以 feature 配置


#### 布局组件
1. 通用布局属性
2. 网格布局 GridLayout
3. 水平布局 HorizontalLayout
4. 垂直布局 VerticalBox

