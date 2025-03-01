# Trouble Shooting

### 1. `ERROR: [SCTP] Failed to connect given AMF    N3IWF=NGAP`

This error occured when N3IWF was started before AMF finishing initialization. This error usually appears when you run the TestNon3GPP in the first time.

Rerun the test should be fine. If it still not be solved, larger the sleeping time in line 110 of `test.sh`.

### 2. TestNon3GPP

TestNon3GPP will modify the `config/amfcfg.conf`. So, if you had killed the TestNon3GPP test before it finished, you might need to copy `config/amfcfg.conf.bak` back to `config/amfcfg.conf` to let other tests pass.

`cp config/amfcfg.conf.bak config/amfcfg.conf`

### 3. DB on TLS to H2C

If you meet any problems about https or mogodb, it maybe couse our new version from v3.0.1 to v3.0.2 has change http to H2C verion. Try the command below.

`mongo --eval "db.NfProfile.drop()" free5gc`

### 4. `MQCreate() Error creating message queue: Too many open files UPF=Util` (UPF)

Remove POSIX message queues

```bash
ls /dev/mqueue/
rm /dev/mqueue/*
```

### 5. Remove gtp devices (using tools in libgtp5gnl) (UPF)
```bash
cd lib/libgtp5gnl/tools
sudo ./gtp5g-link del {Dev-Name}
```

### 6. `UPF Cli Run Error: open Gtp5g: open link: create: file exists`
```bash
sudo ip link del upfgtp
```

### 7. Decode HTTP/2 packet in Wireshark
    
1. Run Network Function

    Check has XXFsslkey.log

2. Edit >> Preference >> Protocols >> SSL (TLS)
    
    ![](https://i.imgur.com/dzzMib5.png)

3. Add keylog

    ![](https://i.imgur.com/gV7x5QU.png)

4. Filter http2

    ![](https://i.imgur.com/ctBIYQy.png)

### 7. Decode H2C (HTTP2 clear text without TLS)

The similar reason as NEA0 NAS message. Althrough H2C is clear text, wirshark still considers these packets as the normal TCP packets and does not decode them by HTTP2.

To see the details of H2C packets, do the following configuration.

1. Analyze → Decode As…

    ![](https://i.imgur.com/NJ2brow.png)

2. click Add button to add the decode rules

    ![](https://i.imgur.com/Ct4KLgO.png)

    Decode the packets from the TCP ports listened by each NF as HTTP2 packets.