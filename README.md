# Dokumentacja Architektury
[cite_start]Autor: S34754 Paweł Koc [cite: 50]

![Schemat Architektury](PCO-Project2.jpg)

## [cite_start]1. Sieć [cite: 51]
[cite_start]**Główny komponent:** [cite: 52]
* [cite_start]Region: us-east-1 (N-Virginia) [cite: 53]
* [cite_start]VPC Name: VPC-01 [cite: 54]
* [cite_start]VPC CIDR: 10.0.0.0 /16 [cite: 55]
* [cite_start]Internet Gateway: VPC-01-IGW (Attached to VPC-01) [cite: 56]

[cite_start]**Podsieci (Subnets):** [cite: 57]

| Nazwa Podsieci | Typ | Availability Zone | Adresacja |
|---|---|---|---|
| VPC-01-Public-A | Publiczna | us-east-1a (use1-az2) | 10.0.1.0/24 |
| VPC-01-Public-B | Publiczna | us-east-1b (use1-az4) | 10.0.2.0/24 |
| VPC-01-Private-A | Prywatna | us-east-1a (use1-az2) | 10.0.10.0/24 |
| VPC-01-Private-B | Prywatna | us-east-1b (use1-az4) | [cite_start]10.0.11.0/24 | [cite: 58]

## [cite_start]2. Tabele Routingu [cite: 59]
* [cite_start]**VPC-01-PublicRT** (Przypisana do: VPC-01-Public-A, VPC-01-Public-B) [cite: 60]
  * [cite_start]10.0.0.0/16 -> local [cite: 61]
  * [cite_start]0.0.0.0/0 -> igw-0f187f0623f3d4a48 <VPC-01-IGW> [cite: 62]
* [cite_start]**VPC-01-PrivateRT-A** (Przypisana do: VPC-01-Private-A) [cite: 63]
  * [cite_start]10.0.0.0/16 -> local [cite: 64]
  * [cite_start]0.0.0.0/0 -> eni-0585655fe732a9738 <VPC-01-NAT-A> [cite: 65]
* [cite_start]**VPC-01-PrivateRT-B** (Przypisana do: VPC-01-Private-B) [cite: 66]
  * [cite_start]10.0.0.0/16 -> local [cite: 67]
  * [cite_start]0.0.0.0/0 -> eni-09c9f188d13f400c3 <VPC-01-NAT-B> [cite: 68]

## [cite_start]3. Instancje [cite: 69]
[cite_start]**Instancje Standalone:** [cite: 70]
* [cite_start]**VPC-01-NAT-A:** [cite: 71]
  * [cite_start]Lokalizacja: VPC-01-Public-A [cite: 72]
  * [cite_start]Wymagania: Wyłączona opcja Source/Destination check, przypisany Elastic IP. [cite: 73]
* [cite_start]**VPC-01-NAT-B:** [cite: 74]
  * [cite_start]Lokalizacja: VPC-01-Public-B [cite: 75]
  * [cite_start]Wymagania: Wyłączona opcja Source/Destination check, przypisany Elastic IP. [cite: 76]
* [cite_start]**VPC-01-Bastion-A:** [cite: 77]
  * [cite_start]Lokalizacja: VPC-01-Private-A [cite: 78]
* [cite_start]**VPC-01-Bastion-B:** [cite: 79]
  * [cite_start]Lokalizacja: VPC-01-Private-B [cite: 80]

[cite_start]**Auto Scaling Group (ASG):** [cite: 81]
* [cite_start]ASG Name: VPC-01-appWP-ASG [cite: 82]
* [cite_start]VPC Subnets: VPC-01-Private-A, VPC-01-Private-B [cite: 83]
* [cite_start]Zarządzane instancje: VPC-01-Wordpress-A, VPC-01-Wordpress-B [cite: 84]

## [cite_start]4. Równoważenie Obciążenia [cite: 85]
* [cite_start]**VPC-01-NetLB (Network Load Balancer):** [cite: 86]
  * [cite_start]Scheme: Internet-facing [cite: 87]
  * [cite_start]Listener: TCP:22 (SSH) [cite: 88]
  * [cite_start]Target Group: Typ Instance [cite: 89][cite_start], przypisane targety: VPC-01-Bastion-A, VPC-01-Bastion-B [cite: 90]
* [cite_start]**VPC-01-APPLB (Application Load Balancer):** [cite: 91]
  * [cite_start]Scheme: Internet-facing [cite: 92]
  * [cite_start]Listener 1: HTTP:80 (Akcja: Przekierowanie na HTTPS:443) [cite: 93]
  * [cite_start]Listener 2: HTTPS:443 (Akcja: Forward do grupy docelowej, wymóg certyfikatu SSL/ACM) [cite: 94]
  * [cite_start]Target Group: Typ Instance, protokół HTTP:80, zintegrowana z VPC-01-appWP-ASG [cite: 95]

## [cite_start]5. Baza Danych [cite: 96]
[cite_start]**Amazon Aurora Cluster:** [cite: 97]
* [cite_start]Typ wdrożenia: Multi-AZ (Primary + Replica) [cite: 98]
* [cite_start]DB Subnet Group: VPC-01-Private-A oraz VPC-01-Private-B [cite: 99]
* [cite_start]Instancja Primary: Strefa us-east-1a [cite: 100]
* [cite_start]Instancja Replica (Async): Strefa us-east-1b [cite: 101]

## [cite_start]6. Bezpieczeństwo (security groups) [cite: 102]
* [cite_start]**VPC-01-NetLB-SG** [cite: 103]
  * [cite_start]Cel: obsługa ruchu administracyjnego [cite: 104]
  * [cite_start]Przypisanie: VPC-01-NetLB [cite: 105]
  * [cite_start]Inbound: Port 22 (TCP) [cite: 106]
* [cite_start]**VPC-01-BastionSG** [cite: 107]
  * [cite_start]Cel: Zabezpieczenie serwerów Bastion. [cite: 108]
  * [cite_start]Przypisanie: VPC-01-Bastion-A, VPC-01-Bastion-B [cite: 109]
  * [cite_start]Inbound: Port 22 (TCP) tylko dla VPC-01-NetLB-SG [cite: 110]
* [cite_start]**NATSG-A oraz VPC-01-NATSG-B** [cite: 111]
  * [cite_start]Cel: Obsługa ruchu wyjściowego z podsieci prywatnych w strefach A i B. [cite: 112]
  * [cite_start]Przypisanie: VPC-01-NAT-A, VPC-01-NAT-B [cite: 113]
  * [cite_start]Inbound: (All trafic) [cite: 114]
* [cite_start]**VPC-01-ALBSG** [cite: 115]
  * [cite_start]Cel: Przyjmowanie publicznego ruchu webowego. [cite: 116]
  * [cite_start]Przypisanie: VPC-01-APPLB [cite: 117]
  * [cite_start]Inbound: Port 80 (HTTP) oraz Port 443 (HTTPS) ze źródła 0.0.0.0/0 [cite: 118]
* [cite_start]**VPC-01-AppSG** [cite: 119]
  * [cite_start]Cel: Zabezpieczenie maszyn aplikacyjnych w Auto Scaling Group. [cite: 120]
  * [cite_start]Przypisanie: VPC-01-appWP-ASG [cite: 121]
  * [cite_start]Inbound: Port 80 (HTTP) – wyłącznie ze VPC-01-ALBSG[cite: 123]. [cite_start]Port 22 (SSH) ze źródła VPC-01-BastionSG[cite: 124].
* [cite_start]**VPC-01-DBSG** [cite: 125]
  * [cite_start]Cel: Ochrona warstwy danych. [cite: 126]
  * [cite_start]Przypisanie: Klastry i instancje Amazon Aurora. [cite: 127]
  * [cite_start]Inbound: Port 3306 (MySQL/Aurora) – wyłącznie ze VPC-01-AppSG [cite: 128]
