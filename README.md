# ☕ Cafeteria Online

## 📘 Visão Geral do Domínio
O sistema modela uma **pequena cafeteria online** com as seguintes entidades principais:

- **Usuario** — representa o cliente/usuário autenticável (implementa `UserDetails` para integração com Spring Security).  
- **Produto** — itens disponíveis para venda (nome, descrição, preço, imagem).  
- **Pedido** — pedido realizado por um `Usuario` (possui status, data e total).  
- **ItemPedido** — associação entre `Pedido` e `Produto`, contendo quantidade e subtotal.  
- **StatusPedido (enum)** — possíveis estados: `PENDENTE`, `APROVADO`, `CANCELADO`.  

### 🎯 Justificativa do Modelo
O modelo separa os conceitos de **catálogo (Produto)** e **vendas (Pedido / ItemPedido)**, o que permite:
- Manter histórico completo de pedidos.  
- Calcular totais por item e por pedido.  
- Rastrear status de cada compra.  
- Integrar autenticação e autorização via `Usuario`.

---

## 🧩 Diagrama Conceitual / Lógico
O modelo a seguir representa o relacionamento entre as entidades principais:

![Diagrama Lógico](./assets/diagrama.png)

---

## 🗂️ Descrição Textual das Relações

### 🔗 Relacionamentos

- **Usuario 1..* Pedido** — um usuário pode realizar vários pedidos; cada pedido pertence a um único usuário.  
- **Pedido 1..* ItemPedido** — um pedido contém um ou mais itens.  
- **ItemPedido *..1 Produto** — cada item faz referência a um produto, capturando o preço atual no momento da criação do pedido.  

---

## ⚙️ Operações Principais (Serviços Implementados)

### 🛍️ ProdutoService
- `salvarProduto(Produto, MultipartFile)` — cria/atualiza produto e armazena imagem.  
- `listarTodos()` — retorna todos os produtos disponíveis.  
- `buscarPorId(Long)` — busca produto por ID ou lança exceção.  
- `atualizarProduto(Long, Produto, MultipartFile)` — atualiza campos e imagem.  
- `excluirProduto(Long)` — remove produto existente.  
- `buscarPorNome(String)` — pesquisa produtos pelo nome.  
- `contarProdutos()` — retorna o total de produtos cadastrados.  

---

### 🧾 CarrinhoService
Carrinho em memória representado por:  
`Map<usuarioId, Map<produtoId, quantidade>>`

- `adicionarAoCarrinho(usuarioId, produtoId, quantidade)` — adiciona ou incrementa item.  
- `removerDoCarrinho(usuarioId, produtoId)` — remove item do carrinho.  
- `atualizarQuantidade(usuarioId, produtoId, quantidade)` — altera quantidade (>0).  
- `getCarrinho(usuarioId)` — converte IDs em objetos `Produto` via `ProdutoService`.  
- `calcularTotal(usuarioId)` — soma (preço × quantidade).  
- `limparCarrinho(usuarioId)` — limpa carrinho do usuário.  
- `getQuantidadeItens(usuarioId)` — retorna quantidade total de itens.  
- `finalizarPedido(Usuario)` — cria `Pedido` com itens do carrinho, calcula total, salva no repositório e limpa o carrinho. Define status `PENDENTE`.

---

### 📦 PedidoService
- `listarPedidosPorUsuario(Usuario)` — retorna pedidos de um usuário específico.  
- `listarTodosPedidos()` — retorna todos os pedidos (admin).  
- `listarPedidosPorStatus(StatusPedido)` — filtra pedidos por status.  
- `buscarPorId(Long)` — busca pedido por ID.  
- `atualizarStatus(Long, StatusPedido)` — altera status do pedido.  
- `excluirPedido(Long)` — remove pedido.  

---

### 👤 CustomUserDetailsService
- `loadUserByUsername(String)` — carrega `Usuario` por e-mail, integrando com autenticação Spring Security.

---

## 🧠 Regras Importantes

- O **carrinho atual é mantido em memória** (`Map<Long, Map<Long,Integer>>`).  
  - Em ambiente de produção, recomenda-se usar **Redis** ou **sessão persistente**.  
- Ao finalizar o pedido:
  - Os preços são copiados do `Produto` no momento da criação do `ItemPedido`.  
  - O total é calculado e armazenado no `Pedido`.  
- Validações básicas:
  - `buscarPorId` lança `RuntimeException` se o recurso não for encontrado.  
  - `finalizarPedido` lança exceção se o carrinho estiver vazio.

---

## 🧪 Exemplos de Uso / Chamadas de API

Assume autenticação básica via e-mail/senha.  
Ajuste as URLs conforme os controladores reais.

### 🔍 Listar produtos
```bash
curl -sS -u user@example.com:senha   -X GET "http://localhost:8080/api/produtos"
```

### ➕ Criar produto (multipart com imagem)
```bash
curl -sS -u admin@example.com:senha   -X POST "http://localhost:8080/api/produtos"   -F "nome=Café Espresso"   -F "descricao=Dose única"   -F "preco=7.50"   -F "imagem=@/caminho/para/arquivo.jpg"
```

### 🛒 Adicionar item ao carrinho
```bash
curl -sS -u user@example.com:senha   -X POST "http://localhost:8080/api/carrinho/123/adicionar"   -H "Content-Type: application/json"   -d '{"produtoId":45,"quantidade":2}'
```

### 🧾 Visualizar carrinho
```bash
curl -sS -u user@example.com:senha   -X GET "http://localhost:8080/api/carrinho/123"
```

### ✅ Finalizar pedido
```bash
curl -sS -u user@example.com:senha   -X POST "http://localhost:8080/api/pedidos/finalizar"
```

### 📜 Listar pedidos do usuário autenticado
```bash
curl -sS -u user@example.com:senha   -X GET "http://localhost:8080/api/pedidos/meus"
```

### 🔄 Atualizar status de um pedido (admin)
```bash
curl -sS -u admin@example.com:senha   -X PUT "http://localhost:8080/api/pedidos/45/status"   -H "Content-Type: application/json"   -d '{"status":"APROVADO"}'
```

---

## 🧰 Tecnologias Utilizadas
- Java 21+
- Spring Boot  
- Spring Data JPA  
- Spring Security  
- Maven  
- MySQL  
- Lombok  

---

## 👨‍💻 Autores
- Nicolas Oliveira Nascimento  
- Paulo Eduardo Messias Grispan  
- Allan Ribeiro de Souza  
- Wallace Araújo da Silva  
- Arthur Vitalino Santos  
- Micael Cadete da Silva Cosme  
