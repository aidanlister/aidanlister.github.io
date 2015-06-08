---
layout: post
title: Better Error Handling with CakePHP
section: developer
summary: |
    CakePHP is a wonderful framework, but it really drops the ball when it comes to practical error management. In production environments (DEBUG = 0), only 404 or 500 errors are displayed to the user, and no errors are written to the log files.
    
    To solve these two problems we override php's error handler to enable production error logging, and cake's error handler to allow forward facing error pages.
---
CakePHP is a wonderful framework, but it really drops the ball when it comes to practical error management. In production environments (DEBUG = 0), only 404 or 500 errors are displayed to the user, and no errors are written to the log files.

This means runtime errors (e.g. an unexpected divide-by-zero) are not logged, and service errors (e.g. when a paypal checkout fails) are not explained to the user.

To solve these two problems we override php's error handler to enable production error logging, and cake's error handler to allow forward facing error pages.

Part 1: To enable production error logging, we can override cake's production error handling code by conditionally setting DISABLE_DEFAULT_ERROR_HANDLING. This code only kicks in when cake is in production and logs all notices and warnings in the cake log file.

{% highlight php %}
<?php
app/config/bootstrap.php
<?php
/**
 * Handle logging errors in production mode
 */
if (Configure::read() === 0) {
    // Disable the default handling and include logger
    define('DISABLE_DEFAULT_ERROR_HANDLING', 1);
    uses('cake_log');
    error_reporting(E_ALL);
    
    /**
     * A function to directly log errors
     *
     * @param $errno The error number
     * @param $errstr The error description
     * @param $errfile The file where the error occured
     * @param $errline The line of the file where the error occured
     * @return bool Success
     */
    function productionError($errno, $errstr, $errfile, $errline) {
        // Ignore E_STRICT and suppressed errors
        if ($errno === 2048 || error_reporting() === 0) {
            return;
        }
        
        // What type of error
        $level = LOG_DEBUG;
        switch ($errno) {
            case E_PARSE:
            case E_ERROR:
            case E_CORE_ERROR:
            case E_COMPILE_ERROR:
            case E_USER_ERROR:
                $error = 'Fatal Error';
                $level = LOG_ERROR;
            break;
            case E_WARNING:
            case E_USER_WARNING:
            case E_COMPILE_WARNING:
            case E_RECOVERABLE_ERROR:
                $error = 'Warning';
                $level = LOG_WARNING;
            break;
            case E_NOTICE:
            case E_USER_NOTICE:
                $error = 'Notice';
                $level = LOG_NOTICE;
            break;
            default:
                return false;
            break;
        }

        // Log
        CakeLog::write($level, sprintf('%s (%d): %s in [%s, line %d]',
            $error, $errno, $errstr, $errfile, $errline));
        
        // Die if fatal
        if ($level === LOG_ERROR) {
            die();
        }
        
        return true;
    }
    
    // Use the above handling
    set_error_handler('productionError');
}
?>
?>
{% endhighlight %}

Part 2: To enable forward facing user error pages, we can define an AppError class. This code allows you to set up custom error handlers, and decide how you want to handle each - be it publicly displayed, logged or mailed to the site managers.

{% highlight php %}
<?php
app/app_error.php:
<?php
class AppError extends ErrorHandler
{   
    /**
     * List of errors which are displayed, even in production mode
     */
    var $displayErrors = array('logic', 'paypal', 'payflow');
    
    /**
     * List of errors which, when occur, information is emailed to
     * the site administrator
     */
    var $emailErrors   = array();
    
    /** 
     * List of errors which, when occur, will result in log entries
     */
    var $logErrors     = array('logic', 'system');
    
    /** 
     * A string containing people to be notified in the event of an
     * error. The string must be acceptable input for the mail function
     */
    var $siteManager   = 'aidan@php.net';
    
    /**
     * Override the default cakeError error handling behaviour
     *
     * By setting the debug switch, the page will be publically visisble
     * Alternatively, or in conjunction with, we can log and notify the
     * site owner
     */
    function __construct($method, $messages)
    {
        if (in_array($method, $this->displayErrors)) {
            Configure::write('debug', 1);
        }
        
        if (in_array($method, $this->logErrors)) {
            $bt = debug_backtrace();
            $errfile = $bt[1]['file'];
            $errline = $bt[1]['line'];
            $parameters = str_replace("\n", '',
                print_r($messages, true));
            $error = sprintf('%s: Called with parameters (%s) in [%s, line %d]',
                $method, $parameters, $errfile, $errline);
            CakeLog::write(LOG_ERROR, $error);        
        }
        
        if (in_array($method, $this->emailErrors)) { 
            $this->_notify($method, $messages);
        }
        
        // Handle as normal
        parent::__construct($method, $messages);
    }
    
    /**
     * Send out email when an error occurs
     */
    function _notify($method, $messages)
    {
        $subject  = '[CakePHP] Site Error';
        $headers  = 'From: cakephp@' . $_SERVER['HTTP_HOST'] . "\r\n";
        $headers .= 'Reply-To: cakephp@' . $_SERVER['HTTP_HOST'];
        $message  = 'An occurred error at ' . $_SERVER['HTTP_HOST'] . ".\n\n";
        foreach ($messages as $key => $value) {
            $message .= sprintf("    %s: %s\n", $key, $value);
        }
        
        mail($this->siteManager, $subject, $message, $headers);
    }

    /**
     * A fatal logic error occured
     *
     * @param $params Expects keys [message]
     */
    function logic($params)
    {
        $this->controller->set('message', $params['message']);
        $this->_outputMessage('logic');
    }
}
?>
?>
{% endhighlight %}

Both of these solutions are independent of each other, and provide a way to practically manage error handling in your CakePHP deployment.

I have also opened a <a href="https://trac.cakephp.org/ticket/6165">RFC</a> to get this behavior included in cake's core.