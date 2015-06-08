---
layout: post
title: Human readable file sizes
section: developer
summary: |
    <code>size_readable()</code> converts a number of bytes into human readable form.
---
A very common task is converting a number of bytes into something for human digestion. This function supports both a maximum unit, both SI prefixes (base 10) and binary prefixes (base 2), and customised return strings.

{% highlight php %}
<?php
/**
 * Return human readable sizes
 *
 * @author      Aidan Lister <aidan@php.net>
 * @version     1.3.0
 * @link        http://aidanlister.com/2004/04/human-readable-file-sizes/
 * @param       int     $size        size in bytes
 * @param       string  $max         maximum unit
 * @param       string  $system      'si' for SI, 'bi' for binary prefixes
 * @param       string  $retstring   return string format
 */
function size_readable($size, $max = null, $system = 'si', $retstring = '%01.2f %s')
{
    // Pick units
    $systems['si']['prefix'] = array('B', 'K', 'MB', 'GB', 'TB', 'PB');
    $systems['si']['size']   = 1000;
    $systems['bi']['prefix'] = array('B', 'KiB', 'MiB', 'GiB', 'TiB', 'PiB');
    $systems['bi']['size']   = 1024;
    $sys = isset($systems[$system]) ? $systems[$system] : $systems['si'];
 
    // Max unit to display
    $depth = count($sys['prefix']) - 1;
    if ($max && false !== $d = array_search($max, $sys['prefix'])) {
        $depth = $d;
    }
 
    // Loop
    $i = 0;
    while ($size >= $sys['size'] && $i < $depth) {
        $size /= $sys['size'];
        $i++;
    }
 
    return sprintf($retstring, $size, $sys['prefix'][$i]);
}
?>
{% endhighlight %}

An example:
{% highlight php %}
<?php
// Simple
echo "Simple:\n";
echo size_readable(5500), "\n";
echo size_readable(17139812000), "\n";
 
// Maximum unit
echo "Max units in MB:\n";
echo size_readable(81620000000, 'MB'), "\n";
 
// 4 decimal accuracy
echo "4 decimals:\n";
echo size_readable(91711816100, null, true, '%01.4f %s'), "\n";
 
// 1 decimal accuracy, units in brackets, max unit in MB
echo "1 decimal, units in brackets, max unit of MB:\n";
$size = disk_total_space('/home');
echo size_readable($size, 'MB', true, '%01.1f (%s)');
?>
{% endhighlight %}

This would result in:
<code>
Simple:
5.50 K
17.14 GB

Max units in MB:
81620.00 MB

4 decimals:
91.7118 GB

1 decimal, units in brackets, max unit of MB:
1231.2 (MB)
</code>