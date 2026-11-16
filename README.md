# 🐧 ERP de gestão de estoque com Linux no lugar do Windows

Trabalho final da disciplina de **Sistemas Operacionais II (SO2)**, do curso de **Análise e Desenvolvimento de Sistemas** da **Fatec Taquaritinga**.

A proposta substitui o **Windows** pelo **Ubuntu LTS** nos computadores de uma pequena empresa e implanta o **ERPNext**, um ERP de código aberto, para controlar o estoque integrado à contabilidade, **sem pagar nenhuma licença**.

| Material | Descrição |
|---|---|
| 📄 [`documentacao_SO2.pdf`](documentacao_SO2.pdf) | Documentação completa do trabalho (56 páginas, normas ABNT) |
| 📊 [`apresentacao_SO2.pptx`](apresentacao_SO2.pptx) | Slides da apresentação (13 slides) |

---

## 🏢 O cliente

A **Central Sul Distribuidora (CSD)** é uma empresa fictícia de pequeno porte:

| Problema | Situação |
|---|---|
| 📑 **Planilhas soltas** | Erros de digitação, saldo diferente do estoque físico e nenhum histórico confiável |
| 🪟 **Só Windows** | Sem servidor, sem ambiente Linux e sem equipe de TI |
| 💸 **Licenças caras** | ERPs comerciais custam mais do que a empresa pode pagar |

**A empresa precisa de:** estoque integrado à contabilidade, em reais (**BRL**), no fuso **America/Sao_Paulo** e **sem custo de licença**.

---

## 💡 A solução

- **Ubuntu Desktop LTS** instalado direto no disco dos computadores que a empresa já tem
- **ERPNext instalado de forma nativa** pela ferramenta oficial **`bench`**
- Serviços rodando como **processos comuns do Ubuntu**, sem máquina virtual e sem containers
- Acesso pelo **navegador**, na porta **80** (Nginx)
- **R$ 0 em licenças**: Ubuntu e ERPNext são livres

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

## 🧪 Validação em laboratório

Como não dava para formatar o computador do grupo, a solução foi testada assim:

| Camada | Como foi usada |
|---|---|
| 🪟 **Windows 11 + WSL 2** | Ubuntu virtualizado, com kernel Linux real |
| 🐳 **Docker Compose** | Serviços do ERPNext em containers, pelo `pwd.yml` oficial do [frappe_docker](https://github.com/frappe/frappe_docker) |
| 🖥️ **Pentium G4560 · 12 GB de RAM** | WSL 2 limitado a **4 GB** e **2 CPUs** pelo `.wslconfig` |

O ERPNext foi configurado no **Setup Wizard** com a empresa **CSD**, moeda **BRL**, fuso **America/Sao_Paulo** e os módulos de **Estoque** e **Contabilidade**, acessado em `http://localhost:8080`.

---

## 🧠 Conceitos de Sistemas Operacionais na prática

| Conceito | O que foi demonstrado |
|---|---|
| **Namespaces** | O container usa o **mesmo kernel** do Ubuntu (`6.18.40.1-microsoft-standard-WSL2`), mas enxerga **só 6 processos**; o PID 1 dele é o `gunicorn`. No Ubuntu aparecem 31. |
| **Cgroups** | Um container criado com `--memory=256m` mostra `268435456` em `memory.max`: exatamente **256 MiB**, o limite imposto pelo kernel. |
| **Recuperação após falha** | Ao parar a pilha, o banco recebe **SIGKILL** depois de 10 s sem responder (código **137 = 128 + 9**). Na volta, o InnoDB reaplica o log de transações (**5.646 páginas**) e nenhum dado se perde. |
| **Gerência de processos e serviços** | Nginx, systemd e Supervisor iniciam os serviços com o sistema e os reiniciam em caso de falha. |
| **Virtualização leve** | O WSL 2 aloca memória de forma dinâmica, consumindo menos que uma VM completa de mesmo tamanho. |

---

## ✅ Resultados dos testes

**13 de 13 casos de teste aprovados**, dois deles com ressalva.

| ID | Teste | Resultado |
|---|---|---|
| CT01 | Acesso ao sistema | ✅ ERPNext acessível em `http://localhost:8080` |
| CT02 | Acesso pela rede local | ✅ Login aberto por um celular na mesma rede Wi-Fi (`192.168.0.9:8080`) |
| CT03 | Cadastro de item | ✅ *Leitura de código de barras simulada por digitação* |
| CT04 | Entrada de estoque | ✅ Entrada de 10 unidades refletida no Livro de Inventário |
| CT05 | Saída e transferência | ✅ Saldos de cada armazém atualizados |
| CT06 | Reflexo contábil | ✅ Entrada de 5 × R$ 10,00 gerou R$ 50,00 a débito e a crédito |
| CT07 | Fuso horário | ✅ America/Sao_Paulo nas configurações e nas movimentações |
| CT08 | Permissões | ✅ Usuário Stock User barrado no módulo Banking |
| CT09 | Persistência dos dados | ✅ Banco recuperado sozinho após o encerramento forçado |
| CT10 | Backup | ✅ Backup do banco e dos arquivos gerado pelo `bench` |
| CT11 | Consumo de recursos | ✅ *Comparação com VM feita por estimativa teórica* |
| CT12 | Isolamento e kernel compartilhado | ✅ Mesmo kernel; o container vê só os seus processos |
| CT13 | Limite por cgroup | ✅ Limite de 256 MiB aplicado e confirmado |

| 13/13 | 5 de 10 | R$ 0 |
|:---:|:---:|:---:|
| casos de teste aprovados | etapas do plano concluídas em laboratório (+1 parcial) | em licenças |

---

## 💰 Modelo de negócio

Cobrança pelo **serviço**, sem cobrança de licença:

| Item | Cobrança | Valor de referência |
|---|---|---|
| Implantação completa | Taxa única | R$ 1.800,00 |
| Suporte e manutenção | Mensal | R$ 200,00/mês |
| Treinamento adicional | Por hora | R$ 80,00/h |
| Personalizações e novos módulos | Por hora, sob orçamento | R$ 100,00/h |

---

## 💻 Código e configuração

**Limite do WSL 2** (`%UserProfile%\.wslconfig`, só no laboratório):

```ini
[wsl2]
memory=4GB
processors=2
```

O `.wslconfig` limita a **VM inteira** do WSL 2, e não cada container.

**Subida da pilha no laboratório** (PowerShell):

```powershell
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
docker compose -f pwd.yml up -d
docker compose -f pwd.yml logs -f create-site
```

**Instalação nativa proposta para a CSD** (Ubuntu):

```bash
bench init frappe-bench
cd frappe-bench
bench new-site <site>
bench get-app erpnext
bench --site <site> install-app erpnext
sudo bench setup production <usuário>
sudo ufw allow 80/tcp
```

**Teste de cgroups:**

```powershell
docker run -d --name teste --memory=256m ubuntu sleep 300
docker exec teste cat /sys/fs/cgroup/memory.max   # 268435456 (256 MiB)
docker rm -f teste
```

> ⚠️ O `pwd.yml` usa senhas padrão e HTTP sem criptografia. Serve só para **testes e estudo**, não para produção.

A lista completa de comandos está no **Apêndice A** da documentação, e os serviços do `pwd.yml` estão no **Apêndice B**.

---

## 🔭 Próximos passos

- Configurar **HTTPS (TLS)** e **backup automatizado** na instalação nativa
- Fazer a **migração real** das estações da CSD para o Ubuntu
- Repetir os testes com **impressora** e **leitor de código de barras** físicos
- Ampliar o uso do ERPNext para **Vendas** e **Compras**

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
- Vitor Gomes de Oliveira
- Carlos Eduardo Carvalho Gerônimo
- Daniel Freire Kim

---

## 📚 Referências principais

- [ERPNext — documentação](https://docs.erpnext.com)
- [bench](https://github.com/frappe/bench)
- [frappe_docker](https://github.com/frappe/frappe_docker)
