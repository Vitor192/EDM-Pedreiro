# Configuração do painel do profissional (uma vez só, ~10 min, grátis)

O painel (`painel.html`) usa o Firebase do Google para guardar os pedidos e para o login com e-mail e senha.
Sem isso, o site continua funcionando normal e envia o orçamento pelo WhatsApp, mas **não registra** os pedidos no painel.

## 1. Criar o projeto
1. Acesse https://console.firebase.google.com e clique em **Adicionar projeto** (nome: `edm-pedreiro`; pode desativar o Google Analytics).
2. Em **Visão geral do projeto > ⚙ Configurações do projeto > Geral**, em "Seus apps", clique em **</> Web**, dê um apelido e registre.
3. Copie `apiKey` e `projectId` para o arquivo `config.js` deste projeto.

## 2. Banco de dados
1. **Criação > Firestore Database > Criar banco de dados** (modo produção, região `southamerica-east1` / São Paulo).
2. Aba **Regras**: cole o texto abaixo, troque `SEU_EMAIL_AQUI` pelo e-mail do profissional e clique em **Publicar**.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /leads/{id} {
      // qualquer visitante pode CRIAR um pedido (com limites); só o profissional lê, edita e apaga
      allow create: if request.resource.data.keys().hasOnly(
          ['nome','whatsapp','email','segmento','local','metragem','servicos','objetivo','prazo','mensagem','status','criadoEm'])
        && request.resource.data.nome is string && request.resource.data.nome.size() < 200
        && request.resource.data.mensagem is string && request.resource.data.mensagem.size() < 6000
        && request.resource.data.status == 'novo';
      allow read, update, delete: if request.auth != null
        && request.auth.token.email == 'SEU_EMAIL_AQUI';
    }

    // Agendamento de visitas
    // "slots" guarda só o horário ocupado (sem dados pessoais) e é público para o site mostrar o que está lotado.
    // O id do documento (ex.: 2026-11-21_0900) impede dois clientes de reservarem o mesmo horário.
    match /slots/{slot} {
      allow read: if true;
      allow create: if slot.matches('^20[0-9]{2}-[01][0-9]-[0-3][0-9]_(0900|1100|1300)$')
        && request.resource.data.keys().hasOnly(['data','hora','criadoEm']);
      allow delete: if request.auth != null && request.auth.token.email == 'SEU_EMAIL_AQUI';
    }
    // "visitas" tem os dados do cliente: qualquer visitante cria, só o profissional lê, edita e apaga
    match /visitas/{id} {
      allow create: if request.resource.data.keys().hasOnly(
          ['chave','data','hora','nome','whatsapp','endereco','observacao','mensagem','status','criadoEm'])
        && request.resource.data.nome is string && request.resource.data.nome.size() < 200
        && request.resource.data.mensagem is string && request.resource.data.mensagem.size() < 3000
        && request.resource.data.status == 'novo';
      allow read, update, delete: if request.auth != null
        && request.auth.token.email == 'SEU_EMAIL_AQUI';
    }
  }
}
```

## 3. Login do profissional
1. **Criação > Authentication > Vamos começar > Método de login > E-mail/senha** → ativar.
2. Aba **Usuários > Adicionar usuário**: e-mail e senha que o profissional vai usar.
3. Aba **Configurações > Ações do usuário**: desmarque **Ativar criação (cadastro)** para ninguém mais criar conta.

## 4. Usar
O painel tem duas abas: **Orçamentos** e **Visitas agendadas** (as visitas marcadas no calendário do site). Ao cancelar uma visita, o horário volta a ficar livre no site.

Abra `https://SEU-SITE/painel.html`, entre com o e-mail e a senha. Cada clique em
"Enviar Solicitação via WhatsApp" cria um pedido no painel na hora, mesmo que o cliente
desista de enviar a mensagem. O painel atualiza sozinho a cada 1 minuto.

Ao cancelar uma visita pelo painel, o registro é mantido como **Cancelada**, o horário é liberado e o WhatsApp abre uma conversa com o cliente e a mensagem de desmarcação preenchida. Para concluir o envio, o profissional ainda precisa tocar em **Enviar** no WhatsApp. Se o navegador bloquear a nova aba, use o botão **Avisar cancelamento no WhatsApp** no histórico de visitas.
