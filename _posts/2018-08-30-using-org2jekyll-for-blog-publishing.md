---
layout: post
title: using org2jekyll for blog publishing
date: 2018-08-30
categories: 
- blog
tags: 
- jekyll org emacs
author: Kai Fan
excerpt: as title
---


# switch to org2jekyll package from homegrow solution

For some time, I was using the \`autoinsert' function to prepare jekyll
post. I have the following auto-insert define in my .emacs, so
whenever I create a new markdown file in jekyll \`<sub>post</sub>\` directory it
prompt me for the jekyll header.

    (define-auto-insert
      '("\\.markdown" . "Jekyll Markdown Post")
      '("TITLE: " "---\nlayout: post\ntitle: "
        str
        "\ndate: "
        (format-time-string "%Y-%m-%d %H:%m:%S %z")
        "\ncategories: "
        ("CATEGORY: " str " ") -1
        "\n---\n" _ "\n")
      t)

It turns out the procedure is too error prone. I got into various
error (filename convention, markdown syntax, etc.) many times when
trying to publish my post.

Today I have the org2jekyll setted up, all steps need to publish a jekyll post is
now simplified to:

-   M-x org2jekyll-create-draft
-   Anwser the questions as usual for jekyll headers
-   type type type for the blog body
-   M-x org2jekyll-publish
-   git commit and push to github.io


# org2jekyll configuration

Instead of using the org2jekyll example configuration I am using the
\`org-md-pubish-to-md' function for blog publish

    '(org-publish-project-alist
       (quote
        (("default" :base-directory "~/workspace/blog/" :base-extension "org" :publishing-directory "~/workspace/fktpp.github.io/" :publishing-function org-html-publish-to-html :headline-levels 4 :section-numbers nil :with-toc nil :html-head "<link rel=\"stylesheet\" href=\"./css/style.css\" type=\"text/css\"/>" :html-preamble t :recursive t :make-index t :html-extension "html" :body-only t)
         ("post" :base-directory "~/workspace/blog/" :base-extension "org" :publishing-directory "~/workspace/fktpp.github.io/_posts" :publishing-function org-md-publish-to-md :headline-levels 4 :section-numbers nil :with-toc nil :body-only t)
         ("images" :base-directory "~/workspace/blog/img" :base-extension "jpg\\|gif\\|png" :publishing-directory "~/workspace/fktpp.github.io/img" :publishing-function org-publish-attachment :recursive t)
         ("js" :base-directory "~/workspace/blog/js" :base-extension "js" :publishing-directory "~/workspace/fktpp.github.io/js" :publishing-function org-publish-attachment :recursive t)
         ("css" :base-directory "~/workspace/blog/css" :base-extension "css\\|el" :publishing-directory "~/workspace/fktpp.github.io/css" :publishing-function org-publish-attachment :recursive t)
         ("web" :components
          ("images" "js" "css")))))

