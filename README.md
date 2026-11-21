
> Nota ética: feito em rede própria. Reconhecer uma rede só deve ser feito em
> redes que nos pertencem ou para as quais temos autorização.

## Porquê mapear a rede

Só se consegue notar o que é **estranho** quando se conhece o que é **normal**.
Saber que aparelhos existem na rede, com que IP e de que fabricante, é a base
para, mais tarde, detetar um intruso — um aparelho que não reconhecemos é logo
um sinal de alerta.

## Passo 1 — Descobrir quem está vivo na rede (Nmap)

```bash
sudo nmap -sn 192.168.1.0/24
```

- `-sn` → *só descoberta*: verifica quem responde, **sem** investigar portas. É
  rápido e leve.
- `192.168.1.0/24` → **toda a rede**, dos endereços `.1` ao `.254`.

### Resultado — aparelhos encontrados

| Aparelho | IP | Fabricante | O que é |
|---|---|---|---|
| Vodafone GW | 192.168.1.1 | Vodafone | **Router** (porteiro da rede) |
| Redmi Note 13 Pro 5G | 192.168.1.113 | Xiaomi | Telemóvel |
| master bedroom TV | 192.168.1.170 | Google | **Smart TV** (Chromecast) |
| Josias Domingos | 192.168.1.172 | Intel Corporate | Portátil **Surface** |

> A TV apareceu com as portas **filtered** (a não responder) — comportamento
> defensivo depois de vários scans seguidos.

## Passo 2 — Pedido ARP: "quem é este IP?"

O **ARP** é o mecanismo pelo qual um aparelho pergunta à rede "quem tem este
IP?" e o dono responde "sou eu", revelando o seu endereço físico (**MAC**).

```bash
sudo arping 192.168.1.170
```

Resposta da TV: MAC `68:C8:C0:CF:05:43`.

A primeira metade do MAC (`68:C8:C0`) identifica o **fabricante** — neste caso a
**Google**, coerente com a TV ser um Chromecast.

> Dica: o `arping` precisa de `sudo` (corre como root). Se der "no such device",
> é por não saber a placa de rede a usar.

## Como os aparelhos se encontram na rede (ARP + switch)

Dois aparelhos na mesma rede local **não precisam do "cérebro" do router** para
falar um com o outro:

1. O aparelho **pergunta** por ARP: "quem tem o IP `192.168.1.170`?"
2. O dono **responde** com o seu MAC (endereço físico).
3. A partir daí falam **diretamente**, através do **switch** (a parte que liga
   cabos e Wi-Fi).

Em casa, o switch está **dentro** da mesma caixa do router, por isso o trânsito
ainda passa fisicamente por lá. Numa rede maior, com switch separado, os dois
aparelhos falariam sem o router ver nada.

> Analogia: o router é o porteiro que trata de quem **entra e sai** do prédio.
> Dois vizinhos do mesmo andar podem bater à porta um do outro sem passar pela
> portaria. O ARP é perguntar "em que porta é que tu moras?".

## O que se liga a MAC → fabricante → identidade

O valor forense deste exercício: liguei um **IP** a um **MAC**, e o MAC a um
**fabricante**. Isto permite identificar um aparelho **com certeza**, mesmo que
ele troque de IP, porque o MAC é fixo para aquela placa de rede.

## Comandos usados

| Comando | O que faz |
|---|---|
| `sudo nmap -sn 192.168.1.0/24` | Descobre os aparelhos vivos na rede (sem scan de portas) |
| `sudo arping <ip>` | Pergunta "quem é este IP?" e devolve o MAC |
| `sudo arp-scan --localnet` | Alternativa: varre a rede e lista IP + MAC + fabricante |
