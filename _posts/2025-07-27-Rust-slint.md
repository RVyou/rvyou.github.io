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


#### FromStr trait

> 实现FromStr trait后，调用 str.parse() 或者 具体类型::from_str(str) 就可以实现字符串到类型的转换。 
> 
> 注意：FromStr 没有生命周期参数(构建出来是具备所有权的)，只能解析本身不包含生命周期的类型。例如，可以解析 i32，不能解析 &i32

#### Deref 和 DerefMut trait

> 自动解引用

#### From trait,Into trait 和 TryFrom, TryInto

> 转换 trait 只要实现一个另一个自动实现，接口抽象能力很强

#### Default trait

> 默认值

#### AsRef<T>和AsMut<T>

> T 实现 AsRef<U>，&T 实现 AsRef<U> 
> 
> T 实现 AsRef<U>，&mut T 实现 AsRef<U> 
> 
> T 实现 AsMut<U>， &mut T 实现 AsMut<U>

#### trait Borrow, BorrowMut,ToOwned,Cow

> T 实现 Borrow<U>，U实现额外的trait(Eq, Ord, Hash等)的时候应该与T相同的行为
> 
> T 实现 BorrowMut trait，那么它也实现了 Borrowed trait
> 
> Borrow<T> 和 BorrowMut<T>，Rust 为泛型 T 和 &T 自动实现这两个 trait
> 
> ToOwned trait 从任意类型 &U 到 T 的转换(Clone 是 &T 到 T )
> 
> Cow 写时 close 读多写少

#### Fn、FuMut、FnOnce traits

> FnOnce: 所有权的方式捕获环境中的变量
> FnMut: 可变引用的方式捕获环境中的变量
> Fn: 不可变引用的方式捕获其环境中的变量

Rust 生命周期消除规则：

> - 每一个引用参数都会获得独自的生命周期 
>   
>   ```rust
>   fn foo(x: & i32, y: &i32){}//两个函数代码是等价
>   fn foo<'a, 'b>(x: &'a i32, y: &'b i32){//独立的生命周期
>   }//如果返回值是其中的某个参数，编译器不清楚返回那个生命周期
>   ```
> 
> - 若只有一个输入生命周期(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的输出生命周期，也就是所有返回值的生命周期都等于该输入生命周期
>   
>   ```rust
>   fn foo(x: &i32) -> &i32{}//两个函数代码是等价
>   fn foo<'a>(x: &'a i32) -> &'a i32{}
>   ```
> 
> - 若存在多个输入生命周期，且其中一个是 &self 或 &mut self，则 &self 的生命周期被赋给所有的输出生命周期
>   
>   ```rust
>   impl<'a> test<'a>{
>       fn foo<'b>(&'a self,s:&'b str)->&'a str{
>       }
>   }//等价与下面函数
>   impl test{
>       fn foo(& self,s:& str)->& str{
>       }
>   }
>   ```

## NLL(Non-Lexical Lifetime)

> 应用的声明周期从借用处开始到最后一次使用的地方(rust 从 1.31 版本开始)

## Reborrow 再借用

> 可以通过 &*v 进行再次借用，再借用生命周期内不能使用 v 的可变借用

```rust
let mut s = String::from("test");
let a = &mut s;
let aa = &*a;
//这里不能使用 a 的可变借用，但是可以使用不可变借用
println!("{}",aa);
//aa的声明周期结束 NLL 规则
println!("{}",a);
```

## ’static

First, you need to set up your account on Google analytics. While you create your account, you must create your first **Property** as well.

1. Head to <https://analytics.google.com/> and click on **Start Measuring**
2. Enter your desired _Account Name_ and choose the desired checkboxes
3. Enter your desired _Property Name_. This is the name of the tracker project that appears on your Google Analytics dashboard
4. Enter the required information _About your business_
5. Hit _Create_ and accept any license popup to set up your Google Analytics account and create your property

### Create Data Stream

With your property created, you now need to set up Data Stream to track your blog traffic. After you signup, the prompt should automatically take you to create your first **Data Stream**. If not, follow these steps:

1. Go to **Admin** on the left column
2. Select the desired property from the drop-down on the second column
3. Click on **Data Streams**
4. Add a stream and click on **Web**
5. Enter your blog's URL

It should look like this:

![google-analytics-data-stream](/posts/20210103/01-google-analytics-data-stream.png){: width="1086" height="542"}

Now, click on the new data stream and grab the **Measurement ID**. It should look something like `G-V6XXXXXXXX`. Copy this to your `_config.yml`{: .filepath} file:

```yaml
google_analytics:
  id: 'G-V6XXXXXXX'   # fill in your Google Analytics ID
  # Google Analytics pageviews report settings
  pv:
    proxy_endpoint:   # fill in the Google Analytics superProxy endpoint of Google App Engine
    cache_path:       # the local PV cache data, friendly to visitors from GFW region
```

{: file="_config.yml"}

When you push these changes to your blog, you should start seeing the traffic on your Google Analytics. Play around with the Google Analytics dashboard to get familiar with the options available as it takes like 5 mins to pick up your changes. You should now be able to monitor your traffic in real time.

![google-analytics-realtime](/posts/20210103/02-google-analytics-realtime.png){: width="616" height="557"}

## Setup Page Views

There is a detailed [tutorial](https://developers.google.com/analytics/solutions/google-analytics-super-proxy) available to set up Google Analytics superProxy. But, if you are interested to just quickly get your Chirpy-based blog display page views, follow along. These steps were tested on a Linux machine. If you are running Windows, you can use the Git bash terminal to run Unix-like commands.

### Setup Google App Engine

1. Visit <https://console.cloud.google.com/appengine>

2. Click on **Create Application**

3. Click on **Create Project**

4. Enter the name and choose the data center close to you

5. Select **Python** language and **Standard** environment

6. Enable billing account. Yeah, you have to link your credit card. But, you won't be billed unless you exceed your free quota. For a simple blog, the free quota is more than sufficient.

7. Go to your App Engine dashboard on your browser and select **API & Services** from the left navigation menu

8. Click on **Enable APIs and Services** button on the top

9. Enable the following APIs: _Google Analytics API_

10. On the left, Click on _OAuth Consent Screen_ and accept **Configure Consent Screen**. Select **External** since your blog is probably hosted for the public. Click on **Publish** under _Publishing Status_

11. Click on **Credentials** on the left and create a new **OAuth Client IDs** credential. Make sure to add an entry under `Authorized redirect URIs` that matches: `https://<project-id>.<region>.r.appspot.com/admin/auth`

12. Note down the **Your Client ID** and **Your Client Secret**. You'll need this in the next section.

13. Download and install the cloud SDK for your platform: <https://cloud.google.com/sdk/docs/quickstart>

14. Run the following commands:
    
    ```console
    [root@bc96abf71ef8 /]# gcloud init
    
    ~snip~
    
    Go to the following link in your browser:
    
        https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=XYZ.apps.googleusercontent.com&redirect_uri=ABCDEFG
    
    Enter verification code: <VERIFICATION CODE THAT YOU GET AFTER YOU VISIT AND AUTHENTICATE FROM THE ABOVE LINK>
    
    You are logged in as: [blah_blah@gmail.com].
    
    Pick cloud project to use:
    [1] chirpy-test-300716
    [2] Create a new project
    Please enter numeric choice or text value (must exactly match list
    item): 1
    ```
    
    [root@bc96abf71ef8 /]# gcloud info
    
    # Your selected project info should be displayed here
    
    ```
    
    ```

### Setup Google Analytics superProxy

1. Clone the **Google Analytics superProxy** project on Github: <https://github.com/googleanalytics/google-analytics-super-proxy> to your local.

2. Remove the first 2 lines in the [`src/app.yaml`{: .filepath}](https://github.com/googleanalytics/google-analytics-super-proxy/blob/master/src/app.yaml#L1-L2) file:
   
   ```diff
   - application: your-project-id
   - version: 1
   ```

3. In `src/config.py`{: .filepath}, add the `OAUTH_CLIENT_ID` and `OAUTH_CLIENT_SECRET` that you gathered from your App Engine Dashboard.

4. Enter any random key for `XSRF_KEY`, your `config.py`{: .filepath} should look similar to this
   
   ```python
   #!/usr/bin/python2.7
   
   __author__ = 'pete.frisella@gmail.com (Pete Frisella)'
   
   # OAuth 2.0 Client Settings
   AUTH_CONFIG = {
     'OAUTH_CLIENT_ID': 'YOUR_CLIENT_ID',
     'OAUTH_CLIENT_SECRET': 'YOUR_CLIENT_SECRET',
     'OAUTH_REDIRECT_URI': '%s%s' % (
       'https://chirpy-test-XXXXXX.ue.r.appspot.com',
       '/admin/auth'
     )
   }
   
   # XSRF Settings
   XSRF_KEY = 'OnceUponATimeThereLivedALegend'
   ```
   
   {: file="src/config.py"}
   
   > You can configure a custom domain instead of `https://PROJECT_ID.REGION_ID.r.appspot.com`.
   > But, for the sake of keeping it simple, we will be using the Google provided default URL.
   > {: .prompt-info }

5. From inside the `src/`{: .filepath} directory, deploy the app
   
   ```console
   [root@bc96abf71ef8 src]# gcloud app deploy
   Services to deploy:
   
   descriptor:      [/tmp/google-analytics-super-proxy/src/app.yaml]
   source:          [/tmp/google-analytics-super-proxy/src]
   target project:  [chirpy-test-XXXX]
   target service:  [default]
   target version:  [VESRION_NUM]
   target url:      [https://chirpy-test-XXXX.ue.r.appspot.com]
   ```
   
    Do you want to continue (Y/n)? Y
   
    Beginning deployment of service [default]...
    ╔════════════════════════════════════════════════════════════╗
    ╠═ Uploading 1 file to Google Cloud Storage                 ═╣
    ╚════════════════════════════════════════════════════════════╝
    File upload done.
    Updating service [default]...done.
    Setting traffic split for service [default]...done.
    Deployed service [default] to [https://chirpy-test-XXXX.ue.r.appspot.com]
   
    You can stream logs from the command line by running:
    $ gcloud app logs tail -s default
   
    To view your application in the web browser run:
    $ gcloud app browse
   
   ```
   
   ```

6. Visit the deployed service. Add a `/admin` to the end of the URL.

7. Click on **Authorize Users** and make sure to add yourself as a managed user.

8. If you get any errors, please Google it. The errors are self-explanatory and should be easy to fix.

If everything went good, you'll get this screen:

![superProxy-deployed](/posts/20210103/03-superProxy-deployed.png){: width="1366" height="354"}

### Create Google Analytics Query

Head to `https://PROJECT_ID.REGION_ID.r.appspot.com/admin` and create a query after verifying the account. **GA Core Reporting API** query request can be created in [Query Explorer](https://ga-dev-tools.appspot.com/query-explorer/).

The query parameters are as follows:

- **start-date**: fill in the first day of blog posting
- **end-date**: fill in `today` (this is a parameter supported by GA Report, which means that it will always end according to the current query date)
- **metrics**: select `ga:pageviews`
- **dimensions**: select `ga:pagePath`

In order to reduce the returned results and reduce the network bandwidth, we add custom filtering rules [^ga-filters]:

- **filters**: fill in `ga:pagePath=~^/posts/.*/$;ga:pagePath!@=`.
  
  Among them, `;` means using _logical AND_ to concatenate two rules.
  
  If the `site.baseurl` is specified, change the first filtering rule to `ga:pagePath=~^/BASE_URL/posts/.*/$`, where `BASE_URL` is the value of `site.baseurl`.

After <kbd>Run Query</kbd>, copy the generated contents of **API Query URI** at the bottom of the page and fill in the **Encoded URI for the query** of SuperProxy on GAE.

After the query is saved on GAE, a **Public Endpoint** (public access address) will be generated, and we will get the query result in JSON format when accessing it. Finally, click <kbd>Enable Endpoint</kbd> in **Public Request Endpoint** to make the query effective, and click <kbd>Start Scheduling</kbd> in **Scheduling** to start the scheduled task.

![superproxy-query](/posts/20210103/04-superproxy-query.png){: width="1100" height="126"}

## Configure Chirpy to Display Page View

Once all the hard part is done, it is very easy to enable the Page View on Chirpy theme. Your superProxy dashboard should look something like below and you can grab the required values.

![superproxy-dashboard](/posts/20210103/05-superproxy-dashboard.png){: width="1210" height="694"}

Update the `_config.yml`{: .filepath} file of [**Chirpy**][chirpy-homepage] project with the values from your dashboard, to look similar to the following:

```yaml
google_analytics:
  id: 'G-V6XXXXXXX'   # fill in your Google Analytics ID
  pv:
    proxy_endpoint: 'https://PROJECT_ID.REGION_ID.r.appspot.com/query?id=<ID FROM SUPER PROXY>'
    cache_path:       # the local PV cache data, friendly to visitors from GFW region
```

{: file="_config.yml"}

Now, you should see the Page View enabled on your blog.

## Reference

[^ga-filters]: [Google Analytics Core Reporting API: Filters](https://developers.google.com/analytics/devguides/reporting/core/v3/reference#filters)

[chirpy-homepage]: https://github.com/cotes2020/jekyll-theme-chirpy/
