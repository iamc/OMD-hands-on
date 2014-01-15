****************************************************************************
System monitoring with Open Monitoring Distribution (OMD): hands-on tutorial
****************************************************************************


:Autor: Iñigo Aldazabal Mensa <inigo_aldazabal@ehu.es>


Abstract
########
 
Some kind of `system monitoring`_ software is essential in any system administrator's toolkit. It allows us being alerted when (or before) problems occur, getting a general overview of the system's health and also examining resource trends and historical records.

Being `Nagios`_ the "de facto" IT monitoring solution, it is well known both for its power and extensibility as well as for  its notorious difficulty to setup and configure. The `Open Monitoring Distribution (OMD)`_ offers us a pre-packaged, fully working Nagios system which, built on top on Nagios and the `Check_MK`_ ecosystem, and together with many other standard Nagios plugins/extensions, makes the installation, setup and maintenance of a full monitoring solution a trivial task.

In this hands-on tutorial we will start from two bare CentOs virtual machines and carry out a step by step procedure which will take us from a zero configuration to a full monitoring system with email notifications, graphs, trends, etc. where one of the machines will become the Nagios/OMD server monitoring the status of both of them. This procedure, for which detailed notes will be provided, should allow us to set up, in a similar fashion, a full monitoring infrastructure for a regular sized (~10-100s of nodes) HPC cluster in a matter of hours.


.. _`System monitoring`: http://en.wikipedia.org/wiki/System_monitor
.. _`Nagios`: http://www.Nagios.org/
.. _`check_mk`: http://mathias-kettner.com/check_mk.html
.. _`Open Monitoring Distribution (OMD)`: http://omdistro.org/


:Targeted audience: system administrators interested in system monitoring
:Content level: beginner/intermediate
:Audience prerequisites: basic GNU/Linux administration: command line and configuration files basic knowledge, minimal VirtualBox usage, familiarity with package managemente and networking concepts, etc. 
:Detailed outline: See table of contents of the full procedure in next page.


License
#######

This work is licensed under a `Creative Commons Attribution 4.0 International License`_.


