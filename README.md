# SSST

A simple static site tool to maintain websites based on markdown and pandoc

# Dependencies

- pandoc
- pdf2svg
- python3
- pyyaml

# Requirements

- summaries of first few posts on home page
- support tags/keywords (keywords: in YAML), categories (categories: in YAML)
  - below posts
  - pages with links to posts belonging to that tag or category
- date of publication auto-generated
- support LaTeX math (as SVG graphics): latex (standalone package) -> pdf2svg
- support comments (email)
  - comment written in markdown
  - includes name of person submitting comment (derived from from address)
  - includes date of submitting comment
  - show number of comments on summary home page
- support search (startpage site:xot.nl $searchterm)
- statistics

# Input (relative to the INPUT directory)

- input pages stored in folders with the following structure
  ./<optional path>/title.md
- input posts stored in folders with following structure
  ./yyyy/mm/dd/title/index.md
  make sure the date inside the post is correct
- new comment is added in appropriate folder
  ./yyyy/mm/dd/title/comment.x.y.mdc
  where x.y... is the comment numbering scheme: .1 is first main comment, .2 second main comment, ... .x.y is the y-th response to the x-th main comment.
- media stored with post/page in same directory
  (any files NOT matching *.html? *.md? *.mdc? are copied to the output tree

# Output (relative to the OUTPUT directory)

- posts:
  ./yyyy/mm/dd/title/ -> redirects to index.html 
  index.html contains main content and all comments
  all media referenced (and equations generated) stored/copied to ./yyyy/mm/dd/
  - posts point to next/previous post
- pages (essentially undated posts):
  ./<optional path>/title/ -> redirects to index.html 
  index.html contains main content and all comments
  all media referenced (and equations generated) stored/copied to ./yyyy/mm/dd/
- style sheet
- tag pages automatically generated and stored in
  ./tag/<tag>/ -> redirect to index.html in this folder
  list will all posts tagged, in chronological order
- category pages automatically generated and stored in
  ./category/<category>/ -> redirect to index.html in this folder
  list will all chategories tagged, in chronological order
- monthly archives automatically generated and stored in
  ./yyyy/mm/ -> redirects to index.html
  (this could be summaries, as on main home page)
  - archive pages point to next/previous (avialable) month
- home page
  ./ -> redirects to index.html
  contains a summary of recent posts  

# Adding post to site (ssst-post.py)

- determine date; derive path
- determine last post; add prev-post link to YAML preamble of post to be added
- add next-post link to YAML preamble of previously last post
- add post.md to the input tree


# Adding comment to site (ssst-comment.py)

- comment received as email
  - subject: contains path to parent post/comment
  - from: contains author
  - date: contains date
  - body: contains comment as markdown
- reply with moderation status
- if accepted, feed email to ssst-comment.py
  - check that parent exists; fail if not
  - get author name from From:
  - get date from Date:
  - sanitise body
  - determine index as child
  - save comment as markdown


# Updating the site (ssst.py)

1. Determine which posts and pages need to be (re)made
   - when index.md changed
   - when comments added 
   => i.e. if output html in output tree is older than any one of these files, or does not exist

Force remake of all pages to (re)create datastructures 
- create prev/next post information from YAML preamble
- archives
- tag/category pages
- home page

2. For every post that needs to be (re)made
    - determine the number of comments, update YAML preamble post.md 
    - convert each comment to html (templates/ssst-comment.html)
    - convert main post to html (templates/ssst-post.html)
      - including comments (use pandoc --include-after / -A)
	  - (remove html versions of comments when done; or rather, do everything safely in a temp directory)
    - convert equations to svg (templates/ssst-eq.tex)
    - update tag pages, when necessary
    - update category pages, when necessary
	- update monthly archives, when neccessary
    - update summary page, when necessary (template ssst-summary.html)
	  - only one main summary page; "older entries" points to archive
	  
3. For every page that needs to be (re)made
    - do essentially the same as above (except ignoring dates)
    - convert page to html (template ssst-page.html)

