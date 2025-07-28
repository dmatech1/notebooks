# ICMP Testing

https://keith.github.io/xcode-man-pages/icmp.4.html

Ping sockets are defined in [net/ipv4/af_inet.c](https://github.com/torvalds/linux/blob/ec2df4364666a96e7868b7257bc7235bae263dcb/net/ipv4/af_inet.c#L1172-L1178) and implemented in [net/ipv4/ping.c](https://github.com/torvalds/linux/blob/ec2df4364666a96e7868b7257bc7235bae263dcb/net/ipv4/ping.c#L990-L1010).  Likewise, IPv6 ping sockets are defined in [net/ipv6/af_inet6.c](https://github.com/torvalds/linux/blob/ec2df4364666a96e7868b7257bc7235bae263dcb/net/ipv6/af_inet6.c#L1107) and implemented in [net/ipv6/ping.c](https://github.com/torvalds/linux/blob/ec2df4364666a96e7868b7257bc7235bae263dcb/net/ipv6/ping.c#L197-L217).  They're effectively a cut-down version of raw sockets with added controls.

```c
socket(AF_INET, SOCK_DGRAM, IPPROTO_ICMP)
```

Generated using this:

```bash
zcat /usr/share/man/man7/ip.7.gz | pandoc --from man --to html --wrap=preserve > ip.html
```

You can use the following options:

<dl>
<dt><strong>IP_ADD_MEMBERSHIP</strong> (since Linux 1.2)</dt>
<dd>
<p>Join a multicast group. Argument is an <em>ip_mreqn</em> structure.</p>

> ```c
> struct ip_mreqn {
>     struct in_addr imr_multiaddr; /* IP multicast group address */
>     struct in_addr imr_address;   /* IP address of local interface */
>     int            imr_ifindex;   /* interface index */
> };
> ```

<p><em>imr_multiaddr</em> contains the address of the multicast group the application wants to join or leave. It must be a valid multicast address (or <strong>setsockopt</strong>(2) fails with the error <strong>EINVAL</strong>). <em>imr_address</em> is the address of the local interface with which the system should join the multicast group; if it is equal to <strong>INADDR_ANY</strong>, an appropriate interface is chosen by the system. <em>imr_ifindex</em> is the interface index of the interface that should join/leave the <em>imr_multiaddr</em> group, or 0 to indicate any interface.</p>
<p>The <em>ip_mreqn</em> structure is available only since Linux 2.2. For compatibility, the old <em>ip_mreq</em> structure (present since Linux 1.2) is still supported; it differs from <em>ip_mreqn</em> only by not including the <em>imr_ifindex</em> field. (The kernel determines which structure is being passed based on the size passed in <em>optlen</em>.)</p>
<p><strong>IP_ADD_MEMBERSHIP</strong> is valid only for <strong>setsockopt</strong>(2).</p>
</dd>
<dt><strong>IP_ADD_SOURCE_MEMBERSHIP</strong> (since Linux 2.4.22 / Linux 2.5.68)</dt>
<dd>
<p>Join a multicast group and allow receiving data only from a specified source. Argument is an <em>ip_mreq_source</em> structure.</p>

> ```c
> struct ip_mreq_source {
>     struct in_addr imr_multiaddr;  /* IP multicast group address */
>     struct in_addr imr_interface;  /* IP address of local interface */
>     struct in_addr imr_sourceaddr; /* IP address of multicast source */
> };
> ```

<p>The <em>ip_mreq_source</em> structure is similar to <em>ip_mreqn</em> described under <strong>IP_ADD_MEMBERSHIP</strong>. The <em>imr_multiaddr</em> field contains the address of the multicast group the application wants to join or leave. The <em>imr_interface</em> field is the address of the local interface with which the system should join the multicast group. Finally, the <em>imr_sourceaddr</em> field contains the address of the source the application wants to receive data from.</p>
<p>This option can be used multiple times to allow receiving data from more than one source.</p>
</dd>
<dt><strong>IP_BIND_ADDRESS_NO_PORT</strong> (since Linux 4.2)</dt>
<dd>
<p>Inform the kernel to not reserve an ephemeral port when using <strong>bind</strong>(2) with a port number of 0. The port will later be automatically chosen at <strong>connect</strong>(2) time, in a way that allows sharing a source port as long as the 4-tuple is unique.</p>
</dd>
<dt><strong>IP_BLOCK_SOURCE</strong> (since Linux 2.4.22 / 2.5.68)</dt>
<dd>
<p>Stop receiving multicast data from a specific source in a given group. This is valid only after the application has subscribed to the multicast group using either <strong>IP_ADD_MEMBERSHIP</strong> or <strong>IP_ADD_SOURCE_MEMBERSHIP</strong>.</p>
<p>Argument is an <em>ip_mreq_source</em> structure as described under <strong>IP_ADD_SOURCE_MEMBERSHIP</strong>.</p>
</dd>
<dt><strong>IP_DROP_MEMBERSHIP</strong> (since Linux 1.2)</dt>
<dd>
<p>Leave a multicast group. Argument is an <em>ip_mreqn</em> or <em>ip_mreq</em> structure similar to <strong>IP_ADD_MEMBERSHIP</strong>.</p>
</dd>
<dt><strong>IP_DROP_SOURCE_MEMBERSHIP</strong> (since Linux 2.4.22 / 2.5.68)</dt>
<dd>
<p>Leave a source-specific group—that is, stop receiving data from a given multicast group that come from a given source. If the application has subscribed to multiple sources within the same group, data from the remaining sources will still be delivered. To stop receiving data from all sources at once, use <strong>IP_DROP_MEMBERSHIP</strong>.</p>
<p>Argument is an <em>ip_mreq_source</em> structure as described under <strong>IP_ADD_SOURCE_MEMBERSHIP</strong>.</p>
</dd>
<dt><strong>IP_FREEBIND</strong> (since Linux 2.4)</dt>
<dd>
<p>If enabled, this boolean option allows binding to an IP address that is nonlocal or does not (yet) exist. This permits listening on a socket, without requiring the underlying network interface or the specified dynamic IP address to be up at the time that the application is trying to bind to it. This option is the per-socket equivalent of the <em>ip_nonlocal_bind</em> <em>/proc</em> interface described below.</p>
</dd>
<dt><strong>IP_HDRINCL</strong> (since Linux 2.0)</dt>
<dd>
<p>If enabled, the user supplies an IP header in front of the user data. Valid only for <strong>SOCK_RAW</strong> sockets; see <strong>raw</strong>(7) for more information. When this flag is enabled, the values set by <strong>IP_OPTIONS</strong>, <strong>IP_TTL</strong>, and <strong>IP_TOS</strong> are ignored.</p>
</dd>
<dt><strong>IP_LOCAL_PORT_RANGE</strong> (since Linux 6.3)</dt>
<dd>
<p>Set or get the per-socket default local port range. This option can be used to clamp down the global local port range, defined by the <em>ip_local_port_range</em> <em>/proc</em> interface described below, for a given socket.</p>
<p>The option takes an <em>uint32_t</em> value with the high 16 bits set to the upper range bound, and the low 16 bits set to the lower range bound. Range bounds are inclusive. The 16-bit values should be in host byte order.</p>
<p>The lower bound has to be less than the upper bound when both bounds are not zero. Otherwise, setting the option fails with EINVAL.</p>
<p>If either bound is outside of the global local port range, or is zero, then that bound has no effect.</p>
<p>To reset the setting, pass zero as both the upper and the lower bound.</p>
</dd>
<dt><strong>IP_MSFILTER</strong> (since Linux 2.4.22 / 2.5.68)</dt>
<dd>
<p>This option provides access to the advanced full-state filtering API. Argument is an <em>ip_msfilter</em> structure.</p>

> ```c
> struct ip_msfilter {
>     struct in_addr imsf_multiaddr; /* IP multicast group address */
>     struct in_addr imsf_interface; /* IP address of local interface */
>     uint32_t       imsf_fmode;     /* Filter-mode */
>     uint32_t       imsf_numsrc;    /* Number of sources in the following array */
>     struct in_addr imsf_slist[1];  /* Array of source addresses */
> };
> ```

<p>There are two macros, <strong>MCAST_INCLUDE</strong> and <strong>MCAST_EXCLUDE</strong>, which can be used to specify the filtering mode. Additionally, the <strong>IP_MSFILTER_SIZE</strong>(n) macro exists to determine how much memory is needed to store <em>ip_msfilter</em> structure with <em>n</em> sources in the source list.</p>
<p>For the full description of multicast source filtering refer to RFC 3376.</p>
</dd>
<dt><strong>IP_MTU</strong> (since Linux 2.2)</dt>
<dd>
<p>Retrieve the current known path MTU of the current socket. Returns an integer.</p>
<p><strong>IP_MTU</strong> is valid only for <strong>getsockopt</strong>(2) and can be employed only when the socket has been connected.</p>
</dd>
<dt><strong>IP_MTU_DISCOVER</strong> (since Linux 2.2)</dt>
<dd>
<p>Set or receive the Path MTU Discovery setting for a socket. When enabled, Linux will perform Path MTU Discovery as defined in RFC 1191 on <strong>SOCK_STREAM</strong> sockets. For non-<strong>SOCK_STREAM</strong> sockets, <strong>IP_PMTUDISC_DO</strong> forces the don't-fragment flag to be set on all outgoing packets. It is the user's responsibility to packetize the data in MTU-sized chunks and to do the retransmits if necessary. The kernel will reject (with <strong>EMSGSIZE</strong>) datagrams that are bigger than the known path MTU. <strong>IP_PMTUDISC_WANT</strong> will fragment a datagram if needed according to the path MTU, or will set the don't-fragment flag otherwise.</p>
<p>The system-wide default can be toggled between <strong>IP_PMTUDISC_WANT</strong> and <strong>IP_PMTUDISC_DONT</strong> by writing (respectively, zero and nonzero values) to the <em>/proc/sys/net/ipv4/ip_no_pmtu_disc</em> file.</p>
<table style="margin-right:auto;margin-left:0px">
<tbody>
<tr>
<th>Path MTU discovery value</th>
<th>Meaning</th>
</tr>
<tr>
<td>IP_PMTUDISC_WANT</td>
<td>Use per-route settings.</td>
</tr>
<tr>
<td>IP_PMTUDISC_DONT</td>
<td>Never do Path MTU Discovery.</td>
</tr>
<tr>
<td>IP_PMTUDISC_DO</td>
<td>Always do Path MTU Discovery.</td>
</tr>
<tr>
<td>IP_PMTUDISC_PROBE</td>
<td>Set DF but ignore Path MTU.</td>
</tr>
</tbody>
</table>
<p>When PMTU discovery is enabled, the kernel automatically keeps track of the path MTU per destination host. When it is connected to a specific peer with <strong>connect</strong>(2), the currently known path MTU can be retrieved conveniently using the <strong>IP_MTU</strong> socket option (e.g., after an <strong>EMSGSIZE</strong> error occurred). The path MTU may change over time. For connectionless sockets with many destinations, the new MTU for a given destination can also be accessed using the error queue (see <strong>IP_RECVERR</strong>). A new error will be queued for every incoming MTU update.</p>
<p>While MTU discovery is in progress, initial packets from datagram sockets may be dropped. Applications using UDP should be aware of this and not take it into account for their packet retransmit strategy.</p>
<p>To bootstrap the path MTU discovery process on unconnected sockets, it is possible to start with a big datagram size (headers up to 64 kilobytes long) and let it shrink by updates of the path MTU.</p>
<p>To get an initial estimate of the path MTU, connect a datagram socket to the destination address using <strong>connect</strong>(2) and retrieve the MTU by calling <strong>getsockopt</strong>(2) with the <strong>IP_MTU</strong> option.</p>
<p>It is possible to implement RFC 4821 MTU probing with <strong>SOCK_DGRAM</strong> or <strong>SOCK_RAW</strong> sockets by setting a value of <strong>IP_PMTUDISC_PROBE</strong> (available since Linux 2.6.22). This is also particularly useful for diagnostic tools such as <strong>tracepath</strong>(8) that wish to deliberately send probe packets larger than the observed Path MTU.</p>
</dd>
<dt><strong>IP_MULTICAST_ALL</strong> (since Linux 2.6.31)</dt>
<dd>
<p>This option can be used to modify the delivery policy of multicast messages. The argument is a boolean integer (defaults to 1). If set to 1, the socket will receive messages from all the groups that have been joined globally on the whole system. Otherwise, it will deliver messages only from the groups that have been explicitly joined (for example via the <strong>IP_ADD_MEMBERSHIP</strong> option) on this particular socket.</p>
</dd>
<dt><strong>IP_MULTICAST_IF</strong> (since Linux 1.2)</dt>
<dd>
<p>Set the local device for a multicast socket. The argument for <strong>setsockopt</strong>(2) is an <em>ip_mreqn</em> or (since Linux 3.5) <em>ip_mreq</em> structure similar to <strong>IP_ADD_MEMBERSHIP</strong>, or an <em>in_addr</em> structure. (The kernel determines which structure is being passed based on the size passed in <em>optlen</em>.) For <strong>getsockopt</strong>(2), the argument is an <em>in_addr</em> structure.</p>
</dd>
<dt><strong>IP_MULTICAST_LOOP</strong> (since Linux 1.2)</dt>
<dd>
<p>Set or read a boolean integer argument that determines whether sent multicast packets should be looped back to the local sockets.</p>
</dd>
<dt><strong>IP_MULTICAST_TTL</strong> (since Linux 1.2)</dt>
<dd>
<p>Set or read the time-to-live value of outgoing multicast packets for this socket. It is very important for multicast packets to set the smallest TTL possible. The default is 1 which means that multicast packets don't leave the local network unless the user program explicitly requests it. Argument is an integer.</p>
</dd>
<dt><strong>IP_NODEFRAG</strong> (since Linux 2.6.36)</dt>
<dd>
<p>If enabled (argument is nonzero), the reassembly of outgoing packets is disabled in the netfilter layer. The argument is an integer.</p>
<p>This option is valid only for <strong>SOCK_RAW</strong> sockets.</p>
</dd>
<dt><strong>IP_OPTIONS</strong> (since Linux 2.0)</dt>
<dd>
<p>Set or get the IP options to be sent with every packet from this socket. The arguments are a pointer to a memory buffer containing the options and the option length. The <strong>setsockopt</strong>(2) call sets the IP options associated with a socket. The maximum option size for IPv4 is 40 bytes. See RFC 791 for the allowed options. When the initial connection request packet for a <strong>SOCK_STREAM</strong> socket contains IP options, the IP options will be set automatically to the options from the initial packet with routing headers reversed. Incoming packets are not allowed to change options after the connection is established. The processing of all incoming source routing options is disabled by default and can be enabled by using the <em>accept_source_route</em> <em>/proc</em> interface. Other options like timestamps are still handled. For datagram sockets, IP options can be set only by the local user. Calling <strong>getsockopt</strong>(2) with <strong>IP_OPTIONS</strong> puts the current IP options used for sending into the supplied buffer.</p>
</dd>
<dt><strong>IP_PASSSEC</strong> (since Linux 2.6.17)</dt>
<dd>
<p>If labeled IPSEC or NetLabel is configured on the sending and receiving hosts, this option enables receiving of the security context of the peer socket in an ancillary message of type <strong>SCM_SECURITY</strong> retrieved using <strong>recvmsg</strong>(2). This option is supported only for UDP sockets; for TCP or SCTP sockets, see the description of the <strong>SO_PEERSEC</strong> option below.</p>
<p>The value given as an argument to <strong>setsockopt</strong>(2) and returned as the result of <strong>getsockopt</strong>(2) is an integer boolean flag.</p>
<p>The security context returned in the <strong>SCM_SECURITY</strong> ancillary message is of the same format as the one described under the <strong>SO_PEERSEC</strong> option below.</p>
<p>Note: the reuse of the <strong>SCM_SECURITY</strong> message type for the <strong>IP_PASSSEC</strong> socket option was likely a mistake, since other IP control messages use their own numbering scheme in the IP namespace and often use the socket option value as the message type. There is no conflict currently since the IP option with the same value as <strong>SCM_SECURITY</strong> is <strong>IP_HDRINCL</strong> and this is never used for a control message type.</p>
</dd>
<dt><strong>IP_PKTINFO</strong> (since Linux 2.2)</dt>
<dd>
<p>Pass an <strong>IP_PKTINFO</strong> ancillary message that contains a <em>pktinfo</em> structure that supplies some information about the incoming packet. This works only for datagram oriented sockets. The argument is a flag that tells the socket whether the <strong>IP_PKTINFO</strong> message should be passed or not. The message itself can be sent/retrieved only as a control message with a packet using <strong>recvmsg</strong>(2) or <strong>sendmsg</strong>(2).</p>

> ```c
> struct in_pktinfo {
>     unsigned int   ipi_ifindex;  /* Interface index */
>     struct in_addr ipi_spec_dst; /* Local address */
>     struct in_addr ipi_addr;     /* Header Destination address */
> };
> ```

<p><em>ipi_ifindex</em> is the unique index of the interface the packet was received on. <em>ipi_spec_dst</em> is the local address of the packet and <em>ipi_addr</em> is the destination address in the packet header. If <strong>IP_PKTINFO</strong> is passed to <strong>sendmsg</strong>(2) and <em>ipi_spec_dst</em> is not zero, then it is used as the local source address for the routing table lookup and for setting up IP source route options. When <em>ipi_ifindex</em> is not zero, the primary local address of the interface specified by the index overwrites <em>ipi_spec_dst</em> for the routing table lookup.</p>
<p>Not supported for <strong>SOCK_STREAM</strong> sockets.</p>
</dd>
<dt><strong>IP_RECVERR</strong> (since Linux 2.2)</dt>
<dd>
<p>Enable extended reliable error message passing. When enabled on a datagram socket, all generated errors will be queued in a per-socket error queue. When the user receives an error from a socket operation, the errors can be received by calling <strong>recvmsg</strong>(2) with the <strong>MSG_ERRQUEUE</strong> flag set. The <em>sock_extended_err</em> structure describing the error will be passed in an ancillary message with the type <strong>IP_RECVERR</strong> and the level <strong>IPPROTO_IP</strong>. This is useful for reliable error handling on unconnected sockets. The received data portion of the error queue contains the error packet.</p>
<p>The <strong>IP_RECVERR</strong> control message contains a <em>sock_extended_err</em> structure:</p>

> ```c
> #define SO_EE_ORIGIN_NONE    0
> #define SO_EE_ORIGIN_LOCAL   1
> #define SO_EE_ORIGIN_ICMP    2
> #define SO_EE_ORIGIN_ICMP6   3
> 
> struct sock_extended_err {
>     uint32_t ee_errno;   /* error number */
>     uint8_t  ee_origin;  /* where the error originated */
>     uint8_t  ee_type;    /* type */
>     uint8_t  ee_code;    /* code */
>     uint8_t  ee_pad;
>     uint32_t ee_info;    /* additional information */
>     uint32_t ee_data;    /* other data */
>     /* More data may follow */
> };
> 
> struct sockaddr *SO_EE_OFFENDER(struct sock_extended_err *);
> ```

<p><em>ee_errno</em> contains the <em>errno</em> number of the queued error. <em>ee_origin</em> is the origin code of where the error originated. The other fields are protocol-specific. The macro <strong>SO_EE_OFFENDER</strong> returns a pointer to the address of the network object where the error originated from given a pointer to the ancillary message. If this address is not known, the <em>sa_family</em> member of the <em>sockaddr</em> contains <strong>AF_UNSPEC</strong> and the other fields of the <em>sockaddr</em> are undefined.</p>
<p>IP uses the <em>sock_extended_err</em> structure as follows: <em>ee_origin</em> is set to <strong>SO_EE_ORIGIN_ICMP</strong> for errors received as an ICMP packet, or <strong>SO_EE_ORIGIN_LOCAL</strong> for locally generated errors. Unknown values should be ignored. <em>ee_type</em> and <em>ee_code</em> are set from the type and code fields of the ICMP header. <em>ee_info</em> contains the discovered MTU for <strong>EMSGSIZE</strong> errors. The message also contains the <em>sockaddr_in of the node</em> caused the error, which can be accessed with the <strong>SO_EE_OFFENDER</strong> macro. The <em>sin_family</em> field of the <strong>SO_EE_OFFENDER</strong> address is <strong>AF_UNSPEC</strong> when the source was unknown. When the error originated from the network, all IP options (<strong>IP_OPTIONS</strong>, <strong>IP_TTL</strong>, etc.) enabled on the socket and contained in the error packet are passed as control messages. The payload of the packet causing the error is returned as normal payload. Note that TCP has no error queue; <strong>MSG_ERRQUEUE</strong> is not permitted on <strong>SOCK_STREAM</strong> sockets. <strong>IP_RECVERR</strong> is valid for TCP, but all errors are returned by socket function return or <strong>SO_ERROR</strong> only.</p>
<p>For raw sockets, <strong>IP_RECVERR</strong> enables passing of all received ICMP errors to the application, otherwise errors are reported only on connected sockets</p>
<p>It sets or retrieves an integer boolean flag. <strong>IP_RECVERR</strong> defaults to off.</p>
</dd>
<dt><strong>IP_RECVOPTS</strong> (since Linux 2.2)</dt>
<dd>
<p>Pass all incoming IP options to the user in a <strong>IP_OPTIONS</strong> control message. The routing header and other options are already filled in for the local host. Not supported for <strong>SOCK_STREAM</strong> sockets.</p>
</dd>
<dt><strong>IP_RECVORIGDSTADDR</strong> (since Linux 2.6.29)</dt>
<dd>
<p>This boolean option enables the <strong>IP_ORIGDSTADDR</strong> ancillary message in <strong>recvmsg</strong>(2), in which the kernel returns the original destination address of the datagram being received. The ancillary message contains a <em>struct sockaddr_in</em>. Not supported for <strong>SOCK_STREAM</strong> sockets.</p>
</dd>
<dt><strong>IP_RECVTOS</strong> (since Linux 2.2)</dt>
<dd>
<p>If enabled, the <strong>IP_TOS</strong> ancillary message is passed with incoming packets. It contains a byte which specifies the Type of Service/Precedence field of the packet header. Expects a boolean integer flag. Not supported for <strong>SOCK_STREAM</strong> sockets.</p>
</dd>
<dt><strong>IP_RECVTTL</strong> (since Linux 2.2)</dt>
<dd>
<p>When this flag is set, pass a <strong>IP_TTL</strong> control message with the time-to-live field of the received packet as a 32 bit integer. Not supported for <strong>SOCK_STREAM</strong> sockets.</p>
</dd>
<dt><strong>IP_RETOPTS</strong> (since Linux 2.2)</dt>
<dd>
<p>Identical to <strong>IP_RECVOPTS</strong>, but returns raw unprocessed options with timestamp and route record options not filled in for this hop. Not supported for <strong>SOCK_STREAM</strong> sockets.</p>
</dd>
<dt><strong>IP_ROUTER_ALERT</strong> (since Linux 2.2)</dt>
<dd>
<p>Pass all to-be forwarded packets with the IP Router Alert option set to this socket. Valid only for raw sockets. This is useful, for instance, for user-space RSVP daemons. The tapped packets are not forwarded by the kernel; it is the user's responsibility to send them out again. Socket binding is ignored, such packets are filtered only by protocol. Expects an integer flag.</p>
</dd>
<dt><strong>IP_TOS</strong> (since Linux 1.0)</dt>
<dd>
<p>Set or receive the Type-Of-Service (TOS) field that is sent with every IP packet originating from this socket. It is used to prioritize packets on the network. TOS is a byte. There are some standard TOS flags defined: <strong>IPTOS_LOWDELAY</strong> to minimize delays for interactive traffic, <strong>IPTOS_THROUGHPUT</strong> to optimize throughput, <strong>IPTOS_RELIABILITY</strong> to optimize for reliability, <strong>IPTOS_MINCOST</strong> should be used for "filler data" where slow transmission doesn't matter. At most one of these TOS values can be specified. Other bits are invalid and shall be cleared. Linux sends <strong>IPTOS_LOWDELAY</strong> datagrams first by default, but the exact behavior depends on the configured queueing discipline. Some high-priority levels may require superuser privileges (the <strong>CAP_NET_ADMIN</strong> capability).</p>
</dd>
<dt><strong>IP_TRANSPARENT</strong> (since Linux 2.6.24)</dt>
<dd>
<p>Setting this boolean option enables transparent proxying on this socket. This socket option allows the calling application to bind to a nonlocal IP address and operate both as a client and a server with the foreign address as the local endpoint. NOTE: this requires that routing be set up in a way that packets going to the foreign address are routed through the TProxy box (i.e., the system hosting the application that employs the <strong>IP_TRANSPARENT</strong> socket option). Enabling this socket option requires superuser privileges (the <strong>CAP_NET_ADMIN</strong> capability).</p>
<p>TProxy redirection with the iptables TPROXY target also requires that this option be set on the redirected socket.</p>
</dd>
<dt><strong>IP_TTL</strong> (since Linux 1.0)</dt>
<dd>
<p>Set or retrieve the current time-to-live field that is used in every packet sent from this socket.</p>
</dd>
<dt><strong>IP_UNBLOCK_SOURCE</strong> (since Linux 2.4.22 / 2.5.68)</dt>
<dd>
<p>Unblock previously blocked multicast source. Returns <strong>EADDRNOTAVAIL</strong> when given source is not being blocked.</p>
<p>Argument is an <em>ip_mreq_source</em> structure as described under <strong>IP_ADD_SOURCE_MEMBERSHIP</strong>.</p>
</dd>
<dt><strong>SO_PEERSEC</strong> (since Linux 2.6.17)</dt>
<dd>
<p>If labeled IPSEC or NetLabel is configured on both the sending and receiving hosts, this read-only socket option returns the security context of the peer socket connected to this socket. By default, this will be the same as the security context of the process that created the peer socket unless overridden by the policy or by a process with the required permissions.</p>
<p>The argument to <strong>getsockopt</strong>(2) is a pointer to a buffer of the specified length in bytes into which the security context string will be copied. If the buffer length is less than the length of the security context string, then <strong>getsockopt</strong>(2) returns -1, sets <em>errno</em> to <strong>ERANGE</strong>, and returns the required length via <em>optlen</em>. The caller should allocate at least <strong>NAME_MAX</strong> bytes for the buffer initially, although this is not guaranteed to be sufficient. Resizing the buffer to the returned length and retrying may be necessary.</p>
<p>The security context string may include a terminating null character in the returned length, but is not guaranteed to do so: a security context "foo" might be represented as either {'f','o','o'} of length 3 or {'f','o','o','\0'} of length 4, which are considered to be interchangeable. The string is printable, does not contain non-terminating null characters, and is in an unspecified encoding (in particular, it is not guaranteed to be ASCII or UTF-8).</p>
<p>The use of this option for sockets in the <strong>AF_INET</strong> address family is supported since Linux 2.6.17 for TCP sockets, and since Linux 4.17 for SCTP sockets.</p>
<p>For SELinux, NetLabel conveys only the MLS portion of the security context of the peer across the wire, defaulting the rest of the security context to the values defined in the policy for the netmsg initial security identifier (SID). However, NetLabel can be configured to pass full security contexts over loopback. Labeled IPSEC always passes full security contexts as part of establishing the security association (SA) and looks them up based on the association for each packet.</p>
</dd>
</dl>

# Generic Socket Options

<dl>
<dt><strong>SO_ACCEPTCONN</strong></dt>
<dd>
<p>Returns a value indicating whether or not this socket has been marked to accept connections with <strong>listen</strong>(2). The value 0 indicates that this is not a listening socket, the value 1 indicates that this is a listening socket. This socket option is read-only.</p>
</dd>
<dt><strong>SO_ATTACH_FILTER</strong> (since Linux 2.2)<br />
<strong>SO_ATTACH_BPF</strong> (since Linux 3.19)</dt>
<dd>
<p>Attach a classic BPF (<strong>SO_ATTACH_FILTER</strong>) or an extended BPF (<strong>SO_ATTACH_BPF</strong>) program to the socket for use as a filter of incoming packets. A packet will be dropped if the filter program returns zero. If the filter program returns a nonzero value which is less than the packet's data length, the packet will be truncated to the length returned. If the value returned by the filter is greater than or equal to the packet's data length, the packet is allowed to proceed unmodified.</p>
<p>The argument for <strong>SO_ATTACH_FILTER</strong> is a <em>sock_fprog</em> structure, defined in <em>&lt;linux/filter.h&gt;</em>:</p>

> ```c
> struct sock_fprog {
>     unsigned short      len;
>     struct sock_filter *filter;
> };
> ```

<p>The argument for <strong>SO_ATTACH_BPF</strong> is a file descriptor returned by the <strong>bpf</strong>(2) system call and must refer to a program of type <strong>BPF_PROG_TYPE_SOCKET_FILTER</strong>.</p>
<p>These options may be set multiple times for a given socket, each time replacing the previous filter program. The classic and extended versions may be called on the same socket, but the previous filter will always be replaced such that a socket never has more than one filter defined.</p>
<p>Both classic and extended BPF are explained in the kernel source file <em>Documentation/networking/filter.txt</em></p>
</dd>
<dt><strong>SO_ATTACH_REUSEPORT_CBPF</strong><br />
<strong>SO_ATTACH_REUSEPORT_EBPF</strong></dt>
<dd>
<p>For use with the <strong>SO_REUSEPORT</strong> option, these options allow the user to set a classic BPF (<strong>SO_ATTACH_REUSEPORT_CBPF</strong>) or an extended BPF (<strong>SO_ATTACH_REUSEPORT_EBPF</strong>) program which defines how packets are assigned to the sockets in the reuseport group (that is, all sockets which have <strong>SO_REUSEPORT</strong> set and are using the same local address to receive packets).</p>
<p>The BPF program must return an index between 0 and N-1 representing the socket which should receive the packet (where N is the number of sockets in the group). If the BPF program returns an invalid index, socket selection will fall back to the plain <strong>SO_REUSEPORT</strong> mechanism.</p>
<p>Sockets are numbered in the order in which they are added to the group (that is, the order of <strong>bind</strong>(2) calls for UDP sockets or the order of <strong>listen</strong>(2) calls for TCP sockets). New sockets added to a reuseport group will inherit the BPF program. When a socket is removed from a reuseport group (via <strong>close</strong>(2)), the last socket in the group will be moved into the closed socket's position.</p>
<p>These options may be set repeatedly at any time on any socket in the group to replace the current BPF program used by all sockets in the group.</p>
<p><strong>SO_ATTACH_REUSEPORT_CBPF</strong> takes the same argument type as <strong>SO_ATTACH_FILTER</strong> and <strong>SO_ATTACH_REUSEPORT_EBPF</strong> takes the same argument type as <strong>SO_ATTACH_BPF</strong>.</p>
<p>UDP support for this feature is available since Linux 4.5; TCP support is available since Linux 4.6.</p>
</dd>
<dt><strong>SO_BINDTODEVICE</strong></dt>
<dd>
<p>Bind this socket to a particular device like “eth0”, as specified in the passed interface name. If the name is an empty string or the option length is zero, the socket device binding is removed. The passed option is a variable-length null-terminated interface name string with the maximum size of <strong>IFNAMSIZ</strong>. If a socket is bound to an interface, only packets received from that particular interface are processed by the socket. Note that this works only for some socket types, particularly <strong>AF_INET</strong> sockets. It is not supported for packet sockets (use normal <strong>bind</strong>(2) there).</p>
<p>Before Linux 3.8, this socket option could be set, but could not retrieved with <strong>getsockopt</strong>(2). Since Linux 3.8, it is readable. The <em>optlen</em> argument should contain the buffer size available to receive the device name and is recommended to be <strong>IFNAMSIZ</strong> bytes. The real device name length is reported back in the <em>optlen</em> argument.</p>
</dd>
<dt><strong>SO_BROADCAST</strong></dt>
<dd>
<p>Set or get the broadcast flag. When enabled, datagram sockets are allowed to send packets to a broadcast address. This option has no effect on stream-oriented sockets.</p>
</dd>
<dt><strong>SO_BSDCOMPAT</strong></dt>
<dd>
<p>Enable BSD bug-to-bug compatibility. This is used by the UDP protocol module in Linux 2.0 and 2.2. If enabled, ICMP errors received for a UDP socket will not be passed to the user program. In later kernel versions, support for this option has been phased out: Linux 2.4 silently ignores it, and Linux 2.6 generates a kernel warning (printk()) if a program uses this option. Linux 2.0 also enabled BSD bug-to-bug compatibility options (random header changing, skipping of the broadcast flag) for raw sockets with this option, but that was removed in Linux 2.2.</p>
</dd>
<dt><strong>SO_DEBUG</strong></dt>
<dd>
<p>Enable socket debugging. Allowed only for processes with the <strong>CAP_NET_ADMIN</strong> capability or an effective user ID of 0.</p>
</dd>
<dt><strong>SO_DETACH_FILTER</strong> (since Linux 2.2)<br />
<strong>SO_DETACH_BPF</strong> (since Linux 3.19)</dt>
<dd>
<p>These two options, which are synonyms, may be used to remove the classic or extended BPF program attached to a socket with either <strong>SO_ATTACH_FILTER</strong> or <strong>SO_ATTACH_BPF</strong>. The option value is ignored.</p>
</dd>
<dt><strong>SO_DOMAIN</strong> (since Linux 2.6.32)</dt>
<dd>
<p>Retrieves the socket domain as an integer, returning a value such as <strong>AF_INET6</strong>. See <strong>socket</strong>(2) for details. This socket option is read-only.</p>
</dd>
<dt><strong>SO_ERROR</strong></dt>
<dd>
<p>Get and clear the pending socket error. This socket option is read-only. Expects an integer.</p>
</dd>
<dt><strong>SO_DONTROUTE</strong></dt>
<dd>
<p>Don't send via a gateway, send only to directly connected hosts. The same effect can be achieved by setting the <strong>MSG_DONTROUTE</strong> flag on a socket <strong>send</strong>(2) operation. Expects an integer boolean flag.</p>
</dd>
<dt><strong>SO_INCOMING_CPU</strong> (gettable since Linux 3.19, settable since Linux 4.4)</dt>
<dd>
<p>Sets or gets the CPU affinity of a socket. Expects an integer flag.</p>

> ```c
> int cpu = 1;
> setsockopt(fd, SOL_SOCKET, SO_INCOMING_CPU, &cpu, sizeof(cpu));
> ```

<p>Because all of the packets for a single stream (i.e., all packets for the same 4-tuple) arrive on the single RX queue that is associated with a particular CPU, the typical use case is to employ one listening process per RX queue, with the incoming flow being handled by a listener on the same CPU that is handling the RX queue. This provides optimal NUMA behavior and keeps CPU caches hot.</p>
</dd>
<dt><strong>SO_INCOMING_NAPI_ID</strong> (gettable since Linux 4.12)</dt>
<dd>
<p>Returns a system-level unique ID called NAPI ID that is associated with a RX queue on which the last packet associated with that socket is received.</p>
<p>This can be used by an application to split the incoming flows among worker threads based on the RX queue on which the packets associated with the flows are received. It allows each worker thread to be associated with a NIC HW receive queue and service all the connection requests received on that RX queue. This mapping between an app thread and a HW NIC queue streamlines the flow of data from the NIC to the application.</p>
</dd>
<dt><strong>SO_KEEPALIVE</strong></dt>
<dd>
<p>Enable sending of keep-alive messages on connection-oriented sockets. Expects an integer boolean flag.</p>
</dd>
<dt><strong>SO_LINGER</strong></dt>
<dd>
<p>Sets or gets the <strong>SO_LINGER</strong> option. The argument is a <em>linger</em> structure.</p>

> ```c
> struct linger {
>     int l_onoff;    /* linger active */
>     int l_linger;   /* how many seconds to linger for */
> };
> ```

<p>When enabled, a <strong>close</strong>(2) or <strong>shutdown</strong>(2) will not return until all queued messages for the socket have been successfully sent or the linger timeout has been reached. Otherwise, the call returns immediately and the closing is done in the background. When the socket is closed as part of <strong>exit</strong>(2), it always lingers in the background.</p>
</dd>
<dt><strong>SO_LOCK_FILTER</strong></dt>
<dd>
<p>When set, this option will prevent changing the filters associated with the socket. These filters include any set using the socket options <strong>SO_ATTACH_FILTER</strong>, <strong>SO_ATTACH_BPF</strong>, <strong>SO_ATTACH_REUSEPORT_CBPF</strong>, and <strong>SO_ATTACH_REUSEPORT_EBPF</strong>.</p>
<p>The typical use case is for a privileged process to set up a raw socket (an operation that requires the <strong>CAP_NET_RAW</strong> capability), apply a restrictive filter, set the <strong>SO_LOCK_FILTER</strong> option, and then either drop its privileges or pass the socket file descriptor to an unprivileged process via a UNIX domain socket.</p>
<p>Once the <strong>SO_LOCK_FILTER</strong> option has been enabled, attempts to change or remove the filter attached to a socket, or to disable the <strong>SO_LOCK_FILTER</strong> option will fail with the error <strong>EPERM</strong>.</p>
</dd>
<dt><strong>SO_MARK</strong> (since Linux 2.6.25)</dt>
<dd>
<p>Set the mark for each packet sent through this socket (similar to the netfilter MARK target but socket-based). Changing the mark can be used for mark-based routing without netfilter or for packet filtering. Setting this option requires the <strong>CAP_NET_ADMIN</strong> or <strong>CAP_NET_RAW</strong> (since Linux 5.17) capability.</p>
</dd>
<dt><strong>SO_OOBINLINE</strong></dt>
<dd>
<p>If this option is enabled, out-of-band data is directly placed into the receive data stream. Otherwise, out-of-band data is passed only when the <strong>MSG_OOB</strong> flag is set during receiving.</p>
</dd>
<dt><strong>SO_PASSCRED</strong></dt>
<dd>
<p>Enable or disable the receiving of the <strong>SCM_CREDENTIALS</strong> control message. For more information, see <strong>unix</strong>(7).</p>
</dd>
<dt><strong>SO_PASSSEC</strong></dt>
<dd>
<p>Enable or disable the receiving of the <strong>SCM_SECURITY</strong> control message. For more information, see <strong>unix</strong>(7).</p>
</dd>
<dt><strong>SO_PEEK_OFF</strong> (since Linux 3.4)</dt>
<dd>
<p>This option, which is currently supported only for <strong>unix</strong>(7) sockets, sets the value of the "peek offset" for the <strong>recv</strong>(2) system call when used with <strong>MSG_PEEK</strong> flag.</p>
<p>When this option is set to a negative value (it is set to -1 for all new sockets), traditional behavior is provided: <strong>recv</strong>(2) with the <strong>MSG_PEEK</strong> flag will peek data from the front of the queue.</p>
<p>When the option is set to a value greater than or equal to zero, then the next peek at data queued in the socket will occur at the byte offset specified by the option value. At the same time, the "peek offset" will be incremented by the number of bytes that were peeked from the queue, so that a subsequent peek will return the next data in the queue.</p>
<p>If data is removed from the front of the queue via a call to <strong>recv</strong>(2) (or similar) without the <strong>MSG_PEEK</strong> flag, the "peek offset" will be decreased by the number of bytes removed. In other words, receiving data without the <strong>MSG_PEEK</strong> flag will cause the "peek offset" to be adjusted to maintain the correct relative position in the queued data, so that a subsequent peek will retrieve the data that would have been retrieved had the data not been removed.</p>
<p>For datagram sockets, if the "peek offset" points to the middle of a packet, the data returned will be marked with the <strong>MSG_TRUNC</strong> flag.</p>
<p>The following example serves to illustrate the use of <strong>SO_PEEK_OFF</strong>. Suppose a stream socket has the following queued input data:</p>

> ```
> aabbccddeeff
> ```

<p></p>
<p>The following sequence of <strong>recv</strong>(2) calls would have the effect noted in the comments:</p>

> ```c
> int ov = 4;                  // Set peek offset to 4
> setsockopt(fd, SOL_SOCKET, SO_PEEK_OFF, &ov, sizeof(ov));
> recv(fd, buf, 2, MSG_PEEK);  // Peeks "cc"; offset set to 6
> recv(fd, buf, 2, MSG_PEEK);  // Peeks "dd"; offset set to 8
> recv(fd, buf, 2, 0);         // Reads "aa"; offset set to 6
> recv(fd, buf, 2, MSG_PEEK);  // Peeks "ee"; offset set to 8
> ```

<p></p>
</dd>
<dt><strong>SO_PEERCRED</strong></dt>
<dd>
<p>Return the credentials of the peer process connected to this socket. For further details, see <strong>unix</strong>(7).</p>
</dd>
<dt><strong>SO_PEERSEC</strong> (since Linux 2.6.2)</dt>
<dd>
<p>Return the security context of the peer socket connected to this socket. For further details, see <strong>unix</strong>(7) and <strong>ip</strong>(7).</p>
</dd>
<dt><strong>SO_PRIORITY</strong></dt>
<dd>
<p>Set the protocol-defined priority for all packets to be sent on this socket. Linux uses this value to order the networking queues: packets with a higher priority may be processed first depending on the selected device queueing discipline. Setting a priority outside the range 0 to 6 requires the <strong>CAP_NET_ADMIN</strong> capability.</p>
</dd>
<dt><strong>SO_PROTOCOL</strong> (since Linux 2.6.32)</dt>
<dd>
<p>Retrieves the socket protocol as an integer, returning a value such as <strong>IPPROTO_SCTP</strong>. See <strong>socket</strong>(2) for details. This socket option is read-only.</p>
</dd>
<dt><strong>SO_RCVBUF</strong></dt>
<dd>
<p>Sets or gets the maximum socket receive buffer in bytes. The kernel doubles this value (to allow space for bookkeeping overhead) when it is set using <strong>setsockopt</strong>(2), and this doubled value is returned by <strong>getsockopt</strong>(2). The default value is set by the <em>/proc/sys/net/core/rmem_default</em> file, and the maximum allowed value is set by the <em>/proc/sys/net/core/rmem_max</em> file. The minimum (doubled) value for this option is 256.</p>
</dd>
<dt><strong>SO_RCVBUFFORCE</strong> (since Linux 2.6.14)</dt>
<dd>
<p>Using this socket option, a privileged (<strong>CAP_NET_ADMIN</strong>) process can perform the same task as <strong>SO_RCVBUF</strong>, but the <em>rmem_max</em> limit can be overridden.</p>
</dd>
<dt><strong>SO_RCVLOWAT</strong> and <strong>SO_SNDLOWAT</strong></dt>
<dd>
<p>Specify the minimum number of bytes in the buffer until the socket layer will pass the data to the protocol (<strong>SO_SNDLOWAT</strong>) or the user on receiving (<strong>SO_RCVLOWAT</strong>). These two values are initialized to 1. <strong>SO_SNDLOWAT</strong> is not changeable on Linux (<strong>setsockopt</strong>(2) fails with the error <strong>ENOPROTOOPT</strong>). <strong>SO_RCVLOWAT</strong> is changeable only since Linux 2.4.</p>
<p>Before Linux 2.6.28 <strong>select</strong>(2), <strong>poll</strong>(2), and <strong>epoll</strong>(7) did not respect the <strong>SO_RCVLOWAT</strong> setting on Linux, and indicated a socket as readable when even a single byte of data was available. A subsequent read from the socket would then block until <strong>SO_RCVLOWAT</strong> bytes are available. Since Linux 2.6.28, <strong>select</strong>(2), <strong>poll</strong>(2), and <strong>epoll</strong>(7) indicate a socket as readable only if at least <strong>SO_RCVLOWAT</strong> bytes are available.</p>
</dd>
<dt><strong>SO_RCVTIMEO</strong> and <strong>SO_SNDTIMEO</strong></dt>
<dd>
<p>Specify the receiving or sending timeouts until reporting an error. The argument is a <em>struct timeval</em>. If an input or output function blocks for this period of time, and data has been sent or received, the return value of that function will be the amount of data transferred; if no data has been transferred and the timeout has been reached, then -1 is returned with <em>errno</em> set to <strong>EAGAIN</strong> or <strong>EWOULDBLOCK</strong>, or <strong>EINPROGRESS</strong> (for <strong>connect</strong>(2)) just as if the socket was specified to be nonblocking. If the timeout is set to zero (the default), then the operation will never timeout. Timeouts only have effect for system calls that perform socket I/O (e.g., <strong>accept</strong>(2), <strong>connect</strong>(2), <strong>read</strong>(2), <strong>recvmsg</strong>(2), <strong>send</strong>(2), <strong>sendmsg</strong>(2)); timeouts have no effect for <strong>select</strong>(2), <strong>poll</strong>(2), <strong>epoll_wait</strong>(2), and so on.</p>
</dd>
<dt><strong>SO_REUSEADDR</strong></dt>
<dd>
<p>Indicates that the rules used in validating addresses supplied in a <strong>bind</strong>(2) call should allow reuse of local addresses. For <strong>AF_INET</strong> sockets this means that a socket may bind, except when there is an active listening socket bound to the address. When the listening socket is bound to <strong>INADDR_ANY</strong> with a specific port then it is not possible to bind to this port for any local address. Argument is an integer boolean flag.</p>
</dd>
<dt><strong>SO_REUSEPORT</strong> (since Linux 3.9)</dt>
<dd>
<p>Permits multiple <strong>AF_INET</strong> or <strong>AF_INET6</strong> sockets to be bound to an identical socket address. This option must be set on each socket (including the first socket) prior to calling <strong>bind</strong>(2) on the socket. To prevent port hijacking, all of the processes binding to the same address must have the same effective UID. This option can be employed with both TCP and UDP sockets.</p>
<p>For TCP sockets, this option allows <strong>accept</strong>(2) load distribution in a multi-threaded server to be improved by using a distinct listener socket for each thread. This provides improved load distribution as compared to traditional techniques such using a single <strong>accept</strong>(2)ing thread that distributes connections, or having multiple threads that compete to <strong>accept</strong>(2) from the same socket.</p>
<p>For UDP sockets, the use of this option can provide better distribution of incoming datagrams to multiple processes (or threads) as compared to the traditional technique of having multiple processes compete to receive datagrams on the same socket.</p>
</dd>
<dt><strong>SO_RXQ_OVFL</strong> (since Linux 2.6.33)</dt>
<dd>
<p>Indicates that an unsigned 32-bit value ancillary message (cmsg) should be attached to received skbs indicating the number of packets dropped by the socket since its creation.</p>
</dd>
<dt><strong>SO_SELECT_ERR_QUEUE</strong> (since Linux 3.10)</dt>
<dd>
<p>When this option is set on a socket, an error condition on a socket causes notification not only via the <em>exceptfds</em> set of <strong>select</strong>(2). Similarly, <strong>poll</strong>(2) also returns a <strong>POLLPRI</strong> whenever an <strong>POLLERR</strong> event is returned.</p>
<p>Background: this option was added when waking up on an error condition occurred only via the <em>readfds</em> and <em>writefds</em> sets of <strong>select</strong>(2). The option was added to allow monitoring for error conditions via the <em>exceptfds</em> argument without simultaneously having to receive notifications (via <em>readfds</em>) for regular data that can be read from the socket. After changes in Linux 4.16, the use of this flag to achieve the desired notifications is no longer necessary. This option is nevertheless retained for backwards compatibility.</p>
</dd>
<dt><strong>SO_SNDBUF</strong></dt>
<dd>
<p>Sets or gets the maximum socket send buffer in bytes. The kernel doubles this value (to allow space for bookkeeping overhead) when it is set using <strong>setsockopt</strong>(2), and this doubled value is returned by <strong>getsockopt</strong>(2). The default value is set by the <em>/proc/sys/net/core/wmem_default</em> file and the maximum allowed value is set by the <em>/proc/sys/net/core/wmem_max</em> file. The minimum (doubled) value for this option is 2048.</p>
</dd>
<dt><strong>SO_SNDBUFFORCE</strong> (since Linux 2.6.14)</dt>
<dd>
<p>Using this socket option, a privileged (<strong>CAP_NET_ADMIN</strong>) process can perform the same task as <strong>SO_SNDBUF</strong>, but the <em>wmem_max</em> limit can be overridden.</p>
</dd>
<dt><strong>SO_TIMESTAMP</strong></dt>
<dd>
<p>Enable or disable the receiving of the <strong>SO_TIMESTAMP</strong> control message. The timestamp control message is sent with level <strong>SOL_SOCKET</strong> and a <em>cmsg_type</em> of <strong>SCM_TIMESTAMP</strong>. The <em>cmsg_data</em> field is a <em>struct timeval</em> indicating the reception time of the last packet passed to the user in this call. See <strong>cmsg</strong>(3) for details on control messages.</p>
</dd>
<dt><strong>SO_TIMESTAMPNS</strong> (since Linux 2.6.22)</dt>
<dd>
<p>Enable or disable the receiving of the <strong>SO_TIMESTAMPNS</strong> control message. The timestamp control message is sent with level <strong>SOL_SOCKET</strong> and a <em>cmsg_type</em> of <strong>SCM_TIMESTAMPNS</strong>. The <em>cmsg_data</em> field is a <em>struct timespec</em> indicating the reception time of the last packet passed to the user in this call. The clock used for the timestamp is <strong>CLOCK_REALTIME</strong>. See <strong>cmsg</strong>(3) for details on control messages.</p>
<p>A socket cannot mix <strong>SO_TIMESTAMP</strong> and <strong>SO_TIMESTAMPNS</strong>: the two modes are mutually exclusive.</p>
</dd>
<dt><strong>SO_TYPE</strong></dt>
<dd>
<p>Gets the socket type as an integer (e.g., <strong>SOCK_STREAM</strong>). This socket option is read-only.</p>
</dd>
<dt><strong>SO_BUSY_POLL</strong> (since Linux 3.11)</dt>
<dd>
<p>Sets the approximate time in microseconds to busy poll on a blocking receive when there is no data. Increasing this value requires <strong>CAP_NET_ADMIN</strong>. The default for this option is controlled by the <em>/proc/sys/net/core/busy_read</em> file.</p>
<p>The value in the <em>/proc/sys/net/core/busy_poll</em> file determines how long <strong>select</strong>(2) and <strong>poll</strong>(2) will busy poll when they operate on sockets with <strong>SO_BUSY_POLL</strong> set and no events to report are found.</p>
<p>In both cases, busy polling will only be done when the socket last received data from a network device that supports this option.</p>
<p>While busy polling may improve latency of some applications, care must be taken when using it since this will increase both CPU utilization and power usage.</p>
</dd>

