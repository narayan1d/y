# Understanding BGP Peering with "show ip bgp summary": A Comprehensive Guide

Border Gateway Protocol (BGP) is the routing protocol that glues the internet together. It's the protocol that allows Autonomous Systems (ASes) – distinct networks under a single administrative domain – to exchange routing information and determine the best paths to reach destinations across the globe. Understanding BGP is crucial for network engineers involved in large enterprise networks, service provider environments, or anyone aiming for a deeper knowledge of internet infrastructure.

Are you looking to master BGP and become a routing expert? For a limited time, I'm offering a free course on BGP fundamentals! Download it now and jumpstart your networking journey: [BGP Fundamentals Free Download](https://udemywork.com/show-ip-bgp-summary)

One of the most frequently used commands in troubleshooting and monitoring BGP is `show ip bgp summary`. This command provides a concise overview of the BGP routing process, including peering relationships, route counts, and overall BGP health. This article will delve into the details of the `show ip bgp summary` command, explaining each section of the output and demonstrating how it can be used to diagnose and resolve common BGP issues.

## Dissecting the `show ip bgp summary` Output

The `show ip bgp summary` command, when executed on a Cisco (or similar) router, displays vital information about the BGP process.  Let's break down the typical output and understand each field:

```
Router# show ip bgp summary
BGP router identifier x.x.x.x, local AS number 65001
BGP table version is 3, main routing table version 3
3 network entries and 2 paths using 380 bytes of memory
BGP attribute entries: 1, route-map cache entries: 0, AS-path list entries: 0
BGP activity 4/0 prefixes, 4/0 paths, scan interval 60 secs

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.1.2     4 65002     130     125        3    0    0 00:10:20        1
192.168.1.3     4 65003     128     123        3    0    0 00:08:55        2
```

Here's a breakdown of each section:

**1. BGP router identifier x.x.x.x, local AS number 65001:**

*   **BGP router identifier:** This is a 32-bit number (usually an IP address) that uniquely identifies the BGP router within the AS.  It's often the highest loopback address of the router. This identifier is used to prevent routing loops.
*   **local AS number:** This indicates the Autonomous System Number (ASN) that the router belongs to.  ASNs are assigned to organizations that have a single, clearly defined routing policy.

**2. BGP table version is 3, main routing table version 3:**

*   **BGP table version:**  This reflects the current version of the BGP routing table.  Each time the BGP table is updated, the version number is incremented.  Discrepancies between peer table versions can indicate synchronization issues.
*   **main routing table version:** This indicates the version of the main routing table (RIB - Routing Information Base) that BGP uses. Ideally, the BGP table version and the main routing table version should be in sync.

**3. 3 network entries and 2 paths using 380 bytes of memory:**

*   **network entries:** The number of unique network prefixes (destinations) that BGP knows about.
*   **paths:** The number of distinct paths to reach those networks. BGP can learn multiple paths to the same network, and this indicates the number of such paths being tracked.  The `bytes of memory` figure provides an indication of the resources BGP is using.

**4. BGP attribute entries: 1, route-map cache entries: 0, AS-path list entries: 0:**

*   This section shows resource utilization related to BGP attributes (such as MED, Local Preference, Community), route-maps (policy-based routing rules), and AS-path lists (used to filter routes based on the AS path). Higher numbers can indicate more complex BGP configurations and potentially higher resource consumption.

**5. BGP activity 4/0 prefixes, 4/0 paths, scan interval 60 secs:**

*   **prefixes:** Indicates the number of prefixes advertised and withdrawn since the last scan interval. The first number is prefixes advertised, and the second is prefixes withdrawn. In our example, 4 prefixes were advertised and 0 withdrawn since the last scan interval.
*   **paths:** Similar to prefixes, shows paths advertised and withdrawn.
*   **scan interval:** The frequency (in seconds) at which BGP scans its routing table.

**6. Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd:**

This is the most important part of the output, providing information about each BGP neighbor (peer):

*   **Neighbor:** The IP address of the BGP neighbor.
*   **V:** The BGP version being used with the neighbor (typically 4).
*   **AS:** The Autonomous System Number (ASN) of the neighbor.
*   **MsgRcvd:** The number of BGP messages received from the neighbor.
*   **MsgSent:** The number of BGP messages sent to the neighbor.  Discrepancies between these numbers can indicate communication problems.
*   **TblVer:** The BGP table version that the neighbor is using. It should match the local BGP table version. Mismatched versions often point to routing inconsistencies.
*   **InQ:** The number of messages in the input queue (waiting to be processed from the neighbor). Non-zero values may indicate processing delays or network congestion.
*   **OutQ:** The number of messages in the output queue (waiting to be sent to the neighbor). Similar to InQ, non-zero values suggest potential problems.
*   **Up/Down:** The amount of time the BGP peering session has been established (if the session is up) or the amount of time since the session went down (if the session is down). This is presented in `dd:hh:mm:ss` format (days:hours:minutes:seconds).
*   **State/PfxRcd:** This field is critical for understanding the status of the BGP peering session.  Common states include:

    *   **Idle:** BGP is not actively trying to establish a connection with the neighbor.
    *   **Connect:** BGP is attempting to establish a TCP connection with the neighbor.
    *   **Active:** BGP has initiated a TCP connection to the neighbor, but has not yet received a response.
    *   **OpenSent:** BGP has sent an OPEN message to the neighbor but has not received one back.
    *   **OpenConfirm:** BGP has received an OPEN message from the neighbor and is waiting to confirm it.
    *   **Established:** The BGP peering session is fully established and routing information is being exchanged.  In the `Established` state, this field will instead show the number of prefixes received from the neighbor (PfxRcd).

## Using `show ip bgp summary` for Troubleshooting

The `show ip bgp summary` command is invaluable for diagnosing BGP issues. Here are some common scenarios:

*   **Neighbor in Idle state:** This usually indicates a configuration problem, such as an incorrect neighbor IP address or AS number. Check the BGP configuration and ensure that the neighbor is reachable.  Firewall rules can also block BGP traffic (TCP port 179).
*   **Neighbor flapping (frequent Up/Down transitions):** This can be caused by network instability, link failures, or BGP configuration errors. Investigate the underlying network connectivity and examine the BGP keepalive and hold timers.
*   **Low or zero PfxRcd:**  If a neighbor is in the `Established` state but the number of prefixes received is unexpectedly low or zero, it suggests that the neighbor is not advertising the expected routes. Check the neighbor's BGP configuration, outbound route filters, and ensure it has routes to advertise.
*   **Mismatched TblVer:** If the local BGP table version and the neighbor's table version are different, it indicates a synchronization problem.  This might be due to slow processing, network congestion, or BGP policy configurations causing route filtering.
*   **High InQ or OutQ:** Persistent high queue lengths suggest that the router is unable to process or send BGP messages quickly enough. Investigate router CPU and memory utilization, and examine the network path for congestion.
*   **Incorrect AS number:** A wrong ASN can also cause BGP peering issues.

## Beyond the Basics: Filtering and Refining the Output

While `show ip bgp summary` provides a general overview, you can combine it with other commands and techniques to gain more specific insights:

*   **Filtering Output:** Use `include`, `exclude`, or `begin` options to filter the output and focus on specific neighbors or ASNs. For example, `show ip bgp summary | include 192.168.1.2` will only show the information for the neighbor with that IP address.
*   **Combining with `show ip bgp neighbors`:**  For a deeper dive into a specific neighbor, use the `show ip bgp neighbors [neighbor-address]` command.  This provides detailed information about the BGP session parameters, keepalive timers, and received/sent capabilities.
*   **Using `show ip route bgp`:**  To examine the BGP routing table, use the `show ip route bgp` command. This will display all the routes learned via BGP.

Ready to elevate your networking skills and become a BGP master? Don't miss out on this free opportunity to learn BGP fundamentals! Download your course now: [Free BGP Course Download](https://udemywork.com/show-ip-bgp-summary)

## Conclusion

The `show ip bgp summary` command is an essential tool for any network engineer working with BGP. By understanding the output of this command, you can quickly assess the health of your BGP peering sessions, identify potential problems, and troubleshoot routing issues. Combined with other BGP commands and a solid understanding of BGP fundamentals, you can effectively manage and optimize your BGP network. Remember to analyze the output regularly, especially after making configuration changes or experiencing network events, to ensure the stability and performance of your BGP infrastructure. Understanding this output well is a key to effective network management.

Start your journey to BGP mastery today with my free course! Get instant access: [BGP Essentials - Free Download](https://udemywork.com/show-ip-bgp-summary)
