---
layout: post
title: Handling tab to space conversions
section: developer
---
When outputting code in a browser, or automatically formatting someone else's code, it's handy to be able to accurately convert tabs to spaces.

This function converts tabs to the appropriate number of spaces to preserve formatting.

{% highlight php %}
<?php
/**
 * Converts tabs to the appropriate amount of spaces while preserving formatting
 *
 * @author      Aidan Lister &lt;aidan@php.net&gt;
 * @version     1.2.0
 * @link        http://aidanlister.com/2004/04/handling-tab-to-space-conversions/
 * @param       string    $text     The text to convert
 * @param       int       $spaces   Number of spaces per tab column
 * @return      string    The text with tabs replaced
 */
function tab2space ($text, $spaces = 4)
{
    // Explode the text into an array of single lines
    $lines = explode(&quot;\n&quot;, $text);
 
    // Loop through each line
    foreach ($lines as $line) {
 
        // Break out of the loop when there are no more tabs to replace
        while (false !== $tab_pos = strpos($line, &quot;\t&quot;)) {
 
            // Break the string apart, insert spaces then concatenate
            $start = substr($line, 0, $tab_pos);
            $tab   = str_repeat(' ', $spaces - $tab_pos % $spaces);
            $end   = substr($line, $tab_pos + 1);
            $line  = $start . $tab . $end;
        }
 
        $result[] = $line;
    }
 
    return implode(&quot;\n&quot;, $result);
}
?>
{% endhighlight %}

A quick example:

{% highlight php %}
<?php
$data  = &quot;fooo\t\tbar\n&quot;;
$data .= &quot;foooo\t\tbar\n&quot;;
$data .= &quot;fooooo\t\tbar\n&quot;;
 
echo &quot;This is the example data:\n&quot;;
echo $data;
echo &quot;n&quot;;
 
echo &quot;With simple replace:\n&quot;;
echo str_replace(&quot;\t&quot;, &quot;    &quot;, $data);
echo &quot;n&quot;;
 
echo &quot;With tab2space:\n&quot;;
echo tab2space($data, 8);
?>
{% endhighlight %}

This would produce the following output:

<code>
This is the example data:
fooobar
foooobar
fooooobar

With simple replace:
fooo        bar
foooo        bar
fooooo        bar

With tab2space:
fooo            bar
foooo           bar
fooooo          bar
</code>