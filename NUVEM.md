# Branch `nuvem` — trabalho em sessões na nuvem do Claude Code

Esta branch existe para mudanças mais pesadas feitas numa sessão na nuvem. O site
publicado (GitHub Pages) sai **só da `main`**, então nada aqui vai ao ar sem merge.

## Atenção: estes arquivos são gerados

`index.html`, `pdfs/` e os `.md` são **gerados** a partir do arquivo-mestre do
acervo, que fica no computador do usuário (não está neste repositório). A cada
lote novo, o gerador sobrescreve tudo na `main`.

Por isso:

* Mudanças de **visual e funcionamento** (CSS e JavaScript do `index.html`) devem
  ficar em commits pequenos e bem descritos: a sessão local leva o mesmo código
  para o arquivo-mestre antes do merge.
* **Não editar as decisões** (`<li class="entry">`, ementas, PDFs, `.md`): o
  conteúdo vem do arquivo-mestre e seria perdido na próxima publicação.
* Testar servindo a pasta por HTTP (`python -m http.server`) — os PDFs abrem de
  `pdfs/`.
* Português, Century Gothic, tema claro por padrão, cores do tribunal nos cartões,
  bloco `<style id="design-v2">` com o visual atual (cabeçalho preto e dourado).
