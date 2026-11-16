# Configuração do painel do profissional

O painel (`painel.html`) usa o Firebase do Google para guardar os pedidos e para o login com e-mail e senha.
Sem isso, o site continua funcionando normal e envia o orçamento pelo WhatsApp, mas **não registra** os pedidos no painel.

## 1. Projeto e aplicativo
O projeto Firebase `edm-pedreiro` e o app web `EDM Construção Site` já foram criados. O arquivo `config.js` contém o `apiKey` e o `projectId` públicos do app. A autorização dos dados é feita pelas regras do Firestore abaixo.

## 2. Banco de dados
O Cloud Firestore já está criado em modo de produção na região `southamerica-east1` (São Paulo), e as regras abaixo já foram publicadas. O acesso administrativo está restrito a `edmpedreiros@gmail.com`.

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
        && request.auth.token.email == 'edmpedreiros@gmail.com';
    }

    // Agendamento de visitas
    // "slots" guarda só o horário ocupado (sem dados pessoais) e é público para o site mostrar o que está lotado.
    // O id do documento (ex.: 2026-11-21_0900) impede dois clientes de reservarem o mesmo horário.
    match /slots/{slot} {
      allow read: if true;
      allow create: if slot.matches('^20[0-9]{2}-[01][0-9]-[0-3][0-9]_(0900|1100|1300)$')
        && request.resource.data.keys().hasOnly(['data','hora','criadoEm']);
      allow delete: if request.auth != null && request.auth.token.email == 'edmpedreiros@gmail.com';
    }
    // "visitas" tem os dados do cliente: qualquer visitante cria, só o profissional lê, edita e apaga
    match /visitas/{id} {
      allow create: if request.resource.data.keys().hasOnly(
          ['chave','data','hora','nome','whatsapp','endereco','observacao','mensagem','status','criadoEm'])
        && request.resource.data.nome is string && request.resource.data.nome.size() < 200
        && request.resource.data.mensagem is string && request.resource.data.mensagem.size() < 3000
        && request.resource.data.status == 'novo';
      allow read, update, delete: if request.auth != null
        && request.auth.token.email == 'edmpedreiros@gmail.com';
    }
  }
}
```

## 3. Login do profissional
O provedor E-mail/senha está ativado e o cadastro público está desabilitado. Falta criar o usuário em **Authentication > Usuários > Adicionar usuário**: use `edmpedreiros@gmail.com` e defina uma senha nova e exclusiva diretamente no Firebase Console; não a compartilhe pelo chat.

## 4. Usar
O painel tem duas abas: **Orçamentos** e **Visitas agendadas** (as visitas marcadas no calendário do site). Ao cancelar uma visita, o horário volta a ficar livre no site.

Abra `https://SEU-SITE/painel.html`, entre com o e-mail e a senha. Cada clique em
"Enviar Solicitação via WhatsApp" cria um pedido no painel na hora, mesmo que o cliente
desista de enviar a mensagem. O painel atualiza sozinho a cada 1 minuto.

Ao cancelar uma visita pelo painel, o registro é mantido como **Cancelada**, o horário é liberado e o WhatsApp abre uma conversa com o cliente e a mensagem de desmarcação preenchida. Para concluir o envio, o profissional ainda precisa tocar em **Enviar** no WhatsApp. Se o navegador bloquear a nova aba, use o botão **Avisar cancelamento no WhatsApp** no histórico de visitas.
