---
title: Developing a Bookshelf Shortcode in Hugo
date: 2024-03-12T08:00:00.00Z
draft: false
featured: false
authors:
  - admin
categories:
  - Hugo
  - Blog
  
image:
  focal_point: Smart
  preview_only: false

bookshelf_item:
  - title: Dune
    cover_img: https://m.media-amazon.com/images/I/81TmnPZWb0L._SL200_.jpg
    cover_link: "https://www.amazon.com/Dune-Chronicles-Book-1/dp/0441013597"
    author: Frank Herbert
  - title: 1984
    cover_img: https://m.media-amazon.com/images/I/71rpa1-kyvL._SL200_.jpg
    cover_link: "https://www.amazon.com/1984-Signet-Classics-George-Orwell/dp/0451524934"
    author: George Orwell
  - title: To Kill a Mockingbird
    cover_img: https://m.media-amazon.com/images/I/81aY1lxk+9L._SL200_.jpg
    cover_link: "https://www.amazon.com/To-Kill-Mockingbird-Harper-Lee/dp/0060935464"
    author: Harper Lee
  - title: The Lord of the Rings
    cover_img: https://m.media-amazon.com/images/I/7125+5E40JL._SL200_.jpg
    cover_link: "https://www.amazon.com/Lord-Rings-J-R-R-Tolkien/dp/0544003411"
    author: J.R.R. Tolkien
  - title: Harry Potter and the Sorcerer's Stone
    cover_img: https://m.media-amazon.com/images/I/91wKDODkgWL._SL200_.jpg
    cover_link: "https://www.amazon.com/Harry-Potter-Sorcerers-Stone-Rowling/dp/059035342X"
    author: J.K. Rowling
  - title: Catch-22
    cover_img: https://m.media-amazon.com/images/I/61gaZF+DIHL._SL200_.jpg
    cover_link: "https://www.amazon.com/Catch-22-50th-Anniversary-Joseph-Heller/dp/1451621175"
    author: Joseph Heller
  - title: The Great Gatsby
    cover_img: https://m.media-amazon.com/images/I/81QuEGw8VPL._SL200_.jpg
    cover_link: "https://www.amazon.com/Great-Gatsby-F-Scott-Fitzgerald/dp/0743273567"
    author: F. Scott Fitzgerald
  - title: Brave New World
    cover_img: https://m.media-amazon.com/images/I/81zE42gT3xL._SL200_.jpg
    cover_link: "https://www.amazon.com/Brave-New-World-Aldous-Huxley/dp/0060850523"
    author: Aldous Huxley
---

[Hugo](https://gohugo.io) is an efficient and fast framework for creating static sites with Markdown. Writing blog posts in Markdown is simple and intuitive, allowing you to focus on the content thanks to Markdown's conciseness. However, sometimes Markdown alone is not enough to design a website exactly as you want. To overcome these limitations, Hugo supports HTML syntax and offers the [Shortcodes](https://gohugo.io/content-management/shortcodes/) feature to provide a richer user experience.

Shortcodes are a powerful feature in Hugo that allows users to easily insert various custom elements into their website without complex HTML, CSS, or JavaScript. This is very convenient when you want to insert elements like image galleries, videos, and tabs directly into your Markdown files. For example, if you want to insert YouTube, you can simply write it in your Markdown as follows:

    {{</* youtube id="w7Ft2ymGmfc" autoplay="true" */>}}


I utilized the advantages of Shortcodes to develop a new Shortcode about bookshelves. This Shortcode I developed helps display books you have read or plan to read on your website in an attractive way. There are various book service sites on the internet, but there was a desire to introduce books directly on one's blog.

Motivated by this, I referred to two different sites:

![petargyurov' site](petargyurov_bookshelf.jpg "petargyurov' site")

![vijay verma' site](vjy_bookshelf.jpg "vijay verma' site")

These sites shared their code, and it was easy to follow when you could directly use HTML. However, my goal was to implement this functionality as a Shortcode for Hugo users to use more conveniently.

While creating the necessary HTML, CSS, and JavaScript files for the Shortcode, I made an example on [CodePen](https://codepen.io/). CodePen is a great site for studying websites, with an editor that renders changes in real-time, making it very useful for development.

<iframe height="400" style="width: 100%;" scrolling="no" title="Virtual-Bookshelf" src="https://codepen.io/Jong-Rok-Kim/embed/NWEWepo?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/Jong-Rok-Kim/pen/NWEWepo">
  Virtual-Bookshelf</a> by Jong Rok Kim (<a href="https://codepen.io/Jong-Rok-Kim">@Jong-Rok-Kim</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

<iframe height="400" style="width: 100%;" scrolling="no" title="Front-Bookeshelf" src="https://codepen.io/Jong-Rok-Kim/embed/vYMKVMP?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/Jong-Rok-Kim/pen/vYMKVMP">
  Front-Bookeshelf</a> by Jong Rok Kim (<a href="https://codepen.io/Jong-Rok-Kim">@Jong-Rok-Kim</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

The implemented Shortcode can be used very simply in Hugo. By adding a few lines of code to the Front Matter of a Markdown file and placing the Shortcode at the desired location in the body, you can display a bookshelf as follows:

    {{</* bookshelf */>}}

    {{</* front-bookshelf */>}}


{{< bookshelf >}}

{{< front-bookshelf >}}

The repositories for the two Shortcodes are as follows:

🤩 [Hugo Bookshelf Shortcode](https://github.com/kjrstory/hugo-shortcode-bookshelf)

😎 [Hugo Front Bookshelf Shortcode](https://github.com/kjrstory/hugo-shortcode-front-bookshelf)


Furthermore, these Shortcodes are distributed under the MIT license, with no restrictions on use other than crediting the source.

Through this project, I also became aware of famous book service sites like [Goodreads](https://www.goodreads.com). In the future, in addition to decorating the two Shortcodes beautifully, I am considering integrating with Goodreads' API.

