# rust-darwin-webkit

[Documentation](https://yamadapc.github.io/rust-darwin-webkit/darwin_webkit/)

**darwin_webkit** exposes bindings to some of the WebKit's API on MacOS for
Rust. It uses the `objc` and `cocoa` crates to bind with Objective-C.

Can be embedded onto audio plug-ins by getting the native webview handle and
adding it to a plug-ins native `NSWindowView` handle.

## Example
```rust
extern crate cocoa;
extern crate darwin_webkit;

use darwin_webkit::helpers::dwk_app::DarwinWKApp;

fn main() {
    unsafe {
        let app = DarwinWKApp::new("Simple WebView");
        let webview = app.create_webview();
        webview.load_url("https://www.google.com.br");
        app.set_webview(&webview);
        app.run();
    }
}
```

## Communication example
```rust
extern crate cocoa;
extern crate darwin_webkit;

use cocoa::base::id;
use darwin_webkit::helpers::dwk_app::DarwinWKApp;
use std::rc::Rc;

fn main() {
    unsafe {
        let app = DarwinWKApp::new("Host an app");
        let webview = Rc::new(app.create_webview());

        let mut callback = |_: id, _: id| {
            println!("JavaScript called rust!");
            webview.evaluate_javascript("document.body.innerHTML += ' -> response from rust';");
        };
        webview.add_message_handler("hello", &mut callback);
        webview.load_html_string(
            "
            <script>
                document.body.innerHTML += 'start';
                window.webkit.messageHandlers.hello.postMessage('hello');
                document.body.innerHTML += ' -> success';
            </script>
            ",
            "",
        );

        app.set_webview(&webview);
        app.run();
    }
}
```

## Missing

- ~~Callbacks from JavaScript to Rust.~~
- TODOs in raw bindings
  * `serverTrust` in `WKWebView`
  * In `NSURLRequest`
    * Working with a Cache Policy
    * Accessing Request Components
    * Getting Header Fields
    * Controlling Request Behavior
    * Accessing the Service Type
    * Supporting Secure Coding
- ...

## Linting
```bash
cargo clippy
```

