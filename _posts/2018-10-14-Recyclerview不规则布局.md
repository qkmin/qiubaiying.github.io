---
layout:     post
title:      Recyclerview不规则布局
subtitle:   
date:       2018-10-14
author:     qkmin
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - Recyclerview
---
# 需求

实现一行一个item或多个item

# 实现

```
GridLayoutManager manager = new GridLayoutManager(this, 6);
manager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
			@Override
			public int getSpanSize(int position) {
				//修改return 来改变item占几个
			}
		});
```

