{% for post in site.posts limit:32 %}

<article>
    <div>
        <!-- day -->
        <span>{{ post.date | date:"%d" }}</span>
        <!-- month -->
        <span>{{ post.date | date:"%b" }}</span>
        <!-- year -->
        <span>{{ post.date | date:"%Y" }}</span>
    </div>
     <header>
        <h2><a href="{{post.url}}" title="{{ post.title }}">{{ post.title }}</a></h2>
     </header>
     <div class="con">
        <p>
            {{ post.content  | | split:'<!--more-->' | first | strip_html }}
            <br>
            <a class="more" href="{{ post.url }}">[继续阅读]</a>
        </p>
     </div>
</article>
<br>

{% endfor %}
