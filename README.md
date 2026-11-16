# EDM Construção | Obras & Acabamentos

Site institucional e de captação de clientes da **EDM Construção**, empresa de construção civil, reformas e acabamentos em Nova Friburgo e região (RJ). O objetivo é transformar visitantes em pedidos de orçamento e visitas, direto no WhatsApp.

- Instagram: [@edmconstrucao](https://www.instagram.com/edmconstrucao/)
- Desenvolvido por: [@vcrk.digital](https://www.instagram.com/vcrk.digital/)

## Sumário

- [Funcionalidades](#funcionalidades)
- [Tecnologias](#tecnologias)
- [Estrutura do projeto](#estrutura-do-projeto)
- [Como rodar localmente](#como-rodar-localmente)
- [Configuração](#configuração)
- [Painel do profissional](#painel-do-profissional)
- [Publicação (GitHub Pages)](#publicação-github-pages)
- [Como personalizar](#como-personalizar)
- [Observações](#observações)

## Funcionalidades

**Site (`index.html`)**
- Hero com casa 3D que é construída conforme a rolagem da página (Three.js + GSAP).
- Seções: Sobre, Serviços, Orçamento, Agendamento de visita, Portfólio (galeria, antes e depois e obras entregues), Por que me contratar e Contato.
- **Orçamento online:** o cliente marca os serviços (29 itens em 6 grupos), preenche nome, WhatsApp, e-mail, segmento, metragem, local, prazo para começar e observações. Tudo é enviado em uma mensagem organizada para o WhatsApp.
- **Agendamento de visitas:** calendário em que só os sábados ficam disponíveis, em blocos de 2 horas (09:00, 11:00 e 13:00, terminando às 15:00). Horários já reservados somem para os outros clientes. Há um botão de WhatsApp para quem não consegue aos sábados.
- **Obras entregues** com foto, endereço e link para o Google Maps.
- Botão flutuante de WhatsApp, menu que escurece aos poucos com a rolagem e layout responsivo (testado de 320 px a 1440 px).

**Painel do profissional (`painel.html`)**
- Acesso restrito com e-mail e senha.
- Aba **Orçamentos:** todos que clicaram em enviar o orçamento, mesmo quem não concluiu o envio no WhatsApp.
- Aba **Visitas agendadas:** próximas e anteriores, com opção de confirmar pelo WhatsApp, marcar como realizada ou cancelar. Ao cancelar, o horário é liberado, o registro fica no histórico e uma mensagem de desmarcação é preparada no WhatsApp.
- Busca, filtros e destaque para pedidos urgentes.

## Tecnologias

- HTML, CSS e JavaScript puros, em um site estático, sem etapa de build.
- [Three.js](https://threejs.org/) (r128) e [GSAP](https://gsap.com/) + ScrollTrigger (3.12.5), carregados por CDN.
- [Firebase](https://firebase.google.com/) (Firestore e Authentication, via API REST) para pedidos, horários de visita e login do painel. É opcional: sem ele, o site funciona e envia tudo pelo WhatsApp.

## Estrutura do projeto

```
.
├── index.html       # Site completo (HTML, CSS e JS em um único arquivo)
├── painel.html      # Painel do profissional (login, orçamentos e visitas)
├── config.js        # Chaves públicas do Firebase (apiKey e projectId)
├── SETUP.md         # Passo a passo da configuração do Firebase e regras de segurança
├── fotos/           # Imagens das obras entregues e do profissional
│   ├── obra-xingu.jpg
│   ├── obra-sitio.jpg
│   └── profissional-edm.jpg
└── README.md
```

## Como rodar localmente

Não precisa instalar nada. Basta abrir o `index.html` no navegador, ou subir um servidor local simples:

```bash
git clone https://github.com/Vitor192/EDM-Pedreiro.git
cd EDM-Pedreiro
python3 -m http.server 8000
# acesse http://localhost:8000
```

É preciso internet para carregar as bibliotecas 3D e de animação (CDN).

## Configuração

### WhatsApp

O número fica no início do script do `index.html`:

```js
const WA_NUMBER = "5522981008988";   // formato: 55 + DDD + número, só dígitos
const WA_DISPLAY = "(22) 98100-8988";
```

O agendamento de visitas usa o mesmo número (constante `WA` no bloco de agendamento). Se trocar o número, altere nos dois lugares e no rodapé.

### Firebase (painel e reserva de horários)

1. Siga o passo a passo do [`SETUP.md`](SETUP.md): criar o projeto, o Firestore, as regras de segurança e o usuário do profissional.
2. Preencha o `config.js`:

```js
window.EDM_FB = {
  apiKey: "SUA_API_KEY",
  projectId: "SEU_PROJECT_ID"
};
```

> A `apiKey` do Firebase para web **não é um segredo**. A proteção vem das regras do Firestore e do login. Nunca publique senhas nem chaves de administrador.

Enquanto o `config.js` estiver vazio, o site envia tudo pelo WhatsApp, mas **não** registra pedidos no painel nem trava horários de visita.

## Painel do profissional

Acesse `https://SEU-SITE/painel.html` e entre com o e-mail e a senha criados no Firebase (Authentication). A página não é indexada por buscadores (`noindex`) e não tem link no site.

## Publicação (GitHub Pages)

1. No repositório, vá em **Settings > Pages**.
2. Em **Build and deployment**, escolha **Deploy from a branch**, branch `main` e pasta `/ (root)`.
3. Salve. O site fica em `https://vitor192.github.io/EDM-Pedreiro/` (pode levar alguns minutos).

Depois de publicar, teste um envio de orçamento e um agendamento para confirmar que chegam no WhatsApp e no painel.

## Como personalizar

| O que mudar | Onde |
| --- | --- |
| Serviços do orçamento (nomes e descrições) | lista `CATS` no script do `index.html` |
| Obras entregues (foto, endereço, link do Maps) | lista `OBRAS` no script do `index.html` |
| Dias e horários de visita | constantes `SLOTS` e `END` no bloco de agendamento, e a regra `slots` em `SETUP.md` |
| Textos, estatísticas e contatos | seções `#sobre` e `footer` no `index.html` |
| Cores | variáveis `--or`, `--bg`, `--bg2`, `--bg3` no início do CSS |
| Fotos das obras | pasta `fotos/` (use imagens leves, de até cerca de 200 KB) |

## Observações

- Os horários de visita seguem o fuso do navegador do cliente (Brasil).
- O arquivo `index.html` é grande (cerca de 1,5 MB) porque algumas imagens do portfólio estão embutidas nele. As imagens das obras entregues ficam em `fotos/`.
- Fotos com marca d'água de terceiros (como prints do Street View) devem ser trocadas por fotos próprias das obras.

## Licença e direitos

© 2026 EDM Construção | Obras & Acabamentos. Todos os direitos reservados.
