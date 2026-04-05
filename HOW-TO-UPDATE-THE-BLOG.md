# How to Update the Blog

Everything is managed directly on GitHub.com — no coding required.

---

## 1. Writing a New Blog Post

### Step 1 — Go to the `_posts` folder
On GitHub, click into the `_posts` folder.

### Step 2 — Create a new file
Click **Add file → Create new file**.

### Step 3 — Name the file
The filename must follow this exact format:
```
YYYY-MM-DD-your-post-title.md
```
Examples:
```
2026-04-10-my-first-weaving.md
2026-05-01-color-experiments-in-wool.md
```
The date controls where the post appears in the list — most recent shows first.

### Step 4 — Write the post
Paste this template at the top of the file, then write your post below it:

```
---
layout: post
title: "Your Post Title Here"
date: 2026-04-10
cover_image: assets/images/your-photo.jpg
tags:
  - weaving
  - color
images:
  - assets/images/photo2.jpg
  - assets/images/photo3.jpg
---

Write your post here. You can use **bold**, *italic*, and regular paragraphs.

## A heading

More text under a heading.
```

**What each field means:**

| Field | Required? | What it does |
|-------|-----------|--------------|
| `title` | Yes | The post title |
| `date` | Yes | Must match the date in the filename |
| `cover_image` | No | The main photo shown on the blog list and at the top of the post |
| `tags` | No | Labels like `weaving`, `knitting` — shown as small pills on the post |
| `images` | No | Extra photos shown in a grid at the bottom of the post AND in the Gallery |

### Step 5 — Save / Commit
Scroll to the bottom of the page, click **Commit new file**. The site updates in about 1 minute.

---

## 2. Uploading Photos

### Step 1 — Go to `assets/images`
Click into the `assets` folder, then `images`.

### Step 2 — Upload your photos
Click **Add file → Upload files**, then drag and drop your photos. Click **Commit changes**.

**Photo tips:**
- Use `.jpg` for photos — **do not upload `.heic` files** (iPhone default format), they won't display in most browsers
- To convert HEIC to JPG on Mac: open in Preview → File → Export → Format: JPEG
- Or select photos in Finder → right-click → Quick Actions → Convert Image → JPEG
- Keep filenames simple, no spaces — use dashes: `my-blue-shawl.jpg`
- Aim for photos under 1–2 MB for fast loading

### Step 3 — Reference the photo in your post
Use the path `assets/images/your-photo.jpg` wherever a photo path is needed in the post front matter.

---

## 3. Adding Standalone Gallery Photos (not tied to a post)

Open the file `gallery.md`. At the top you'll see the front matter (the section between the `---` lines). Add photos like this:

```yaml
---
layout: default
title: Gallery
permalink: /gallery/
extra_images:
  - image: assets/images/my-photo.jpg
    caption: "Optional caption here"
  - image: assets/images/another-photo.jpg
    caption: "Another optional caption"
    link: /blog/2026/04/10/my-post/   ← optional: click takes user to this URL
---
```

Click **Edit** (the pencil icon), make your changes, then **Commit changes**.

---

## 4. Editing the Home Page

Open `index.md`. You can:
- Change the welcome paragraph text
- Set a hero photo: uncomment the `hero_image` line and set your photo path
- Change the `hero_subtitle`

---

## 5. Changing Site Name / Your Name / Bio

Open `_config.yml` and edit:
```yaml
title: "Your Blog Name"
author:
  name: "Your Name"
  bio: "A short bio shown in the footer."
```

---

## Quick Reference: Markdown Formatting

```
**bold text**
*italic text*
# Big heading
## Medium heading
### Small heading
- bullet item
1. numbered item

[Link text](https://example.com)
![image alt text](/assets/images/photo.jpg)
```

---

## Troubleshooting

**My post isn't showing up:**
- Check the filename matches `YYYY-MM-DD-title.md` exactly
- Check the date in the front matter matches the filename date
- Wait a minute or two — GitHub Pages takes a moment to rebuild

**My photo isn't showing:**
- Check the path is exactly right: `assets/images/filename.jpg`
- Check the filename in `assets/images/` matches what you typed (case-sensitive!)

**Something looks broken:**
- Go to your repo → **Actions** tab → look for a red ✗ to see error details
