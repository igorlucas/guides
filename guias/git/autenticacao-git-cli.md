# Guia Completo de Autenticação Git para Repositórios Remotos

Este guia apresenta os principais métodos de autenticação utilizados atualmente com Git em plataformas de repositórios remotos, como:

- GitHub
- GitLab
- Azure DevOps
- Bitbucket
- GitHub Enterprise
- GitLab Self-Managed
- Gitea
- Gerrit

O objetivo é explicar os conceitos e fornecer exemplos práticos para uso no dia a dia.

---

# Conceito Fundamental

Uma dúvida muito comum é:

> "Como faço login no Git?"

Na prática, o Git não possui um mecanismo próprio de autenticação de usuários.

O Git é apenas uma ferramenta de controle de versão.

Quando você executa comandos como:

```bash
git clone
git pull
git push
```

o Git delega a autenticação para o servidor remoto.

Em outras palavras:

```text
Git
 ↓
Protocolo (HTTPS ou SSH)
 ↓
Provedor remoto
 ↓
Autenticação
```

O responsável por validar suas credenciais é o repositório remoto:

- GitHub
- GitLab
- Bitbucket
- Azure DevOps
- etc.

---

# Métodos de Autenticação Mais Utilizados Atualmente

Atualmente existem três formas principais de autenticação.

## 1. Login via Navegador (OAuth / Device Flow)

Experiência mais comum para usuários finais.

Fluxo típico:

```text
Git
 ↓
Credential Manager
 ↓
Browser
 ↓
Login
 ↓
MFA
 ↓
Autorização
 ↓
Token armazenado localmente
```

Exemplos:

- Git Credential Manager (GCM)
- GitHub CLI (`gh auth login`)
- Azure CLI
- Ferramentas corporativas integradas a SSO

### Vantagens

✅ Não exige copiar tokens

✅ Compatível com MFA

✅ Compatível com SSO corporativo

✅ Fácil para usuários finais

---

## 2. Personal Access Token (PAT)

Substitui o uso de senha em conexões HTTPS.

Fluxo típico:

```text
Git
 ↓
HTTPS
 ↓
PAT
 ↓
Provedor remoto
```

### Vantagens

✅ Fácil de automatizar

✅ Funciona em scripts

✅ Funciona em pipelines

✅ Compatível com praticamente qualquer ambiente

### Desvantagens

⚠️ Necessidade de gerenciar o token

⚠️ Risco de vazamento caso armazenado incorretamente

---

## 3. SSH

Muito utilizado por desenvolvedores e equipes técnicas.

Fluxo típico:

```text
Git
 ↓
SSH Key
 ↓
SSH Agent
 ↓
Provedor remoto
```

### Vantagens

✅ Excelente para uso diário

✅ Não exige PAT

✅ Não exige login recorrente

✅ Bastante utilizado em ambientes corporativos

---

# Qual Método Escolher?

## Desenvolvedor utilizando Git diariamente

Recomendação:

```text
SSH
```

ou

```text
Credential Manager
```

---

## Usuários Corporativos com SSO e MFA

Recomendação:

```text
Login via navegador
```

---

## Scripts e Automações

Recomendação:

```text
PAT
```

---

## Ambientes Temporários

Recomendação:

```text
PAT temporário
```

sem persistência em arquivos.

---

# Compatibilidade dos Exemplos

Os exemplos deste guia utilizam sintaxe POSIX Shell.

Funcionam diretamente em:

| Sistema Operacional | Shell |
|----------|----------|
| Linux | bash |
| Linux | zsh |
| Linux | sh |
| macOS | zsh |
| macOS | bash |
| Windows | Git Bash |
| Windows | WSL |

Podem exigir adaptação em:

| Sistema Operacional | Shell |
|----------|----------|
| Windows | PowerShell |
| Windows | CMD |
| Linux/macOS | Fish |

> O helper personalizado apresentado neste guia depende de sintaxe compatível com shell POSIX.

---

# Como Descobrir Qual Shell Está em Uso

## Linux / macOS / Git Bash / WSL

```bash
echo $SHELL
```

Exemplo:

```text
/bin/bash
```

ou

```text
/bin/zsh
```

---

## Fish

```fish
echo $SHELL
```

---

## PowerShell

```powershell
$PSVersionTable
```

---

## CMD

```cmd
echo %COMSPEC%
```

Resultado típico:

```text
C:\Windows\System32\cmd.exe
```

---

# Como o Git Utiliza um PAT?

Ao executar:

```bash
git clone https://github.com/acme-finance/api-pagamentos.git
```

o comando não contém credenciais.

Para funcionar, o Git precisa obter as credenciais de alguma forma.

As opções mais comuns são:

---

## Opção A - Git Solicita as Credenciais

```bash
git clone https://github.com/acme-finance/api-pagamentos.git
```

Git pergunta:

```text
Username:
```

Informe:

```text
x-access-token
```

ou seu usuário GitHub.

Depois:

```text
Password:
```

Informe seu PAT:

```text
github_pat_xxxxxxxxxxxxxxxxxxxxxxxxx
```

---

## Opção B - PAT Fornecido por Helper

Primeiro o PAT é armazenado em uma variável:

```bash
export GITHUB_PAT="github_pat_1234567890abcdefghijklmnopqrstuvwxyz"
```

Depois configuramos o helper:

```bash
git config credential.helper \
'!f() { echo username=x-access-token; echo "password=$GITHUB_PAT"; }; f'
```

Somente então executamos:

```bash
git clone https://github.com/acme-finance/api-pagamentos.git
```

Fluxo:

```text
PAT
 ↓
Variável de Ambiente
 ↓
Credential Helper
 ↓
Git
 ↓
GitHub
```

---

## Opção C - Credential Manager

Você executa:

```bash
git clone https://github.com/acme-finance/api-pagamentos.git
```

O sistema abre um navegador para autenticação.

Fluxo:

```text
Git
 ↓
Credential Manager
 ↓
Browser
 ↓
Login
 ↓
Token seguro
```

---

## Opção D - SSH

Nenhum PAT é utilizado.

O Git usa uma chave SSH previamente cadastrada.

Exemplo:

```bash
git clone git@github.com:acme-finance/api-pagamentos.git
```

Fluxo:

```text
SSH Key
 ↓
GitHub
```

---

# HTTPS + PAT

## PAT Temporário via Variável de Ambiente

### Exemplo Didático

⚠️ O token abaixo é fictício.

```bash
export GITHUB_PAT="github_pat_1234567890abcdefghijklmnopqrstuvwxyz"
```

Configurar helper:

```bash
git config credential.helper \
'!f() { echo username=x-access-token; echo "password=$GITHUB_PAT"; }; f'
```

Clonar:

```bash
git clone https://github.com/acme-finance/api-pagamentos.git
```

Entrar no projeto:

```bash
cd api-pagamentos
```

Executar pull:

```bash
git pull
```

Executar push:

```bash
git push origin main
```

Limpar:

```bash
unset GITHUB_PAT

git config --unset credential.helper
```

---

## PAT Informado Interativamente

Sem deixar o PAT visível no comando.

```bash
read -s -p "GitHub PAT: " GITHUB_PAT
echo
```

Configurar helper:

```bash
git config credential.helper \
'!f() { echo username=x-access-token; echo "password=$GITHUB_PAT"; }; f'
```

Executar:

```bash
git clone https://github.com/acme-finance/api-pagamentos.git
```

Finalizar:

```bash
unset GITHUB_PAT

git config --unset credential.helper
```

---

## Utilização para Apenas um Comando

```bash
read -s -p "GitHub PAT: " GITHUB_PAT
echo

git \
-c credential.helper='!f() { echo username=x-access-token; echo "password='"$GITHUB_PAT"'"; }; f' \
clone https://github.com/acme-finance/api-pagamentos.git

unset GITHUB_PAT
```

---

# SSH (Recomendado para Uso Diário)

## Verificar se já existem chaves

Linux/macOS/Git Bash:

```bash
ls -la ~/.ssh
```

PowerShell:

```powershell
Get-ChildItem $HOME\.ssh
```

---

## Gerar uma chave SSH

```bash
ssh-keygen -t ed25519 -C "joao.silva@acme-finance.com"
```

Pressione ENTER para aceitar o caminho padrão.

Exemplo:

```text
/home/usuario/.ssh/id_ed25519
```

---

## Iniciar SSH Agent

```bash
eval "$(ssh-agent -s)"
```

Adicionar chave:

```bash
ssh-add ~/.ssh/id_ed25519
```

---

## Exibir chave pública

Linux/macOS/Git Bash:

```bash
cat ~/.ssh/id_ed25519.pub
```

PowerShell:

```powershell
Get-Content $HOME\.ssh\id_ed25519.pub
```

Exemplo:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIExemplo1234567890abcdef joao.silva@acme-finance.com
```

---

## Adicionar chave ao GitHub

Acesse:

```text
Settings
→ SSH and GPG Keys
→ New SSH Key
```

Cole a chave pública.

---

## Testar conexão

```bash
ssh -T git@github.com
```

Resultado esperado:

```text
Hi username! You've successfully authenticated.
```

---

## Clonar utilizando SSH

HTTPS:

```text
https://github.com/acme-finance/api-pagamentos.git
```

SSH:

```text
git@github.com:acme-finance/api-pagamentos.git
```

Clone:

```bash
git clone git@github.com:acme-finance/api-pagamentos.git
```

---

# Variáveis de Ambiente por Shell

## Bash / Zsh / Sh

```bash
export GITHUB_PAT="<SEU_PAT>"
```

Consultar:

```bash
echo $GITHUB_PAT
```

Remover:

```bash
unset GITHUB_PAT
```

---

## Fish

```fish
set -x GITHUB_PAT "<SEU_PAT>"
```

Consultar:

```fish
echo $GITHUB_PAT
```

Remover:

```fish
set -e GITHUB_PAT
```

---

## PowerShell

```powershell
$env:GITHUB_PAT="<SEU_PAT>"
```

Consultar:

```powershell
$env:GITHUB_PAT
```

Remover:

```powershell
Remove-Item Env:GITHUB_PAT
```

---

## CMD

```cmd
set GITHUB_PAT=<SEU_PAT>
```

Consultar:

```cmd
echo %GITHUB_PAT%
```

Remover:

```cmd
set GITHUB_PAT=
```

---

# O Que Evitar

## ❌ PAT na URL

Embora algumas plataformas permitam isso:

```bash
git clone https://x-access-token:github_pat_xxxxxxxxxxxxx@github.com/acme-finance/api-pagamentos.git
```

ou

```bash
git clone https://usuario:github_pat_xxxxxxxxxxxxx@github.com/acme-finance/api-pagamentos.git
```

não é recomendado.

O PAT pode ficar exposto em:

- Histórico do terminal
- Logs
- Scripts
- Ferramentas de monitoramento
- Lista de processos

---

## ❌ PAT Persistido em Arquivos

Evite armazenar PAT em:

- `.gitconfig`
- `.bashrc`
- `.zshrc`
- `.profile`
- Scripts versionados
- Repositórios Git
- Wikis compartilhadas

---

## ❌ Commitar Tokens Acidentalmente

Antes de enviar alterações:

```bash
git diff
```

e revise o conteúdo cuidadosamente.

---

# Comparação dos Métodos

| Método | Uso Diário | Automação | Facilidade |
|----------|----------|----------|----------|
| Login via Navegador | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| PAT | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| SSH | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

# Recomendação Final

## Melhor experiência para usuários corporativos

```text
Login via navegador
```

através de:

```text
Git Credential Manager
GitHub CLI
Azure CLI
```

---

## Melhor experiência para desenvolvedores

```text
SSH
```

---

## Melhor opção para automação

```text
PAT
```

---

# TL;DR

Uso ocasional:

```bash
read -s -p "GitHub PAT: " GITHUB_PAT && echo

git \
-c credential.helper='!f() { echo username=x-access-token; echo "password='"$GITHUB_PAT"'"; }; f' \
clone https://github.com/acme-finance/api-pagamentos.git

unset GITHUB_PAT
```

Uso recorrente:

```bash
ssh-keygen -t ed25519 -C "seu-email@empresa.com"

git clone git@github.com:ORG/REPO.git
```

✅ Sem PAT na URL

✅ Sem PAT em arquivos

✅ Compatível com GitHub, GitLab, Azure DevOps e Bitbucket

✅ Mais seguro para utilização diária