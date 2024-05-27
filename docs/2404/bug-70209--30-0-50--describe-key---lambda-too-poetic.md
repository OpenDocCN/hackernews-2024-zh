<!--yml
category: 未分类
date: 2024-05-27 12:59:13
-->

# bug#70209: 30.0.50; describe key + lambda too poetic

> 来源：[https://yhetil.org/emacs-bugs/874jcgbekr.fsf@gmail.com/T/#u](https://yhetil.org/emacs-bugs/874jcgbekr.fsf@gmail.com/T/#u)

```
* **bug#70209: 30.0.50; describe key + lambda too poetic**
@ 2024-04-05  4:47 No Wayman
  2024-04-05  5:07 ` No Wayman
  0 siblings, 1 reply; 8+ messages in thread
From: No Wayman @ 2024-04-05  4:47 UTC (permalink / raw)
  To: 70209

GNU Emacs 30.0.50 (build 1, x86_64-pc-linux-gnu, GTK+ Version 
3.24.41, cairo version 1.18.0) of 2024-04-02  

emacs -Q --batch \
--eval '(global-set-key (kbd "c") (lambda () (interactive) t))' \
--eval '(describe-key "c")' \
--eval '(with-current-buffer "*Help*" (print 
  (buffer-substring-no-properties (point-min) (point-max))))'

Outputs the following poetry:

"c runs the command #<closure 0FE> (found in global-map), which is 
.

It is bound to c.

(anonymous)
"

"which is..."?
What is describe-key trying to tell us?
Is it respecting the function's wish to remain anonymous?
Is it commentary on the limits of descriptive language?
I can only respond with the output of M-x describe-feelings:

"It is bound to c, yet I see a bind:
Though closure mentioned, none I've yet to find."
~ Anonymous

^ permalink raw reply	[**flat**|nested] 8+ messages in thread
```

* * *

```
* bug#70209: 30.0.50; describe key + lambda too poetic
  2024-04-05  4:47 bug#70209: 30.0.50; describe key + lambda too poetic No Wayman
@ 2024-04-05  5:07 ` No Wayman
  2024-04-05  5:43   ` Eli Zaretskii
  2024-04-05 11:44   ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
  0 siblings, 2 replies; 8+ messages in thread
From: No Wayman @ 2024-04-05  5:07 UTC (permalink / raw)
  To: 70209

[-- Attachment #1: Type: text/plain, Size: 32 bytes --]

See attached patch, which is.

[-- Warning: decoded text below may be mangled, UTF-8 assumed --]
[-- Attachment #2: 0001-Include-lambda-type-in-describe-key-output.patch --]
[-- Type: text/x-patch, Size: 903 bytes --]

From 65f0f2fa251a7b18c15698460c499394b931d09d Mon Sep 17 00:00:00 2001
From: Nicholas Vollmer <iarchivedmywholelife@gmail.com>
Date: Thu, 4 Apr 2024 23:54:42 -0400
Subject: [PATCH] Include lambda type in describe-key output

* lisp/help-fns.el (help-fns-function-description-header):
Add case to describe lambda forms (Bug#70209).
---
 lisp/help-fns.el | 2 ++
 1 file changed, 2 insertions(+)

[diff](#iZ2e.:..:87wmpc9z38.fsf::40gmail.com:2lisp:help-fns.el) --git a/lisp/help-fns.el b/lisp/help-fns.el
index a291893e9a2..3a5984d5b84 100644
--- a/lisp/help-fns.el
+++ b/lisp/help-fns.el @@ [-1102,6](../../a291893e9a2/s/?b=lisp/help-fns.el#n1102) [+1102,8](../../3a5984d5b84/s/?b=lisp/help-fns.el#n1102) @@ help-fns-function-description-header
 				elts nil))
 		      (setq elts (cdr-safe elts)))
 		    (concat beg (if is-full "keymap" "sparse keymap"))))
+                 ((eq (car-safe def) 'lambda)
+                  (concat beg "anonymous Lisp function"))
 		 (t ""))))
     (with-current-buffer standard-output
       (insert description))
-- 
2.44.0

^ permalink raw reply related	[**flat**|nested] 8+ messages in thread
```

* * *

```
* bug#70209: 30.0.50; describe key + lambda too poetic
  2024-04-05  5:07 ` No Wayman
@ 2024-04-05  5:43   ` Eli Zaretskii
  2024-04-05 11:44   ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
  1 sibling, 0 replies; 8+ messages in thread
From: Eli Zaretskii @ 2024-04-05  5:43 UTC (permalink / raw)
  To: No Wayman, Stefan Monnier; +Cc: 70209

> From: No Wayman <iarchivedmywholelife@gmail.com>
> Date: Fri, 05 Apr 2024 01:07:39 -0400
> 
> See attached patch, which is. 
Thanks.  Stefan, any comments?

^ permalink raw reply	[**flat**|nested] 8+ messages in thread
```

* * *

```
* bug#70209: 30.0.50; describe key + lambda too poetic
  2024-04-05  5:07 ` No Wayman
  2024-04-05  5:43   ` Eli Zaretskii
@ 2024-04-05 11:44   ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
  2024-04-05 20:58     ` No Wayman
  1 sibling, 1 reply; 8+ messages in thread
From: Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors @ 2024-04-05 11:44 UTC (permalink / raw)
  To: No Wayman; +Cc: 70209

Thank you for your enjoyable bug report and the suggested patch.

> @@ -1102,6 +1102,8 @@ help-fns-function-description-header
>  				elts nil))
>  		      (setq elts (cdr-safe elts)))
>  		    (concat beg (if is-full "keymap" "sparse keymap"))))
> +                 ((eq (car-safe def) 'lambda)
> +                  (concat beg "anonymous Lisp function"))
>  		 (t ""))))
>      (with-current-buffer standard-output
>        (insert description)) 
Actually, I think this won't help because in my test the `car` of `def`
is `closure` rather than `lambda`.

I installed the patch below instead.

        Stefan

diff --git a/lisp/help-fns.el b/lisp/help-fns.el
index a291893e9a2..27011575333 100644
--- a/lisp/help-fns.el
+++ b/lisp/help-fns.el @@ [-1086,13](../../a291893e9a2/s/?b=lisp/help-fns.el#n1086) [+1086,6](../../27011575333/s/?b=lisp/help-fns.el#n1086) @@ help-fns-function-description-header
 		      ;; need to check macros before functions.
 		      (macrop function))
 		  (concat beg "Lisp macro"))
-		 ((atom def)
-		  (let ((type (or (oclosure-type def) (cl-type-of def))))
-		    (concat beg (format "%s"
-		                        (make-text-button
-		                         (symbol-name type) nil
-		                         'type 'help-type
-		                         'help-args (list type))))))
 		 ((keymapp def)
 		  (let ((is-full nil)
 			(elts (cdr-safe def)))
@@ [-1102,7](../../a291893e9a2/s/?b=lisp/help-fns.el#n1102) [+1095,16](../../27011575333/s/?b=lisp/help-fns.el#n1095) @@ help-fns-function-description-header
 				elts nil))
 		      (setq elts (cdr-safe elts)))
 		    (concat beg (if is-full "keymap" "sparse keymap"))))
-		 (t "")))) +		 (t
+		  (let ((type
+		         (if (and (consp def) (symbolp (car def)))
+		             (car def)
+		           (or (oclosure-type def) (cl-type-of def)))))
+		    (concat beg (format "%s"
+		                        (make-text-button
+		                         (symbol-name type) nil
+		                         'type 'help-type
+		                         'help-args (list type)))))))))
     (with-current-buffer standard-output
       (insert description))

^ permalink raw reply related	[**flat**|nested] 8+ messages in thread
```

* * *

```
* bug#70209: 30.0.50; describe key + lambda too poetic
  2024-04-05 11:44   ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
@ 2024-04-05 20:58     ` No Wayman
  2024-04-05 22:32       ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
  0 siblings, 1 reply; 8+ messages in thread
From: No Wayman @ 2024-04-05 20:58 UTC (permalink / raw)
  To: Stefan Monnier; +Cc: 70209

Stefan Monnier <monnier@iro.umontreal.ca> writes:

> Thank you for your enjoyable bug report and the suggested patch. 
:)

> Actually, I think this won't help because in my test the `car` 
> of `def`
> is `closure` rather than `lambda`.
>
> I installed the patch below instead.
> diff --git a/lisp/help-fns.el b/lisp/help-fns.el
> index a291893e9a2..27011575333 100644 
With patch applied, thy *scratch* prepared to itch which I bemoan,
*Help* link replied, its ink ensnared, with "lambda, type 
 Unknown."[1]
Apostrophized: "Be tossed, repaired, my user-error thrown!"
With paren pride, I then declared a test which I have shown.[2]
Test now revised as thou compared. Composure hath no throne
when teary-eyed, in fear I blared, "My closure! Type Unknown?"

[1]:
emacs -Q --batch \
--eval '(global-set-key (kbd "c") `(lambda () (interactive) t)))' 
  \
--eval '(describe-key "c")' \
--eval '(with-current-buffer "*Help*"
           (forward-button 2)
           (push-button)
           (print (buffer-substring-no-properties (point-min) 
           (point-max))))'

Unknown type lambda

[2]:
emacs -Q --batch \
--eval '(global-set-key (kbd "c") (lambda () (interactive) t)))' \
--eval '(describe-key "c")' \
--eval '(with-current-buffer "*Help*"
           (forward-button 2)
           (push-button)
           (print (buffer-substring-no-properties (point-min) 
           (point-max))))'

Unknown type closure

^ permalink raw reply	[**flat**|nested] 8+ messages in thread
```

* * *

```
* bug#70209: 30.0.50; describe key + lambda too poetic
  2024-04-05 20:58     ` No Wayman
@ 2024-04-05 22:32       ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
  2024-04-06  5:36         ` No Wayman
  0 siblings, 1 reply; 8+ messages in thread
From: Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors @ 2024-04-05 22:32 UTC (permalink / raw)
  To: No Wayman; +Cc: 70209

> [1]:
> emacs -Q --batch \
> --eval '(global-set-key (kbd "c") `(lambda () (interactive) t)))'   \
> --eval '(describe-key "c")' \
> --eval '(with-current-buffer "*Help*"
>           (forward-button 2)
>           (push-button)
>           (print (buffer-substring-no-properties (point-min)
>            (point-max))))'
>
> Unknown type lambda 
That's mild punishment for quoting a lambda.

> [2]:
> emacs -Q --batch \
> --eval '(global-set-key (kbd "c") (lambda () (interactive) t)))' \
> --eval '(describe-key "c")' \
> --eval '(with-current-buffer "*Help*"
>           (forward-button 2)
>           (push-button)
>           (print (buffer-substring-no-properties (point-min)
>            (point-max))))'
>
> Unknown type closure 
OK, OK, I relent, it shouldn't be button.
Should be fixed now on `master`, thanks to the handy patch below.

        Stefan

diff --git a/lisp/help-fns.el b/lisp/help-fns.el
index 27011575333..cfe27077055 100644
--- a/lisp/help-fns.el
+++ b/lisp/help-fns.el @@ [-1096,15](../../27011575333/s/?b=lisp/help-fns.el#n1096) [+1096,15](../../cfe27077055/s/?b=lisp/help-fns.el#n1096) @@ help-fns-function-description-header
 		      (setq elts (cdr-safe elts)))
 		    (concat beg (if is-full "keymap" "sparse keymap"))))
 		 (t
-		  (let ((type
-		         (if (and (consp def) (symbolp (car def)))
-		             (car def)
-		           (or (oclosure-type def) (cl-type-of def)))))
-		    (concat beg (format "%s"
-		                        (make-text-button
-		                         (symbol-name type) nil
-		                         'type 'help-type
-		                         'help-args (list type))))))))) +		  (concat beg (format "%s"
+		                      (if (and (consp def) (symbolp (car def)))
+		                          (car def)
+		                        (let ((type (or (oclosure-type def)
+		                                        (cl-type-of def))))
+		                          (make-text-button
+		                           (symbol-name type) nil
+		                           'type 'help-type
+		                           'help-args (list type))))))))))
     (with-current-buffer standard-output
       (insert description))

^ permalink raw reply related	[**flat**|nested] 8+ messages in thread
```

* * *

```
* bug#70209: 30.0.50; describe key + lambda too poetic
  2024-04-05 22:32       ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
@ 2024-04-06  5:36         ` No Wayman
  2024-04-06 13:17           ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
  0 siblings, 1 reply; 8+ messages in thread
From: No Wayman @ 2024-04-06  5:36 UTC (permalink / raw)
  To: Stefan Monnier; +Cc: 70209

Stefan Monnier <monnier@iro.umontreal.ca> writes: 

> That's mild punishment for quoting a lambda. 
Alas, that form doth quote itself!
A parcel pulled from off the shelf
unfurls a worm somewhat shoddy 
when passed betwixt macro's body.

> OK, OK, I relent, it shouldn't be button.  Should be fixed now 
> on `master`, thanks to the handy patch below. 
And though the worm may bore the fruit
thou treat it warm! No frore repute!
Soil tilled from core to clover.
Thy patch doth bring a type of closure.

;; poem.el ends here

Thanks, Stefan

^ permalink raw reply	[**flat**|nested] 8+ messages in thread
```

* * *

```
* bug#70209: 30.0.50; describe key + lambda too poetic
  2024-04-06  5:36         ` No Wayman
@ 2024-04-06 13:17           ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
  0 siblings, 0 replies; 8+ messages in thread
From: Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors @ 2024-04-06 13:17 UTC (permalink / raw)
  To: No Wayman; +Cc: 70209-done

> Thanks, Stefan 
My pleasure, closing,

        Stefan

^ permalink raw reply	[**flat**|nested] 8+ messages in thread
```

* * *

```
end of thread, other threads:[~2024-04-06 13:17 UTC | newest]

Thread overview: 8+ messages (download: mbox.gz / follow: Atom feed)
-- links below jump to the message on this page --
2024-04-05  4:47 bug#70209: 30.0.50; describe key + lambda too poetic No Wayman
2024-04-05  5:07 ` No Wayman
2024-04-05  5:43   ` Eli Zaretskii
2024-04-05 11:44   ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
2024-04-05 20:58     ` No Wayman
2024-04-05 22:32       ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors
2024-04-06  5:36         ` No Wayman
2024-04-06 13:17           ` Stefan Monnier via Bug reports for GNU Emacs, the Swiss army knife of text editors

```

* * *

```
Code repositories for project(s) associated with this public inbox https://git.savannah.gnu.org/cgit/emacs.git

This is a public inbox, see mirroring instructions
for how to clone and mirror all data and code used for this inbox;
as well as URLs for read-only IMAP folder(s) and NNTP newsgroup(s).
```