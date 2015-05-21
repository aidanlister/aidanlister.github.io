---
layout: post
title: Bulk package tracking for Australia Post with Google Docs
section: developer
---
One of my companies sells <a href="https://www.kigu.me">animal onesies</a> (buy one, because they are awesome), and it imports a good quantity of goods into Australia via EMS.

I hunted around for a website that would let me instantly see the status of my packages waiting to be delivered. Australia's post website is pretty impressive, and I take my hats off to the agency responsible because they've done a good job; but I wanted an overview without clicking each box and/or reentering my package number a dozen times.

I tried a few iPhone apps, but having to type all of the numbers in one-by-one was a real pain, what I really wanted was a spreadsheet. Could Google Docs help me?

It turns out, yes it can. Google recently implemented a <a href="http://support.google.com/docs/bin/answer.py?hl=en&amp;answer=155184">importXML</a> function which lets us do all sorts of fancy things.

Column A. Enter your tracking numbers
Column B. Construct our "API" endpoint URL, <code>=CONCAT("http://auspost.com.au/track/track.html?id=",A2)</code>
Column C. Fetch the package status, <code>=importXML(B2, "//div[@class='layout-third-column']/span")</code>
Column D. Fetch the latest date/activity/location row, <code>=importXML(B2, "//div[@class='trackingDetails clearfix']//table/tbody/tr[2]")</code>

The last column will spread into Column E and F automatically. Brilliant, this works perfectly and our staff can now see the status of all our deliveries in a single glance.

Here's what it looks like:
<a href="/sites/default/files/wp-content/uploads/2012/06/google-docs-postage-tracking-e1339781804323.png"><img src="/sites/default/files/wp-content/uploads/2012/06/google-docs-postage-tracking-e1339781804323.png" alt="" title="Tracking parcel delivery with Google Docs" width="500" height="117" class="aligncenter size-full wp-image-538" /></a>