---
layout: post
title: Quick JavaScript email obfuscation
section: developer
---
<p>There are many methods to quickly obscure an email address from spam harvesters, this approach relies on the fact that most email harvesters won't understand JavaScript.</p>

{% highlight php %}
<?php
/**
 * Obfuscate an email address
 *
 * @author      Aidan Lister &lt;aidan@php.net&gt;
 * @version     1.1.0
 * @link        http://aidanlister.com/2004/04/quick-javascript-email-obfuscation/
 * @param       string      $email      E-mail
 * @param       string      $text       Text
 */
function mail_obfuscate($email, $text = '')
{
    // Default text
    if (empty($text)) {
$text = $email;
    }
    
    // Create string
    $string = sprintf('document.write(\'&lt;a href=&quot;mailto:%s&quot;&gt;%s&lt;/a&gt;\');',
            htmlspecialchars($email),
            htmlspecialchars($text));

    // Encode    
    for ($encode = '', $i = 0; $i &lt; strlen($string); $i++) {
        $encode .= '%' . bin2hex($string[$i]);
    }

    // Javascript
    $javascript = '&lt;script language=&quot;javascript&quot;&gt;eval(unescape(\'' . $encode . '\'))&lt;/script&gt;';

    return $javascript;
}
?>
{% endhighlight %}

<p>A quick example:</p>
{% highlight php %}
<?php
echo mail_obfuscate('aidan@php.net');
?>
{% endhighlight %}

<p>Would result in:</p>
<code>
&lt;script language=&quot;javascript&quot;&gt;eval(unescape('%64%6f%63%75%6d%65%6e%74%2e%77%72%69%74%65%28%27%3c%61%20%68%72%65%66%3d%22%6d%61%69%6c%74%6f%3a%61%69%64%61%6e%40%70%68%70%2e%6e%65%74%22%3e%61%69%64%61%6e%40%70%68%70%2e%6e%65%74%3c%2f%61%3e%27%29%3b'))&lt;/script&gt;
</code>

<p>As you can see, the output no longer remotely resembles an email address and is very unlikely to get picked up by a spam harvester.</p>