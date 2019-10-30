---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

Available in PDF format [here](/files/BenWeintraubCV.pdf)

<h1 align="center">Ben Weintraub</h1><br>

# Education
* Ph.D. in Computer Science, focus in Cybersecrity, Northeastern University, Spring 2023 (expected)
  * __Advisor__: [Professor Cristina Nita-Rotaru](http://cnitarot.github.io/)
* B.S. in Electrical Engineering, minors in Mathematics and Computer Science, University of Iowa, Spring 2013

# Research Interests
* __Topics__: security, privacy, distributed systems, networks
* __Specifics__: payment channel networks, software-defined networks, privacy-preserving routing, blockchains


Work experience
======
* __Research Assistant__, Northeastern University, Cybersecurity and Privacy Institute, May 2019-present
  * Investigating security and privacy implications of routing protocols in both cryptocurrency networks and software-defined networks (SDNs)

* __Software Engineer__, BlueTalon, 2015-2018
  * Designed and prototyped a security policy enforcement plugin for Elasticsearch queries and filed a patent for the work
  * Created a suite of scripts for automated performance testing of Hadoop MapReduce jobs
  * Created a Docker microservice architecture of 15 BlueTalon service components, and wrote scripts to initialize and orchestrate the microservice cluster
  * Wrote a Python script that configures a Cloudera Manager instance by parsing a user-defined declarative property files and applying those properties to Cloudera Manager
  * Revamped a high performance data pipeline using Elasticsearch as a sink and Kafka as a source. This offered increased record throughput from 7K records/min (in the old version) to 700K records/min
  * Architected containerized proxy solution for policy enforcement using Docker, including containerized data sources (Postgres, Impala, Hive) and filed a patent for the work
  * Designed and implemented a REST API for Policy Application using Play! Framework
  * Created easy-to-use installer for all BlueTalon components by integrating previous work with Apache Ambari and Cloudera Manager, and published an overview on the [BlueTalon](https://medium.com/bluetalon/using-ambari-to-administer-bluetalon-on-hortonworks-hadoop-687cb462b30) blog
  * Led research project geared at using Java byte-code injection for protecting data accessed from Hive CLI

* __Software Engineer__, IBM, Netezza Data Warehouse Appliances, 2013-2015
  * Wrote Hadoop codec for on-the-fly database table decompression
  * Fixed critical concurrency issue causing deadlock between producer and consumer threads on a Linux named pipe
  * Saved business relationship with $10M government client by hardening security to meet with strict compliance standards
  * Reduced processing time 4 hours, by creating a suite of Bash scripts automating security hardening
  * Reduced backlog of high-availability defects by 75%
  * Added disk health reporting feature to General Parallel File System cluster, while maintaining compatibility with Hadoop Distributed File System
  * Awarded _IBM Blue Points_ as formal recognition for a security compliance guide and accompanying testing methods
  * Selected for Engineering Response Team, resolving escalated customer issues

* __Undergraduate Research Assistant__, University of Iowa, Virtual Soldier Research Program, 2011-2013
  * Reduced render time by 99% by optimizing algorithm to facilitate 3D rendering of printed circuit boards
  * Integrated circuit-component thermal analysis into printed circuit board visualization software
  * Developed an application to display interactive, 3D models of printed circuit boards for use by the US Air Force
  * Developed method to streamline extraction process of geometric data from ISO AP 203 STEP files

Publications
======
  <ul>{% for post in site.publications %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
  
Patents
======
  <ul>{% for post in site.patents %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Skills
======
* __Languages__: Java, C/C++, Go, Bash, Clojure, MATLAB, SQL, Python, JSON, XML
* __Operating Systems__: GNU/Linux (Ubuntu, Red Hat, CentOS), Windows, Mac OSX
* __Tools__: Git, Subversion, Hadoop, Postgres, Gradle, Play! Framework, Docker, Tomcat, Maven, Jenkins, Linux-HA, GNU Make

Awards and Leadership
======
* Best Project in Distributed Systems class as voted on by peers (2018)
* VP of Membership, Toastmasters-- Stanford University chapter (2015-2016)
* Recipient, University of Iowa National Scholars Award (2008-2012)
* Recipient, Engineering Excellence Award (2009)
* Captain, Iowa Men's Ultimate Frisbee Team (2012-2013)

Membership
======
* Member, Association for Computing Machinery (ACM), July 2016-Present
* Member, Institute of Electrical and Electronics Engineers (IEEE), July 2016-Present
