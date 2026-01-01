# rox-sniffer
[![Ask DeepWiki](https://devin.ai/assets/askdeepwiki.png)](https://deepwiki.com/met7s/rox-sniffer)

`rox-sniffer` is a simple, command-line network packet sniffer written in Python. It captures network traffic from your machine's interface and displays detailed header information for Ethernet, IPv4, TCP, UDP, and ICMP packets in real-time.

## Features

-   Captures live network traffic at the data link layer (Layer 2).
-   Decodes and displays headers for:
    -   **Ethernet:** Source/Destination MAC addresses.
    -   **IPv4:** Source/Destination IP addresses, TTL, protocol, and more.
    -   **TCP:** Source/Destination ports, sequence/acknowledgment numbers, and flags (SYN, ACK, FIN, etc.).
    -   **UDP:** Source/Destination ports and length.
    -   **ICMP:** Type, code, and checksum.
-   Displays the raw data payload for recognized protocols.
-   Built with Python's standard libraries, requiring no external dependencies.

## Requirements

-   Python 3
-   Root privileges (required to create raw sockets).

## Usage

To run the sniffer, you must use `sudo` because capturing raw packets requires elevated permissions.

1.  Clone the repository:
    ```sh
    git clone https://github.com/met7s/rox-sniffer.git
    cd rox-sniffer
    ```

2.  Run the script with root privileges:
    ```sh
    sudo python3 rox_sniffer.py
    ```

3.  The tool will start capturing and displaying packets immediately. To stop the capture, press `Ctrl+C`.

## Example Output

```
[*] rox-sniffer started... Waiting for packets (Press Ctrl+C to stop)

------------------------------------------------------------
Ethernet Frame:
	 - Destination: 01:00:5E:00:00:FB, Source: E8:6B:D1:XX:XX:XX, Protocol: 8
	 - IPv4 Packet:
		 - Version: 4, Header Length: 20, TTL: 1
		 - Protocol: 17, Source: 192.168.1.1, Target: 224.0.0.251
		 - UDP Segment:
			 - Source Port: 5353, Destination Port: 5353, Length: 298

------------------------------------------------------------
Ethernet Frame:
	 - Destination: A0:B3:CC:XX:XX:XX, Source: B4:DE:31:XX:XX:XX, Protocol: 8
	 - IPv4 Packet:
		 - Version: 4, Header Length: 20, TTL: 64
		 - Protocol: 6, Source: 192.168.1.100, Target: 172.217.16.10
		 - TCP Segment:
			 - Source Port: 51012, Destination Port: 443
			 - Sequence: 3372863013, Acknowledgment: 1813735817
			 - Flags:
				 - URG: 0, ACK: 1, PSH: 1, RST: 0, SYN: 0, FIN: 0
		 - Data:
			 	 \x17\x03\x03\x01\x18\x00\x00\x00...

------------------------------------------------------------
Ethernet Frame:
	 - Destination: C0:56:27:XX:XX:XX, Source: A0:B3:CC:XX:XX:XX, Protocol: 8
	 - IPv4 Packet:
		 - Version: 4, Header Length: 20, TTL: 64
		 - Protocol: 1, Source: 192.168.1.100, Target: 8.8.8.8
		 - ICMP Packet:
			 - Type: 8, Code: 0, Checksum: 27956
		 - Data:
			 	 \x00\x01\xde\x8e\x00\x01\x02\x03\x04...
```

## How It Works

The script operates by creating a raw network socket that listens to all traffic on a public network interface.

1.  **Socket Creation:** `socket.socket(socket.AF_PACKET, socket.SOCK_RAW, socket.ntohs(0x0003))` creates a raw socket at Layer 2. `socket.AF_PACKET` is specific to Linux for low-level packet access. `socket.ntohs(0x0003)` ensures that all protocols (ETH_P_ALL) are captured.
2.  **Packet Capture Loop:** The script enters an infinite loop, using `conn.recvfrom(65535)` to wait for and receive raw data packets as they arrive.
3.  **Unpacking Data:** For each received packet, the script uses Python's `struct` module to unpack the binary data according to the standardized header formats of Ethernet, IPv4, and its encapsulated protocols (TCP, UDP, ICMP).
4.  **Displaying Information:** The parsed header fields and payload data are printed to the console in a structured and readable format.
