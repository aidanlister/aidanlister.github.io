---
layout: post
title: Quickly converting a MySQL timestamp to a Unix Timestamp
section: developer
---
A commonly asked question is converting MySQL timestamps into unix timestamps. Although it's usually easiest to do this on the mysql server with <code>UNIX_TIMESTAMP()</code>, you can achieve the same thing with this function.

{% highlight php %}
<?php
/**
 * Convert MySQL timestamp to unix timestamp
 *
 * @author      Aidan Lister &lt;aidan@php.net&gt;
 * @version     1.1.0
 * @link        http://aidanlister.com/2004/04/quickly-converting-a-mysql-timestamp-to-a-unix-timestamp/
 * @param       string      $timestamp      MySQL timestamp
 */
function mysql2unixtime($timestamp)
{
    $parts = sscanf($timestamp, '%04u%02u%02u%02u%02u%02u');
    $string = vsprintf('%04u-%02u-%02u %02u:%02u:%02u', $parts);
 
    return strtotime($string);
}
?>
{% endhighlight %}

A quick example:
{% highlight php %}
<?php
// Get a MySQL timestamp
$result = mysql_query(&quot;SELECT NOW() + 0&quot;);
$datetime = mysql_result($result, 0);
echo &quot;MySQL Datetime: $datetime\n&quot;;
 
// Convert to unix timestamp
$timestamp = mysql2unixtime($datetime);
echo &quot;Unix Timestamp: $timestamp\n&quot;;
 
// Better
$result = mysql_query(&quot;SELECT UNIX_TIMESTAMP(NOW())&quot;);
$datetime = mysql_result($result, 0);
echo &quot;Unix Timestamp, from MySQL: $datetime\n&quot;;
?>
{% endhighlight %}

This would output:
<code>
MySQL Datetime: 20090408234019.000000
Unix Timestamp: 1239198019
Unix Timestamp, from MySQL: 1239198019
</code>