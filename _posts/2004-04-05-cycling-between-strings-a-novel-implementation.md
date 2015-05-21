---
layout: post
title: Cycling between strings, a novel implementation
section: developer
---
When outputting a table, a common task is to alternate the row colours to give a zebra effect - an effect commonly said (though debated) to increase readability.

A novel solution to this problem involves using PHP5's new <code>__toString</code> method: introducing Cycle.

{% highlight php %}
<?php
/**
 * A simple cycle class for alternating between strings.
 *
 * @author      Aidan Lister &lt;aidan@php.net&gt;
 * @version     1.0.1
 * @link        http://aidanlister.com/2004/04/cycling-between-strings-a-novel-implementation/
 */
class Cycle
{
    /**
     * Array of strings to be cycled
     *
     * @var     array
     */
    private $_args;


    /**
     * The current position in the stack
     *
     * @var     array
     */
    private $_key;
    

    /**
     * Constructor
     *
     * @param   overloader  Strings to be cycled through
     */
    function __construct()
    {
        $this-&gt;_args = func_get_args();
        $this-&gt;_key = -1;
    }
    

    /**
     * Convert to a string
     *
     * @return  string      The next string in the stack
     */
    function __toString()
    {
        return (string) isset($this-&gt;_args[$this-&gt;_key += 1]) ?
            $this-&gt;_args[$this-&gt;_key] :
            $this-&gt;_args[$this-&gt;_key = 0] ;
    }
}
?>
{% endhighlight %}

Thus with minimal effort, simply echoing the string will seemingly magically alternate between the specified outputs. An example:

{% highlight php %}
<?php
require_once 'Cycle.php';
$rowclass = new Cycle('odd', 'even');

for ($i = 0; $i &lt; 5; $i++) {
    echo &quot;&lt;td class=$rowclass&gt;\n&quot;;
}
?>
{% endhighlight %}

This would output:

<code>
&lt;td class=odd&gt;
&lt;td class=even&gt;
&lt;td class=odd&gt;
&lt;td class=even&gt;
</code>

And there we have it, zebra striped tables without a messy counter or other tricks.