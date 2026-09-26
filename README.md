# Site com Efeito Parallax

Demonstração de efeito parallax em **HTML e CSS puros**, sem uma linha de JavaScript.
Derivado do [HTML5 Boilerplate 6.0.1](https://html5boilerplate.com/), enxuto ao ponto de só
sobrar o que a página realmente usa.

## Como o parallax funciona

O efeito inteiro está em **uma propriedade**:

```css
.pimg1, .pimg2, .pimg3, .pimg4 {
    position: relative;
    opacity: 0.70;
    background-position: center;
    background-size: cover;
    background-repeat: no-repeat;
    background-attachment: fixed;   /* <- isto aqui */
}
```

Cada painel é uma `div` com uma imagem de fundo. Com `background-attachment: fixed`, a
imagem fica **presa à viewport** enquanto o elemento rola, produzindo a sensação de
profundidade. Troque por `scroll` e o efeito desaparece — a imagem passa a rolar junto.

Não há listener de scroll, `requestAnimationFrame` nem cálculo de `transform`. O navegador
faz o trabalho.

O texto da legenda é posicionado por cima de cada painel:

```html
<div class="pimg1">
    <div class="ptext"><span class="border">Texto da 1ª imagem</span></div>
</div>
```

`.ptext` usa `position: absolute` e depende do `position: relative` do `.pimgN`. Sem esse
`relative` no painel, as quatro legendas se empilham no mesmo lugar.

## Por que no celular o parallax é desligado

```css
@media (max-width: 568px) {
    .pimg1, .pimg2, .pimg3, .pimg4 {
        background-attachment: scroll;
    }
}
```

Em telas pequenas, `fixed` causa Problemas de rolagem em navegadores móveis e consome
 bateria. O `scroll` é a alternativa.

## A lição: não use prefers-reduced-motion para isso

Um detalhe que parece óbvio e está **errado**: desligar o parallax com
`prefers-reduced-motion: reduce` não funciona na prática.

O Windows com animações desligadas — configuração muito comum — faz o Chromium considerar
que a pessoa pede movimento reduzido. Resultado: o site ficaria **sem parallax na maioria
dos computers**, para quem nem tinha problema com isso. A media query certa é a de largura,
não a de preferência.

## O bug do `.pmig` — a lição mais útil deste projeto

Este site já foi publicado com um erro de uma letra. A lista de seletores compartilhados
estava assim:

```css
/* ERRADO */
.pmig1, .pmig2, .pmig3, .pmig4 { ... }
```

`pmig` em vez de `pimg`. O bloco não casava com **nada** no HTML, e o navegador não reclamou
— regras que não casam com elemento nenhum são silenciosamente descartadas.

O resultado: cada `.pimgN` ficou só com `background-image` e `min-height`. Perdeu-se
`background-size: cover`, `background-attachment`, `opacity` e `position: relative` de uma
vez. As imagens apareceram ladrilhadas no tamanho natural, sem parallax, e as quatro legendas
se sobrepuseram no mesmo ponto da tela.

A página ainda **parecia** plausível o suficiente para passar. Uma letra matou seis
propriedades, e nada no console reclamou.

> Se o parallax parar de funcionar, confera primeiro se a lista de seletores compartilhados
> realmente casa com as classes do HTML.

## Como rodar

Não há build, dependências, gerenciador de pacotes nem servidor de desenvolvimento.

```bash
git clone https://github.com/cirleno/basic-parallax-website.git
cd basic-parallax-website
```

Abra o `index.html` no navegador. Só isso.

O `css/` é cacheado por 1 ano pelo `.htaccess` e não há versionamento de arquivo nem
cache-busting, então depois de editar o CSS pode ser preciso **hard reload** (Ctrl+Shift+R).
Alterações no HTML aparecem na hora.

## Estrutura

```
index.html              a página única
404.html
css/
  style.css             todo o CSS: reset de 3 linhas, o projeto, seleção,
                        .visuallyhidden e o bloco de impressão
img/
  Hello_World.webp      fundo do .pimg1
  Gramafone.webp        fundo do .pimg2
  Vinil_playing.webp    fundo do .pimg3
  F40_black.webp        fundo do .pimg4
  Brasil_Progresso.jpg  fonte de todos os ícones
favicon.ico             16x16 + 32x32, dois quadros PNG dentro de um ICO
icon.png                192x192, PWA
icon-512.png            512x512, PWA
icon-maskable-512.png   512x512, ícone adaptativo do Android
apple-touch-icon.png    180x180, iOS
mstile-*.png            tiles do Windows (70, 150, 310)
tile-wide.png           558x270, tile largo do Windows
site.webmanifest        manifesto PWA
browserconfig.xml       tiles do Windows
```

### O reset: uma linha em vez de um arquivo

O projeto **não usa nenhum stylesheet de terceiros**. O `normalize.css` v7 foi substituído por
uma regra só, no topo do `style.css`:

```css
html {
    -webkit-text-size-adjust: 100%;
}
```

A auditoria que motivou a decisão cruzou as ~30 regras do normalize com os elementos que a página
realmente usa (`a body div h1 h2 main p section span`):

| regra do normalize.css | o que acontece na prática |
|---|---|
| `body { margin: 0 }` | duplicata — o bloco do projeto já define |
| `html { line-height: 1.15 }` | sobrescrito pelo `html { line-height: 1.4 }` do projeto |
| `section`, `main` → `display: block` | já é o default do navegador |
| `h1 { font-size: 2em; margin: .67em 0 }` | o `<h1>` é `.visuallyhidden` — clipped, `margin: -1px` |
| `a { background-color: transparent }` | o `.skip-link` já define `#111` |
| `img { border-style: none }` | não existe `<img>` no `index.html` |
| 164 linhas de formulários | não existe um único `<input>`, `<button>` ou `<textarea>` — 37% do arquivo |
| o resto (tabelas, mídia, listas, `hr`, `pre`, `details`, `[hidden]`…) | nenhum desses elementos existe |
| **`html { -webkit-text-size-adjust: 100% }`** | **a única regra viva** — anti-inflação de texto no iOS em paisagem |

As 7 regras da base do H5BP (`hr`, `fieldset`, `textarea`, media, `html`) foram pelo mesmo
caminho — mortas nesta página. `::selection` foi preservada: ela é o único detalhe de UX que o
boilerplate realmente entrega aqui.

Ganho: **-8.940 bytes crus, -2.595 bytes gzip, uma requisição a menos** — o payload de CSS foi de
14.314 para 5.374 bytes e de 4.548 para 1.953 gzip. Parte desse ganho é remover código morto,
parte é o dicionário de compressão agora ser compartilhado num arquivo só.

Isso **não** é uma otimização de performance. As quatro imagens somam 276 KB e respondem por
94,9% dos 284 KB que o navegador baixa, então o CSS é cerca de 1,8% do peso da página. O motivo
é manutenção: 447 linhas de `normalize.css` mais as regras base do H5BP eram passivo, não
ativo. As regras do próprio projeto não mudaram — a limpeza removeu código de terceiros e
regras sem elemento correspondente, nada mais.

Se um dia entrar um `<form>`, `<table>`, `<img>` ou áudio, um reset vendorizado passa a ser
justificável — nesse caso, adicione-o de propósito e avise. Não volte por reflexo.

Para adicionar um painel novo, o `.pimgN` precisa aparecer em **três** lugares: o bloco HTML,
a regra de `background-image` em `style.css`, e as **duas** listas de seletores
compartilhados em `style.css` (a base e a dentro da media query de 568px). Esquecer a
terceira lista e o painel continua aparecendo no celular com o efeito ligado.

## Acessibilidade

- `<html lang="pt-BR">` e `lang` também no manifesto
- `<h1>` com a mesma string do `<title>`, para leitores de tela anunciarem uma coisa só
- link "Pular para o conteúdo" como primeiro elemento focável
- hierarquia `H1, H2, H2, H2` — um `h1` só
- anel de foco visível no skip-link
- a página **não tem um único elemento `<img>`**: as quatro fotos são `background-image`, que é
  decorativo por definição e não aceita `alt`. Todo o texto visível está em `.ptext .border` e
  nos `<h2>`, então nada depende de texto alternativo. (Se um `<img>` entrar, ele precisa de `alt`.)

## Compatibilidade

- **WebP** para os fundos, declarados como `background-image` simples, sem `<picture>` nem
  fallback em PNG. Suportado em todos os navegadores atuais.
- **PNG dentro de ICO** no favicon, que preserva os pixels exatos sem recomprimir. Os dois
  quadros (16x16 e 32x32) são PNG embutidos literalmente, não DIB re-codificado.

## Estado

Projeto de estudo. **As três seções de texto ainda contêm Lorem ipsum** de exemplo — não
são conteúdo real. As imagens de fundo e o material do ícone são pessoais.

## Licença

[HTML5 Boilerplate](https://github.com/h5bp/html5-boilerplate), MIT. O aviso está em
`LICENSE.txt` e precisa acompanhar qualquer cópia, porque o MIT exige que o copyright
original seja preservado.
