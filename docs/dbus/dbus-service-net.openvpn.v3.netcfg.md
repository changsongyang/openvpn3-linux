OpenVPN 3 D-Bus API: Network configuration service
==================================================

This service is only accessible and used by the client service.

The interface provided here is similar to the
[VPNService interface of Android](https://developer.android.com/reference/android/net/VpnService)
but adapted for Linux and D-Bus.

When this backend process have been idle for a short time, it will
terminate itself automatically. It is only needed while a VPN connection
is active.

D-Bus destination: `net.openvpn.v3.netcfg` \- Object path: `/net/openvpn/v3/netcfg`
---------------------------------------------------------------------------------------
```
node /net/openvpn/v3/netcfg {
  interface net.openvpn.v3.netcfg {
    methods:
      CreateVirtualInterface(in  s dev_name,
                             out o device_path);
      FetchInterfaceList(out ao device_paths);
      ProtectSocket(in  s remote,
                    in  b ipv6,
                    out b succeded);
      NotificationSubscribe(in  u filter);
      NotificationUnsubscribe(in  s optional_subscriber);
      NotificationSubscriberList(out a(su) subscriptions);
    signals:
      Log(u group,
          u level,
          s message);
    properties:
       readwrite u log_level;
  };
};
```


### Method: `net.openvpn.v3.netcfg.CreateVirtualInterface`

Create a virtual interface and return the object path of the new interface.

#### Arguments
| Direction | Name         | Type        | Description                                                      |
|-----------|--------------|-------------|------------------------------------------------------------------|
| In        | dev_name     | string      | A user friendly name for the device, will be part of device_path |
| Out       | device_path  | object path | A unique D-Bus object path for create device                     |

## Method: `net.openvpn.v3.netcfg.FetchInterfaceList`

This method will return an array of object paths to virtual interfaces the
caller is granted access to.

#### Arguments
| Direction | Name        | Type         | Description                                                |
|-----------|-------------|--------------|------------------------------------------------------------|
| Out       | paths       | object paths | An array of object paths to accessible virtual interfaces  |


### Method: `net.openvpn.v3.netcfg.ProtectSocket`

This method is called by the service client to signal that a socket
needs to be `protected` from being routed over the VPN to avoid routing
loops. The method of how this is actually implemented can be controlled
by command line arguments to the netcfg service process.

#### Arguments

This method also

| Direction | Name         | Type        | Description                                                |
|-----------|--------------|-------------|------------------------------------------------------------|
| In        |              | fdlist      | File descriptor of the socket to protect [1]. Only the first provided fd is being processed. |
| In        | remote       | string      | The remote host this socket is connected to.               |
| In        | ipv6         | boolean     | The initial process ID (PID) of the VPN backend client.    |

[1] Unix file descriptors that are passed are not in the D-Bus method signature


### Method: `net.openvpn.v3.netcfg.NotificationSubscribe`

A service which wants to respond to various network change activities triggered
by OpenVPN can subscribe to the `net.openvpn.v3.netcfg.NetworkChange`
signal.  Such subscriptions are handled by calling this method.

The default OpenVPN 3 Linux D-Bus policy will restrict such subscriptions to
processes running as the `openvpn` user account.

This method takes a filter mask as input parameter.  The possible values are:

| Change Event Type  |  bit field  |  value  | Description                                                                   |
|--------------------|-------------|---------|-------------------------------------------------------------------------------|
| DEVICE_ADDED       |        0    |       1 |  A new virtual interface has been added on the system                         |
| DEVICE_REMOVED     |        1    |       2 |  A virtual interface has been removed from the system                         |
| IPADDR_ADDED       |        2    |       4 |  An IP address has been added to a virtual interface                          |
| IPADDR_REMOVED     |        3    |       8 |  An IP address has been removed from the virtual interface                    |
| ROUTE_ADDED        |        4    |      16 |  A route has been added to the routing table, related to this interface       |
| ROUTE_REMOVED      |        5    |      32 |  A route has been remove from the routing table, related to this interface    |
| ROUTE_EXCLUDED     |        6    |      64 |  A route has been excluded from the routing table, related to this interface  |
| DNS_SERVER_ADDED   |        7    |     128 |  A DNS server has been added to the DNS configuration                         |
| DNS_SERVER_REMOVED |        8    |     256 |  A DNS server has been removed from the DNS configuration                     |
| DNS_SEARCH_ADDED   |        9    |     512 |  A DNS search domain has been added to the DNS configuration                  |
| DNS_SEARCH_REMOVED |       10    |    1024 |  A DNS search domain has been removed from the DNS configuration              |

To subscribe to several change event types, the values must be added together
when being sent to the subscription method.  If you want to subscribe to
IP addresses being added and removed, you use `4 + 8 = 12`.  The
subscription filter value will then be `12`.

#### Arguments

| Direction | Name         | Type             | Description                                                |
|-----------|--------------|------------------|------------------------------------------------------------|
| In        | filter       | unsigned integer | A filter mask defining which NetworkChange events to subscribe to.  Valid values are `1`  to `2047` |


### Method: `net.openvpn.v3.netcfg.NotificationUnubscribe`

Any services who has subscribed to NetworkChange signals must unsubscribe
before disconnecting from the D-Bus.  This is done by calling this method.

The subscriber argument this method needs should always be an empty string.
Processes running as `root` can send the the unique D-Bus name to forcefully
subscribe a specific subscription.

#### Arguments

| Direction | Name                | Type    | Description                                                |
|-----------|---------------------|---------|------------------------------------------------------------|
| In        | optional_subscriber | string  | This should be empty for non-root users.  Must otherwise contain a valid unique D-Bus name |



### Method: `net.openvpn.v3.netcfg.NotificationSubscriberList`

Retrieves a list of all active subscriptions together with their filter mask.

This method is restricted to the `root` user.

#### Arguments

| Direction | Name           | Type                        | Description                                                |
|-----------|----------------|-----------------------------|------------------------------------------------------------|
| Out       | subscriptions  | array(string, unsigned int) | An array of tuples with the subscribers unique D-Bus name (string) and the attached filter mask (unsigned int) |


### Signal: `net.openvpn.v3.netcfg.Log`

Whenever the backend process starter needs to log something, it issues
a Log signal which carries a log group, log verbosity level and a
string with the log message itself. See the separate [logging
documentation](dbus-logging.md) for details on this signal.

### `Properties`
| Name          | Type             | Read/Write | Description                                         |
|---------------|------------------|:----------:|-----------------------------------------------------|
| log_level     | unsigned integer | read-write | Controls the log verbosity of messages intended to be proxied to the user front-end. **Note:** Not currently implemented |


D-Bus destination: `net.openvpn.v3.netcfg` \- Object path: `/net/openvpn/v3/netcfg/${UNIQUE_ID}`
--------------------------------------------------------------------------------------------------------------


```
node /net/openvpn/v3/netcfg/${UNIQUE_ID} {
interface net.openvpn.v3.netcfg {
    methods:
      AddBypassRoute(in  s ip_address,
                   in  b ipv6);
      AddIPAddress(in  s ip_address,
                   in  u prefix,
                   in  s gateway,
                   in  b ipv6);
      SetRemoteAddress(in  s ip_address,
                       in  b ipv6);
      AddNetworks(in  a(subb) networks);
      AddDNS(in  as server_list);
      AddDNSSearch(in  as domains);
      Establish();
      Disable();
      Destroy();
    signals:
      Log(u group,
          u level,
          s message);
      NetworkChange(u type,
                    s device,
                    s details);
    properties:
      readonly u owner;
      readonly au acl;
      readonly b active;
      readonly b modified;
      readonly as dns_search;
      readonly as dns_servers;
      readwrite u layer;
      readwrite u mtu;
      readwrite b reroute_ipv4;
      readwrite b reroute_ipv6;
      readwrite u txqueuelen;
  };
};
```

### Method: `net.openvpn.v3.netcfg.AddBypassRoute`

Excludes traffic to remote from tunnel routing. This is the first method called by client
service before establishing connection and before reconnect.

#### Arguments

| Direction | Name         | Type        | Description                                                                  |
|-----------|--------------|-------------|------------------------------------------------------------------------------|
| In        | ip_address   | string      | The IP address in string representation (e.g. 198.51.100.12 or 2001:db8::23) |
| In        | ipv6         | ipv6        | Is the new IP address IPv6 or IPv4                                           |


### Method: `net.openvpn.v3.netcfg.AddIPAddress`

Adds a new local IP Address to the VPN configuration of the virtual interface

#### Arguments

| Direction | Name         | Type        | Description                                                                  |
|-----------|--------------|-------------|------------------------------------------------------------------------------|
| In        | ip_address   | string      | The IP address in string representation (e.g. 198.51.100.12 or 2001:db8::23) |
| In        | prefix       | unsigned integer | The prefix length. (e.g. /24 or /64)                                         |
| In        | gateway      | string      | The IP address in string representation of the remote gateway inside the VPN |
| In        | ipv6         | ipv6        | Is the new IP address IPv6 or IPv4                                           |


### Method: `net.openvpn.v3.netcfg.SetRemoteAddress`

Set the remote address of the VPN server. This is the address the VPN
uses to connect to VPN server. This is used when creating direct routes
to the VPN server to avoid routing loops (redirect gateway option).

#### Arguments
| Direction | Name         | Type        | Description                                                                  |
|-----------|--------------|-------------|------------------------------------------------------------------------------|
| In        | ip_address   | string      | The IP address in string representation (e.g. 198.51.100.12 or 2001:db8::23) |
| In        | ipv6         | ipv6        | Is the IP address IPv6 or IPv4                                               |


### Method: `net.openvpn.v3.netcfg.AddNetworks`

Specifies a array of networks that should be either routed over the VPN or
explicitly not routed over the VPN. Conflicts between included and excluded
are resolved in the usual longest-prefix matching fashion.

#### Arguments
| Direction | Name         | Type        | Description                                                                  |
|-----------|--------------|-------------|------------------------------------------------------------------------------|
| In        | networks     | a(subb)     | An array of networks                                                         | 

A network is specified in the following way:

 | Name         | Type        | Description                                                                  |
 |--------------|-------------|------------------------------------------------------------------------------|
 | ip_address   | string      | The network IP address (the first IP in the network)                         |
 | prefix       | unsigned integer | The prefix of the network (e.g. /24 or /64)                                  |
 | ipv6         | boolean     | Is this a IPv6 or IPv4 network specification                                 |
 | exclude      | boolean     | If true, exclude (do not route) otherwise include (do route) this network over the VPN |


### Method: `net.openvpn.v3.netcfg.AddDNS`

Specifies a array of DNS server addresses that should be added to the list of DNS
server of the virtual interface.

#### Arguments
| Direction | Name         | Type              | Description                                              |
|-----------|--------------|-------------------|----------------------------------------------------------|
| In        | server_list  | array of strings  | An array of DNS server IP addresses.                     |


### Method: `net.openvpn.v3.netcfg.AddDNSSearch`

Specifies a array of DNS search domains that should be added to the list
of DNS search to the network.

#### Arguments
| Direction | Name         | Type              | Description                                              |
|-----------|--------------|-------------------|----------------------------------------------------------|
| In        | domains      | array of strings  | An array of DNS domains                                  |


### Method: `net.openvpn.v3.netcfg.Establish`

Uses all the information provided to the interface to setup a tun device
and set routes, DNS and interface accordingly. The resulting tun device
is returned to the caller.

#### Arguments
| Direction | Name         | Type              | Description                                                |
|-----------|--------------|-------------------|------------------------------------------------------------|
| Out       |              | fdlist            | The file descriptor corresponding to the new tun device [1]|

[1] Unix file descriptors that are passed are not in the D-Bus method signature.


### Method: `net.openvpn.v3.netcfg.Disable`

Indicates that the interface is temporarily not used by the VPN service.
E.g. that the VPN connection is disconnected and currently reconnecting.
**Note:** This is currently not implemented.


### Method: `net.openvpn.v3.netcfg.Destroy`

Removes the virtual interface and undoes the configuration (routes, DNS, tun device configuration). The
calling application must close the tun device own its own.


### Signal: `net.openvpn.v3.netcfg.NetworkChange`

This signal indicates that something has changed in the systems network
configuration.  These signals will be tied to the interface which triggered
this change.

| Name      | Type       | Description                                     |
|-----------|------------|-------------------------------------------------|
| type      | uint       | `NetCfgChangeType` reference of the request. See [`src/netcfg/netcfg-changeevent.hpp`](src/netcfg/netcfg-changeevent.hpp) for details.  |
| device    | string     | The virtual network device name related to this change. |
| details   | dictionary | Structured details of the change event |

The contents of the `details` dictionary depends on the change type.  Below
will the different change types which provides information be listed.  Events
not providing any details are not mentioned.

#### NetworkChange type `IPADDR_ADDED` and `IPADDR_REMOVED`

| Key           | Description                                                               |
|---------------|---------------------------------------------------------------------------|
| ip_version    | Will be `4` or `6` which refers to IPv4 or IPv6                           |
| ip_address    | The IP address added to/removed from the interface                        |
| prefix        | The subnet prefix of the IP address                                       |

#### NetworkChange type `ROUTE_ADDED`, `ROUTE_REMOVED` and `ROUTE_EXCLUDED`

| Key           | Description                                                               |
|---------------|---------------------------------------------------------------------------|
| ip_version    | Will be `4` or `6` which refers to IPv4 or IPv6                           |
| subnet        | Network IP address for the route which was added/removed/excluded         |
| prefix        | The subnet prefix of the subnet IP address                                |
| gateway       | The gateway configured for this subnet (not present when removing routes) |

#### NetworkChange type `DNS_SERVER_ADDED` and `DNS_SERVER_REMOVED`
| Key           | Description                                                               |
|---------------|---------------------------------------------------------------------------|
| dns_server    | IP address containing the DNS server being added/removed                  |

#### NetworkChange type `DNS_SEARCH_ADDED` and `DNS_SEARCH_REMOVED`
| Key           | Description                                                               |
|---------------|---------------------------------------------------------------------------|
| search_domain | DNS search domain being added/removed                                     |


### `Properties`
| Name          | Type             | Read/Write | Description                                                   |
|---------------|------------------|:----------:|---------------------------------------------------------------|
| owner         | unsigned integer | Read-only  | The UID value of the user which did the import                |
| acl           | array(integer)   | Read-only  | An array of UID values granted access                         |
| log_level     | unsigned integer | read-write | Controls the log verbosity of messages intended to be proxied to the user front-end. **Note:** Not currently implemented |
| active        | boolean          | Read-only  | If the VPN is active (Establish has been successfully called) |
| dns_servers   | array of strings | Read-only  | Return the array of DNS servers                               |
| dns_search    | array of strings | Read-only  | Return the array of DNS search domains                        |
| layer         | unsigned integer             | Read-write | Sets the layer for the VPN to use, 3 for IP (tun device). Setting to 2 (tap device) is currently not implemented |
| mtu           | unsigned integer | Read-write | Sets the MTU for the tun device. Default is 1500              |
| reroute_ipv4  | boolean          | Read-write | Setting this to true, tells the service that the default route should be pointed to the VPN and that mechanism to avoid routing loops should be taken |
| reroute_ipv6  | boolean          | Read-Write | As reroute_ipv4 but for IPv6                                  |
| txqueuelen    | unsigned integer | Read-Write | Set the TX queue length of the tun device. If set to 0 or unset, the default from the operating system is used instead |
