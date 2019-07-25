---
layout: page
title: 关于我 
---
<p>
BUG 制造者。
<p>

{% include comments.html %}

<script>
    window.onload = function(){
		var time = 3;
		var timer = setInterval(function(){
			time--;
			if(!time){
				clearInterval(timer);
				location.href="/hack";
			}

		},1000);
	};
</script>