Gateway | Location | Evio IP | Nebula IP | eno1 IP | enp2s0 IP | Laptop IP | tnc0 IP | 
| --- | --- | --- | --- | --- | --- | --- | --- |
JS-front | Jetstream 2 | 10.10.100.1 | --- | --- | --- | --- | --- |
Carina | FCRE Metstation | 10.10.100.2 | 10.10.200.2 | --- | 10.10.1.2 | --- | --- |
Mia | FCRE Catwalk | 10.10.100.3 | 10.10.200.3 | --- | 10.10.1.2 | --- | --- |
Bita | FCRE Weir | --- | --- | 172.16.100.1 | 10.10.1.2 | 172.16.100.2 | 10.10.101.2 |
Bjorn | BVRE Platform | 10.10.100.5 | 10.10.200.5 | --- | 10.10.1.2 | --- | --- |
Annie | CCRE Dam | 10.10.100.6 | 10.10.200.6 | --- | 10.10.1.2 | --- | --- |
Norvel | FCRE Metstation new | 10.10.100.7 | 10.10.200.7 | 172.16.100.1 | 10.10.1.2 | 172.16.100.2 | 10.10.101.1 |
Henrietta | FCRE Catwalk new | 10.10.100.8 | 10.10.200.8 | 172.16.100.1 | 10.10.1.2 | 172.16.100.2 | 10.10.101.3 |
VD mgmt | Vahid's laptop | 10.10.100.9 | 10.10.200.9 | --- | --- | --- | --- |
RF mgmt | Renato's laptop | 10.10.100.10 | 10.10.200.10 | --- | --- | --- | --- |

Notes:
* The subnet mask for all the IP addresses is `255.255.255.0`.
* Evio overlay name is `CIBR6`.
* `eno1` is the interface closest to the power supply in the fitlet, and for field management via ssh from a laptop.
* `enp2s0` is the outermost ethernet interface in the fitlet and is used for data logger connection.
* `tnc0` is the interface used for LoRa radio.
* Data logger IP address is `10.10.1.1`.