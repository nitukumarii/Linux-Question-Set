If an application is not reachable, the typical troubleshooting flow is:

```text
1. ip addr        → Is the interface up? Does it have an IP?
2. ip route       → Is the default gateway correct?
3. ping Gateway   → Can I reach the gateway?
4. ping Target    → Is the destination reachable?
5. nslookup/dig   → Is DNS resolving?
6. ss -tulnp      → Is the application listening on the expected port?
7. curl -I        → Is the application responding?
8. traceroute     → Where is the network path failing?
9. tcpdump        → Are packets reaching/leaving the server?
10. journalctl    → Check service/network logs.
