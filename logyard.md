% Logyard architecture
%
% April 18, 2013

*This was originally posted in
 [ActiveState blog](http://www.activestate.com/blog/2013/04/logyard-why-and-how-stackatos-logging-system)
 using the title "Logyard: The Why and How of Stackato's Logging
 System" describing the architecture of the logging system
 ([Logyard](https://github.com/activestate/logyard)) I wrote during my
 time at ActiveState [2009-2014].*

Early on in the development of [Stackato](http://www.activestate.com/stackato), we recognized that good logging features would be critical to developers and administrators. We also knew that we didn&rsquo;t want to reinvent the wheel, and made the intentional decision _not_ to build log analysis and aggregation tools into the product itself.

There are quite a few log aggregators and indexers available already ([Splunk](http://www.splunk.com), [Loggly](http://loggly.com), [Papertrail](http://papertrailapp.com), [Logstash](http://logstash.net/), et al) and many developers and sysadmins have already made a significant commitment to using their favorite one.

Stackato's logging subsystem is called Logyard, and its purpose is to facilitate treating logs as event streams. Nothing else.

Logyard works _with_ those tools, it does not replace them. By treating logs as event streams, Stackato lets users work with their logs using the powerful tools they are already familiar with.

This post provides a look behind the scenes at Logyard's design, and shows how the design allows users maximum flexibility in analyzing and managing their logs.

## Logyard core

  ![Logyard Design](/images/logyard-design-diagram.png "Logyard Design")

Central to everything is the Logyard &ldquo;core&rdquo;, which acts like a message bus accepting only clients from the local machine (this is important for a design with no single point of failure). Under [kato process list](http://docs.stackato.com/3.2/admin/reference/kato-ref.html#kato-command-ref-process-list), the `logyard` process represents Logyard core.

Logyard core uses [zeromq](http://www.zeromq.org/) to forward events. Unlike many message bus systems, zeromq is brokerless (which readily ensures a no-SPoF design).

## Systail

Systail is the second component, and it acts as one of the Logyard clients. A Logyard client publishes events to Logyard core (all of this happens on the same node - never on an external node). The purpose of systail is to `tail -f` stackato log files (under _/s/logs/_) and send the lines to Logyard core (via zeromq) on that node.

In a Stackato cluster, every node will run its own Logyard core and systail processes, independent of each other. 

There are other Logyard clients similar to systail: apptail and cloud_events. We'll discuss them later.

## Drains

Logyard also has the concept of [drains](http://docs.stackato.com/admin/server/logging.html#drains). Logyard core can be told to set up a drain that acts like a bridge between Logyard core and an arbitrary third-party service. The drain channels log events on that node to the specified drain target, which could be a third-party service. For example, a Papertrail drain would channel log events to the logs.papertrailapp.com UDP service.

**Edit** (2014-10-5): *We have moved away from doozer to redis for storing logyard drains.*

Logyard uses [doozer](http://www.activestate.com/blog/2012/08/doozer-distributed-configuration-used-heroku-and-stackato)
  for configuration and keeping the list of drains. Admins and users can add or remove drains. When they add a drain, the following
  happens:

1.  The new drain URI is written to doozer
2.  Logyard core (on all nodes across the cluster) gets notified of this change in doozer
3.  Logyard core starts a new drain to channel the specified events from that node to the specified drain target.

This happens on every node in the cluster, so effectively we are channeling logs from across the cluster.

The &ldquo;drain URI&rdquo; contains:

1.  the protocol (e.g.: tcp)
2.  the host and port
3.  drain options, including [format](http://docs.stackato.com/admin/server/logging.html#log-format)

For example: `udp://logs.papertrailapp.com:12345/?filter=systail.dea`

A URI contains the entire configuration necessary for that drain.

If the drain target (third-party service) stops responding, Logyard will retry that drain [periodically with decreasing frequency](http://docs.stackato.com/admin/server/logging.html#logging-drains-timeouts).

Currently, Logyard supports the following drain types (protocols):

*   tcp
*   udp
*   redis (stores events in a redis LIST data structure, bounded size)
*   file (local file)

## Apptail

We mentioned that systail is not the only Logyard client. The other clients are apptail and cloud_events.

Apptail acts like systail, but works on application logs stored in application containers (LXC). Apptail monitors the NATS message bus for new application instances being started (as a result of `push/ update/ restart/ HM` or whatever) and, if a new instance was started, it begins tailing its _logs/stdout.log_ and _logs/stderr.log_ files, sending the resultant lines to Logyard core via zeromq, just like systail.

## Interlude

Is it becoming clear as to how Logyard treats logs as streams?

First, it never conceptually differentiates between system logs (systail) and application logs (apptail) even though their location and permanence differs radically.

Second, it presents these log data as homogenous &ldquo;streams&rdquo;. When a client sends an event to Logyard core, it will identify that event with a key. This key can be specified when adding a drain, telling it to filter messages by the key. What this means is that only messages with that key prefix (note: key prefix, not key) will be sent to the drain target. For example, systail running on 10.1.5.34 and seeing a line from _dea.log_ will send that log line using the key &ldquo;systail.dea.10.1.5.34&rdquo;. When a new drain is added, all the following keys are valid, and they all will include that particular log line:

*   systail
*   systail.dea
*   systail.dea.10

## Cloud Events

**cloud_events** is another Logyard client, a special one, because it both sends to and receives from Logyard core. cloud_events receives the &ldquo;systail&rdquo; stream (from that node only; remember, Logyard never streams across nodes), parses it to look for special events, and sends those parsed/formatted events back to Logyard core under a different key called &ldquo;event&rdquo;, for example &ldquo;event.dea_start&rdquo;.

All of these processes run on all nodes in the cluster (as part of the &ldquo;base&rdquo; role), except for apptail which only runs on the nodes with the DEA or Stager role.

The [`kato log tail`](http://docs.stackato.com/reference/kato-ref.html#kato-command-ref-log-tail) command leverages Logyard &ndash; not as a Logyard client, but as a drain. It starts a TCP server and adds itself as a drain listening for the &ldquo;systail&rdquo; stream. We made `kato log tail` a drain instead of client because a client can only send-to/receive-from the local Logyard core process, whereas a single drain target will be sent the log stream from the Logyard core process in all nodes in the cluster. `kato log tail` is supposed to be tailing logs from across the cluster.

Just like `kato log tail`, we have three built-in drains; seen from `kato log drain list`:


    builtin.apptail      redis://stackato-core:6464/?filter=apptail&amp;limit=400
    builtin.cloudevents  redis://stackato-core:6464/?filter=event&amp;limit=256&amp;key=cloud_events
    builtin.katohistory  redis://stackato-core:6464/?filter=event.kato_action&amp;limit=256&amp;key=kato_history

1.  **builtin.apptail** stores the most recent logs from all applications in Redis, which will in turn be consumed by `stackato logs` etc.
2.  **builtin.cloudevents** likewise stores the last 256 events from the cloud events stream in redis, which will in turn be consumed by the WebUI cloud events.
3.  **builtin.katohistory** is similar to the above, but only stores cloud events with &ldquo;event.kato_action&rdquo; prefix, thus effectively allowing the &ldquo;kato history&rdquo; command to print kato commands run from across the cluster.

Hopefully, this gives the curious a look behind the curtain at the internals of Logyard. A regular end user, and even most admins, will generally not have to worry about the how the system works at this level of detail. Following the documentation for [application logs](http://docs.stackato.com/deploy/app-logs.html), [system logs](http://docs.stackato.com/server/logging.html), and the [examples for using drains with external services](http://docs.stackato.com/best-practices/logging-examples.html), typical logging operations should &ldquo;just work&rdquo;.

