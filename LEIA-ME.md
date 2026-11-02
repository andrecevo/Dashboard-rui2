# Dashboard de Faturamento + Manutenção de Base

Site estático (GitHub Pages) com login e banco de dados no **Supabase**.

## Arquivos do repositório

| Arquivo | Para que serve |
|---|---|
| `index.html` | O dashboard (as duas telas, login e gestão de usuários) |
| `config.js` | URL e chave pública do seu projeto Supabase — **você preenche** |
| `schema.sql` | Estrutura do banco: tabelas, regras de acesso e regras de negócio. **Não precisa ir para o GitHub** — é só colado no Supabase (passo 2) |
| `dados.csv` | Base de exemplo da tela Faturamento (abre automaticamente) |
| `manutencao.csv` | Base de exemplo para a primeira carga da Manutenção de Base |

## Passo a passo (uma vez só)

### 1. Criar o projeto no Supabase
1. Entre em supabase.com → **New project**.
2. Região: **South America (São Paulo)** — os dados têm CPF (LGPD).
3. Guarde a senha do banco num lugar seguro (não vai no site).

### 2. Criar a estrutura do banco
1. No projeto: **SQL Editor → New query**.
2. Abra o `schema.sql` no Bloco de Notas, copie tudo (Ctrl+A, Ctrl+C), cole no editor do Supabase e clique **Run**.
3. Deve aparecer "Success". Pode rodar de novo no futuro sem perder dados.

### 3. Ajustar o login
Em **Authentication → Sign In / Providers → Email**:
- **Confirm email**: desligue se quiser que usuários criados pelo painel entrem na hora. Se deixar ligado, cada pessoa precisa clicar no link do e-mail antes do primeiro acesso.
- **Allow new users to sign up**: deixe **ligado** (o painel do administrador cria usuários por aqui). Isso é seguro porque todo usuário novo entra **sem acesso** até um administrador liberar.

Em **Authentication → URL Configuration**:
- **Site URL**: o endereço do site no GitHub Pages (ex.: `https://maxximarj-blip.github.io/NOME-DO-REPOSITORIO/`).
- **Redirect URLs**: adicione o mesmo endereço. É para onde vai o link de "Esqueci minha senha".

### 4. Criar o primeiro administrador
Em **Authentication → Users → Add user → Create new user**: e-mail e senha, marque **Auto Confirm User**.
O **primeiro usuário criado vira Administrador automaticamente**. Os próximos você cria pelo próprio dashboard.

### 5. Ligar o site ao banco (arquivo `config.js`)

**5.1 Pegar a URL do projeto**
1. No Supabase, abra o seu projeto.
2. Menu da esquerda, lá embaixo: ícone de engrenagem **Project Settings**.
3. Clique em **Data API** (em versões antigas do painel o nome é **API**).
4. Copie o campo **Project URL** (algo como `https://abcdefghijklmno.supabase.co`).

**5.2 Pegar a chave pública**
1. Ainda em **Project Settings**, clique em **API Keys**.
2. Se aparecerem as abas **"API Keys"** e **"Legacy API Keys"**, abra **Legacy API Keys** e copie a chave **anon / public** (um texto longo que começa com `eyJ`).
3. Se o seu painel só mostrar a chave **Publishable** (`sb_publishable_...`), use essa.
4. **Não copie** a chave **service_role** nem a **Secret** (`sb_secret_...`): elas dão acesso total ao banco e não podem ir para o site.

**5.3 Colar no `config.js`**
Abra o `config.js` (no Bloco de Notas, ou no GitHub: arquivo → ícone de lápis ✏️ → editar → **Commit changes**) e deixe assim, trocando pelos seus valores:
```js
window.DASH_CONFIG = {
  supabaseUrl: 'https://abcdefghijklmno.supabase.co',
  supabaseAnonKey: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.....'
};
```
Cuidados:
- Os valores ficam **entre aspas simples** `'...'`.
- Mantenha a **vírgula** depois da URL.
- A URL vai **sem barra no final** e sem `/rest/v1`.
- A chave é longa: copie **inteira**, sem espaços nem quebras de linha.

**5.4 Conferir**
Abra o site (no GitHub Pages pode levar 1–2 minutos para atualizar; use **Ctrl+F5**):
- Apareceu a tela **ENTRAR** → está ligado.
- Apareceu **CONFIGURAÇÃO PENDENTE** → o `config.js` não foi lido ou está em branco (confira o nome do arquivo e se ele está na mesma pasta do `index.html`).
- Entrou mas aparece erro de permissão → o `schema.sql` (passo 2) não foi executado.

### 6. Subir no GitHub
Suba para o repositório, na raiz (mesma pasta): `index.html`, `config.js`, `dados.csv` e, se quiser, o `manutencao.csv` e este `LEIA-ME.md`. O `schema.sql` não precisa ir. Depois acesse o site e entre com o administrador.

## Papéis de usuário

| Papel | Pode |
|---|---|
| **Administrador** | Tudo + criar/liberar usuários, mudar papéis, remover cargas |
| **Editor** | Carregar planilhas e alterar status |
| **Leitor** | Consultar, abrir detalhes e exportar |

Usuários: menu com seu nome (topo) → **Usuários e acessos**.

## Regras da Manutenção de Base (ficam no banco)
- Cada planilha carregada **soma** à base e recebe a data da carga. O mesmo arquivo não entra duas vezes.
- Se o envio falhar no meio, a carga é desfeita inteira (nada fica pela metade).
- **INCOBRÁVEL NA BASE** marca o CPF em todas as linhas, de todas as cargas, e também nas cargas futuras.
- Tirar uma linha de INCOBRÁVEL NA BASE libera o CPF e devolve as outras linhas ao status anterior.
- DISPONIBILIZAR, INCOBRÁVEL NO CLIENTE e FALECIDO valem só para a linha alterada.
- Toda alteração fica no histórico (quem, quando, de → para) e pode ser desfeita, desde que ninguém tenha mexido nas mesmas linhas depois.
- Telas abertas por outras pessoas atualizam sozinhas quando alguém altera algo.

## Observações
- A tela **Faturamento** continua lendo o CSV/XLSX no navegador (não grava no banco).
- Plano gratuito do Supabase: o projeto pausa após ~7 dias sem uso; reative no painel.
- Backup: **Exportar base completa** gera um Excel com status original, atual, data da carga e quem alterou.
