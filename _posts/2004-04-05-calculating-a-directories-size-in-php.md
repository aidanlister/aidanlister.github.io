---
layout: post
title: Calculating a directories size in PHP
section: developer
---
This function provides a heavily optimised method for calculating the size of large directories.

{% highlight php %}
<?php
/**
 * Calculate the size of a directory by iterating its contents
 *
 * @author      Aidan Lister &lt;aidan@php.net&gt;
 * @version     1.2.0
 * @link        http://aidanlister.com/2004/04/calculating-a-directories-size-in-php/
 * @param       string   $directory    Path to directory
 */
function dirsize($path)
{
    // Init
    $size = 0;
 
    // Trailing slash
    if (substr($path, -1, 1) !== DIRECTORY_SEPARATOR) {
        $path .= DIRECTORY_SEPARATOR;
    }
 
    // Sanity check
    if (is_file($path)) {
        return filesize($path);
    } elseif (!is_dir($path)) {
        return false;
    }
 
    // Iterate queue
    $queue = array($path);
    for ($i = 0, $j = count($queue); $i &lt; $j; ++$i)
    {
        // Open directory
        $parent = $i;
        if (is_dir($queue[$i]) &amp;&amp; $dir = @dir($queue[$i])) {
            $subdirs = array();
            while (false !== ($entry = $dir-&gt;read())) {
                // Skip pointers
                if ($entry == '.' || $entry == '..') {
                    continue;
                }
 
                // Get list of directories or filesizes
                $path = $queue[$i] . $entry;
                if (is_dir($path)) {
                    $path .= DIRECTORY_SEPARATOR;
                    $subdirs[] = $path;
                } elseif (is_file($path)) {
                    $size += filesize($path);
                }
            }
 
            // Add subdirectories to start of queue
            unset($queue[0]);
            $queue = array_merge($subdirs, $queue);
 
            // Recalculate stack size
            $i = -1;
            $j = count($queue);
 
            // Clean up
            $dir-&gt;close();
            unset($dir);
        }
    }
 
    return $size;
}
?>
{% endhighlight %}

A quick usage example:
{% highlight php %}
<?php
// Show the current directory
echo dirsize('.');
echo &quot;\n&quot;;
 
// We can use size_readable to format the result
// http://aidanlister.com/repos/v/function.size_readable.php
echo size_readable(dirsize('.'));
?>
{% endhighlight %}

This would output:
<code>
591677
591.68 K
</code>