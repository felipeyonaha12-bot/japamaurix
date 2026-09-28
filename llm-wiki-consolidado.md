# LLM Wiki: Bases de Conhecimento Mantidas por LLMs

> Consolidação do tweet de Andrej Karpathy ("LLM Knowledge Bases"), do *idea file* "LLM Wiki" e do diagrama de arquitetura.

---

## 1. A ideia central

A maioria das pessoas usa LLMs com documentos no modelo **RAG**: você envia arquivos, o LLM recupera trechos na hora da pergunta e gera uma resposta. Funciona, mas **nada se acumula**. A cada pergunta o conhecimento é redescoberto do zero. NotebookLM, uploads no ChatGPT e a maioria dos sistemas RAG funcionam assim.

A proposta é outra: o LLM **constrói e mantém incrementalmente uma wiki persistente**, uma coleção interligada de arquivos Markdown que fica entre você e as fontes brutas. Quando uma nova fonte chega, o LLM:

- lê e extrai o que importa;
- atualiza páginas de entidades e conceitos;
- revisa resumos de tópicos;
- sinaliza onde dados novos contradizem afirmações antigas;
- fortalece ou questiona a síntese em evolução.

O conhecimento é **compilado uma vez e mantido atualizado**. As referências cruzadas já existem, as contradições já foram marcadas e a síntese já reflete tudo o que foi lido. **A wiki é um artefato cumulativo.**

**Divisão de papéis:**

| Humano | LLM |
|---|---|
| Curar as fontes | Resumir |
| Direcionar a exploração | Criar referências cruzadas |
| Fazer boas perguntas | Arquivar e organizar |
| Pensar no significado | Toda a manutenção |

Você raramente edita a wiki: ela é domínio do LLM.

> **Obsidian é a IDE. O LLM é o programador. A wiki é a base de código.**

---

## 2. Arquitetura

### 2.1 As três camadas

1. **Fontes brutas (`raw/`)**: artigos, papers, repositórios, datasets e imagens. São **imutáveis**: o LLM lê, mas nunca modifica. É a fonte da verdade.
2. **A wiki (`wiki/`)**: Markdown gerado pelo LLM, com resumos, páginas de entidades e de conceitos, comparações, visão geral e sínteses. O LLM é dono total desta camada.
3. **O schema (`CLAUDE.md` / `AGENTS.md`)**: define estrutura, convenções e workflows (ingestão, consulta, manutenção). É o que transforma o LLM em um **mantenedor disciplinado**, e não em um chatbot genérico. Evolui junto com o uso.

### 2.2 Fluxo completo

```
Fontes ──► Web Clipper ──► raw/ ──► Motor LLM ──► wiki/ ──► Saídas
                                    (compile,               (md, Marp,
                                     Q&A, lint,              gráficos)
                                     indexação)                 │
                                        ▲                       │
                                        └──── arquivar de volta ◄┘
                                               ("explorations add up")
```

| Bloco | Componentes | Função |
|---|---|---|
| **Ingestão** | Articles, Papers, Repos, Datasets, Images + Obsidian Web Clipper | Capturar fontes em Markdown |
| **Armazenamento bruto** | `raw/` (+ `raw/assets/` para imagens) | Documentos originais, imutáveis |
| **Ferramentas extras** | Search (Web UI + CLI), CLI tools | Busca e processamento determinístico |
| **Motor LLM** | Compile, Q&A, Linting, Indexing | Operações sobre a wiki |
| **Knowledge Store** | `wiki/` (.md): artigos, conceitos, categorias, backlinks | Estado canônico do conhecimento |
| **Saídas** | Markdown, slides Marp, gráficos (matplotlib) | Respostas renderizadas |
| **Feedback loop** | *Filed back → wiki* | Explorações voltam para a base |
| **Frontend** | Obsidian: raw, wiki, visualizações, slides | Leitura e navegação humana |
| **Futuro** | Dados sintéticos + fine-tuning; visão de produto | Próximos horizontes |

**Escala de referência (Karpathy):** cerca de 100 artigos e 400 mil palavras, ainda sem precisar de RAG.

---

## 3. Operações

### 3.1 Ingestão (Compile: raw → wiki)

1. Você coloca uma fonte em `raw/` e pede ao LLM para processá-la.
2. O LLM lê a fonte e discute os principais aprendizados com você.
3. Escreve uma página de resumo na wiki.
4. Atualiza `index.md`.
5. Atualiza as páginas de entidades e conceitos relacionadas (uma fonte costuma tocar **10 a 15 páginas**).
6. Acrescenta uma entrada em `log.md`.

**Modos:** uma fonte por vez, com supervisão (preferência de Karpathy), ou em lote, com menos supervisão. O workflow escolhido deve ser documentado no schema.

**Imagens:** o LLM não lê Markdown com imagens inline em uma só passada. Ele deve ler primeiro o texto e depois visualizar as imagens referenciadas separadamente.

### 3.2 Consulta (Q&A)

1. O LLM lê `index.md` para achar as páginas relevantes.
2. Lê essas páginas.
3. Sintetiza uma resposta **com citações**.

**Formatos de saída:** página Markdown, tabela comparativa, slides Marp, gráfico matplotlib, canvas.

> **Insight-chave:** boas respostas devem ser **arquivadas de volta na wiki** como novas páginas. Comparações, análises e conexões descobertas não podem morrer no histórico do chat. Assim suas explorações se acumulam como as fontes.

### 3.3 Lint (health check)

Periodicamente, o LLM audita a wiki em busca de:

- contradições entre páginas;
- afirmações desatualizadas, superadas por fontes mais novas;
- páginas órfãs (sem links de entrada);
- conceitos mencionados sem página própria;
- referências cruzadas ausentes;
- lacunas de dados preenchíveis via busca na web;
- conexões interessantes que renderiam novos artigos.

O LLM também sugere **novas perguntas** e **novas fontes** a investigar.

### 3.4 Indexação

O LLM mantém resumos, índices, mapas de conteúdo e backlinks. Na escala de ~100 fontes e centenas de páginas, isso substitui uma infraestrutura de embeddings.

---

## 4. Arquivos especiais

### `index.md`: orientado a conteúdo

- Catálogo de todas as páginas: link, resumo de uma linha e metadados opcionais (data, número de fontes).
- Organizado por categoria (entidades, conceitos, fontes etc.).
- Atualizado a **cada ingestão**.
- É o **primeiro arquivo lido** em toda consulta.

### `log.md`: cronológico

- Registro *append-only* de ingestões, consultas e lints.
- Prefixo consistente para ser processável com ferramentas Unix:

```markdown
## [2026-04-02] ingest | Título do Artigo
```

```bash
grep "^## \[" log.md | tail -5   # últimas 5 entradas
```

---

## 5. Ferramentas

- **Busca:** em escala pequena, o `index.md` basta. Quando a wiki cresce:
  - [qmd](https://github.com/tobi/qmd): busca local em Markdown, híbrida BM25/vetorial com reranking por LLM, via CLI e servidor MCP;
  - ou um script simples feito pelo próprio LLM, usado por uma web UI ou, de preferência, pela CLI do agente.
- **CLI tools:** scripts de conversão e extração que o LLM chama pelo shell.

---

## 6. Dicas com o Obsidian

- **Web Clipper:** converte artigos da web em Markdown direto para `raw/`.
- **Imagens locais:**
  - *Settings → Files and links → Attachment folder path* = `raw/assets/`
  - *Settings → Hotkeys* → "Download attachments for current file" → ex.: `Ctrl+Shift+D`
  - Após capturar um artigo, o atalho baixa todas as imagens, evitando URLs quebradas.
- **Graph View:** mostra hubs, conexões e páginas órfãs.
- **Marp:** slides em Markdown gerados a partir da wiki.
- **Dataview:** consultas sobre o frontmatter YAML (tags, datas, número de fontes) geram tabelas dinâmicas.
- **Git:** a wiki é um repositório de Markdown, com histórico, branches e colaboração de graça.

---

## 7. Aplicações

- **Pessoal:** metas, saúde, psicologia, diário, notas de podcasts.
- **Pesquisa:** semanas ou meses de papers e relatórios, com uma tese em evolução.
- **Leitura de livro:** personagens, temas e tramas capítulo a capítulo (algo como um *Tolkien Gateway* pessoal).
- **Empresas/equipes:** wiki interna alimentada por Slack, reuniões, documentos e chamadas, com revisão humana opcional.
- **Outros:** análise competitiva, due diligence, viagens, cursos, hobbies.

---

## 8. Explorações futuras

- **Dados sintéticos + fine-tuning:** fazer o modelo "conhecer" a wiki nos pesos, não só na janela de contexto.
- **Produto:** passar de "uma coleção improvisada de scripts" para um produto coeso de gestão de conhecimento.

---

## 9. Por que funciona

O difícil em uma base de conhecimento não é ler nem pensar: é a **manutenção**. Atualizar referências cruzadas, manter resumos em dia, registrar contradições e manter a consistência entre dezenas de páginas. Humanos abandonam wikis porque o custo de manutenção cresce mais rápido que o valor.

LLMs não se entediam, não esquecem uma referência cruzada e editam 15 arquivos em uma passada. **O custo de manutenção tende a zero.**

Isso ecoa o **Memex** de Vannevar Bush (1945): um acervo pessoal, curado ativamente, em que as conexões valem tanto quanto os documentos. Bush não resolveu quem faria a manutenção. O LLM resolve.

---

## 10. Observação

Este padrão é **intencionalmente abstrato e modular**. A estrutura de pastas, as convenções do schema, os formatos de página e as ferramentas dependem do seu domínio, das suas preferências e do LLM usado. Use o que servir e ignore o resto: sem imagens, dispense o tratamento de imagens; com uma wiki pequena, dispense a busca; sem interesse em slides, fique só no Markdown.

> **TL;DR:** fontes brutas vão para `raw/` → o LLM as compila em uma wiki `.md` → consultas e lints por CLI enriquecem a wiki continuamente → tudo é visualizado no Obsidian. Você quase nunca edita a wiki: ela pertence ao LLM.
