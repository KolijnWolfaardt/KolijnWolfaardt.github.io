---
layout: 		plain
permalink : 	/posts/
title:  		Posts
author:	 		Kolijn
---
<h2>All Posts</h2>

<div class="textSpace">
These are all the posts I've made:
<table>
	{% for post in site.posts %}
	<tr>
		<td>
			<a href="{{ post.url }}">{{ post.title }}</a>
		</td>
		<td>
			{{ post.date | date: "%-d %B %Y" }}
		</td>
	</tr>
	{% endfor %}
</table>
</div>
