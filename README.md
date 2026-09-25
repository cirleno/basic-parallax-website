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
  main.css              CSS do projeto (bloco "Author's custom styles")
  normalize.css         reset vendorizado
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

Para adicionar um painel novo, o `.pimgN` precisa aparecer em **três** lugares: o bloco HTML,
a regra de `background-image` em `main.css`, e as **duas** listas de seletores
compartilhados em `main.css` (a base e a dentro da media query de 568px). Esquecer a
terceira lista e o painel continua aparecendo no celular com o efeito ligado.

## Acessibilidade

- `<html lang="pt-BR">` e `lang` também no manifesto
- `<h1>` com a mesma string do `<title>`, para leitores de tela anunciarem uma coisa só
- link "Pular para o conteúdo" como primeiro elemento focável
- hierarquia `H1, H2, H2, H2` — um `h1` só
- anel de foco visível no skip-link
- `alt` em todas as imagens

## Compatibilidade

O site depende de dois formatos com requisitos de data similar:

- **WebP** para os fundos, declarados como `background-image` simples, sem `<picture>` nem
  fallback em PNG. Suportado em todos os navegadores atuais.
- **PNG dentro de ICO** no favicon (Vista/IE11 ou superior), que preserva os pixels exatos
  sem recomprimir.

## Estado

Projeto de estudo. **As três seções de texto ainda contêm Lorem ipsum** de exemplo — não
são conteúdo real. As imagens de fundo e o material do ícone são pessoais e foram editados
para remover uma faixa branca que vinha embutida nas artes.

## Licença

[HTML5 Boilerplate](https://github.com/h5bp/html5-boilerplate), MIT. O aviso está em
`LICENSE.txt` e precisa acompanhar qualquer cópia, porque o MIT exige que o copyright
original seja preservado.
