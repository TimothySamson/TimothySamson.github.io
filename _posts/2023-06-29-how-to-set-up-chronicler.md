---
title: How to set up Chronicler™ development for the first time
author: tim
date: 2023-06-29 11:33:00 +0800
categories: [chronicler, tutorial]
tags: [tutorial, set-up, ]
pin: true
math: true
mermaid: true
image:
  path: /img/chronicler-logo.jpg
  alt: Here there be dragons
---

How to set up Chronicler for development. 

1. install [**VSCodium**](https://vscodium.com/)
   - **optional:** in the 'Extension' tabs in VSCodium, install "Svelte for VS Code" and "Tailwind CSS IntelliSense"
   - **optional, and highly recommended**: install 'vim' (look for the one with ~100k installs) extension and become superhuman
   
## Setting up VSCodium
1. go to 'Source Control' and click 'Clone Repository'
2. put in the GitHub repo: <https://github.com/TimothySamson/chronicler>

## Setting up node.js & npm
1. Install [**node.js**](https://nodejs.org/en)
2. Open your command prompt, navigate to the project root directory, and install dependencies
```console
$ cd chronicler
/chronicler $ npm install
```
3. Build the project, then preview the project
```console
/chronicler $ npm run build
/chronicler $ npm run preview
```
4. Follow instructions in the console to view the sitein your browser. It should say
```console
> chronicler@0.0.0 preview
> vite preview
 ➜  Local:   http://localhost:4173/chronicler/
 ➜  Network: use --host to expose
```
but your port number might be different.
