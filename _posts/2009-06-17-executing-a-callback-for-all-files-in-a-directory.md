---
layout: post
title: Executing a callback for all files in a directory
section: developer
---
A common question when dealing with deep directory structures concerns how a function can be applied recursively to all the files in the target directory, regardless of the depth. This function, <code>directory_walk</code>, builds an internal stack (uses no recursion) and iteratively applies the user supplied callback providing a fast and flexible approach.

{% highlight php %}
<?php
/**
 * Allows running a callback on all files in a deep directory structure
 *
 * @author Aidan Lister <aidan@php.net>
 * @param string $dirname The directory to walk
 * @param callable $callable The callable to execute on all files found
 * @param mixed $arg{n} Extra parameters to be passed to the callable
 * @return mixed The return value of the last callable run
 */
function directory_walk($dirname, $callable)
{
    $ignore = array('.', '..', '.DS_Store');
    $args = func_get_args();
    array_shift($args);

    // Sanity check    
    if (!file_exists($dirname)) {
        return false;
    }

    // Create and iterate stack
    $stack = array($dirname);
    while ($entry = array_pop($stack)) {
        if (is_link($entry)) continue;
        
        // Run the action
        if (is_file($entry)) {
          $ret = call_user_func_array($callable, array($entry) + $args);
          continue;
        }
        
        // Add the directory into the stack
        $dh = opendir($entry);
        while (false !== $child = readdir($dh)) {
            if (in_array($child, $ignore)) continue;
            $child = $entry . DIRECTORY_SEPARATOR . $child;
            $stack[] = $child;
        }
        closedir($dh);
    }

    return $ret;
}
?>
{% endhighlight %}

So, for some examples. We'll start simple and simply print the directory:

{% highlight php %}
<?php
$func = create_function('$file', 'echo "found $file";');
directory_walk('target-dir', $func);
?>
{% endhighlight %}

Which for me outputs:
<code>
found target-dir/foo
found target-dir/bar
found target-dir/baz
found target-dir/baz/ding
found target-dir/baz/dong
found target-dir/baz/dong/witch
</code>

What if we wanted to add a .txt extension to all of these files? We could write:
{% highlight php %}
<?php
function my_rename($entry, $extension) {
    if (is_file($entry)) {
        rename($entry, $entry . $extension);
    }
}
directory_walk('target-dir', 'my_rename', '.txt');
?>
{% endhighlight %}

Another example might be deleting all the .svn or CVS folders in a directory. We could write:
{% highlight php %}
<?php
function delsvn($file) {
  $ext = substr($file, strlen($file)-4,strlen($file));
  if ($ext === '.svn') rmdirr($ext);
}
directory_walk('test', 'delsvn');
?>
{% endhighlight %}

Note that the above example used the <a href="/2004/04/recursively-deleting-a-folder-in-php/">recursive delete function</a> also.

Happy stacking.