---
layout: post
title: Retrieve time from an NTP server 
section: developer
summary: |
    <code>ntp_time()</code> allows you to quickly retrieve the time from an NTP server as a unix timestamp.
---
There's a lot of code floating around the internet to communicate with NTP servers, unfortunately none of the ones I found actually worked. Here's a low level implementation:

{% highlight php %}
<?php
/**
 * Retrieve time from an NTP server
 *
 * @param    string   $host   The NTP server to retrieve the time from
 * @return   int      The current unix timestamp
 * @author   Aidan Lister <aidan@php.net>
 * @link     http://aidanlister.com/2010/02/retrieve-time-from-an-ntp-server/
 */
function ntp_time($host) {
  
  // Create a socket and connect to NTP server
  $sock = socket_create(AF_INET, SOCK_DGRAM, SOL_UDP);
  socket_connect($sock, $host, 123);
  
  // Send request
  $msg = "\010" . str_repeat("\0", 47);
  socket_send($sock, $msg, strlen($msg), 0);
  
  // Receive response and close socket
  socket_recv($sock, $recv, 48, MSG_WAITALL);
  socket_close($sock);

  // Interpret response
  $data = unpack('N12', $recv);
  $timestamp = sprintf('%u', $data[9]);
  
  // NTP is number of seconds since 0000 UT on 1 January 1900
  // Unix time is seconds since 0000 UT on 1 January 1970
  $timestamp -= 2208988800;
  
  return $timestamp;
}
?>
{% endhighlight %}

You can use any NTP host you want, ideally something local to you. Otherwise, you can use a random server from the <a href="http://www.pool.ntp.org">NTP Server Pool</a>.