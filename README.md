# 🖥️ PCCT — Monitoramento de Laboratórios de Informática

Sistema de monitoramento de desempenho e disponibilidade de **60 computadores** em **3 laboratórios de informática** do IFAM Campus Coari, usando **Zabbix**, **Grafana** e **Proxmox VE**.

Projeto de Conclusão de Curso Técnico (PCCT) do curso **Técnico em Manutenção e Suporte em Informática**, com 250 horas de carga de trabalho.

---

## 📐 Arquitetura

```
 Windows 11 (60 PCs)          Proxmox VE
 ┌──────────────────┐         ┌──────────────────────────────┐
 │ Zabbix Agent 2   │ ──────► │ VM Linux Mint                │
 │ + UserParameters │  :10050 │  ├─ Zabbix Server 7.4        │
 │   (PowerShell)   │         │  ├─ MariaDB                  │
 └──────────────────┘         │  ├─ Nginx (frontend Zabbix)  │
                              │  └─ Grafana OSS              │
                              └──────────────┬───────────────┘
                                             │ SMTP (STARTTLS)
                                             ▼
                                     Alertas por e-mail
```

Os hosts são monitorados **pelo nome DNS**, não pelo IP, porque a rede usa DHCP. Veja [docs/estrategia-dns.md](docs/estrategia-dns.md).

---

## 📦 O que tem neste repositório

| Pasta | Conteúdo |
|---|---|
| [`template/`](template/) | Template do Zabbix (`.yaml`) pronto para importar |
| [`grafana/`](grafana/) | Dashboard do Grafana (`.json`) pronto para importar |
| [`agent/`](agent/) | UserParameters do Zabbix Agent 2 (métricas personalizadas em PowerShell) |
| [`docs/`](docs/) | Instalação do agente, alertas por e-mail e estratégia de DNS |

---

## 🧩 Template "Lab Escola" (Zabbix 7.4)

| Item | Quantidade |
|---|---|
| Itens de monitoramento | 53 |
| Triggers | 33 |
| Regras de descoberta (LLD) | 2 (partições de disco e interfaces de rede) |
| Protótipos de itens / triggers | 11 / 1 |
| Macros | 9 |
| Dashboard nativo do Zabbix | 1 |

**O que é monitorado:** CPU (uso, carga, temperatura), memória RAM, paginação, disco por partição, tráfego de rede por interface, serviços do Windows (Spooler, Defender, Windows Update, hora, áudio, log de eventos), uptime, processos, usuários logados, ping e latência (interna e para a internet), ativação do Windows, atualizações pendentes, firewall, dispositivos USB e falhas de login.

**Severidades:** cinco níveis, de Atenção a Desastre. Os limites de CPU, RAM, temperatura e disco ficam em macros e podem ser ajustados.

### Métricas personalizadas

9 itens dependem de UserParameters em PowerShell, que estão em [`agent/userparameters.conf`](agent/userparameters.conf):

`windows.top.process` · `windows.defender.daysoutdated` · `windows.defender.realtime` · `windows.usb.count` · `windows.login.failures` · `windows.firewall.status` · `windows.activation.status` · `temperatura.cpu.celsius` · `windows.updates.pending`

---

## 📊 Dashboard do Grafana

- **33 painéis** em **5 seções**: visão geral, sistema, performance, rede e eventos
- Variáveis encadeadas: selecione o **grupo** (laboratório) e depois o **host**
- Usa o plugin [`alexanderzobnin-zabbix-datasource`](https://github.com/grafana/grafana-zabbix)

---

## 🚀 Como reproduzir

1. **Servidor:** instale o Zabbix Server 7.4 com MariaDB e Nginx (no projeto, em uma VM no Proxmox).
2. **Template:** no Zabbix, vá em *Data collection → Templates → Import* e envie `template/template-lab-escola.yaml`.
3. **Agentes:** em cada Windows, instale o Zabbix Agent 2 e adicione o conteúdo de `agent/userparameters.conf`. Passo a passo em [docs/instalacao-agente.md](docs/instalacao-agente.md).
4. **Hosts:** cadastre os computadores no Zabbix com o template aplicado.
5. **Grafana:** instale o plugin do Zabbix, configure a fonte de dados e importe `grafana/lab-escola-dashboard.json` escolhendo a fonte quando o Grafana pedir (`DS_ZABBIX`).
6. **Alertas:** configure o e-mail conforme [docs/alertas-email.md](docs/alertas-email.md).

---

## ⚠️ Limitações conhecidas

- A **temperatura da CPU** usa WMI (`MSAcpi_ThermalZoneTemperature`). Em máquinas que não expõem esse dado, o item retorna `-1`.
- Se o registro DNS de uma máquina estiver desatualizado, o Zabbix tenta conectar no IP errado e o host aparece como indisponível. No projeto, isso foi comunicado à equipe de TI da instituição.
- A instalação dos agentes foi manual em cada computador, porque o deploy remoto por script foi bloqueado pelas políticas da rede.
- `UnsafeUserParameters=1` é necessário para os scripts. Use apenas em rede confiável.

---

## 👥 Autores

Projeto desenvolvido no **IFAM Campus Coari** por:

- Efraim da Silva
- Eloin da Silva e Silva
- Marcos Andrey Santos Abensur

**Orientador:** Prof. Carlos Henrique Ferreira Neto

---

## 🛠️ Tecnologias

Zabbix 7.4 · Grafana OSS · Proxmox VE · Linux Mint · MariaDB · Nginx · Windows 11 · PowerShell · WMI
