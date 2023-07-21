---
title: How to set up TailwindCSS for Leptos
author: tim
date: 2023-07-20 11:33:00 +0800
categories: [non-fiction, programming]
tags: [tutorial, set-up, programming, rust, web development]
pin: true
math: true
mermaid: true
image:
  path: /img/tailwind.jpeg
  alt: AI still can't do math (yet)
---

> Ah, good taste! What a dreadful thing! Taste is the enemy of creativeness.
> 
> Pablo Picasso


## Set up Leptos

1. Add WASM as a compilation target for Rust

    ```console
    > rustup target add wasm32-unknown-unknown

    ```

   - **Optional**: It is highly recommended to uses the `nightly` version of Rust, so instead of writing 
   
   ```rust
   let (count, set_count) = create_signal(cx, 0);
   ...
   <p> {move || count.get()}</p>

   ```
   
   to show `count` reactively, we can write
   
   ```rust
   let (count, set_count) = create_signal(cx, 0);
   ...
   <p> {count} </p>
   ```
   
   bypassing the need to use a closure to show a reactive value. This is because
   the `nightly` version of Rust allows objects to implement the `Fn` trait.
   Signals are already functions in Leptos.
   
   
2. install `trunk` and `cargo-leptos`. `trunk` implements hot reloading for the
   frontend, and `cargo-leptos` for both front end and back end
   
   ```console
   > cargo install trunk cargo-leptos
   ```
   
3. set up the `leptos-rs/start` template

    ```console
    > cargo leptos new --git https://github.com/leptos-rs/start
    ```

4. Test to see if everything works. 

    ```console
    > cd my_project
    my_project> cargo leptos watch
    ```
    
## Set up Tailwind

1. Install Tailwind and create a `tailwind.config.js` file

    ```console
    my_project> npm install -D tailwindcss
    my_project> npx tailwindcss init
    ```
    
2. Modify `content` key in the `tailwind.config.js`

    ```js
    module.exports = {
    content: {
        files: ["*.html", "./src/**/*.rs"],
    },
    ...
    ```
    {: file='tailwind.config.js'}
    
3. Create an `input.css` file in the root directory

    ```css
    @tailwind base;
    @tailwind components;
    @tailwind utilities;
    ```
    {: file='input.css'}
    
4. Change the style file in `Cargo.toml`

    ```toml
    ...
    style-file = "style/output.css"
    ...
    ```

5. Run the Tailwind process in a separate terminal:

    ```
    my_project> npx tailwindcss -i ./input.css -o ./style/output.css --watch
    ```
    
Done! Add 

```rust
        <main class="my-0 mx-auto max-w-3xl text-center">
            <h2 class="p-6 text-4xl">"Welcome to Leptos with Tailwind"</h2>
            <p class="px-10 pb-10 text-left">"Tailwind will scan your Rust files for Tailwind class names and compile them into a CSS file."</p>
            <button
                class="bg-amber-600 hover:bg-sky-700 px-5 py-3 text-white rounded-lg"
                on:click=move |_| set_count.update(|count| *count += 1)
            >
                "Something's here | "
                {move || if count() == 0 {
                    "Click me!".to_string()
                } else {
                    count().to_string()
                }}
                " | Some more text"
            </button>
        </main>
```

into `App.rs` to see if it works :)
