---
layout: post
title: Viewing binary data as a hexdump in PHP
section: developer
---
When dealing with binary data it's always helpful to see exactly what PHP sees - this function allows you to dump a binary data stream into a human-friendly format.

{% highlight php %}
<?php
/**
 * View any string as a hexdump.
 *
 * This is most commonly used to view binary data from streams
 * or sockets while debugging, but can be used to view any string
 * with non-viewable characters.
 *
 * @version     1.3.2
 * @author      Aidan Lister <aidan@php.net>
 * @author      Peter Waller <iridum@php.net>
 * @link        http://aidanlister.com/2004/04/viewing-binary-data-as-a-hexdump-in-php/
 * @param       string  $data        The string to be dumped
 * @param       bool    $htmloutput  Set to false for non-HTML output
 * @param       bool    $uppercase   Set to true for uppercase hex
 * @param       bool    $return      Set to true to return the dump
 */
function hexdump ($data, $htmloutput = true, $uppercase = false, $return = false)
{
    // Init
    $hexi   = '';
    $ascii  = '';
    $dump   = ($htmloutput === true) ? '<pre>' : '';
    $offset = 0;
    $len    = strlen($data);
 
    // Upper or lower case hexadecimal
    $x = ($uppercase === false) ? 'x' : 'X';
 
    // Iterate string
    for ($i = $j = 0; $i < $len; $i++)
    {
        // Convert to hexidecimal
        $hexi .= sprintf("%02$x ", ord($data[$i]));
 
        // Replace non-viewable bytes with '.'
        if (ord($data[$i]) >= 32) {
            $ascii .= ($htmloutput === true) ?
                            htmlentities($data[$i]) :
                            $data[$i];
        } else {
            $ascii .= '.';
        }
 
        // Add extra column spacing
        if ($j === 7) {
            $hexi  .= ' ';
            $ascii .= ' ';
        }
 
        // Add row
        if (++$j === 16 || $i === $len - 1) {
            // Join the hexi / ascii output
            $dump .= sprintf("%04$x  %-49s  %s", $offset, $hexi, $ascii);
            
            // Reset vars
            $hexi   = $ascii = '';
            $offset += 16;
            $j      = 0;
            
            // Add newline            
            if ($i !== $len - 1) {
                $dump .= "\n";
            }
        }
    }
 
    // Finish dump
    $dump .= $htmloutput === true ?
                '</pre>' :
                '';
    $dump .= "\n";
 
    // Output method
    if ($return === false) {
        echo $dump;
    } else {
        return $dump;
    }
}
?>
{% endhighlight %}

We can generate some garbage as an example:
{% highlight php %}
<?php
// Generate a string with all sorts of funny characters
for ($string = '', $i = 0; $i < 255; $i++) {
    $string .= chr($i);
}
 
// Dump it
hexdump($string);
?>
{% endhighlight %}

This would produce the following dump (make sure you click the view-source button on the code sample, because it looks odd in the site template).
<code>
0000  00 01 02 03 04 05 06 07  08 09 0a 0b 0c 0d 0e 0f   ........ ........
0010  10 11 12 13 14 15 16 17  18 19 1a 1b 1c 1d 1e 1f   ........ ........
0020  20 21 22 23 24 25 26 27  28 29 2a 2b 2c 2d 2e 2f    !"#$%&' ()*+,-./
0030  30 31 32 33 34 35 36 37  38 39 3a 3b 3c 3d 3e 3f   01234567 89:;<=>?
0040  40 41 42 43 44 45 46 47  48 49 4a 4b 4c 4d 4e 4f   @ABCDEFG HIJKLMNO
0050  50 51 52 53 54 55 56 57  58 59 5a 5b 5c 5d 5e 5f   PQRSTUVW XYZ[]^_
0060  60 61 62 63 64 65 66 67  68 69 6a 6b 6c 6d 6e 6f   `abcdefg hijklmno
0070  70 71 72 73 74 75 76 77  78 79 7a 7b 7c 7d 7e 7f   pqrstuvw xyz{|}~
0080  80 81 82 83 84 85 86 87  88 89 8a 8b 8c 8d 8e 8f   €‚ƒ„…†‡ ˆ‰Š‹ŒŽ
0090  90 91 92 93 94 95 96 97  98 99 9a 9b 9c 9d 9e 9f   ‘’“”•–— ˜™š›œžŸ
00a0  a0 a1 a2 a3 a4 a5 a6 a7  a8 a9 aa ab ac ad ae af    ¡¢£¤¥¦§ ¨©ª«¬®¯
00b0  b0 b1 b2 b3 b4 b5 b6 b7  b8 b9 ba bb bc bd be bf   °±²³´µ¶· ¸¹º»¼½¾¿
00c0  c0 c1 c2 c3 c4 c5 c6 c7  c8 c9 ca cb cc cd ce cf   ÀÁÂÃÄÅÆÇ ÈÉÊËÌÍÎÏ
00d0  d0 d1 d2 d3 d4 d5 d6 d7  d8 d9 da db dc dd de df   ÐÑÒÓÔÕÖ× ØÙÚÛÜÝÞß
00e0  e0 e1 e2 e3 e4 e5 e6 e7  e8 e9 ea eb ec ed ee ef   àáâãäåæç èéêëìíîï
00f0  f0 f1 f2 f3 f4 f5 f6 f7  f8 f9 fa fb fc fd fe      ðñòóôõö÷ øùúûüýþ
</code>