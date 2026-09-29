# Alertas por e-mail no Zabbix 7.4

Configuração usada para receber notificações dos problemas detectados. O exemplo usa Gmail como provedor SMTP.

## Pré-requisitos

- Acesso administrativo ao frontend do Zabbix
- Uma conta de e-mail **dedicada ao projeto**, com verificação em duas etapas ativada
- Uma **senha de aplicativo** (16 caracteres) gerada para essa conta
- Hosts cadastrados com o template "Lab Escola"

## Passo 1: gerar a senha de aplicativo

1. Ative a verificação em duas etapas em `myaccount.google.com/security`.
2. Acesse `myaccount.google.com/apppasswords`.
3. Dê um nome à credencial (por exemplo, "Zabbix Server") e clique em criar.
4. Copie a senha de 16 caracteres, sem os espaços. Ela aparece uma única vez.

> Nunca publique essa senha nem coloque o e-mail da conta em repositórios públicos.

## Passo 2: criar o tipo de mídia

Em *Alerts → Media types*:

| Campo | Valor |
|---|---|
| Nome | Email - Laboratorios |
| Provedor | Generic SMTP |
| Servidor SMTP | `smtp.gmail.com` |
| Porta | `587` |
| E-mail | conta dedicada do projeto |
| SMTP helo | `gmail.com` |
| Segurança da conexão | STARTTLS |
| Autenticação | Usuário e senha (e-mail da conta + senha de aplicativo) |

Use o link **Test** para enviar um e-mail de teste. O resultado esperado é `Sent successfully`. Se falhar, a mensagem de erro do Zabbix costuma indicar a causa. No projeto, o erro foi um erro de digitação no servidor SMTP.

## Passo 3: grupo de hosts

Crie o grupo `Laboratorios - PCs Monitorados` em *Data collection → Host groups* e adicione os hosts com **Mass update**.

## Passo 4: mídia do usuário

Em *Users → Users*, abra o usuário que vai receber os alertas, aba **Media**:

- Tipo: o tipo de mídia criado no passo 2
- Enviar para: o e-mail de destino
- Ativo quando: `1-7,00:00-24:00`
- Severidades: pelo menos **Average**, **High** e **Disaster**

## Passo 5: ação de trigger

Em *Alerts → Actions → Trigger actions*, crie uma ação com:

- **Condições** (operador E): grupo de hosts igual a `Laboratorios - PCs Monitorados` e severidade maior ou igual a **Average**
- **Operação:** enviar mensagem ao usuário, usando o tipo de mídia de e-mail
- **Operação de recuperação:** igual à anterior, para avisar quando o problema for resolvido

## Passo 6: validar

Acompanhe com hosts ativos e confirme que os e-mails chegam. Se não chegarem, veja *Reports → Action log*, que mostra o status de cada envio e o erro correspondente.
