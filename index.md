---
layout: home
---

<div class="index-content post"> <!-- 空格，代表第二个属性？ -->
    <div class="section">
        <ul class="artical-cate">
            <li class="on"><a href="/"><span>Post</span></a></li>
            <li style="text-align:center"><a href="/category"><span>Category</span></a></li>
            <li style="text-align:right"><a href="/project"><span>Project</span></a></li>
        </ul>

        <div class="cate-bar"><span id="cateBar"></span></div>

        <ul class="artical-list">
        {% for post in site.categories.post %}
            <li>
                <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
                <div class="title-desc">{{ post.description }}</div>
            </li>
        {% endfor %}
        </ul>
    </div>

    <!-- aside即旁白位置，显示了图片-->
    <div class="aside">
    </div>
</div>
