---
layout: post
title: Quick and easy random string generation
section: developer
---
Generating a random string is an incredibly common task, this function provides quick string generation with four available output types: alpha, numeric, alphanum and hexadec.

{% highlight php %}
<?php
/**
 * Generate and return a random string
 *
 * The default string returned is 8 alphanumeric characters.
 *
 * The type of string returned can be changed with the output parameter.
 * Four types are available: alpha, numeric, alphanum and hexadec. 
 *
 * If the output parameter does not match one of the above, then the string
 * supplied is used.
 *
 * @author      Aidan Lister <aidan@php.net>
 * @version     2.1.0
 * @link        http://aidanlister.com/2004/04/quick-and-easy-random-string-generation/
 * @param       int     $length  Length of string to be generated
 * @param       string  $seeds   Seeds string should be generated from
 */
function str_rand($length = 8, $output = 'alphanum')
{
    // Possible seeds
    $outputs['alpha']    = 'abcdefghijklmnopqrstuvwqyz';
    $outputs['numeric']  = '0123456789';
    $outputs['alphanum'] = 'abcdefghijklmnopqrstuvwqyz0123456789';
    $outputs['hexadec']  = '0123456789abcdef';
 
    // Choose seed
    if (isset($outputs[$output])) {
        $output = $outputs[$output];
    }
 
    // Seed generator
    list($usec, $sec) = explode(' ', microtime());
    $seed = (float) $sec + ((float) $usec * 100000);
    mt_srand($seed);
 
    // Generate
    $str = '';
    $output_count = strlen($output);
    for ($i = 0; $length > $i; $i++) {
        $str .= $output{mt_rand(0, $output_count - 1)};
    }
 
    return $str;
}
?>
{% endhighlight %}

A quick example:
{% highlight php %}
<?php
// Simple
echo str_rand();
 
// Longer
echo str_rand(15);
 
// Really big number
echo str_rand(15, 'numeric');
 
// Custom seeds
echo str_rand(15, '01');
?>
{% endhighlight %}

This would produce the following output:
<code>
m2dy5ofe
remdjynd47b66hq
504359393089603
111001110111101
</code>