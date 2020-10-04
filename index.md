# viper140.github.io

## 标题 1

{% for post in site.posts limit:32 %}

<article class="post">
    <div class="date">
        <span class="day">{{ post.date | date:"%d" }}</span>
        <span class="month">{{ post.date | date:"%b" }}</span>
        <span class="year">{{ post.date | date:"%Y" }}</span>
    </div>
     <header>
        <h2><a href="{{post.url}}" title="{{ post.title }}">{{ post.title }}</a></h2>
     </header>
     <div class="con">
        <p>
            {{ post.content  | | split:'<!--more-->' | first | strip_html }}
            <a class="more" href="{{ post.url }}">[继续阅读]</a>
        </p>
     </div>
</article>
{% endfor %}

## 标题 2

<div class="date">
    <span class="day">{{ post.date | date:"%d" }}</span>
    <span class="month">{{ post.date | date:"%b" }}</span>
    <span class="year">{{ post.date | date:"%Y" }}</span>
</div>
    <header>
    <h2><a href="{{post.url}}" title="{{ post.title }}">{{ post.title }}</a></h2>
    </header>
    <div class="con">
    <p>
        {{ post.content  | | split:'<!--more-->' | first | strip_html }}
        <a class="more" href="{{ post.url }}">[继续阅读]</a>
    </p>
</div>