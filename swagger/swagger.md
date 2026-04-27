# Documentação da API - Arquitetura e Decisões RESTful

A especificação OpenAPI (Swagger) contida na raiz deste repositório (`openapi.json`) descreve o contrato de comunicação entre o Frontend e o Backend do nosso clone do Instagram.

Este documento tem como objetivo explicar e justificar as decisões arquiteturais adotadas no desenho dessa API.

## 1. Padrão de Resposta Universal

Todas as respostas da API, independentemente de sucesso ou erro, devem seguir a estrutura padronizada abaixo.

```json
{
  "status": "sucesso | erro",
  "codigo": "STRING_IDENTIFICADORA",
  "mensagem": "Mensagem descritiva para o usuário ou desenvolvedor",
  "dados": { ... } // Payload opcional dependendo do endpoint
}
```

### Por que esta escolha? (Compatibilidade Multi-Stack)
Garantir que um único padrão envelopado (Envelope Pattern) trafegue na rede mitiga a dissonância cognitiva no frontend. Como este projeto pretende ser compatível com N stacks (Node+Express, Spring Boot, Laravel, Django, FastAPI), ao invés de depender puramente do HTTP Status Code que cada framework lida de um jeito (ex: Spring joga exceções que viram 500, Express lida nativamente), forçamos os desenvolvedores de Backend a moldarem seus `Exception Handlers` para cuspir esse contrato json uniforme. No Frontend, o Axios interage apenas com `res.data.status` e `res.data.codigo` para tomar decisões.

## 2. Nomenclatura de Recursos (Resource Naming)

A API segue estritamente os princípios do REST (Representational State Transfer), modelando as URLs em torno de **Substantivos Plurais**, evitando verbos na URL.

- **Correto:** `GET /usuarios`, `POST /postagens`
- **Incorreto:** `GET /pegarUsuarios`, `POST /criarPost`

### Sub-recursos e Relacionamentos
Para ações que envolvem relacionamento direto, utilizamos o padrão de aninhamento de rotas (Nested Routes) até 1 ou 2 níveis para manter clareza.

- `GET /usuarios/{id}/seguidores` (Recurso: Seguidores do Usuário)
- `POST /postagens/{id}/curtidas` (Criar um vínculo de curtida atrelada ao post)
- `DELETE /postagens/{post_id}/comentarios/{comentario_id}` (Remover recurso específico de uma coleção pai)

## 3. Uso Correto de Verbos HTTP

- **GET:** Para busca e recuperação de dados. NUNCA altera estado do banco. (Ex: `GET /postagens/feed`).
- **POST:** Para criação de novos recursos. Retorna `201 Created` quando aplicável. (Ex: Cadastro, Postar foto).
- **PATCH:** Para atualizações parciais. (Ex: `PATCH /usuarios/me` para alterar apenas a bio sem enviar a senha). *Nota: Preferimos `PATCH` sobre `PUT` porque `PUT` semanticamente exige a substituição total do recurso, o que no contexto de edição de perfil de Instagram quase nunca ocorre simultaneamente.*
- **DELETE:** Remoção ou desativação de recursos. (Ex: `DELETE /usuarios/me` -> Soft Delete, `DELETE /usuarios/{id}/seguir` -> Unfollow).

## 4. Gestão de Imagens (Uploads)

A criação de postagens com imagem (`POST /postagens`) foi projetada utilizando `multipart/form-data` ao invés de JSON padrão.

**Por que?**
O RNF03 e RNF06 ditam restrições estritas de tamanho (5MB a 10MB) e formato (JPG, PNG). Trafegar imagens pesadas em Base64 dentro de payloads JSON causa overhead de aproximadamente 33% no peso do arquivo e estrangula o event loop no Node.js ou gasta excessiva RAM na JVM do Spring Boot ao parsear o JSON. O multipart permite o uso otimizado de *Streams* nas diferentes linguagens de Backend.

## 5. Padrão de Autenticação (Bearer JWT)

Conforme RNF02 e RNF04, toda a proteção de rotas usa `Authorization: Bearer <TOKEN>`.
A Rota de Login foi separada da entidade puramente CRUD de Usuários: usamos `POST /auth/login`. Isso é uma convenção de mercado (separar "Recursos" de "Processos de Segurança").

## 6. Mapeamento de Regras de Negócio no Contrato

O arquivo JSON do Swagger foi refatorado não apenas para descrever endpoints, mas para "vazar" as validações do backend (schema definitions) diretamente no contrato, auxiliando geradores de código.
- `minLength: 3`, `maxLength: 60` no atributo `nome_completo`.
- Padrões Regex `^[a-z0-9_]+$` para nome de usuário embutidos no OpenAPI.

Assim, independentemente da stack escolhida, basta seguir fielmente a especificação deste arquivo YAML/JSON para estar compatível com os Requisitos Funcionais do Frontend construído.
