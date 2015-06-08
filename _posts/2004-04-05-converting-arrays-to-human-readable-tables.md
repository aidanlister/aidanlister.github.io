---
layout: post
title: Converting arrays to human readable tables
section: developer
summary: |
    <code>array2table()</code> allows you to quickly display the contents of an array as a HTML table. It can also recursively display data for nested arrays.
---
This function allows you to quickly display the contents of an array as a HTML table. It can also recursively display data for nested arrays.

{% highlight php %}
<?php
/**
 * Translate a result array into a HTML table
 *
 * @author      Aidan Lister <aidan@php.net>
 * @version     1.3.2
 * @link        http://aidanlister.com/2004/04/converting-arrays-to-human-readable-tables/
 * @param       array  $array      The result (numericaly keyed, associative inner) array.
 * @param       bool   $recursive  Recursively generate tables for multi-dimensional arrays
 * @param       string $null       String to output for blank cells
 */
function array2table($array, $recursive = false, $null = '&nbsp;')
{
    // Sanity check
    if (empty($array) || !is_array($array)) {
        return false;
    }

    if (!isset($array[0]) || !is_array($array[0])) {
        $array = array($array);
    }

    // Start the table
    $table = "<table>\n";

    // The header
    $table .= "\t<tr>";
    // Take the keys from the first row as the headings
    foreach (array_keys($array[0]) as $heading) {
        $table .= '<th>' . $heading . '</th>';
    }
    $table .= "</tr>\n";

    // The body
    foreach ($array as $row) {
        $table .= "\t<tr>" ;
        foreach ($row as $cell) {
            $table .= '<td>';

            // Cast objects
            if (is_object($cell)) { $cell = (array) $cell; }
            
            if ($recursive === true && is_array($cell) && !empty($cell)) {
                // Recursive mode
                $table .= "\n" . array2table($cell, true, true) . "\n";
            } else {
                $table .= (strlen($cell) > 0) ?
                    htmlspecialchars((string) $cell) :
                    $null;
            }

            $table .= '</td>';
        }

        $table .= "</tr>\n";
    }

    $table .= '</table>';
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
<table>
<tr><th>Foo</th><th>Bar</th><th>Baz</th></tr>
<tr><td>Data 1</td><td>Data 2</td><td>Data 3</td></tr>
<tr><td>Data 4</td><td>Data 5</td><td>Data 6</td></tr>
</table>
</code>