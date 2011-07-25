---
layout: post
title: "Functional State Management"
date: 2011-07-25 16:32
comments: true
categories: 
---

Most games need to manage the current state of a game, be it chess positions or number of cards played or the size of the pot. One could always use an global variable like turn and then set the variable to the next player. However, another approach could be to use functional programing idioms and let the state flow from one player to the next without having to create any such variable.

Suppose there is a list of players, then we could recursively work on the list and get to the next player.

For example:

{% codeblock %}

(define (play players)
    (if  (null? players)	 
     'end
     (begin  
      (do-sth (car players))
      (play (cdr players)))))

{% endcodeblock %}

Here we recurse on the list players and call do-sth on the first player in the list.

Here we have completely removed the use for a turn variable. However, in this simplistic example there is no  state management. In the game of flush, there are a several of variables we need to manage, the current size of the pot, the current minimum allowable bet amount and the amount of money each player has and info on whether a player has passed or not.

Lets add the first two variable into our example. Also we change do-sth to play? which looks at the cards of the current player and decides whether it wants to play or not.

{% codeblock %}

(define (play players pot minbet)
    (if (null? players) 
	'end
	(if (play? (car players))
	    (play (cdr players) (+ pot minibet) minibet)
	    (play (cdr players) pot minibet))))

{% endcodeblock %}

Now, when a player plays, the size of the pot increases and this information is passed on to the next player. (Note that we don’t have the functionality right now to increase the minibet yet.)

But a coulple of information are missing like the amount of money player1 has and the pass state of a player. If a player has already passed, it will simply pass all the information to the next player.

{% codeblock %}

(define (play players pot minibet ms ps)
    (if (null? players) 
	'end
	(if (and (car ps) (play? (car players) minibet pot (car ms)))
	    (play (cdr players) (+ pot minibet) minibet (cdr ms)  (cdr ps))
	    (play (cdr players) pot minibet (cdr ms) (cdr ps)))))

{% endcodeblock %}

Ok, now we’ve made a couple of changes. First we added the list ms (money ) and ps (pass status) for each player. The (car ps) is set to true if a player is still playing, so we’ve also added that in the if statement. Also we’ve added more parameters to the play? function since you need the information on how big the pot is, how much money you have and what the minimum bet is to make your decision.

Still a couple of things are missing here. First when you run out of players, it just stops. We’ll worry about that later. Another thing that’s missing is the new state of money and the pass state. While we have information on the amount of money you have right now, we have no way to record the amount of money you have after you decide to play, Also, if you decide not to play this turn, there is no way to record that info.

Lets digress here to talk about some of the idioms of functional programming. Here’s a common way to recursively calculate length of a list.

{% codeblock %}

(define (length lst)
    (if (null? lst) 
	0 
	(+ 1 (length (cdr lst)))))

{% endcodeblock %}

The problem with this is that scheme has to keep track of the recursion and what is left to do. For eg. if you calculate length of a list (1 2 3) it recurs three times, and comes back up three times adding 1 for each recursive call. This problem is usually solved using accumulators.

{% codeblock %}

(define (length-helper lst value)
    (if (null? lst) 
	value
	(length-helper (cdr lst) (+ value 1))))

(define (length lst)
    (length-helper lst 0))

{% endcodeblock %}
           
You can see here that length calls length-helper with an initial value of 0. The length-helper recurs till it exhausts itself of the list and returns the variable's value. One advantage of this approach is that there is no work left to do. In fact, scheme implementations are guaranteed to optimize the use of accumulators so that you get the speed of an iterative approach even though you’re actually recursing through a list.

Ok, so back to our problem. Well, the solution is simple, you just add new accumulators for the new values of money and pass state.

{% codeblock %}

(define (play players pot minibet ms ps new-ms new-ps )
    (if (null? players) 
	'end
	 (if (and (car ps) (play? (car players) minibet pot (car ms)))
	     (play (cdr players) 
		   (+ pot minibet)
		   minibet
		   (cdr ms)  
		   (cdr ps)
		   (cons (- (car ms) minibet)  new-ms)
		   (cons #t new-ps))
	     (play (cdr players) 
		   pot 
		   minibet 
		   (cdr ms) 
		   (cdr ps)
		   (cons (car ms) new-ms)
		   (cons #f new-ps)))))

{% endcodeblock %}


So now we are building up a new money and pass state list while consuming the previous version. Of course, the lists are being built in opposite order so you would need to reverse each list when you start a new round.


{% codeblock %}

(define orig-players (list of players))
(define orig-money (list of money))
(define orig-pass (list of pass))

(define (play players pot minibet ms ps new-ms new-ps )
    (if (null? players)
	(play orig-players pot minibet (reverse new-ms) (reverse new-ps) '() '())
	(if (and (car ps) (play? (car players) minibet pot (car ms)))
	    (play (cdr players) 
		  (+ pot minibet)
		  minibet
		  (cdr ms)  
		  (cdr ps)
		  (cons (- (car ms) minibet)  new-ms)
		  (cons #t new-ps))
	    (play (cdr players) 
		  pot 
		  minibet 
		  (cdr ms) 
		  (cdr ps)
		  (cons (car ms) new-ms)
		  (cons #f new-ps)))))

(define (game)
    (play orig-players 0 20 orig-money orig-pass '() '()))

{% endcodeblock %}

So there you go, we’ve managed state in a flush game without using one set! or using a global variable. Of course, this program above goes into an infinite loop but we’ll fix that later.....

To recap what we’ve accomplished here is 

* We never had to keep track of a turn variable. 
* We managed to keep track of the current pot and minimum bet by passing it as a functional argument to the next player.
* We used accumulators to generate new state for money and pass state while consuming an older version.
* No use of set! or mutations at all!!

I hope you enjoyed this article and feel free to [fork it](http://github.com/alokthapa/flush). 




