<!--yml
category: 未分类
date: 2024-05-27 14:42:25
-->

# Using accesskey attribute in HTML forms and links

> 来源：[https://jkorpela.fi/forms/accesskey.html](https://jkorpela.fi/forms/accesskey.html)

# Using `accesskey` attribute in HTML forms and links

The `accesskey` attribute, aimed at making web pages more accessible e.g. to people with motoric disabilities, has proved out to be poorly designed and poorly implemented. Although it is still endorsed by some recommendations, it tends to reduce accessibility rather than help people, though there are some special situations where it might improve useability.

The original version of this document had a much more positive attitude to the `accesskey` attribute. Experience and analysis has shown, however, that the idea of author-defined shortcuts is generally not useful on the Web. Moreover, serious damage is often caused by the way in which the attribute has been implemented in browsers: it uses key combinations that override built-in functionality in browsers and other software. Ihave preserved the technical descriptions, however, since they might be of some use.

In [HTML](../html-primer.html "Getting Started with HTML; a primer by J. Korpela"), one can use the `accesskey` attribute in [forms](index.html "Annotated links to tutorials, references, and specialized documents about HTML forms") and [links](../HTML3.2/4.7.html "Discussion of links in an HTML 3.2 tutorial"). When supported by a browser, it allows the user to use keyboard keys for functions which are normally done using a mouse. For example, to follow a link, one might (in some environments) use the Alt key and a letter, as alternative to clicking on the link. This requires that the author has specified an access key assignment for that link, using the `accesskey` attribute. Unfortunately, browser support to the attribute is limited, and rather primitive. The `accesskey` attribute tends to mask out the functionality of a browser's predefined keyboard control, which is often much more important than page-specific access keys. Moreover, browsers do not indicate that access keys are available. Authors may wish to consider including explicit notes about access key assignments in their documents; this document suggests some conventions about this. It also presents reasons for using only *digit* keys as access keys.

In the W3C note [Techniques for Web Content Accessibility Guidelines 1.0](http://www.w3.org/TR/WAI-WEBCONTENT-TECHS/), section [Keyboard access](http://www.w3.org/TR/WAI-WEBCONTENT-TECHS/#keyboard), the need for access keys is motivated as follows:

> Not every user has a graphic environment with a mouse or other pointing device. Some users rely on keyboard, alternative keyboard or voice input to navigate links, activate form controls, etc. Content developers should always ensure that users may interact with a page with devices other than a pointing device. A page designed for keyboard access (in addition to mouse access) will generally be accessible to users with other input devices. What's more, designing a page for keyboard access will usually improve its overall design as well.

This can be crucial to people with motoric disabilities. Of course there are other conditions where it could be necessary or useful. To take a trivial example, if your mouse is temporarily broken, you might still wish to do some Web surfing. And in technologies which differ from "normal" PCs or terminals, such as WebTV, laptops, and handheld devices, a pointing device or function - if available at all - can be significantly more difficult to use for exact pointing than a good mouse.

Browsers generally support "tabbing" from one field to another in a form. Thus, a form could usually be filled out without using a mouse or other pointing device. No special effort (like writing extra attributes) is required from the author, except that he should put form fields into a logical order; but that should be done anyway, so it's no *extra* effort. Note that <dfn>order</dfn> here refers to the order in which the form field elements appear in the HTML source. If this does not coincide with the order in which they appear on *screen*, problems may arise; this is one reason why one should not use tables e.g. for formatting a table that way but only for [expressing the structure of a form](tables.html "How to use tables to structurize forms in HTML"). The problems caused by unnatural ordering of form fields could, in principle, be reduced by using the [`tabindex` attribute](http://www.w3.org/TR/REC-html40/interact/forms.html#adef-tabindex), but that too has limited browser support.

For a **large form** with lots of optional fields, so that the user would have to "tab" through several fields when he is ready to submit the form, an author could make things easier. He could include an **accesskey** attribute for the submit field. That way, a user could use the access key to get into that field and press enter or return key to submit the form.

The following presentation contains some explanations to the [description of the `accesskey` attribute](http://www.w3.org/TR/html4/interact/forms.html#adef-accesskey) in the [HTML 4.0 specification](http://www.w3.org/TR/html4/). (Minor editorial changes have been made to the text of the specification, in order to make this presentation require less horizontal space.)

| Quotation from the HTML 4.0 specification | Comments  |
|  
*Attribute definitions*

<samp class="adef">accesskey</samp> = [*character*](http://www.w3.org/TR/REC-html40/types.html#type-character) [[CN]](http://www.w3.org/TR/REC-html40/types.html#case-neutral)

This attribute assigns an access key to an element. An access key is a single character from the document character set. **Note.** Authors should consider the input method of the expected reader when specifying an accesskey.

 | In principle, the access key character could be any [Unicode](../chars.html#10646) character. In practice, the [support](#support "On accesskey support on browsers") is much more limited; e.g., IE 4.0 supports only the Latin letters "a" through "z" as access keys and treats upper and lower case letters as identical in this context. |
| Pressing an access key assigned to an element gives focus to the element. The action that occurs when an element receives focus depends on the element. For example, when a user activates a link defined by the [<samp class="einst">A</samp>](http://www.w3.org/TR/REC-html40/struct/links.html#edef-A) element, the user agent generally follows the link. When a user activates a radio button, the user agent changes the value of the radio button. When the user activates a text field, it allows input, etc. |  
"Pressing an access key" is apparently not to be taken literally only (see

[notes on invocation methods](#invoc)

below). The point is that the user can use the keyboard instead of a mouse (or a pointing device in general).

 |
| The following elements support the [<samp class="ainst">ACCESSKEY</samp>](http://www.w3.org/TR/REC-html40/interact/forms.html#adef-accesskey) attribute: [<samp class="einst">A</samp>](http://www.w3.org/TR/REC-html40/struct/links.html#edef-A), [<samp class="einst">AREA</samp>](http://www.w3.org/TR/REC-html40/struct/objects.html#edef-AREA), [<samp class="einst">BUTTON</samp>](http://www.w3.org/TR/REC-html40/interact/forms.html#edef-BUTTON), [<samp class="einst">INPUT</samp>](http://www.w3.org/TR/REC-html40/interact/forms.html#edef-INPUT), [<samp class="einst">LABEL</samp>](http://www.w3.org/TR/REC-html40/interact/forms.html#edef-LABEL), and [<samp class="einst">LEGEND</samp>](http://www.w3.org/TR/REC-html40/interact/forms.html#edef-LEGEND), and [<samp class="einst">TEXTAREA</samp>](http://www.w3.org/TR/REC-html40/interact/forms.html#edef-TEXTAREA). | The `A` element is related to links. The `AREA` element is used for clickable image maps (which also define links in a sense). The other elements mentioned are for form input fields. |
|  
This example assigns the access key "U" to a label associated with an [<samp class="einst">INPUT</samp>](http://www.w3.org/TR/REC-html40/interact/forms.html#edef-INPUT) control. Typing the access key gives focus to the label which in turn gives it to the associated control. The user may then enter text into the [<samp class="einst">INPUT</samp>](http://www.w3.org/TR/REC-html40/interact/forms.html#edef-INPUT) area.

```

<FORM action="..." method="post">
<P>
<LABEL for="fuser" accesskey="U">
User Name
</LABEL>
<INPUT type="text" name="user" id="fuser">
</P>
</FORM>

```

 | The user could fill out and submit the form without using the mouse at all. He might, for example, type Alt-u followed by his input text and press the enter key.Note, however, that the sketchy form presented in the example has no explicit submit field; [it would improve the accessibility to add an `INPUT TYPE="SUBMIT"` element](../HTML3.2/5.25.html#submit "Notes on form submission (by pressing enter and other means)"). |
|  
In this example, we assign an access key to a link defined by the [<samp class="einst">A</samp>](http://www.w3.org/TR/REC-html40/struct/links.html#edef-A) element. Typing this access key takes the user to another document, in this case, a table of contents.

```

<P><A accesskey="C"
rel="contents" href=
"http://someplace.com/spec/contents.html"
>Table of Contents</A>

```

 | Instead of the typical method of following a link by clicking (on the words "Table of Contents" in this case), the user could alternatively use the "C" access key (e.g. Alt-C). |
| The invocation of access keys depends on the underlying system. For instance, on machines running MS Windows, one generally has to press the "alt" key in addition to the access key. On Apple systems, one generally has to press the "cmd" key in addition to the access key. | Thus, for example, if we have `accesskey="a"`, pressing the `a` key typically has no effect or could be taken as normal input (in a form input field) or as a command to the browser; a user would typically need to type Alt-a (that is, hit the `a` key while keeping the Alt key pressed down). |
| The rendering of access keys depends on the user agent. We recommend that authors include the access key in label text or wherever the access key is to apply. User agents should render the value of an access key in such a way as to emphasize its role and to distinguish it from other characters (e.g., by underlining it). | It seems to be the intent that the author specifies a label (using the `LABEL` element) and that the browser uses that information for making it clear to the user what the access key assignments are. In practice, things don't work that way at present. See [below](#display "Displaying the access key assignments") for more discussion. |

Browser support to the `accesskey` attribute is getting wider, but it's still not universal; e.g., Netscape 4 lacks it. Since the whole idea is to provide *additional possibilities* to users, this is not so serious as it may sound. As an author, you can start using the attribute, and those users who really need those possibilities will hopefully be able to select a browser which supports them.

As some support has been added, the **quality** of implementations has become an issue. The following discussion will show that the implementations are still rather immature. Naturally, the situation keeps changing, and some of the notes below might be somewhat outdated. See also some interesting notes on implementations in section [Accessibility Attributes](http://www.blooberry.com/indexdot/html/tagpages/attributes/accessibility.htm) of Index DOT Html.

Generally, as regards to support to the features introduced in the HTML 4.0 specification, see the document [HTML 4.0 in Netscape and Explorer](http://www.webreference.com/dev/html4nsie/) (and [my margin notes](../www/4.html) to it). There has of course been progress after versions 4.0 of those browsers, but the steps are far smaller than you would expect if you took the version number game seriously.

**Internet Explorer on Windows** has to some extent supported `accesskey` since version 4.0\. On IE 4.0, if the value of that attribute is a letter of the English alphabet, then using the corresponding keyboard key together with the Alt key will focus on the element where the attribute appears. On newer versions, the support is wider; e.g., IE 5.5 supports digits as access keys, too. For details, see Microsoft's documentation of [`accesskey` on IE](http://msdn.microsoft.com/workshop/author/dhtml/reference/properties/accesskey.asp). Microsoft calls it "accelerator key" in prose, since similar functionality in Windows user interface (e.g., in command menus) was already called that way. The name reflects the idea of speeding things up, rather than accessibility. It is often faster to use keyboard keys rather than the mouse.

Note that on IE, using the access key typically *only* gives focus to the element. If the element is a link, it is not followed. The user can thus move to a link (e.g., in order to proceed by tabbing then), as separately from following it, which can be activating by hitting the Enter key when the focus is on the link. I think this is how things should be, although there's the counterargument that *mostly* people just want to follow links when they use access keys for links. But the IE implementation might be seen as differing from what the specification suggests, especially if you consider the example there that says "In this example, we assign an access key to a link defined by the `A` element. Typing this access key takes the user to another document, in this case, a table of contents."

The implementation of accesskeys on IE is not very consistent. For example, using an accesskey for a checkbox does not just focus on it but also toggles its setting. I can imagine the logic behind this, since for a link, there are several things you could do with it (e.g., open in a new window or view link properties), whereas for a checkbox, you normally just want to toggle it.

There are many other problems as regards to what using an access key really causes. For example, if a submit button has an `accesskey` attribute, should we assume that using the access key *submits* the form, instead of just *focusing* on submit button? It would be rather natural to interpret that it should indeed, if we consider what the specification says:

> Pressing an access key assigned to an element gives focus to the element. The action that occurs when an element receives focus depends on the element. For example, when a user activates a link defined by the [<samp class="einst">A</samp>](http://www.w3.org/TR/html4/struct/links.html#edef-A) element, the user agent generally follows the link. When a user activates a radio button, the user agent changes the value of the radio button. When the user activates a text field, it allows input, etc.

This seems to say implicitly that "giving focus" automagically causes some "activation", intentionally leaving it more or less open what "activation" means in each case. The specification should be clarified quite a lot as regards to the semantics. But I'm afraid there's little hope; the work is oriented towards [XForms](http://www.w3.org/MarkUp/Forms/ "XForms - The Next Generation of Web Forms") and other XML based approaches that break continuity with HTML.

Testing on Netscape 6.1, Opera 6.0, and IE 5.5, all on Windows 98, I noticed confusing differences as regards to an access key for a submit button:

*   Opera doesn't support access keys
*   IE focuses on a link when an access key is used but does not follow the link (hitting "Enter" is needed), and it submits a form if an access key for the submit button is used
*   Netscape follows a link when an access key is used.

In IE on the Mac platform, access key is performed with the control key, in combination with the key specified in the `accesskey` attribute. For links, the implementation follows the link rather than just setting focus on it.

On **Netscape 6**, the support is similar. Using an access key to select a link causes the link to be followed. And it seems that access keys are not supported for form buttons.

On Netscape, the focus must first be on the web page itself before the accesskey will work. In other words, if you just bring up Netscape and go to a page with accesskeys specified, the accesskeys will not work until you either tab into the web page or click on it with your mouse.

The `accesskey` attribute implementation typically uses the same technique (Alt key) as the browser's built-in user interface. This means that access key assignments in a document may mask out some basic functionality which users might be familiar with or, less seriously, the built-in assignments might mask out the page-specific assignments.

For example, if you specify `accesskey="f"` for some link or field, an IE user will not be able to use Alt-F in its *normal* meaning within IE (opening the File menu). Whether the gain compensates for the loss depends on the document and on the user. But the browser's own shortcut assignments could be very important to users, especially to people with disabilities, so overriding them could cause severe problems, especially when the user does not know about the situation. It would be rather distracting if you are accustomed to, say, using Alt-a to Add a page to a bookmark list, and then one day you note that on some page, it submitted a form that you hadn't quite filled out yet.

For a survey of conflicts, see the document [Accesskeys and Reserved Keystroke Combinations](http://www.wats.ca/resources/accesskeysandkeystrokes/38) at [WATS.com](http://www.wats.ca "WATS.ca - Web Accessibility Testing and Services").

However, the user can hit first the Alt key, then a normal key (say, the "a" key), to invoke IE's built-in shortcut assignment. I wonder how many users know that. Moreover, this really works so that hitting the Alt key separately activates the menu bar. Pressing IE's built-in hot key for the menu you want then opens that menu. This means that the workaround helps in some situations only.

To make things worse, it is impossible to list the "standard" Alt key assignments in browsers since the assignments depend on the *language* of the user interface. For example, in the Finnish version of IE, Alt-F is unassigned but Alt-T (from Finnish Tiedosto 'file') opens a file menu.

Moreover, access key assignments may clash with other technologies as well. For example, if you use IE in combination with [IBM Home Page Reader](http://www-3.ibm.com/able/hpr.html) and the document contains an element with `accesskey="l"`, then Alt-L will not work as an access key for the element but as Home Page Reader's command (for "links reading mode"). The keyboard key assignments are often affected by settings at the operating system level, too. The problems are illustrated in the detailed document [Guidelines for Keyboard User Interface Design](http://msdn.microsoft.com/library/?url=/library/en-us/dnacc/html/ATG_KeyboardShortcuts.asp?frame=true). And it needs to be remembered that the variation within Windows platforms is just one part of the variation in the World Wide Web.

Assuming that you are using a browser which supports `accesskey`, you can see how the following works.

The following simple form has accesskeys for all fields. You can try submitting the form; this will only result in the form data being echoed back to you.

A simple link: [**J**ukka K. Korpela](../personal.html "Personal home page of Jukka K. Korpela, IT generalist and specialist") (accesskey: J).

Another simple link, with a digit as accesskey: [Home (main page of this site)](../index.html) (accesskey: digit 1).

As mentioned [above](#asg), one cannot rely on browsers displaying the access key assignments (at present or in the near future). To clarify the situation, consider the following example given in the HTML 4.0 specification:

```

 <LABEL for="fuser" accesskey="U">
 User Name
 <INPUT type="text" name="user" id="fuser">
 </LABEL>

```

On your current browser, this looks like the following:

The specification seems to suggest that a browser should automatically indicate to the user that "U" is an accesskey for the input field, e.g. by underlining that character (presumably as it appears within the `LABEL` element). But although IE supports (from version 4.0 onwards) `accesskey`, there's no underlining in the presentation. In fact there is no evidence of real support to `LABEL`. It is also debatable whether the specification really suggests what it seems to be suggesting--that browsers should, in a case like this, check the content of the `LABEL` field and underline (any?) occurrence of the accesskey character!

Thus it seems that the **author** should take care of the underlining or other specific presentation, e.g. writing the content of `LABEL` in this case as

```

 <U>U</U>ser Name

```

Such explicit presentational suggestions may become unnecessary in the future if the browser support improves, of course. But for the time being, we need to consider how to present them.

To let users benefit from access keys, you need to take care of displaying the access key assignments.

The method used in the [examples](#ex) above--explicitly telling the access key after an element, and additionally bolding the access key letter when applicable--is somewhat naivistic but it can reasonably be expected to carry the message to those who need it. The main problem with it would probably not be the naivity but the fact that a reference to access keys might confuse people who do *not* need that information. They might have difficulties in understanding whether "accesskey" is something they need to know about. You might consider making that word a link to a suitable document about access keys. (The present document of mine can be used for the purpose, although it would probably be all too detailed for that.)

If there is a large number of access key assignments on a page or if you would otherwise like to express the assignments concisely, you could use one or several of the following alternatives:

| Sample | Explanation of the method  |
| home | Make the access key letter underlined, using the `U` element. This would correspond to widespread usage for analogous assignments in many widely used programs (e.g. Web browsers).  |
| Home  | Make the access key letter appear in UPPERCASE. This too would correspond to widespread practice |
| **h**ome | Make the access key letter **strongly emphasized**, using the `STRONG` element, or **bolded**, using the `B` element. (In this context, there isn't much practical difference between these; but in principle, a speech-based user agent can be expected to present `STRONG` in some meaningful way whereas `B` would be meaningless to it.) This would make it more prominent. You might even consider suggesting--via a stylesheet or via the deprecated `FONT` element--a presentation where the letter appears in some distinctive color (although this might easily interfere with link colors).  |

These are all based on the assumption that the access key letter appears in the context e.g. in the link text or in the explanatory text (or label) preceding an input field. Usually that can be arranged; as an author, *you* formulate those texts and you select the access key assignments.

It can hardly cause harm to use *all* of the methods mentioned to make the message clearer.

For normal submit (or reset) **buttons** in forms, underlining and bolding are impossible, since the text in a button is specified using an attribute, and attribute values cannot contain HTML markup. Thus, for example, if you have

```

<INPUT TYPE="SUBMIT" VALUE="Add this" accesskey="A">

```

then you cannot underline or bold the letter "A" in "Add this". Theoretically, you could use `INPUT TYPE="IMAGE"` and an image which contains the text "Add this" with A underlined (and/or bolded), but in practice [image buttons cause more problems than they might solve](imagebutton.html). Since the use of uppercase alone is not a very good signal--even if you otherwise use all lowercase in submit button texts--it seems to be necessary to include an explicit remark about the access key.

For *links* it seems best to present the access key letter underlined, bold face, and in upper case. For *buttons*, write the access key letter in upper case and provide an explanatory statement about the access key.

This is illustrated with the following simple examples:

| HTML markup | Presentation on your current browser |
|  
```

<A HREF="index.html" accesskey="C"
>table of <STRONG><U>C</U></STRONG>ontent</A>

```

 | [table of **C**ontent](index.html) |
|  
```

<INPUT TYPE="SUBMIT" NAME="action" VALUE="Add this"
accesskey="A"> (accesskey:&nbsp;A)

```

 |  |

It might be desirable to have some common conventions on access key assignments, e.g. so that some specific access key would always take you upwards in a document hierarchy. There seems be little hope of uniformity in this area. Quite often the access keys *need* to be application-dependent; for example, when you have a form with lots of input fields, you might have to use all characters a - z for them. Moreover, access key assignments in a document might interfere with *browser "standards"*, if the browser implements `accesskey` similarly to access keys in its own built-in user interface (as [several browsers do](#warn)).

But should you like to adopt some *site-wide* "standard", here is one possibility:

Suggested "standard" access key assignments

| key | Meaning (and corresponding [`REL` value](../HTML3.2/4.7.html#relrev) if applicable) |
| A | **a**uthor's home page |
| B | **b**eginning of a series of documents to which the current document belongs (`REL="Start"`) |
| C | table of **c**ontents for current document (`REL="Contents"`) |
| G | **g**lossary for current document (`REL="Glossary"`) |
| H | **h**elp for using current document (`REL="Help"`) |
| I | (keyword) **i**ndex for current document (`REL="Index"`) |
| M | **m**ail to author (for a `mailto:` link with the author's address) |
| N | **n**ext document in a series of documents to which the current document belongs (`REL="Next"`) |
| P | **p**revious document in a series of documents to which the current document belongs (`REL="Prev"`) |
| R | **r**eset a form |
| S | **s**ubmit a form |
| U | **u**pwards in a hierarchy--typically, this leads to a page describing (e.g. as a list of links) a collection of documents to which the current document belongs |

That would still leave several letters unassigned, so they could be used for form fields and for essential links which are not covered in the general scheme.

The [WebAIM](http://www.webaim.org/) (Web Accessibility In Mind) site uses the following assignments:

WebAIM access keys

| key | Meaning |
| 1 | home page (i.e., main page of site) |
| 2 | skip navigation (i.e., location after navigational links at the start of page) |
| 3 | printer version |
| 4 | index/search |

Paul Bohman has explained this in his [message](http://www.webaim.org/discussion/mail_message.php?id=800) on the [WebAIM Forum mailing list](http://www.webaim.org/discussion/):

> You'll notice that we used numbers rather than letters. I would have preferred to use letters, but, unfortunately, when you use letters, there is a much greater likelihood that you will interfere with pre-existing keyboard shortcuts in either the browser or the assistive technologies (e.g. JAWS, Home Page Reader).

On the other hand, even digit keys are used as built-in access keys by several programs. However, this typically means that in the case of conflict, the built-in assignment is in use.

In the United Kingdom, the [Guidelines for UK Government websites](http://www.cabinetoffice.gov.uk/e-government/resources/handbook/introduction.asp) specify, in section [2.4.4UK Government accesskeys standard](http://www.cabinetoffice.gov.uk/e-government/resources/handbook/html/2-4.asp#2.4.4), recommends the following assignments:

| S | skip navigation |
| 1 | home page |
| 2 | what's new page |
| 3 | site map |
| 4 | search facility on the site |
| 5 | frequently asked questions (FAQ) |
| 6 | help page/facility |
| 7 | complaints procedure |
| 8 | terms and conditions (including privacy statement) |
| 9 | feedback page |
| 0 | information about of accesskeys |

In practice, UK government sites contain other accesskeys as well. For example the [CabinetOffice](http://www.cabinetoffice.gov.uk) site also assigns C, I, J, K, L, M, N, O, P, Q, R, U, X, Y, and Z. In practice this guarantees that there are many conflicts with built-in keyboard shortcuts in browsers and other software.

The UK recommendation adds:

> When this navigational system is made available, it is important to inform your website users, as soon as they enter. Otherwise, users who are least able to do so will be faced with a mouse-dependent navigational system that could have been bypassed. Each page could display a message, e.g. ‘UKgovernment accesskeys system’

This means that all users will receive an announcement about technicalities, breaking a fundamental design principle. Moreover, the announcement is apparently meant to be prominent. This means causing disturbing all users in order to potentially help a few.

Jesper Tverskov has made an interesting suggestion: [Use first letter as accesskey](http://www.klapmusen.dk/artikel.aspx?xml=20021031&lg=en). Systematically using the first letter of a link text as the accesskey could be a simple, easy to understand and easy to remember scheme.

The *need* for access keys (in some sense) is obvious. There are many reasons why pages should be made useable using keyboard only, without a mouse, and in a convenient way. But the `accesskey` attributes don't seem to help in solving the problem very well. They might be useful, if a site-wide system of access keys makes a site more easily navigable. If you use them, the assignments should be described separately, not in `title` attributes or relying on underlining or other small hints. And I would recommend using just *digits* for access keys, with digit 0 acting as access key to a document that describes the other assignments.

* * *