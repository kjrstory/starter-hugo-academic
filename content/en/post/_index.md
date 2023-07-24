---
title: Posts
cms_exclude: false

# View.
#   1 = List
#   2 = Compact
#   3 = Card
#view: 3

# Optional header image (relative to `static/media/` folder).
#header:
#  caption: ''
#  image: ''

sections:
  - block: collection
    id: posts
    content:
      title: Posts
      subtitle: 
      text: 'Check out my recent blog posts below!'
      # Choose how many pages you would like to display (0 = all pages)
      count: 10
      # Filter on criteria
      filters:
        folders:
          - post
        author: ""
        category: ""
        tag: ""
        publication_type: ""
        featured_only: false
        exclude_featured: false
        exclude_future: false
        exclude_past: false
      # Choose how many pages you would like to offset by
      # Useful if you wish to show the first item in the Featured widget
      offset: 0
      # Field to sort by, such as Date or Title
      sort_by: 'Date'
      sort_ascending: false
    design:
      # Choose a listing view
      view: list
      # Choose single or dual column layout
      columns: '1'  
---
