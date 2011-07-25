---
layout: post
title: "Post With GWT"
date: 2011-07-25 16:14
comments: true
categories: 
---

This article is about integrating GWT with Grails. Here, GWT makes request to the Grails server which then returns a JSON object back to the GWT client. Here's a way to do post using GWT.

{% codeblock %}
RequestBuilder rb = new RequestBuilder(RequestBuilder.POST,
 "http://localhost:9098/json/json/jsontest");
rb.setHeader("Content-Type",
                   "application/x-www-form-urlencoded");
try 
{
    rb.sendRequest("jsontext="+hellothere.toString(), 
                          new DateCallbackHandler());
}
catch (RequestException e) {
    Window.alert(e.toString());
}

{% endcodeblock %}


By the way, its calling a controller of a grails project. Here's how it handles it


{% codeblock %}


def jsontest = {
   String _json = (params.jsontext)?
                         params.jsontext:'{"JSON":"text"}'
   JSONObject obj = new JSONObject(_json)
   render obj.getString("JSON")
}


{% endcodeblock %}
