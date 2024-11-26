+++
date = '2024-11-23T23:00:55+11:00'
draft = false
title = 'How did I build this blog ?'
+++

## Inspiration
I had always wanted to build my own website, no Wordpress or any third-party hosting bloats that rip your pocket! This is also for storing information and experiences I gained throughout my learning journey in the world of tech and engineering so I don't have to crawling through my dusting physical A4 notebooks or proprietary gg docs :/

## How?
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

8. Once you satisfy with everything simply publish the website by typing `hugo`, remember to delete everything in *public* folder everytime you publish to avoid any collision cause `hugo` will generate a new website in *public* folder everytime.

9. Add, Commit and Push to Gihub repos and enjoy your page at *USERNAME.github.io*. 


