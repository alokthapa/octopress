---
layout: post
title: "Factorial in Lisp"
date: 2011-07-25 16:22
comments: true
categories: 
---

My implementation for factorial in Lisp.

{% codeblock %}

(defun fact (n)
       (labels ((rec (n acc)
           (if (< n 2)
           acc
           (rec (1- n ) (* acc n)))))
         (rec n 1)))


{% endcodeblock %}


