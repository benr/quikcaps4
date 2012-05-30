quikcaps4
=========

<i>quikcaps4</i> is a tool for examining the resource controls of Solaris Zone ("Container").  Its purpose is to allow operators to quickly inspect multi-tenant zones on a system which may be experiencing cohabitation squabbles or improperly sized.  The tool is based on Kstats and implemented in PERL, so it runs extremely quickly and has zero impact on the running zones.



---
Example Output:

<pre>
$ quikcaps4 
+-----------+--------------------------------+------------------------+--------------------------------+-----------------+--------+---------------+
| Zone Name |          Disk (ZFS)            |      C P U             |   Resident Memory  (Rcap)      |  Virtual Memory | Uptime |  I/O Activity |
|           |    Free  >   Used   /    Quota | Used / Cap  Max  Over% |  Used  /  Cap  >  Over     VM  |  Used  /   Cap  |  Days  |  Read   Write |
|-----------+--------------------------------+------------------------+--------------------------------+-----------------|--------+---------------|
|  zone001  |    41.5G >    8.52G /      50G |   0 / 800 > 830   0.00 |  1567M / 4096M >    0K   1908M |     1G /     6G |    559 |  40.5T  1.11T |
|  zone002  |    13.1G >    1.87G /      15G |   0 / 800 > 707   0.00 |   218M / 1024M > 1280M    278M |   309M /     2G |    482 |  4.22T   489G |
|  zone003  |    14.5G >     506M /      15G |   0 / 800 > 453   0.00 |   163M / 1024M >    0K    542M |   542M /     2G |    538 |  3.48T  2.59G |
|  zone004  |    12.1G >    2.92G /      15G |   0 / 800 > 550   0.00 |   130M / 1024M >    0K    532M |   574M /     2G |    518 |  3.11T  2.13T |
|  zone005  |    14.6G >     365M /      15G |   0 / 800 > 198   0.00 |   132M / 1024M >    0K    535M |   535M /     2G |    491 |  2.87G   425M |
|  zone006  |    12.5G >    2.47G /      15G |   0 / 800 > 812   0.00 |   363M / 1024M > 1254M    744M |   744M /     2G |    449 |   170G  8.75G |
|  zone007  |    13.8G >    1.18G /      15G |   0 / 800 > 423   0.00 |   134M / 1024M >    0K    537M |   536M /     2G |    405 |  4.40G  1.72G |
|  zone008  |    10.8G >    4.22G /      15G |   0 / 800 > 345   0.00 |   244M / 1024M >    0K    577M |   577M /     2G |    394 |  15.2G  7.30G |
|  zone009  |    19.9G >    5.09G /      25G |   2 / 800 > 342   0.00 |   924M / 2048M >   18G   2599M |     2G /     4G |     86 |   690G  39.2G |
|  zone010  |    47.6G >    2.44G /      50G |   0 / 800 > 234   0.00 |   464M / 4096M >    0K   2173M |     3G /     6G |    259 |  3.40G  3.62G |
|  zone011  |    22.8G >    27.2G /      50G |   0 / 800 > 806   0.00 |  2460M / 4096M >    0K   2459M |     2G /     6G |    248 |  2.05T  4.25T |
|  zone012  |    3.37G >    6.63G /      10G |   0 / 800 > 245   0.00 |   103M /  512M >  377M    443M |   456M /     1G |    143 |   135G   947G |
|  zone013  |    36.8G >    13.2G /      50G |   0 / 800 > 811   0.00 |   450M / 1024M >   11G    849M |   849M /     2G |     96 |  1.55T   114G |
|  zone014  |     469G >    30.8G /     500G |   0 / 800 > 331   0.00 |   182M / 1024M >    0K    563M |   563M /     2G |     29 |   423T  8.29T |
+-----------+--------------------------------+------------------------+--------------------------------+-----------------+--------+---- +Joyent --+ 
</pre>

The fields here are:

* ZFS:
**  Free: Free disk space for the ZFS Dataset
**  Used: Used disk space for the ZFS Dataset
** Quota: The ZFS Dataset Quota
* CPU: (Values are in the form: 100 = 1 CPU Core)
** Used: CPU in use
** Cap: CPU Cap (800 == 8 Cores)
** Max: High Water mark on CPU for the zone; this is the most the zone has used since boot
** Over: Percentage of time since boot that the zone has exceeded its CPU Cap, this helps us determine zones which are undersized; values of anything but 0.00 indicate a problem
* Resident Memory (RCAP):
** Used: RSS memory in use
** Cap: RSS memory cap 
** Over: Ammount of cap slip (The capping daemon can't stop everything from exceeding the cap, this is the cummative count of slippage)
** VM: Estimated VM ("swap") usage as reported by rcapd
* VM:
** Used: Amount of VM Cap in use
** Cap: The VM ("Swap") Cap
* Uptime: A guess as to the uptime based on ticks
* I/O Activity:
** Read: Bytes read since boot
** Write: Bytes written since boot

---
Answers to common questions:

* If you get an error like "Attempt to access disallowed key 'cpucaps_zone_13' in a restricted hash at" this is because caps aren't set on a zone.  In the example here, CPU caps aren't defined for zone 13.
* We report VM usage twice because rcapd keeps its own stats on VM usage; the rcap VM value <i>should</i> match the "VM Used" count by the kernel, but may not aways, so we report them both.
