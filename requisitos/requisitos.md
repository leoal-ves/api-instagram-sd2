# Requisitos Funcionais e Não Funcionais - API Instagram

Este documento formaliza as capacidades do sistema e restrições arquiteturais para o clone da API do Instagram.

## Requisitos Funcionais (RF)

O que o sistema deve fazer e quais funcionalidades deve oferecer.

- **RF01 - Cadastro e Desativação:** O sistema deve permitir cadastro de novos usuários. Campos obrigatórios: Nome, Usuário (@), Senha, E-mail. O sistema deve permitir que a conta seja desativada (soft delete), mas não excluída fisicamente do banco de dados.
- **RF02 - Login e Autenticação:** O sistema deve aceitar o login mediante preenchimento obrigatório de "usuário" e "senha", validando as credenciais. A autenticação utilizará Token JWT enviado via cabeçalho (Bearer Token), contendo internamente a data de emissão/expiração (`iat` / `exp`).
- **RF03 - Logout:** O sistema deve permitir que o usuário encerre sua sessão (logout).
- **RF04 - Seguir/Deixar de Seguir:** O usuário autenticado deve poder seguir e deixar de seguir outros usuários.
- **RF05 - Restrição de Auto-Seguir:** Um usuário não pode seguir a si mesmo.
- **RF06 - Contadores do Perfil:** O perfil de cada usuário deve exibir estatísticas quantitativas: número de seguidores e de usuários que ele segue.
- **RF07 - Feed Restrito:** O feed deve exibir exclusivamente as postagens dos usuários que o usuário autenticado segue, em ordem cronológica reversa.
- **RF08 - Listagem de Conexões:** O usuário deve poder visualizar a lista completa de seus seguidores e das pessoas que ele segue.
- **RF09 - Edição de Perfil:** O usuário deve poder editar suas informações de perfil: Nome completo, Nome de usuário, Foto de perfil e Biografia (máximo 150 caracteres).
- **RF10 - Visualização de Perfil:** O usuário deve poder acessar e visualizar o perfil público de outros usuários.

---

## Requisitos Não Funcionais (RNF)

Como o sistema deve operar e quais restrições tecnológicas deve respeitar.

### Comunicação e Segurança
- **RNF01 - API REST via HTTP:** A comunicação Cliente-Servidor ocorrerá exclusivamente via troca de mensagens JSON sobre protocolo HTTP(S), seguindo arquitetura REST.
- **RNF04 - Formato do Token:** A autenticação exigirá o envio do token JWT no cabeçalho HTTP da requisição (`Authorization: Bearer <token>`).

### Validação de Campos de Usuário
- **RNF02 - Regras do Formulário de Cadastro:** Todos os campos são obrigatórios.
  - **Nome Completo:** Entre 3 e 60 caracteres. Apenas letras e espaços (proibido números ou caracteres especiais).
  - **Nome de Usuário:** Entre 3 e 30 caracteres. Apenas letras minúsculas, números e underline (`_`). Proibido letras maiúsculas, espaços e caracteres especiais.
  - **E-mail:** Formato válido (ex: `xxx@xxxx.com`) e tamanho entre 10 e 35 caracteres.
  - **Senha:** Entre 8 e 24 caracteres. Permitidos apenas letras e números. Visualização oculta (`****`).
  - **Biografia:** Máximo de 150 caracteres.
  - **Foto de Perfil:** Decisão técnica: será salva no servidor via `multipart/form-data` ou referenciada por URL externa, dependendo da implementação em etapas futuras.

### Validação de Postagens (Fotos)
- **RNF03 - Upload de Novas Fotos:** É obrigatório o envio de 1 (uma) imagem (tamanho máximo: 5MB) exclusivamente no formato `.JPG`. Opcionalmente, pode ser enviada uma legenda de até 50 caracteres neste fluxo. (Nota: RNF05 estipula regras de legenda mais amplas, que se aplicam em sobreposição a depender do contexto de validação do domínio).
- **RNF05 - Restrições de Legenda:** A legenda deve conter entre 5 e 200 caracteres, aceitando letras e espaços (proibido acentos e caracteres especiais).
- **RNF06 - Formatos e Limites Ampliados:** Em contextos gerais de sistema, os formatos de imagem aceitos devem ser `JPG`, `JPEG` e `PNG`, com tamanho restrito ao limite máximo absoluto de 10MB.
- **RNF07 - Edição Parcial:** O usuário pode editar apenas o texto da legenda de uma postagem, nunca a imagem.
- **RNF08 - Exclusão Integral:** O usuário pode excluir a postagem inteira, mas não pode deletar isoladamente a foto mantendo a legenda (ou vice-versa).

### Curtidas
- **RNF09 - Limite de Curtida:** Um usuário pode curtir ou descurtir uma postagem no máximo 1 vez por registro (toggle on/off).
- **RNF10 - Totalização:** O sistema fornecerá no payload a contagem total de curtidas por postagem.
- **RNF11 - Bloqueio de Acesso:** Usuários não-autenticados não podem curtir publicações.

### Comentários
- **RNF12 - Inserção de Comentários:** Usuários autenticados podem comentar em postagens. Limite de tamanho: entre 1 e 300 caracteres.
- **RNF13 - Exclusão Própria:** Um usuário pode deletar os comentários que ele mesmo escreveu.
- **RNF14 - Moderação pelo Dono do Post:** O proprietário de uma publicação tem permissão para excluir qualquer comentário feito em seu próprio post, independentemente de quem seja o autor original.
