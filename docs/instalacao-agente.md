# Instalação do Zabbix Agent 2 no Windows 11

Procedimento usado em cada um dos 60 computadores.

## Pré-requisitos

- Windows 11 com acesso de administrador local
- Instalador do **Zabbix Agent 2 v7.4** (`.msi`)
- Um editor de texto (no projeto, Notepad++)

## Passo 1: instalar o agente

Execute o `.msi` e siga o assistente com as configurações padrão. O agente é instalado em `C:\Program Files\Zabbix Agent 2\`.

## Passo 2: permitir scripts locais

No PowerShell como administrador:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope LocalMachine -Force
```

## Passo 3: abrir o arquivo de configuração

```powershell
& "C:\Program Files\Notepad++\notepad++.exe" "C:\Program Files\Zabbix Agent 2\zabbix_agent2.conf"
```

## Passo 4: ajustar o timeout

Localize a linha `#Timeout=` e troque por:

```
Timeout=30
```

Os scripts que consultam o log de eventos e o Defender passam do padrão de 3 segundos e geram o erro `Timeout occurred while gathering data`.

## Passo 5: adicionar os UserParameters

Copie o conteúdo de [`agent/userparameters.conf`](../agent/userparameters.conf) para o final do `zabbix_agent2.conf`. Se `Timeout=30` já estiver definido no passo 4, não repita a linha.

## Passo 6: reiniciar o serviço

```powershell
Restart-Service "Zabbix Agent 2"
```

## Verificação

```powershell
Get-Service "Zabbix Agent 2"
Get-Content "C:\Program Files\Zabbix Agent 2\zabbix_agent2.log" -Tail 20
```

No servidor Zabbix, teste uma métrica personalizada:

```bash
zabbix_get -s NOME-DO-HOST -k windows.firewall.status
```

## Atenção

Os itens de log de eventos do template são **checks ativos**. Confira no `zabbix_agent2.conf` se `Server`, `ServerActive` e `Hostname` apontam para o seu servidor e se o `Hostname` é igual ao nome do host cadastrado no Zabbix.
