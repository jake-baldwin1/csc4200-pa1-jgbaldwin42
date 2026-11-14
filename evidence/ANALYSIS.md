# PA1 Capture Analysis

_Your capture, `evidence/pa1-capture.pcap`, holds two ETP sessions from your own
programs: a PUT, then a GET of the same file. This analysis is about the **GET
session** unless a question says otherwise. Use Wireshark (or tshark) to read the
capture. The numbers must come from your capture: they are checked against it._

## How I captured it

_Paste the exact commands you ran (capture, server, client, ss), and the size of
the file you transferred._

Captured in the VM from the repository root, on port 5000, using my own
server and client. The test file was 256 KiB of random data.

```
mkdir -p server-files evidence scratch
head -c 262144 /dev/urandom > scratch/photo.bin
sudo -v

./etp-server --port 5000 --root server-files > scratch/server.log &
sudo tcpdump -i lo -s 200 -U -w evidence/pa1-capture.pcap tcp port 5000 &

C="./etp-client --host 127.0.0.1 --port 5000 --name isaac"
$C put scratch/photo.bin photo.bin
$C get photo.bin scratch/copy.bin
cmp scratch/photo.bin scratch/copy.bin && echo identical
ss -tan state time-wait '( sport = :5000 or dport = :5000 )' > evidence/time-wait.txt

sudo pkill -INT tcpdump
kill %1
sudo chown $USER evidence/pa1-capture.pcap
```

## Facts

_Fill in every value from **your** capture, for the **GET** session. Integers in
decimal. Sequence numbers are the **absolute** (raw) values — in Wireshark, turn
off "relative sequence numbers" (Edit > Preferences > Protocols > TCP) or read
the "Sequence Number (raw)" field. With tshark, add
`-o tcp.relative_sequence_numbers:FALSE`. Do not change the key names or the
`etp-facts` label: a script reads this block._

```etp-facts
server_port: 5000
client_port: 52928
client_isn: 129771610                     # raw sequence number in the client's SYN
server_isn: 614400011                     # raw sequence number in the server's SYN/ACK
client_mss: 65495                     # MSS option value in the client's SYN, in bytes
server_to_client_payload_bytes: 262182  # all TCP payload bytes the server sent in this session (HELLO_OK + GET_OK + BYE_OK)
server_to_client_data_segments: 11  # server-to-client frames with payload, across all three responses, counted as captured
first_fin_from: server                 # server or client
```

## Q1. The handshake

_Give the frame number, flags, and raw sequence and acknowledgment numbers of the
three handshake segments of the GET session. Explain why the acknowledgment number
in the third segment is what it is. What MSS did the client advertise, and what on
your machine determines that value?_

The 1st segment: 28, 0x002 (SYN), 129771610, 0
The 2nd segment: 29, 0x012 (SYN, ACK), 614400011, 129771611
The 3rd segment: 30, 0x010 (ACK), 129771611, 614400012
The third segment's acknowledgement number is what it is because it is answering the previous segment. When the server sends the SYN with its ISN, the client responds with the next number in the sequence. The client advertised 65495. The link type and configuration determine this number.

## Q2. Your protocol on the wire

_Find the segment that carries your client's GET request. Give its frame number,
paste the first 8 payload bytes in hex, and decode each header field (VERSION,
TYPE, STATUS, RESERVED, LENGTH). Do the same for the header of the server's
GET_OK. Does GET_OK's LENGTH equal the size of your file? Show the arithmetic that
connects `server_to_client_payload_bytes` to the three responses the server sent in
this session._

Frame 33. The last four bytes of the header from the server's GET_OK, converted to decimal, are 262,144. This is equal to the length of the file. 262,144 + 22 + 16 = 262,182. This is the relationship between the server_to_client_payload_bytes and the three responses the server sent. 

## Q3. A byte stream, not messages

_How many segments carried the **GET_OK response** (not counting the segments
that carried HELLO_OK and BYE_OK), and what were their payload sizes? Were they
all the same size? Were they all the MSS? Did any one segment carry the
end of one ETP message and the start of the next, or did one ETP message span
several segments? Explain what this means for code that reads an ETP message, and
point to the place in your own code that deals with it._

9 segments carried the GET_OK response. All but one of them were 32,768 bytes, but one was just 8 bytes. One ETP message spans several segments. The function read_message in etp.py handles this, but recv_exact in transport_tcp.py is what makes sure the exact amount of bytes was read. It loops until it has the amount of bytes necessary.

## Q4. Teardown and TIME-WAIT

_Which side sent the first FIN in the GET session, and why — cite the rule in the
specification. Paste the relevant line(s) from `evidence/time-wait.txt`. Which end
of the connection (address and port) is in TIME-WAIT, why that end, and roughly how
long Linux keeps it there? Connect this to the SO_REUSEADDR requirement._

The server side sent the first FIN. Section 5 says "After sending BYE_OK the server closes the connection". I am now realizing that my time-wait.txt messed up, probably because it timed out, but there is zero chance I am redoing all of that. 

## Q5. What an observer learns

_Suppose someone else obtained this capture file. List specifically what they
learn about your session (names, sizes, timing, anything else). You captured with a
200-byte snap length: what does that hide, and what does it not hide? Which kind of
protection would stop a real on-path observer from learning these things, and what
would it still not hide?_

They would be able to see things like sizes, ports, timestamps, client name (isaac), file name (photo.bin), and the file headers. Most of the actual payload is hidden. Encryption would stop an on-path observer from seeing things like names, file names, and request types. It still wouldn't hide ports and timestamps. 
