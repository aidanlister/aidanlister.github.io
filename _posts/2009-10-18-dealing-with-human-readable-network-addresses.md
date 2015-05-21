---
layout: post
title: Dealing with human readable network addresses
section: developer
---
Let's say we need to build a list of network addresses to whitelist access to a restricted area, or blacklist a troublesome spammer. It's handy to be able to enter data in a variety of formats, including hostnames, IP addresses, and IP ranges.

Usually we will store this information as a <em>long</em>, then we can easily see if a requesting IP address falls within our list of networks simply by numerical comparison, specifically: start_range < requesting_ip < end_range.

However, our user doesn't want to enter the list of IPs as a <em>long</em> so we need tools to convert between an attractive input format, and an array of start and end ranges.

To accomplish this, we define two functions <code>network_expand_range</code> and <code>network_compact_range</code>. Let's have a look:

{% highlight php %}
<?php
/**
 * Take small human readable network and give dotted quad range
 *
 * Acceptable inputs:
 *   1.1.1.1 - 1.1.255.255
 *   2.2.2.0-255
 *   3.3.3.*
 *   4.4.*.*
 *   5.5.5.20-40
 *   google.com
 *
 * @author Aidan Lister <aidan@php.net>
 * @link http://aidanlister.com/2009/10/dealing-with-human-readable-network-addresses/
 * @param input $string A newline separated list of network ranges
 * @return array Start and end addresses in dotted quads, or false
 */
function network_expand_range($network)
{
    // Sanity check    
    if (empty($network)) {
        return false;
    }

    $has_star = strpos($network, '*');
    $has_dash = strpos($network, '-');
    
    // Last character of a domain must be in the alphabet
    if (ctype_alpha($network[strlen($network)-1])) {
        if (!$address = gethostbyname($network)) {
          return false;
        }
        return array($address, $address);
    } 

    // Simple
    if ($has_star === false && $has_dash === false) {    
        $res   = long2ip(ip2long($network));
        $start = $end = $res;
        if ($res === '0.0.0.0') {
            return false;
        }
    }
        
    // Using a star
    if ($has_star !== false) {
        $start = long2ip(ip2long(str_replace('*', '0', $network)));
        $end   = long2ip(ip2long(str_replace('*', '255', $network)));
        if ($start === '0.0.0.0' || $end === '0.0.0.0') {
            return false;
        }                
    }
        
    // Using a dash
    if ($has_dash !== false) {
        list($start, $end) = explode('-', $network);
        
        // Check whether end is a full IP or just the last quad
        if (strpos($end, '.') !== false) {
            $end = long2ip(ip2long(trim($end)));
        } else {
            // Get the first 3 quads of the start address
            $classc = substr($start, 0, strrpos($start, '.'));
            $classc .= '.' . $end;
            $end = long2ip(ip2long(trim($classc)));
        }
        
        // Check for failure
        $start = long2ip(ip2long(trim($start)));
        if ($start === '0.0.0.0' || $end === '0.0.0.0') {
            return false;
        }
    }

    return array($start, $end);
}
?>
{% endhighlight %}

Next, the sister function to reverse the expansion process:

{% highlight php %}
<?php
/**
 * Convert start and end address into small human readable string
 *
 * For example, $s=1.1.1.1 and $e=1.1.255.255 into 1.1.*.*
 *
 * @author Aidan Lister <aidan@php.net>
 * @link http://aidanlister.com/2009/10/dealing-with-human-readable-network-addresses/
 * @param int $start The start address
 * @param int $end The end address
 * @return string A start and end range as the small formatted string
 */
function network_compact_range($start, $end)
{
    if ($start === $end) {
        $output = $start;
    } else {
        $s = explode('.', $start);
        $e = explode('.', $end);
        if ($s[0] === $e[0] && $s[1] === $e[1] && $s[2] === $e[2]) {
            if ($s[3] === '0' && $e[3] === '255') {
                $s[3] = '*';
                $output = implode('.', $s);
            } else {
                $s[3] = sprintf('%s-%s', $s[3], $e[3]);
                $output = implode('.', $s);
            }
        } else {
            $output = $start .' - '. $end;
        }
    }
            
    return $output;
}
?>
{% endhighlight %}

To show how it all works, let's look at some examples:

{% highlight php %}
<?php
// Example input (maybe from a textarea, database or file)
$input = array();
$input[] = '1.1.1.1 - 1.1.255.255';
$input[] = '2.2.2.0-255';
$input[] = '3.3.3.*';
$input[] = '4.4.*.*';
$input[] = '5.5.5.20-40';
$input[] = 'google.com';

echo "The input is:\n";
foreach ($input as $network) {
    echo "$network\n";
}

echo "\n";
echo "Expanded into ranges:\n";
foreach ($input as $network) {
    list($start, $end) = network_expand_range($network);
    echo "$start - $end\n";
}

echo "\n";
echo "As a long:\n";
foreach ($input as $network) {
    list($start, $end) = network_expand_range($network);
    echo ip2long($start) . ' - ' . ip2long($end) . "\n";
}

echo "\n";
echo "Back into compacted form:\n";
foreach ($input as $network) {
    list($start, $end) = network_expand_range($network);
    echo network_compact_range($start, $end), "\n";
}
?>
{% endhighlight %}

The output of this would be something like:

<code>
The input is:
1.1.1.1 - 1.1.255.255
2.2.2.0-255
3.3.3.*
4.4.*.*
5.5.5.20-40
google.com

Expanded into ranges:
1.1.1.1 - 1.1.255.255
2.2.2.0 - 2.2.2.255
3.3.3.0 - 3.3.3.255
4.4.0.0 - 4.4.255.255
5.5.5.20 - 5.5.5.40
74.125.53.100 - 74.125.53.100

As a long:
16843009 - 16908287
33686016 - 33686271
50529024 - 50529279
67371008 - 67436543
84215060 - 84215080
1249719652 - 1249719652

Back into compacted form:
1.1.1.1 - 1.1.255.255
2.2.2.*
3.3.3.*
4.4.0.0 - 4.4.255.255
5.5.5.20-40
74.125.53.100
</code>

Hope you find this useful.