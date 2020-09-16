# Workspace for evangelism content

This repository will serve as a workspace to develop and collect unpublished web content until we know what our CMS will eventually be. For the time being we will write content in Mardown since that format is easy to convert to anything. For now, don't focus on looks & layout - let's focus on content. 

# What you need on your machine

- git client
- text editor (vi, nano, emacs, notepad, etc.) or IDE (VSCode, Atom, Sublime). My recommendation is [Visual Studio Code](https://code.visualstudio.com), since it is available on any platform and lends itself well to both basic and advanced usage. 
- Local install of [Hugo](https://gohugo.io) if you want to render web content locally (see below). 
- Images should be created with [draw.io](https://app.diagrams.net) preferably (see below as well). 

# What goes where?

The two subdirectories you need to care about are 
- `content` - here's where your blog posts go. Please create one file for each article. 
- `static/img/YOUR_BLOG_NAME` - here's where your images go. Please create a subdirectory for each blog post so that we don;t end up with a huge heap of images. 

# Basic Workflow

1. Clone repo to your machine

    git clone git@github.com:SUSE-Developer-Community/web-content.git

2. Create topic branch (please do not commit to the main branch directly)

    git branch my-topic-branch

3. Switch to your new topic branch 

    git checkout my-topic-branch

4. Edit your files
5. Commit to topic branch

    git add my-blog.md
    git commit -m "I wrote a cool blog"
    git push

6. Create a pull request when ready to merge to main

# Rendering content locally

Change to your local repo clone's root directory, then run

    hugo serve -D

in a local shell. 

# Other useful resources

- [Basic Markdown syntax guide](https://www.markdownguide.org/basic-syntax/)
