---
layout: post
title: Recursively deleting a folder in PHP
section: developer
---
PHP's <code>rmdir</code> function does not allow the deletion of a folder if it is not empty. The <code>rmdirr</code> function below allows the removal of a folder and all its contents recursively, as would <code>rm -rf</code> on linux.

{% highlight php %}
<?php
/**
 * Delete a file, or a folder and its contents (recursive algorithm)
 *
 * @author      Aidan Lister <aidan@php.net>
 * @version     1.0.3
 * @link        http://aidanlister.com/2004/04/recursively-deleting-a-folder-in-php/
 * @param       string   $dirname    Directory to delete
 * @return      bool     Returns TRUE on success, FALSE on failure
 */
function rmdirr($dirname)
{
    // Sanity check
    if (!file_exists($dirname)) {
        return false;
    }
 
    // Simple delete for a file
    if (is_file($dirname) || is_link($dirname)) {
        return unlink($dirname);
    }
 
    // Loop through the folder
    $dir = dir($dirname);
    while (false !== $entry = $dir->read()) {
        // Skip pointers
        if ($entry == '.' || $entry == '..') {
            continue;
        }
 
        // Recurse
        rmdirr($dirname . DIRECTORY_SEPARATOR . $entry);
    }
 
    // Clean up
    $dir->close();
    return rmdir($dirname);
}
?>
{% endhighlight %}

All good code should include tests, so here we go:

{% highlight php %}
<?php
// Create a directory and file tree
mkdir('testdelete');
mkdir('testdelete/one-a');
touch('testdelete/one-a/testfile');
mkdir('testdelete/one-b');

// Add some hidden files for good measure
touch('testdelete/one-b/.hiddenfile');
mkdir('testdelete/one-c');
touch('testdelete/one-c/.hiddenfile');

// Add some more depth
mkdir('testdelete/one-c/two-a');
touch('testdelete/one-c/two-a/testfile');
mkdir('testdelete/one-d/');

// Test that symlinks are not followed
mkdir('testlink');
touch('testlink/testfile');
symlink(getcwd() . '/testlink/testfile', 'testdelete/one-d/my-symlink');
symlink(getcwd() . '/testlink', 'testdelete/one-d/my-symlink-dir');

// Run the actual delete
$status = rmdirr('testdelete');

// Check if we passed the test
if ($status === true &&
        !file_exists('testdelete') &&
        file_exists('testlink/testfile')) {
    echo 'TEST PASSED';
    rmdirr('testlink');
} else {
    echo 'TEST FAILED';
}
?>
{% endhighlight %}

This simple testing script should yield a "TEST PASSED" message when run from your browser or command line.

Although I worry that the paradigm-of-choice may overwhelm some readers, in the interests of completeness I have also included a stack based (instead of recursion based) version of <code>rmdirr</code> below:

{% highlight php %}
<?php
/**
 * Delete a file, or a folder and its contents (stack algorithm)
 *
 * @author      Aidan Lister <aidan@php.net>
 * @version     1.0.0
 * @link        http://aidanlister.com/repos/v/function.rmdirr.php
 * @param       string   $dirname    Directory to delete
 * @return      bool     Returns TRUE on success, FALSE on failure
 */
function rmdirr($dirname)
{
    // Sanity check
    if (!file_exists($dirname)) {
        return false;
    }

    // Simple delete for a file
    if (is_file($dirname) || is_link($dirname)) {
        return unlink($dirname);
    }
    
    // Create and iterate stack
    $stack = array($dirname);
    while ($entry = array_pop($stack)) {
        // Watch for symlinks
        if (is_link($entry)) {
            unlink($entry);
            continue;
        }
        
        // Attempt to remove the directory
        if (@rmdir($entry)) {
            continue;
        }
        
        // Otherwise add it to the stack
        $stack[] = $entry;
        $dh = opendir($entry);
        while (false !== $child = readdir($dh)) {
            // Ignore pointers
            if ($child === '.' || $child === '..') {
                continue;
            }
            
            // Unlink files and add directories to stack
            $child = $entry . DIRECTORY_SEPARATOR . $child;
            if (is_dir($child) && !is_link($child)) {
                $stack[] = $child;
            } else {
                unlink($child);
            }
        }
        closedir($dh);
        print_r($stack);
    }
    
    return true;
}
?>
{% endhighlight %}

Rather than the function calling itself once for each directory, this function maintains an internal stack. Because this function uses the <a href="http://php.net/operators.errorcontrol">error suppression operator</a> it incurs a speed penalty; however, it will work for very deep directory structures.

The recursive method can cause <a href="http://ilia.ws/archives/5_Top_10_ways_to_crash_PHP.html">PHP to segfault</a> when PHP's own internal stack grows too large (stack overflow), but this is only a problem if you have directories nested 1000+ layers deep.

Summary: use the recursive method (listed first) unless you really don't like recursion, or have a very deep tree structure. Happy deleting!