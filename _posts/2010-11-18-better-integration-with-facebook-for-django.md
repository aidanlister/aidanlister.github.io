---
layout: post
title: Better integration with Facebook for Django
section: developer
summary: |
    I have released <a href="https://github.com/aidanlister/django-facebook">django-facebook</a> which offers total and seamless integration with Facebook for your social apps.
---
I've really been enjoying working with <a href="http://djangoproject.com/">Django</a> for the last few months, it's an incredibly powerful and elegant framework and I sorely wish there was an equivalent in PHP.

This said, I was sadly unimpressed with the social landscape support. Facebook's own <a href="">Python SDK</a> has a slew of bugs and is not actively maintained, I recommend people use <a href="https://github.com/dziegler/python-sdk">David Ziegler's fork</a>.

For integration with Django, there was <a href="https://github.com/tschellenbach/Django-facebook">Django-facebook</a> which had good intentions but doesn't quite cut the mustard, <a href="https://github.com/sciyoshi/pyfacebook/">Pyfacebook</a> which is no longer actively maintained and does not work with the new Graph API, and <a href="https://github.com/flashingpumpkin/django-socialregistration">Social registration</a> which was the best of the bunch, but still offered poor integration with the django authentication system.

To this end, I have released <a href="https://github.com/aidanlister/django-facebook">django-facebook</a> which offers total and seamless integration with Facebook for your social apps.

It has a bunch of nifty tools including template tags, a nice middleware, an authentication backend and some <a href="https://github.com/aidanlister/django-facebook/blob/master/README.md">actual documentation</a>!

Spread the word, and watch this space.