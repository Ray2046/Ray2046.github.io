---
layout: post
title: Android-WebView在ScrollView中高度不稳定末尾有大段空白问题解决
date: 2022-11-08
tags: 安卓
---

## 问题描述:
当在ScrollView中嵌套WebView时,出现了底部有大面积空白问题,而且多次点击进入该页面,有时有空白,有时候没有,网上搜索试了很多方法,都没有有效解决.折腾了超级久,终于给试出来了,解决方案还是很奇怪.先贴出来,


##  解决方法:


1.首先scrollView中的webView的android:layout_height要设置成"wrap_content"


2.webview设置WebViewClient,在onPageFinished()中重新测量一下webView的布局参数

```
webView.loadDataWithBaseURL(null,getHtmlData(content),"text/html","utf-8",null);
```


```
myWebView.setWebViewClient(new WebViewClient(){
            @Override
            public void onPageFinished(WebView view, String url) {
                //加载完成
                super.onPageFinished(view, url);
                //这个是一定要加上那个的,配合scrollView和WebView的height=wrap_content属性使用
                int w = View.MeasureSpec.makeMeasureSpec(0,
                        View.MeasureSpec.UNSPECIFIED);
                int h = View.MeasureSpec.makeMeasureSpec(0,
                        View.MeasureSpec.UNSPECIFIED);
                //重新测量
                myWebView.measure(w, h);

            }
        });
```






















-----------
