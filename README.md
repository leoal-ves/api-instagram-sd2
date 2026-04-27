# Arquitetura e Contrato da API - Instagram Clone

Bem-vindo ao repositório de documentação da API do Instagram Clone.

Este diretório contém os artefatos de documentação que definem o contrato de comunicação entre o Frontend (Cliente) e os diversos Backends (Servidores) desenvolvidos por diferentes times utilizando stacks variadas (ex: Java/Spring, PHP/Laravel, Node/Express, JS/Elysia, TS/Adonis, Python/Django/FastAPI, C#/Dotnet).

## 🗂 Estrutura do Diretório

- [`/requisitos/requisitos.md`](./requisitos/requisitos.md) - Especificação funcional e não-funcional base do sistema. Define TODAS as restrições que cada backend obrigatoriamente tem que respeitar.
- [`/swagger/swagger.md`](./swagger/swagger.md) - Documento com decisões arquiteturais de design de API RESTful.
- [`openapi.json`](./openapi.json) - O schema principal OpenAPI 3.0 para importação em Postman, Insomnia ou geração de código automatizada (Swagger Codegen).

## 🚀 Como Consumir

1. Leia primeiro o documento de `requisitos.md` para entender o escopo de negócio.
2. Em seguida, acesse `swagger.md` para entender as convenções de resposta JSON padronizada (Envelope Pattern) para uniformidade em todos os ecossistemas.
3. Importe o arquivo JSON OpenAPI no seu HTTP Client favorito para interagir e testar as especificações.

---

> **Ponto de Atenção para os Desenvolvedores:**
> Qualquer implementação Backend DEVE garantir que os schemas de retorno sejam exatamente o Envelope Pattern descrito no OpenAPI, caso contrário, os times de Front-end enfrentarão quebras severas na visualização (parsing de dados de `res.data`).
