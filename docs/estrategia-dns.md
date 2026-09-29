# Estratégia de monitoramento por nome DNS

## Problema

Os 60 computadores recebem IP por **DHCP**. O IP de uma máquina muda após reinício ou renovação da concessão, então usar IP como identificador no Zabbix quebraria o histórico e interromperia o monitoramento.

## Solução

Cada host é cadastrado no Zabbix pelo **nome DNS completo** (`hostname.dominio-interno`), e o Zabbix usa esse nome como interface de conexão com o agente (porta TCP 10050). O nome sempre resolve para o IP atual da máquina.

## Validação antes de cadastrar

Na VM do Zabbix, para cada host:

```bash
nslookup NOME-DO-HOST.dominio-interno
ping -c 4 NOME-DO-HOST.dominio-interno
```

Na rede do projeto, a latência média foi de cerca de 1 ms.

## Problema encontrado: registros DNS desatualizados

Parte das máquinas de um dos laboratórios tinha registros DNS com IPs que já não correspondiam aos atuais. Nesse caso o Zabbix tenta conectar no IP errado e registra:

- `No route to host`
- `Cannot establish TCP connection`

Para diagnosticar, compare o resultado do `nslookup` com o IP real da máquina (`ipconfig` no Windows). Se forem diferentes, o registro DNS precisa ser corrigido pela equipe responsável pelo servidor de nomes.
