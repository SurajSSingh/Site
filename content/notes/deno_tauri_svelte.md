+++
title = "Deno+Tauri+Svelte: How I got things setup"
date = 2023-01-01
updated = 2023-01-16

[taxonomies]
programming-language = ["shell"]

[extra]
allow_comments = true
+++

This is a note to setup Deno and Tauri using Rust's Cargo package manager and inside Svelte through Deno using NPM compatibility.  
<!-- more -->
## Installations

### Deno
1. Install deno via cargo: `cargo install deno --locked`
    * `--locked` might be optional, though recommended
2. Check deno installed with `deno --version`

### Tauri
1. Install create-tauri-app using cargo: `cargo install create-tauri-app`
2. Install tauri-cli: `cargo install tauri-cli`
3. Use either:
    1. `cargo tauri init` on existing repository to add Tauri
    2. `cargo create-tauri-app` and setup a vanilla project; need to manually add Deno + Svelte

## Setup

### Svelte with Deno
1. `deno run --allow-read --allow-write --allow-env npm:create-vite-extra`
    1. Notes on flag:
        * `-allow-read`: Read files with `Deno.cwd()` and `Deno.readDirSync()`
        * `-allow-write`: Write to current repository with `Deno.mkdirSync()`
        * `-allow-env`: Allow access to "FORCE_COLOR", "NODE_DISABLE_COLORS", "TERM" env variables
    2. Select project name
    3. Select template (e.g., deno-svelte)
2. change directory to project
3. Test development build: `deno task dev`
    * Make note of where it is locally served (e.g., `http://localhost:5173/`), to be used when initializing Tauri

### Initialize Tauri
1. Go into project if not already there.
2. Run `cargo tauri init`
    1. Provide app name (bundle name of app)
    2. Provide window title (title that appears on window bar)
    3. Provide relative location of web assets folder (e.g., `../build` or `../dist`)
        * Try running `deno task build` to see what folder is built. That will be what is used
    4. Provide dev server (locally served url)
    5. Provide frontend dev command: `deno task dev`
    5. Provide frontend build command: `deno task build`
3. Test with `cargo tauri dev`

## Additional File Changes
1. (Optional) .vscode/extensions.json:
```json
{
  "recommendations": ["svelte.svelte-vscode", "tauri-apps.tauri-vscode", "rust-lang.rust-analyzer"]
}
```