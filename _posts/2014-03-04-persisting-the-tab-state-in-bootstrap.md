---
layout: post
title: Persisting the tab state in Bootstrap
section: developer
---
<a href="http://getbootstrap.com/javascript/#tabs">Bootstrap tabs</a> are great, but not having the tab persisted between page navigations can be a little frustrating for end users.

There's a few things that need to be done in a complete solution:

<ol>
<li>Show the correct tab when the page is loaded if there is a hash in the URL.</li>
<li>Change the hash in the URL when the tab is changed.</li>
<li>Change the tab when the hash changes in the URL (back / forward buttons), and make sure the first tab is cycled correctly.</li>
</ol>

This was a bit much for a one liner, so I've made a simple jQuery plugin:

[js]
/**
 * jQuery Plugin: Sticky Tabs
 *
 * @author Aidan Lister <aidan@php.net>
 * @version 1.2.0
 */
(function ( $ ) {
    $.fn.stickyTabs = function( options ) {
        var context = this

        var settings = $.extend({
            getHashCallback: function(hash, btn) { return hash }
        }, options );

        // Show the tab corresponding with the hash in the URL, or the first tab.
        var showTabFromHash = function() {
          var hash = window.location.hash;
          var selector = hash ? 'a[href="' + hash + '"]' : 'li.active > a';
          $(selector, context).tab('show');
        }

        // We use pushState if it's available so the page won't jump, otherwise a shim.
        var changeHash = function(hash) {
          if (history && history.pushState) {
            history.pushState(null, null, '#' + hash);
          } else {
            scrollV = document.body.scrollTop;
            scrollH = document.body.scrollLeft;
            window.location.hash = hash;
            document.body.scrollTop = scrollV;
            document.body.scrollLeft = scrollH;
          }
        }

        // Set the correct tab when the page loads
        showTabFromHash(context)

        // Set the correct tab when a user uses their back/forward button
        $(window).on('hashchange', showTabFromHash);

        // Change the URL when tabs are clicked
        $('a', context).on('click', function(e) {
          var hash = this.href.split('#')[1];
          var adjustedhash = settings.getHashCallback(hash, this);
          changeHash(adjustedhash);
        });

        return this;
    };
}( jQuery ));
[/js]

You can call the plugin like so:

    <code>$('.nav-tabs-sticky').stickyTabs();</code>

You can also find <a href="https://github.com/aidanlister/jquery-stickytabs">jquery.stickytabs.js</a> on GitHub or install via bower with <code>bower install jquery-stickytabs</code>.
