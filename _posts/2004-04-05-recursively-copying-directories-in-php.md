---
layout: post
title: Recursively copying directories in PHP
section: developer
---
This function allows you to copy an entire directory tree recursively. Symlinks on both operating systems are also accommodated for.

{% highlight php %}
<?php
/**
 * Copy a file, or recursively copy a folder and its contents
 *
 * @author      Aidan Lister <aidan@php.net>
 * @version     1.0.1
 * @link        http://aidanlister.com/2004/04/recursively-copying-directories-in-php/
 * @param       string   $source    Source path
 * @param       string   $dest      Destination path
 * @return      bool     Returns TRUE on success, FALSE on failure
 */
function copyr($source, $dest)
{
    // Check for symlinks
    if (is_link($source)) {
        return symlink(readlink($source), $dest);
    }
    
    // Simple copy for a file
    if (is_file($source)) {
        return copy($source, $dest);
    }

    // Make destination directory
    if (!is_dir($dest)) {
        mkdir($dest);
    }

    // Loop through the folder
    $dir = dir($source);
    while (false !== $entry = $dir->read()) {
        // Skip pointers
        if ($entry == '.' || $entry == '..') {
            continue;
        }

        // Deep copy directories
        copyr("$source/$entry", "$dest/$entry");
    }

    // Clean up
    $dir->close();
    return true;
}
?>
{% endhighlight %}

Just to make sure this function works perfectly, we can do some testing:

{% highlight php %}
<?php
// Create a directory and file tree
mkdir('testcopy');
mkdir('testcopy/one-a');
touch('testcopy/one-a/testfile');
mkdir('testcopy/one-b');

// Add some hidden files for good measure
touch('testcopy/one-b/.hiddenfile');
mkdir('testcopy/one-c');
touch('testcopy/one-c/.hiddenfile');

// Add some more depth
mkdir('testcopy/one-c/two-a');
touch('testcopy/one-c/two-a/testfile');
mkdir('testcopy/one-d/');

// Test that symlinks are created properly
mkdir('testlink');
touch('testlink/testfile');
symlink(getcwd() . '/testlink/testfile', 'testcopy/one-d/my-symlink');
symlink(getcwd() . '/testlink', 'testcopy/one-d/my-symlink-dir');
symlink('../', 'testcopy/one-d/my-symlink-relative');


$status = copyr('testcopy', 'testcopy-copy');

if (file_exists('testcopy-copy')
        && file_exists('testcopy-copy/one-b/.hiddenfile')
        && file_exists('testcopy-copy/one-c/two-a/testfile')
        && is_link('testcopy-copy/one-d/my-symlink-relative')
        && (readlink('testcopy-copy/one-d/my-symlink-relative') == '../')
        && is_link('testcopy-copy/one-d/my-symlink')) {
    echo "TEST PASSED";
} else {
    echo "TEST FAILED";
}
?>
{% endhighlight %}

If everything works as expected, you should see a "TEST PASSED".