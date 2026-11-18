# Dashboard de Faturamento + Manutenção de Base

Site estático (GitHub Pages) com login e banco de dados no **Supabase**.

## Arquivos do repositório

| Arquivo | Para que serve |
|---|---|
| `index.html` | O dashboard (as duas telas, login e gestão de usuários) |
| `config.js` | URL e chave pública do seu projeto Supabase — **você preenche** |
| `supabase/schema.sql` | Estrutura do banco: tabelas, regras de acesso e regras de negócio |
| `dados.csv` | Base de exemplo da tela Faturamento. **Não suba para o GitHub**: arquivos do repositório ficam públicos, sem login |
| `manutencao.csv` | Base de exemplo para testar a Manutenção de Base. **Também não suba para o GitHub** |

## Passo a passo (uma vez só)

### 1. Criar o projeto no Supabase
1. Entre em supabase.com → **New project**.
2. Região: **South America (São Paulo)** — os dados têm CPF (LGPD).
3. Guarde a senha do banco num lugar seguro (não vai no site).

### 2. Criar a estrutura do banco
1. No projeto: **SQL Editor → New query**.
2. Cole todo o conteúdo de `supabase/schema.sql` e clique **Run**.
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

### 5. Ligar o site ao banco
Em **Settings → API**, copie:
- **Project URL**
- **anon public** (chave pública)

Cole no `config.js`:
```js
window.DASH_CONFIG = {
  supabaseUrl: 'https://xxxxxxxx.supabase.co',
  supabaseAnonKey: 'eyJhbGciOi...'
};
```
⚠️ **Nunca** use a chave `service_role` no site.

### 6. Subir no GitHub
Suba para o repositório, na raiz (mesma pasta), **apenas** `index.html` e `config.js` (e, se quiser, este `LEIA-ME.md`). Planilhas (`.csv`/`.xlsx`) nunca vão para o GitHub: tudo que está no repositório pode ser baixado por qualquer pessoa, sem login. Os dados entram pelo botão **Carregar planilha**, já logado. O `schema.sql` não precisa ir. Depois acesse o site e entre com o administrador.

## Papéis de usuário

| Papel | Pode |
|---|---|
| **Administrador** | Tudo + criar/liberar usuários, mudar papéis, remover cargas |
| **Editor** | Carregar planilhas e alterar status |
| **Leitor** | Consultar, abrir detalhes e exportar |

Usuários: menu com seu nome (topo) → **Usuários e acessos**.

### Telas liberadas por usuário
Na coluna **Telas** de “Usuários e acessos”, marque o que cada pessoa pode ver: **Faturamento** e/ou **Manutenção de base**.
- A aba de uma tela não liberada some, e o banco também recusa os dados dessa tela para essa pessoa.
- O papel continua valendo dentro da tela (ex.: Editor só no Faturamento carrega planilhas de Faturamento, mas não vê a Manutenção de Base).
- Administrador vê todas as telas, sempre.
- Usuário sem nenhuma tela marcada vê só um aviso para pedir liberação.
- Mudanças valem no próximo acesso da pessoa (ou ao recarregar a página).

## Regras da Manutenção de Base (ficam no banco)
- Cada planilha carregada **soma** à base e recebe a data da carga. O mesmo arquivo não entra duas vezes.
- Se o envio falhar no meio, a carga é desfeita inteira (nada fica pela metade).
- **INCOBRÁVEL NA BASE** marca o CPF em todas as linhas, de todas as cargas, e também nas cargas futuras.
- Tirar uma linha de INCOBRÁVEL NA BASE libera o CPF e devolve as outras linhas ao status anterior.
- DISPONIBILIZAR, INCOBRÁVEL NO CLIENTE e FALECIDO valem só para a linha alterada.
- Toda alteração fica no histórico (quem, quando, de → para) e pode ser desfeita, desde que ninguém tenha mexido nas mesmas linhas depois.
- Telas abertas por outras pessoas atualizam sozinhas quando alguém altera algo.

## Observações
- Tela **Faturamento**: cada planilha carregada por Editor ou Administrador fica salva no **Histórico de cargas** (botão no rodapé). Ao entrar, o painel abre a carga mais recente. Pelo histórico dá para reabrir uma carga antiga ou, se for Administrador, removê-la. Planilhas carregadas por Leitor aparecem só na tela de quem carregou.
- Ao carregar uma planilha de Faturamento, mantenha a página aberta até a barra "Salvando no histórico" terminar.
- **Performance de Vencimentos**: clique numa barra para abrir a tabela dia a dia do mês (previsto × pago × falta receber, em R$, quantidade e %). O botão "Exportar Excel" baixa o resumo por dia e todas as cobranças do mês. O botão "Filtrar o painel por este mês" dentro da janela aplica o filtro como antes.
- Plano gratuito do Supabase: o projeto pausa após ~7 dias sem uso; reative no painel.
- Backup: **Exportar base completa** gera um Excel com status original, atual, data da carga e quem alterou.
