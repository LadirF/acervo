# PORTAR.md: o que levar do v3 para o arquivo-mestre

Este arquivo lista tudo o que mudou no `index.html` da branch `nuvem` (comparado com o commit `5bd3758`, o início da branch), para levar à mão ao arquivo-mestre. Os trechos abaixo foram copiados do `index.html` do commit `6245480`; se o arquivo mudar depois, vale o que estiver nele.

## Resumo

O conteúdo das decisões (`<li class="entry">`, ementas, PDFs, `.md`) não mudou. Tudo o que é novo está em:

1. **Seis ganchos no script principal** (seção A): umas poucas linhas, todas marcadas com `v3` nos comentários. Sem o bloco `recursos-v3`, os ganchos ficam vazios e a página funciona como antes.
2. **Dois blocos inteiros** (seção B): `<style id="design-v3">` e `<script id="recursos-v3">`.
3. **Marcação nova fora das decisões** (seção C): painel de busca e filtros, estado vazio, aviso, botão "Voltar ao topo" e leitor de PDF.
4. **Dependências de atributos e de nomes** (seção D), que o gerador precisa continuar respeitando.

### Ordem sugerida para levar ao arquivo-mestre

1. Aplicar os seis ganchos da seção A no script principal.
2. Colar o bloco `<style id="design-v3">` logo depois do `</style>` do `<style id="design-v2">`, antes de `</head>`.
3. Trocar a marcação da seção C (painel, estado vazio, fim da `div.wrap`, leitor de PDF).
4. Colar o bloco `<script id="recursos-v3">` logo depois do `</script>` do script principal (o que termina em `lerEndereco();` e `})();`), antes do comentário `<!-- PDF.js 3.11.174`.
5. Gerar o HTML e conferir a lista da seção E (principalmente o leitor de PDF no HTML do Drive, com os PDFs embutidos).

> **Atenção ao copiar o `recursos-v3`:** algumas expressões regulares usam sequências como `\u0300`, `\u00a0` e `\u2028`, escritas com a barra invertida. Elas precisam continuar assim, como texto. Se alguma ferramenta transformar `\u2028` no caractere de verdade, o bloco inteiro deixa de funcionar (o JavaScript entende esse caractere como quebra de linha).

---

## A. Ganchos no script principal

São as únicas mudanças em funções que já existiam. Em cada item: onde entra, o trecho **antes** (como está no arquivo-mestre hoje) e o trecho **depois**, com algumas linhas de contexto.

### 1. Objeto `ganchos`, logo depois de `var state`

**Onde:** Logo depois da linha `var state = { search: …, crime: 'all' };`, antes de `function normalize(str)`.

**Antes:**

```js
  var state = { search: '', tribunal: 'all', decisao: 'all', relator: 'all', crime: 'all' };

  function normalize(str) {
```

**Depois:**

```js
  var state = { search: '', tribunal: 'all', decisao: 'all', relator: 'all', crime: 'all' };

  // v3 (19/10/2026): ganchos preenchidos pelo bloco de script "recursos-v3"
  // (busca sem acento, período, salvas, ordem, leitor de PDF com zoom e busca).
  // Sem esse bloco ficam vazios e a página funciona como antes.
  var ganchos = {};

  function normalize(str) {
```

### 2. Em `apply()`: busca, filtro extra e ordem

**Onde:** Dentro do `entries.forEach` de `apply()`: muda a linha de `matchesSearch` e a de `filtros`; logo depois do `forEach` entra a linha do `ganchos.ordenar`, antes de `var visible = visiveis.length;`.

**Antes:**

```js
    var total = 0;
    var term = normalize(state.search.trim());
    entries.forEach(function (li) {
      var texto = (li.getAttribute('data-search') || '') + ' ' + normalize(li.getAttribute('data-crimes') || '');
      var matchesSearch = term === '' || texto.indexOf(term) !== -1;
      var matchesTribunal = state.tribunal === 'all' || grupoTribunal(li.getAttribute('data-tribunal')) === state.tribunal;
      var matchesDecisao = state.decisao === 'all' || li.getAttribute('data-decisao') === state.decisao;
      var matchesRelator = state.relator === 'all' || li.getAttribute('data-relator') === state.relator;
      var matchesCrime = state.crime === 'all' || crimesDe(li).indexOf(state.crime) !== -1;
      var filtros = matchesTribunal && matchesDecisao && matchesRelator && matchesCrime;
      if (filtros) total++;
      if (matchesSearch && filtros) visiveis.push(li);
    });
    var visible = visiveis.length;
```

**Depois:**

```js
    var total = 0;
    var term = normalize(state.search.trim());
    entries.forEach(function (li) {
      var texto = (li.getAttribute('data-search') || '') + ' ' + normalize(li.getAttribute('data-crimes') || '');
      var matchesSearch = term === '' || (ganchos.busca ? ganchos.busca(li) : texto.indexOf(term) !== -1);
      var matchesTribunal = state.tribunal === 'all' || grupoTribunal(li.getAttribute('data-tribunal')) === state.tribunal;
      var matchesDecisao = state.decisao === 'all' || li.getAttribute('data-decisao') === state.decisao;
      var matchesRelator = state.relator === 'all' || li.getAttribute('data-relator') === state.relator;
      var matchesCrime = state.crime === 'all' || crimesDe(li).indexOf(state.crime) !== -1;
      var filtros = matchesTribunal && matchesDecisao && matchesRelator && matchesCrime && (!ganchos.filtro || ganchos.filtro(li));
      if (filtros) total++;
      if (matchesSearch && filtros) visiveis.push(li);
    });
    if (ganchos.ordenar) visiveis = ganchos.ordenar(visiveis);
    var visible = visiveis.length;
```

### 3. Em `apply()`: "Limpar filtros" e o gancho `depois`

**Onde:** No fim de `apply()`: a condição do botão "Limpar filtros" ganha o `ganchos.filtroAtivo`, e depois de `gravarEndereco();` entra a linha do `ganchos.depois`, antes do `}` que fecha `apply()`.

**Antes:**

```js
    // "Limpar filtros" só aparece com algum filtro ou busca ativa
    document.getElementById('limparFiltros').hidden = !(term !== '' ||
      state.tribunal !== 'all' || state.decisao !== 'all' || state.relator !== 'all' || state.crime !== 'all');
    gravarEndereco();
  }
```

**Depois:**

```js
    // "Limpar filtros" só aparece com algum filtro ou busca ativa
    document.getElementById('limparFiltros').hidden = !(term !== '' ||
      state.tribunal !== 'all' || state.decisao !== 'all' || state.relator !== 'all' || state.crime !== 'all' ||
      (ganchos.filtroAtivo ? ganchos.filtroAtivo() : false));
    gravarEndereco();
    if (ganchos.depois) ganchos.depois({ visiveis: visiveis, mostradas: visiveis.slice(inicio, inicio + POR_PAGINA), total: total, pagina: pagina, paginas: paginas });
  }
```

### 4. Em `limparFiltros()`: o gancho `limpar`

**Onde:** Primeira linha dentro de `limparFiltros()`, antes de `state.search = '';`.

**Antes:**

```js
  // Esc (no campo de busca ou fora dele) limpa todos os filtros e a busca.
  // Não age quando o leitor de PDF ou uma janelinha de filtro está aberta
  // (nesses casos o Esc só fecha a janela).
  function limparFiltros() {
    state.search = '';
    searchInput.value = '';
```

**Depois:**

```js
  // Esc (no campo de busca ou fora dele) limpa todos os filtros e a busca.
  // Não age quando o leitor de PDF ou uma janelinha de filtro está aberta
  // (nesses casos o Esc só fecha a janela).
  function limparFiltros() {
    if (ganchos.limpar) ganchos.limpar();
    state.search = '';
    searchInput.value = '';
```

### 5. Em `mostrarPdf()`: o leitor novo

**Onde:** Primeiras linhas dentro de `mostrarPdf(bytes, nome)`, antes de `var token = ++renderToken;`. O leitor novo recebe os mesmos bytes que vêm de `obterPdf()` (PDF embutido ou `pdfs/ID.pdf`).

**Antes:**

```js
  function mostrarPdf(bytes, nome) {
    var token = ++renderToken;
    if (observador) {
```

**Depois:**

```js
  function mostrarPdf(bytes, nome) {
    // v3: leitor novo (zoom, busca no documento). Recebe os mesmos bytes de
    // obterPdf(); se o bloco recursos-v3 não existir, segue o desenho abaixo.
    if (ganchos.mostrarPdf) { ganchos.mostrarPdf(bytes, nome); return; }
    var token = ++renderToken;
    if (observador) {
```

### 6. `window.acervo`, logo antes de `lerEndereco();` no fim do script principal

**Onde:** Entre o fim de `function lerEndereco() { … }` e a chamada `lerEndereco();` que fecha o script principal.

**Antes:**

```js
    if (n > 1) { pagina = n; apply(true); }
  }

  lerEndereco();
})();
</script>
```

**Depois:**

```js
    if (n > 1) { pagina = n; apply(true); }
  }

  // v3: acesso do bloco recursos-v3 ao estado e às funções desta página
  window.acervo = {
    state: state,
    entries: entries,
    listas: listas,
    ganchos: ganchos,
    aplicar: apply,
    limpar: limparFiltros,
    irParaPagina: function (n) { pagina = n; apply(true); },
    grupoTribunal: grupoTribunal,
    copiarTexto: copiarTexto,
    avisoBotao: avisoBotao,
    carregarPdfjs: carregarPdfjs,
    obterPdf: obterPdf,
    porPagina: POR_PAGINA
  };

  lerEndereco();
})();
</script>
```

### O que cada gancho faz

| Gancho | Chamado em | Para quê |
|---|---|---|
| `ganchos.busca(li)` | `apply()`, no lugar de `texto.indexOf(term)` | busca sem acento, palavras em qualquer ordem, aspas, ementa |
| `ganchos.filtro(li)` | `apply()`, junto dos quatro filtros | Período e "Só salvas" |
| `ganchos.ordenar(lista)` | `apply()`, antes da paginação | Mais recentes, Mais antigas, Relevância |
| `ganchos.filtroAtivo()` | `apply()`, botão "Limpar filtros" | o botão aparece também com Período ou "Só salvas" |
| `ganchos.depois(info)` | fim de `apply()` | destaque, trecho da ementa, contagens, fichas, endereço etc. |
| `ganchos.limpar()` | começo de `limparFiltros()` (Esc e "Limpar filtros") | zera Período e "Só salvas" |
| `ganchos.mostrarPdf(bytes, nome)` | começo de `mostrarPdf()` | leitor novo (zoom, pinça, busca no documento) |

`window.acervo` dá ao bloco v3 acesso a: `state`, `entries`, `listas`, `ganchos`, `aplicar` (= `apply`), `limpar` (= `limparFiltros`), `irParaPagina(n)`, `grupoTribunal`, `copiarTexto`, `avisoBotao`, `carregarPdfjs`, `obterPdf` e `porPagina`. O leitor novo usa `carregarPdfjs()` (o mesmo carregamento do PDF.js de sempre) e recebe os bytes de `obterPdf()` pelo caminho de sempre (clique em "Abrir PDF" → `obterPdf()` → `abrirLink()` → `mostrarPdf()`); ele nunca busca `pdfs/` por conta própria.

Um comentário do script principal também mudou de texto (sem efeito): o que vem antes de `var ganchos = {};` (já incluído no item 1).

---

## B. Blocos inteiros

### B.1 `<style id="design-v3">` (760 linhas)

**Onde:** logo depois do `</style>` que fecha o `<style id="design-v2">`, antes de `</head>`.

```html
<style id="design-v3">
  /* ===== Design v3 (19/10/2026): mantém o cabeçalho preto e dourado, a Century Gothic,
     o tema claro como padrão e a cor do tribunal na borda dos cartões. Acrescenta o visual
     dos recursos v3 (bloco de script "recursos-v3"). Para voltar ao design v2: apagar
     este bloco, o script recursos-v3 e a marcação listada no PORTAR.md. ===== */

  /* --- Base: dourado de texto com contraste AA no tema claro (o --ouro fica nos detalhes) --- */
  :root {
    --ouro-texto: #806616;
    --toque: 40px;
  }

  :root[data-theme="dark"] { --ouro-texto: #CDB458; }

  .filter-group .label,
  :root[data-theme="dark"] .filter-group .label { color: var(--ouro-texto); }

  select:focus-visible, [tabindex]:focus-visible { outline: 2px solid var(--focus); outline-offset: 2px; }

  /* --- Painel de busca e filtros --- */
  .controls { gap: 16px; }
  .search-field { position: relative; }

  .count-row {
    justify-content: space-between;
    gap: 10px 14px;
    margin-top: 2px;
    padding-top: 14px;
    border-top: 1px solid var(--rule);
  }

  .count-line, .limpar-filtros { margin-top: 0; }
  .count-line { margin-right: auto; }

  .resultados-acoes { display: flex; flex-wrap: wrap; align-items: center; gap: 8px; }
  .ordem-rotulo { font-size: 13px; color: var(--ink-muted); }

  /* Seletores nativos (Ordenar, Período): no iPhone abrem a roda de opções do sistema */
  .campo-v3 {
    -webkit-appearance: none;
            appearance: none;
    min-height: 34px;
    max-width: 100%;
    padding: 5px 34px 5px 12px;
    font: inherit;
    font-size: 13.5px;
    color: var(--ink);
    background: var(--paper-raised) url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%236B6560' stroke-width='2.2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='m6 9 6 6 6-6'/%3E%3C/svg%3E") no-repeat right 11px center / 13px;
    border: 1px solid var(--rule);
    border-radius: 10px;
    box-shadow: var(--shadow-sm);
    cursor: pointer;
    transition: border-color .18s var(--ease), box-shadow .18s var(--ease);
  }

  :root[data-theme="dark"] .campo-v3 {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%23A39C92' stroke-width='2.2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='m6 9 6 6 6-6'/%3E%3C/svg%3E");
  }

  .periodo-filter .campo-v3 { min-width: 170px; min-height: 34px; font-size: 14px; }
  .campo-v3.ativo {
    font-weight: 600;
    color: var(--ativo-ink);
    background-color: var(--ativo-bg);
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%23F4F1EA' stroke-width='2.2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='m6 9 6 6 6-6'/%3E%3C/svg%3E");
    border-color: var(--ativo-bg);
    box-shadow: inset 0 -2px 0 var(--ativo-marca);
  }

  :root[data-theme="dark"] .campo-v3.ativo {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%23111111' stroke-width='2.2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='m6 9 6 6 6-6'/%3E%3C/svg%3E");
  }

  .campo-v3:hover { border-color: var(--rule-strong); }
  .campo-v3:focus-visible { outline: none; border-color: var(--focus); box-shadow: 0 0 0 3px var(--focus-ring); }
  .campo-v3 option { color: var(--ink); background: var(--paper-raised); }

  /* --- Cartões: ações numa linha alinhada (os parágrafos ocupam a largura toda) --- */
  li.entry {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    column-gap: 8px;
  }

  li.entry > p { flex: 0 0 100%; min-width: 0; }
  li.entry > .tema { margin: 0 0 12px; }
  li.entry > .cite { margin: 0 0 6px; }
  li.entry > .meta { margin: 0; }
  li.entry > .crimes { margin: 10px 0 0; }

  li.entry > .pdf-link { margin: 14px 0 0; min-height: 36px; }
  li.entry > .pdf-link.info, li.entry > .pdf-link.copiar { margin-left: 0; }

  li.entry:focus-visible { outline: 2px solid var(--focus); outline-offset: 3px; }

  /* "Copiar referência": mesmo desenho de "Copiar ementa", com ícone de aspas */
  .pdf-link.ref-v3 { font-weight: 500; }

  .pdf-link.ref-v3::before {
    -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M10 7H6a2 2 0 0 0-2 2v3a2 2 0 0 0 2 2h2v1a3 3 0 0 1-3 3M20 7h-4a2 2 0 0 0-2 2v3a2 2 0 0 0 2 2h2v1a3 3 0 0 1-3 3'/%3E%3C/svg%3E");
            mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M10 7H6a2 2 0 0 0-2 2v3a2 2 0 0 0 2 2h2v1a3 3 0 0 1-3 3M20 7h-4a2 2 0 0 0-2 2v3a2 2 0 0 0 2 2h2v1a3 3 0 0 1-3 3'/%3E%3C/svg%3E");
  }

  .pdf-link.ref-v3.copiado { background: var(--case-color, var(--stj)); color: var(--on-accent); }

  /* Ações novas do cartão (link, salvar): à direita da linha de botões */
  .acoes-v3 { display: inline-flex; align-items: center; gap: 4px; margin: 14px 0 0 auto; }

  .acao-v3 {
    display: inline-grid;
    place-items: center;
    width: 36px;
    height: 36px;
    padding: 0;
    color: var(--ink-muted);
    background: transparent;
    border: 1px solid transparent;
    border-radius: 50%;
    cursor: pointer;
    transition: color .18s var(--ease), background-color .18s var(--ease), border-color .18s var(--ease);
  }

  .acao-v3::before {
    content: "";
    width: 18px;
    height: 18px;
    background: currentColor;
    -webkit-mask: var(--icone) center / contain no-repeat;
            mask: var(--icone) center / contain no-repeat;
  }

  .acao-v3:hover { color: var(--ink); background: var(--paper); border-color: var(--rule); }
  .acao-v3:focus-visible { outline: 2px solid var(--focus); outline-offset: 1px; }

  .acao-link { --icone: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M10 13a5 5 0 0 0 7.5.5l3-3a5 5 0 0 0-7-7l-1.7 1.7'/%3E%3Cpath d='M14 11a5 5 0 0 0-7.5-.5l-3 3a5 5 0 0 0 7 7l1.7-1.7'/%3E%3C/svg%3E"); }
  .acao-link.compartilhar { --icone: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M12 3v12M8 7l4-4 4 4'/%3E%3Cpath d='M7 10H6a1 1 0 0 0-1 1v9a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1v-9a1 1 0 0 0-1-1h-1'/%3E%3C/svg%3E"); }

  .acao-salvar { --icone: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2' stroke-linejoin='round'%3E%3Cpath d='M6.5 3.5h11a1 1 0 0 1 1 1v16l-6.5-4-6.5 4v-16a1 1 0 0 1 1-1z'/%3E%3C/svg%3E"); }
  .acao-salvar[aria-pressed="true"] {
    --icone: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='black' stroke='black' stroke-width='2' stroke-linejoin='round'%3E%3Cpath d='M6.5 3.5h11a1 1 0 0 1 1 1v16l-6.5-4-6.5 4v-16a1 1 0 0 1 1-1z'/%3E%3C/svg%3E");
    color: var(--ouro-texto);
  }

  /* Decisão salva: fita dourada no alto do cartão */
  li.entry.salva::after {
    content: "";
    position: absolute;
    top: -1px;
    right: 26px;
    width: 13px;
    height: 18px;
    background: var(--metal);
    -webkit-clip-path: polygon(0 0, 100% 0, 100% 100%, 50% 74%, 0 100%);
            clip-path: polygon(0 0, 100% 0, 100% 100%, 50% 74%, 0 100%);
    pointer-events: none;
  }

  /* "Só salvas (N)" e a barra da lista salva */
  .salvas-toggle {
    display: inline-flex;
    align-items: center;
    gap: 7px;
    min-height: 34px;
    padding: 0 8px 0 12px;
    font: inherit;
    font-size: 13px;
    font-weight: 600;
    color: var(--ink);
    background: var(--paper-raised);
    border: 1px solid var(--rule);
    border-radius: 999px;
    box-shadow: var(--shadow-sm);
    cursor: pointer;
  }

  .salvas-toggle::before {
    content: "";
    width: 13px;
    height: 15px;
    color: var(--ouro-texto);
    background: currentColor;
    -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='black'%3E%3Cpath d='M6.5 2.5h11a1.5 1.5 0 0 1 1.5 1.5v18l-7-4.3L5 22V4a1.5 1.5 0 0 1 1.5-1.5z'/%3E%3C/svg%3E") center / contain no-repeat;
            mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='black'%3E%3Cpath d='M6.5 2.5h11a1.5 1.5 0 0 1 1.5 1.5v18l-7-4.3L5 22V4a1.5 1.5 0 0 1 1.5-1.5z'/%3E%3C/svg%3E") center / contain no-repeat;
  }

  .salvas-toggle:hover { border-color: var(--rule-strong); }
  .salvas-toggle[hidden] { display: none; }
  .salvas-toggle[aria-pressed="true"] { color: var(--ativo-ink); background: var(--ativo-bg); border-color: var(--ativo-bg); box-shadow: inset 0 -2px 0 var(--ativo-marca); }
  .salvas-toggle[aria-pressed="true"]::before { color: var(--ativo-marca); }

  .salvas-qtd {
    display: inline-grid;
    place-items: center;
    min-width: 20px;
    height: 20px;
    padding: 0 6px;
    font-size: 11.5px;
    border-radius: 999px;
    background: color-mix(in srgb, var(--ouro) 20%, transparent);
  }

  .salvas-barra {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 8px 10px;
    margin: 16px 0 0;
    padding: 10px 12px 10px 16px;
    background: var(--paper-raised);
    border: 1px solid var(--rule);
    border-left: 3px solid var(--ouro);
    border-radius: 12px;
    box-shadow: var(--shadow-sm);
  }

  .salvas-barra[hidden] { display: none; }
  .salvas-texto { flex: 1 1 220px; margin: 0; font-size: 14px; color: var(--ink); }

  .salvas-acao {
    min-height: 34px;
    padding: 0 14px;
    font: inherit;
    font-size: 13px;
    font-weight: 600;
    color: var(--ink);
    background: var(--paper);
    border: 1px solid var(--rule);
    border-radius: 999px;
    cursor: pointer;
  }

  .salvas-acao:hover { border-color: var(--rule-strong); }
  .salvas-acao[hidden] { display: none; }

  /* Cartão aberto por link direto (?id=): anel dourado que some sozinho */
  li.entry.em-destaque { animation: destaque-v3 4.5s var(--ease); }
  li.entry.em-destaque:focus { outline: none; }

  @keyframes destaque-v3 {
    0%, 70% { box-shadow: 0 0 0 3px var(--ouro), var(--shadow-md); }
    100%    { box-shadow: var(--shadow-sm); }
  }

  /* Aviso curto ("Link copiado") */
  .aviso-v3 {
    position: fixed;
    left: 50%;
    bottom: calc(20px + env(safe-area-inset-bottom, 0px));
    z-index: 60;
    max-width: calc(100vw - 32px);
    padding: 10px 18px;
    font-size: 14px;
    font-weight: 600;
    color: var(--hero-ink);
    background: var(--hero-bg);
    border: 1px solid rgba(181, 151, 41, 0.55);
    border-radius: 999px;
    box-shadow: var(--shadow-pop);
    opacity: 0;
    transform: translate(-50%, 12px);
    pointer-events: none;
    transition: opacity .2s var(--ease), transform .2s var(--ease);
  }

  .aviso-v3.visivel { opacity: 1; transform: translate(-50%, 0); }

  /* Toque (celular): sem o "pulo" do hover preso depois de tocar */
  @media (hover: none) {
    li.entry:hover { transform: none; box-shadow: var(--shadow-sm); border-color: var(--rule); border-left-color: var(--case-color, var(--rule)); }
    .chips button:hover, .lista-trigger:hover, .pdf-link:hover, .paginacao button:hover,
    .limpar-filtros:hover, .tema-toggle:hover { transform: none; }
  }

  /* Áreas de toque maiores em telas de toque */
  @media (pointer: coarse) {
    .chips button { min-height: var(--toque); padding: 7px 14px; }
    .segmented button { min-height: 36px; }
    .lista-trigger { min-height: 42px; }
    .lista-clear { min-width: 42px; min-height: 42px; }
    .lista-opcoes li { min-height: 44px; }
    .paginacao button { min-height: 42px; min-width: 42px; }
    .limpar-filtros { min-height: 36px; }
    .crimes button { min-height: 28px; padding: 2px 10px; font-size: 12px; text-align: left; }
    li.entry > .pdf-link { min-height: 40px; padding: 7px 15px; font-size: 14px; }
    .tema-toggle { width: var(--toque); height: var(--toque); }
    /* 16 px: com menos que isso o iPhone amplia a página ao tocar no campo */
    .lista-busca, .campo-v3, .periodo-filter .campo-v3 { font-size: 16px; }
    .campo-v3 { min-height: var(--toque); }
    .periodo-filter .campo-v3 { min-height: 42px; }
    .acao-v3 { width: 42px; height: 42px; }
    .salvas-toggle, .salvas-acao { min-height: var(--toque); }
  }

  /* --- Busca: botão apagar, dica "/" (só com mouse), destaque dos termos e trecho da ementa --- */
  .search-field.tem-texto input { padding-right: 52px; }

  .busca-limpar {
    position: absolute;
    top: 50%;
    right: 8px;
    display: inline-grid;
    place-items: center;
    width: 38px;
    height: 38px;
    padding: 0;
    font: inherit;
    font-size: 14px;
    line-height: 1;
    color: var(--ink-muted);
    background: transparent;
    border: 0;
    border-radius: 50%;
    transform: translateY(-50%);
    cursor: pointer;
  }

  .busca-limpar:hover { color: var(--ink); background: var(--paper-raised); }
  .busca-limpar[hidden] { display: none; }
  .busca-limpar:focus-visible { outline: 2px solid var(--focus); outline-offset: 0; }

  .busca-atalho {
    position: absolute;
    top: 50%;
    right: 14px;
    display: none;
    min-width: 24px;
    padding: 1px 7px;
    font: 600 12px/1.5 var(--fonte);
    text-align: center;
    color: var(--ink-muted);
    background: var(--paper-raised);
    border: 1px solid var(--rule-strong);
    border-bottom-width: 2px;
    border-radius: 6px;
    transform: translateY(-50%);
    pointer-events: none;
  }

  @media (hover: hover) and (pointer: fine) {
    .search-field input { padding-right: 52px; }
    .search-field input:placeholder-shown:not(:focus) ~ .busca-atalho { display: block; }
  }

  ::highlight(busca-v3) { background-color: rgba(226, 199, 102, 0.6); }
  :root[data-theme="dark"] ::highlight(busca-v3) { background-color: rgba(212, 182, 74, 0.4); }

  .trecho-v3 {
    margin: 12px 0 0;
    padding: 8px 12px;
    font-size: 13.5px;
    line-height: 1.55;
    color: var(--ink-muted);
    background: color-mix(in srgb, var(--ouro) 7%, transparent);
    border-left: 2px solid var(--ouro);
    border-radius: 0 8px 8px 0;
    overflow-wrap: anywhere;
  }

  .trecho-v3 .trecho-rotulo { font-weight: 700; color: var(--ouro-texto); }
  .trecho-v3 mark { padding: 0 1px; color: var(--ink); background: rgba(226, 199, 102, 0.6); border-radius: 2px; }
  :root[data-theme="dark"] .trecho-v3 mark { background: rgba(212, 182, 74, 0.4); }

  /* --- Temas frequentes e quantidades nas fichas --- */
  .temas-frequentes { align-items: center; gap: 6px; margin: 0; }

  .temas-rotulo {
    margin-right: 4px;
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 0.14em;
    text-transform: uppercase;
    color: var(--ouro-texto);
  }

  .temas-frequentes button { padding: 3px 10px; font-size: 12.5px; color: var(--ink); background: var(--paper); }
  .temas-frequentes button::before { content: none; }
  .temas-frequentes button[aria-pressed="true"] { border-color: var(--ouro); background: color-mix(in srgb, var(--ouro) 16%, var(--paper)); }

  .qtd-v3 { margin-left: 6px; font-size: 0.86em; font-weight: 600; color: var(--ink-muted); font-variant-numeric: tabular-nums; }
  .chips button .qtd-v3 { margin-left: 0; }
  .chips button.active .qtd-v3 { color: inherit; opacity: 0.8; }
  /* Tribunal sem resultado com os filtros atuais: borda tracejada e bolinha vazada
     (sem esmaecer o texto, que mantém o contraste) */
  .chips button.sem-resultado:not(.active) { border-style: dashed; box-shadow: none; }
  .chips button.sem-resultado:not(.active) .dot { background: transparent; box-shadow: inset 0 0 0 1.5px var(--chip); }

  @media (max-width: 640px) {
    .temas-frequentes {
      flex-wrap: nowrap;
      margin: 0 -14px;
      padding: 2px 14px;
      overflow-x: auto;
      overscroll-behavior-x: contain;
      scrollbar-width: none;
      -webkit-mask-image: linear-gradient(to right, transparent, #000 14px, #000 calc(100% - 26px), transparent);
              mask-image: linear-gradient(to right, transparent, #000 14px, #000 calc(100% - 26px), transparent);
    }
    .temas-frequentes::-webkit-scrollbar { display: none; }
    .temas-frequentes button, .temas-rotulo { flex: none; white-space: nowrap; }
    .temas-frequentes button { min-height: 34px; }
  }

  /* --- Filtros recolhíveis no celular: botão "Filtros (N)" e fichas dos filtros ativos --- */
  .painel-barra { display: none; }

  .filtros-toggle {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    min-height: var(--toque);
    padding: 0 14px 0 12px;
    font: inherit;
    font-size: 14px;
    font-weight: 600;
    color: var(--ink);
    background: var(--paper-raised);
    border: 1px solid var(--rule);
    border-radius: 10px;
    box-shadow: var(--shadow-sm);
    cursor: pointer;
  }

  .filtros-toggle::before {
    content: "";
    width: 17px;
    height: 17px;
    background: currentColor;
    -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2' stroke-linecap='round'%3E%3Cpath d='M4 6h9M19 6h1M4 12h3M13 12h7M4 18h11M21 18h-1'/%3E%3Ccircle cx='16' cy='6' r='2.5'/%3E%3Ccircle cx='10' cy='12' r='2.5'/%3E%3Ccircle cx='18' cy='18' r='2.5'/%3E%3C/svg%3E") center / contain no-repeat;
            mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2' stroke-linecap='round'%3E%3Cpath d='M4 6h9M19 6h1M4 12h3M13 12h7M4 18h11M21 18h-1'/%3E%3Ccircle cx='16' cy='6' r='2.5'/%3E%3Ccircle cx='10' cy='12' r='2.5'/%3E%3Ccircle cx='18' cy='18' r='2.5'/%3E%3C/svg%3E") center / contain no-repeat;
  }

  .filtros-toggle::after {
    content: "";
    width: 7px;
    height: 7px;
    margin: -3px 0 0 2px;
    border-right: 1.5px solid currentColor;
    border-bottom: 1.5px solid currentColor;
    transform: rotate(45deg);
    opacity: .7;
    transition: transform .18s var(--ease);
  }

  .filtros-toggle[aria-expanded="true"] { border-color: var(--focus); box-shadow: 0 0 0 3px var(--focus-ring); }
  .filtros-toggle[aria-expanded="true"]::after { transform: rotate(225deg); margin-top: 3px; }

  .filtros-qtd {
    display: inline-grid;
    place-items: center;
    min-width: 20px;
    height: 20px;
    padding: 0 6px;
    font-size: 12px;
    font-weight: 700;
    color: var(--ativo-ink);
    background: var(--ativo-bg);
    border-radius: 999px;
  }

  .filtros-qtd[hidden] { display: none; }

  .filtros-ativos { display: flex; flex-wrap: wrap; gap: 6px; }
  .filtros-ativos[hidden] { display: none; }

  .ficha-ativa {
    display: inline-flex;
    align-items: center;
    gap: 7px;
    min-height: 32px;
    max-width: 100%;
    padding: 3px 10px 3px 12px;
    font: inherit;
    font-size: 13px;
    text-align: left;
    color: var(--ink);
    background: color-mix(in srgb, var(--ouro) 12%, var(--paper-raised));
    border: 1px solid color-mix(in srgb, var(--ouro) 45%, transparent);
    border-radius: 999px;
    cursor: pointer;
  }

  .ficha-ativa::after { content: "✕"; font-size: 11px; color: var(--ink-muted); }
  .ficha-ativa:hover { border-color: var(--ouro); }

  @keyframes filtros-v3 {
    from { opacity: 0; transform: translateY(-6px); }
    to   { opacity: 1; transform: none; }
  }

  @media (max-width: 640px) {
    .painel-barra { display: flex; flex-wrap: wrap; align-items: center; gap: 8px; }
    .controls:not(.filtros-abertos) #painelFiltros { display: none; }
    .controls.filtros-abertos .filtros-ativos { display: none; }
    .controls.filtros-abertos #painelFiltros { animation: filtros-v3 .22s var(--ease); }
  }

  /* --- Estado vazio com sugestões --- */
  .empty-state.show { padding: 44px 16px 10px; font-size: 15.5px; color: var(--ink); }

  .empty-state::before {
    content: "";
    display: block;
    width: 44px;
    height: 44px;
    margin: 0 auto 14px;
    background: var(--ouro-texto);
    -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='1.6' stroke-linecap='round'%3E%3Ccircle cx='11' cy='11' r='7'/%3E%3Cpath d='m20 20-3.5-3.5M9 9l4 4M13 9l-4 4'/%3E%3C/svg%3E") center / contain no-repeat;
            mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='1.6' stroke-linecap='round'%3E%3Ccircle cx='11' cy='11' r='7'/%3E%3Cpath d='m20 20-3.5-3.5M9 9l4 4M13 9l-4 4'/%3E%3C/svg%3E") center / contain no-repeat;
  }

  .vazio-sugestoes {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 14px;
    padding: 4px 16px 28px;
    text-align: center;
  }

  .vazio-sugestoes[hidden] { display: none; }
  .vazio-dica { margin: 0; font-size: 14px; color: var(--ink-muted); }
  .vazio-fichas { display: flex; flex-wrap: wrap; justify-content: center; align-items: center; gap: 6px; }
  .vazio-rotulo { font-size: 13px; color: var(--ink-muted); }
  .vazio-acoes { display: flex; flex-wrap: wrap; justify-content: center; gap: 8px; }

  .vazio-acao {
    min-height: 36px;
    padding: 0 16px;
    font: inherit;
    font-size: 14px;
    font-weight: 600;
    color: var(--ink);
    background: var(--paper-raised);
    border: 1px solid var(--rule);
    border-radius: 999px;
    box-shadow: var(--shadow-sm);
    cursor: pointer;
  }

  .vazio-acao:hover { border-color: var(--rule-strong); }
  .vazio-acao.principal { margin-left: 4px; color: var(--ativo-ink); background: var(--ativo-bg); border-color: var(--ativo-bg); box-shadow: inset 0 -2px 0 var(--ativo-marca); }

  /* --- "Voltar ao topo": botão preto com dourado, depois de rolar uma tela --- */
  .topo-v3 {
    position: fixed;
    right: 16px;
    bottom: calc(18px + env(safe-area-inset-bottom, 0px));
    z-index: 55;
    display: grid;
    place-items: center;
    width: 46px;
    height: 46px;
    padding: 0;
    color: var(--ouro-claro);
    background: var(--hero-bg);
    border: 1px solid rgba(181, 151, 41, 0.55);
    border-radius: 50%;
    box-shadow: var(--shadow-pop);
    cursor: pointer;
    opacity: 0;
    visibility: hidden;
    transform: translateY(10px);
    transition: opacity .2s var(--ease), transform .2s var(--ease), visibility 0s linear .2s;
  }

  .topo-v3::before {
    content: "";
    width: 18px;
    height: 18px;
    background: currentColor;
    -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2.4' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M12 19V5M6 11l6-6 6 6'/%3E%3C/svg%3E") center / contain no-repeat;
            mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2.4' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M12 19V5M6 11l6-6 6 6'/%3E%3C/svg%3E") center / contain no-repeat;
  }

  .topo-v3.visivel { opacity: 1; visibility: visible; transform: none; transition: opacity .2s var(--ease), transform .2s var(--ease); }
  .topo-v3:hover { color: #FFFFFF; border-color: var(--ouro); }
  .topo-v3:focus-visible { outline: 2px solid var(--ouro-claro); outline-offset: 2px; }
  .topo-v3[hidden] { display: none; }

  @media (max-width: 560px) {
    .aviso-v3 { bottom: calc(76px + env(safe-area-inset-bottom, 0px)); }
  }

  /* --- Transições: os cartões que entram na lista (busca, filtro, página) deslizam de leve.
     A classe vem do recursos-v3 e não vale na carga da página (199 animações ao abrir pesavam). --- */
  @keyframes entrar-v3 {
    from { opacity: 0; transform: translateY(6px); }
    to   { opacity: 1; transform: none; }
  }

  li.entry.entrando { animation: entrar-v3 .3s var(--ease) backwards; }

  /* --- Leitor de PDF v3: barra com zoom e página; a pinça amplia só o documento --- */
  .viewer-head, .viewer-ferramentas, .viewer-busca { position: relative; z-index: 2; }
  .viewer-body { position: relative; z-index: 1; touch-action: pan-x pan-y; }
  .viewer-body:focus-visible { outline: 2px solid var(--focus); outline-offset: -2px; }

  .viewer-ferramentas {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 6px 12px;
    background: var(--paper-raised);
    border-bottom: 1px solid var(--rule);
  }

  .viewer-ferramentas[hidden] { display: none; }

  .vf-grupo {
    display: inline-flex;
    align-items: center;
    gap: 2px;
    padding: 2px;
    background: var(--paper);
    border: 1px solid var(--rule);
    border-radius: 10px;
  }

  .vf-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 6px;
    min-width: 36px;
    height: 32px;
    padding: 0 10px;
    font: inherit;
    font-size: 13px;
    font-weight: 600;
    color: var(--ink);
    background: transparent;
    border: 0;
    border-radius: 8px;
    cursor: pointer;
  }

  .vf-btn:hover:not(:disabled) { background: var(--paper-raised); box-shadow: var(--shadow-sm); }
  .vf-btn:disabled { opacity: 0.4; cursor: default; }
  .vf-btn:focus-visible { outline: 2px solid var(--focus); outline-offset: 1px; }

  .vf-menos::before, .vf-mais::before {
    content: "";
    width: 16px;
    height: 16px;
    background: currentColor;
    -webkit-mask: var(--icone) center / contain no-repeat;
            mask: var(--icone) center / contain no-repeat;
  }

  .vf-menos { --icone: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2.4' stroke-linecap='round'%3E%3Cpath d='M5 12h14'/%3E%3C/svg%3E"); }
  .vf-mais { --icone: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2.4' stroke-linecap='round'%3E%3Cpath d='M12 5v14M5 12h14'/%3E%3C/svg%3E"); }
  .vf-zoom { min-width: 60px; font-variant-numeric: tabular-nums; }

  .vf-pagina {
    margin: 0 0 0 auto;
    font-size: 13px;
    color: var(--ink-muted);
    font-variant-numeric: tabular-nums;
    white-space: nowrap;
  }

  .vf-buscar::before, .vf-anterior::before, .vf-proximo::before, .vf-fechar::before {
    content: "";
    width: 16px;
    height: 16px;
    background: currentColor;
    -webkit-mask: var(--icone) center / contain no-repeat;
            mask: var(--icone) center / contain no-repeat;
  }

  .vf-buscar { --icone: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2.2' stroke-linecap='round'%3E%3Ccircle cx='11' cy='11' r='7'/%3E%3Cpath d='m20 20-3.5-3.5'/%3E%3C/svg%3E"); border: 1px solid var(--rule); }
  .vf-buscar[aria-expanded="true"] { border-color: var(--focus); box-shadow: 0 0 0 3px var(--focus-ring); }
  .vf-anterior { --icone: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2.4' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='m6 15 6-6 6 6'/%3E%3C/svg%3E"); }
  .vf-proximo { --icone: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2.4' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='m6 9 6 6 6-6'/%3E%3C/svg%3E"); }
  .vf-fechar { --icone: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='black' stroke-width='2.4' stroke-linecap='round'%3E%3Cpath d='M6 6l12 12M18 6 6 18'/%3E%3C/svg%3E"); }

  .viewer-busca {
    display: flex;
    align-items: center;
    gap: 4px;
    padding: 6px 12px;
    background: var(--paper-raised);
    border-bottom: 1px solid var(--rule);
  }

  .viewer-busca[hidden] { display: none; }

  .vf-campo {
    flex: 1 1 auto;
    min-width: 0;
    height: 34px;
    margin-right: 4px;
    padding: 0 12px;
    font: inherit;
    font-size: 14px;
    color: var(--ink);
    background: var(--paper);
    border: 1px solid var(--rule);
    border-radius: 8px;
  }

  .vf-campo:focus { outline: none; border-color: var(--focus); box-shadow: 0 0 0 3px var(--focus-ring); }

  .vf-resultado {
    min-width: 72px;
    font-size: 13px;
    text-align: center;
    color: var(--ink-muted);
    font-variant-numeric: tabular-nums;
    white-space: nowrap;
  }

  ::highlight(pdf-busca-v3) { background-color: rgba(255, 196, 0, 0.4); }
  ::highlight(pdf-atual-v3) { background-color: rgba(255, 115, 0, 0.55); }
  .textLayer span.vf-marca { background: rgba(255, 196, 0, 0.4); }
  .textLayer span.vf-marca-atual { background: rgba(255, 115, 0, 0.55); }

  @media (pointer: coarse) {
    .vf-btn { min-width: 42px; height: 40px; }
    .vf-campo { height: 40px; font-size: 16px; }
  }

  @media (max-width: 560px) {
    .vf-buscar .vf-rotulo { position: absolute; width: 1px; height: 1px; margin: -1px; overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap; }
    .vf-resultado { min-width: 56px; font-size: 12.5px; }
    .viewer-ferramentas, .viewer-busca { padding: 6px 10px; }
  }

  /* --- Celular: cabeçalho mais baixo, para a primeira decisão aparecer logo --- */
  @media (max-width: 560px) {
    header.page-head { padding: 24px 16px 74px 26px; }
    header.page-head .head-acoes { position: absolute; top: 24px; right: 16px; gap: 10px; }
    header.page-head .eyebrow { min-height: 42px; margin-bottom: 10px; padding-right: 100px; }
    header.page-head h1 { font-size: clamp(25px, 7vw, 29px); margin-bottom: 8px; }
    header.page-head p.sub { font-size: 14px; line-height: 1.45; }
    header.page-head .brand { width: 42px; height: 42px; border-radius: 11px; margin-top: 0; }
    .controls { gap: 14px; padding: 16px 14px 14px; }
    .count-row { padding-top: 12px; }
    ul.docket { margin-top: 20px; gap: 14px; }
    /* "Ordenar" vira um ícone dentro do seletor (o rótulo continua para leitores de tela) */
    .ordem-rotulo { position: absolute; width: 1px; height: 1px; margin: -1px; overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap; }
    #ordemSelect {
      padding-left: 34px;
      background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%236B6560' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M7 4v16M3 16l4 4 4-4M17 20V4M13 8l4-4 4 4'/%3E%3C/svg%3E"), url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%236B6560' stroke-width='2.2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='m6 9 6 6 6-6'/%3E%3C/svg%3E");
      background-position: left 11px center, right 11px center;
      background-size: 15px, 13px;
    }
    :root[data-theme="dark"] #ordemSelect {
      background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%23A39C92' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M7 4v16M3 16l4 4 4-4M17 20V4M13 8l4-4 4 4'/%3E%3C/svg%3E"), url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%23A39C92' stroke-width='2.2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='m6 9 6 6 6-6'/%3E%3C/svg%3E");
    }
  }

  @media (prefers-reduced-motion: reduce) {
    .controls.filtros-abertos #painelFiltros, li.entry.entrando { animation: none; }
    li.entry.em-destaque { animation: none; box-shadow: 0 0 0 3px var(--ouro), var(--shadow-md); }
    .aviso-v3 { transform: translate(-50%, 0); }
    .topo-v3, .topo-v3.visivel { transform: none; }
  }
</style>
```

### B.2 `<script id="recursos-v3">` (1700 linhas)

**Onde:** logo depois do `</script>` do script principal (o bloco que termina com `lerEndereco();` e `})();`), antes do comentário `<!-- PDF.js 3.11.174 (Mozilla, licença Apache-2.0) … -->`.

```html
<script id="recursos-v3">
/* ===== Recursos v3 (19/10/2026): busca sem acento e na ementa, ordem, período, temas
   frequentes, link direto, referência, decisões salvas e leitor de PDF com zoom e busca.
   Usa os ganchos do script principal (window.acervo). Para voltar ao comportamento
   anterior: apagar este bloco e o bloco de estilo "design-v3". ===== */
(function () {
  var A = window.acervo;
  if (!A || !A.ganchos) return;
  var G = A.ganchos;

  // Cada recurso registra aqui o que faz em cada gancho do script principal.
  var filtros = [];   // function (li): false quando o recurso esconde a decisão
  var ativos = [];    // function (): true quando o recurso tem um filtro ligado
  var aoLimpar = [];  // function (): zera o filtro do recurso (Esc e "Limpar filtros")
  var aoAplicar = []; // function (info): roda depois de cada atualização da lista
  var fichasExtras = []; // function (): { rotulo, remover } quando o recurso tem filtro ligado

  G.filtro = function (li) {
    for (var i = 0; i < filtros.length; i++) if (!filtros[i](li)) return false;
    return true;
  };
  G.filtroAtivo = function () {
    return ativos.some(function (f) { return f(); });
  };
  G.limpar = function () {
    aoLimpar.forEach(function (f) { f(); });
  };
  G.depois = function (info) {
    aoAplicar.forEach(function (f) { f(info); });
  };

  // ---- Filtros recolhíveis no celular (até 640 px) e fichas dos filtros ativos ----
  // No celular o painel mostra só a busca e o botão "Filtros (N)"; os filtros
  // ligados aparecem como fichas, e tocar numa ficha desliga aquele filtro.
  var painel = document.querySelector('.controls');
  var filtrosToggle = document.getElementById('filtrosToggle');
  var filtrosQtd = document.getElementById('filtrosQtd');
  var filtrosAtivos = document.getElementById('filtrosAtivos');

  // Texto do botão do filtro, sem a contagem que o v3 acrescenta (ex.: "STJ")
  function rotuloBotao(grupo, valor) {
    var b = document.querySelector('[data-filter-group="' + grupo + '"] button[data-value="' + valor + '"]');
    if (!b) return valor;
    return Array.prototype.filter.call(b.childNodes, function (n) { return n.nodeType === 3; })
      .map(function (n) { return n.textContent; }).join('').trim();
  }

  // Volta um grupo de botões (Tribunal, Decisão) para "Todos" pelo próprio botão
  function clicarTodos(grupo) {
    var b = document.querySelector('[data-filter-group="' + grupo + '"] button[data-value="all"]');
    if (b) b.click();
  }

  function filtrosLigados() {
    var s = A.state, lista = [];
    if (s.tribunal !== 'all') lista.push({ rotulo: rotuloBotao('tribunal', s.tribunal), remover: function () { clicarTodos('tribunal'); } });
    if (s.decisao !== 'all') lista.push({ rotulo: rotuloBotao('decisao', s.decisao), remover: function () { clicarTodos('decisao'); } });
    if (s.relator !== 'all' && A.listas.relator) lista.push({ rotulo: 'Relator: ' + s.relator, remover: function () { A.listas.relator.escolher('all', false); } });
    if (s.crime !== 'all' && A.listas.crime) lista.push({ rotulo: s.crime, remover: function () { A.listas.crime.escolher('all', false); } });
    fichasExtras.forEach(function (f) { var x = f(); if (x) lista.push(x); });
    return lista;
  }

  if (painel && filtrosToggle && filtrosAtivos) {
    filtrosToggle.addEventListener('click', function () {
      var abrir = !painel.classList.contains('filtros-abertos');
      painel.classList.toggle('filtros-abertos', abrir);
      filtrosToggle.setAttribute('aria-expanded', String(abrir));
      if (!abrir) Object.keys(A.listas).forEach(function (k) { A.listas[k].fechar(false); });
    });

    aoAplicar.push(function () {
      var lista = filtrosLigados();
      filtrosQtd.hidden = !lista.length;
      filtrosQtd.textContent = lista.length ? String(lista.length) : '';
      filtrosToggle.setAttribute('aria-label', lista.length ? 'Filtros, ' + lista.length + (lista.length === 1 ? ' ligado' : ' ligados') : 'Filtros');
      filtrosAtivos.innerHTML = '';
      filtrosAtivos.hidden = !lista.length;
      lista.forEach(function (f) {
        var b = document.createElement('button');
        b.type = 'button';
        b.className = 'ficha-ativa';
        b.textContent = f.rotulo;
        b.setAttribute('aria-label', 'Remover filtro: ' + f.rotulo);
        b.addEventListener('click', function () {
          f.remover();
          filtrosToggle.focus();
        });
        filtrosAtivos.appendChild(b);
      });
    });
  }

  // ---- Texto "dobrado" para comparar ----
  // Minúsculas, sem acento, espaços simples e sem o ponto entre algarismos
  // (210.983 = 210983). Com `mapa`, guarda para cada caractere dobrado a
  // posição dele no texto original (serve para destacar o trecho certo).
  var MARCAS = /[\u0300-\u036f]/g;
  var ESPACOS = /\s{2,}|[\t\n\r\f\v\u00a0\u1680\u2000-\u200a\u2028\u2029\u202f\u205f\u3000\ufeff]/g; // só o que precisa virar um espaço simples
  var dobraDoChar = {}; // memória: á -> a, Ç -> c, º -> o…

  function ehDigito(c) { return c >= '0' && c <= '9'; }
  function ehLetraOuDigito(c) { return (c >= 'a' && c <= 'z') || ehDigito(c) || c > '\u007f'; }

  function dobrarChar(c) {
    var d = dobraDoChar[c];
    if (d === undefined) {
      d = c.normalize('NFKD').replace(MARCAS, '').toLowerCase();
      if (d && d.trim() === '') d = ' ';
      dobraDoChar[c] = d;
    }
    return d;
  }

  function dobrar(s, mapa) {
    // Sem mapa (texto pesquisável): o mesmo resultado, feito de uma vez pelo navegador
    if (!mapa) {
      return s.normalize('NFKD').replace(MARCAS, '').toLowerCase()
        .replace(/(\d)\.(?=\d)/g, '$1').replace(ESPACOS, ' ').trim();
    }
    var out = '';
    var ultimo = ' ';
    for (var i = 0; i < s.length; i++) {
      var code = s.charCodeAt(i);
      var d;
      if (code < 128) {
        if (code >= 65 && code <= 90) d = String.fromCharCode(code + 32);
        else if (code === 32 || (code >= 9 && code <= 13)) d = ' ';
        else if (code === 46 && ehDigito(ultimo) && ehDigito(s.charAt(i + 1))) d = '';
        else d = s.charAt(i);
      } else {
        d = dobrarChar(s.charAt(i));
      }
      for (var k = 0; k < d.length; k++) {
        var ch = d.charAt(k);
        if (ch === ' ' && ultimo === ' ') continue;
        out += ch;
        if (mapa) mapa.push(i);
        ultimo = ch;
      }
    }
    if (out.charAt(out.length - 1) === ' ') {
      out = out.slice(0, -1);
      if (mapa) mapa.pop();
    }
    return out;
  }

  // ---- Busca: sem acento, palavras em qualquer ordem, "frase entre aspas", ementa ----
  // O texto pesquisável de cada decisão é preparado uma única vez, aqui, ao abrir a
  // página; a cada tecla a busca só compara. Campos, do mais forte para o mais
  // fraco na Relevância: número do recurso, resumo, crimes, palavras-chave, linha
  // do relator e ementa oficial.
  var PESOS = [8, 6, 5, 4, 3, 1];
  var C_CITE = 0, C_TEMA = 1, C_CRIMES = 2, C_CHAVES = 3, C_META = 4, C_EMENTA = 5;
  var dados = new Map();

  function textoEmPartes(el) {
    return el ? Array.prototype.map.call(el.childNodes, function (n) { return n.textContent; }).join(' ') : '';
  }

  // Ementa oficial (texto do <template class="ementa">), lida uma vez na preparação
  function ementaDe(li) {
    var d = dados.get(li);
    if (d) return d.ementaTexto;
    var tpl = li.querySelector('template.ementa');
    return tpl ? (tpl.content ? tpl.content.textContent : tpl.textContent).trim() : '';
  }

  A.entries.forEach(function (li, i) {
    var ementa = ementaDe(li);
    dados.set(li, {
      ordem: i,
      ementaTexto: ementa,
      grupo: A.grupoTribunal(li.getAttribute('data-tribunal') || ''),
      decisao: li.getAttribute('data-decisao'),
      relator: li.getAttribute('data-relator'),
      crimes: (li.getAttribute('data-crimes') || '').split('|'),
      campos: [
        dobrar(textoEmPartes(li.querySelector('.cite'))),
        dobrar(textoEmPartes(li.querySelector('.tema'))),
        dobrar((li.getAttribute('data-crimes') || '').split('|').join(' · ')),
        dobrar(li.getAttribute('data-search') || ''),
        dobrar(textoEmPartes(li.querySelector('p.meta'))),
        dobrar(ementa)
      ]
    });
  });

  // Termos da busca: separados por espaço; "entre aspas" vale como frase. Letra
  // solta e palavras de ligação ("de", "em", "para") só contam dentro das aspas.
  // Termo que começa com algarismo não pode vir colado a outro algarismo antes
  // ("33" não acha "2033").
  var PONTAS = /^[.,;:!?()[\]{}'"\u00ab\u00bb\u201c\u201d\u2018\u2019\u2013\u2014\/-]+|[.,;:!?()[\]{}'"\u00ab\u00bb\u201c\u201d\u2018\u2019\u2013\u2014\/-]+$/g;
  var LIGACAO = ' o os as de da do das dos em no na nos nas um uma uns umas por para pra com que se ao aos ou pelo pela pelos pelas e ';

  function lerBusca(texto) {
    var termos = [];
    var re = /"([^"]*)"?|(\S+)/g, m;
    texto = String(texto || '').replace(/[\u201c\u201d\u00ab\u00bb]/g, '"');
    while ((m = re.exec(texto))) {
      var frase = m[1] !== undefined;
      var t = frase ? dobrar(m[1]) : dobrar(m[2].replace(/"/g, '')).replace(PONTAS, '');
      if (!t) continue;
      if (!frase && ((t.length === 1 && !ehDigito(t)) || LIGACAO.indexOf(' ' + t + ' ') !== -1)) continue;
      termos.push({ t: t, frase: frase, numero: ehDigito(t.charAt(0)) });
    }
    return termos;
  }

  var buscaLida = { texto: null, termos: [] };
  function termosDaBusca() {
    if (A.state.search !== buscaLida.texto) {
      buscaLida.texto = A.state.search;
      buscaLida.termos = lerBusca(A.state.search);
    }
    return buscaLida.termos;
  }

  function achar(campo, t, desde) {
    var i = campo.indexOf(t.t, desde || 0);
    if (t.numero) while (i > 0 && ehDigito(campo.charAt(i - 1))) i = campo.indexOf(t.t, i + 1);
    return i;
  }

  var pontos = new Map(); // decisão -> pontuação na busca atual (usada pela Relevância)
  var casou = new Map();  // decisão -> passou ou não na busca atual (usado pelas contagens)

  G.busca = function (li) {
    var r = casaBusca(li);
    casou.set(li, r);
    return r;
  };

  function casaBusca(li) {
    var termos = termosDaBusca();
    var d = dados.get(li);
    if (!d) return true;
    var soma = 0;
    for (var k = 0; k < termos.length; k++) {
      var t = termos[k], achou = false;
      for (var c = 0; c < d.campos.length; c++) {
        var i = achar(d.campos[c], t);
        if (i === -1) continue;
        achou = true;
        var inicio = i === 0 || !ehLetraOuDigito(d.campos[c].charAt(i - 1));
        soma += PESOS[c] * (inicio ? 1.5 : 1) * (t.frase ? 1.5 : 1);
      }
      if (!achou) return false;
    }
    pontos.set(li, soma);
    return true;
  }

  // Destaque dos termos no cartão (resumo, número, linha do relator e fichas), sem
  // mexer no HTML das decisões: usa a CSS Custom Highlight API quando existe.
  var temDestaque = !!(window.CSS && CSS.highlights && typeof window.Highlight === 'function');
  var dobradosDoNo = new WeakMap();

  function dobradoDoNo(no) {
    var x = dobradosDoNo.get(no);
    if (!x || x.valor !== no.nodeValue) {
      var m = [];
      x = { valor: no.nodeValue, t: dobrar(no.nodeValue, m), m: m };
      dobradosDoNo.set(no, x);
    }
    return x;
  }

  function ocorrencias(texto, t) {
    var lista = [], i = achar(texto, t);
    while (i !== -1) {
      lista.push(i);
      i = achar(texto, t, i + t.t.length);
    }
    return lista;
  }

  function destacarCartoes(mostradas, termos) {
    if (!temDestaque) return;
    if (!termos.length) { CSS.highlights.delete('busca-v3'); return; }
    var h = new Highlight();
    mostradas.forEach(function (li) {
      li.querySelectorAll('.tema, .cite, p.meta, .crimes button').forEach(function (el) {
        var w = document.createTreeWalker(el, NodeFilter.SHOW_TEXT), no;
        while ((no = w.nextNode())) {
          var x = dobradoDoNo(no);
          termos.forEach(function (t) {
            ocorrencias(x.t, t).forEach(function (a) {
              var r = document.createRange();
              r.setStart(no, x.m[a]);
              r.setEnd(no, x.m[a + t.t.length - 1] + 1);
              h.add(r);
            });
          });
        }
      });
    });
    CSS.highlights.set('busca-v3', h);
  }

  // Trecho da ementa: quando algum termo não aparece no cartão e só está na
  // ementa, mostra a passagem em volta da primeira ocorrência, com os termos marcados.
  var comTrecho = new Set();

  function intervalosNaEmenta(d, termos, de, ate) {
    var lista = [];
    termos.forEach(function (t) {
      ocorrencias(d.ementaDob, t).forEach(function (a) {
        var ini = d.ementaMapa[a], fim = d.ementaMapa[a + t.t.length - 1] + 1;
        if (ini >= de && fim <= ate) lista.push([ini, fim]);
      });
    });
    lista.sort(function (x, y) { return x[0] - y[0]; });
    var juntos = [];
    lista.forEach(function (x) {
      var ult = juntos[juntos.length - 1];
      if (ult && x[0] <= ult[1]) ult[1] = Math.max(ult[1], x[1]); else juntos.push([x[0], x[1]]);
    });
    return juntos;
  }

  function montarTrecho(li, d, termos, alvo) {
    if (!d.ementaMapa) {
      d.ementaMapa = [];
      d.ementaDob = dobrar(d.ementaTexto, d.ementaMapa);
    }
    var a = achar(d.ementaDob, alvo);
    if (a === -1) return null;
    var orig = d.ementaTexto;
    var ini = d.ementaMapa[a], fim = d.ementaMapa[a + alvo.t.length - 1] + 1;
    var curto = !!(window.matchMedia && window.matchMedia('(max-width: 560px)').matches); // celular: trecho menor
    var de = Math.max(0, ini - (curto ? 60 : 90)), ate = Math.min(orig.length, fim + (curto ? 110 : 170));
    if (de > 0) { var esp = orig.lastIndexOf(' ', de); de = esp > de - 30 && esp !== -1 ? esp + 1 : de; }
    if (ate < orig.length) { var esp2 = orig.indexOf(' ', ate); ate = esp2 !== -1 && esp2 < ate + 30 ? esp2 : ate; }
    var p = document.createElement('p');
    p.className = 'trecho-v3';
    var rotulo = document.createElement('span');
    rotulo.className = 'trecho-rotulo';
    rotulo.textContent = 'Na ementa:';
    p.appendChild(rotulo);
    p.appendChild(document.createTextNode(' ' + (de > 0 ? '…' : '')));
    var pos = de;
    intervalosNaEmenta(d, termos, de, ate).forEach(function (x) {
      if (x[0] > pos) p.appendChild(document.createTextNode(orig.slice(pos, x[0]).replace(/\s+/g, ' ')));
      var mk = document.createElement('mark');
      mk.textContent = orig.slice(x[0], x[1]);
      p.appendChild(mk);
      pos = x[1];
    });
    p.appendChild(document.createTextNode(orig.slice(pos, ate).replace(/\s+/g, ' ') + (ate < orig.length ? '…' : '')));
    return p;
  }

  function atualizarTrechos(mostradas, termos) {
    var agora = new Set();
    if (termos.length) {
      mostradas.forEach(function (li) {
        var d = dados.get(li);
        if (!d || !d.campos[C_EMENTA]) return;
        var alvo = null;
        for (var k = 0; k < termos.length && !alvo; k++) {
          var t = termos[k];
          var noCartao = achar(d.campos[C_CITE], t) !== -1 || achar(d.campos[C_TEMA], t) !== -1 ||
            achar(d.campos[C_CRIMES], t) !== -1 || achar(d.campos[C_META], t) !== -1;
          if (!noCartao && achar(d.campos[C_EMENTA], t) !== -1) alvo = t;
        }
        if (!alvo) return;
        var chave = buscaLida.texto;
        var velho = li.querySelector(':scope > .trecho-v3');
        if (velho && velho._busca === chave) { agora.add(li); return; }
        var novo = montarTrecho(li, d, termos, alvo);
        if (!novo) return;
        novo._busca = chave;
        if (velho) velho.replaceWith(novo);
        else {
          var antes = li.querySelector(':scope > .crimes') || li.querySelector(':scope > p.meta');
          if (antes) antes.after(novo); else li.appendChild(novo);
        }
        agora.add(li);
      });
    }
    comTrecho.forEach(function (li) {
      if (!agora.has(li)) {
        var t = li.querySelector(':scope > .trecho-v3');
        if (t) t.remove();
      }
    });
    comTrecho = agora;
  }

  aoAplicar.push(function (info) {
    var termos = termosDaBusca();
    atualizarTrechos(info.mostradas, termos);
    destacarCartoes(info.mostradas, termos);
  });

  // Botão ✕ para apagar a busca, dica "/" no computador e atalho "/" (que não
  // dispara quando o foco já está num campo de texto).
  var campoBusca = document.getElementById('searchInput');
  var buscaLimpar = document.getElementById('buscaLimpar');
  var caixaBusca = campoBusca ? campoBusca.parentNode : null;

  function marcarCampoBusca() {
    var tem = !!(campoBusca && campoBusca.value);
    if (buscaLimpar) buscaLimpar.hidden = !tem;
    if (caixaBusca) caixaBusca.classList.toggle('tem-texto', tem);
  }

  if (campoBusca) {
    campoBusca.setAttribute('aria-keyshortcuts', '/');
    campoBusca.addEventListener('input', marcarCampoBusca);
    aoAplicar.push(marcarCampoBusca);
    if (buscaLimpar) buscaLimpar.addEventListener('click', function () {
      campoBusca.value = '';
      campoBusca.dispatchEvent(new Event('input', { bubbles: true }));
      campoBusca.focus();
    });

    // No celular, um exemplo mais curto cabe inteiro no campo
    var exemploLongo = campoBusca.getAttribute('placeholder');
    var telaEstreita = window.matchMedia ? window.matchMedia('(max-width: 560px)') : null;
    var ajustarExemplo = function () {
      campoBusca.setAttribute('placeholder', telaEstreita && telaEstreita.matches ? 'Buscar tema, relator, número…' : exemploLongo);
    };
    ajustarExemplo();
    if (telaEstreita) {
      if (telaEstreita.addEventListener) telaEstreita.addEventListener('change', ajustarExemplo);
      else if (telaEstreita.addListener) telaEstreita.addListener(ajustarExemplo);
    }
  }

  function ehCampoDeTexto(el) {
    if (!el || el === document.body) return false;
    if (el.isContentEditable) return true;
    var tag = el.tagName;
    if (tag === 'TEXTAREA' || tag === 'SELECT') return true;
    if (tag !== 'INPUT') return false;
    var tipo = (el.getAttribute('type') || 'text').toLowerCase();
    return ['button', 'checkbox', 'radio', 'submit', 'reset', 'range', 'color', 'file', 'image'].indexOf(tipo) === -1;
  }

  document.addEventListener('keydown', function (e) {
    if (e.key !== '/' || !campoBusca || e.defaultPrevented) return;
    if (e.metaKey || e.ctrlKey !== e.altKey) return; // aceita AltGr (Ctrl+Alt), recusa Ctrl ou Cmd sozinhos
    if (ehCampoDeTexto(e.target) || ehCampoDeTexto(document.activeElement)) return;
    var leitorAberto = document.getElementById('pdfViewer');
    if (leitorAberto && leitorAberto.open) return;
    e.preventDefault();
    campoBusca.focus();
    campoBusca.select();
  });

  // ---- Endereço: parâmetros dos recursos v3 (ordem, período), ao lado dos do script principal ----
  var noEndereco = []; // function (URLSearchParams): grava ou apaga o parâmetro do recurso
  var lidoDoEndereco = (function () {
    try { return new URLSearchParams(location.search); } catch (e) { return null; }
  })();

  function gravarEnderecoV3() {
    try {
      var p = new URLSearchParams(location.search);
      noEndereco.forEach(function (f) { f(p); });
      var qs = p.toString();
      var novo = location.pathname + (qs ? '?' + qs : '') + location.hash;
      if (novo !== location.pathname + location.search + location.hash) history.replaceState(null, '', novo);
    } catch (e) { /* ex.: arquivo aberto do disco em navegador que bloqueia */ }
  }

  // ---- Ordem: Mais recentes (padrão), Mais antigas e Relevância ----
  // Com busca e sem escolha manual, a ordem passa sozinha para Relevância e volta
  // a Mais recentes quando a busca é apagada. Se a pessoa escolheu a ordem no
  // seletor, a escolha fica (também no endereço: ?ordem=antigas). Esc e "Limpar
  // filtros" não mudam a ordem escolhida.
  var ordemSelect = document.getElementById('ordemSelect');
  var ORDENS = ['recentes', 'antigas', 'relevancia'];
  var ordemEscolhida = null; // null = automática

  // Data de publicação (AAAAMMDD), tirada da linha "Publicado em DD/MM/AAAA"
  dados.forEach(function (d, li) {
    var meta = li.querySelector('p.meta');
    var m = meta && /Publicado em (\d{2})\/(\d{2})\/(\d{4})/.exec(meta.textContent);
    d.data = m ? Number(m[3] + m[2] + m[1]) : 0;
  });

  function ordemAtual() {
    if (termosDaBusca().length) return ordemEscolhida || 'relevancia';
    return ordemEscolhida && ordemEscolhida !== 'relevancia' ? ordemEscolhida : 'recentes';
  }

  G.ordenar = function (lista) {
    var ordem = ordemAtual();
    return lista.slice().sort(function (a, b) {
      var da = dados.get(a), db = dados.get(b);
      if (!da || !db) return 0;
      if (ordem === 'relevancia') {
        var dif = (pontos.get(b) || 0) - (pontos.get(a) || 0);
        if (dif) return dif;
      }
      if (da.data !== db.data) return ordem === 'antigas' ? da.data - db.data : db.data - da.data;
      return da.ordem - db.ordem;
    });
  };

  // Põe os cartões da página na ordem certa também no documento (leitor de tela e
  // Tab seguem essa ordem). Só mexe quando a ordem mudou.
  function ordenarNaPagina(mostradas) {
    for (var i = 1; i < mostradas.length; i++) {
      if (!(mostradas[i - 1].compareDocumentPosition(mostradas[i]) & Node.DOCUMENT_POSITION_FOLLOWING)) break;
    }
    if (i >= mostradas.length) return;
    var foco = document.activeElement;
    var lista = mostradas[0].parentNode;
    mostradas.forEach(function (li) { lista.appendChild(li); });
    if (foco && foco !== document.activeElement && lista.contains(foco)) foco.focus({ preventScroll: true });
  }

  if (ordemSelect) {
    var pedida = lidoDoEndereco && lidoDoEndereco.get('ordem');
    if (ORDENS.indexOf(pedida) !== -1) ordemEscolhida = pedida;
    ordemSelect.addEventListener('change', function () {
      ordemEscolhida = ordemSelect.value;
      A.aplicar();
    });
    aoAplicar.push(function (info) {
      var comBusca = termosDaBusca().length > 0;
      var rel = ordemSelect.querySelector('option[value="relevancia"]');
      rel.disabled = !comBusca;
      rel.textContent = comBusca ? 'Relevância' : 'Relevância (com busca)';
      ordemSelect.value = ordemAtual();
      ordenarNaPagina(info.mostradas);
    });
    noEndereco.push(function (p) {
      if (ordemEscolhida) p.set('ordem', ordemEscolhida); else p.delete('ordem');
    });
  }

  // ---- Período: Qualquer data, Últimos 12 meses, os quatro anos mais recentes e "Até AAAA" ----
  // Montado a partir da data de publicação, com a quantidade de decisões de cada
  // opção. No endereço: ?periodo=12m, ?periodo=2025 ou ?periodo=ate-2022.
  var periodoSelect = document.getElementById('periodoSelect');
  var periodo = '';

  var hoje = new Date();
  var desde12m = (hoje.getFullYear() - 1) * 10000 + (hoje.getMonth() + 1) * 100 + hoje.getDate();

  function anoDe(d) { return Math.floor(d.data / 10000); }

  function casaPeriodo(d, valor) {
    if (!valor) return true;
    if (valor === '12m') return d.data >= desde12m;
    if (valor.indexOf('ate-') === 0) return anoDe(d) <= Number(valor.slice(4));
    return anoDe(d) === Number(valor);
  }

  function rotuloPeriodo(valor) {
    if (valor === '12m') return 'últimos 12 meses';
    if (valor.indexOf('ate-') === 0) return 'até ' + valor.slice(4);
    return valor;
  }

  function contarPeriodo(valor) {
    var n = 0;
    dados.forEach(function (d) { if (d.data && casaPeriodo(d, valor)) n++; });
    return n;
  }

  function opcaoPeriodo(valor, texto) {
    var o = document.createElement('option');
    o.value = valor;
    o.textContent = texto + ' (' + contarPeriodo(valor) + ')';
    periodoSelect.appendChild(o);
    return o;
  }

  function mudarPeriodo(valor) {
    periodo = valor;
    periodoSelect.value = valor;
    periodoSelect.classList.toggle('ativo', !!valor);
  }

  if (periodoSelect) {
    var anos = [];
    dados.forEach(function (d) { var a = anoDe(d); if (a && anos.indexOf(a) === -1) anos.push(a); });
    anos.sort(function (a, b) { return b - a; });
    opcaoPeriodo('12m', 'Últimos 12 meses');
    anos.slice(0, 4).forEach(function (a) { opcaoPeriodo(String(a), String(a)); });
    if (anos.length > 4) opcaoPeriodo('ate-' + anos[4], 'Até ' + anos[4]);

    // Valor pedido pelo endereço; um ano fora da lista ganha a sua própria opção
    var pedido = lidoDoEndereco && lidoDoEndereco.get('periodo');
    if (pedido && /^(12m|\d{4}|ate-\d{4})$/.test(pedido)) {
      if (!periodoSelect.querySelector('option[value="' + pedido + '"]')) {
        opcaoPeriodo(pedido, pedido.indexOf('ate-') === 0 ? 'Até ' + pedido.slice(4) : pedido);
      }
      mudarPeriodo(pedido);
    }

    periodoSelect.addEventListener('change', function () {
      mudarPeriodo(periodoSelect.value);
      A.aplicar();
    });
    filtros.push(function (li) {
      var d = dados.get(li);
      return !periodo || !d || casaPeriodo(d, periodo);
    });
    ativos.push(function () { return !!periodo; });
    aoLimpar.push(function () { mudarPeriodo(''); });
    fichasExtras.push(function () {
      return periodo ? { rotulo: 'Período: ' + rotuloPeriodo(periodo), remover: function () { mudarPeriodo(''); A.aplicar(); } } : null;
    });
    noEndereco.push(function (p) {
      if (periodo) p.set('periodo', periodo); else p.delete('periodo');
    });
  }

  // ---- Temas frequentes: atalhos para o filtro Crime / Tema ----
  // As fichas ficam num bloco com a classe .crimes, como as palavras-chave dos
  // cartões: o clique (liga e desliga o filtro) e o destaque do tema escolhido
  // vêm do script principal. A quantidade é a do acervo inteiro.
  function rotuloComQtd(nome, n) {
    return nome + ', ' + n + (n === 1 ? ' decisão' : ' decisões');
  }

  var temasFrequentes = document.getElementById('temasFrequentes');
  if (temasFrequentes) {
    var porTema = {};
    A.entries.forEach(function (li) {
      (li.getAttribute('data-crimes') || '').split('|').forEach(function (c) {
        if (c) porTema[c] = (porTema[c] || 0) + 1;
      });
    });
    Object.keys(porTema)
      .sort(function (a, b) { return porTema[b] - porTema[a] || a.localeCompare(b, 'pt'); })
      .slice(0, 7)
      .forEach(function (c) {
        var b = document.createElement('button');
        b.type = 'button';
        b.setAttribute('data-crime', c);
        b.setAttribute('aria-label', rotuloComQtd(c, porTema[c]));
        b.title = 'Filtrar por ' + c;
        b.appendChild(document.createTextNode(c));
        var q = document.createElement('span');
        q.className = 'qtd-v3';
        q.setAttribute('aria-hidden', 'true');
        q.textContent = porTema[c];
        b.appendChild(q);
        temasFrequentes.appendChild(b);
      });
    aoAplicar.push(function () {
      temasFrequentes.querySelectorAll('button[data-crime]').forEach(function (b) {
        b.setAttribute('aria-pressed', String(b.getAttribute('data-crime') === A.state.crime));
      });
    });
  }

  // ---- Quantidade em cada ficha de Tribunal, de acordo com os outros filtros e a busca ----
  var qtdTribunal = {};
  document.querySelectorAll('[data-filter-group="tribunal"] button[data-value]').forEach(function (b) {
    var q = document.createElement('span');
    q.className = 'qtd-v3';
    q.setAttribute('aria-hidden', 'true');
    b.appendChild(q);
    qtdTribunal[b.getAttribute('data-value')] = q;
  });

  aoAplicar.push(function () {
    var s = A.state, cont = { all: 0 };
    var comBusca = s.search.trim() !== '';
    dados.forEach(function (d, li) {
      if (s.decisao !== 'all' && d.decisao !== s.decisao) return;
      if (s.relator !== 'all' && d.relator !== s.relator) return;
      if (s.crime !== 'all' && d.crimes.indexOf(s.crime) === -1) return;
      if (comBusca && casou.get(li) === false) return;
      if (!G.filtro(li)) return;
      cont[d.grupo] = (cont[d.grupo] || 0) + 1;
      cont.all++;
    });
    Object.keys(qtdTribunal).forEach(function (v) {
      var n = cont[v] || 0, q = qtdTribunal[v];
      q.textContent = n;
      q.parentNode.classList.toggle('sem-resultado', n === 0);
      q.parentNode.setAttribute('aria-label', rotuloComQtd(rotuloBotao('tribunal', v), n));
    });
  });

  // ---- Aviso curto (role="status"), para os botões só com ícone ----
  var aviso = document.getElementById('avisoV3');
  function avisar(msg) {
    if (!aviso) return;
    clearTimeout(aviso._tempo);
    aviso.textContent = '';
    aviso.classList.remove('visivel');
    void aviso.offsetWidth; // reinicia a animação e o anúncio
    aviso.textContent = msg;
    aviso.classList.add('visivel');
    aviso._tempo = setTimeout(function () { aviso.classList.remove('visivel'); }, 2600);
  }

  // ---- Identificação de cada decisão (link direto e lista de salvas) ----
  // ID = data-pdf do botão "Abrir PDF" (ex.: pdf-hc-1109495). Sem PDF, vem do
  // tribunal e do número do recurso (ex.: stf-are-1585074-sp).
  var porId = {};
  dados.forEach(function (d, li) {
    var a = li.querySelector('a.pdf-link[data-pdf]:not(.info)');
    var id = a ? a.getAttribute('data-pdf') : '';
    if (!id) {
      var cite = li.querySelector('.cite');
      var numero = cite ? Array.prototype.filter.call(cite.childNodes, function (n) { return n.nodeType === 3; })
        .map(function (n) { return n.textContent; }).join(' ').split(/\s[^\s\w\/.]\s/)[0] : '';
      id = (li.getAttribute('data-tribunal') || 'decisao') + '-' +
        dobrar(numero).replace(/(^|\s)n[o.]?(?=\s)/g, ' ').replace(/[^a-z0-9]+/g, '-').replace(/^-+|-+$/g, '');
    }
    var base = id, n = 2;
    while (porId[id]) id = base + '-' + n++;
    porId[id] = li;
    d.id = id;
  });

  // Partes da linha do recurso: "STJ", "AgRg no RHC nº 210983 / DF", "Recurso desprovido"
  function citacaoDe(li) {
    var cite = li.querySelector('.cite');
    if (!cite) return { sigla: '', recurso: '', resultado: '' };
    var tag = cite.querySelector('.tribunal-tag'), res = cite.querySelector('.resultado-tag');
    var recurso = Array.prototype.filter.call(cite.childNodes, function (n) { return n.nodeType === 3; })
      .map(function (n) { return n.textContent; }).join(' ').replace(/\s+/g, ' ').trim();
    return { sigla: tag ? tag.textContent.trim() : '', recurso: recurso, resultado: res ? res.textContent.trim() : '' };
  }

  function tituloDe(li) {
    var c = citacaoDe(li);
    return [c.sigla, c.recurso, c.resultado].filter(Boolean).join(' · ');
  }

  // ---- Ações novas em cada cartão, à direita da linha de botões ----
  // Criadas aqui (como as palavras-chave do script principal); o HTML das
  // decisões não muda. Os botões existentes continuam onde estão.
  function acoesDe(li) {
    var caixa = li.querySelector(':scope > .acoes-v3');
    if (!caixa) {
      caixa = document.createElement('span');
      caixa.className = 'acoes-v3';
      li.appendChild(caixa);
    }
    return caixa;
  }

  function botaoIcone(classe, rotulo) {
    var b = document.createElement('button');
    b.type = 'button';
    b.className = 'acao-v3 ' + classe;
    b.setAttribute('aria-label', rotulo);
    b.title = rotulo;
    return b;
  }

  // ---- Link direto para uma decisão: ?id=pdf-hc-1109495 ----
  // Fora do site publicado (HTML aberto do computador ou dentro do claude.ai), o
  // link copiado aponta para o endereço público abaixo.
  var ENDERECO_PUBLICO = 'https://ladirf.github.io/acervo/';
  var noSite = /^https?:$/.test(location.protocol) && !(window.claude && typeof window.claude.use === 'function');
  var compartilha = !!(navigator.share && window.matchMedia && window.matchMedia('(pointer: coarse)').matches);

  function linkDe(li) {
    var d = dados.get(li);
    return (noSite ? location.origin + location.pathname : ENDERECO_PUBLICO) + '?id=' + encodeURIComponent(d.id);
  }

  dados.forEach(function (d, li) {
    var b = botaoIcone('acao-link' + (compartilha ? ' compartilhar' : ''), (compartilha ? 'Compartilhar: ' : 'Copiar link: ') + tituloDe(li));
    b.title = compartilha ? 'Compartilhar' : 'Copiar link';
    acoesDe(li).appendChild(b);
  });

  document.addEventListener('click', function (e) {
    var b = e.target.closest('.acao-link');
    if (!b) return;
    var li = b.closest('li.entry');
    var url = linkDe(li);
    function copiar() {
      A.copiarTexto(url).then(function () { avisar('Link copiado'); }, function () { avisar('Não foi possível copiar o link'); });
    }
    if (compartilha) {
      navigator.share({ title: tituloDe(li), text: tituloDe(li), url: url }).catch(function (err) {
        if (!err || err.name !== 'AbortError') copiar();
      });
    } else copiar();
  });

  // ---- Copiar referência: só a última linha da ementa, entre parênteses ----
  // Ex.: "(AgRg no RHC n. 210.983/DF, relator Ministro Og Fernandes, …, DJEN de
  // 16/10/2026.)". O botão fica logo depois de "Copiar ementa" e não usa a classe
  // .copiar (que é a do botão da ementa no script principal).
  function referenciaDe(li) {
    var linhas = ementaDe(li).split('\n').map(function (s) { return s.trim(); }).filter(Boolean);
    var ultima = linhas[linhas.length - 1] || '';
    return /^\(.*\)$/.test(ultima) ? ultima : '';
  }

  dados.forEach(function (d, li) {
    var copiar = li.querySelector(':scope > .pdf-link.copiar');
    if (!copiar || !referenciaDe(li)) return;
    var b = document.createElement('button');
    b.type = 'button';
    b.className = 'pdf-link ref-v3';
    b.textContent = 'Copiar referência';
    copiar.after(b);
  });

  document.addEventListener('click', function (e) {
    var b = e.target.closest('.ref-v3');
    if (!b) return;
    var ref = referenciaDe(b.closest('li.entry'));
    if (!ref) return;
    A.copiarTexto(ref).then(
      function () { A.avisoBotao(b, 'Referência copiada ✓', true); },
      function () { A.avisoBotao(b, 'Não foi possível copiar', false); }
    );
  });

  // ---- Decisões salvas ("lista para este caso"), guardadas neste navegador ----
  // Marcador em cada cartão; "Só salvas (N)" filtra a lista; "Copiar lista" leva
  // resumo, referência e link de cada uma; "Esvaziar lista" apaga tudo. Sem
  // armazenamento (ex.: navegação privada bloqueada), vale só nesta visita.
  var CHAVE_SALVAS = 'acervo-salvas';
  var salvasToggle = document.getElementById('salvasToggle');
  var salvasQtd = document.getElementById('salvasQtd');
  var salvasBarra = document.getElementById('salvasBarra');
  var salvasTexto = document.getElementById('salvasTexto');
  var salvasCopiar = document.getElementById('salvasCopiar');
  var salvasEsvaziar = document.getElementById('salvasEsvaziar');
  var soSalvas = false;
  var avisouSemGuardar = false;

  function lerSalvas() {
    try {
      var v = JSON.parse(localStorage.getItem(CHAVE_SALVAS) || '[]');
      return Array.isArray(v) ? v.filter(function (x) { return typeof x === 'string'; }) : [];
    } catch (e) { return []; }
  }

  function gravarSalvas() {
    try { localStorage.setItem(CHAVE_SALVAS, JSON.stringify(salvas)); return true; } catch (e) { return false; }
  }

  var salvas = lerSalvas();

  function ehSalva(li) { return salvas.indexOf(dados.get(li).id) !== -1; }
  function quantasSalvas() { return salvas.filter(function (id) { return porId[id]; }).length; }

  // Marcador e fita de cada cartão: só quando a lista muda
  function pintarSalvas() {
    dados.forEach(function (d, li) {
      var salva = salvas.indexOf(d.id) !== -1;
      if (d.botaoSalvar) d.botaoSalvar.setAttribute('aria-pressed', String(salva));
      li.classList.toggle('salva', salva);
    });
  }

  function mostrarSalvas() {
    var n = quantasSalvas();
    salvasToggle.hidden = !n && !soSalvas;
    salvasToggle.setAttribute('aria-pressed', String(soSalvas));
    salvasToggle.setAttribute('aria-label', 'Só salvas, ' + n + (n === 1 ? ' decisão' : ' decisões'));
    salvasQtd.textContent = n;
    salvasBarra.hidden = !soSalvas;
    salvasTexto.textContent = n
      ? (n === 1 ? '1 decisão salva' : n + ' decisões salvas') + ' neste navegador.'
      : 'Nenhuma decisão salva neste navegador. Use o marcador no canto de cada decisão.';
    salvasCopiar.hidden = salvasEsvaziar.hidden = !n;
  }

  // Texto da lista: resumo, referência (ou a linha do relator, quando não há
  // ementa) e link de cada decisão, na ordem mostrada.
  function textoDaLista(lista) {
    return lista.map(function (li, i) {
      var tema = li.querySelector('.tema'), meta = li.querySelector('p.meta');
      return [
        (i + 1) + '. ' + tituloDe(li),
        tema ? tema.textContent.trim() : '',
        referenciaDe(li) || (meta ? meta.textContent.trim() : ''),
        linkDe(li)
      ].filter(Boolean).join('\n');
    }).join('\n\n');
  }

  if (salvasToggle && salvasQtd && salvasBarra && salvasTexto && salvasCopiar && salvasEsvaziar) {
    dados.forEach(function (d, li) {
      var b = botaoIcone('acao-salvar', 'Salvar: ' + tituloDe(li));
      b.title = 'Salvar decisão';
      b.setAttribute('aria-pressed', 'false');
      acoesDe(li).appendChild(b);
      d.botaoSalvar = b;
    });
    pintarSalvas();

    document.addEventListener('click', function (e) {
      var b = e.target.closest('.acao-salvar');
      if (!b) return;
      var id = dados.get(b.closest('li.entry')).id;
      var i = salvas.indexOf(id);
      if (i === -1) salvas.push(id); else salvas.splice(i, 1);
      if (!gravarSalvas() && !avisouSemGuardar) {
        avisouSemGuardar = true;
        avisar('Este navegador não guarda a lista: ela vale só nesta visita');
      } else avisar(i === -1 ? 'Decisão salva' : 'Removida das salvas');
      pintarSalvas();
      if (soSalvas) A.aplicar(true); else mostrarSalvas();
    });

    salvasToggle.addEventListener('click', function () {
      soSalvas = !soSalvas;
      A.aplicar();
    });
    salvasCopiar.addEventListener('click', function () {
      var lista = (ultimaLista ? ultimaLista.visiveis : []).filter(ehSalva);
      if (!lista.length) return;
      A.copiarTexto(textoDaLista(lista)).then(
        function () { avisar(lista.length === 1 ? 'Lista copiada: 1 decisão' : 'Lista copiada: ' + lista.length + ' decisões'); },
        function () { avisar('Não foi possível copiar a lista'); }
      );
    });
    salvasEsvaziar.addEventListener('click', function () {
      var n = quantasSalvas();
      if (!n || !window.confirm(n === 1 ? 'Remover a decisão salva neste navegador?' : 'Remover as ' + n + ' decisões salvas neste navegador?')) return;
      salvas = [];
      gravarSalvas();
      pintarSalvas();
      soSalvas = false;
      A.aplicar();
      avisar('Lista esvaziada');
      salvasToggle.focus();
    });
    filtros.push(function (li) { return !soSalvas || ehSalva(li); });
    ativos.push(function () { return soSalvas; });
    aoLimpar.push(function () { soSalvas = false; });
    aoAplicar.push(mostrarSalvas);
    // Outra aba do mesmo navegador mudou a lista
    window.addEventListener('storage', function (e) {
      if (e.key !== CHAVE_SALVAS) return;
      salvas = lerSalvas();
      pintarSalvas();
      A.aplicar(true);
    });
  }

  // Ao abrir com ?id=…: vai à página da decisão, rola até ela e a destaca. Se os
  // filtros do endereço a escondem, limpa os filtros. O id sai do endereço depois.
  var ultimaLista = null;
  aoAplicar.push(function (info) { ultimaLista = info; });

  function abrirDecisao(id) {
    try {
      var p = new URLSearchParams(location.search);
      p.delete('id');
      var qs = p.toString();
      history.replaceState(null, '', location.pathname + (qs ? '?' + qs : '') + location.hash);
    } catch (e) {}
    var li = porId[id];
    if (!li) { avisar('Decisão não encontrada no acervo'); return; }
    if (!ultimaLista || ultimaLista.visiveis.indexOf(li) === -1) A.limpar();
    var pos = ultimaLista ? ultimaLista.visiveis.indexOf(li) : -1;
    if (pos === -1) return;
    var pag = Math.floor(pos / A.porPagina) + 1;
    if (pag !== ultimaLista.pagina) A.irParaPagina(pag);
    li.setAttribute('tabindex', '-1');
    li.classList.add('em-destaque');
    var calmo = window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    requestAnimationFrame(function () {
      li.scrollIntoView({ block: 'center', behavior: calmo ? 'auto' : 'smooth' });
      li.focus({ preventScroll: true });
    });
    setTimeout(function () { li.classList.remove('em-destaque'); }, 4500);
  }

  // ---- Estado vazio com sugestões: quantas a busca acharia sem os filtros, ----
  // ---- fichas para tirar cada filtro, apagar a busca ou limpar tudo        ----
  var vazioSugestoes = document.getElementById('vazioSugestoes');
  var linhaContagem = document.getElementById('countLine');

  function focarContagem() {
    if (!linhaContagem) return;
    linhaContagem.setAttribute('tabindex', '-1');
    linhaContagem.focus({ preventScroll: true });
  }

  function botaoSugestao(classe, texto, rotulo, acao) {
    var b = document.createElement('button');
    b.type = 'button';
    b.className = classe;
    b.textContent = texto;
    if (rotulo) b.setAttribute('aria-label', rotulo);
    b.addEventListener('click', function () { acao(); focarContagem(); });
    return b;
  }

  // Tira os filtros e mantém a busca digitada
  function soLimparFiltros() {
    var q = A.state.search;
    A.limpar();
    if (campoBusca) {
      campoBusca.value = q;
      campoBusca.dispatchEvent(new Event('input', { bubbles: true }));
    }
  }

  if (vazioSugestoes) {
    aoAplicar.push(function (info) {
      var vazio = info.visiveis.length === 0;
      vazioSugestoes.hidden = !vazio;
      vazioSugestoes.innerHTML = '';
      if (!vazio) return;
      var temBusca = A.state.search.trim() !== '';
      var ligados = filtrosLigados();
      if (soSalvas) ligados.push({ rotulo: 'Só salvas', remover: function () { soSalvas = false; A.aplicar(); } });
      if (temBusca && ligados.length) {
        var n = 0;
        A.entries.forEach(function (li) { if (casou.get(li)) n++; });
        if (n) {
          var dica = document.createElement('p');
          dica.className = 'vazio-dica';
          dica.textContent = 'Sem os filtros, a busca acha ' + (n === 1 ? '1 decisão.' : n + ' decisões.');
          dica.appendChild(document.createTextNode(' '));
          dica.appendChild(botaoSugestao('vazio-acao principal', n === 1 ? 'Ver a decisão' : 'Ver as ' + n, null, soLimparFiltros));
          vazioSugestoes.appendChild(dica);
        }
      }
      if (ligados.length) {
        var fichas = document.createElement('div');
        fichas.className = 'vazio-fichas';
        var rot = document.createElement('span');
        rot.className = 'vazio-rotulo';
        rot.textContent = ligados.length === 1 ? 'Filtro ligado:' : 'Filtros ligados:';
        fichas.appendChild(rot);
        ligados.forEach(function (f) {
          fichas.appendChild(botaoSugestao('ficha-ativa', f.rotulo, 'Remover filtro: ' + f.rotulo, f.remover));
        });
        vazioSugestoes.appendChild(fichas);
      }
      if (temBusca && termosDaBusca().length > 1) {
        var menos = document.createElement('p');
        menos.className = 'vazio-dica';
        menos.textContent = 'Com várias palavras, a decisão precisa ter todas. Tente com menos palavras.';
        vazioSugestoes.appendChild(menos);
      }
      var acoes = document.createElement('div');
      acoes.className = 'vazio-acoes';
      if (temBusca && campoBusca) acoes.appendChild(botaoSugestao('vazio-acao', 'Apagar a busca', null, function () {
        campoBusca.value = '';
        campoBusca.dispatchEvent(new Event('input', { bubbles: true }));
      }));
      if (ligados.length) acoes.appendChild(botaoSugestao('vazio-acao', temBusca ? 'Limpar tudo' : 'Limpar filtros', null, function () { A.limpar(); }));
      vazioSugestoes.appendChild(acoes);
    });
  }

  // ---- Transições: os cartões que entram na lista deslizam de leve ----
  // Só os que acabaram de aparecer (não na carga da página, nem os que já estavam).
  var mostradasAntes = null;
  aoAplicar.push(function (info) {
    if (mostradasAntes) info.mostradas.forEach(function (li) {
      if (!mostradasAntes.has(li)) li.classList.add('entrando');
    });
    mostradasAntes = new Set(info.mostradas);
  });
  document.addEventListener('animationend', function (e) {
    if (e.animationName === 'entrar-v3') e.target.classList.remove('entrando');
  });

  // ---- Botão "Voltar ao topo", depois de rolar mais de uma tela ----
  var voltarTopo = document.getElementById('voltarTopo');
  if (voltarTopo) {
    voltarTopo.hidden = false;
    var quadroTopo = 0;
    var marcarTopo = function () {
      quadroTopo = 0;
      voltarTopo.classList.toggle('visivel', window.scrollY > window.innerHeight * 1.2);
    };
    window.addEventListener('scroll', function () {
      if (!quadroTopo) quadroTopo = requestAnimationFrame(marcarTopo);
    }, { passive: true });
    voltarTopo.addEventListener('click', function (e) {
      var calmo = window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches;
      window.scrollTo({ top: 0, behavior: calmo ? 'auto' : 'smooth' });
      // Pelo teclado, o foco vai para a busca; no toque, não abre o teclado do celular
      if (e.detail === 0 && campoBusca) campoBusca.focus({ preventScroll: true });
    });
    // No primeiro quadro (ler a rolagem agora forçaria o navegador a medir a página antes da hora)
    quadroTopo = requestAnimationFrame(marcarTopo);
  }

  // ---- Leitor de PDF v3: zoom (botões, teclado e pinça) e indicador de página ----
  // O arquivo chega pelo caminho de sempre: o clique em "Abrir PDF" usa obterPdf()
  // (PDF embutido no HTML do Drive ou pdfs/ID.pdf no site) e o script principal
  // chama mostrarPdf(bytes, nome), que passa para cá pelo gancho. O PDF.js vem de
  // carregarPdfjs(), o mesmo carregamento da página. Como antes, só as páginas
  // perto da área visível são desenhadas; ao mudar o zoom, o desenho antigo fica
  // (esticado) até o novo ficar pronto.
  var leitorDialog = document.getElementById('pdfViewer');
  var leitorCorpo = document.getElementById('viewerBody');
  var leitorArquivo = document.getElementById('viewerFile');
  var leitorFerramentas = document.getElementById('viewerFerramentas');
  var zoomMenos = document.getElementById('zoomMenos');
  var zoomValor = document.getElementById('zoomValor');
  var zoomMais = document.getElementById('zoomMais');
  var vfPagina = document.getElementById('vfPagina');
  var aoTextoDaPagina = []; // function (folha): a camada de texto da página ficou pronta
  var aoFecharPdf = [];     // function (): o documento foi fechado

  if (leitorDialog && leitorCorpo && leitorFerramentas && zoomMenos && zoomValor && zoomMais && vfPagina) {
    var L = { token: 0, doc: null, lib: null, base: 0, zoom: 1, folhas: [], quadro: 0, textos: null };
    var ZOOMS = [0.5, 0.75, 1, 1.25, 1.5, 2, 2.5, 3, 4];
    var LIMITE_PIXELS = 12000000; // por página; o Safari do iPhone recusa canvas muito grandes

    leitorFerramentas.hidden = false;
    leitorCorpo.tabIndex = 0; // rolagem pelo teclado

    var zoomMaximo = function () { return leitorCorpo.clientWidth < 700 ? 4 : 3; };

    var statusLeitor = function (msg) {
      leitorCorpo.innerHTML = '';
      var p = document.createElement('p');
      p.className = 'viewer-status';
      p.textContent = msg;
      leitorCorpo.appendChild(p);
    };

    // iPhone: zerar o tamanho libera a memória do canvas na hora
    var zerarCanvas = function (c) { c.width = 0; c.height = 0; };

    var liberar = function (f) {
      if (!f._ativa && !f.firstChild) return;
      f._ativa = false;
      f._ger++;
      if (f._tarefa) { f._tarefa.cancel(); f._tarefa = null; }
      Array.prototype.forEach.call(f.querySelectorAll('canvas'), zerarCanvas);
      f.innerHTML = '';
      f._textoPronto = false;
      f._divs = f._strs = null;
    };

    // Marca a folha para ser desenhada de novo (novo zoom ou nova largura),
    // sem apagar o desenho atual, que fica esticado até o novo ficar pronto
    var redesenharDepois = function (f) {
      if (!f._ativa && !f._tarefa) return;
      f._ativa = false;
      f._ger++;
      if (f._tarefa) { f._tarefa.cancel(); f._tarefa = null; }
    };

    var fecharDoc = function () {
      L.token++;
      L.folhas.forEach(liberar);
      L.folhas = [];
      if (L.doc) { L.doc.destroy(); L.doc = null; }
      L.textos = null;
      aoFecharPdf.forEach(function (f) { f(); });
    };

    var larguraBase = function () {
      return Math.min(leitorCorpo.clientWidth - (leitorCorpo.clientWidth < 600 ? 16 : 40), 920);
    };

    // Tamanho de cada folha: largura ajustada ao leitor vezes o zoom
    var medir = function () {
      var w = L.base * L.zoom;
      L.folhas.forEach(function (f) {
        f.style.width = w + 'px';
        f.style.height = (w * f._prop) + 'px';
        if (f._escala1) f.style.setProperty('--scale-factor', f._escala1 * L.zoom);
      });
    };

    var folhaEm = function (y) {
      for (var i = 0; i < L.folhas.length; i++) {
        var f = L.folhas[i];
        if (y < f.offsetTop + f.offsetHeight + 14) return f;
      }
      return L.folhas[L.folhas.length - 1] || null;
    };

    var desenhar = function (f) {
      if (f._ativa) return;
      f._ativa = true;
      var ger = ++f._ger, token = L.token, lib = L.lib;
      L.doc.getPage(f._n).then(function (page) {
        if (token !== L.token || ger !== f._ger) return;
        var v1 = page.getViewport({ scale: 1 });
        f._prop = v1.height / v1.width;
        f._escala1 = L.base / v1.width;
        var vp = page.getViewport({ scale: f._escala1 * L.zoom });
        f.style.width = vp.width + 'px';
        f.style.height = vp.height + 'px';
        f.style.setProperty('--scale-factor', vp.scale);
        var dpr = Math.min(window.devicePixelRatio || 1, Math.sqrt(LIMITE_PIXELS / (vp.width * vp.height)));
        var canvas = document.createElement('canvas');
        canvas.width = Math.floor(vp.width * dpr);
        canvas.height = Math.floor(vp.height * dpr);
        var texto = document.createElement('div');
        texto.className = 'textLayer';
        var antigos = Array.prototype.slice.call(f.children);
        if (!antigos.length) { f.appendChild(canvas); f.appendChild(texto); }
        f._tarefa = page.render({
          canvasContext: canvas.getContext('2d'),
          viewport: vp,
          transform: dpr !== 1 ? [dpr, 0, 0, dpr, 0, 0] : null
        });
        return f._tarefa.promise.then(function () {
          if (token !== L.token || ger !== f._ger) { zerarCanvas(canvas); return; }
          f._tarefa = null;
          if (antigos.length) {
            antigos.forEach(function (el) { if (el.tagName === 'CANVAS') zerarCanvas(el); el.remove(); });
            f.appendChild(canvas);
            f.appendChild(texto);
          }
          f._divs = [];
          f._strs = [];
          f._textoPronto = false;
          var conteudo = L.textos && L.textos[f._n - 1] ? L.textos[f._n - 1].conteudo : page.streamTextContent();
          return lib.renderTextLayer({ textContentSource: conteudo, container: texto, viewport: vp, textDivs: f._divs, textContentItemsStr: f._strs }).promise.then(function () {
            if (token !== L.token || ger !== f._ger) return;
            f._textoPronto = true;
            aoTextoDaPagina.forEach(function (fn) { fn(f); });
          });
        });
      }).catch(function () {
        if (f._ger === ger) f._ativa = false; // desenho cancelado: pode tentar de novo
      });
    };

    // Desenha as folhas perto da área visível (menos margem com zoom alto, para
    // poupar memória) e libera as distantes; atualiza "Página X de N".
    var atualizarLeitor = function () {
      if (!L.doc || !L.folhas.length) return;
      var topo = leitorCorpo.scrollTop, h = leitorCorpo.clientHeight;
      var margem = h * (L.zoom > 1.5 ? 0.6 : 1.5);
      L.folhas.forEach(function (f) {
        var a = f.offsetTop, b = a + f.offsetHeight;
        if (b > topo - margem && a < topo + h + margem) desenhar(f); else liberar(f);
      });
      var atual = folhaEm(topo + h / 2);
      vfPagina.textContent = atual ? 'Página ' + atual._n + ' de ' + L.folhas.length : '';
    };

    var mostrarZoom = function () {
      var pct = Math.round(L.zoom * 100) + '%';
      zoomValor.textContent = pct;
      zoomValor.setAttribute('aria-label', 'Zoom ' + pct + '. Ajustar à largura');
      zoomMenos.disabled = !L.doc || L.zoom <= ZOOMS[0] + 0.001;
      zoomMais.disabled = !L.doc || L.zoom >= zoomMaximo() - 0.001;
      zoomValor.disabled = !L.doc;
    };

    // Muda o zoom mantendo parado o ponto do documento que estava sob `de`
    // (posição dentro do leitor); com `para`, esse ponto termina sob `para`.
    var ajustarZoom = function (novo, de, para) {
      if (!L.doc || !L.folhas.length) return;
      novo = Math.max(ZOOMS[0], Math.min(zoomMaximo(), novo));
      if (Math.abs(novo - L.zoom) < 0.001) return;
      de = de || { x: leitorCorpo.clientWidth / 2, y: leitorCorpo.clientHeight / 2 };
      para = para || de;
      var px = leitorCorpo.scrollLeft + de.x, py = leitorCorpo.scrollTop + de.y;
      var f = folhaEm(py);
      var rx = (px - f.offsetLeft) / f.offsetWidth, ry = (py - f.offsetTop) / f.offsetHeight;
      L.zoom = novo;
      L.folhas.forEach(redesenharDepois);
      medir();
      leitorCorpo.scrollLeft = f.offsetLeft + rx * f.offsetWidth - para.x;
      leitorCorpo.scrollTop = f.offsetTop + ry * f.offsetHeight - para.y;
      mostrarZoom();
      atualizarLeitor();
    };

    var passoZoom = function (dir) {
      var z = L.zoom, max = zoomMaximo();
      var lista = ZOOMS.filter(function (x) { return x <= max + 0.001; });
      var prox = dir > 0
        ? lista.filter(function (x) { return x > z + 0.001; })[0]
        : lista.filter(function (x) { return x < z - 0.001; }).pop();
      if (prox) ajustarZoom(prox);
    };

    G.mostrarPdf = function (bytes, nome) {
      fecharDoc();
      var token = L.token;
      L.zoom = 1;
      mostrarZoom();
      vfPagina.textContent = '';
      leitorCorpo.scrollLeft = 0;
      statusLeitor('Carregando…');
      var lib;
      try { lib = A.carregarPdfjs(); } catch (err) { lib = null; }
      if (!lib) { statusLeitor('Não foi possível exibir o PDF aqui. Use “Baixar PDF”.'); return; }
      L.lib = lib;
      lib.getDocument({ data: bytes.slice(), isEvalSupported: false }).promise.then(function (doc) {
        if (token !== L.token) { doc.destroy(); return; }
        L.doc = doc;
        if (leitorArquivo) leitorArquivo.textContent = nome + ' · ' + doc.numPages + (doc.numPages === 1 ? ' página' : ' páginas');
        return doc.getPage(1).then(function (p1) {
          if (token !== L.token) return;
          leitorCorpo.innerHTML = '';
          L.base = larguraBase();
          var v = p1.getViewport({ scale: 1 });
          for (var n = 1; n <= doc.numPages; n++) {
            var f = document.createElement('div');
            f.className = 'pdf-page';
            f._n = n;
            f._prop = v.height / v.width;
            f._ger = 0;
            leitorCorpo.appendChild(f);
            L.folhas.push(f);
          }
          medir();
          mostrarZoom();
          atualizarLeitor();
        });
      }).catch(function () {
        if (token === L.token) statusLeitor('Não foi possível exibir o PDF aqui. Use “Baixar PDF”.');
      });
    };

    zoomMenos.addEventListener('click', function () { passoZoom(-1); });
    zoomMais.addEventListener('click', function () { passoZoom(1); });
    zoomValor.addEventListener('click', function () { ajustarZoom(1); });

    leitorCorpo.addEventListener('scroll', function () {
      if (L.quadro) return;
      L.quadro = requestAnimationFrame(function () { L.quadro = 0; atualizarLeitor(); });
    }, { passive: true });

    // Celular girado ou janela redimensionada: refaz a largura ajustada,
    // mantendo a página que estava no topo.
    window.addEventListener('resize', function () {
      if (!L.doc || !leitorDialog.open) return;
      clearTimeout(L.tempoTela);
      L.tempoTela = setTimeout(function () {
        if (!L.doc || !L.folhas.length || !leitorDialog.open) return; // fechou durante a espera
        var nova = larguraBase();
        if (Math.abs(nova - L.base) < 1) { atualizarLeitor(); return; }
        var f = folhaEm(leitorCorpo.scrollTop);
        var ry = (leitorCorpo.scrollTop - f.offsetTop) / f.offsetHeight;
        var fator = nova / L.base;
        L.base = nova;
        L.folhas.forEach(function (x) {
          if (x._escala1) x._escala1 *= fator;
          redesenharDepois(x);
        });
        medir();
        leitorCorpo.scrollTop = f.offsetTop + ry * f.offsetHeight;
        mostrarZoom();
        atualizarLeitor();
      }, 150);
    });

    // Teclado: + e - mudam o zoom, 0 ajusta à largura (fora dos campos de texto)
    leitorDialog.addEventListener('keydown', function (e) {
      if (e.ctrlKey || e.metaKey || e.altKey || ehCampoDeTexto(e.target)) return;
      if (e.key === '+' || e.key === '=') { e.preventDefault(); passoZoom(1); }
      else if (e.key === '-' || e.key === '_') { e.preventDefault(); passoZoom(-1); }
      else if (e.key === '0') { e.preventDefault(); ajustarZoom(1); }
    });

    // Pinça: durante o gesto a imagem é ampliada na hora (transform); ao soltar,
    // as páginas são redesenhadas nítidas no novo zoom, com o mesmo ponto sob os dedos.
    var pinca = null;
    var comecarPinca = function (x, y) {
      var caixa = leitorCorpo.getBoundingClientRect();
      pinca = { z0: L.zoom, x0: x - caixa.left, y0: y - caixa.top, x: x - caixa.left, y: y - caixa.top, s: 1, caixa: caixa };
      leitorCorpo.style.transformOrigin = pinca.x0 + 'px ' + pinca.y0 + 'px';
    };
    var moverPinca = function (escala, x, y) {
      var z = Math.max(ZOOMS[0], Math.min(zoomMaximo(), pinca.z0 * escala));
      pinca.s = z / pinca.z0;
      pinca.x = x - pinca.caixa.left;
      pinca.y = y - pinca.caixa.top;
      leitorCorpo.style.transform = 'translate(' + (pinca.x - pinca.x0) + 'px, ' + (pinca.y - pinca.y0) + 'px) scale(' + pinca.s + ')';
    };
    var terminarPinca = function () {
      if (!pinca) return;
      var p = pinca;
      pinca = null;
      leitorCorpo.style.transform = '';
      leitorCorpo.style.transformOrigin = '';
      if (Math.abs(p.s - 1) > 0.02) ajustarZoom(p.z0 * p.s, { x: p.x0, y: p.y0 }, { x: p.x, y: p.y });
    };
    var distancia = function (t) { return Math.sqrt(Math.pow(t[0].clientX - t[1].clientX, 2) + Math.pow(t[0].clientY - t[1].clientY, 2)); };

    leitorCorpo.addEventListener('touchstart', function (e) {
      if (e.touches.length !== 2 || !L.doc) return;
      comecarPinca((e.touches[0].clientX + e.touches[1].clientX) / 2, (e.touches[0].clientY + e.touches[1].clientY) / 2);
      pinca.d0 = distancia(e.touches);
    }, { passive: true });
    leitorCorpo.addEventListener('touchmove', function (e) {
      if (!pinca || e.touches.length !== 2 || !pinca.d0) return;
      e.preventDefault();
      moverPinca(distancia(e.touches) / pinca.d0, (e.touches[0].clientX + e.touches[1].clientX) / 2, (e.touches[0].clientY + e.touches[1].clientY) / 2);
    }, { passive: false });
    leitorCorpo.addEventListener('touchend', function (e) { if (e.touches.length < 2) terminarPinca(); });
    leitorCorpo.addEventListener('touchcancel', terminarPinca);

    // Safari: a pinça não amplia a página inteira; no Mac (trackpad, sem toque) vira zoom do PDF
    var semToque = !navigator.maxTouchPoints;
    leitorCorpo.addEventListener('gesturestart', function (e) {
      e.preventDefault();
      if (semToque && L.doc && !pinca) comecarPinca(e.clientX, e.clientY);
    });
    leitorCorpo.addEventListener('gesturechange', function (e) {
      e.preventDefault();
      if (semToque && pinca) moverPinca(e.scale, e.clientX, e.clientY);
    });
    leitorCorpo.addEventListener('gestureend', function (e) {
      e.preventDefault();
      if (semToque) terminarPinca();
    });

    // Ctrl + roda do mouse ou pinça no trackpad (Chrome, Edge, Firefox): zoom do PDF
    leitorCorpo.addEventListener('wheel', function (e) {
      if (!e.ctrlKey || !L.doc) return;
      e.preventDefault();
      if (!pinca) { comecarPinca(e.clientX, e.clientY); pinca.acumulado = 1; }
      pinca.acumulado *= Math.exp(-e.deltaY / 250);
      moverPinca(pinca.acumulado, pinca.x0 + pinca.caixa.left, pinca.y0 + pinca.caixa.top);
      clearTimeout(L.tempoRoda);
      L.tempoRoda = setTimeout(terminarPinca, 180);
    }, { passive: false });

    // Fechou (✕, Esc ou clique fora): para os desenhos e libera a memória. O
    // evento pode chegar atrasado, depois de reabrir; aí não faz nada.
    leitorDialog.addEventListener('close', function () {
      if (leitorDialog.open) return;
      pinca = null;
      leitorCorpo.style.transform = '';
      fecharDoc();
      mostrarZoom();
      vfPagina.textContent = '';
    });

    // ---- Busca no documento: "3 de 12", anterior e próxima, termos destacados ----
    // O texto de todas as páginas é lido uma vez por documento (do próprio PDF.js);
    // a busca ignora acentos e maiúsculas, como a busca do acervo. Ctrl+F ou Cmd+F
    // com o leitor aberto abre esta busca; Esc no campo fecha só a busca.
    var vfBuscar = document.getElementById('vfBuscar');
    var vfBusca = document.getElementById('vfBusca');
    var vfCampo = document.getElementById('vfCampo');
    var vfResultado = document.getElementById('vfResultado');
    var vfAnterior = document.getElementById('vfAnterior');
    var vfProximo = document.getElementById('vfProximo');
    var vfFechar = document.getElementById('vfFechar');

    if (vfBuscar && vfBusca && vfCampo && vfResultado && vfAnterior && vfProximo && vfFechar) {
      var B = { termo: '', resultados: [], atual: -1, tempo: 0, contou: 0 };
      var aoLerPagina = []; // function (n): o texto da página n foi lido (n = 0: terminou)

      // Devolve a vez ao navegador entre uma página e outra: aqui o PDF.js roda na
      // própria página (sem Web Worker), e ler tudo de uma vez travaria a tela.
      var canalVez = new MessageChannel(), filaVez = [];
      canalVez.port1.onmessage = function () { var f = filaVez.shift(); if (f) f(); };
      var darVez = function (f) { filaVez.push(f); canalVez.port2.postMessage(0); };

      // Texto de cada página, lido uma vez por documento, em segundo plano e na
      // ordem das páginas: itens do PDF.js juntos (fim de linha vira espaço), a
      // versão dobrada e o mapa de volta para o texto original.
      var lerTextos = function () {
        if (L.textos || !L.doc) return;
        var doc = L.doc, token = L.token, n = 0;
        L.textos = [];
        L.textosProntos = false;
        var proxima = function () {
          if (token !== L.token) return;
          if (n >= doc.numPages) {
            L.textosProntos = true;
            aoLerPagina.forEach(function (fn) { fn(0); });
            return;
          }
          var num = ++n;
          doc.getPage(num).then(function (page) {
            return page.getTextContent().then(function (tc) {
              if (token !== L.token) return;
              var itens = tc.items.filter(function (it) { return it.str !== undefined; });
              var inicios = [], orig = '';
              itens.forEach(function (it) {
                inicios.push(orig.length);
                orig += it.str;
                if (it.hasEOL) orig += ' ';
              });
              var mapa = [];
              var dob = dobrar(orig, mapa);
              L.textos[num - 1] = { conteudo: tc, itens: itens, inicios: inicios, dob: dob, mapa: new Uint32Array(mapa), vp1: page.getViewport({ scale: 1 }) };
              aoLerPagina.forEach(function (fn) { fn(num); });
            });
          }).catch(function () { /* página sem texto legível: segue para a próxima */ })
            .then(function () { darVez(proxima); });
        };
        proxima();
      };

      // Item de texto que contém a posição `pos` do texto original da página
      var itemDe = function (t, pos) {
        var a = 0, b = t.inicios.length - 1;
        while (a < b) {
          var m = (a + b + 1) >> 1;
          if (t.inicios[m] <= pos) a = m; else b = m - 1;
        }
        return a;
      };

      var intervaloDe = function (f, t, r) {
        if (!f._textoPronto || !f._divs || f._divs.length !== t.itens.length) return null;
        var oa = t.mapa[r.a], ob = t.mapa[r.b - 1] + 1;
        var i = itemDe(t, oa), j = itemDe(t, ob - 1);
        var ni = f._divs[i] && f._divs[i].firstChild, nj = f._divs[j] && f._divs[j].firstChild;
        if (!ni || !nj) return null;
        var rg = document.createRange();
        rg.setStart(ni, Math.min(Math.max(0, oa - t.inicios[i]), ni.length));
        rg.setEnd(nj, Math.min(Math.max(0, ob - t.inicios[j]), nj.length));
        return { rg: rg, i: i, j: j };
      };

      var limparDestaquesPdf = function () {
        if (temDestaque) {
          CSS.highlights.delete('pdf-busca-v3');
          CSS.highlights.delete('pdf-atual-v3');
        }
        Array.prototype.forEach.call(leitorCorpo.querySelectorAll('.vf-marca'), function (s) {
          s.classList.remove('vf-marca', 'vf-marca-atual');
        });
      };

      // Destaca os resultados nas páginas já desenhadas (as outras ganham o
      // destaque quando forem desenhadas). Sem a Highlight API, marca o trecho inteiro.
      var destacarPdf = function () {
        limparDestaquesPdf();
        if (!B.resultados.length || !L.textos) return;
        var todos = temDestaque ? new Highlight() : null, atual = temDestaque ? new Highlight() : null;
        B.resultados.forEach(function (r, k) {
          var f = L.folhas[r.pag - 1];
          var x = f && intervaloDe(f, L.textos[r.pag - 1], r);
          if (!x) return;
          if (temDestaque) (k === B.atual ? atual : todos).add(x.rg);
          else for (var s = x.i; s <= x.j; s++) f._divs[s].classList.add('vf-marca', k === B.atual ? 'vf-marca-atual' : 'vf-marca');
        });
        if (temDestaque) {
          CSS.highlights.set('pdf-busca-v3', todos);
          CSS.highlights.set('pdf-atual-v3', atual);
        }
      };

      aoTextoDaPagina.push(function () { if (B.resultados.length) destacarPdf(); });

      // "3 de 12" (com "…" enquanto ainda lê as páginas seguintes)
      var mostrarContagemPdf = function () {
        var n = B.resultados.length;
        var lendo = !!(L.textos && !L.textosProntos);
        vfAnterior.disabled = vfProximo.disabled = n < 2;
        if (!B.termo || B.termo.length < 2) vfResultado.textContent = '';
        else if (!n || B.atual < 0) vfResultado.textContent = lendo ? 'Procurando…' : 'Nenhum resultado';
        else vfResultado.textContent = (B.atual + 1) + ' de ' + n + (lendo ? '…' : '');
      };

      // Vai ao resultado k: rola até a linha (posição do texto no PDF, mesmo antes
      // de a página ser desenhada) e destaca
      var irPara = function (k) {
        var n = B.resultados.length;
        if (!n) return;
        B.atual = ((k % n) + n) % n;
        var r = B.resultados[B.atual];
        var f = L.folhas[r.pag - 1], t = L.textos[r.pag - 1];
        var it = t.itens[itemDe(t, t.mapa[r.a])];
        var pt = t.vp1.convertToViewportPoint(it.transform[4], it.transform[5]);
        var escala = f.offsetWidth / t.vp1.width;
        var x = f.offsetLeft + pt[0] * escala, y = f.offsetTop + pt[1] * escala;
        leitorCorpo.scrollTop = Math.max(0, y - leitorCorpo.clientHeight / 3);
        if (x < leitorCorpo.scrollLeft + 20 || x > leitorCorpo.scrollLeft + leitorCorpo.clientWidth - 60) {
          leitorCorpo.scrollLeft = Math.max(0, x - 40);
        }
        mostrarContagemPdf();
        atualizarLeitor();
        destacarPdf();
      };

      var procurarNaPagina = function (num) {
        var t = L.textos[num - 1], q = B.termo, achou = 0;
        if (!t) return 0;
        var a = t.dob.indexOf(q);
        while (a !== -1) {
          B.resultados.push({ pag: num, a: a, b: a + q.length });
          achou++;
          a = t.dob.indexOf(q, a + q.length);
        }
        return achou;
      };

      // Assim que aparece um resultado na página da tela ou depois dela, vai até
      // ele; se só houver antes, volta ao primeiro quando terminar de ler
      var irAoPrimeiro = function () {
        if (B.atual !== -1 || !B.resultados.length) return;
        var aqui = folhaEm(leitorCorpo.scrollTop + 1);
        var desde = aqui ? aqui._n : 1;
        var k = B.resultados.findIndex(function (r) { return r.pag >= desde; });
        if (k !== -1) irPara(k);
        else if (L.textosProntos) irPara(0);
      };

      // Busca progressiva: as páginas já lidas na hora, as outras conforme chegam
      aoLerPagina.push(function (num) {
        if (!B.termo || B.termo.length < 2) return;
        var novos = num ? procurarNaPagina(num) : 0;
        irAoPrimeiro();
        var f = num && L.folhas[num - 1];
        if (novos && f && f._textoPronto) destacarPdf();
        // contagem no máximo a cada 0,4 s enquanto lê (o leitor de tela não repete tanto)
        var agora = Date.now();
        if (!num || agora - B.contou > 400) { B.contou = agora; mostrarContagemPdf(); }
      });

      var buscarNoPdf = function () {
        clearTimeout(B.tempo);
        B.tempo = 0;
        var q = dobrar(vfCampo.value);
        if (q === B.termo) return;
        B.termo = q;
        B.resultados = [];
        B.atual = -1;
        limparDestaquesPdf();
        if (q.length < 2 || !L.doc) { mostrarContagemPdf(); return; }
        lerTextos();
        L.textos.forEach(function (t, i) { if (t) procurarNaPagina(i + 1); });
        irAoPrimeiro();
        B.contou = Date.now();
        mostrarContagemPdf();
        if (B.resultados.length) destacarPdf();
      };

      var abrirBuscaPdf = function () {
        vfBusca.hidden = false;
        vfBuscar.setAttribute('aria-expanded', 'true');
        vfCampo.focus();
        vfCampo.select();
        lerTextos(); // começa a ler o texto já, enquanto a pessoa digita
      };

      var fecharBuscaPdf = function (devolverFoco) {
        clearTimeout(B.tempo);
        vfBusca.hidden = true;
        vfBuscar.setAttribute('aria-expanded', 'false');
        B.termo = '';
        B.resultados = [];
        B.atual = -1;
        limparDestaquesPdf();
        mostrarContagemPdf();
        if (devolverFoco) vfBuscar.focus();
      };

      vfBuscar.addEventListener('click', function () {
        if (vfBusca.hidden) abrirBuscaPdf(); else fecharBuscaPdf(true);
      });
      vfFechar.addEventListener('click', function () { fecharBuscaPdf(true); });
      vfAnterior.addEventListener('click', function () { irPara(B.atual - 1); });
      vfProximo.addEventListener('click', function () { irPara(B.atual + 1); });
      vfCampo.addEventListener('input', function () {
        clearTimeout(B.tempo);
        B.tempo = setTimeout(buscarNoPdf, 220);
      });
      vfCampo.addEventListener('keydown', function (e) {
        if (e.key !== 'Enter') return;
        e.preventDefault();
        var volta = e.shiftKey;
        if (B.tempo || dobrar(vfCampo.value) !== B.termo) buscarNoPdf();
        else irPara(B.atual + (volta ? -1 : 1));
      });

      leitorDialog.addEventListener('keydown', function (e) {
        if ((e.ctrlKey || e.metaKey) && !e.altKey && (e.key === 'f' || e.key === 'F')) {
          e.preventDefault();
          abrirBuscaPdf();
        }
      });
      // Esc com o foco na busca fecha só a busca, não o leitor
      leitorDialog.addEventListener('cancel', function (e) {
        if (!vfBusca.hidden && vfBusca.contains(document.activeElement)) {
          e.preventDefault();
          fecharBuscaPdf(true);
        }
      });

      aoFecharPdf.push(function () { fecharBuscaPdf(false); });
      mostrarContagemPdf();
    }

    mostrarZoom();
  }

  // Primeira atualização com os recursos v3 (mantém a página lida do endereço).
  aoAplicar.push(gravarEnderecoV3);
  A.aplicar(true);

  var idPedido = lidoDoEndereco && lidoDoEndereco.get('id');
  if (idPedido) abrirDecisao(idPedido);
})();
</script>
```

---

## C. Marcação alterada (fora das decisões)

Nenhuma linha das decisões mudou. Os botões novos dos cartões ("Copiar referência", link, salvar) e o trecho "Na ementa" são criados pelo JavaScript, como as palavras-chave do script principal.

### C.1 Painel de busca e filtros (`div.controls`)

Mudanças, em ordem:

- dentro de `div.search-field`, depois do `input#searchInput`: `button#buscaLimpar` e `kbd.busca-atalho` (o `input` não muda);
- depois de `div.search-field`: `div#temasFrequentes` (com a classe `crimes`, de propósito: ver seção D);
- depois dela: `div.painel-barra` com `button#filtrosToggle` e `div#filtrosAtivos` (só aparece no celular);
- `div.filter-row` ganha `id="painelFiltros"`;
- logo depois do grupo Tribunal: `div.filter-group.periodo-filter` (sem `data-filter-group`, para o script principal não tratá-lo como seu);
- em `div.count-row`, entre `p#countLine` e `button#limparFiltros`: `div#resultadosAcoes` com `button#salvasToggle` e o seletor `select#ordemSelect`;
- logo depois da `div.controls`: `div#salvasBarra` (a barra da lista salva).

**Antes:**

```html
  <div class="controls">
    <div class="search-field">
      <input type="text" id="searchInput" aria-label="Buscar decisões" enterkeyhint="search" placeholder="Buscar por tema, relator, número do processo…" autocomplete="off">
    </div>
    <div class="filter-row">
      <div class="filter-group" data-filter-group="tribunal" role="group" aria-label="Filtrar por tribunal">
        <span class="label">Tribunal</span>
        <div class="chips">
          <button type="button" class="active" aria-pressed="true" data-value="all"><span class="dot"></span>Todos</button>
          <button type="button" aria-pressed="false" data-value="stj"><span class="dot"></span>STJ</button>
          <button type="button" aria-pressed="false" data-value="stf"><span class="dot"></span>STF</button>
          <button type="button" aria-pressed="false" data-value="trf"><span class="dot"></span>TRFs</button>
          <button type="button" aria-pressed="false" data-value="tj"><span class="dot"></span>TJs</button>
        </div>
      </div>
      <div class="filter-divider" aria-hidden="true"></div>
      <div class="filter-group" data-filter-group="decisao" role="group" aria-label="Filtrar por tipo de decisão">
        <span class="label">Decisão</span>
        <div class="segmented">
          <button type="button" class="active" aria-pressed="true" data-value="all">Todas</button>
          <button type="button" aria-pressed="false" data-value="monocratica">Monocrática</button>
          <button type="button" aria-pressed="false" data-value="colegiada">Colegiada</button>
        </div>
      </div>
      <div class="filter-group lista-filter" data-lista="relator" data-atributo="data-relator" data-todos="Todos os relatores" data-busca="Filtrar relator…">
        <span class="label" id="rotulo-relator">Relator</span>
        <div class="lista-controle">
          <button type="button" class="lista-trigger" aria-haspopup="listbox" aria-expanded="false" aria-labelledby="rotulo-relator atual-relator">
            <span class="lista-atual" id="atual-relator">Todos</span>
            <span class="chevron" aria-hidden="true"></span>
          </button>
          <button type="button" class="lista-clear" aria-label="Limpar filtro de relator" title="Limpar" hidden>✕</button>
        </div>
        <div class="lista-pop" hidden>
          <input type="text" class="lista-busca" autocomplete="off" aria-label="Filtrar lista de relatores">
          <ul class="lista-opcoes" role="listbox" aria-label="Relatores"></ul>
        </div>
      </div>
      <div class="filter-group lista-filter" data-lista="crime" data-atributo="data-crimes" data-todos="Todos os crimes e temas" data-busca="Filtrar crime ou tema…">
        <span class="label" id="rotulo-crime">Crime / Tema</span>
        <div class="lista-controle">
          <button type="button" class="lista-trigger" aria-haspopup="listbox" aria-expanded="false" aria-labelledby="rotulo-crime atual-crime">
            <span class="lista-atual" id="atual-crime">Todos</span>
            <span class="chevron" aria-hidden="true"></span>
          </button>
          <button type="button" class="lista-clear" aria-label="Limpar filtro de crime ou tema" title="Limpar" hidden>✕</button>
        </div>
        <div class="lista-pop" hidden>
          <input type="text" class="lista-busca" autocomplete="off" aria-label="Filtrar lista de crimes e temas">
          <ul class="lista-opcoes" role="listbox" aria-label="Crimes e temas"></ul>
        </div>
      </div>
    </div>
    <div class="count-row">
      <p class="count-line" id="countLine" aria-live="polite">199 decisões</p>
      <button type="button" class="limpar-filtros" id="limparFiltros" hidden>✕ Limpar filtros</button>
    </div>
  </div>
```

**Depois** (inclui a `div#salvasBarra`, que fica logo depois da `div.controls`):

```html
  <div class="controls">
    <div class="search-field">
      <input type="text" id="searchInput" aria-label="Buscar decisões" enterkeyhint="search" placeholder="Buscar por tema, relator, número do processo…" autocomplete="off">
      <button type="button" class="busca-limpar" id="buscaLimpar" aria-label="Apagar a busca" title="Apagar a busca" hidden>✕</button>
      <kbd class="busca-atalho" aria-hidden="true" title="Atalho: tecla /">/</kbd>
    </div>
    <div class="temas-frequentes crimes" id="temasFrequentes" role="group" aria-labelledby="temasRotulo">
      <span class="temas-rotulo" id="temasRotulo">Temas frequentes</span>
    </div>
    <div class="painel-barra">
      <button type="button" class="filtros-toggle" id="filtrosToggle" aria-expanded="false" aria-controls="painelFiltros">Filtros<span class="filtros-qtd" id="filtrosQtd" hidden></span></button>
      <div class="filtros-ativos" id="filtrosAtivos" role="group" aria-label="Filtros ativos" hidden></div>
    </div>
    <div class="filter-row" id="painelFiltros">
      <div class="filter-group" data-filter-group="tribunal" role="group" aria-label="Filtrar por tribunal">
        <span class="label">Tribunal</span>
        <div class="chips">
          <button type="button" class="active" aria-pressed="true" data-value="all"><span class="dot"></span>Todos</button>
          <button type="button" aria-pressed="false" data-value="stj"><span class="dot"></span>STJ</button>
          <button type="button" aria-pressed="false" data-value="stf"><span class="dot"></span>STF</button>
          <button type="button" aria-pressed="false" data-value="trf"><span class="dot"></span>TRFs</button>
          <button type="button" aria-pressed="false" data-value="tj"><span class="dot"></span>TJs</button>
        </div>
      </div>
      <div class="filter-group periodo-filter">
        <label class="label" for="periodoSelect">Período</label>
        <select class="campo-v3" id="periodoSelect">
          <option value="">Qualquer data</option>
        </select>
      </div>
      <div class="filter-divider" aria-hidden="true"></div>
      <div class="filter-group" data-filter-group="decisao" role="group" aria-label="Filtrar por tipo de decisão">
        <span class="label">Decisão</span>
        <div class="segmented">
          <button type="button" class="active" aria-pressed="true" data-value="all">Todas</button>
          <button type="button" aria-pressed="false" data-value="monocratica">Monocrática</button>
          <button type="button" aria-pressed="false" data-value="colegiada">Colegiada</button>
        </div>
      </div>
      <div class="filter-group lista-filter" data-lista="relator" data-atributo="data-relator" data-todos="Todos os relatores" data-busca="Filtrar relator…">
        <span class="label" id="rotulo-relator">Relator</span>
        <div class="lista-controle">
          <button type="button" class="lista-trigger" aria-haspopup="listbox" aria-expanded="false" aria-labelledby="rotulo-relator atual-relator">
            <span class="lista-atual" id="atual-relator">Todos</span>
            <span class="chevron" aria-hidden="true"></span>
          </button>
          <button type="button" class="lista-clear" aria-label="Limpar filtro de relator" title="Limpar" hidden>✕</button>
        </div>
        <div class="lista-pop" hidden>
          <input type="text" class="lista-busca" autocomplete="off" aria-label="Filtrar lista de relatores">
          <ul class="lista-opcoes" role="listbox" aria-label="Relatores"></ul>
        </div>
      </div>
      <div class="filter-group lista-filter" data-lista="crime" data-atributo="data-crimes" data-todos="Todos os crimes e temas" data-busca="Filtrar crime ou tema…">
        <span class="label" id="rotulo-crime">Crime / Tema</span>
        <div class="lista-controle">
          <button type="button" class="lista-trigger" aria-haspopup="listbox" aria-expanded="false" aria-labelledby="rotulo-crime atual-crime">
            <span class="lista-atual" id="atual-crime">Todos</span>
            <span class="chevron" aria-hidden="true"></span>
          </button>
          <button type="button" class="lista-clear" aria-label="Limpar filtro de crime ou tema" title="Limpar" hidden>✕</button>
        </div>
        <div class="lista-pop" hidden>
          <input type="text" class="lista-busca" autocomplete="off" aria-label="Filtrar lista de crimes e temas">
          <ul class="lista-opcoes" role="listbox" aria-label="Crimes e temas"></ul>
        </div>
      </div>
    </div>
    <div class="count-row">
      <p class="count-line" id="countLine" aria-live="polite">199 decisões</p>
      <div class="resultados-acoes" id="resultadosAcoes">
        <button type="button" class="salvas-toggle" id="salvasToggle" aria-pressed="false" hidden>Só salvas<span class="salvas-qtd" id="salvasQtd">0</span></button>
        <label class="ordem-rotulo" for="ordemSelect">Ordenar</label>
        <select class="campo-v3" id="ordemSelect">
          <option value="recentes">Mais recentes</option>
          <option value="antigas">Mais antigas</option>
          <option value="relevancia">Relevância</option>
        </select>
      </div>
      <button type="button" class="limpar-filtros" id="limparFiltros" hidden>✕ Limpar filtros</button>
    </div>
  </div>

  <div class="salvas-barra" id="salvasBarra" hidden>
    <p class="salvas-texto" id="salvasTexto"></p>
    <button type="button" class="salvas-acao" id="salvasCopiar">Copiar lista</button>
    <button type="button" class="salvas-acao" id="salvasEsvaziar">Esvaziar lista</button>
  </div>
```

### C.2 Estado vazio

O `p#emptyState` continua igual; logo depois dele entra `div#vazioSugestoes`.

**Antes:**

```html
  <p class="empty-state" id="emptyState">Nenhuma decisão corresponde à busca ou aos filtros aplicados.</p>

  <footer class="site-footer">
```

**Depois:**

```html
  <p class="empty-state" id="emptyState">Nenhuma decisão corresponde à busca ou aos filtros aplicados.</p>
  <div class="vazio-sugestoes" id="vazioSugestoes" hidden></div>

  <footer class="site-footer">
```

### C.3 Fim da `div.wrap`: aviso curto e "Voltar ao topo"

Entre o `</div>` que fecha a `div.wrap` e o `<dialog>` do leitor entram `div#avisoV3` e `button#voltarTopo`.

**Antes:**

```html
    <p>© 2026 Cury Advogados. Compilado por Ladir Ferreira.</p>
  </footer>

</div>

<dialog class="viewer"
```

**Depois:**

```html
    <p>© 2026 Cury Advogados. Compilado por Ladir Ferreira.</p>
  </footer>

</div>

<div class="aviso-v3" id="avisoV3" role="status" aria-live="polite"></div>
<button type="button" class="topo-v3" id="voltarTopo" aria-label="Voltar ao topo" title="Voltar ao topo" hidden></button>

<dialog class="viewer"
```

### C.4 Leitor de PDF (`dialog#pdfViewer`)

Entre `div.viewer-head` e `div.viewer-body` entram a barra `div#viewerFerramentas` (zoom, página, botão Buscar) e a barra de busca `div#vfBusca`. As duas começam com `hidden`: o v3 mostra a primeira ao iniciar e a segunda ao tocar em Buscar. O resto do leitor não muda.

**Antes:**

```html
<dialog class="viewer" id="pdfViewer" aria-labelledby="viewerTitle">
  <div class="viewer-head">
    <div class="viewer-title">
      <strong id="viewerTitle"></strong>
      <span id="viewerFile"></span>
    </div>
    <a class="viewer-btn primary" id="viewerDownload" href="#" download>Baixar PDF</a>
    <button type="button" class="viewer-btn close" id="viewerClose" aria-label="Fechar">✕</button>
  </div>
  <div class="viewer-body" id="viewerBody" aria-label="Documento PDF"></div>
</dialog>
```

**Depois:**

```html
<dialog class="viewer" id="pdfViewer" aria-labelledby="viewerTitle">
  <div class="viewer-head">
    <div class="viewer-title">
      <strong id="viewerTitle"></strong>
      <span id="viewerFile"></span>
    </div>
    <a class="viewer-btn primary" id="viewerDownload" href="#" download>Baixar PDF</a>
    <button type="button" class="viewer-btn close" id="viewerClose" aria-label="Fechar">✕</button>
  </div>
  <div class="viewer-ferramentas" id="viewerFerramentas" role="toolbar" aria-label="Ferramentas do documento" hidden>
    <div class="vf-grupo" role="group" aria-label="Zoom">
      <button type="button" class="vf-btn vf-menos" id="zoomMenos" aria-label="Diminuir o zoom" title="Diminuir o zoom (tecla -)"></button>
      <button type="button" class="vf-btn vf-zoom" id="zoomValor" title="Ajustar à largura (tecla 0)">100%</button>
      <button type="button" class="vf-btn vf-mais" id="zoomMais" aria-label="Aumentar o zoom" title="Aumentar o zoom (tecla +)"></button>
    </div>
    <p class="vf-pagina" id="vfPagina"></p>
    <button type="button" class="vf-btn vf-buscar" id="vfBuscar" aria-expanded="false" aria-controls="vfBusca" title="Buscar no documento (Ctrl+F)"><span class="vf-rotulo">Buscar</span></button>
  </div>
  <div class="viewer-busca" id="vfBusca" role="search" hidden>
    <input type="text" class="vf-campo" id="vfCampo" placeholder="Buscar no documento" aria-label="Buscar no documento" enterkeyhint="search" autocomplete="off">
    <span class="vf-resultado" id="vfResultado" aria-live="polite"></span>
    <button type="button" class="vf-btn vf-anterior" id="vfAnterior" aria-label="Resultado anterior" title="Anterior (Shift+Enter)"></button>
    <button type="button" class="vf-btn vf-proximo" id="vfProximo" aria-label="Próximo resultado" title="Próximo (Enter)"></button>
    <button type="button" class="vf-btn vf-fechar" id="vfFechar" aria-label="Fechar a busca" title="Fechar a busca (Esc)"></button>
  </div>
  <div class="viewer-body" id="viewerBody" aria-label="Documento PDF"></div>
</dialog>
```

---

## D. Dependências de atributos e nomes

O v3 lê o HTML das decisões sem alterá-lo. O gerador precisa continuar produzindo:

| O que | Usado para | Observação |
|---|---|---|
| `a.pdf-link[data-pdf]` (o primeiro que não é `.info`) | identificação da decisão: link direto `?id=…` e lista de salvas | o `data-pdf` precisa continuar único e estável (mudar o ID de uma decisão quebra links e salvas antigos) |
| decisão sem PDF | id de reserva tirado de `data-tribunal` e do texto do `.cite` (ex.: `stf-are-1585074-sp`) | hoje só a ARE 1.585.074 |
| `p.meta` com "Publicado em DD/MM/AAAA" | ordem (recentes/antigas) e Período | sem essa frase, a decisão fica com data 0 (vai para o fim) e fora dos anos |
| `<template class="ementa">`: última linha não vazia entre parênteses | "Copiar referência" e a lista copiada | sem parênteses, o botão não aparece naquela decisão |
| texto da `<template class="ementa">` | busca na ementa e trecho "Na ementa" | lido uma vez ao abrir a página |
| `.cite` com `span.tribunal-tag`, o texto do recurso e, às vezes, `span.resultado-tag` | títulos ("STJ · AgRg no RHC nº … · Recurso desprovido") no link, nas salvas e na lista | mesma estrutura que o leitor de PDF já usava |
| `data-search`, `data-crimes`, `data-relator`, `data-decisao`, `data-tribunal` | busca, contagens, temas frequentes | os mesmos de antes |
| `.pdf-link.copiar` seguido da `<template class="ementa">` | "Copiar referência" entra logo depois desse botão | o botão novo usa a classe `ref-v3`, não `copiar` |
| `[data-filter-group="tribunal"] button[data-value]` | quantidade em cada ficha de Tribunal (um `span.qtd-v3` é acrescentado dentro do botão) | |
| classe `crimes` em `div#temasFrequentes` | o clique nas fichas de temas usa o tratamento do script principal (`.crimes button[data-crime]`), e a ficha escolhida ganha `.ativo` como as do cartão | se o script principal mudar esse seletor, os temas frequentes param de filtrar |
| ids `searchInput`, `countLine`, `emptyState`, `limparFiltros`, `pdfViewer`, `viewerBody`, `viewerFile` | já existiam; o v3 usa todos | |
| `localStorage` `acervo-salvas` | lista de salvas (novo; `acervo-tema` continua igual) | sempre com try/catch |
| parâmetros do endereço `ordem`, `periodo`, `id` | novos, ao lado de `q`, `tribunal`, `decisao`, `relator`, `crime`, `pagina` | `id` sai do endereço depois de usado |
| `ENDERECO_PUBLICO = 'https://ladirf.github.io/acervo/'` (no `recursos-v3`) | link copiado fora do site (HTML do Drive aberto do computador, ou dentro do claude.ai) | trocar aqui se o endereço do site mudar |
| CSS Custom Highlight API | destaque dos termos (cartões e PDF) | Safari 17.2+; em navegador mais antigo a busca funciona sem o destaque (no PDF, marca o trecho inteiro) |

---

## E. O que conferir depois de levar ao arquivo-mestre

Testado na nuvem com Chromium (Playwright), em 1280, 390 e 375 px e no tema escuro: 59 verificações dos comportamentos de antes e 190 dos recursos novos, sem erros no console. Também foi testado um HTML no formato do Drive (PDFs embutidos em base64, sem a pasta `pdfs/` e sem o índice `pdf-nomes`, aberto do disco): o PDF chega por `obterPdf()`, sem nenhum pedido de rede. O Safari do iPhone não pôde ser testado na nuvem.

No HTML do Drive (PDFs embutidos):

- [ ] "Abrir PDF" abre, com o nome do arquivo e o número de páginas no cabeçalho;
- [ ] − / + e o valor (ajustar à largura) mudam o zoom; no iPhone, a pinça amplia só o PDF e ele fica nítido ao soltar;
- [ ] "Buscar" no leitor acha termos sem acento ("diligencias"), com "3 de 12", anterior e próximo; Esc fecha só a busca;
- [ ] "Baixar PDF" continua baixando o arquivo embutido;
- [ ] o botão de link copia um endereço `https://ladirf.github.io/acervo/?id=…`.

No site e no HTML:

- [ ] busca: "pronuncia" acha "pronúncia"; "210.983" e "210983" acham a mesma decisão; termo só na ementa mostra "Na ementa: …";
- [ ] com busca, a ordem passa para Relevância e volta para Mais recentes ao apagar; escolhida à mão, fica;
- [ ] Período, "Só salvas", Esc e "✕ Limpar filtros" (Esc limpa tudo, inclusive Período e "Só salvas");
- [ ] no celular: filtros recolhidos atrás de "Filtros (N)", fichas dos filtros ligados, temas frequentes rolando de lado;
- [ ] `?id=pdf-hc-1109495` abre na página certa e destaca o cartão;
- [ ] contagem "52 decisões · página 2 de 4" e "X de Y decisões"; filtros no endereço (`?crime=Infanticídio`);
- [ ] tema escuro; "Copiar ementa" e "Copiar referência".

## F. Como voltar atrás

Apagar os blocos `<style id="design-v3">` e `<script id="recursos-v3">` e devolver a marcação da seção C ao "antes". Os ganchos da seção A podem ficar: sem o bloco v3 eles não fazem nada.
