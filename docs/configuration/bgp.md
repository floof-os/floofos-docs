# BGP Configuration

This section covers BGP (Border Gateway Protocol) configuration in FloofOS using BIRD2 as the routing daemon. FloofOS uses [Pathvector](https://pathvector.io) for BGP automation built on top of BIRD2.

---

## Global Settings

### Required Settings

```
set bgp asn <number>
set bgp router-id <ip-address>
set bgp prefixes <cidr>
```

| Command | Description | Example |
|---------|-------------|---------|
| `set bgp asn <number>` | Set your Autonomous System Number (1-4294967295) | `set bgp asn 65000` |
| `set bgp router-id <ip>` | Set BGP router identifier | `set bgp router-id 192.0.2.1` |
| `set bgp prefixes <cidr>` | Add a prefix to announce (can be repeated) | `set bgp prefixes 192.0.2.0/24` |

### IRR and RPKI Settings

| Command | Description | Example |
|---------|-------------|---------|
| `set bgp rtr-server <server:port>` | Set RPKI RTR server | `set bgp rtr-server 172.65.0.2:8282` |
| `set bgp irr-server <server>` | Set IRR server | `set bgp irr-server rr.ntt.net` |
| `set bgp irr-query-timeout <seconds>` | IRR query timeout | `set bgp irr-query-timeout 30` |
| `set bgp bgpq-args "<args>"` | Set BGPQ4 arguments | `set bgp bgpq-args "-S AFRINIC,APNIC,ARIN,LACNIC,RIPE"` |
| `set bgp rpki-enable` | Enable RPKI globally | `set bgp rpki-enable` |
| `set bgp rpki-enable disable` | Disable RPKI globally | `set bgp rpki-enable disable` |

### PeeringDB Settings

| Command | Description | Example |
|---------|-------------|---------|
| `set bgp peeringdb-api-key "<key>"` | Set PeeringDB API key | `set bgp peeringdb-api-key "abc123"` |
| `set bgp peeringdb-url <url>` | Set PeeringDB URL | `set bgp peeringdb-url https://peeringdb.com` |
| `set bgp peeringdb-query-timeout <sec>` | PeeringDB query timeout | `set bgp peeringdb-query-timeout 30` |
| `set bgp peeringdb-cache` | Enable PeeringDB cache | `set bgp peeringdb-cache` |
| `set bgp peeringdb-cache disable` | Disable PeeringDB cache | `set bgp peeringdb-cache disable` |

### Community Settings

| Command | Description | Example |
|---------|-------------|---------|
| `set bgp local-communities <community>` | Add local community tag | `set bgp local-communities 65000:1:0` |
| `set bgp origin-communities <community>` | Add origin community tag | `set bgp origin-communities 65000:1:0` |
| `set bgp prefix-communities <cidr> <comm>` | Add per-prefix community | `set bgp prefix-communities 192.0.2.0/24 65000:100:1` |

### Route Behavior

| Command | Description |
|---------|-------------|
| `set bgp default-route` | Enable default route redistribution |
| `set bgp default-route disable` | Disable default route redistribution |
| `set bgp accept-default` | Accept default routes from peers |
| `set bgp accept-default disable` | Reject default routes from peers |
| `set bgp merge-paths` | Enable ECMP merge-paths |
| `set bgp merge-paths disable` | Disable ECMP merge-paths |
| `set bgp keep-filtered` | Keep filtered routes in memory |
| `set bgp no-announce` | Disable all announcements |
| `set bgp no-accept` | Reject all incoming routes |
| `set bgp stun` | Enable STUN |

### Source Address

| Command | Description | Example |
|---------|-------------|---------|
| `set bgp source4 <ipv4>` | Set source IPv4 address | `set bgp source4 192.0.2.1` |
| `set bgp source6 <ipv6>` | Set source IPv6 address | `set bgp source6 2001:db8::1` |

### Blocklist and Filtering

| Command | Description | Example |
|---------|-------------|---------|
| `set bgp blocklist <value>` | Add to prefix blocklist | `set bgp blocklist 198.51.100.0/24` |
| `set bgp blocklist-urls <url>` | Add blocklist URL source | `set bgp blocklist-urls https://example.com/blocklist.txt` |
| `set bgp blocklist-files <path>` | Add blocklist file source | `set bgp blocklist-files /etc/bgp-blocklist.txt` |
| `set bgp bogon-asns <asn>` | Add bogon ASN | `set bgp bogon-asns 64496` |
| `set bgp bogons4 <cidr>` | Add custom IPv4 bogon | `set bgp bogons4 192.168.0.0/16` |
| `set bgp bogons6 <cidr>` | Add custom IPv6 bogon | `set bgp bogons6 fc00::/7` |
| `set bgp transit-asns <asn>` | Add transit ASN for filtering | `set bgp transit-asns 174` |
| `set bgp blackhole-bogon-asns` | Blackhole routes with bogon ASNs | `set bgp blackhole-bogon-asns` |
| `set bgp authorized-providers <prefix> <asn>` | Authorize provider for prefix | `set bgp authorized-providers 192.0.2.0/24 174` |

### Advanced Global

| Command | Description | Example |
|---------|-------------|---------|
| `set bgp confederation <number>` | Set confederation identifier | `set bgp confederation 65000` |
| `set bgp raw-config "<bird-config>"` | Inject raw BIRD configuration | `set bgp raw-config "protocol static { ... }"` |

---

## Policy (Template) Configuration

Policies define reusable sets of BGP attributes that can be applied to multiple peers. Define policies first, then reference them when configuring peers.

```
set bgp policy <name> <group> <option> [value]
```

Available option groups: `session`, `route`, `community`, `filter`, `limit`, `security`, `role`, `capability`, `hook`. Each group accepts the same options as documented in the [Peer Configuration](#peer-configuration) section below.

### Example - Upstream Policy

```
set bgp policy upstream route local-pref 100
set bgp policy upstream community add-on-import 65000:1:1
set bgp policy upstream community announce 65000:1:0
set bgp policy upstream community announce 65000:1:4
set bgp policy upstream filter filter-rpki
set bgp policy upstream filter filter-bogon-routes
set bgp policy upstream filter filter-bogon-asns
set bgp policy upstream limit import-limit4 1500000
set bgp policy upstream limit import-limit6 300000
set bgp policy upstream route allow-local-as
set bgp policy upstream community remove-all-communities 65000
```

### Example - Route Server Policy

```
set bgp policy routeserver route local-pref 200
set bgp policy routeserver community add-on-import 65000:1:2
set bgp policy routeserver community announce 65000:1:0
set bgp policy routeserver community announce 65000:1:4
set bgp policy routeserver filter filter-rpki
set bgp policy routeserver filter filter-transit-asns
set bgp policy routeserver filter filter-bogon-routes
set bgp policy routeserver filter filter-bogon-asns
set bgp policy routeserver limit auto-import-limits
set bgp policy routeserver security enforce-first-as disable
set bgp policy routeserver security enforce-peer-nexthop disable
set bgp policy routeserver community remove-all-communities 65000
```

### Example - Peer Policy

```
set bgp policy peer route local-pref 300
set bgp policy peer community add-on-import 65000:1:3
set bgp policy peer community announce 65000:1:0
set bgp policy peer community announce 65000:1:4
set bgp policy peer filter filter-rpki
set bgp policy peer filter filter-irr
set bgp policy peer filter filter-transit-asns
set bgp policy peer filter filter-bogon-routes
set bgp policy peer filter filter-bogon-asns
set bgp policy peer limit auto-as-set
set bgp policy peer limit auto-import-limits
set bgp policy peer community remove-all-communities 65000
```

### Example - Downstream Policy

```
set bgp policy downstream route local-pref 400
set bgp policy downstream community add-on-import 65000:1:4
set bgp policy downstream community announce 65000:1:0
set bgp policy downstream community announce 65000:1:1
set bgp policy downstream community announce 65000:1:2
set bgp policy downstream community announce 65000:1:3
set bgp policy downstream filter filter-rpki
set bgp policy downstream filter filter-irr
set bgp policy downstream filter filter-transit-asns
set bgp policy downstream filter filter-bogon-routes
set bgp policy downstream filter filter-bogon-asns
set bgp policy downstream limit auto-as-set
set bgp policy downstream limit auto-import-limits
set bgp policy downstream community allow-blackhole-community
set bgp policy downstream community remove-all-communities 65000
```

### Example - iBGP Policy

```
set bgp policy ibgp route local-pref 150
set bgp policy ibgp community add-on-import 65000:1:5
set bgp policy ibgp community announce 65000:1:0
set bgp policy ibgp community announce 65000:1:1
set bgp policy ibgp community announce 65000:1:2
set bgp policy ibgp community announce 65000:1:3
set bgp policy ibgp community announce 65000:1:4
set bgp policy ibgp community announce 65000:1:5
set bgp policy ibgp route next-hop-self
set bgp policy ibgp session direct
set bgp policy ibgp route allow-local-as
set bgp policy ibgp security enforce-first-as disable
set bgp policy ibgp security enforce-peer-nexthop disable
set bgp policy ibgp filter filter-irr disable
set bgp policy ibgp filter filter-rpki disable
set bgp policy ibgp community remove-all-communities 65000
```

### Policy Reference

| Template | Local Preference | Use Case |
|----------|------------------|----------|
| `upstream` | 100 | Transit providers |
| `ibgp` | 150 | Internal BGP |
| `routeserver` | 200 | IXP route servers |
| `peer` | 300 | Bilateral/PNI peers |
| `downstream` | 400 | Customers |

Higher local preference values are preferred during BGP best path selection.

---

## Peer Configuration

Peers are configured using the `set bgp peer` command tree. Each peer is identified by a unique name.

### Basic Peer Settings

```
set bgp peer <name> policy <policy-name>
set bgp peer <name> remote-as <asn>
set bgp peer <name> neighbor <ip>
set bgp peer <name> description "<text>"
set bgp peer <name> tags <tag>
set bgp peer <name> shutdown
set bgp peer <name> shutdown disable
```

**Example - Add an upstream peer:**

```
set bgp peer Cogent policy upstream
set bgp peer Cogent remote-as 174
set bgp peer Cogent neighbor 154.54.0.1
set bgp peer Cogent description "Cogent Transit"
```

**Example - Add an IXP route server peer:**

```
set bgp peer DE-CIX_RS policy routeserver
set bgp peer DE-CIX_RS remote-as 6695
set bgp peer DE-CIX_RS neighbor 80.81.192.157
set bgp peer DE-CIX_RS neighbor 80.81.192.158
```

**Example - Add an iBGP peer:**

```
set bgp peer Router2 policy ibgp
set bgp peer Router2 remote-as 65000
set bgp peer Router2 neighbor 10.255.0.2
set bgp peer Router2 session listen4 10.255.0.1
set bgp peer Router2 session multihop
```

### Session Options

Configured via `set bgp peer <name> session <option> [value]`:

| Option | Description | Example |
|--------|-------------|---------|
| `password "<pwd>"` | Set BGP MD5 password | `session password "secret123"` |
| `multihop` | Enable eBGP multihop | `session multihop` |
| `passive` | Enable passive mode (wait for connection) | `session passive` |
| `direct` | Enable direct connection mode | `session direct` |
| `listen4 <ip>` | Set local IPv4 listen address | `session listen4 10.0.0.1` |
| `listen6 <ip>` | Set local IPv6 listen address | `session listen6 2001:db8::1` |
| `local-port <port>` | Set local BGP port | `session local-port 1790` |
| `neighbor-port <port>` | Set neighbor BGP port | `session neighbor-port 1790` |
| `bfd` | Enable BFD (Bidirectional Forwarding Detection) | `session bfd` |

### Route Options

Configured via `set bgp peer <name> route <option> [value]`:

| Option | Description | Example |
|--------|-------------|---------|
| `local-pref <value>` | Set local preference | `route local-pref 200` |
| `local-pref4 <value>` | Set IPv4-specific local preference | `route local-pref4 150` |
| `local-pref6 <value>` | Set IPv6-specific local preference | `route local-pref6 150` |
| `default-local-pref <value>` | Set default local preference | `route default-local-pref 100` |
| `allow-local-as` | Allow local AS in path | `route allow-local-as` |
| `remove-private-asns` | Remove private ASNs from path | `route remove-private-asns` |
| `prefer-older-routes` | Prefer older routes | `route prefer-older-routes` |
| `announce-default` | Announce default route to peer | `route announce-default` |
| `announce-originated` | Announce originated routes | `route announce-originated` |
| `announce-all` | Announce all routes | `route announce-all` |
| `next-hop-self` | Set next-hop to self | `route next-hop-self` |
| `next-hop-self-ebgp` | Set next-hop-self for eBGP | `route next-hop-self-ebgp` |
| `next-hop-self-ibgp` | Set next-hop-self for iBGP | `route next-hop-self-ibgp` |
| `prepends <count>` | Set AS-path prepend count | `route prepends 2` |
| `prepend-path <asn>` | Add ASN to prepend path | `route prepend-path 65000` |
| `clear-path` | Clear AS-path | `route clear-path` |
| `transit-lock <asn>` | Lock routes to specific transit | `route transit-lock 174` |
| `as-prefs <asn> <pref>` | Set per-AS preference | `route as-prefs 13335 250` |
| `import-next-hop <ip>` | Override import next-hop | `route import-next-hop 10.0.0.1` |
| `export-next-hop <ip>` | Override export next-hop | `route export-next-hop 10.0.0.1` |
| `set-local-pref` | Enable local-pref setting | `route set-local-pref` |

Append `disable` to any boolean option to disable it (e.g., `route next-hop-self disable`).

### Community Options

Configured via `set bgp peer <name> community <option> [value]`:

| Option | Description | Example |
|--------|-------------|---------|
| `add-on-import <community>` | Tag imported routes with community | `community add-on-import 65000:1:1` |
| `add-on-export <community>` | Tag exported routes with community | `community add-on-export 65000:2:1` |
| `announce <community>` | Announce routes matching community | `community announce 65000:1:0` |
| `remove-communities <community>` | Strip community on import | `community remove-communities 65000:999:0` |
| `remove-all-communities <asn>` | Strip all communities for ASN | `community remove-all-communities 65000` |
| `prefix-communities <cidr> <comm>` | Per-prefix community tagging | `community prefix-communities 192.0.2.0/24 65000:100:1` |
| `interpret-communities` | Enable community interpretation | `community interpret-communities` |
| `allow-blackhole-community` | Allow blackhole community from peer | `community allow-blackhole-community` |
| `blackhole-in` | Enable blackhole on import | `community blackhole-in` |
| `blackhole-out` | Enable blackhole on export | `community blackhole-out` |
| `community-prefs <comm> <pref>` | Set preference based on community | `community community-prefs 65000:1:1 200` |

### Filter Options

Configured via `set bgp peer <name> filter <option> [value]`:

| Option | Description | Example |
|--------|-------------|---------|
| `filter-rpki` | Enable RPKI validation | `filter filter-rpki` |
| `filter-irr` | Enable IRR prefix filtering | `filter filter-irr` |
| `filter-bogon-routes` | Filter bogon routes | `filter filter-bogon-routes` |
| `filter-bogon-asns` | Filter bogon ASNs | `filter filter-bogon-asns` |
| `filter-transit-asns` | Filter transit ASNs | `filter filter-transit-asns` |
| `filter-as-set` | Filter by AS-SET | `filter filter-as-set` |
| `filter-blocklist` | Apply blocklist filter | `filter filter-blocklist` |
| `filter-prefix-length` | Filter by prefix length | `filter filter-prefix-length` |
| `filter-max-prefix` | Filter by max prefix | `filter filter-max-prefix` |
| `filter-never-via-route-servers` | Filter never-via-RS | `filter filter-never-via-route-servers` |
| `filter-aspa` | Enable ASPA validation | `filter filter-aspa` |
| `strict-rpki` | Enable strict RPKI mode | `filter strict-rpki` |
| `irr-accept-child-prefixes` | Accept child prefixes from IRR | `filter irr-accept-child-prefixes` |
| `dont-announce <cidr>` | Don't announce specific prefix | `filter dont-announce 10.0.0.0/8` |
| `only-announce <cidr>` | Only announce specific prefix | `filter only-announce 192.0.2.0/24` |
| `prefixes <cidr>` | Set peer-specific allowed prefixes | `filter prefixes 203.0.113.0/24` |
| `as-set <as-set>` | Set peer AS-SET | `filter as-set AS-CUSTOMER` |
| `as-set-members <asn>` | Add AS-SET member | `filter as-set-members 65001` |
| `dont-receive <cidr>` | Reject specific prefix from peer | `filter dont-receive 10.0.0.0/8` |
| `only-receive <cidr>` | Only accept specific prefix from peer | `filter only-receive 203.0.113.0/24` |

Append `disable` to boolean options to disable them (e.g., `filter filter-rpki disable`).

### Limit Options

Configured via `set bgp peer <name> limit <option> [value]`:

| Option | Description | Example |
|--------|-------------|---------|
| `import-limit4 <number>` | Max IPv4 imported prefixes | `limit import-limit4 1500000` |
| `import-limit6 <number>` | Max IPv6 imported prefixes | `limit import-limit6 300000` |
| `import-limit-violation <action>` | Action on limit violation | `limit import-limit-violation restart` |
| `receive-limit4 <number>` | Max IPv4 received prefixes | `limit receive-limit4 2000000` |
| `receive-limit6 <number>` | Max IPv6 received prefixes | `limit receive-limit6 500000` |
| `receive-limit-violation <action>` | Action on receive limit | `limit receive-limit-violation block` |
| `export-limit4 <number>` | Max IPv4 exported prefixes | `limit export-limit4 1000` |
| `export-limit6 <number>` | Max IPv6 exported prefixes | `limit export-limit6 500` |
| `export-limit-violation <action>` | Action on export limit | `limit export-limit-violation restart` |
| `auto-import-limits` | Auto-set import limits from PeeringDB | `limit auto-import-limits` |
| `auto-as-set` | Auto-discover AS-SET | `limit auto-as-set` |
| `auto-as-set-members` | Auto-discover AS-SET members | `limit auto-as-set-members` |

### Security Options

Configured via `set bgp peer <name> security <option>`:

| Option | Description |
|--------|-------------|
| `enforce-first-as` | Enforce first AS must match peer's ASN |
| `enforce-peer-nexthop` | Enforce next-hop must be peer address |
| `force-peer-nexthop` | Force next-hop to peer address |
| `ttl-security` | Enable TTL security (GTSM) |

### Role Options

Configured via `set bgp peer <name> role <option> [value]`:

| Option | Description |
|--------|-------------|
| `route-reflector` | Mark peer as route reflector client |
| `route-server` | Mark peer as route server client |
| `confederation-member` | Mark peer as confederation member |
| `role <type>` | Set BGP role (`provider`, `rs`, `rs-client`, `customer`, `peer`) |
| `require-roles` | Require BGP roles negotiation |

### Capability Options

Configured via `set bgp peer <name> capability <option>`:

| Option | Description |
|--------|-------------|
| `add-path-tx` | Enable add-path transmit |
| `add-path-rx` | Enable add-path receive |
| `honor-graceful-shutdown` | Honor GRACEFUL_SHUTDOWN community |
| `mp-unicast-46` | Enable multiprotocol IPv4+IPv6 unicast |
| `advertise-hostname` | Advertise hostname capability |
| `disable-after-error` | Disable peer after protocol error |

### Hook Options (Advanced BIRD Filters)

Configured via `set bgp peer <name> hook <option> "<bird-filter-code>"`:

| Option | Description |
|--------|-------------|
| `session-global` | Inject code into BGP session block |
| `pre-import-filter` | Code executed before import filter |
| `post-import-filter` | Code executed after import filter |
| `pre-import-accept` | Code executed before import accept |
| `pre-export` | Code executed before export filter |
| `pre-export-final` | Code executed at end of export filter |

---

## System Configuration

### Kernel Settings

Configured via `set bgp system kernel <option> [value]`:

| Option | Description | Example |
|--------|-------------|---------|
| `learn` | Learn routes from kernel | `set bgp system kernel learn` |
| `export` | Export routes to kernel | `set bgp system kernel export` |
| `table <number>` | Set kernel routing table | `set bgp system kernel table 254` |
| `scan-time <seconds>` | Kernel route scan interval | `set bgp system kernel scan-time 10` |
| `reject-connected` | Reject connected routes | `set bgp system kernel reject-connected` |
| `accept4 <protocol>` | Accept IPv4 routes from protocol | `set bgp system kernel accept4 static` |
| `accept6 <protocol>` | Accept IPv6 routes from protocol | `set bgp system kernel accept6 static` |
| `reject4 <protocol>` | Reject IPv4 routes from protocol | `set bgp system kernel reject4 direct` |
| `reject6 <protocol>` | Reject IPv6 routes from protocol | `set bgp system kernel reject6 direct` |
| `statics <network> <gateway>` | Add static route | `set bgp system kernel statics 0.0.0.0/0 10.0.0.1` |
| `srd-communities <community>` | Add SRD community | `set bgp system kernel srd-communities 65000:666:0` |

### Optimizer Settings

Configured via `set bgp system optimizer <option> [value]`:

| Option | Description | Example |
|--------|-------------|---------|
| `targets <ip>` | Add optimization target | `set bgp system optimizer targets 8.8.8.8` |
| `latency-threshold <ms>` | Set latency threshold | `set bgp system optimizer latency-threshold 100` |
| `packet-loss-threshold <percent>` | Set loss threshold | `set bgp system optimizer packet-loss-threshold 1.5` |
| `modifier <value>` | Set preference modifier | `set bgp system optimizer modifier 10` |
| `probe-count <number>` | Number of probes | `set bgp system optimizer probe-count 5` |
| `probe-timeout <seconds>` | Probe timeout | `set bgp system optimizer probe-timeout 3` |
| `probe-interval <seconds>` | Probe interval | `set bgp system optimizer probe-interval 60` |
| `cache-size <number>` | Optimizer cache size | `set bgp system optimizer cache-size 1000` |
| `probe-udp` | Use UDP for probes | `set bgp system optimizer probe-udp` |
| `exit-on-cache-full` | Exit when cache is full | `set bgp system optimizer exit-on-cache-full` |

---

## BGP Logging

```
set bgp logging enable
set bgp logging disable
```

---

## Delete Commands

Delete commands remove BGP configuration. Use `delete bgp` followed by the target.

### Delete Global Settings

```
delete bgp asn
delete bgp router-id
delete bgp prefixes <cidr>
delete bgp local-communities <community>
delete bgp origin-communities <community>
delete bgp prefix-communities <cidr> [community]
delete bgp authorized-providers <prefix> [asn]
delete bgp rtr-server
delete bgp irr-server
delete bgp irr-query-timeout
delete bgp bgpq-args
delete bgp peeringdb-api-key
delete bgp peeringdb-url
delete bgp peeringdb-query-timeout
delete bgp peeringdb-cache
delete bgp source4
delete bgp source6
delete bgp confederation
delete bgp raw-config
delete bgp accept-default
delete bgp default-route
delete bgp merge-paths
delete bgp keep-filtered
delete bgp rpki-enable
delete bgp blackhole-bogon-asns
delete bgp no-announce
delete bgp no-accept
delete bgp stun
delete bgp blocklist <value>
delete bgp blocklist-urls <url>
delete bgp blocklist-files <path>
delete bgp bogon-asns <asn>
delete bgp bogons4 <cidr>
delete bgp bogons6 <cidr>
delete bgp transit-asns <asn>
```

### Delete Peer

```
delete bgp peer <name>                          # Delete entire peer
delete bgp peer <name> neighbor <ip>            # Remove specific neighbor
delete bgp peer <name> <option>                 # Remove specific peer option
delete bgp peer <name> session <option>         # Remove session option
delete bgp peer <name> route <option>           # Remove route option
delete bgp peer <name> filter <option>          # Remove filter option
delete bgp peer <name> community <option>       # Remove community option
delete bgp peer <name> limit <option>           # Remove limit option
delete bgp peer <name> security <option>        # Remove security option
delete bgp peer <name> role <option>            # Remove role option
delete bgp peer <name> capability <option>      # Remove capability option
delete bgp peer <name> hook <option>            # Remove hook option
```

### Delete Policy (Template)

```
delete bgp policy <name>                        # Delete entire policy
delete bgp policy <name> <option>               # Remove specific option (same groups as peer)
```

### Delete System Settings

```
delete bgp system kernel <option>
delete bgp system optimizer <option>
```

---

## Show Commands

### Show BGP Summary

```
show bgp
show bgp summary
```

Displays a table of all BGP peers with their status, ASN, state, uptime, and prefixes received.

### Show BGP Peer

```
show bgp peer <name>
show bgp peer <name> summary
show bgp peer <name> advertised-routes
show bgp peer <name> received-routes
show bgp peer <name> rejected-routes
```

!!! note "Peer Name Format in BIRD"
    BIRD removes dashes from peer names and converts to uppercase. For example, `ibgp-cyber` becomes `IBGPCYBER` in BIRD protocol names.

### Show BGP Neighbor (by IP)

```
show bgp neighbor <ip-address>
show bgp neighbor <ip-address> summary
```

### Show BGP Logging

```
show bgp logging
show bgp logging last <N>
```

### Show Routes

```
show route                                       # Show all routes
show route for <destination>                     # Show route for specific destination
show route protocol <name>                       # Show routes from specific protocol
show route table <name>                          # Show routes in specific table
show route filter <name>                         # Show routes matching filter
show route where <condition>                     # Show routes matching condition
show route export <protocol|table>               # Show exported routes
show route import <protocol|table>               # Show imported routes
show route preexport <protocol>                  # Show routes before export filter
show route noexport <protocol>                   # Show routes rejected by export
show route origin-as <asn>                       # Show routes originated by AS
show route transit-as <asn>                      # Show routes transiting AS
show route community <community>                 # Show routes matching community
show route rpki <valid|invalid|unknown>          # Show routes by RPKI status
show route in <protocol>                         # Show routes in import table
show route stats                                 # Show route statistics
```

---

## Clear Commands

```
clear bgp peer <name> soft                      # Soft reset (refresh routes)
clear bgp peer <name> hard                      # Hard reset (restart session, traffic impact)
```

---

## Applying Configuration

After making changes, apply the configuration:

```
commit
```

Example output:

```
floofos(config)# commit
Validating configuration...
Configuration committed successfully
```

---

## Direct YAML Configuration

For advanced users or bulk configuration, FloofOS also supports directly editing the Pathvector YAML configuration file:

```
edit bgp raw
```

This opens `/etc/pathvector.yml` in an interactive editor with the following controls:

- Mouse navigation
- Copy/paste (`Ctrl+C`, `Ctrl+V`)
- Save and exit (`Ctrl+Q`)

After saving, run `commit` to apply the changes. The CLI commands and the YAML file are kept in sync -- changes made via `edit bgp raw` will be reflected in the CLI state and vice versa.

### YAML Configuration Structure

```yaml
asn: 65000
router-id: 192.0.2.1

prefixes:
  - 192.0.2.0/24
  - 2001:db8::/32

local-communities:
  - 65000:1:0

default-route: false
accept-default: false
merge-paths: true

bgpq-args: "-S AFRINIC,APNIC,ARIN,LACNIC,RIPE"
irr-server: rr.ntt.net
irr-query-timeout: 30

rtr-server: 172.65.0.2:8282

#peeringdb-api-key: "your-api-key-here"

kernel:
  learn: true

templates:
  upstream:
    local-pref: 100
    add-on-import:
      - 65000:1:1
    announce:
      - 65000:1:0
      - 65000:1:4
    filter-rpki: true
    filter-bogon-routes: true
    filter-bogon-asns: true
    import-limit4: 1500000
    import-limit6: 300000
    allow-local-as: true
    remove-all-communities: 65000

  routeserver:
    local-pref: 200
    add-on-import:
      - 65000:1:2
    announce:
      - 65000:1:0
      - 65000:1:4
    filter-rpki: true
    filter-transit-asns: true
    filter-bogon-routes: true
    filter-bogon-asns: true
    auto-import-limits: true
    enforce-first-as: false
    enforce-peer-nexthop: false
    remove-all-communities: 65000

  peer:
    local-pref: 300
    add-on-import:
      - 65000:1:3
    announce:
      - 65000:1:0
      - 65000:1:4
    filter-rpki: true
    filter-irr: true
    filter-transit-asns: true
    filter-bogon-routes: true
    filter-bogon-asns: true
    auto-as-set: true
    auto-import-limits: true
    remove-all-communities: 65000

  downstream:
    local-pref: 400
    add-on-import:
      - 65000:1:4
    announce:
      - 65000:1:0
      - 65000:1:1
      - 65000:1:2
      - 65000:1:3
    filter-rpki: true
    filter-irr: true
    filter-transit-asns: true
    filter-bogon-routes: true
    filter-bogon-asns: true
    auto-as-set: true
    auto-import-limits: true
    allow-blackhole-community: true
    remove-all-communities: 65000

  ibgp:
    local-pref: 150
    add-on-import:
      - 65000:1:5
    announce:
      - 65000:1:0
      - 65000:1:1
      - 65000:1:2
      - 65000:1:3
      - 65000:1:4
      - 65000:1:5
    next-hop-self: true
    direct: true
    allow-local-as: true
    enforce-first-as: false
    enforce-peer-nexthop: false
    filter-irr: false
    filter-rpki: false
    remove-all-communities: 65000

peers:
  Cogent:
    template: upstream
    asn: 174
    neighbors:
      - 154.54.x.x
    password: "secret"

  DE-CIX_RS:
    template: routeserver
    asn: 6695
    neighbors:
      - 80.81.192.157
      - 80.81.192.158

  Cloudflare:
    template: peer
    asn: 13335
    neighbors:
      - 192.0.2.1

  Customer_ABC:
    template: downstream
    asn: 64512
    as-set: AS-CUSTOMER
    neighbors:
      - 192.0.2.10

  Router2:
    template: ibgp
    asn: 65000
    listen4: 10.255.0.1
    neighbors:
      - 10.255.0.2
    multihop: true
```

For the full list of available YAML parameters, see the [Pathvector Configuration Reference](https://pathvector.io/docs/configuration).

---

## Static Default Route

For management connectivity and traceroute ASN lookups, configure a static default route via BIRD:

**CLI method:**

```
set bgp raw-config "protocol static default_route { ipv4 { preference 80; }; ipv6 { preference 80; }; route 0.0.0.0/0 via YOUR_GATEWAY_IPV4; route ::/0 via YOUR_GATEWAY_IPV6; }"
```

**YAML method (via `edit bgp raw`):**

```yaml
global-config: |
  protocol static default_route {
    ipv4 { preference 80; };
    ipv6 { preference 80; };
    route 0.0.0.0/0 via YOUR_GATEWAY_IPV4;
    route ::/0 via YOUR_GATEWAY_IPV6;
  }
```

!!! note "Preference Value"
    The preference of 80 is lower than BGP (100+), ensuring BGP-learned routes take precedence.

---

## BGP Large Communities

FloofOS templates use BGP Large Communities for route tagging:

| Community | Meaning |
|-----------|---------|
| `YOUR_ASN:1:0` | Originated locally (your prefixes) |
| `YOUR_ASN:1:1` | Learned from upstream provider |
| `YOUR_ASN:1:2` | Learned from route server |
| `YOUR_ASN:1:3` | Learned from bilateral peer |
| `YOUR_ASN:1:4` | Learned from downstream/customer |
| `YOUR_ASN:1:5` | Learned from iBGP |

Replace `YOUR_ASN` with your actual ASN (e.g., `65000:1:0`).

---

## Quick Start Example

Complete example to set up BGP with an upstream provider:

```
# 1. Set global BGP identity
set bgp asn 65000
set bgp router-id 192.0.2.1
set bgp prefixes 192.0.2.0/24
set bgp prefixes 2001:db8::/32
set bgp local-communities 65000:1:0

# 2. Set global options
set bgp merge-paths
set bgp rtr-server 172.65.0.2:8282
set bgp irr-server rr.ntt.net
set bgp bgpq-args "-S AFRINIC,APNIC,ARIN,LACNIC,RIPE"
set bgp system kernel learn

# 3. Create an upstream policy
set bgp policy upstream route local-pref 100
set bgp policy upstream community add-on-import 65000:1:1
set bgp policy upstream community announce 65000:1:0
set bgp policy upstream community announce 65000:1:4
set bgp policy upstream filter filter-rpki
set bgp policy upstream filter filter-bogon-routes
set bgp policy upstream filter filter-bogon-asns
set bgp policy upstream limit import-limit4 1500000
set bgp policy upstream limit import-limit6 300000
set bgp policy upstream route allow-local-as
set bgp policy upstream community remove-all-communities 65000

# 4. Add a peer using the policy
set bgp peer Cogent policy upstream
set bgp peer Cogent remote-as 174
set bgp peer Cogent neighbor 154.54.0.1

# 5. Enable BGP logging
set bgp logging enable

# 6. Apply configuration
commit
```

---

## Command Reference Summary

| Command | Description |
|---------|-------------|
| `set bgp <option> <value>` | Set BGP global configuration |
| `set bgp policy <name> ...` | Configure BGP policy (template) |
| `set bgp peer <name> ...` | Configure BGP peer |
| `set bgp system kernel ...` | Configure kernel settings |
| `set bgp system optimizer ...` | Configure optimizer settings |
| `set bgp logging <enable\|disable>` | Enable/disable BGP logging |
| `delete bgp <option>` | Delete BGP configuration |
| `delete bgp peer <name>` | Delete BGP peer |
| `delete bgp policy <name>` | Delete BGP policy |
| `show bgp` | Show BGP peer summary |
| `show bgp peer <name>` | Show specific BGP peer |
| `show bgp neighbor <ip>` | Show BGP neighbor by IP |
| `show bgp logging` | Show BGP routing logs |
| `show route [options]` | Show routing table |
| `clear bgp peer <name> soft` | Soft reset BGP peer |
| `clear bgp peer <name> hard` | Hard reset BGP peer |
| `edit bgp raw` | Edit Pathvector YAML directly |
| `commit` | Apply BGP configuration changes |

---

## Additional Resources

- [Pathvector Documentation](https://pathvector.io/docs/configuration) - Complete Pathvector reference
- [BIRD2 Documentation](https://bird.network.cz/?get_doc) - BIRD routing daemon documentation
