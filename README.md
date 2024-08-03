# malek_questions_to_eric

```bash

[dependencies]
image = "0.25.1"
mime_guess = "2.0.5"
serde = { version = "1.0.204", features = ["derive"] }
serde_json = "1.0.120"
wry = { version = "0.41.0", features = [
	"devtools",
	"transparent",
	"fullscreen",
	"protocol",
	
] }
allocator_api = "0.6.0"
tao = "0.28.1"
ureq = { version = "2.10.0", default-features = false, features = [
	"native-tls",
	"gzip",
	"json",
] }
base64 = { version = "0.22.1", features = ["alloc"] }
native-tls = "0.2.12"
os_info = "3.8.2"
directories = "5.0.1"
opener = "0.7.1"
rfd = { version = "0.14.1", default-features = false, features = [
	"xdg-portal",
] }
fs_extra = "1.3.0"
anyhow = "1.0.86"
flate2 = "1.0.30"
png = "0.17.13"
sys-locale = "0.3.1"
url = "2.5.2"
glob = "0.3.1"
tauri-winres = "0.1.1"
serde-pyobject = "0.4.0"

[target.'cfg(target_os = "macos")'.dependencies]
cocoa = { version = "0.25.0" }
objc = "0.2.7"
active-win-pos-rs = "0.8.3"

[target.'cfg(target_os = "windows")'.dependencies]
winapi = "0.3.9"
windows = { version = "0.58.0", features = ["Win32_Foundation"] }




```


### first idea


```rust
use tao::{
    event::{Event, WindowEvent}, event_loop::{ControlFlow, EventLoop, EventLoopBuilder}, window::{Window, WindowAttributes, WindowBuilder}
};
use wry::{WebView, WebViewAttributes, WebViewBuilder};




pub enum PyEventAPI {}





struct WindowFrame {
    win_attrs: WindowAttributes,
}


impl WindowFrame{

    pub fn new() -> WindowFrame {
        Self {
            win_attrs: Default::default(),
        }
    }
    pub fn with_title(&mut self, title:&str){
        self.win_attrs.title = title.into();
    }
}




pub struct WebViewFrame {
    pub webview_attrs: WebViewAttributes,
}

impl WebViewFrame {

    pub fn new() -> WebViewFrame {
        Self {
            webview_attrs: Default::default(),
        }
    }

    pub fn with_url(&mut self, url: &str) {
        self.webview_attrs.url = Some(url.into());
    }
}

 

 struct PyFrame {
     event_loop: EventLoop<PyEventAPI>,
     window: Window,
     webview: WebView,
 }
 

 impl PyFrame {

    pub fn new(
        window:&WindowFrame,
        webview:&WebViewFrame,
        _web_context: Option<bool>,
    ) -> PyFrame {
        
        let event_loop = EventLoopBuilder::<PyEventAPI>::with_user_event().build();
        let mut window_builder = WindowBuilder::new();

        // We need the Python class instance to set the WindowBuilder methods
      
        window_builder.window = window.win_attrs.clone();

        let main_window = window_builder.build(&event_loop).unwrap();

        let mut webview_builder = WebViewBuilder::new(&main_window);

        let attributes = &webview.webview_attrs;
        webview_builder.attrs = attributes;   //  mismatched types expected `WebViewAttributes`, found `&WebViewAttributes`

        let main_webview = webview_builder.build().unwrap();

            Self {
                event_loop: event_loop,
                window: main_window,
                webview: main_webview,
            }
    }
    pub fn run(self){
        self.event_loop.run(move |event, _, control_flow| {
            *control_flow = ControlFlow::Wait;
    
            if let Event::WindowEvent {
                event: WindowEvent::CloseRequested,
                ..
            } = event
            {
                *control_flow = ControlFlow::Exit
            }
        });
    }
}
 
 // Implement `Send` for `WebViewFrame` if it is necessary and safe
 unsafe impl Send for WebViewFrame {}
 // Implement `Send` for `PyFrame` if it is necessary and safe
 unsafe impl Send for PyFrame {}
 



fn main() {
    // Initialize WindowFrame and WebViewFrame
    let window_frame = WindowFrame::new();
    let webview_frame = WebViewFrame::new();

    // Create PyFrame
    let py_frame = PyFrame::new(&window_frame, &webview_frame, None);

    // Run the event loop (if needed)
    py_frame.run();
}

```



### second idea


```rust


use tao::{
    event::{Event, WindowEvent}, event_loop::{ControlFlow, EventLoop, EventLoopBuilder}, window::{Window, WindowAttributes, WindowBuilder}
};
use wry::{WebView, WebViewAttributes, WebViewBuilder};




pub enum PyEventAPI {}





struct WindowFrame {
    win_attrs: WindowAttributes,
}


impl WindowFrame{

    pub fn new() -> WindowFrame {
        Self {
            win_attrs: Default::default(),
        }
    }
    pub fn with_title(&mut self, title:&str){
        self.win_attrs.title = title.into();
    }
}




pub struct WebViewFrame {
    webview_attrs: WebViewAttributes,
}

impl WebViewFrame {

    pub fn new() -> WebViewFrame {
        Self {
            webview_attrs: Default::default(),
        }
    }

    pub fn with_url(&mut self, url: &str) {
        self.webview_attrs.url = Some(url.into());
    }



    pub fn build_webview(&self, main_window: &Window) -> WebView {
        let mut webview_builder = WebViewBuilder::new(&main_window);
        webview_builder.attrs = self.webview_attrs // cannot move out of `self.webview_attrs` which is behind a shared reference move occurs because `self.webview_attrs` has type `WebViewAttributes`, which does not implement the `Copy` ;   

    let main_webview = webview_builder.build().unwrap();
    main_webview

    }

}

 

 struct PyFrame {
    event_loop: EventLoop<PyEventAPI>,
    window: Window,
    webview: WebView,
 }
 

 impl PyFrame {

    pub fn new(
        window:&WindowFrame,
        webview:&WebViewFrame,
        _web_context: Option<bool>,
    ) -> PyFrame {
        
        let event_loop = EventLoopBuilder::<PyEventAPI>::with_user_event().build();
        let mut window_builder = WindowBuilder::new();

        // We need the Python class instance to set the WindowBuilder methods
      
        window_builder.window = window.win_attrs.clone();

        let main_window = window_builder.build(&event_loop).unwrap();
        let main_webview = webview.build_webview(&main_window);



        Self {
            event_loop: event_loop,
            window: main_window,
            webview: main_webview,
        }
    }



    pub fn run(self){
        self.event_loop.run(move |event, _, control_flow| {
            *control_flow = ControlFlow::Wait;
    
            if let Event::WindowEvent {
                event: WindowEvent::CloseRequested,
                ..
            } = event
            {
                *control_flow = ControlFlow::Exit
            }
        });
    }
}

 
 // Implement `Send` for `WebViewFrame` if it is necessary and safe
 unsafe impl Send for WebViewFrame {}
 // Implement `Send` for `PyFrame` if it is necessary and safe
 unsafe impl Send for PyFrame {}
 



fn main() {
    // Initialize WindowFrame and WebViewFrame
    let window_frame = WindowFrame::new();
    let webview_frame = WebViewFrame::new();

    // Create PyFrame
    let py_frame = PyFrame::new(&window_frame, &webview_frame, None);

    // Run the event loop (if needed)
    py_frame.run();
}


```
