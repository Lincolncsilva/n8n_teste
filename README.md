# 🚀 Desafio Técnico: Automação de Matrículas e Boas-vindas

Este projeto consiste em uma automação desenvolvida no **n8n** para o processamento de matrículas de novos alunos. [cite_start]O sistema recebe dados via Webhook, realiza o tratamento técnico das informações via Python, faz o registro inteligente em banco de dados PostgreSQL e automatiza o envio de e-mails transacionais[cite: 4, 6, 22].

## 📺 Demonstração e Explicação Técnica
Assista ao vídeo abaixo para entender a arquitetura do fluxo, as expressões utilizadas e a resolução do desafio "oculto":

[cite_start][Clique aqui para assistir ao vídeo de apresentação](https://github.com/user-attachments/assets/d71136d2-d61c-41e0-88ff-debdb1b3c836) [cite: 48, 49]

---

## 🛠️ Tecnologias e Recursos Utilizados
* [cite_start]**n8n**: Orquestração e lógica do fluxo[cite: 4, 33, 37].
* [cite_start]**Python (Beta Node)**: Tratamento avançado de strings, datas e validações[cite: 13, 14, 24].
* [cite_start]**PostgreSQL**: Persistência de dados com foco em integridade[cite: 12, 30, 31].
* [cite_start]**Gmail Node**: Comunicação transacional personalizada[cite: 16, 33].
* [cite_start]**Webhooks**: Porta de entrada para os dados dos alunos (Método POST)[cite: 10, 23].

---

## 🧠 Solução do "Desafio Oculto"
[cite_start]A descrição original do processo possuía um ponto falho que poderia gerar erros operacionais graves[cite: 35, 36]. 

[cite_start]**O Risco:** A falta de verificação de duplicidade[cite: 37]. [cite_start]Sem um controle, um mesmo aluno poderia ser cadastrado múltiplas vezes, gerando duplicidade no banco e e-mails de boas-vindas repetidos[cite: 34, 36].

**A Solução:** Implementei uma camada dupla de proteção:
1. **No Banco de Dados:** As tabelas possuem uma constraint `UNIQUE` no campo de e-mail.
2. **Na Query SQL:** Utilizei uma **CTE (Common Table Expression)** com a cláusula `ON CONFLICT ("email") DO NOTHING`.
   * O banco ignora a inserção se o e-mail já existir.
   * A query retorna um `status` ('inserido' ou 'duplicado').
   * [cite_start]O fluxo n8n utiliza um nó **IF** para garantir que o e-mail de boas-vindas só seja enviado para novos cadastros[cite: 33, 34].

---

## 📋 Tratamento de Dados Realizado
[cite_start]Conforme os requisitos obrigatórios, os dados passam pelas seguintes transformações[cite: 7, 8]:

* [cite_start]**Nome Completo**: Normalizado para que a primeira letra de cada nome seja maiúscula[cite: 26].
* [cite_start]**Telefone**: Limpeza total de caracteres especiais (parênteses, espaços e traços) via Regex, mantendo apenas números[cite: 27].
* [cite_start]**Data de Nascimento**: Conversão do formato brasileiro (DD/MM/YYYY) para o padrão americano/ISO (YYYY-MM-DD)[cite: 14, 28].
* [cite_start]**Roteamento de Curso**: Uso de lógica **Switch/IF** para separar alunos entre as tabelas `matriculas_tech` e `matriculas_biz`[cite: 11, 30, 31].

---

## 🗄️ Estrutura do Banco de Dados
Para suportar a automação, foram criadas tabelas com a seguinte estrutura:

```sql
CREATE TABLE matriculas_tech (
    nome VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    curso VARCHAR(50) NOT NULL,
    telefone VARCHAR(20) NOT NULL,
    data_nascimento DATE NOT NULL
    );

-- O mesmo padrão foi aplicado à tabela matriculas_biz
