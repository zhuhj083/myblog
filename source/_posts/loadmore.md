---
title: 到底部上滑加载更多
date: 2018-06-21 20:51:29
tags: [js loadmore]
categories: js
---
# 到底部，上滑加载更多
```javascript
var loading = false ;

$(window).on('scroll',function(){
    if(scrollTop() + windowHeight() >= documentHeight()){
       loadMore();
	   }
});

//上滑加载
$(window).bind("scroll", $.debounce(200, function() {
		loadMore()
}));

function loadMore() {
	if (document.body.scrollTop + $(window).height() > $(document).height() - 200) {
		if (isloading)
			return;
		isloading = true;
   //loadMore的逻辑
		}
	}

```
