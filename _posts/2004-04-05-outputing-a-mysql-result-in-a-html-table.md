---
layout: post
title: Outputing a MySQL result in a HTML table
section: developer
summary: |
    <code>mysql_draw_table()</code> outputs a MySQL result set as a HTML table.
---
When debugging a queryset without the luxuries of Sequel Pro or PHPMyAdmin, sometimes it's handy to dump a resultset straight into your page as a HTML table.

{% highlight php %}
<?php
/**
 * Replicate the output from `mysql --html`
 *
 * Draws a HTML table from a query resource
 *
 * @author      Aidan Lister <aidan@php.net>
 * @version     1.0.3
 * @link        http://aidanlister.com/2004/04/outputing-a-mysql-result-in-a-html-table/
 * @param       array   $result    The result of a mysql_query
 * @param       string  $null      Text to replace empty values with
 */
function mysql_draw_table($result, $null = '&nbsp;')
{
    // Sanity check
    if (!is_resource($result) ||
        substr(get_resource_type($result), 0, 5) !== 'mysql') {
        return false;
    }

    $out = "<table>\n";
  
    // Table header
    $out .= "\t<tr>";
    for ($i = 0, $ii = mysql_num_fields($result); $i < $ii; $i++) {
        $out .= '<th>' . mysql_field_name($result, $i) . '</th>';
    }
    $out .= "</tr>\n";
  
    // Table content
    for ($i = 0, $ii = mysql_num_rows($result); $i < $ii; $i++) {
        $out .= "\t<tr>";

        $row = mysql_fetch_row($result);
        foreach ($row as $value) {
            // Display empty cells
            $value = (empty($value) && ($value != '0')) ?
                $null :
                htmlspecialchars($value);

            $out .= '<td>' . $value . '</td>';
        }

        $out .= "</tr>\n";
    }
  
    $out .= "</table>\n";
    echo $out;
}
?>
{% endhighlight %}

For example,
{% highlight php %}
<?php
$result = mysql_query("SELECT VERSION(), USER()");
mysql_draw_table($result);
?>
{% endhighlight %}

Would result in:
<code>
<table>
<tr><th>VERSION()</th><th>USER()</th></tr>
<tr><td>5.0.51a-24</td><td>aidan@localhost</td></tr>
</table>
</code>