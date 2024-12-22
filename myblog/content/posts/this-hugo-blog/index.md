+++
date = '2024-11-23T23:00:55+11:00'
title = 'How did I build this blog ?'
toc = true
+++

## Inspiration
I had always wanted to build my own website, no Wordpress or any third-party hosting bloats that rip your pocket! This is also for storing information and experiences I gained throughout my learning journey in the world of tech and engineering so I don't have to crawling through my dusting physical A4 notebooks or proprietary gg docs :/

## Set Up HUGO
I built this website with Hugo and Github Page, a cool fast minimalist combo for hosting with no databases, just raw mark-down texts and basic html, css, js skills :)

1. Download [Hugo](https://gohugo.io/getting-started/quick-start/)

2. Set up 2 new Github repositories: "USERNAME.github.io" and "blog" 
   
   (*blog* is where I keep all the drafts and themes so I could modify locally, *USERNAME.github.io* is a github page for publishing the real website in the "public" folder of hugo workspace). 

3. Clone your *blog* repository to your local machine `git clone https://github.com/USERNAME/blog`.

4. `cd blog` and create a hugo workspace `hugo new site myblog`.

5. `cd myblog`
   `git clone https://github.com/hugo-sid/hugo-blog-awesome.git themes/hugo-blog-awesome` ([hugo-blog-awesome](https://themes.gohugo.io/themes/hugo-blog-awesome/))

6. Create a git submodule of public folder, link it to the publishing page *USERNAME.github.io*:
   `git submodule add -b main https://github.com/USERNAME/USERNAME.github.io.git public/` ([Refs](https://www.youtube.com/watch?v=LIFvgrRxdt4&list=WL&index=23))
   
   *NOTE: if error, delete the folder *public* !

7. Now you can customise any html, css, js files you see in *themes* folder and preview with `hugo serve`

8. You can publish new posts with `hugo new content content/posts/my-first-post.md`.

8. Once you satisfy with everything simply publish the website by typing `hugo`, remember to delete everything in *public* folder everytime you publish to avoid any collision cause `hugo` will generate a new website in *public* folder everytime.

9. Add, Commit and Push to Gihub repos and enjoy your page at *USERNAME.github.io*. 

---
## How To ?
### 1. Link Images 

#### Step 1: Organise Your Image Files
1. Navigate to the `static` directory in your Hugo project:
   ```bash
   cd <your-hugo-project>/static
   ```

2. Create a folder to store your images (e.g., `images`):
   ```bash
   mkdir images
   ```

3. Add your image file(s) to the newly created folder:
   ```bash
   cp /path/to/your/image.jpg <your-hugo-project>/static/images/
   ```

#### Step 2: Link the Image in Your Post
1. Open your post file in the `content` directory (e.g., `content/posts/example.md`):
   ```bash
   nano content/posts/example.md
   ```

2. Use the Markdown syntax to add the image:
   ```markdown
   ![Alt text describing the image](/images/image.jpg)
   ```
   - Replace `Alt text describing the image` with a short description of the image.
   - Adjust the path (`/images/image.jpg`) to match your file location.


#### Step 3: Resize or Align Images (Optional)
##### Resize Images
Use the `width` and `height` attributes in HTML within your Markdown file:
```markdown
<img src="/images/image.jpg" alt="Alt text" width="600" height="400">
```

##### Centre Align Images
Wrap the image in a `<div>` tag with a `style` attribute:
```markdown
<div style="text-align: center;">
  <img src="/images/image.jpg" alt="Alt text" width="600">
</div>
```

#### Step 4: Test Your Post
1. Start the Hugo development server:
   ```bash
   hugo server
   ```

2. Open your browser and navigate to `http://localhost:1313` to verify that the image appears correctly.


#### Step 5: Deploy Your Changes
If the image is displayed as expected, deploy your Hugo site to make it live:
```bash
hugo
scp -r public/ user@your-server:/path/to/site
```

Now your post is complete with the linked image properly displayed.

---
## References
- For more information regarding the theme, click [here](https://hba.sid.one/posts/)
