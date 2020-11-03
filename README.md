# Workspace for evangelism content

This repository will serve as a workspace to develop and collect unpublished web content until we know what our CMS will eventually be. For the time being we will write content in [Markdown](https://www.markdownguide.org/basic-syntax/) since that format is easy to convert to anything. For now, don't focus on looks & layout - let's focus on content. 

# What you need on your machine

- Git client (dowload from e.g. [https://git-scm.com/downloads](https://git-scm.com/downloads))
- Text editor (vi, nano, emacs, notepad, etc.) or IDE (VSCode, Atom, Sublime). My recommendation is [Visual Studio Code](https://code.visualstudio.com), since it is available on any platform and lends itself well to both basic and advanced usage. 
- If you want to render web content locally, install [Hugo](https://gohugo.io) (see below for more info). 
- Images should be created with [draw.io](https://app.diagrams.net) preferably. 

# What goes where?

The two subdirectories you need to care about are 
- `content` - here's where your blog posts go. Please create one file for each article. 
- `static/img/YOUR_BLOG_NAME` - here's where your images go. Please create a subdirectory for each blog post so that we don't end up with a huge heap of images. 

# Basic Workflow

1. Clone repo to your machine

    `git clone git@github.com:SUSE-Developer-Community/web-content.git`

2. Create topic branch (please do not commit to the main branch directly)

    `git branch my-topic-branch`

3. Switch to your new topic branch 

    `git checkout my-topic-branch`

4. Edit your files
5. Commit to topic branch for the first time

    `git add my-blog.md`

    `git commit -m "I wrote a cool blog"`
    
    `git push --set-upstream origin my-topic-branch`

6. For consecutive commits it is sufficient to do 

    `git add my-blog.md`
    
    `git commit -m "I made a change to my cool blog"`
   
    `git push`

7. Create a pull request when ready for review. If you don't know what this is, check out [this video](https://youtu.be/For9VtrQx58) and if you're looking for instructions how to create a pull request on Github's web GUI, check [this video](https://youtu.be/rgbCcBNZcdQ). 

# Rendering content locally

Change to your local repo clone's root directory, then run

    `hugo serve -D`

in a local shell. You should now be able to access your content at [http://localhost:1313](http://localhost:1313). 

# Other useful resources

- [Basic Markdown syntax guide](https://www.markdownguide.org/basic-syntax/)
