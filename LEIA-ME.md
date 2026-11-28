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

## Aba Incobráveis (restrições fixas)
- Guarda, por CPF, os **motivos de restrição**. Padrão: **INCOBRÁVEL NA BASE**, **FALECIDO** (valem para o CPF em tudo) e **INCOBRÁVEL NO CLIENTE** (só no cliente indicado).
- Os motivos são cadastrados na tela **Premissas** (abaixo). Eles aparecem na Manutenção de Base (status, alteração em lote, quadros), na importação e nos filtros.
- Prioridade: quando o mesmo CPF tem mais de um motivo, vale o de menor número (Na base = 1, Falecido = 2, No cliente = 3, novos depois).
- Entram sozinhas quando alguém marca uma linha com um motivo na Manutenção de Base, ou pelos botões **Adicionar** e **Importar planilha**.
- **Importar planilha** lê **todas as abas** que têm coluna CPF. O motivo de cada linha vem, nesta ordem: coluna TIPO → nome da aba (ex.: aba "CLIENTE" = Incobrável no cliente, "FALECIDOS", "NA BASE") → CLIENTE preenchido = Incobrável no cliente → motivo padrão escolhido na tela. Antes de importar aparece a prévia de quantas linhas vão para cada motivo.
- Marque **substituir** na importação para corrigir uma importação errada: o CPF perde os motivos antigos que não estão na planilha.
- **Alterar** troca o motivo/cliente de uma restrição; **Liberar** tira a restrição (as linhas do CPF com aquele status voltam ao anterior). Tudo vai para o Histórico de alterações e pode ser desfeito.
- A coluna **Na base hoje** mostra em quais clientes e com qual status o CPF está na base. "Linhas" em laranja = 0: a restrição está gravada, mas o CPF não aparece na base (ou não naquele cliente).
- Os quadros contam **CPFs**; a Manutenção de Base conta **linhas** (um CPF pode ter várias linhas). O subtítulo de cada quadro mostra as duas coisas.
- **Toda carga nova** da Manutenção de Base é carimbada automaticamente; o status original da planilha fica guardado.
- **Apagar restrições** (só Administrador): apaga de uma vez as restrições do filtro da tela (todas, ou só um motivo, ou só o resultado de uma busca). Pede para digitar APAGAR. As linhas marcadas voltam ao status anterior, e dá para desfazer pelo Histórico de alterações.
- Editor e Administrador incluem, importam, alteram e liberam; Leitor consulta e exporta.

## Tela Premissas (só Administrador)
- Aparece apenas para administradores (não entra nas caixinhas de "Telas" dos usuários).
- Lista todos os **motivos de incobráveis**, com quantos CPFs e quantas linhas da base usam cada um.
- **+ Novo motivo**: nome e se vale para o CPF em todas as cargas e clientes ou só no cliente indicado. O motivo novo entra com a próxima prioridade.
- **Apagar**: só para motivos sem uso. Se estiver "em uso", libere os CPFs na aba Incobráveis (ou use Apagar restrições filtrando o motivo) e altere as linhas da base antes.
- **INCOBRÁVEL NA BASE** é fixo: é ele que bloqueia o CPF em todas as cargas. FALECIDO e INCOBRÁVEL NO CLIENTE podem ser apagados e não voltam ao rodar o `schema.sql` de novo.

### Premissas › Comissão sobre recebimento
- **Empresas do split**: cadastre as empresas que recebem parte da comissão (só sai do cadastro se nenhuma regra usar).
- **Regra**: tipo de cobrança (ex.: Cobrança Débito, Cobrança Ano Corrente) + % de comissão sobre o valor pago + split entre as empresas (soma 100%) + **clientes marcados** + **vigência**.
- A regra vale para cobranças **pagas**, escolhida pelo tipo, pelo cliente e pela **data de pagamento** dentro da vigência.
- **Nova vigência**: para mudar % ou split a partir de uma data. A regra atual termina no dia anterior e a nova começa na data escolhida; o histórico fica preservado.
- O sistema não deixa duas regras do mesmo tipo e cliente com vigências sobrepostas.
- Na tela **Faturamento**: quadro **Comissão gerada**, linha laranja de comissão no **Fluxo de Pagamentos** (eixo da direita, com o detalhe por empresa ao passar o mouse) e filtro **Empresa (split)**. Ao escolher uma empresa, todos os gráficos passam a mostrar só os recebimentos que geraram comissão para ela, e a comissão mostra só a parte dela.

## Observações
- Tela **Faturamento**: cada planilha carregada por Editor ou Administrador fica salva no **Histórico de cargas** (botão no rodapé). Ao entrar, o painel abre a carga mais recente. Pelo histórico dá para reabrir uma carga antiga ou, se for Administrador, removê-la. Planilhas carregadas por Leitor aparecem só na tela de quem carregou.
- Ao carregar uma planilha de Faturamento, mantenha a página aberta até a barra "Salvando no histórico" terminar.
- **Performance de Vencimentos**: clique numa barra para abrir a tabela dia a dia do mês (previsto × pago × falta receber, em R$, quantidade e %). O botão "Exportar Excel" baixa o resumo por dia e todas as cobranças do mês. O botão "Filtrar o painel por este mês" dentro da janela aplica o filtro como antes.
- Plano gratuito do Supabase: o projeto pausa após ~7 dias sem uso; reative no painel.
- Backup: **Exportar base completa** gera um Excel com status original, atual, data da carga e quem alterou.
