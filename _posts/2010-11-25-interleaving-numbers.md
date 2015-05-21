---
layout: post
title: Interleaving numbers 
section: developer
---
When you have two numbers that need to be indexed together for speedy lookups there are a variety of mechanisms that can be used, a fast and efficient mechanism is called the <a href="http://www.codexon.com/posts/morton-codes">Morton Interleave</a>.

{% highlight php %}
<?php
/**
 * Calculate a Morton Interleave for two numbers.
 *
 * @param    int   $x   The first number
 * @param    int   $y   The second number
 * @return   int      The Morton Interleave
 * @author   Aidan Lister <aidan@php.net>
 * @link     http://aidanlister.com/2010/11/interleaving-numbers/
 */
function interleave($x, $y) {
    $result = 0;
    $position = 0;
    $bit = 1;
 
    while ($bit <= $x || $bit <= $y) {
        if ($bit & $x)
            $result |= 1 << (2*$position+1);
        if ($bit & $y)
            $result |= 1 << (2*$position);
 
        $position += 1;
        $bit = 1 << $position;
    }
    return $result;
}

?>
{% endhighlight %}

A common use case for the interleave is storing a conversation between two users. For example, to retrieve all messages between user 231 and user 119 one would normally query the database with <code>WHERE (sender_id=231 AND receiver_id=119) OR (sender_id=119 AND receiver_id=231)</code>.

When using an interleave field, you could do something like <code>WHERE conversation_id=48447</code>. The number 48447 being the interleave, calculated as:

{% highlight php %}
<?php

$uids = array(231, 119);
interleave(max($uids), min($uids));

?>
{% endhighlight %}

This means faster queries and cleaner code!