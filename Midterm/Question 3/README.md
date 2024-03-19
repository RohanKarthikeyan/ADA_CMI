In this README, we will be outlining the steps to convert the PCAP file into a CSV file.

The authors of the UNSW-NB15 dataset did the following:
* Collected PCAP files;
* Split them into smaller files each of size 1000 MB; and
* Used two tools: Argus and Bro-IDS for feature extraction.

**NOTE:** [This SO question](https://stackoverflow.com/questions/8092380/export-pcap-data-to-csv-timestamp-bytes-uplink-downlink-extra-info) covers an example of both tools in action.

We'll be following the same procedure. Here are the step-wise commands for the same:
> Splitting the PCAP:

```
tcpdump -r Task_3.pcap -w <new_file_name.pcap> -C 1000
```
This splits our original PCAP into smaller PCAPs of size just under 1GB.

> Using the tools:

* Argus:
```
# Convert to flow based format
argus -r Task_3.pcap -w Three.argus

# Create features: refer NUSW CSV file for the names of the fields
ra -r Three.argus -n -u -s <field1, field2, ...> > <new_file_name>.txt
E.g., ra -r <File>.argus -n -u -s stime, saddr, daddr, sport, dport, proto, state, dur, sbytes, dbytes, sttl, dttl, sloss, dloss, sload, dload, spkts, dpkts, smeansz, dmeansz, sjit, djit, sintpkt, dintpkt, synack, ackdat

# Use pandas.read_table with sep='\s+', header=0, and custom header names.
```
See the [man page](https://manpages.ubuntu.com/manpages/lunar/en/man1/ra.1.html) for more details.

* Bro-IDS (now called Zeek): To install Zeek, see here: https://github.com/BlueWhale87/myfiles#readme
```
# Step 1: Go to folder where PCAPs are present

# Step 2: Create folder to hold Zeek logs and go there
mkdir <folder_name>
cd <folder_name>/

# Step 3: Run the Zeek command to generate logs
/opt/zeek/bin/zeek -C -r ../Yours.pcap

# Step 4: Extract required fields using zeek-cut as a TSV file
/opt/zeek/bin/zeek-cut <field 1 field 2 ...> < conn.log > Your_file.tsv
E.g., /opt/zeek/bin/zeek-cut ts id.orig_h id.orig_p id.resp_h id.resp_p proto service < conn.log
```
See the [documentation](https://docs.zeek.org/en/master/log-formats.html) for more details.

We're using Zeek primarily because Argus isn't giving one service (http, etc.) Rather, it's converting the port numbers used in each flow to the appropriate service, both on the source and destination side. In contrast, Zeek gives only one service value.

* This would necessitate us to do a comparison of the files obtained from both tools and see how much they are in concordance with one another.
* A preliminary observation on the example PCAP files shows perfect concordance in the record starting times of both tools. Hallelujah!
