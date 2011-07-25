---
layout: post
title: "Satori--a Web Based Teaching App"
date: 2011-07-25 16:26
comments: true
categories: 
---
I was inspired by the way "The Little Schemer" was used to teach scheme, with question on the left and answers on the right. I wondered how hard it would be to build a web app that could teach you in such an interactive way. Not so hard as I later found out, it needed less than 70 lines of code to build a prototype. Not only is the code small, but all you need to do is create a "nodes" datatype to create your own custom book! 

The application first starts with a list of nodes. Each node contains a question to ask and possible responses. Each response can either do nothing (i.e. you move on to the next node in the list), jump you to another named node or have their own child nodes that only they can visit. Each node may or may not have a label, so we can only use them when we need to jump. 

Here's a small example that tries to teach you French.


{% codeblock %}
#lang racket
(require web-server/servlet)
(require web-server/servlet-env)


(define nodes 
  '(("how do you do?" (("not bad! how about you?")) home)
    ("I'm fine thank you!!" 
     (("I don't really care!!" home)
      ("you're welcome")))
    ("Bonjour" (("Bonjour! Is that French?")) bonjour)
    ("Oui! Bonjour means 'good morning' in French"
     (("Ahh! Je comprend!")))
    ("What do you think 'bon' in bonjour means?"
     (("umm I don't know..." ("it means 'good'" (("ahh.."))))
      ("I guess it means good" ("awesome" (("thanks"))))) complicatednode)
    ("Ok now what might 'jour' mean?"
     (("night" ("not quite" (("sorry.."))))
      ("day! it means day!" ("awesome" (("thanks!"))))))
    ("too complicated" 
     (("yes!" bonjour)
      ("not really")))))

(define (start request)
  (show-nodes nodes request))
  
(define atom? 
   (lambda (x) 
     (cond 
       ((null? x) #f)
       ((pair? x)   #f)
       (else #t))))

(define (response-text r) (car r))

(define (find-node lbl)
    (local [(define (find-lbl n)
              (if (null? n) '()
                  (let ((lblname (last (car n))))
                    (cond 
                      ((and 
                        (atom? lblname) 
                        (eq? lblname lbl)) n)
                      (else (find-lbl (cdr n)))))))]
      (find-lbl nodes)))
             

(define (node-ask n) (first n))
 
(define (response-action r)
  (if (pair? (cdr r))
      (cadr r)
      '()))
 
(define (node-responses n)
  (second n))

(define (node-label n)
  (let ((lbl (last n)))
    (if (atom? lbl)
        lbl
        'empty-label)))

(define (has-label? n)
  (eq? 'emtpy-label (node-label n)))
      
(define (show-nodes n request)
  (local [(define (current-node) (car n)) 
          (define (response-generator embed/url)
            `(html (head (title "Graph Traversal example"))
                   (body
                    (p ,(node-ask (current-node)))
                    ,@(map (lambda (r)
                             `(p(a ((href ,(embed/url (new-node-location r)))) ,(response-text r))))
                           (node-responses (current-node))))))
          (define (new-node-location resp)
            (let ((action (response-action resp)))
              (cond
                ((null? action) (lambda (req) (display 'is-null) (show-nodes (cdr n) req)))
                ((atom? action) (lambda (req) (display 'is-atom) (show-nodes (find-node action) req)))
                ((pair? action) (lambda (req) (display 'is-pair) (show-nodes (cons action (cdr n)) req))))))]
    (send/suspend/dispatch response-generator)))

(serve/servlet start #:quit? #t
               #:listen-ip #f
               #:servlet-path ""
               #:servlet-regexp #rx""
               #:port 8000)

{% endcodeblock %}


While this is an initial version of the app, I have already polished it a little bit. You can check out the source [here](http://github.com/alokthapa/satori)  and is live [here](http://satori.alokt.com).

Enjoy.


