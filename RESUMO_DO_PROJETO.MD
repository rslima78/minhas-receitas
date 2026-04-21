# Resumo do Projeto: Minhas Receitas (receitasBOB)

Este documento serve como um guia técnico detalhando a estrutura, as decisões arquiteturais e o funcionamento do aplicativo "Minhas Receitas". É leitura obrigatória para obter o contexto necessário antes de adicionar ou modificar qualquer funcionalidade do projeto.

---

## 1. Visão Geral

"Minhas Receitas" é um **Single Page Application (SPA)** projetado para gerenciar e visualizar receitas culinárias. 
Seu principal diferencial é a ausência total de dependências externas (Zero-Dependency), o que significa que não utiliza frameworks de JavaScript (React, Vue, etc.) ou bibliotecas externas. Todo o aplicativo é escrito em Vanilla HTML, CSS e JavaScript, compactado primariamente em um único arquivo.

**Características Principais:**
- **Mobile-first:** A interface de usuário é projetada com foco em dispositivos móveis (com largura máxima fixada em 480px em telas maiores e centralizada).
- **Offline-capable e Local-first:** A aplicação armazena todas as alterações ativas (novas receitas editadas, modificações, e itens marcados da lista de compras/passos) no `localStorage` do navegador, permitindo total funcionalidade offline ininterrupta após o carregamento inicial.
- **Sincronização Server-Side / Import & Export:** A aplicação provê mecanismos visuais (via menu de 3 pontos) para puxar alterações do servidor onde está hospedado (o arquivo `receitas.md`), ou para exportar as receitas atuais de volta para um arquivo `.md`.

---

## 2. Estrutura de Arquivos

O projeto é incrivelmente simples e encapsulado, consistindo de apenas dois arquivos vitais:

1. **`index.html`**
   - É o "coração" da aplicação. Contém **toda** a marcação HTML, todos os estilos embutidos via tag `<style>` e toda a lógica de estado e interação via tag `<script>`.
2. **`receitas.md`**
   - É a **fonte de verdade (Canonical Data Source)** do servidor. Contém todas as receitas formatadas em Markdown. Utilizado quando o aplicativo inicializa (e o usuário não possui dados prévios) ou quando o usuário dispara a ação "Recarregar receitas do servidor" manualmente no UI.

---

## 3. Lógica de Funcionamento e Dados

### 3.1. Estado Em Memória & LocalStorage
Os dados de estado são gerenciados por variáveis globais simples no escopo do JavaScript e persistentemente salvos através do Local Storage.
- `receitas_data`: Chave no localStorage que guarda um array de objetos das receitas (incluindo as criadas pelo próprio usuário).
- `recipe_{id}_ingredients` / `recipe_{id}_steps`: Chaves geradas dinamicamente no localStorage contendo os estados das checkboxes de cada passo/ingrediente. (ex: para saber o que já foi mercado ou comprado).

### 3.2. Fluxo de Inicialização de Dados (`initApp()`)
O aplicativo utiliza uma lógica resiliente e progressiva para carregar os dados ao ser aberto:
1. **Verificação de Cache Local:** Primeiro tenta carregar dados do `localStorage` (`receitas_data`). Se encotrar e for válido, exibe as receitas guardadas na memória do navegador.
2. **Busca ao Servidor:** Se o cache estiver vazio, dispara um `fetch("./receitas.md")`. Se obtiver sucesso e o arquivo apresentar receitas processadas em Markdown válido, popula toda a base local e renderiza.
3. **Fallback Hardcoded:** Se tudo falhar (ex: rodar a aplicação localmente pelo sistema de arquivos, sem servidor estático local/CORS e com cache zerado), ele injeta uma constante `SEED_RECEITAS` injetada no próprio script.

---

## 4. Funcionalidades Principais (Funcional e Visual)

- **Menu Principal (Home App):** 
  - Listagem da biblioteca de receitas do usuário, busca visual via texto, botão fixo estilo Material Design para nova receita. 
  - O design incorpora funcionalidade de Imagem de Capa, permitindo cadastrar na edição do item (ou via MD) uma URL de imagem que é centralizada perfeitamente; com fallback nativo inteligente para renderizar apenas o Emoji caso a imagem não exista.
  - Menu Dropdown para Exportar Arquivos MD, Importar arquivo MD local, ou Sincronizar via fetch com Servidor.
- **Tela de Detalhes:** 
  - Abas (Tabs) persistentes (Ingredientes e Modo de preparo).
  - Listas Clicáveis (Checklists dinâmicas via LocalStorage).
  - Temporizador (Timer) embutido no topo da tela de preparo (Faz apitos programáveis de término via Web Audio API, sem mp3 ou som estático gravado).
  - Copiar para Área de Transferência da "lista de itens faltando" dos ingredientes de uma receita com formatação.
- **Tela de Edição (CRUD):**
  - Permite alterar Emojis (representação visual), Nome, e Arrays de Passos/Ingredientes com capacidade de mover pra cima/trás e deleção de passos e total da receita.

---

## 5. Como Adicionar ou Modificar Funcionalidades

### Injeção de Estilos/CSS:
- Qualquer alteração de Design deve ser feita na tag `<style>` do cabeçalho do `index.html`.
- Variáveis CSS nativas são usadas no topo (`:root`) para o esquema de cor `(--bg, --primary, --secondary)`, modificar estas variáveis mudará as cores chave do app todo.

### Injeção de Estrutura Visual (HTML):
- O HTML é separado em blocos grandes baseados em `display: block/none` controlados via Javascript. Temos `home-screen`, `detail-screen` e `edit-screen`. Tudo deve ser mapeado em um painel correspondente oculto inicialmente.

### Injeção de Scripts (Lidando com os Dados):
- Modificações aos dados: Todo dado processado passa inevitavelmente pela função `saveReceitas(arr)`.
- Lidando com estado visual: O aplicativo sempre modifica o DOM manualmente `document.getElementById(...).innerHTML`  ao receber/atualizar dados. Isso significa que após uma refatoração da lógica ou criação/modificação de dados visualizáveis, você deverá repopular ativamente ou chamar renderizadores, ex: `renderHomeGrid()`, `renderRecipeDetail(id)`, ou `renderEditForm(...)`.

### Manipulação do `receitas.md`
- Este arquivo usa um formato *parser* não-trivial no JS. Segue regramento exigente: 
  - O formato exige cabeçalhos nível 2 como "## [EMOJI_SE_EXISTIR] NOME_DA_RECEITA".
  - (Opcional) A URL da imagem de capa, se existir, é lida e escrita baseada na primeira marcação de imagem markdown válida logo após o título principal da receita, no formato clássico: `![Capa](https://...)`
  - Seções marcadas obrigatoriamente como "### Ingredientes" (Usa listas de Markdown `- xxxx`) e "### Passos" (Usam `1. xxxxx`). Alterar bruscamente palavras-chave neste MD, quebrará o app durante a leitura (Parser: `parseMarkdownRecipes(md)`).
