
## Rust App前端开发框架

Rust的前端App开发框架目前还处在比较早期，还没有产生有巨大影响力的可以挑战目前主流前端开发框架的项目，但是Rust语言最初是被发明来做浏览器开发的，所以Rust的语言特性对前端应用开发也是非常有价值的，例如：安全的并发，异步并发，基于tratis和generics的OO设计，zaro-overhead abstraction也非常有利于减少app分发尺寸。




![](https://i.imgur.com/u8flvKB.png)

## Tauri
Rust前端应用目前主要针对桌面App的开发，移动应用专用框架还没有。开发路线基本有两条，主要是图形渲染机制的差异。一条使用基于CSS/HTML的渲染机制，可以很好集成web生态的前端设计生态，最典型的项目如Tauri(https://tauri.app/)，这个项目类似跨平台混合应用的electron（https://www.electronjs.org/）
区别在于Electron为了解决跨平台的体验一致性，打包了基于chromium的webview和nodejs框架，所以分发尺寸比较大，而Tauri则利用了OS本身内置的webview引擎，另外使用Rust作为应用逻辑开发，而不自带nodejs，所以分发的尺寸比较小。Tauri的核心是对基于webview渲染的窗口管理，提供了自己的窗口管理（https://github.com/tauri-apps/tao，fork于winit）
和对多平台的webview抽象层（https://github.com/tauri-apps/wry），
在此之上，提供了webview和Rust后台直接的事件和命令对通bridge，这样在app后段的Rust应用可以接受来自webview的event，webview前端JS代码也可以调用后端的Rust应用能力。Tauri通过plugin机制可以和其他的Rust应用框架对通，例如：可以和yew这样的web应用框架对接，yew提供了更加完善的前端应用需要的UI状态和View管理等功能，有了yew，就可以把一个完整的web app移植到Tauri。也可以和Deno对通，deno提供了大量成熟的高性能host API，例如：网络和文件系统，和事件编程模式，也支持使用WASM模块，移动应用需要的计算和I/O密集型业务可以借用deno生态开发。Tauri主要提供了抽象和粘合能力，充分利用了现有的webview生态和Rust异步应用生态，

Tauri作为移动前端开发还处在早期，有个基于Android webview的demo，最大的不同在用移动应用不像桌面应用需要管理独立的多个窗口，移动应用的窗口数目有限，窗口管理机制也不同。可能需要针对移动应用的特点，开发不同的窗口管理机制。

## Servo

webview的路线能很好的利用web前端生态的优势，但是目前具有良好生态的浏览器内核chromium和webkit都是基于C++开发的，如果未来打造纯Rust栈的前端，则需要考虑Servo项目。

Servo是Mozilla使用Rust开发的浏览器内核，也是当初创造Rust语言的第一个需求，Servo内部使用了Rust的内存安全和并发安全机制，实现了异步并行的CSS/HTML解析和渲染管线，而且起模块化的设计非常适合作为嵌入式webview引擎。Servo的两个高价值的实现是

### pathfinder
https://github.com/servo/pathfinder
Pathfinder实现了高性能的SVG向量图渲染和字体渲染，这是任何一个UIKit都必须实现的基础功能

### webrender
https://github.com/servo/webrender
webrender并行渲染图形库基于全新的webgpu标准，chromium和android都基于Skia图形库，Skia具有悠久的历史，基本上Skia实现的是所谓retained模式的渲染机制，即把图形渲染过程分为多个view窗口，每个view可以单独来更新，view最终可以在GPU做叠加，这种机制适应前端UI有大量局部变化的场景，局部的优化只会触发相应的view的重新绘制，而不影响其他view。另一种图形渲染思路是完全使用GPU，利用GPU的shader渲染器不仅仅可以做view的叠加也可以做view的渲染，事实上因为GPU主要是为了做3D渲染来加速游戏等业务，3D渲染的管线是完全交给GPU做的，每个pixel的绘制都又GPU完成，因为GPU是高度并行化的，安装每秒60fps来渲染每个pixel并不占用额外的计算资源和功耗，既然3D的渲染已经完全GPU化，那么2D渲染也完全使用GPU也是有道理的，这种完全不使用内存缓存view优化技术，而直接执行绘制命令的渲染模式又叫做direct mode。Mozilla的这篇博文详细介绍了2D渲染和使用GPU做2D渲染的思路（https://hacks.mozilla.org/2017/10/the-whole-web-at-maximum-fps-how-webrender-gets-rid-of-jank/）

Servo目前进入了linux foundation，目前其比较欠缺的特性是对CSS的完整解析和支持。


## Makepad
另一种思路是完全独立开发基于Rust的图形库和UI框架，这里面比较有代表性的是Makepad项目。Makepad的作者Rik Arends也是web IDE cloud9(https://github.com/c9) 的主要开发者，有丰富的JS前端开发经验。Makepad项目想克服cloud9项目中选择JS和HTML/CSS带来的种种问题，构建一个实时协作的，媲美桌面IDE性能的Rust图形库和IDE产品。Makepad UIKit的主要特点是

* 基于GPU的direct mode渲染
* 使用Rust开发，可以跨Web，Windows，MacOS和Linux平台
* 提供类似React JSX和SwiftUI这样的声明式前端开发DSL，DSL可以动态transcompile为GPU的shader language，不需要重新编译应用就可以修改UI界面
* 提供一套基于UI DSL的UI组件库
* 基于Operation Transformation实现的实时协作编辑

https://github.com/makepad/makepad_docs

Makepad是一款Rust语言开发的模块化和高性能的UIKit，和React JSX或者SWiftUI这样的前端DSL一样，Makepad也有Makepad UI DSL，可以描述UI组件和页面布局，不同的地方是Makepad的DSL充分兼容了figma这样的前端设计工具的语法，这样很容易把设计师通过设计工具产生的UI设计转化为Makepad UI DSL，南向，Makepad的DSL可以动态被编译为GPU的shader language，通过GPU实时渲染出页面，这样可以直接通过修改和更新Makepad UI DSL实现对UI的实时修改。通过直接调用GPU的shader能力做渲染，可以同时支持2D和3D图形渲染，具有很好的前瞻性。Makepad这种直接通过GPU渲染的方式称为Direct Mode，它的优点是渲染任务完全卸载给GPU，不占用CPU资源，传统2D图形库充分利用了2D UI具有很多局部变化的特点，对渲染页面做了内存优化，以大幅度减少需要渲染的工作量，这种方式叫做retained mode，Makepad实现中也借鉴了retained mode，减少过多使用GPU带来的功耗问题，


Makepad架构

* Makepad北向提供了基于UI DSL描述的组件库，这些组件库可以实现动态更新和动态编译，不需要重新编译Rust应用，就可以动态调试UI界面，达到了类似web应用开发的速度和体验。这个特性主要为了支持Makepad的studio产品，studio融合和设计师的设计界面，对标figma，也是前端工程师的IDE界面，设计师和前端工程师直接的hand-off流程是通过Makepad DSL实现的，DSL的语法非常接近figma输出的UI组件描述语法，如下面的代码所示，描述一个页面框，调用了frame组件，通过UI DSL描述大小、边缘、填充色等，这些UI外观属性可以动态修改，不需要重新编译Rust代码，所见即所得。这个思路和之前的想法很一致（https://github.com/mistyminds/mistyminds/blob/master/posts/2020-07-27-%E6%96%B0%E4%B8%80%E4%BB%A3%E7%9A%84app%E5%BC%80%E5%8F%91.md）

```
live_register!{
    use makepad_component::frame::*;
    use FrameComponent::*;
    App: {{App}} {
        frame: {
            width: Fill
            height: Fill
            layout:{align: {x: 0.0, y: 0.5}, padding: 30,spacing: 30.}
            Solid {bg:{color: #0f0}, width: Fill, height: 40}
            Solid {
                bg:{color: #0ff},
                layout:{padding: 10, flow: Down, spacing: 10},
                width: Fit,
                height: 300
                Solid {bg:{color: #00f}, width: 40, height: Fill}
                Solid {bg:{color: #f00}, width: 40, height: 40}
                Solid {bg:{color: #00f}, width: 40, height: 40}
            }
            Solid {bg:{color: #f00}, width: 40, height: 40}
            Solid {bg:{color: #f0f}, width: Fill, height: 60}
            Solid {bg:{color: #f00}, width: 40, height: 40}
        }
    }
}
```

* UI DSL被内置在App内的基于Rust实现的transcompiler翻译为对应的组件，是通过Rust的Macro实现的，组件编写借用了Shardertoy的2D API，（https://www.shadertoy.com/view/lslXW8）
```

live_register!{
    use makepad_platform::shader::std::*;
    
    DrawLabelText: {{DrawLabelText}} {
        text_style: {
            font: {
                //path: d"resources/IBMPlexSans-SemiBold.ttf"
            }
            font_size: 11.0
        }
        fn get_color(self) -> vec4 {
            return mix(
                mix(
                    #9,
                    #f,
                    self.hover
                ),
                #9,
                self.pressed
            )
        }
    }
    
    Button: {{Button}} {
        bg_quad: {
            instance hover: 0.0
            instance pressed: 0.0
            
            const BORDER_RADIUS: 3.0
            
            fn pixel(self) -> vec4 {
                let sdf = Sdf2d::viewport(self.pos * self.rect_size);
                let grad_top = 5.0;
                let grad_bot = 1.0;
                let body = mix(mix(#53, #5c, self.hover), #33, self.pressed);
                let body_transp = vec4(body.xyz, 0.0);
                let top_gradient = mix(body_transp, mix(#6d, #1f, self.pressed), max(0.0, grad_top - sdf.pos.y) / grad_top);
                let bot_gradient = mix(
                    mix(body_transp, #5c, self.pressed),
                    top_gradient,
                    clamp((self.rect_size.y - grad_bot - sdf.pos.y - 1.0) / grad_bot, 0.0, 1.0)
                );
                
                // the little drop shadow at the bottom
                let shift_inward = BORDER_RADIUS + 4.0;
                sdf.move_to(shift_inward,self.rect_size.y-BORDER_RADIUS);
                sdf.line_to(self.rect_size.x-shift_inward,self.rect_size.y-BORDER_RADIUS);
                sdf.stroke(
                    mix(mix(#2f,#1f,self.hover), #0000, self.pressed),
                    BORDER_RADIUS
                )

                sdf.box(
                    1.,
                    1.,
                    self.rect_size.x - 2.0,
                    self.rect_size.y - 2.0,
                    BORDER_RADIUS
                )
                sdf.fill_keep(body)
                
                sdf.stroke(
                    bot_gradient,
                    1.0
                )
                
                return sdf.result
            }
        }
        
        walk: {
            width: Size::Fit,
            height: Size::Fit,
            margin: {left: 1.0, right: 1.0, top: 1.0, bottom: 1.0},
        }
        
        layout: {
            align: {x: 0.5, y: 0.5},
            padding: {left: 14.0, top: 10.0, right: 14.0, bottom: 10.0}
        }
        
        state:{
            hover = {
                default: off,
                off = {
                    from: {all: Play::Forward {duration: 0.1}}
                    apply: {
                        bg_quad: {pressed: 0.0, hover: 0.0}
                        label_text: {pressed: 0.0, hover: 0.0}
                    }
                }
                
                on = {
                    from: {
                        all: Play::Forward {duration: 0.1}
                        pressed: Play::Forward {duration: 0.01}
                    }
                    apply: {
                        bg_quad: {pressed: 0.0, hover: [{time: 0.0, value: 1.0}],}
                        label_text: {pressed: 0.0, hover: [{time: 0.0, value: 1.0}],}
                    }
                }
                
                pressed = {
                    from: {all: Play::Forward {duration: 0.2}}
                    apply: {
                        bg_quad: {pressed: [{time: 0.0, value: 1.0}], hover: 1.0,}
                        label_text: {pressed: [{time: 0.0, value: 1.0}], hover: 1.0,}
                    }
                }
            }
        }
    }
}

```

* 并注册适当的event handler，组件的外观描述和其event handler被封装在一个组件模块，实现模式类似JSX

```
impl Button {
    
    pub fn handle_event(&mut self, cx: &mut Cx, event: &mut Event) -> ButtonAction {
        
        self.state_handle_event(cx, event);
        let res = self.button_logic.handle_event(cx, event, self.bg_quad.area());
        
        match res.state {
            ButtonState::Pressed => self.animate_state(cx, ids!(hover.pressed)),
            ButtonState::Default => self.animate_state(cx, ids!(hover.off)),
            ButtonState::Hover => self.animate_state(cx, ids!(hover.on)),
            _ => ()
        };
        res.action
    }
    
    pub fn draw_label(&mut self, cx: &mut Cx2d, label: &str) {
        self.bg_quad.begin(cx, self.walk, self.layout);
        self.label_text.draw_walk(cx, Walk::fit(), Align::default(), label);
        self.bg_quad.end(cx);
    }
    
    pub fn draw_walk(&mut self, cx: &mut Cx2d, walk: Walk) {
        self.bg_quad.begin(cx, walk, self.layout);
        self.label_text.draw_walk(cx, Walk::fit(), Align::default(), &self.label);
        self.bg_quad.end(cx);
    }
}
Footer

```
* 对组件的绘制更新操作会被内置的在线layout引擎管理，做去重和batch化等优化操作，形成一个渲染节点组成的tree。



* 对组件的操作构成一个绘制列表，发送给下游的shader编译器，编译器会把GLSL动态编译为底层Graphics API需要的底层语言，例如：SPIR-V，进而发送给GPU进行绘制操作。
* 借鉴了servo项目的pathfinder2，实现了基于GPU的字体渲染

![](https://i.imgur.com/yStkSi9.png)


Makepad UI Kit和应用的集成分为两种模式

* 一种是设计师设计流程和App开发流程整合在一个IDE中，这样设计师可以持续修改设计，前端工程师可以持续开发代码，因为Makepad studio具备类似google doc这样的实时协作编辑能力，设计师和前端工程师的工作始终可以同步。
* 另一种是，前端工程师使用已经pregenerated UI 组件，不需要再更改外观设计

![](https://i.imgur.com/oeEKi9l.png)


## ZED IDE项目
https://zed.dev/tech
无独有偶，Github的ATOM团队今年也宣布暂停基于Electron框架的ATOM IDE开发，同时启动了一个新的IDE项目，基本思路和Makepad的思路类似
 
* 基于Rust语言实现跨平台
* 基于GPU实现的direct 渲染模式的图形库，称为GPUI
* 基于CRDT实现的实时协作编辑




## WGPU 

https://wgpu.rs


WGPU提供对WebGPU标准API的封装， 基于Rust实现，是目前WebGPU生态最重要的底层graphic library。WebGPU是W3C标准，致力支持最新的底层Graphic API，例如：Vulcan，Metal，D3D12，

https://blog.devgenius.io/will-webgpu-be-the-webgl-killer-60a49509b806

https://surma.dev/things/webgpu/

WebGPU的优势
* Reduce CPU overheads
* Good support of multi-threading
* Bringing the Power of General-Purpose Computing (GPGPU) to the Web with Compute Shaders
* 提供基于Rust语法的新的shader language WSGL
* Technology that will support “Real-time ray tracing” in the future

WGPU支持基于web核native的的应用场景




![](https://i.imgur.com/3ARyyxj.png)

