# Guapeco · Analisador de Contratos

Ferramenta para o time de vendas subir contratos comentados pelo jurídico de leads e
receber automaticamente: (1) um resumo Aceitar/Contrapor/Rejeitar e (2) o mesmo .docx
com comentários do Word já inseridos, atribuídos a João Fonseca.

## Como funciona (arquitetura)

- **`index.html`** → página que o time acessa (hospedada no GitHub Pages).
- **`netlify/functions/analyze.js`** → função que guarda sua chave da API da Claude em
  segredo e faz a análise + inserção de comentários (hospedada na Netlify, de graça).

A chave da API **nunca** fica no GitHub nem visível para o time — ela fica só nas
variáveis de ambiente da Netlify.

---

## Passo 1 — O que é uma API key e como criar a sua

Uma **API key** é como uma senha que permite que um programa (nesse caso, a função da
Netlify) converse diretamente com a Claude, sem passar pelo site claude.ai. Ela é
diferente da sua assinatura do Claude.ai — é um serviço separado, cobrado por uso
(geralmente poucos centavos por análise de contrato).

1. Acesse **https://console.anthropic.com** e crie uma conta (pode usar o e-mail da Guapeco).
2. No menu, vá em **Settings → Billing** e adicione um cartão / crédito (é pré-pago,
   você define quanto quer colocar).
3. Vá em **Settings → API Keys → Create Key**. Dê um nome, ex: `guapeco-analyzer`.
4. Copie a chave (começa com `sk-ant-...`). **Guarde em local seguro** — ela só aparece
   uma vez. Você vai colar essa chave na Netlify no Passo 3, nunca no GitHub.

## Passo 2 — Subir o código no GitHub

1. Crie um repositório novo no GitHub (pode ser privado), ex: `guapeco-analyzer`.
2. Suba todos os arquivos desta pasta para o repositório (`index.html`, `netlify.toml`,
   `package.json`, `netlify/functions/analyze.js`, este `README.md`).
3. Em **Settings → Pages** do repositório, ative o GitHub Pages apontando para a branch
   `main` e pasta raiz (`/`). Isso vai te dar uma URL tipo
   `https://seu-usuario.github.io/guapeco-analyzer/` — essa é a página que o time vai usar.

## Passo 3 — Conectar a Netlify (para rodar a função com segurança)

1. Acesse **https://app.netlify.com** e crie uma conta gratuita.
2. Clique em **Add new site → Import an existing project** e conecte o mesmo
   repositório do GitHub.
3. A Netlify vai detectar o `netlify.toml` automaticamente. Clique em **Deploy**.
4. Depois do deploy, vá em **Site configuration → Environment variables** e adicione:
   - `ANTHROPIC_API_KEY` → cole a chave que você criou no Passo 1.
   - `ACCESS_PASSPHRASE` → invente uma senha simples (ex: `guapeco2026`) — é o que
     protege sua função para que só o time de vendas (que conhece a senha) consiga usá-la.
5. Vá em **Deploys** e clique em **Trigger deploy → Deploy site** para aplicar as
   variáveis de ambiente.
6. Copie a URL do seu site Netlify (ex: `https://guapeco-analyzer.netlify.app`). O
   endereço da função será:
   `https://guapeco-analyzer.netlify.app/.netlify/functions/analyze`

## Passo 4 — Configurar a página

1. Abra a página do GitHub Pages (`https://seu-usuario.github.io/guapeco-analyzer/`).
2. Clique em **⚙️ Configurar endereço da ferramenta / senha de acesso**.
3. Cole o endereço da função (do Passo 3.6) e a senha (`ACCESS_PASSPHRASE` do Passo 3.4).
4. Clique em **Salvar**. Pronto — essa configuração fica salva no navegador de cada
   pessoa do time (elas só precisam fazer isso uma vez).

## Passo 5 — Testar

1. Selecione o tipo de minuta.
2. Suba um .docx de teste.
3. Clique em **Analisar contrato**. Em ~30-60 segundos deve aparecer o resumo e o botão
   de download do contrato com comentários.

---

## Sobre o playbook usado na análise

O prompt embutido na função (`analyze.js`, constante `PLAYBOOK`) já reflete o que
conversamos: não-negociáveis (compartilhamento de e-mail, exclusividade de 12 meses,
reajuste anual, Guapeco como Controladora de dados), pontos onde é possível ceder
(remoção de multa B2B em Via Cartão), e o precedente da Cláusula Quinta (multa mensal
recorrente, não recuperação de implantação). Se o playbook mudar, é só editar esse
texto no arquivo e fazer um novo commit — a Netlify republica sozinha.

## Limitações a ter em mente

- A inserção de comentários localiza o parágrafo certo procurando um trecho exato do
  texto sugerido pela Claude. Em contratos com formatação muito diferente do padrão
  Guapeco, algum comentário pode não ser localizado automaticamente — nesse caso a
  ferramenta avisa na tela ("⚠️ não foi possível localizar automaticamente") para revisão
  manual daquele ponto específico.
- Isso não substitui a revisão final de alguém do jurídico/comercial antes de enviar a
  devolutiva ao cliente — é uma primeira triagem para acelerar o trabalho do time
  enquanto você estiver de férias.
- Custo: cada análise consome créditos da sua API key (não do plano Claude.ai). Vale
  acompanhar o uso em console.anthropic.com → Usage.
