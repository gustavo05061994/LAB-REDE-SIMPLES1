# Projeto CCNA 200-301 — Rede Corporativa Acumulativa no Cisco Packet Tracer

Projeto de estudo para a certificação CCNA 200-301, desenvolvido com metodologia acumulativa — cada lab incrementa o anterior, resultando em uma rede corporativa completa construída do zero.

---

## Sobre o projeto

A proposta foi diferente dos labs tradicionais: em vez de configurações isoladas, uma única topologia evoluiu ao longo de 13 labs, adicionando complexidade a cada etapa. Isso simula como redes reais são construídas e mantidas em ambiente corporativo.

Ao final do projeto, a rede conta com dois sites interligados, quatro VLANs, roteamento dinâmico e estático, serviços de rede, segurança em camada 2 e 3 e hardening completo dos dispositivos.

---

## Topologia final

```
Site A                                    Site B
                                          
PC1 (192.168.10.11) ─┐                ┌─ PC5 (192.168.30.12)
PC2 (192.168.10.12) ─┤                ├─ PC6 (192.168.30.11)
PC3 (192.168.20.11) ─┤                ├─ PC7 (192.168.40.12)
PC4 (192.168.20.12) ─┤                ├─ PC8 (192.168.40.11)
                      │                │
                     S1 ══════════════ S2   (EtherChannel LACP Fa0/6-7)
                      │    \      /    │
                      │     \    /     │
                      │      S3        │
                      │   (STP bloqueado em Fa0/2)
                      │                │
                     R1 ────────────── R2
                  Gi0/0.10  10.0.0.0/30  Gi0/1.30
                  Gi0/0.20               Gi0/1.40
                  Gi0/1 (10.0.0.1)   Gi0/0 (10.0.0.2)
```

### Equipamentos
- 2x Roteador Cisco 1941
- 3x Switch Cisco 2960-24TT
- 8x PC

### Endereçamento IP

| Dispositivo | Interface | IP | VLAN |
|---|---|---|---|
| R1 | Gi0/0.10 | 192.168.10.1/24 | 10 (TI) |
| R1 | Gi0/0.20 | 192.168.20.1/24 | 20 (RH) |
| R1 | Gi0/1 | 10.0.0.1/30 | — |
| R2 | Gi0/0 | 10.0.0.2/30 | — |
| R2 | Gi0/1.30 | 192.168.30.1/24 | 30 (TI-B) |
| R2 | Gi0/1.40 | 192.168.40.1/24 | 40 (RH-B) |
| PC1 | Fa0 | 192.168.10.11 (DHCP) | 10 |
| PC2 | Fa0 | 192.168.10.12 (DHCP) | 10 |
| PC3 | Fa0 | 192.168.20.11 (DHCP) | 20 |
| PC4 | Fa0 | 192.168.20.12 (DHCP) | 20 |
| PC5 | Fa0 | 192.168.30.12 (DHCP) | 30 |
| PC6 | Fa0 | 192.168.30.11 (DHCP) | 30 |
| PC7 | Fa0 | 192.168.40.12 (DHCP) | 40 |
| PC8 | Fa0 | 192.168.40.11 (DHCP) | 40 |

---

## Labs

### Lab 1 — Configuração básica
Configuração inicial dos roteadores: hostname, interfaces Gigabit com endereçamento IP, ativação das interfaces e verificação de conectividade básica entre PCs.

**Comandos principais:** `hostname`, `interface gi0/0`, `ip address`, `no shutdown`, `show ip interface brief`

---

### Lab 2 — VLANs + Router-on-a-Stick
Criação das VLANs 10 (TI) e 20 (RH) no S1, configuração de portas de acesso e trunk, e implementação do Router-on-a-Stick no R1 com subinterfaces e encapsulamento dot1q.

**Comandos principais:** `vlan 10`, `switchport mode trunk`, `interface gi0/0.10`, `encapsulation dot1q 10`, `show vlan brief`, `show interfaces trunk`

---

### Lab 3 — OSPF multi-site
Expansão da topologia para o Site B com S2, VLANs 30 e 40, e roteador R2. Configuração do link WAN 10.0.0.0/30 e OSPF em area 0 nos dois roteadores para roteamento dinâmico entre os sites.

**Comandos principais:** `router ospf 1`, `network x.x.x.x wildcard area 0`, `show ip ospf neighbor`, `show ip route`

---

### Lab 4 — STP + EtherChannel LACP
Adição do S3 formando um triângulo entre os switches. Configuração do EtherChannel LACP (modo active) entre S1 e S2 nas portas Fa0/6-7. O STP elegeu o S1 como root bridge e bloqueou a porta Fa0/2 do S3 para evitar loop.

**Comandos principais:** `interface range fa0/6-7`, `channel-group 1 mode active`, `show etherchannel summary`, `show spanning-tree`

---

### Lab 5 — CDP + LLDP
Verificação de vizinhança com CDP e LLDP em todos os dispositivos. Troubleshooting real: o S1 aparecia nos vizinhos do R1 de forma assimétrica — identificado como limitação do Packet Tracer com interfaces trunk sem IP direto.

**Comandos principais:** `cdp run`, `lldp run`, `show cdp neighbors`, `show cdp neighbors detail`, `show lldp neighbors detail`

---

### Lab 6 — Rotas estáticas e default route
Remoção do OSPF e configuração manual de rotas estáticas nos dois roteadores. Adição de default route no R1 simulando saída para internet. Troubleshooting completo: PC1 não pingava o Site B por estar na VLAN errada no S1 — diagnosticado com `show vlan brief` e `show mac address-table`.

**Comandos principais:** `ip route x.x.x.x mask next-hop`, `ip route 0.0.0.0 0.0.0.0 next-hop`, `show ip route`, `show mac address-table`

---

### Lab 7 — HSRP (teórico)
Estudo do Hot Standby Router Protocol: IP virtual, MAC virtual, eleição por prioridade, preempt e timers. Não implementado na prática por limitação física da topologia — R2 não possui conexão direta com S1.

**Comandos principais:** `standby 1 ip`, `standby 1 priority`, `standby 1 preempt`, `show standby brief`

---

### Lab 8 — DHCP server
Configuração do R1 como servidor DHCP para VLANs 10 e 20, e R2 para VLANs 30 e 40. Endereços 1 a 10 excluídos de cada pool. Troubleshooting: PC2 não recebia IP — S3 não tinha VLANs configuradas nem trunk para o S1.

**Comandos principais:** `ip dhcp excluded-address`, `ip dhcp pool`, `network`, `default-router`, `dns-server`, `show ip dhcp binding`, `show ip dhcp pool`

---

### Lab 9 — NAT + PAT
Configuração de PAT no R1 traduzindo as redes 192.168.10.0/24 e 192.168.20.0/24 para o IP público 10.0.0.1. Validação em tempo real com `show ip nat translations` mostrando o mapeamento de IPs e portas.

**Comandos principais:** `ip access-list standard`, `ip nat inside source list overload`, `ip nat inside`, `ip nat outside`, `show ip nat translations`, `show ip nat statistics`

---

### Lab 10 — NTP + SSH
R1 configurado como servidor NTP stratum 1. R2 sincronizou com sucesso (stratum 2, referência 10.0.0.1). SSH versão 2 habilitado nos dois roteadores com chave RSA 1024 bits, usuário local com privilege 15 e linhas VTY restritas a SSH.

**Comandos principais:** `ntp master 1`, `ntp server`, `show ntp status`, `ip domain-name`, `crypto key generate rsa`, `username admin privilege 15 secret`, `transport input ssh`, `ip ssh version 2`, `show ip ssh`

---

### Lab 11 — ACLs padrão e estendidas
Cenário 1: ACL padrão bloqueando PC1 (192.168.10.11) de acessar o Site B, aplicada no R2 perto do destino. Cenário 2: ACL estendida bloqueando Telnet (porta 23) da VLAN 10 para VLAN 30, aplicada no R1 perto da origem. Validação via contadores de match.

**Comandos principais:** `ip access-list standard`, `ip access-list extended`, `deny`, `permit`, `ip access-group in/out`, `show access-lists`

---

### Lab 12 — Port Security
Configuração de Port Security com sticky MAC nas portas de acesso do S1 (Fa0/1 e Fa0/4), máximo de 1 MAC por porta e modo de violação shutdown. Simulação real de violação com PC teste — switch registrou o MAC invasor, colocou a porta em err-disabled e gerou log de violação. Recuperação com shutdown/no shutdown.

**Comandos principais:** `switchport port-security`, `switchport port-security maximum 1`, `switchport port-security mac-address sticky`, `switchport port-security violation shutdown`, `show port-security`, `show port-security address`

---

### Lab 13 — Hardening e senhas
Aplicação de boas práticas de segurança em todos os dispositivos: senha privilegiada criptografada com MD5, criptografia de todas as senhas em texto claro, banner de aviso legal, timeout de sessão de 5 minutos no console e VTY, e desabilitação de CDP na interface WAN externa.

**Comandos principais:** `enable secret`, `service password-encryption`, `security passwords min-length 8`, `banner motd`, `exec-timeout 5 0`, `no cdp enable`, `show running-config | include secret`

---

## Troubleshooting real documentado

Um dos diferenciais do projeto foi o troubleshooting real que aconteceu em cada etapa:

**Lab 5** — CDP assimétrico entre R1 e S1. O S1 enxergava o R1 pelas três interfaces (Gi0/0, Gi0/0.10, Gi0/0.20), mas o R1 não listava o S1. Causa: limitação do Packet Tracer com interface trunk sem IP direto na interface física.

**Lab 6** — PC1 não pingava o Site B mesmo com rotas corretas nos roteadores. Diagnóstico camada por camada: ping estendido do R1 com source 192.168.10.1 funcionou, provando que os roteadores estavam ok. `show vlan brief` revelou que a porta Fa0/1 do S1 estava na VLAN 1 padrão. Correção: `switchport access vlan 10`.

**Lab 8** — PC2 não recebia IP via DHCP. Causa: S3 não tinha VLANs configuradas nem trunk habilitado para o S1. Correção: criação das VLANs, configuração da porta de acesso e trunk no S3.

**Lab 12** — Port Security com sticky não registrava violação imediatamente. O switch aguardou o tráfego do PC teste para detectar o MAC invasor e disparar o err-disabled automaticamente.

---

## Comandos por categoria

### Interfaces e conectividade
```
interface gi0/0              — acessa interface
ip address x.x.x.x mask     — atribui IP
no shutdown                  — ativa interface
show ip interface brief      — resumo de interfaces
```

### VLANs e switching
```
vlan 10 / name TI            — cria VLAN
switchport mode access       — porta de acesso
switchport access vlan 10    — atribui porta à VLAN
switchport mode trunk        — porta trunk
show vlan brief              — lista VLANs e portas
show interfaces trunk        — detalhes das trunks
```

### Roteamento
```
router ospf 1                — habilita OSPF
network x.x.x.x wild area 0 — anuncia rede no OSPF
ip route x.x.x.x mask gw    — rota estática
ip route 0.0.0.0 0.0.0.0 gw — default route
show ip route                — tabela de roteamento
show ip ospf neighbor        — vizinhos OSPF
```

### Serviços
```
ip dhcp pool NOME            — cria pool DHCP
ip nat inside source list overload  — configura PAT
ntp master 1                 — servidor NTP
ip ssh version 2             — SSH versão 2
```

### Segurança
```
ip access-list extended NOME — ACL estendida
switchport port-security     — habilita port security
enable secret SENHA          — senha privilegiada MD5
service password-encryption  — criptografa senhas
banner motd #TEXTO#          — banner de aviso
```

---

## Sobre o autor

Supervisor de TIC na Minsait, atuando na Petrobras REFAP. Tecnólogo em Segurança da Informação pela Uniasselvi. Certificações: ITIL V4, HDI, NSE1/NSE2/NSE3, CCST Networking.

Preparação para CCNA 200-301 (julho/2026) e NSE4 Fortinet (dezembro/2026). Objetivo: especialização em firewall e arquitetura de redes.

---

*Estudando com o professor Gustavo Kalau no YouTube.*
