---
layout: post
title: Making time periods readable
section: developer
---
This is a flexible function for making time periods readable.

It allows for the conversion of an integer number of seconds into a readable string. For example, "121" into "2 minutes, 1 second".

The <code>$use</code> parameter allows you to define which time periods should be used (default is null, which is all time periods). The <code>$zeros</code> parameter allows you to set whether zero time periods should be displayed, e.g. "0 years, 0 months" (defaults to false).

{% highlight php %}
<?php
/**
 * A function for making time periods readable
 *
 * @author      Aidan Lister &lt;aidan@php.net&gt;
 * @version     2.0.1
 * @link        http://aidanlister.com/2004/04/making-time-periods-readable/
 * @param       int     number of seconds elapsed
 * @param       string  which time periods to display
 * @param       bool    whether to show zero time periods
 */
function time_duration($seconds, $use = null, $zeros = false)
{
    // Define time periods
    $periods = array (
        'years'     =&gt; 31556926,
        'Months'    =&gt; 2629743,
        'weeks'     =&gt; 604800,
        'days'      =&gt; 86400,
        'hours'     =&gt; 3600,
        'minutes'   =&gt; 60,
        'seconds'   =&gt; 1
        );

    // Break into periods
    $seconds = (float) $seconds;
    $segments = array();
    foreach ($periods as $period =&gt; $value) {
        if ($use &amp;&amp; strpos($use, $period[0]) === false) {
            continue;
        }
        $count = floor($seconds / $value);
        if ($count == 0 &amp;&amp; !$zeros) {
            continue;
        }
        $segments[strtolower($period)] = $count;
        $seconds = $seconds % $value;
    }

    // Build the string
    $string = array();
    foreach ($segments as $key =&gt; $value) {
        $segment_name = substr($key, 0, -1);
        $segment = $value . ' ' . $segment_name;
        if ($value != 1) {
            $segment .= 's';
        }
        $string[] = $segment;
    }

    return implode(', ', $string);
}
?>
{% endhighlight %}

Let's have a look at some examples:

{% highlight php %}
<?php
echo time_duration(100000000);
echo time_duration(100000000, null, true);
echo time_duration(100000000, 'yMw');
echo time_duration(100000000, 'd');
?>
{% endhighlight %}

This would produce the following output:

<code>
3 years, 2 months, 19 hours, 22 minutes, 16 seconds
3 years, 2 months, 0 weeks, 0 days, 19 hours, 22 minutes, 16 seconds
3 years, 2 months
1157 days
</code>