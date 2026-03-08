# 🚀 Desafio Técnico: Automação de Matrículas e Boas-vindas

Este projeto consiste em uma automação desenvolvida no **n8n** para o processamento de matrículas de novos alunos. O sistema recebe dados via Webhook, realiza o tratamento técnico das informações via Python, faz o registro inteligente em banco de dados PostgreSQL e automatiza o envio de e-mails transacionais.

## 📺 Demonstração e Explicação Técnica
Assista ao vídeo abaixo para entender a arquitetura do fluxo, as expressões utilizadas e a resolução do desafio "oculto":

[Clique aqui para assistir ao vídeo de apresentação](https://github.com/user-attachments/assets/d71136d2-d61c-41e0-88ff-debdb1b3c836) 

---

## 🛠️ Tecnologias e Recursos Utilizados
* **n8n**: Orquestração e lógica do fluxo.
* **Python (Beta Node)**: Tratamento avançado de strings, datas e validações.
* **PostgreSQL**: Persistência de dados com foco em integridade.
* **Gmail Node**: Comunicação transacional personalizada.
* **Webhooks**: Porta de entrada para os dados dos alunos (Método POST).

---

## 🧠 Solução do "Desafio Oculto"
A descrição original do processo possuía um ponto falho que poderia gerar erros operacionais graves. 

**O Risco:** A falta de verificação de duplicidade. Sem um controle, um mesmo aluno poderia ser cadastrado múltiplas vezes, gerando duplicidade no banco e e-mails de boas-vindas repetidos.

**A Solução:** Implementei uma camada dupla de proteção:
1. **No Banco de Dados:** As tabelas possuem uma constraint `UNIQUE` no campo de e-mail.
2. **Na Query SQL:** Utilizei uma **CTE (Common Table Expression)** com a cláusula `ON CONFLICT ("email") DO NOTHING`.
   * O banco ignora a inserção se o e-mail já existir.
   * A query retorna um `status` ('inserido' ou 'duplicado').
   * O fluxo n8n utiliza um nó **IF** para garantir que o e-mail de boas-vindas só seja enviado para novos cadastros.

---

## 📋 Tratamento de Dados Realizado
Conforme os requisitos obrigatórios, os dados passam pelas seguintes transformações:

* **Nome Completo**: Normalizado para que a primeira letra de cada nome seja maiúscula.
* **Telefone**: Limpeza total de caracteres especiais (parênteses, espaços e traços) via Regex, mantendo apenas números.
* **Data de Nascimento**: Conversão do formato brasileiro (DD/MM/YYYY) para o padrão americano/ISO (YYYY-MM-DD).
* **Roteamento de Curso**: Uso de lógica **Switch/IF** para separar alunos entre as tabelas `matriculas_tech` e `matriculas_biz`.

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
