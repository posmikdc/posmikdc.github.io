# posmikdc.github.io

Personal academic website built with plain HTML/CSS, hosted on GitHub Pages. Live at [posmikdc.github.io](https://posmikdc.github.io).

---

## Folder Structure

```
posmikdc.github.io/
├── index.html          # Main page (bio, research interests, publications)
├── blog.html           # Blog index (list of posts)
├── style.css           # All styling for index.html and blog.html
├── photo.png           # Profile photo
├── images/             # Publication card thumbnails
│   └── xyz.png
└── posts/              # Blog posts (Quarto → HTML)
    ├── _quarto.yml     # Quarto config for all posts
    ├── _nav.html       # Shared nav injected into each post
    └── xyz.qmd       
```

---

## Adding a Publication

**1.** Add a thumbnail image to `images/` (any format, will be cropped to fit).

**2.** Copy this block into the `pub-grid` div in `index.html`:

```html
<details class="pub-card">
  <summary>
    <img src="images/yourimage.jpg" alt="Paper description">
    <div class="pub-card-info">
      <p class="pub-card-title">Your Paper Title</p>
      <p class="pub-card-authors"><span class="me">D. Posmik</span>, Co-Author</p>
      <p class="pub-card-venue">Journal / Conference Year</p>
    </div>
  </summary>
  <div class="pub-card-body">
    <p class="pub-abstract-label">Abstract</p>
    <p>Your abstract here. Supports LaTeX: $\hat{\beta} = (X^TX)^{-1}X^Ty$</p>
    <a href="link-to-pdf">[PDF]</a>
  </div>
</details>
```

---

## Adding a Blog Post

**1.** Write your post as `posts/your-post-name.qmd` with this front matter:

```yaml
---
title: "Your Post Title"
date: YYYY-MM-DD
description: "One sentence summary."
---
```

**2.** Render it:

```bash
cd posts
quarto render your-post-name.qmd
cd ..
```

**3.** Add a card to `blog.html` inside the `blog-grid` div:

```html
<a class="blog-card" href="posts/your-post-name.html">
  <p class="blog-date">Month Year</p>
  <p class="blog-title">Your Post Title</p>
  <p class="blog-excerpt">One sentence summary.</p>
  <span class="blog-read">Read article →</span>
</a>
```

---

## Math

LaTeX is supported everywhere in `index.html` via KaTeX. Use `$...$` for inline and `$$...$$` for display math. In blog posts, Quarto handles math natively.

---

## Updating Your CV

The CV link points to:
```
https://github.com/posmikdc/latex/blob/main/cv/cv.pdf
```
Just push an updated `cv.pdf` to that repo and the link updates automatically.


