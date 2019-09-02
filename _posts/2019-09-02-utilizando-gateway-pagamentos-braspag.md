---
layout: post
title:  "Série: utilizando gateway de pagamentos Braspag - parte 1"
author: martin
categories: [ Payment Gateways ]
image: assets/images/posts/moeda-real-low-wide.jpg
image-credits: <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@ojsant?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from ORLANDO SANT&#x27;ANNA"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">ORLANDO SANT&#x27;ANNA</span></a>
featured: true
hidden: true
---

Nessa série de posts vou compartilhar com vocês a árdua tarefa de se configurar um gateway de pagamentos para um site no Brasil utilizando o [**Braspag**](https://www.braspag.com.br/), uma solução 100% tupiniquim de gateway de pagamentos. Além de poder oferecer cobrança por boletos, evita-se a taxação de IOF quando cartões de crédito Brasileiros são utilizados. Para quem vai fazer um negócio no exterior voltado para pessoas que morem e tenham cartões de crédito lá, talvez seja mais interessante uma alternativa como [Stripe](https://stripe.com), que possui taxas menores por operação.

### Cadastro na API *sandbox* do Braspag
Antes de mais nada, vale esclarecer que *sandbox* nada mais é do que um ambiente de testes; ele funciona exatamente da mesma forma que um ambiente de produção, mas as operações executadas nele - por exemplo, transferências de dinheiro - não acarretam operações financeiras no mundo real. Então ~~cagad~~ erros na programação não vão criar prejuízos e pode-se testar a API quanto tempo quiser, sem custos.

Para utilizar a API *sandbox*, acesse o [site de cadastro](https://cadastrosandbox.braspag.com.br/). **Não é necessário entrar com dados reais** - apesar de que usar um mail existente talvez não seja má idéia. Vai, pode colocar o seu telefone (99)99999-9999.

Feito o cadastro, serão criados para você um *MerchantId* e um *MerchantKey*; guarde esses identificadores pois irá precisar deles mais tarde para comunicar com a API.