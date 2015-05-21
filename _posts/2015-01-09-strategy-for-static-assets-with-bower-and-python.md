---
layout: post
title: A strategy for static assets with Bower on Python/Django, Heroku and S3
section: developer
---
The case for frontend package management is pretty strong. Once you’ve got jQuery, Bootstrap, half a dozen webcomponents and a few javascript libraries things are getting difficult to maintain and collecting static assets to S3 is taking 45 minutes.

You’ve already given up on the heroku build process and you’re using heroku config:set DISABLE_COLLECTSTATIC 1 and running the collectstatic process as part of your build process like heroku run:detached python manage.py collectstatic --noinput.

So now you’re ready for Bower. There’s a few options like django-bower, but I found this got in the way and gave up many of the benefits of using Bower itself like having a bower.json.

So here’s a different strategy: We’ll collect static assets to S3 directly from your development machine.

Firstly let’s set up Bower. In your project root let’s add your bower.json:

{% highlight json %}
    {
      "name": "yourproject",
      "version": "0.0.0",
      "homepage": "https://github.com/yourname/yourproject",
      "authors": [
        "Aidan Lister <aidan@aidanlister.com>"
      ],
      "license": "MIT",
      "private": true,
      "ignore": [
        "**/.*",
        "node_modules",
        "bower_components",
        "test",
        "tests"
      ],
      "dependencies": {
        "aws-sdk": "~2.1",
        "bootstrap": "~3.3",
        "bootstrap-datepicker": "~1.3",
        "google-map": "~0.4.0",
        "handsontable": "handsontable/handsontable#~0.12.2",
        "jquery": "~1.11",
        "jquery-stickytabs": "aidanlister/jquery-stickytabs",
      }
    }
{% endhighlight %}

You’ll also need to add a .bowerrc to tell Bower where you’re putting everything:
