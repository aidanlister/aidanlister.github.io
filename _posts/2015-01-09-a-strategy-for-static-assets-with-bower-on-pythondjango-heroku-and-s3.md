---
layout: post
title: A strategy for static assets with Bower on Python/Django, Heroku and S3
section: developer
summary: |
    We look at best practices for managing your static assets on Heroku and S3 using Bower.
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

You'll also need to add a .bowerrc to tell Bower where you’re putting everything:

{% highlight json %}
{
    "directory": "yourproject/static/bower_components"
}
{% endhighlight %}

Also, because we don’t need anything in that directory added to source control (which will slim down your git repository and reduce your slug and build time), add bower_components to your .gitignore.

In <em>.gitignore</em> add:

```
yourproject/static/bower_components
```

Next up set up your STATICFILES_DIRS, I found three folders covered it:

In your project settings file:

{% highlight python %}
STATICFILES_DIRS = (
    os.path.join(PROJECT_DIR, 'static/libraries'), # Commercial libraries not available on GitHub or via Bower
    os.path.join(PROJECT_DIR, 'static/custom'), # Custom frontend assets in css/js subfolders
    os.path.join(PROJECT_DIR, 'static/bower_components'), # Bower controlled assets
)
{% endhighlight %}

Run bower update to download all the bower_components and you’re ready to go. Now we need to collect all of our static assets to S3. We can do this using the built-in collectstatic management command:

```
python manage.py collectstatic --noinput --settings=yourproject.settings.production
```

This will collect static files from your local machine and upload them to the S3 bucket you’ve configured in your production settings. Here’s how my production static storage is set up:

In <em>yourproject/settings/production.py</em>:

```
STATICFILES_STORAGE = 'yourproject.utils.storage.StaticStorage'
```

In <em>yourproject/utils/storage.py</em>:

{% highlight python %}
from storages.backends.s3boto import S3BotoStorage
class StaticStorage(S3BotoStorage):
    def __init__(self, *args, **kwargs):
        kwargs['location'] = 'static'
        kwargs['bucket'] = 'your-bucket'
        kwargs['secure_urls'] = True
        kwargs['custom_domain'] = 'your-bucket.s3.amazonaws.com'
        super(StaticStorage, self).__init__(*args, **kwargs)
{% endhighlight %}

If you’ve using fig, run it in a detached container like so:

```
fig run -d web python manage.py collectstatic --noinput --settings=yourproject.settings.production
```

If you’re using fabric (and you should!), here’s the task:

{% highlight python %}
@task
def collectstatic(figenv, settingsfile):
    """ Update the static assets. e.g. fab collectstatic:abas,production

    This would use the fig env testproject with the StaticFiles configured in abas.settings.production
    """
    cmd = 'fig -p %s run -d web python manage.py collectstatic --noinput --settings=testproject.settings.%s'
    local(cmd % (figenv, settingsfile))
{% endhighlight %}

Hope this helps!
