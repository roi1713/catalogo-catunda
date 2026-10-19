# Catálogo Catunda

Site estático de venda direta da coleção da avó do Rodrigo (quadros, objetos, livros, um móvel).
Substitui o site antigo feito no ChatGPT (colecao-particular-arte-rodrigo.diboa-tabacaria.chatgpt.site).
Retirada em Pinheiros, SP. Contato: WhatsApp (11) 97212-7766.

## Estrutura

```
index.html      página única, HTML/CSS/JS puro, sem build. Lê items.json via fetch.
items.json      todos os itens. É a única coisa que muda no dia a dia.
img/t/<SKU>-1.jpg   miniatura (560 px) — só da 1ª foto de cada item, usada na grade
img/g/<SKU>-N.jpg   fotos da ficha (1300 px, JPEG q74), N = 1, 2, 3...
```

Não há framework nem dependência. Fontes vêm do Google Fonts. Para testar localmente:
`python -m http.server` dentro da pasta e abrir http://localhost:8000 (abrir o arquivo direto não funciona por causa do fetch).

## items.json

```json
{ "updated": "2026-10-13", "items": [ {
  "sku": "QDR-004",              // código. Prefixos: QDR quadros, OBJ objetos, LIV livros, MOV móveis
  "cat": "quadros",              // quadros | objetos | livros | moveis
  "sub": "Pinturas",             // subcategoria (filtro). Quadros: Pinturas, Gravuras e papel, Relevos e entalhes, Reproduções.
                                 // Objetos: Madeira, Cristais, Porcelanas, Estanho, Metais prateados, Câmeras e ópticos, Diversos
  "title": "...", "artist": "...", "technique": "...", "condition": "...", "dims": "35 × 27 cm",
  "price": 1260,                 // venda direta em R$. null = "Preço a combinar"
  "priceNote": null,             // texto opcional no lugar do preço do ML (ex.: "8 pequenas R$ 120 · 8 grandes R$ 180")
  "mlPrice": 1690,               // preço de referência no Mercado Livre (só aparece se maior que price)
  "status": "available",         // available | sold | hidden
  "desc": "...",
  "images": ["QDR-004-1", "QDR-004-2"],   // nomes sem extensão; a 1ª precisa ter miniatura em img/t
  "links": [{"label": "Ver no Mercado Livre", "url": "https://..."}]   // opcional
} ] }
```

- **Vendido**: `status: "sold"`. Sai da grade; aparece só com "Mostrar vendidos" ligado.
- **Esconder sem apagar**: `status: "hidden"`. Nunca aparece.
- Ao mudar qualquer coisa, atualizar `updated` (aparece no rodapé).
- Item novo: próximo número livre do prefixo. Gerar `img/g/<SKU>-N.jpg` (lado maior 1300 px, q74) e `img/t/<SKU>-1.jpg` (560 px, q74), girando pela EXIF. Fotos de iPhone chegam em HEIC: converter com pillow-heif.
- Fotos originais ficam no Google Drive em `G:\Meu Drive\02_Catunda` (Quadros, Resta, novas) e em `C:\Users\roi17\Desktop\LOJA CATUNDA\JPG`.

## Pendências (em 13/10/2026)

Itens com `status: "hidden"` esperando o Rodrigo confirmar se ainda existem:
OBJ-003 gamela (falta foto), OBJ-080 chaleira prateada, OBJ-081 manteigueira, OBJ-082 leiteira,
OBJ-083 conjunto de saquê, OBJ-084 vaso oriental, OBJ-085 pote oriental com tampa.

Itens sem preço (`price: null`), Rodrigo define: OBJ-014, 026–031, 046, 047, 070–072.
OBJ-013 (flûtes): confirmar quantidade.

## Publicação

Site público em GitHub Pages (repositório `roi1713/catalogo-catunda`, branch `main`, raiz). O gh do agente está logado como `diboatabacaria-ctrl` com acesso de escrita (sem admin).
Fluxo de atualização: editar items.json / imagens → commit → push. O Pages publica sozinho em ~1 min.
O Rodrigo não usa terminal: o agente roda git/gh por ele.

Existe também uma prévia como Artifact no claude.ai (Catálogo Catunda), privada.

## Painel de edição (admin.html)

`admin.html` (mesma pasta, `noindex`) permite ao Rodrigo editar preços/situação, trocar fotos, adicionar e excluir peças sem usar o agente.
Ele fala direto com a API do GitHub (Git Data API) usando um token fine-grained do dono, com permissão Contents: read/write só neste repo, guardado no localStorage do navegador dele. Nada de token no repositório.
Ao publicar faz um único commit: regrava `items.json` (indentação 1, sem quebra final, igual ao arquivo atual), envia `img/g` e `img/t` novos (redimensiona no navegador: 1300 px / 560 px, q74) e apaga fotos órfãs. Se o `items.json` mudou no repo desde que o painel abriu (ex.: commit do agente), ele recusa publicar e pede para recarregar.
Ao editar `items.json` pelo agente, manter esse formato (`JSON.stringify(d,null,1)`) para não gerar ruído.
