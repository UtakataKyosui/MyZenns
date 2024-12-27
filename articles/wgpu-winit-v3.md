---
title: "winitのv3を使っている場合のRustでのWebGPUプログラム"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust","webgpu"]
published: false
---

# winitの更新が来た話
RustでGUIアプリを作られている皆さん、
winitのバージョンが変わりましたね。
そこで、皆さんに残念？なお知らせがあります。
**コードの書き方が大きく変わってしまいました。**

どう変わったかを確認してもらうため、
以下のコードを見てもらいます。
まず、v3以前のコードです。
wgpuクレートの動かすための勉強が出来るサイトの、
winitのプログラムの部分から引用しています

## v3以前
```rust:main.rs
use winit::{
    event::*,
    event_loop::EventLoop,
    keyboard::{KeyCode, PhysicalKey},
    window::WindowBuilder,
};

pub fn run() {
    env_logger::init();
    let event_loop = EventLoop::new().unwrap();
    let window = WindowBuilder::new().build(&event_loop).unwrap();

    event_loop.run(move |event, control_flow| match event {
        Event::WindowEvent {
            ref event,
            window_id,
        } if window_id == window.id() => match event {
            WindowEvent::CloseRequested
            | WindowEvent::KeyboardInput {
                event:
                    KeyEvent {
                        state: ElementState::Pressed,
                        physical_key: PhysicalKey::Code(KeyCode::Escape),
                        ..
                    },
                ..
            } => control_flow.exit(),
            _ => {}
        },
        _ => {}
    });
}
```
## v3以後
続いて、v3以後で同じ動作をするコードです
これは`doc.rs`にあるwinitのサンプルコードから引用しています

```rust
use winit::application::ApplicationHandler;
use winit::event::WindowEvent;
use winit::event_loop::{ActiveEventLoop, ControlFlow, EventLoop};
use winit::window::{Window, WindowId};

#[derive(Default)]
struct App {
    window: Option<Window>,
}

impl ApplicationHandler for App {
    fn resumed(&mut self, event_loop: &ActiveEventLoop) {
        self.window = Some(event_loop.create_window(Window::default_attributes()).unwrap());
    }

    fn window_event(&mut self, event_loop: &ActiveEventLoop, id: WindowId, event: WindowEvent) {
        match event {
            WindowEvent::CloseRequested => {
                println!("The close button was pressed; stopping");
                event_loop.exit();
            },
            WindowEvent::RedrawRequested => {
                // Redraw the application.
                //
                // It's preferable for applications that do not render continuously to render in
                // this event rather than in AboutToWait, since rendering in here allows
                // the program to gracefully handle redraws requested by the OS.

                // Draw.

                // Queue a RedrawRequested event.
                //
                // You only need to call this if you've determined that you need to redraw in
                // applications which do not always need to. Applications that redraw continuously
                // can render here instead.
                self.window.as_ref().unwrap().request_redraw();
            }
            _ => (),
        }
    }
}

fn main() {
    let event_loop = EventLoop::new().unwrap();

    // ControlFlow::Poll continuously runs the event loop, even if the OS hasn't
    // dispatched any events. This is ideal for games and similar applications.
    event_loop.set_control_flow(ControlFlow::Poll);

    // ControlFlow::Wait pauses the event loop if no events are available to process.
    // This is ideal for non-game applications that only update in response to user
    // input, and uses significantly less power/CPU time than ControlFlow::Poll.
    event_loop.set_control_flow(ControlFlow::Wait);

    let mut app = App::default();
    event_loop.run_app(&mut app);
}
```

**結構変わったなあ～！** 

## どのような変化なのか
1. `event_loop`が実行するプログラムはApplicationHandlerを継承した構造体に関連づけられたプログラムになった
2. `WindowBuilder`を用いらなくなり、event_loopの型にあるメソッドとして`Window`を作成する関数があり、それを用いるようになった。
3. `ControlFlow`はEventLoopの実行前に設定するようになった
4. `WindowEvent`の内容によって実行するプログラムの内容は、`EventLoop`のP`run()`の引数として関数を渡すのではなく、ApplicationHandlerを継承した構造体に関連づけられた関数として記述するようになった
