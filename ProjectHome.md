Shlisp is a toy lisp language implemented in pure Bash, no external commands are used.
It is not really useful for anything (if you are looking for using a real lisp language in your shell, maybe e.g. [Scsh](http://en.wikipedia.org/wiki/Scsh) is of interest to you).

Shlisp is a lisp-1. Supported types are integers, strings, lists (cons cells), symbols, lambdas, and macros. Quote and back-quote/un-quote/un-quote-at reader macros are implemented.
A naive garbage collector based on reference counting is invoked explicitly from within the REPL.

Shlisp is contained in a single file which is found [here](http://code.google.com/p/shlisp/source/browse/shlisp)

Noticeable shortcomings are e.g. lack of file io capabilities.

Since shlisp make use of the "declare -g" feature to create global associative arrays from within functions, it is dependant of Bash version 4.2 or newer.

## Simple examples ##
A few [examples](http://code.google.com/p/shlisp/source/browse/test.shlisp) are included.

### T and nil, numbers and strings evaluates to itself ###
<pre>
T     ; -> T<br>
()    ; -> nil<br>
nil   ; -> nil<br>
0     ; -> 0<br>
-1    ; -> -1<br>
"one" ; -> "one"<br>
</pre>

### Lists and strings ###
<pre>
(cons 0 (cons 1 (cons 2 ())))   ; -> (0 1 2)<br>
(list 0 1 2)                    ; -> (0 1 2)<br>
"012"                           ; -> "012"<br>
<br>
;; the length builtin can be used on lists and strings<br>
(mapcar length '(() "" (1 2 3) "some string"))  ; -> (0 0 3 11)<br>
<br>
;; reverse only works with lists<br>
(reverse '(first second third fourth fifth))    ; -> (fifth fourth third second first)<br>
</pre>

### defun/lambda/defmacro/macro ###
<pre>
((lambda () 'simple))       ; -> simple<br>
<br>
;; Dotted end in lambda/macro/defun/defmacro argument list gets bound to "rest"<br>
(defun f (head . rest)<br>
(cons head (cons (length rest) rest)))<br>
(f 'first 'second 'third)   ; -> (first 2 second third)<br>
(f 'single-argument)        ; -> (single-argument 0)<br>
<br>
;; A single symbol gets bound to all arguments<br>
(defun g all (length all))<br>
(g 1 2 3)                   ; -> 3<br>
(g)                         ; -> 0<br>
</pre>

### quote and backquote ###
<pre>
'(quote will stop evaluation)<br>
; -> (quote will stop evaluation)<br>
`(with backquote ,(+ 1 2 3 4) things ,@(list 1 2 3) can be spliced)<br>
; -> (with backquote 10 things 1 2 3 can be spliced)<br>
`(1 2 ,@3)<br>
; -> (1 2 . 3)<br>
</pre>

### setq will bind existing symbols, or create new ones in global scope ###
<pre>
(setq x 10)<br>
(let ((x 14))<br>
(setq y x)<br>
(setq x 78)<br>
x)          ; -> 78<br>
x             ; -> 10<br>
y             ; -> 14<br>
</pre>

### Simple macros can be created ###
<pre>
(defmacro double (arg)<br>
(let ((g (gensym)))<br>
`(let ((,g ,arg))<br>
(list ,g ,g))))<br>
(double 1)    ; -> (1 1)   /The 1 is evaluated only once/<br>
</pre>

### let and lambda creates lexical environments ###
<pre>
;; Mimic OO-style stuff<br>
(defun counter-class (val step-size)<br>
(let ((direction 'up)<br>
(step-size (if (> step-size 0) step-size 1)))<br>
(let ((set-step (lambda (x)<br>
(setq step-size (if (> x 0) x 1))))<br>
(get-step (lambda ()<br>
step-size))<br>
(flip (lambda ()<br>
(setq direction (if (eq direction 'up) 'down 'up))))<br>
(step (lambda ()<br>
(setq val (if (eq direction 'up) (+ val step-size) (- val step-size))))))<br>
(lambda (method . args)<br>
(cond ((eq method 'step) (step))<br>
((eq method 'flip) (flip))<br>
((eq method 'set-step) (set-step (car args)))<br>
((eq method 'get-step) (get-step))<br>
(T "No such method"))))))<br>
;<br>
(setq c1 (counter-class 0 1))<br>
(setq c2 (counter-class 0 7))<br>
(c1 'step)           ; -> 1<br>
(c1 'step)           ; -> 2<br>
(double (c2 'step))  ; -> (7 7)<br>
(double (c2 'step))  ; -> (14 14)<br>
;<br>
(c1 'set-step (* 3 (c1 'get-step)))<br>
(c1 'step)           ; -> 5<br>
(c1 'flip)<br>
(c1 'step)           ; -> 2<br>
(c1 'step)           ; -> -1<br>
</pre>
