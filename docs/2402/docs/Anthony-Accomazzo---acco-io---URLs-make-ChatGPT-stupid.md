<!--yml
category: 未分类
date: 2024-05-29 13:26:00
-->

# Anthony Accomazzo @ acco.io | URLs make ChatGPT stupid

> 来源：[https://www.acco.io/urls-make-chatgpt-stupid](https://www.acco.io/urls-make-chatgpt-stupid)

Ran into something funny today. I asked ChatGPT (v4) to read a particular web page and extract the data into a simple markdown structure:

Its response was uncharacteristically oblivious to my request:

Notably, it clearly did not access the website – there was no moment that it switched into a “Browsing...” state.

My theory was that I had triggered some sort of cache. I linked it to a URL it's probably seen before. I’m assuming web browsing is expensive and therefore something it tries to avoid.

What’s funny is that it clearly didn’t cache *the page*, but instead cached its response to the page (way cheaper!)

Dropping a URL in does not automatically shut its faculties off:

It seems like the kill switch is when I ask it to perform an operation on the URL!

It *really* doesn’t want to access the page. When I tell it to check the page again, it clearly doesn’t access the page (the word “checkbox” does indeed appear on the page):

However, it did cheekily name our conversation “Crear Objectos Personalizados”, which is just *delightful*:

To confirm my theory, I tried something else: I saved the webpage as a PDF, then uploaded the PDF to ChatGPT. That worked just fine: