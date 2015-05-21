---
layout: post
title: Highlighting a search string in HTML text
section: developer
---
Several hundred user notes on the str_replace page later, and we have a function that provides a variety of text highlighting options for plaintext or HTML strings.

This function, for example, would allow you to parse a HTML document and wrap certain words in HTML tags to direct the users attention.

{% highlight php %}
<?php
/**
 * Perform a simple text replace
 * This should be used when the string does not contain HTML
 * (off by default)
 */
define('STR_HIGHLIGHT_SIMPLE', 1);

/**
 * Only match whole words in the string
 * (off by default)
 */
define('STR_HIGHLIGHT_WHOLEWD', 2);

/**
 * Case sensitive matching
 * (off by default)
 */
define('STR_HIGHLIGHT_CASESENS', 4);

/**
 * Overwrite links if matched
 * This should be used when the replacement string is a link
 * (off by default)
 */
define('STR_HIGHLIGHT_STRIPLINKS', 8);

/**
 * Highlight a string in text without corrupting HTML tags
 *
 * @author      Aidan Lister <aidan@php.net>
 * @version     3.1.1
 * @link        http://aidanlister.com/2004/04/highlighting-a-search-string-in-html-text/
 * @param       string          $text           Haystack - The text to search
 * @param       array|string    $needle         Needle - The string to highlight
 * @param       bool            $options        Bitwise set of options
 * @param       array           $highlight      Replacement string
 * @return      Text with needle highlighted
 */
function str_highlight($text, $needle, $options = null, $highlight = null)
{
    // Default highlighting
    if ($highlight === null) {
        $highlight = '<strong>\1</strong>';
    }

    // Select pattern to use
    if ($options & STR_HIGHLIGHT_SIMPLE) {
        $pattern = '#(%s)#';
        $sl_pattern = '#(%s)#';
    } else {
        $pattern = '#(?!<.*?)(%s)(?![^<>]*?>)#';
        $sl_pattern = '#<a\s(?:.*?)>(%s)</a>#';
    }

    // Case sensitivity
    if (!($options & STR_HIGHLIGHT_CASESENS)) {
        $pattern .= 'i';
        $sl_pattern .= 'i';
    }

$needle = (array) $needle;
foreach ($needle as $needle_s) {
        $needle_s = preg_quote($needle_s);

        // Escape needle with optional whole word check
        if ($options & STR_HIGHLIGHT_WHOLEWD) {
            $needle_s = '\b' . $needle_s . '\b';
        }

        // Strip links
        if ($options & STR_HIGHLIGHT_STRIPLINKS) {
            $sl_regex = sprintf($sl_pattern, $needle_s);
            $text = preg_replace($sl_regex, '\1', $text);
        }

        $regex = sprintf($pattern, $needle_s);
$text = preg_replace($regex, $highlight, $text);
}

    return $text;
}
?>
{% endhighlight %}

Let's do a quick example:
{% highlight php %}
<?php
// Simple Example
$string = 'This is a site about PHP and SQL';
$search = array('php', 'sql');
echo str_highlight($string, $search);
echo "\n";
 
// With HTML in the text
$string = 'Link to <a href="php">php</a>';
$search = 'php';
echo htmlspecialchars(str_highlight($string, $search));
echo "\n";
 
// Matching whole words only
$string = 'I like to eat bananas with my nana!';
$search = 'Nana';
echo str_highlight($string, $search, STR_HIGHLIGHT_SIMPLE|STR_HIGHLIGHT_WHOLEWD);
echo "\n";
 
// With custom highlighting
$string = 'With custom highlighting!';
$search = 'custom';
$highlight = '<span style="text-decoration: underline;">\1</span>';
echo str_highlight($string, $search, STR_HIGHLIGHT_SIMPLE, $highlight);
echo "\n";
 
// With links
$string = 'I am a <a href="http://www.php.net">link</a>';
$search = 'link';
$highlight = '<a href="http://www.google.com/">\1</a>';
echo htmlspecialchars(str_highlight($string, $search, STR_HIGHLIGHT_STRIPLINKS, $highlight));
?>
{% endhighlight %}

This code would produce the following output:
<code>
This is a site about <strong>PHP</strong> and <strong>SQL</strong>
Link to <a href="/php/"><strong>php</strong></a>
I like to eat bananas with my <strong>nana</strong>!
With <span style="text-decoration: underline;">custom highlighting</span>!
I am a <a href="http://www.google.com/">link</a>
</code>