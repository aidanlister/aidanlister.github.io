---
layout: post
title: Converting arrays to human readable tables
section: developer
---
This function allows you to quickly display the contents of an array as a HTML table. It can also recursively display data for nested arrays.

{% highlight php %}
<?php
/**
 * Translate a result array into a HTML table
 *
 * @author      Aidan Lister &lt;aidan@php.net&gt;
 * @version     1.3.2
 * @link        http://aidanlister.com/2004/04/converting-arrays-to-human-readable-tables/
 * @param       array  $array      The result (numericaly keyed, associative inner) array.
 * @param       bool   $recursive  Recursively generate tables for multi-dimensional arrays
 * @param       string $null       String to output for blank cells
 */
function array2table($array, $recursive = false, $null = '&amp;nbsp;')
{
    // Sanity check
    if (empty($array) || !is_array($array)) {
        return false;
    }

    if (!isset($array[0]) || !is_array($array[0])) {
        $array = array($array);
    }

    // Start the table
    $table = &quot;&lt;table&gt;\n&quot;;

    // The header
    $table .= &quot;\t&lt;tr&gt;&quot;;
    // Take the keys from the first row as the headings
    foreach (array_keys($array[0]) as $heading) {
        $table .= '&lt;th&gt;' . $heading . '&lt;/th&gt;';
    }
    $table .= &quot;&lt;/tr&gt;\n&quot;;

    // The body
    foreach ($array as $row) {
        $table .= &quot;\t&lt;tr&gt;&quot; ;
        foreach ($row as $cell) {
            $table .= '&lt;td&gt;';

            // Cast objects
            if (is_object($cell)) { $cell = (array) $cell; }
            
            if ($recursive === true &amp;&amp; is_array($cell) &amp;&amp; !empty($cell)) {
                // Recursive mode
                $table .= &quot;\n&quot; . array2table($cell, true, true) . &quot;\n&quot;;
            } else {
                $table .= (strlen($cell) &gt; 0) ?
                    htmlspecialchars((string) $cell) :
                    $null;
            }

            $table .= '&lt;/td&gt;';
        }

        $table .= &quot;&lt;/tr&gt;\n&quot;;
    }

    $table .= '&lt;/table&gt;';
    return $table;
}
?>
{% endhighlight %}

A quick example,
{% highlight php %}
<?php
$data[0]['Foo'] = 'Data 1';
$data[0]['Bar'] = 'Data 2';
$data[0]['Baz'] = 'Data 3';

$data[1]['Foo'] = 'Data 4';
$data[1]['Bar'] = 'Data 5';
$data[1]['Baz'] = 'Data 6';

echo array2table($data);
?>
{% endhighlight %}

Would produce the output:
<code language="html">
&lt;table&gt;
&lt;tr&gt;&lt;th&gt;Foo&lt;/th&gt;&lt;th&gt;Bar&lt;/th&gt;&lt;th&gt;Baz&lt;/th&gt;&lt;/tr&gt;
&lt;tr&gt;&lt;td&gt;Data 1&lt;/td&gt;&lt;td&gt;Data 2&lt;/td&gt;&lt;td&gt;Data 3&lt;/td&gt;&lt;/tr&gt;
&lt;tr&gt;&lt;td&gt;Data 4&lt;/td&gt;&lt;td&gt;Data 5&lt;/td&gt;&lt;td&gt;Data 6&lt;/td&gt;&lt;/tr&gt;
&lt;/table&gt;
</code>