# TCP Connectivity Troubleshooting & Packet Analysis | Wireshark

Diagnosed TCP connectivity failures using Wireshark, netstat, and Test-NetConnection; analyzed SYN, RST/ACK, and TCP three-way handshake behavior to identify a non-listening service and verify connectivity after remediation.

## Objectives

The objective of this project is to troubleshoot a TCP service connectivity problem using Windows networking tools and Wireshark. The project examines TCP packets during a failed connection, identifies the cause of the failure, applies a solution, and verifies that TCP connectivity is restored.

## Lab Environment

The lab was performed on a Windows computer using Wireshark, Command Prompt, and PowerShell. TCP port 9999 was selected as a temporary test port. The tests were performed through the local loopback interface so that the client and test service could operate on the same computer.

## Checking if any service is listening on port 9999

On cmd:

```cmd
netstat -ano | findstr :9999
```

The finding: No TCP application/service is listening on port 9999 which is to be expected.

## Trying tcp connection on local host port:9999

```powershell
powershell Test-NetConnection localhost -Port 9999
```

Result:

```text
Client-> localhost
SYN

Localhost-> Client
RST, ACK
```

The TCP connection was rejected/reset and was not established. The packet capture showed that the client first transmitted a TCP SYN packet to destination port 9999. Instead of receiving the expected SYN-ACK, the client received a RST-ACK response. Packet was received but ack was rejected.

## Fixing the issue

Creating temporary TCP service on port 9999

I typed following command one PowerShell

```powershell
$listener = [System.Net.Sockets.TcpListener]::new([System.Net.IPAddress]::Loopback,9999)
$listener.Start()
$client = $listener.AcceptTcpClient()
```

This started a service that listens on port 9999

Command Provided:

```cmd
netstat -ano | findstr :9999
```

The command prompt shows listening state on port 9999

## Verification

Testing the connection again on port 9999

Command:

```powershell
powershell Test-NetConnection localhost -Port 9999
```

```text
TCPTestSucceeded: True
```

ON wireshark, we can see

At first, it failed, but after sometime, it displayed syn, syn-ack, ack message so the troubleshooting and setting up of tcp listening on port 9999 was successful.

## Conclusion

This project demonstrated TCP connectivity troubleshooting using Wireshark and Windows networking tools. Wireshark identified the failed connection through SYN and RST-ACK packets, while netstat confirmed that no service was listening on port 9999. After starting the TCP service, a successful three-way handshake confirmed that connectivity was restored.
