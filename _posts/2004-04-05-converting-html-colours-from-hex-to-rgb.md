---
layout: post
title: Converting HTML colours from HEX to RGB
section: developer
---
HTML colours are usually represented as a 6 character hexadecimal string. To convert to an RGB value, you can use this function.

{% highlight php %}
<?php
/**
 * Convert a Color-HEX string into an RGB string
 *
 * @version     1.0.0
 * @author      Aidan Lister <aidan@php.net>
 * @link        http://aidanlister.com/2004/04/converting-html-colours-from-hex-to-rgb/
 * @param       string  $hex        The hex string
 * @param       string  $format     Format of the output
 */
function hex2rgb ($hex, $format = 'rgb(%d, %d, %d)')
{
    if (strlen($hex) === 3) {
        $rgb = sprintf($format,
            hexdec($hex[0]),
            hexdec($hex[1]),
            hexdec($hex[2]));
    } elseif (strlen($hex) === 6) {
        $rgb = sprintf($format,
            hexdec(substr($hex, 0, 2)),
            hexdec(substr($hex, 2, 2)),
            hexdec(substr($hex, 4, 2)));
    } else {
        $rgb = false;
    }

    return $rgb;
}
?>
{% endhighlight %}