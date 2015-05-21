---
layout: post
title: Improved PHP syntax highlighting
section: developer
---
PHP_Highlight uses PHP's built in tokenizer to provide reliable syntax highlighting for PHP code when server-side highlighting is required.

This will generates valid XHTML output, with function referencing (links back to the PHP manual for PHP functions) and configurable line numbering.

Extendable output methods provide loads of flexibility:
<ul>
<li><code>toHtml()</code> outputs highlighted PHP, lines ending with a &lt;br&gt;</li>
        <li><code>toHtmlBlock()</code> was designed for highlighting PHP code in user comments. Text is unaffected, but PHP code is wrapped and styled.</li>
        <li><code>toList()</code> outputs highlighted PHP in an orderedlist.</li>
        <li><code>toArray()</code> outputs the highlighted PHP as an array, allowing for further customisation.</li>
</ul>

Highlighting can be inline (with styles), or the same as highlight_file() where colors are taken from php.ini.

{% highlight php %}
<?php
/**
 * PHP 5 added a set of new constants which need to be declared in this file for
 * effective PHP 5 highlighting. It also removed constants, which need to be
 * included for PHP 4 highlighting.
 *
 * The following file will define constants for PHP 4 / PHP 5 compatability
 *
 * The source of this file can be found at:
 * http://tinyurl.com/plmlo
 *
 * It is part of the PEAR PHP_Compat package:
 * http://pear.php.net/package/PHP_Compat
 */
require_once 'PHP/Compat/Constant/T.php';


/**
 * Improved PHP syntax highlighting.
 *
 * Generates valid XHTML output with function referencing
 * and line numbering.
 *
 * Extendable output methods provide maximum flexibility,
 * toHtml(), toHtmlComment(), toList() and toArray().
 *
 * Highlighting can be inline (with styles), or the same as
 * highlight_file() where colors are taken from php.ini.
 *
 * @author      Aidan Lister &lt;aidan@php.net&gt;
 * @version     1.4.3
 * @link        http://aidanlister.com/2004/04/improved-php-syntax-highlighting/
 */
class PHP_Highlight
{
    /**
     * Hold highlight colors
     *
     * Contains an associative array of token types and colours.
     * By default, it contains the colours as specified by php.ini
     *
     * For example, to change the colour of strings, use something
     * simular to $h-&gt;highlight['string'] = 'blue';
     *
     * @var         array
     * @access      public
     */
    var $highlight;

    /**
     * Things to be replaced for formatting or otherwise reasons
     *
     * The first element contains the match array, the second the replace
     * array.
     *
     * @var         array
     * @access      public
     */
    var $replace = array(
        &quot;\t&quot;    =&gt; '&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;',
        ' '     =&gt; '&amp;nbsp;');

    /**
     * Format of the link to the PHP manual page
     *
     * @var         string
     * @access      public
     */
    var $manual = '&lt;a href=&quot;http://www.php.net/function.%s&quot;&gt;%s&lt;/a&gt;';

    /**
     * Format of the span tag to be wrapped around each token
     *
     * @var         string
     * @access      public
     */
    var $span;

    /**
     * Hold the source
     *
     * @var         string
     * @access      private
     */
    var $_source = false;

    /**
     * Hold plaintext keys
     *
     * An array of lines which are plaintext
     *
     * @var         array
     * @access      private
     */
    var $_plaintextkeys = array();


    /**
     * Constructor
     *
     * Populates highlight array
     *
     * @param   bool  $inline     If inline styles rather than colors are to be used
     * @param   bool  $plaintext  Do not format code outside PHP tags
     */
    function PHP_Highlight($inline = false)
    {
        // Inline
        if ($inline === false) {
            // Default colours from php.ini
            $this-&gt;highlight = array(
                'string'    =&gt; ini_get('highlight.string'),
                'comment'   =&gt; ini_get('highlight.comment'),
                'keyword'   =&gt; ini_get('highlight.keyword'),
                'bg'        =&gt; ini_get('highlight.bg'),
                'default'   =&gt; ini_get('highlight.default'),
                'html'      =&gt; ini_get('highlight.html')
            );
            $this-&gt;span = '&lt;span style=&quot;color: %s;&quot;&gt;%s&lt;/span&gt;';
        } else {
            // Basic styles
            $this-&gt;highlight = array(
                'string'    =&gt; 'string',
                'comment'   =&gt; 'comment',
                'keyword'   =&gt; 'keyword',
                'bg'        =&gt; 'bg',
                'default'   =&gt; 'default',
                'html'      =&gt; 'html'
            );
            $this-&gt;span = '&lt;span class=&quot;%s&quot;&gt;%s&lt;/span&gt;';
        }
    }


    /**
     * Load a file
     *
     * @access  public
     * @param   string      $file       The file to load
     * @return  bool        Returns TRUE
     */
    function loadFile($file)
    {
        $this-&gt;_source = file_get_contents($file);
        return true;
    }


    /**
     * Load a string
     *
     * @access  public
     * @param   string      $string     The string to load
     * @return  bool        Returns TRUE
     */
    function loadString($string)
    {
        $this-&gt;_source = $string;
        return true;
    }


    /**
     * Parse the loaded string into an array
     * Source is returned with the element key corresponding to the line number
     *
     * @access  public
     * @param   bool      $funcref   Reference functions to the PHP manual
     * @param   bool      $blocks    Whether to ignore processing plaintext
     * @return  array     An array of highlighted source code
     */
    function toArray($funcref = true, $blocks = false)
    {
        // Ensure source has been loaded
        if ($this-&gt;_source == false) {
            return false;
        }

        // Init
        $tokens     = token_get_all($this-&gt;_source);
        $manual     = $this-&gt;manual;
        $span       = $this-&gt;span;
        $stringflag = false;
        $i          = 0;
        $out        = array();
        $out[$i]    = '';

        // Loop through each token
        foreach ($tokens as $j =&gt; $token) {
            // Single char
            if (is_string($token)) {

                // Entering or leaving a quoted string
                if ($token === '&quot;' &amp;&amp; $tokens[$j - 1] !== '\\') {
                    $stringflag = !$stringflag;
                    $out[$i] .= sprintf($span, $this-&gt;highlight['string'], $token);
                } else {
                    // Skip token2color check for speed
                    $out[$i] .= sprintf($span, $this-&gt;highlight['keyword'], htmlspecialchars($token));

                    // Heredocs behave strangely
                    list($tb) = isset($tokens[$j - 1]) ? $tokens[$j - 1] : false;
                    if ($tb === T_END_HEREDOC) {
                        $out[++$i] = '';
                    }
                }

                continue;
            }

            // Proper token
            list ($token, $value) = $token;

            // Make the value safe
            $value = htmlspecialchars($value);
            $value = str_replace(
                        array_keys($this-&gt;replace),
                        array_values($this-&gt;replace),
                        $value);

            // Process
            if ($value === &quot;\n&quot;) {
                // End this line and start the next
                $out[++$i] = '';
            } else {
                // Function linking
                if ($funcref === true &amp;&amp; $token === T_STRING) {
                    // Look ahead 1, look ahead 2, and look behind 3
                    // For a function we expect T_FUNCTION T_STRING [T_WHITESPACE] (
                    if ((isset($tokens[$j + 1]) &amp;&amp; $tokens[$j + 1] === '(' ||
                        isset($tokens[$j + 2]) &amp;&amp; $tokens[$j + 2] === '(') &amp;&amp;
                        isset($tokens[$j - 3][0]) &amp;&amp; $tokens[$j - 3][0] !== T_FUNCTION
                        &amp;&amp; function_exists($value)) {

                        // Insert the manual link
                        $value = sprintf($manual, $value, $value);
                    }
                }

                // Explode token block
                $lines = explode(&quot;\n&quot;, $value);
                foreach ($lines as $jj =&gt; $line) {
                    $line = trim($line);
                    if ($line !== '') {
                        // Uncomment for debugging
                        //$out[$i] .= token_name($token);

                        // Check for plaintext
                        if ($blocks === true &amp;&amp; $token === T_INLINE_HTML) {
                            $this-&gt;_plaintextkeys[] = $i;
                            $out[$i] .= $line;
                        } else {
                            // Highlight encased strings
                            $colour = ($stringflag === true) ?
                                $this-&gt;highlight['string'] :
                                $this-&gt;_token2color($token);
                            $out[$i] .= sprintf($span, $colour, $line);
                        }
                    }


                    // Start a new line
                    if (isset($lines[$jj + 1])) {
                        $out[++$i] = '';
                    }
                }
            }
        }

        return $out;
    }


    /**
     * Convert the source to an ordered list.
     * Each line is wrapped in &lt;li&gt; tags.
     *
     * @access  public
     * @param   bool      $return    Return rather than print the results
     * @param   bool      $funcref   Reference functions to the PHP manual
     * @param   bool      $blocks    Whether to use code blocks around plaintext
     * @return  string    A HTML ordered list
     */
    function toList($return = false, $funcref = true, $blocks = true)
    {
        // Ensure source has been loaded
        if ($this-&gt;_source == false) {
            return false;
        }

        // Format list
        $source = $this-&gt;toArray($funcref, $blocks);
        $out = &quot;&lt;ol&gt;\n&quot;;
        foreach ($source as $i =&gt; $line) {
            $out .= &quot;    &lt;li&gt;&quot;;

            // Some extra juggling for lines which are not code
            if (empty($line)) {
                $out .= '&amp;nbsp;';
            } elseif ($blocks === true &amp;&amp; in_array($i, $this-&gt;_plaintextkeys)) {
                $out .= $line;
            } else {
                $out .= &quot;&lt;code&gt;$line&lt;/code&gt;&quot;;
            }

            $out .= &quot;&lt;/li&gt;\n&quot;;
        }
        $out .= &quot;&lt;/ol&gt;\n&quot;;

        if ($return === true) {
            return $out;
        } else {
            echo $out;
        }
    }


    /**
     * Convert the source to formatted HTML.
     * Each line ends with &lt;br /&gt;.
     *
     * @access  public
     * @param   bool      $return       Return rather than print the results
     * @param   bool      $linenum      Display line numbers
     * @param   string    $format       Specify format of line numbers displayed
     * @param   bool      $funcref      Reference functions to the PHP manual
     * @return  string    A HTML block of code
     */
    function toHtml($return = false, $linenum = false, $format = null, $funcref = true)
    {
        // Ensure source has been loaded
        if ($this-&gt;_source == false) {
            return false;
        }

        // Line numbering
        if ($linenum === true &amp;&amp; $format === null) {
            $format = '&lt;span&gt;%02d&lt;/span&gt; ';
        }

        // Format code
        $source = $this-&gt;toArray($funcref);
        $out = &quot;&lt;code&gt;\n&quot;;
        foreach ($source as $i =&gt; $line) {
            $out .= '    ';

            if ($linenum === true) {
                $out .= sprintf($format, $i);
            }

            $out .= empty($line) ? '&amp;nbsp;' : $line;
            $out .= &quot;&lt;br /&gt;\n&quot;;
        }
        $out .= &quot;&lt;/code&gt;\n&quot;;

        if ($return === true) {
            return $out;
        } else {
            echo $out;
        }
    }


    /**
     * Convert the source to formatted HTML blocks.
     * Each line ends with &lt;br /&gt;.
     *
     * This method ensures only PHP is between &lt;&lt;code&gt;&gt; blocks.
     *
     * @access  public
     * @param   bool      $return       Return rather than print the results
     * @param   bool      $linenum      Display line numbers
     * @param   string    $format       Specify format of line numbers displayed
     * @param   bool      $reset        Reset the line numbering each block
     * @param   bool      $funcref      Reference functions to the PHP manual
     * @return  string    A HTML block of code
     */
    function toHtmlBlocks($return = false, $linenum = false, $format = null, $reset = true, $funcref = true)
    {
        // Ensure source has been loaded
        if ($this-&gt;_source == false) {
            return false;
        }

        // Default line numbering
        if ($linenum === true &amp;&amp; $format === null) {
            $format = '&lt;span&gt;%03d&lt;/span&gt; ';
        }

        // Init
        $source     = $this-&gt;toArray($funcref, true);
        $out        = '';
        $wasplain   = true;
        $k          = 0;

        // Loop through each line and decide which block to use
        foreach ($source as $i =&gt; $line) {
            // Empty line
            if (empty($line)) {
                if ($wasplain === true) {
                    $out .= '&amp;nbsp;';
                } else {
                    if (in_array($i+1, $this-&gt;_plaintextkeys)) {
                        $out .= &quot;&lt;/code&gt;\n&quot;;

                        // Reset line numbers
                        if ($reset === true) {
                            $k = 0;
                        }
                    } else {
                        $out .= '     ';
                        // Add line number
                        if ($linenum === true) {
                            $out .= sprintf($format, ++$k);
                        }
                    }
                }

            // Plain text
            } elseif (in_array($i, $this-&gt;_plaintextkeys)) {
                if ($wasplain === false) {
                    $out .= &quot;&lt;/code&gt;\n&quot;;

                    // Reset line numbers
                    if ($reset === true) {
                        $k = 0;
                    }
                }

                $wasplain = true;
                $out .= str_replace('&amp;nbsp;', ' ', $line);

            // Code
            } else {
                if ($wasplain === true) {
                    $out .= &quot;&lt;code&gt;\n&quot;;
                }
                $wasplain = false;

                $out .= '     ';
                // Add line number
                if ($linenum === true) {
                    $out .= sprintf($format, ++$k);
                }
                $out .= $line;
            }

            $out .= &quot;&lt;br /&gt;\n&quot;;
        }

        // Add final code tag
        if ($wasplain === false) {
            $out .= &quot;&lt;/code&gt;\n&quot;;
        }

        // Output method
        if ($return === true) {
            return $out;
        } else {
            echo $out;
        }
    }


    /**
     * Assign a color based on the name of a token
     *
     * @access  private
     * @param   int     $token      The token
     * @return  string  The color of the token
     */
    function _token2color($token)
    {
        switch ($token):
            case T_CONSTANT_ENCAPSED_STRING:
                return $this-&gt;highlight['string'];
                break;

            case T_INLINE_HTML:
                return $this-&gt;highlight['html'];
                break;

            case T_COMMENT:
            case T_DOC_COMMENT:
            case T_ML_COMMENT:
                return $this-&gt;highlight['comment'];
                break;

            case T_ABSTRACT:
            case T_ARRAY:
            case T_ARRAY_CAST:
            case T_AS:
            case T_BOOLEAN_AND:
            case T_BOOLEAN_OR:
            case T_BOOL_CAST:
            case T_BREAK:
            case T_CASE:
            case T_CATCH:
            case T_CLASS:
            case T_CLONE:
            case T_CONCAT_EQUAL:
            case T_CONTINUE:
            case T_DEFAULT:
            case T_DOUBLE_ARROW:
            case T_DOUBLE_CAST:
            case T_ECHO:
            case T_ELSE:
            case T_ELSEIF:
            case T_EMPTY:
            case T_ENDDECLARE:
            case T_ENDFOR:
            case T_ENDFOREACH:
            case T_ENDIF:
            case T_ENDSWITCH:
            case T_ENDWHILE:
            case T_END_HEREDOC:
            case T_EXIT:
            case T_EXTENDS:
            case T_FINAL:
            case T_FOREACH:
            case T_FUNCTION:
            case T_GLOBAL:
            case T_IF:
            case T_INC:
            case T_INCLUDE:
            case T_INCLUDE_ONCE:
            case T_INSTANCEOF:
            case T_INT_CAST:
            case T_ISSET:
            case T_IS_EQUAL:
            case T_IS_IDENTICAL:
            case T_IS_NOT_IDENTICAL:
            case T_IS_SMALLER_OR_EQUAL:
            case T_NEW:
            case T_OBJECT_CAST:
            case T_OBJECT_OPERATOR:
            case T_PAAMAYIM_NEKUDOTAYIM:
            case T_PRIVATE:
            case T_PROTECTED:
            case T_PUBLIC:
            case T_REQUIRE:
            case T_REQUIRE_ONCE:
            case T_RETURN:
            case T_SL:
            case T_SL_EQUAL:
            case T_SR:
            case T_SR_EQUAL:
            case T_START_HEREDOC:
            case T_STATIC:
            case T_STRING_CAST:
            case T_SWITCH:
            case T_THROW:
            case T_TRY:
            case T_UNSET_CAST:
            case T_VAR:
            case T_WHILE:
                return $this-&gt;highlight['keyword'];
                break;

            case T_CLOSE_TAG:
            case T_OPEN_TAG:
            case T_OPEN_TAG_WITH_ECHO:
            default:
                return $this-&gt;highlight['default'];

        endswitch;
    }

}
?>
{% endhighlight %}

Usage is very easy, for example:

{% highlight php %}
<?php
$h = new PHP_Highlight;
$h-&gt;loadFile(__FILE__);
 
// Print source as an array
echo &quot;&lt;h3&gt;As an array&lt;/h3&gt;&quot;;
echo &quot;&lt;pre&gt;&quot;;
print_r($h-&gt;toArray());
echo &quot;&lt;/pre&gt;&quot;;
 
// Print source as an ordered list
echo &quot;&lt;h3&gt;As an ordered list&lt;/h3&gt;&quot;;
$h-&gt;toList(false);
 
// Print source as a html block
echo &quot;&lt;h3&gt;As normal HTML&lt;/h3&gt;&quot;;
$h-&gt;toHtml(false);
?>
{% endhighlight %}

This will output the contents of the current file with syntax highlighting and function referencing in the three different output formats.