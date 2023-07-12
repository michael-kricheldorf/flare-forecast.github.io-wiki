Gateway   | Location            | Evio IP      | Nebula IP    | eno1 IP (static) | enp1s0 (edge of device) | tnc0 IP     | Reverse SSH Port 
---       | ---                 | ---          | ---          | ---              | ---                     | ---         | ---
JS-front  | Jetstream 2         | 10.10.100.1  | ---          | ---              | ---                     | ---         | ---
Diana     | FCRE Weir           | ---          | ---          | 10.10.1.2        | dhcp                    | 10.10.101.2 | ---
Carina    | Vahid's Fitlet      | ---          | 10.10.200.2  | 10.10.1.2        | dhcp                    | ---         | ---
Bjorn     | BVRE Platform       | 10.10.100.5  | 10.10.200.5  | 10.10.1.2        | dhcp                    | ---         | 60005
Annie     | CCRE Dam            | 10.10.100.6  | 10.10.200.6  | 10.10.1.2        | dhcp                    | ---         | 60006
Norvel    | FCRE Metstation     | 10.10.100.7  | 10.10.200.7  | 10.10.1.2        | dhcp                    | 10.10.101.1 | 60007
Henrietta | FCRE Catwalk        | 10.10.100.11 | 10.10.200.11 | 10.10.1.2        | dhcp                    | 10.10.101.3 | 60008
VD mgmt   | Vahid's laptop      | 10.10.100.9  | 10.10.200.9  | ---              | ---                     | ---         | ---
RF mgmt   | Renato's laptop     | 10.10.100.10 | 10.10.200.10 | ---              | ---                     | ---         | ---


Notes:
* The subnet mask for all the IP addresses is `255.255.255.0`.
* Evio overlay name is `CIBR6`.
* `eno1` is the interface closest to the power supply in the fitlet, and is used for data logger and field management via ssh from a laptop. Only one (laptop or data logger) can be connected at a time unless an Ethernet switch is used in the field.
* `enp1s0` is the outermost ethernet interface in the fitlet and is used for data logger connection.
* `tnc0` is the interface used for LoRa radio.
* Data logger IP address is `10.10.1.1`.
* Field laptop IP address must be set to `10.10.1.3`.

Old stuff - to be deleted:

Gateway   | Location            | Evio IP      | Nebula IP    | eno1 IP (static) | enp1s0 (edge of device) | tnc0 IP     | Reverse SSH Port 
---       | ---                 | ---          | ---          | ---              | ---                     | ---         | ---
Carina    | FCRE Metstation     | 10.10.100.2  | 10.10.200.2  | 10.10.1.2        | dhcp                    | ---         | 60002
Mia       | FCRE Catwalk        | 10.10.100.3  | 10.10.200.3  | 10.10.1.2        | dhcp                    | ---         | 60003
Dianna    | Vahid's Fitlet      | 10.10.100.11 | 10.10.200.11 | ---              | ---                     | ---         | 60011
