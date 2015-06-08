---
layout: post
title: Cycling between strings, a novel implementation
section: developer
summary: |
    <code>Cycle</code> provides a novel way to iterate through an array. This is commonly used for zebra-striping tables.
---
When outputting a table, a common task is to alternate the row colours to give a zebra effect - an effect commonly said (though debated) to increase readability.

A novel solution to this problem involves using PHP5's new <code>__toString</code> method: introducing Cycle.

{% highlight php %}
<?php
/**
 * A simple cycle class for alternating between strings.
 *
 * @author      Aidan Lister <aidan@php.net>
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
        $this->_args = func_get_args();
        $this->_key = -1;
    }
    

    /**
     * Convert to a string
     *
     * @return  string      The next string in the stack
     */
    function __toString()
    {
        return (string) isset($this->_args[$this->_key += 1]) ?
            $this->_args[$this->_key] :
            $this->_args[$this->_key = 0] ;
    }
}
?>
{% endhighlight %}

Thus with minimal effort, simply echoing the string will seemingly magically alternate between the specified outputs. An example:

{% highlight php %}
<?php
require_once 'Cycle.php';
$rowclass = new Cycle('odd', 'even');

for ($i = 0; $i < 5; $i++) {
    echo "<td class=$rowclass>\n";
}
?>
{% endhighlight %}

This would output:

<code>
<td class=odd>
<td class=even>
<td class=odd>
<td class=even>
</code>

And there we have it, zebra striped tables without a messy counter or other tricks.