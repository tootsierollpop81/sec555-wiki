Kibana
========
Abstract
---------
Kibana is a report engine designed for Elasticsearch. It is a major open source component of the Elastic Stack. It is used to search data and visualize your logs through charts and tables as well as dashboards.

Where to Acquire
---------
Kibana can be downloaded from https://www.elastic.co/products/kibana. It is open source but also has a commercial support offering.

Search Filters
---------
Below are some of the common search filters used with Kibana.

This is an example of looking for an logs that contain the string "password":
```bash
password
```

This is an example of looking for logs that contain the name jhenderson stored in a field called user:
```bash
user:jhenderson
```

Note: Sometimes a string needs to be surrounded with double quotes.

Example:
```bash
"sec555.com"
```

This is an example of looking for logs that contain a source port greater than 40000:
```bash
source_port:>40000
```

This is an example of looking for logs that contain a destination IP between 10.0.0.0 and 10.255.255.255:
```bash
destination_ip:[10.0.0.0 TO 10.255.255.255]
```

This is an example of looking for logs that have a field named tls:
```bash
_exists_:tls
```

This is an example of looking for logs that do not have a field named tls:
```bash
_missing_:tls
```

This is an example of looking for logs that do not have a tag of pci:
```bash
-tags:pci
```

This is an example of looking for logs that are between a specific date:
```bash
@timestamp:[2017-05-01 TO 2017-05-28]
```

#### Combining search filters

Search filters can be combined using (), AND, and OR

This is an example of looking for a network connection sourcing from 192.168.0.1 going to 8.8.8.8:
```bash
source_ip:192.168.0.1 AND destination_ip:8.8.8.8
```

This is an example of looking for a network connection coming from 192.168.0.1 or 192.168.0.2:
```bash
source_ip:192.168.0.1 OR source_ip:192.168.0.2
```

This is an example of looking for a network connection coming from 192.168.0.1 or 192.168.0.2 that is destined for 8.8.8.8:
```bash
(source_ip:192.168.0.1 OR source_ip:192.168.0.2) AND destination_ip:8.8.8.8
```

This is an example of looking for network connections coming from 192.168.0.1 that are not going to 8.8.8.8:
```bash
source_ip:192.168.0.1 AND -destination_ip:8.8.8.8
```

Note: Using AND is not required when using an exclusion filter

Here is the same example as above that still works:
```bash
source_ip:192.168.0.1 -destination_ip:8.8.8.8
```

This is an example of looking for network connections that are not going to a private IP address:
```bash
-destination_ip:[10.0.0.0 TO 10.255.255.255] -destination_ip:[192.168.0.0 TO 192.168.255.255] -destination_ip:[172.16.0.0 TO 172.16.32.255.255]
```
Visualization
------
#### Example:
**Kibana Version 5.2.0**

The following assumes that you have the appropriate logs, like firewall logs, to find destination_ip, source_ip, and bytes_sent.  The names of your fields may be different.

We want to know about outbound traffic.  There are several ways to do this.  One way is to create a bar chart.  The following chart may be helpful in finding compromised endpoints.  If there is a lot of traffic from an endpoint directly to the Internet, it could signify command and control activity that could indicate that the endpoint is a part of a botnet or that the endpoint has other malware.  (I’m not interested in internal traffic to internal interfaces right now-although that could be good for detecting worm activity or lateral movement.) On the X-Axis, it would display the Top 5 Internal IP Addresses connecting outbound to IP Addresses on the Internet.  On the Y-Axis, it would have the sum of the bytes sent which is also the total amount of outbound traffic between the two endpoints.  That sounds strange, but if you think of logs, it’s a snapshot of a particular time or session-we want the total of those bytes sent between two endpoints, not just one time-point or session between them.

Log into Kibana, and click on the Visualize tab.  On the "From A New Search" column, select the "Bar Chart" Visualization.  You should be directed to a new screen.

In the filter bar, at the top of the screen in Kibana, Type:

```bash  
-destination_ip:[10.0.0.0 TO 10.255.255.255] -destination_ip:[192.168.0.0 TO 192.168.255.255] -destination_ip:[172.16.0.0 TO 172.31.255.255]
```

Press Enter.  It doesn’t actually put the filter on the data until the enter button is pressed.  This filters out private IP ranges for the destination ip address, so that it only shows Internet addresses.

On the left hand side, there are two aggregation types, one is metrics.  The other is buckets.  

On the metrics portion, click on the “Y-Axis” down arrow.
Click the elevator bar below the “Aggregation” label, and select the “Sum” option.  
Click the elevator bar below the “Field” label and select “bytes_sent”.

You may want to add a custom label so that the information is more clearly defined.  Under the “Custom Label” label, type in “Outbound Traffic” and press enter.

On the buckets portion, under the “Select buckets type” label click on “Split Rows”.  Split Rows makes the bars on the chart.  

Click the “Aggregation” elevator bar, under the “X-Axis” label, and select the “Terms” option. Click on the elevator bar below the “Field” label, and select “source_ip”.  
Click the elevator bar under “Order By” and select “metric: Outbound Traffic”.  Note:  This will be whatever you typed into the text box under “Custom Label” in metrics.
  
Click the elevator bar under the “Order” label and select “Descending”.  
Click the elevator bar under the “Size” label and select “5”.  
Again, you may want to create a custom label that describes the information more clearly.  Under the “Custom Label” label, type “Source IP”.

Now, we must add the destination IPs.  If we don’t link each source IP to a destination IP, it will simply display the total amount of bytes sent from that source IP, which could be useful in some cases, but not in this one.

Click on the “Add sub-buckets” button at the bottom of the buckets section.  
Click the “Split Bars” option under the “Select buckets type” label. Visually, this will split each bar on the bar chart into different colored sections.  Each of these different sections will represent a different destination IP.  Some of them won’t have much traffic, so you won’t see much visually, but those that do have a lot of traffic between them will stick out like a sore thumb.

Click on the elevator bar under the “Sub Aggregation” label, Select “Terms”.

Click on the elevator bar under the “Field” label, Select “destination_ip”.

Click on the elevator bar under the “Order By” label, and select “metric:Outbound Traffic”.  Note:  This will be whatever you typed into the text box under “Custom Label” in metrics.

Click the elevator bar under the “Order” label and select “Descending”.  
Click the elevator bar under the “Size” label and select “5”.  (This doesn't have to be 5.  You can select 3 or 10, just note that the bar will be split up into however many sections you choose based on this number.)

Again, you may want to create a custom label that describes the information more clearly.  Under the “Custom Label” label, type “Destination IP”.

If you would like change the appearance of the bar chart, those options can be found in the "options" tab above the “metrics” label.

Click the play button at the top right, underneath the elevator bar where you select the log that you want to work on.

Assuming you have logs from traffic in the timeframe selected, you should see a bar chart.  

If you don’t, see a bar chart, click on the time clock icon in the far right, of the bar that has “New, Save, Open, Share, Refresh, Reporting” options.  Then select a different time period.  This week should be a good option.  You should see a bar chart showing the outbound traffic to the Internet.
