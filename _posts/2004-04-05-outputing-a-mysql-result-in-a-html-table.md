---
layout: post
title: Outputing a MySQL result in a HTML table
section: developer
---
When debugging a queryset without the luxuries of Sequel Pro or PHPMyAdmin, sometimes it's handy to dump a resultset straight into your page as a HTML table.

{% highlight php %}
<?php
/**
 * Replicate the output from `mysql --html`
 *
 * Draws a HTML table from a query resource
 *
 * @author      Aidan Lister &lt;aidan@php.net&gt;
 * @version     1.0.3
 * @link        http://aidanlister.com/2004/04/outputing-a-mysql-result-in-a-html-table/
 * @param       array   $result    The result of a mysql_query
 * @param       string  $null      Text to replace empty values with
 */
function mysql_draw_table($result, $null = '&amp;nbsp;')
{
    // Sanity check
    if (!is_resource($result) ||
        substr(get_resource_type($result), 0, 5) !== 'mysql') {
        return false;
    }

    $out = &quot;&lt;table&gt;\n&quot;;
  
    // Table header
    $out .= &quot;\t&lt;tr&gt;&quot;;
    for ($i = 0, $ii = mysql_num_fields($result); $i &lt; $ii; $i++) {
        $out .= '&lt;th&gt;' . mysql_field_name($result, $i) . '&lt;/th&gt;';
    }
    $out .= &quot;&lt;/tr&gt;\n&quot;;
  
    // Table content
    for ($i = 0, $ii = mysql_num_rows($result); $i &lt; $ii; $i++) {
        $out .= &quot;\t&lt;tr&gt;&quot;;

        $row = mysql_fetch_row($result);
        foreach ($row as $value) {
            // Display empty cells
            $value = (empty($value) &amp;&amp; ($value != '0')) ?
                $null :
                htmlspecialchars($value);

            $out .= '&lt;td&gt;' . $value . '&lt;/td&gt;';
        }

        $out .= &quot;&lt;/tr&gt;\n&quot;;
    }
  
    $out .= &quot;&lt;/table&gt;\n&quot;;
    echo $out;
}
?>
{% endhighlight %}

For example,
{% highlight php %}
<?php
$result = mysql_query(&quot;SELECT VERSION(), USER()&quot;);
mysql_draw_table($result);
?>
{% endhighlight %}

Would result in:
<code>
&lt;table&gt;
&lt;tr&gt;&lt;th&gt;VERSION()&lt;/th&gt;&lt;th&gt;USER()&lt;/th&gt;&lt;/tr&gt;
&lt;tr&gt;&lt;td&gt;5.0.51a-24&lt;/td&gt;&lt;td&gt;aidan@localhost&lt;/td&gt;&lt;/tr&gt;
&lt;/table&gt;
</code>