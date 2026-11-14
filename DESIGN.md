# PA1 Design — ETP/1 Client and Server

_One to two pages. Short, specific answers beat long general ones. Every section
should be true of the code you submit. Delete these italic prompts when you write._

## 1. Architecture

client.py -> etp.py, transport.py
server.py -> transport.py
transport.py -> transport_tcp.py

etp.py has the header struct, constants, and the encode/read_message functions.
client.py runs connect, hello, bye
transport_tcp has the socket calls exported through transport.py
server.py has the accept loop

## 2. The transport seam

The transport file is transport_tcp.py. The functions are listen, accept, connect, send_all, recv_exact, and close
When the PA2 transport is swapped in, we would have to adjust the makefile but as long as it has the same functions everything else should be fine.

## 3. Framing

read_message calls recv_exact to read the byte stream. 
recv loops until it has the entire message. 
A header that arrives in pieces would just be reassembled.
When two messages arrive, the extra bytes stay in the buffer until they are consumed.
read_message checks version and length before reading the payload to prevent blocking.

## 4. Concurrency

If one client stalls half way through a header it should only block its own thread because each connection should only have one thread.
Put writes to a temporary file so doing a get at the same time should not affect anything negatively.

## 5. Errors and disconnects

Fatal: the error is sent and the session is ended.
Non-fatal: session continues in the same state because the payload was already consumed.
If a client dsconnects during a large get, only that session should be closed.
The client only writes if the whole session succeeds so there should be no partial output.

## 6. Threat model

Traffic is unencrypted and unauthenticated, anyone on the network can read and modify files, any client could overwrite any file.
The server resists path traversal because names and length are checked.
The server does not resist resource exhaustion, a put could hold a large amount of memory and there is no max on the number of connections. 

## 7. Testing

I had claude check our code before we ran it.
It found a missing '()' after a bytearray function in recv_excact that would have caused it to not run correctly.
