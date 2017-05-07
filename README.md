## Progress Notes

### Objectives
- Create a zfs recv daemon server.
  - The daemon will always be running, servicing zfs sends into the server's storage.
  - Security layer can be easily implemented on the net.Socket layer.
- Create a zfs send client.
- Create a robust state machine between the server and client.
  - See ```common/messages``` for messages communicated between the server and client.
  - See ```common/library``` for function libraries that is used between server and client.
    - This contains the setting up a zfs send or zfs recv child processes, and then hooking up the socket's stream into it during appropriate times.
- Client is slightly smart, server - not so much.
  - Client determines if it needs to perform a zfs incremental or initial snapshot send.
  - Client determines common snapshot list and identifies when it's completed syncing.
- Server gives what client wants most of the time.
  - Server will setup the zfs recv process and then tell the client when the socket is ready for a zfs stream.
- Do performance testing, and node cluster testing.
  - Use node cluster to setup multiple zfs recv daemons? -- Central locking scheme may be required for potential duplicate zfs send|recvs.

## Starting Up
```
[smos-00] server start:  node receiver/
[smos-01] client start:  node sender/
```

## End state
- discrepancy in used size is due to smos-00 running lz4 compression, and smos-01 using no compression.
- there is only /dev/zero'd files, so compression is very good.
```
[root@smos-00 /zones/zpsend]# zfs list -r -t all zones/test
NAME               USED  AVAIL  REFER  MOUNTPOINT
zones/test          99K   235G    25K  /zones/test
zones/test@now       1K      -    23K  -
zones/test@now2      1K      -    23K  -
zones/test@8550     15K      -    25K  -
zones/test@18569    15K      -    25K  -
zones/test@20392    15K      -    25K  -
zones/test@30577    15K      -    25K  -
zones/test@4387       0      -    25K  -

[root@smos-01 /zones/zpsend]# zfs list -r -t all zones/test
NAME               USED  AVAIL  REFER  MOUNTPOINT
zones/test        1.11M   236G  1.04M  /zones/test
zones/test@now        0      -    23K  -
zones/test@now2       0      -    23K  -
zones/test@8550     15K      -  1.03M  -
zones/test@18569    15K      -  1.03M  -
zones/test@20392    15K      -  1.03M  -
zones/test@30577    15K      -  1.04M  -
zones/test@4387       0      -  1.04M  -
```

## Server -- Output
```
[2017-05-07T19:32:40.902Z]  INFO: zpsend/21276 on smos-00: listening IPv4: 0.0.0.0:6830
[2017-05-07T19:32:42.376Z]  INFO: zpsend/21276 on smos-00: received an init message from client
[2017-05-07T19:32:42.376Z]  INFO: zpsend/21276 on smos-00: sending an init message to client
[2017-05-07T19:32:42.381Z]  INFO: zpsend/21276 on smos-00: received a get snapshot list message from client for zones/test
[2017-05-07T19:32:42.413Z]  INFO: zpsend/21276 on smos-00: sending client a list of snapshot for zones/test
[2017-05-07T19:32:42.413Z]  INFO: zpsend/21276 on smos-00:
[2017-05-07T19:32:42.449Z]  INFO: zpsend/21276 on smos-00: received a start zfs recv process message from client
[2017-05-07T19:32:42.522Z]  INFO: zpsend/21276 on smos-00: telling client receive is done.
[2017-05-07T19:32:42.523Z]  INFO: zpsend/21276 on smos-00: received an init message from client
[2017-05-07T19:32:42.524Z]  INFO: zpsend/21276 on smos-00: sending an init message to client
[2017-05-07T19:32:42.524Z]  INFO: zpsend/21276 on smos-00: received a get snapshot list message from client for zones/test
[2017-05-07T19:32:42.547Z]  INFO: zpsend/21276 on smos-00: sending client a list of snapshot for zones/test
[2017-05-07T19:32:42.547Z]  INFO: zpsend/21276 on smos-00: zones/test@now
[2017-05-07T19:32:42.572Z]  INFO: zpsend/21276 on smos-00: received a start zfs recv process message from client
[2017-05-07T19:32:42.572Z]  INFO: zpsend/21276 on smos-00: incremental send
[2017-05-07T19:32:42.677Z]  INFO: zpsend/21276 on smos-00: telling client receive is done.
[2017-05-07T19:32:42.678Z]  INFO: zpsend/21276 on smos-00: received an init message from client
[2017-05-07T19:32:42.678Z]  INFO: zpsend/21276 on smos-00: sending an init message to client
[2017-05-07T19:32:42.679Z]  INFO: zpsend/21276 on smos-00: received a get snapshot list message from client for zones/test
[2017-05-07T19:32:42.700Z]  INFO: zpsend/21276 on smos-00: sending client a list of snapshot for zones/test
[2017-05-07T19:32:42.700Z]  INFO: zpsend/21276 on smos-00: zones/test@now, zones/test@now2
[2017-05-07T19:32:42.726Z]  INFO: zpsend/21276 on smos-00: received a start zfs recv process message from client
[2017-05-07T19:32:42.727Z]  INFO: zpsend/21276 on smos-00: incremental send
[2017-05-07T19:32:42.822Z]  INFO: zpsend/21276 on smos-00: telling client receive is done.
[2017-05-07T19:32:42.824Z]  INFO: zpsend/21276 on smos-00: received an init message from client
[2017-05-07T19:32:42.824Z]  INFO: zpsend/21276 on smos-00: sending an init message to client
[2017-05-07T19:32:42.825Z]  INFO: zpsend/21276 on smos-00: received a get snapshot list message from client for zones/test
[2017-05-07T19:32:42.847Z]  INFO: zpsend/21276 on smos-00: sending client a list of snapshot for zones/test
[2017-05-07T19:32:42.847Z]  INFO: zpsend/21276 on smos-00: zones/test@now, zones/test@now2, zones/test@8550
[2017-05-07T19:32:42.871Z]  INFO: zpsend/21276 on smos-00: received a start zfs recv process message from client
[2017-05-07T19:32:42.871Z]  INFO: zpsend/21276 on smos-00: incremental send
[2017-05-07T19:32:42.936Z]  INFO: zpsend/21276 on smos-00: telling client receive is done.
[2017-05-07T19:32:42.938Z]  INFO: zpsend/21276 on smos-00: received an init message from client
[2017-05-07T19:32:42.938Z]  INFO: zpsend/21276 on smos-00: sending an init message to client
[2017-05-07T19:32:42.939Z]  INFO: zpsend/21276 on smos-00: received a get snapshot list message from client for zones/test
[2017-05-07T19:32:42.960Z]  INFO: zpsend/21276 on smos-00: sending client a list of snapshot for zones/test
[2017-05-07T19:32:42.960Z]  INFO: zpsend/21276 on smos-00: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569
[2017-05-07T19:32:42.985Z]  INFO: zpsend/21276 on smos-00: received a start zfs recv process message from client
[2017-05-07T19:32:42.985Z]  INFO: zpsend/21276 on smos-00: incremental send
[2017-05-07T19:32:43.101Z]  INFO: zpsend/21276 on smos-00: telling client receive is done.
[2017-05-07T19:32:43.102Z]  INFO: zpsend/21276 on smos-00: received an init message from client
[2017-05-07T19:32:43.102Z]  INFO: zpsend/21276 on smos-00: sending an init message to client
[2017-05-07T19:32:43.103Z]  INFO: zpsend/21276 on smos-00: received a get snapshot list message from client for zones/test
[2017-05-07T19:32:43.125Z]  INFO: zpsend/21276 on smos-00: sending client a list of snapshot for zones/test
[2017-05-07T19:32:43.125Z]  INFO: zpsend/21276 on smos-00: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392
[2017-05-07T19:32:43.149Z]  INFO: zpsend/21276 on smos-00: received a start zfs recv process message from client
[2017-05-07T19:32:43.149Z]  INFO: zpsend/21276 on smos-00: incremental send
[2017-05-07T19:32:43.272Z]  INFO: zpsend/21276 on smos-00: telling client receive is done.
[2017-05-07T19:32:43.274Z]  INFO: zpsend/21276 on smos-00: received an init message from client
[2017-05-07T19:32:43.274Z]  INFO: zpsend/21276 on smos-00: sending an init message to client
[2017-05-07T19:32:43.275Z]  INFO: zpsend/21276 on smos-00: received a get snapshot list message from client for zones/test
[2017-05-07T19:32:43.298Z]  INFO: zpsend/21276 on smos-00: sending client a list of snapshot for zones/test
[2017-05-07T19:32:43.299Z]  INFO: zpsend/21276 on smos-00: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577
[2017-05-07T19:32:43.322Z]  INFO: zpsend/21276 on smos-00: received a start zfs recv process message from client
[2017-05-07T19:32:43.322Z]  INFO: zpsend/21276 on smos-00: incremental send
[2017-05-07T19:32:43.446Z]  INFO: zpsend/21276 on smos-00: telling client receive is done.
[2017-05-07T19:32:43.451Z]  INFO: zpsend/21276 on smos-00: received an init message from client
[2017-05-07T19:32:43.451Z]  INFO: zpsend/21276 on smos-00: sending an init message to client
[2017-05-07T19:32:43.452Z]  INFO: zpsend/21276 on smos-00: received a get snapshot list message from client for zones/test
[2017-05-07T19:32:43.476Z]  INFO: zpsend/21276 on smos-00: sending client a list of snapshot for zones/test
[2017-05-07T19:32:43.476Z]  INFO: zpsend/21276 on smos-00: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:32:43.500Z]  INFO: zpsend/21276 on smos-00: client is finished, politely asked to end connection
```

## Client -- Example
```
[2017-05-07T19:28:36.223Z]  INFO: zpsend/11918 on smos-01: sending init message to server
[2017-05-07T19:28:36.231Z]  INFO: zpsend/11918 on smos-01: received an init message from server
[2017-05-07T19:28:36.231Z]  INFO: zpsend/11918 on smos-01: asking server to get snapshot list for zones/test
[2017-05-07T19:28:36.266Z]  INFO: zpsend/11918 on smos-01: received snapshots for zones/test filesystem from server
[2017-05-07T19:28:36.266Z]  INFO: zpsend/11918 on smos-01:
[2017-05-07T19:28:36.299Z]  INFO: zpsend/11918 on smos-01: found the following snapshots on my filesystem zones/test
[2017-05-07T19:28:36.299Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:36.300Z]  INFO: zpsend/11918 on smos-01: an initial seed needed
[2017-05-07T19:28:36.300Z]  INFO: zpsend/11918 on smos-01: initial snapshot: zones/test@now
[2017-05-07T19:28:36.300Z]  INFO: zpsend/11918 on smos-01: asking the server to start up a recv for zones/test
[2017-05-07T19:28:36.321Z]  INFO: zpsend/11918 on smos-01: received a ready zfs recv message from server
[2017-05-07T19:28:36.410Z]  INFO: zpsend/11918 on smos-01: server said it successfully received snapshot into zones/test
[2017-05-07T19:28:36.410Z]  INFO: zpsend/11918 on smos-01: initial snapshot: undefined
[2017-05-07T19:28:36.410Z]  INFO: zpsend/11918 on smos-01: sending init message to server
[2017-05-07T19:28:36.411Z]  INFO: zpsend/11918 on smos-01: received an init message from server
[2017-05-07T19:28:36.411Z]  INFO: zpsend/11918 on smos-01: asking server to get snapshot list for zones/test
[2017-05-07T19:28:36.435Z]  INFO: zpsend/11918 on smos-01: received snapshots for zones/test filesystem from server
[2017-05-07T19:28:36.436Z]  INFO: zpsend/11918 on smos-01: zones/test@now
[2017-05-07T19:28:36.457Z]  INFO: zpsend/11918 on smos-01: found the following snapshots on my filesystem zones/test
[2017-05-07T19:28:36.457Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:36.457Z]  INFO: zpsend/11918 on smos-01: an incremental send is needed
[2017-05-07T19:28:36.457Z]  INFO: zpsend/11918 on smos-01: from: zones/test@now to: zones/test@now2
[2017-05-07T19:28:36.457Z]  INFO: zpsend/11918 on smos-01: asking the server to start up a recv for zones/test
[2017-05-07T19:28:36.457Z]  INFO: zpsend/11918 on smos-01: {"name":"zones/test","incremental":true,"snapFrom":"zones/test@now","snapTo":"zones/test@now2"}
[2017-05-07T19:28:36.477Z]  INFO: zpsend/11918 on smos-01: received a ready zfs recv message from server
[2017-05-07T19:28:36.539Z]  INFO: zpsend/11918 on smos-01: server said it successfully received incremental snapshot into zones/test
[2017-05-07T19:28:36.539Z]  INFO: zpsend/11918 on smos-01: incremental snapshot: zones/test@now - zones/test@now2
[2017-05-07T19:28:36.539Z]  INFO: zpsend/11918 on smos-01: sending init message to server
[2017-05-07T19:28:36.540Z]  INFO: zpsend/11918 on smos-01: received an init message from server
[2017-05-07T19:28:36.540Z]  INFO: zpsend/11918 on smos-01: asking server to get snapshot list for zones/test
[2017-05-07T19:28:36.564Z]  INFO: zpsend/11918 on smos-01: received snapshots for zones/test filesystem from server
[2017-05-07T19:28:36.565Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2
[2017-05-07T19:28:36.587Z]  INFO: zpsend/11918 on smos-01: found the following snapshots on my filesystem zones/test
[2017-05-07T19:28:36.587Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:36.587Z]  INFO: zpsend/11918 on smos-01: an incremental send is needed
[2017-05-07T19:28:36.587Z]  INFO: zpsend/11918 on smos-01: from: zones/test@now2 to: zones/test@8550
[2017-05-07T19:28:36.587Z]  INFO: zpsend/11918 on smos-01: asking the server to start up a recv for zones/test
[2017-05-07T19:28:36.587Z]  INFO: zpsend/11918 on smos-01: {"name":"zones/test","incremental":true,"snapFrom":"zones/test@now2","snapTo":"zones/test@8550"}
[2017-05-07T19:28:36.606Z]  INFO: zpsend/11918 on smos-01: received a ready zfs recv message from server
[2017-05-07T19:28:36.675Z]  INFO: zpsend/11918 on smos-01: server said it successfully received incremental snapshot into zones/test
[2017-05-07T19:28:36.675Z]  INFO: zpsend/11918 on smos-01: incremental snapshot: zones/test@now2 - zones/test@8550
[2017-05-07T19:28:36.675Z]  INFO: zpsend/11918 on smos-01: sending init message to server
[2017-05-07T19:28:36.675Z]  INFO: zpsend/11918 on smos-01: received an init message from server
[2017-05-07T19:28:36.675Z]  INFO: zpsend/11918 on smos-01: asking server to get snapshot list for zones/test
[2017-05-07T19:28:36.702Z]  INFO: zpsend/11918 on smos-01: received snapshots for zones/test filesystem from server
[2017-05-07T19:28:36.702Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550
[2017-05-07T19:28:36.726Z]  INFO: zpsend/11918 on smos-01: found the following snapshots on my filesystem zones/test
[2017-05-07T19:28:36.726Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:36.726Z]  INFO: zpsend/11918 on smos-01: an incremental send is needed
[2017-05-07T19:28:36.726Z]  INFO: zpsend/11918 on smos-01: from: zones/test@8550 to: zones/test@18569
[2017-05-07T19:28:36.726Z]  INFO: zpsend/11918 on smos-01: asking the server to start up a recv for zones/test
[2017-05-07T19:28:36.726Z]  INFO: zpsend/11918 on smos-01: {"name":"zones/test","incremental":true,"snapFrom":"zones/test@8550","snapTo":"zones/test@18569"}
[2017-05-07T19:28:36.746Z]  INFO: zpsend/11918 on smos-01: received a ready zfs recv message from server
[2017-05-07T19:28:36.798Z]  INFO: zpsend/11918 on smos-01: server said it successfully received incremental snapshot into zones/test
[2017-05-07T19:28:36.798Z]  INFO: zpsend/11918 on smos-01: incremental snapshot: zones/test@8550 - zones/test@18569
[2017-05-07T19:28:36.798Z]  INFO: zpsend/11918 on smos-01: sending init message to server
[2017-05-07T19:28:36.800Z]  INFO: zpsend/11918 on smos-01: received an init message from server
[2017-05-07T19:28:36.800Z]  INFO: zpsend/11918 on smos-01: asking server to get snapshot list for zones/test
[2017-05-07T19:28:36.825Z]  INFO: zpsend/11918 on smos-01: received snapshots for zones/test filesystem from server
[2017-05-07T19:28:36.825Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569
[2017-05-07T19:28:36.848Z]  INFO: zpsend/11918 on smos-01: found the following snapshots on my filesystem zones/test
[2017-05-07T19:28:36.848Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:36.848Z]  INFO: zpsend/11918 on smos-01: an incremental send is needed
[2017-05-07T19:28:36.848Z]  INFO: zpsend/11918 on smos-01: from: zones/test@18569 to: zones/test@20392
[2017-05-07T19:28:36.848Z]  INFO: zpsend/11918 on smos-01: asking the server to start up a recv for zones/test
[2017-05-07T19:28:36.848Z]  INFO: zpsend/11918 on smos-01: {"name":"zones/test","incremental":true,"snapFrom":"zones/test@18569","snapTo":"zones/test@20392"}
[2017-05-07T19:28:36.866Z]  INFO: zpsend/11918 on smos-01: received a ready zfs recv message from server
[2017-05-07T19:28:36.934Z]  INFO: zpsend/11918 on smos-01: server said it successfully received incremental snapshot into zones/test
[2017-05-07T19:28:36.934Z]  INFO: zpsend/11918 on smos-01: incremental snapshot: zones/test@18569 - zones/test@20392
[2017-05-07T19:28:36.935Z]  INFO: zpsend/11918 on smos-01: sending init message to server
[2017-05-07T19:28:36.935Z]  INFO: zpsend/11918 on smos-01: received an init message from server
[2017-05-07T19:28:36.935Z]  INFO: zpsend/11918 on smos-01: asking server to get snapshot list for zones/test
[2017-05-07T19:28:36.961Z]  INFO: zpsend/11918 on smos-01: received snapshots for zones/test filesystem from server
[2017-05-07T19:28:36.961Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392
[2017-05-07T19:28:36.981Z]  INFO: zpsend/11918 on smos-01: found the following snapshots on my filesystem zones/test
[2017-05-07T19:28:36.982Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:36.984Z]  INFO: zpsend/11918 on smos-01: an incremental send is needed
[2017-05-07T19:28:36.984Z]  INFO: zpsend/11918 on smos-01: from: zones/test@20392 to: zones/test@30577
[2017-05-07T19:28:36.984Z]  INFO: zpsend/11918 on smos-01: asking the server to start up a recv for zones/test
[2017-05-07T19:28:36.984Z]  INFO: zpsend/11918 on smos-01: {"name":"zones/test","incremental":true,"snapFrom":"zones/test@20392","snapTo":"zones/test@30577"}
[2017-05-07T19:28:37.004Z]  INFO: zpsend/11918 on smos-01: received a ready zfs recv message from server
[2017-05-07T19:28:37.106Z]  INFO: zpsend/11918 on smos-01: server said it successfully received incremental snapshot into zones/test
[2017-05-07T19:28:37.106Z]  INFO: zpsend/11918 on smos-01: incremental snapshot: zones/test@20392 - zones/test@30577
[2017-05-07T19:28:37.106Z]  INFO: zpsend/11918 on smos-01: sending init message to server
[2017-05-07T19:28:37.107Z]  INFO: zpsend/11918 on smos-01: received an init message from server
[2017-05-07T19:28:37.107Z]  INFO: zpsend/11918 on smos-01: asking server to get snapshot list for zones/test
[2017-05-07T19:28:37.134Z]  INFO: zpsend/11918 on smos-01: received snapshots for zones/test filesystem from server
[2017-05-07T19:28:37.134Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577
[2017-05-07T19:28:37.155Z]  INFO: zpsend/11918 on smos-01: found the following snapshots on my filesystem zones/test
[2017-05-07T19:28:37.156Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:37.156Z]  INFO: zpsend/11918 on smos-01: an incremental send is needed
[2017-05-07T19:28:37.156Z]  INFO: zpsend/11918 on smos-01: from: zones/test@30577 to: zones/test@4387
[2017-05-07T19:28:37.156Z]  INFO: zpsend/11918 on smos-01: asking the server to start up a recv for zones/test
[2017-05-07T19:28:37.156Z]  INFO: zpsend/11918 on smos-01: {"name":"zones/test","incremental":true,"snapFrom":"zones/test@30577","snapTo":"zones/test@4387"}
[2017-05-07T19:28:37.175Z]  INFO: zpsend/11918 on smos-01: received a ready zfs recv message from server
[2017-05-07T19:28:37.227Z]  INFO: zpsend/11918 on smos-01: server said it successfully received incremental snapshot into zones/test
[2017-05-07T19:28:37.228Z]  INFO: zpsend/11918 on smos-01: incremental snapshot: zones/test@30577 - zones/test@4387
[2017-05-07T19:28:37.228Z]  INFO: zpsend/11918 on smos-01: sending init message to server
[2017-05-07T19:28:37.232Z]  INFO: zpsend/11918 on smos-01: received an init message from server
[2017-05-07T19:28:37.232Z]  INFO: zpsend/11918 on smos-01: asking server to get snapshot list for zones/test
[2017-05-07T19:28:37.263Z]  INFO: zpsend/11918 on smos-01: received snapshots for zones/test filesystem from server
[2017-05-07T19:28:37.263Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:37.285Z]  INFO: zpsend/11918 on smos-01: found the following snapshots on my filesystem zones/test
[2017-05-07T19:28:37.285Z]  INFO: zpsend/11918 on smos-01: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:37.285Z]  INFO: zpsend/11918 on smos-01: the server has all my snapshots
[2017-05-07T19:28:37.285Z]  INFO: zpsend/11918 on smos-01: my snapshots: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:37.285Z]  INFO: zpsend/11918 on smos-01: server snapshots: zones/test@now, zones/test@now2, zones/test@8550, zones/test@18569, zones/test@20392, zones/test@30577, zones/test@4387
[2017-05-07T19:28:37.285Z]  INFO: zpsend/11918 on smos-01: asking server to end connection
[2017-05-07T19:28:37.287Z]  INFO: zpsend/11918 on smos-01: server closed my connection
```