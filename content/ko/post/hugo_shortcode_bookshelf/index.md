---
title: Hugo에서 책장 Shortcode 개발
date: 2024-03-12T08:00:00.00Z
draft: false
featured: false
authors:
  - admin
categories:
  - Hugo
  - Blog

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


[Hugo](https://gohugo.io)는 마크다운으로 정적 사이트를 만드는 효율적이고 빠른 프레임워크입니다. 
마크다운을 사용하여 블로그 글을 작성하는 것은 간단하고 직관적이며, 마크다운의 간결함 덕분에 콘텐츠에 집중할 수 있습니다.
하지만 때로는 마크다운만으로는 웹사이트를 원하는 대로 멋지게 꾸밀 수 없는 경우가 있습니다.
이러한 한계를 극복하기 위해 Hugo는 HTML 문법을 지원하며, 더욱 풍부한 사용자 경험을 제공하기 위한 [Shortcodes](https://gohugo.io/content-management/shortcodes/) 기능을 제공합니다.

Shortcodes는 Hugo에서 제공하는 강력한 기능으로, 사용자가 웹사이트에 복잡한 HTML, CSS, 혹은 JavaScript 없이도 다양한 커스텀 요소를 쉽게 삽입할 수 있게 해줍니다. 
예를 들어, 이미지 갤러리, 비디오, 탭 등을 마크다운 파일에 직접 삽입하고 싶을 때 Shortcode를 사용하면 매우 편리합니다.
유튜브를 삽입하고 싶으면 마크다운에서 다음과 같이 작성하면 됩니다.
```markdown
{{< youtube id="w7Ft2ymGmfc" autoplay="true" >}}
```

저는 이러한 Shortcode의 장점을 활용하여, 책장(Bookshelf)에 관한 새로운 Shortcode를 개발했습니다.
제가 개발한 이 Shortcode는 사용자가 읽은 책이나 앞으로 읽을 책들을 웹사이트에서 보기 좋게 디스플레이 할 수 있도록 도와줍니다.
인터넷에는 다양한 도서 서비스 사이트가 있지만, 개인의 블로그에서 직접 책들을 소개하고 싶은 욕구가 있었습니다.

이러한 동기에서 출발하여, 저는 두 개의 다른 사이트를 참고하였습니다.

![](petargyurov_bookshelf.jpg)

![](vjy_bookshelf.jpg)

이들 사이트는 자신들의 코드를 공유하고 있었으며, HTML을 직접 사용할 수 있는 경우에는 쉽게 따라 할 수 있었습니다.
그러나 Hugo 사용자들이 보다 편리하게 이 기능을 사용할 수 있도록 Shortcode로 구현하는 것을 목표로 했습니다.

Shortcode를 구현하기 위해 필요한 HTML, CSS, JavaScript 파일들을 만들면서, 저는 코드펜(CodePen)에 예제를 만들었습니다.
코드펜은 웹사이트를 공부하기 매우 좋은 사이트로 웹사이트안에서 에디터가 있어 코드를 변경할 때 마다 실시간으로 렌더링되므로 개발할 때 매우 용이합니다.

<iframe height="300" style="width: 100%;" scrolling="no" title="Virtual-Bookshelf" src="https://codepen.io/Jong-Rok-Kim/embed/NWEWepo?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/Jong-Rok-Kim/pen/NWEWepo">
  Virtual-Bookshelf</a> by Jong Rok Kim (<a href="https://codepen.io/Jong-Rok-Kim">@Jong-Rok-Kim</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>


<iframe height="300" style="width: 100%;" scrolling="no" title="Front-Bookeshelf" src="https://codepen.io/Jong-Rok-Kim/embed/vYMKVMP?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/Jong-Rok-Kim/pen/vYMKVMP">
  Front-Bookeshelf</a> by Jong Rok Kim (<a href="https://codepen.io/Jong-Rok-Kim">@Jong-Rok-Kim</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

구현한 Shortcode는 Hugo에서 매우 간단하게 사용할 수 있습니다. 마크다운 파일의 Front Matter에 몇 줄의 코드를 추가하고 본문에 원하는 위치에  {{< bookshelf >}}와 {{< front-bookshelf >}}을 사용하면 책장을 디스플레이 할 수 있게 됩니다.

또한, 이러한 Shortcode들은 MIT 라이선스 하에 배포되어 있으며, 출처를 밝히는 것 외에는 사용에 제한이 없습니다. 

이 프로젝트를 진행하며, [Goodreads](https://www.goodreads.com)와 같은 유명 도서 서비스 사이트의 존재도 알게 되었습니다. 
미래에 위의 두 Shortcodes를 멋있게 꾸미는 일외에도 goodreads의 API를 이용한 연동도 고려하고 있습니다.


