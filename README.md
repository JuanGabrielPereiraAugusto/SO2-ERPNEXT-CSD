# 🐧 Implantação de ERP de gestão de estoque com substituição do Windows por Linux

Trabalho final da disciplina de **Sistemas Operacionais II (SO2)**, do curso de **Análise e Desenvolvimento de Sistemas** da **Fatec Taquaritinga**.

Proposta de troca do Windows pelo **Ubuntu LTS** nos computadores de uma pequena empresa e de implantação do **ERPNext**, um ERP de código aberto, para controle de estoque integrado à contabilidade, **sem custo de licenciamento**.

📄 **Documentação completa:** [`documentacao_SO2.pdf`](documentacao_SO2.pdf)

---

## 🏢 O cenário

A **Central Sul Distribuidora (CSD)** é uma empresa fictícia de pequeno porte que:

- controla o estoque em **planilhas**, com retrabalho de conferência;
- usa apenas computadores com **Windows**;
- não tem **servidor** nem verba para **licenças comerciais**.

## 💡 A solução proposta

- Remover o Windows e instalar o **Ubuntu Desktop LTS** direto nos computadores que a empresa já tem, sem máquina virtual.
- Instalar o **ERPNext de forma nativa** com a ferramenta oficial **`bench`**, em modo de produção (Nginx + Supervisor), sem containers.
- Ativar os módulos de **Estoque** e **Contabilidade**, com saldo por armazém e reflexo contábil automático de cada movimentação.

### Arquitetura

```
 Usuários (Estoque · Financeiro · Gestor · Administrador)
            │  navegador · HTTP porta 80 · rede local
            ▼
┌──────────────────── Ubuntu Desktop LTS ────────────────────┐
│  bench (instala, atualiza e faz backup)                     │
│  systemd + Supervisor (iniciam e reiniciam os serviços)     │
│                                                             │
│  nginx · frappe-web · socketio · schedule                   │
│  worker short · worker long · redis-cache · redis-queue     │
│  mariadb                                                    │
│                                                             │
│  Kernel Linux: processos · memória · arquivos · permissões  │
└─────────────────────────────────────────────────────────────┘
   Hardware: os mesmos computadores da empresa
```

---

## 🛠️ Tecnologias e conceitos de SO aplicados

| Tecnologia | Função na solução | Conceito de SO aplicado |
|---|---|---|
| **Ubuntu LTS** | Sistema operacional no lugar do Windows | Gerência de hardware, processos, memória, arquivos e permissões |
| **WSL 2** *(só laboratório)* | Executa o Ubuntu dentro do Windows | Virtualização leve baseada em Hyper-V |
| **Docker / Compose** *(só laboratório)* | Executa os serviços do ERPNext em containers | Namespaces, cgroups e kernel compartilhado |
| **Nginx, systemd, Supervisor** | Recebem os acessos e mantêm os serviços no ar | Gerenciamento de processos e serviços |
| **MariaDB** | Banco de dados do ERPNext | Persistência e recuperação após falhas |
| **Redis** | Cache e filas de tarefas | Comunicação entre processos |
| **ERPNext / Frappe** | Aplicação web, workers e agendador | Processos concorrentes |
| **bench e Git** | Instalam e atualizam o ERPNext | Instalação de software e controle de versão |

---

## 🧪 Ambiente de laboratório

Como não era possível remover o Windows do computador do grupo, a solução foi validada com:

- **Ubuntu no WSL 2**, limitado a 4 GB de RAM e 2 processadores;
- serviços do ERPNext em **containers Docker**, orquestrados pelo `pwd.yml` do repositório oficial [frappe_docker](https://github.com/frappe/frappe_docker).

Essa escolha agilizou os testes e permitiu observar na prática o **isolamento de processos** e o **controle de recursos pelo kernel**.

---

## ✅ Resultados dos testes

Os **13 casos de teste foram aprovados**, dois deles com ressalva.

| ID | Teste | Resultado |
|---|---|---|
| CT01 | Acesso ao sistema | ✅ ERPNext acessível em `http://localhost:8080` |
| CT02 | Acesso pela rede local | ✅ Login aberto por um celular na mesma rede Wi-Fi |
| CT03 | Cadastro de item | ✅ *Leitura de código de barras simulada por digitação* |
| CT04 | Entrada de estoque | ✅ Saldo atualizado no Livro de Inventário |
| CT05 | Saída e transferência | ✅ Saldos de cada armazém atualizados |
| CT06 | Reflexo contábil | ✅ Débito e crédito gerados automaticamente em BRL |
| CT07 | Fuso horário | ✅ America/Sao_Paulo nas configurações e nas movimentações |
| CT08 | Permissões | ✅ Usuário de estoque bloqueado no módulo Banking |
| CT09 | Persistência dos dados | ✅ MariaDB se recuperou após encerramento forçado |
| CT10 | Backup | ✅ Backup do banco e dos arquivos gerado pelo `bench` |
| CT11 | Consumo de recursos | ✅ *Comparação com VM feita por estimativa teórica* |
| CT12 | Isolamento e kernel compartilhado | ✅ Container vê 6 processos; o Ubuntu vê 31; mesmo kernel |
| CT13 | Limite por cgroup | ✅ Limite de 256 MiB aplicado e confirmado |

---

## 📋 Plano de implementação

| # | Etapa | Situação |
|---|---|---|
| 1 | Preparação | ✅ Concluída (laboratório) |
| 2 | Backup dos dados | 🕓 Planejada |
| 3 | Instalação do Linux | ✅ Concluída (laboratório) |
| 4 | Configuração do sistema | ✅ Concluída (laboratório) |
| 5 | Instalação dos programas | ✅ Concluída (laboratório) |
| 6 | Usuários e permissões | 🕓 Planejada |
| 7 | Rede e periféricos | ⚠️ Parcial (sem impressora e leitor para testar) |
| 8 | Testes | ✅ Concluída (laboratório) |
| 9 | Treinamento do usuário | 🕓 Planejada |
| 10 | Implantação definitiva | 🕓 Planejada |

---

## 💰 Modelo de negócio

Cobrança pelo **serviço**, não por licença (valores de referência):

| Item | Cobrança | Valor |
|---|---|---|
| Implantação completa | Taxa única | R$ 1.800,00 |
| Suporte e manutenção | Mensal | R$ 200,00/mês |
| Treinamento adicional | Por hora | R$ 80,00/h |
| Personalizações e novos módulos | Por hora, sob orçamento | R$ 100,00/h |

---

## 🚀 Como reproduzir o ambiente de laboratório

**Requisitos:** Windows 10/11 com virtualização habilitada, WSL 2 e Docker Desktop com backend WSL 2.

1. Limite os recursos do WSL 2 no arquivo `%UserProfile%\.wslconfig`:
   ```ini
   [wsl2]
   memory=4GB
   processors=2
   ```

2. Baixe o frappe_docker e suba a pilha do ERPNext:
   ```powershell
   git clone https://github.com/frappe/frappe_docker.git
   cd frappe_docker
   docker compose -f pwd.yml up -d
   ```

3. Acompanhe a criação do site:
   ```powershell
   docker compose -f pwd.yml logs -f create-site
   ```

4. Acesse `http://localhost:8080`. O usuário e a senha padrão são os definidos no `pwd.yml`.

> ⚠️ **Atenção:** o `pwd.yml` usa senhas padrão e HTTP sem criptografia. Serve só para testes e estudo, não para produção. O arquivo usado no trabalho é a versão consultada em outubro de 2026; a versão atual pode ter pequenas diferenças.

**Comandos úteis:**

```powershell
docker compose -f pwd.yml ps -a      # estado dos containers
docker stats --no-stream             # consumo de CPU e memória
docker compose -f pwd.yml stop       # parar a pilha
docker compose -f pwd.yml start      # iniciar de novo
```

---

## 🔭 Trabalhos futuros

- Configurar **HTTPS (TLS)** e **backup automático** na instalação nativa.
- Ampliar a comparação de desempenho entre instalação nativa, containers e máquina virtual.
- Usar outros módulos do ERPNext, como **Vendas** e **Compras**.
- Fazer a **migração real** dos computadores para o Ubuntu e repetir os testes com impressora e leitor de código de barras.

---

## 🎓 Informações acadêmicas

| | |
|---|---|
| **Instituição** | Fatec Taquaritinga |
| **Curso** | Análise e Desenvolvimento de Sistemas |
| **Disciplina** | Sistemas Operacionais II (SO2) |
| **Professor** | Marivaldo Aparecido Alcantara |
| **Ano** | 2026 |

## 👥 Integrantes (Grupo Jaboticabal)

- Juan Gabriel Pereira Augusto
- Carlos Eduardo Carvalho Gerônimo
- Vitor Gomes de Oliveira
- Daniel Freire Kim

---

## 📚 Referências principais

- [ERPNext — documentação](https://docs.erpnext.com)
- [bench](https://github.com/frappe/bench)
- [frappe_docker](https://github.com/frappe/frappe_docker)
