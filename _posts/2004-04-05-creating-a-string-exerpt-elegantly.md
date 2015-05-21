---
layout: post
title: Creating a string exerpt elegantly
section: developer
---
Shortening a chunk of text into something suitable for display is an exceptionally common task, this function provides a number of options for shortening text and URLs.

{% highlight php %}
<?php
/**
 * Chop a string into a smaller string.
 *
 * @author      Aidan Lister &lt;aidan@php.net&gt;
 * @version     1.1.0
 * @link        http://aidanlister.com/2004/04/creating-a-string-exerpt-elegantly/
 * @param       mixed  $string   The string you want to shorten
 * @param       int    $length   The length you want to shorten the string to
 * @param       bool   $center   If true, chop in the middle of the string
 * @param       mixed  $append   String appended if it is shortened
 */
function str_chop($string, $length = 60, $center = false, $append = null)
{
    // Set the default append string
    if ($append === null) {
        $append = ($center === true) ? ' ... ' : ' ...';
    }
 
    // Get some measurements
    $len_string = strlen($string);
    $len_append = strlen($append);
 
    // If the string is longer than the maximum length, we need to chop it
    if ($len_string &gt; $length) {
        // Check if we want to chop it in half
        if ($center === true) {
            // Get the lengths of each segment
            $len_start = $length / 2;
            $len_end = $len_start - $len_append;
 
            // Get each segment
            $seg_start = substr($string, 0, $len_start);
            $seg_end = substr($string, $len_string - $len_end, $len_end);
 
            // Stick them together
            $string = $seg_start . $append . $seg_end;
        } else {
            // Otherwise, just chop the end off
            $string = substr($string, 0, $length - $len_append) . $append;
        }
    }
 
    return $string;
}
?>
{% endhighlight %}

An example of this function in action:
{% highlight php %}
<?php
$longtext = &quot;this is some really long text with long words that should be chopped&quot;;
$longlink = &quot;http://thisisareally.longlink/with/lots/of/stupid/paths/&quot;;
 
// Chop at default length
echo str_chop($longtext);
echo &quot;\n&quot;;
 
// Chop in the middle
echo str_chop($longtext, 60, true);
echo &quot;\n&quot;;
 
// Chop a link
echo str_chop($longlink, 40, true);
echo &quot;\n&quot;;
 
// Chop a link whirlpool style
echo &quot;&lt;a href=\&quot;$longlink\&quot;&gt;&quot; . str_chop($longlink, 40, true) . '&lt;/a&gt;';
?>
{% endhighlight %}

This would produce the following output:

<code>
this is some really long text with long words that shoul ...
this is some really long text  ... ds that should be chopped
http://thisisareally ... f/stupid/paths/
&lt;a href=&quot;http://thisisareally.longlink/with/lots/of/stupid/paths/&quot;&gt;http://thisisareally ... f/stupid/paths/&lt;/a&gt;
</code>