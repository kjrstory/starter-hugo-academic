---
title: 테마테스트
date: 2023-05-05T07:22:43.471Z
draft: false
featured: false
authors:
  - admin
categories:
  - Blog
---


"Callout"이란 블로그에서 글을 작성하는 중에 특별히 강조하고 싶은 문구들을 블록으로 만들어서 강조 처리하는 것을 의미합니다.
Wowchemy의 [Callout문서](https://wowchemy.com/docs/content/writing-markdown-latex/#callouts)를 읽어보면 note와 warning만을 지원합니다.
아래는 Wowchemy에서 지원하는 Callout입니다.

{{% callout note %}}
This is an example text.
{{% /callout %}}

{{% callout warning %}}
This is an example text.
{{% /callout %}}

종류가 두 개 밖에 없다는것이 매우 아쉽습니다. 아래 그림은 Doit테마에서 Callout과 비슷하게 사용되는 admornition에  관한것입니다. 12개를 지원하는 것을 알 수 있습니다.
일단 Wowchemy에서 비슷한 시도가 있는지 검색해보았습니다. 이 [이슈](https://github.com/wowchemy/wowchemy-hugo-themes/issues/1698#issuecomment-637773325)를 읽어보면 Custom으로 추가가 가능하다고 합니다.
하지만 Custom은 Hugo 테마라면 대부분 되는것 이기 때문에 큰 장점은 아닙니다.

아쉬운대로 Cusotom하게 Callout을 추가해보았습니다. 

{{% callout newnote %}}
이것은 note 타입입니다.
{{% /callout %}}
{{% callout abstract %}}
이것은 abstarct 타입입니다.
{{% /callout %}}
{{% callout info %}}
이것은 info 타입입니다.
{{% /callout %}}
{{% callout tip %}}
이것은 tip 타입입니다.
{{% /callout %}}
{{% callout success %}}
이것은 success 타입입니다.
{{% /callout %}}
{{% callout question %}}
이것은 question 타입입니다.
{{% /callout %}}
{{% callout newwarning %}}
이것은 warning 타입입니다.
{{% /callout %}}
{{% callout failure %}}
이것은 failure 타입입니다.
{{% /callout %}}
{{% callout danger %}}
이것은 danger 타입입니다.
{{% /callout %}}
{{% callout bug %}}
이것은 bug 타입입니다.
{{% /callout %}}
{{% callout example %}}
이것은 example 타입입니다.
{{% /callout %}}
{{% callout quote %}}
이것은 quote 타입입니다.
{{% /callout %}}

{{% github_kjrstory_admonition note %}}
admonition note
{{% /github_kjrstory_admonition %}}
{{% github_kjrstory_admonition abstract %}}
admonition abstract
{{% /github_kjrstory_admonition %}}
{{% github_kjrstory_admonition info %}}
admonition info
{{% /github_kjrstory_admonitio %}}
{{% github_kjrstory_admonition tip %}}
admonition tip
{{% /github_kjrstory_admonition %}}
{{% github_kjrstory_admonition success %}}
admonition success
{{% /github_kjrstory_admonition %}}
{{% github_kjrstory_admonition question %}}
admonition question
{{% /github_kjrstory_admonition %}}
