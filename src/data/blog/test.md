---
pubDatetime: 2022-09-23T04:58:53Z
modDatetime: 2026-01-10T13:04:53.851Z
title: How to create blog on AstroPaper

slug: how-to-create-blog-on-astropaper
featured: true
draft: false
tags:
  - configuration
  - docs
description: A brief Guide on how AstroPaper handels md to html conversion.
---

## Table of contents

## Boiler Plate

``` md 
---
pubDatetime: 2022-09-23T04:58:53Z
modDatetime: 2026-01-10T13:04:53.851Z
title: How to create blog on AstroPaper

slug: how-to-create-blog-on-astropaper
featured: true
draft: false
tags:
  - configuration
  - docs
description: A brief Guide on how AstroPaper handels md to html conversion.
---

```

## Index Page

This should be a part of boiler plate only , it requires you to add a 
```md file=post.md 
## Table of contents
```
Then You can write your own post content from below it , the table of contents will automatically be filled in as long as you stick with the format. Which is All 
1. section titles must be  ## 
1. sub section titles in ###  

## Code Blocks
Follow the following Code block in your post.md
```md file=post.md
    ```js file=src/config.ts
    const logic = true + true == 2
    const nlogic = true!==1

    console.log(logic)
    console.log(nlogic)
    ```
```
It will result in following output -
```js file=src/config.ts
    const logic = true + true == 2
    const nlogic = true!==1

    console.log(logic)
    console.log(nlogic)
```
## Tables

``` md file=post.md

| Symbol | Description|
| :---  | :---|
| \| | Pipe symbol seprates two column entries within a row , treate it as ',' in a csv.|
| :--- | Should be used only once, it left alligns all content in that column.|
| ---:| ... Right aligns all content in that column|
| ---:| ... Right aligns all content in that column|
| :---: | ... Centre aligns all content in that column|
| `ctrl+A` | Represents stuff written in quotes as key , should not have spaces.|
```
| Symbol | Description|
| :---  | :---|
| \| | Pipe symbol seprates two column entries within a row , treate it as ',' in a csv.|
| :--- | Should be used only once, it left alligns all content in that column.|
| ---:| ... Right aligns all content in that column|
| ---:| ... Right aligns all content in that column|
| :---: | ... Centre aligns all content in that column|
| `ctrl+A` | Represents stuff written in quotes as key , should not have spaces.|

## Misclaneous

1. Links: 
<https://www.google.com>
[Go to Google](https://www.google.com)
```md file=post.md
<https://www.google.com>
[Go to Google](https://www.google.com)
```
2. Image:
```md file=post.md 

![AltText](imageURL)
<img src="imageURL" alt="image alt" width="300" height="200">
```
![Slugterra image](https://upload.wikimedia.org/wikipedia/en/1/14/Slugterra_logo_picture.png)
<img src="https://upload.wikimedia.org/wikipedia/en/1/14/Slugterra_logo_picture.png" alt="Slugterra image" width="300" height="200">

3. Img Links:
```md 
[![Github Logo](ImagrURL)](RedirectURL)

```
[![Github Logo](https://brand.github.com/_next/static/media/logo-03.cc5e5332.png)](https://github.com)
