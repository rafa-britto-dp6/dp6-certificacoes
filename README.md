# dp6-certificacoes
📊 DP6 — Plataforma de Certificações
Portal interno para acompanhar o cronograma anual de certificações da DP6, organizado por trimestre. Desenvolvido como single-page application hospedada no GitHub Pages, com banco de dados em tempo real via Supabase.
🔗 Acesse: rafa-britto-dp6.github.io/dp6-certificacoes

✨ Funcionalidades

Cronograma por trimestre — acordeões Q1 a Q4 com animações suaves
Dois tipos de certificação — Obrigatórias (destaque em cyan) e Opcionais
7 áreas de atuação — tags coloridas por time: Consulting/BA, DE, DS, Infra, Lideranças, Todos, Outros
Modal de acesso — clique em qualquer card para ver detalhes e acessar o link de inscrição
Painel Admin protegido por senha com 4 abas:

📅 Trimestres — visualize e realoque certificações entre quarters
📦 Gerenciar Certificações — CRUD completo (adicionar, editar, alocar, excluir)
⚙️ Configurações — troque o ano ativo (arquiva o atual) e altere a senha
🗂 Arquivo — histórico read-only de anos anteriores com timestamp


Indicador de servidor — badge no header mostra "Servidor ativo" ou "Servidor inativo" em tempo real
100% online — sem cache local, todos os dados vêm do banco a cada acesso


🏗️ Arquitetura
GitHub Pages (frontend)
       │
       │  GET / PATCH
       ▼
Supabase Edge Function (dp6-store)
       │
       │  REST API (service role key — sem CORS)
       ▼
Supabase PostgreSQL (tabela dp6_store)
Por que Edge Function?
O Supabase bloqueia requisições CORS de domínios externos quando o projeto é de organização. A Edge Function atua como proxy interno: o browser chama a função, e ela faz a requisição ao banco de dentro do próprio servidor Supabase — sem nenhum problema de CORS.

🗄️ Banco de Dados
Projeto Supabase: dp6-certs-2
Tabela: dp6_store
sqlcreate table dp6_store (
  id   int primary key default 1,
  data jsonb not null default '{}'::jsonb
);
A tabela armazena um único registro JSON com toda a estrutura do app:
json{
  "activeYear": "2026",
  "adminPass": "••••••••••••••••••",
  "years": {
    "2026": {
      "q1": [ { "id": "abc123", "name": "Google Analytics 4", "url": "https://...", "tags": [{"id": "de"}], "tipo": "obrigatorias" } ],
      "q2": [],
      "q3": [],
      "q4": []
    }
  },
  "bank": [
    { "id": "abc123", "name": "Google Analytics 4", "url": "https://...", "tags": [{"id": "de"}], "tipo": "obrigatorias" }
  ],
  "archive": {
    "2025": {
      "q1": [...],
      "q2": [...],
      "q3": [...],
      "q4": [...],
      "_archivedAt": "07/03/2026 14:35"
    }
  }
}

⚙️ Edge Function
Localização: supabase/functions/dp6-store/index.ts
MétodoAçãoGETCarrega o store completo do bancoPATCHSalva o store atualizado no bancoOPTIONSResponde ao preflight CORS
A função usa a SUPABASE_SERVICE_ROLE_KEY (variável de ambiente automática no Supabase) para acessar o banco com permissão total, independente do RLS.

🚀 Setup & Deploy
Pré-requisitos

Conta no Supabase
Conta no GitHub
Node.js instalado
Supabase CLI instalado via Scoop (Windows):

powershellSet-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex
scoop bucket add supabase https://github.com/supabase/scoop-bucket.git
scoop install supabase
1. Criar projeto no Supabase

Acesse supabase.com → New project
Escolha a região South America (São Paulo)
Aguarde ~2 minutos o projeto subir

2. Criar a tabela
No SQL Editor do projeto, execute:
sqlcreate table dp6_store (
  id   int primary key default 1,
  data jsonb not null default '{}'::jsonb
);

insert into dp6_store (id, data) values (1, '{
  "activeYear": "2026",
  "adminPass": "SUA_SENHA_AQUI",
  "years": {"2026": {"q1": [], "q2": [], "q3": [], "q4": []}},
  "bank": [],
  "archive": {}
}'::jsonb);

alter table dp6_store enable row level security;
create policy "service role full access" on dp6_store using (true) with check (true);
3. Fazer deploy da Edge Function
powershell# Na pasta do projeto
cd dp6-supabase

# Login no Supabase CLI
supabase login

# Deploy
supabase functions deploy dp6-store --project-ref SEU_PROJECT_ID
4. Configurar o index.html
No arquivo index.html, substitua as constantes no topo do <script>:
javascriptconst EDGE_URL        = 'https://SEU_PROJECT_ID.supabase.co/functions/v1/dp6-store';
const SUPABASE_ANON_KEY = 'eyJ...'; // anon key do projeto
5. Publicar no GitHub Pages

Suba o index.html para o repositório
Vá em Settings → Pages
Em Source, selecione a branch main e a pasta / (root)
Aguarde ~1 minuto e acesse a URL gerada


🔐 Acesso Admin
O painel admin é acessado pelo botão Admin no header do site.

A senha é armazenada no campo adminPass dentro do JSON no banco
Para alterar a senha via SQL:

sqlupdate dp6_store
set data = jsonb_set(data, '{adminPass}', '"NOVA_SENHA"')
where id = 1;

Também é possível alterar pela aba ⚙️ Configurações dentro do próprio painel admin


🏷️ Tags disponíveis
TagCorTimeconsultingAzul claroConsulting / BAdeVerdeData EngineeringdsRoxoData ScienceinfraLaranjaInfra / Help DeskliderancasSalmãoLideranças / ManagerstodosCyanTodos os timesoutrosCinzaPersonalizado

📁 Estrutura do Repositório
dp6-certificacoes/
│
├── index.html                          # App completo (HTML + CSS + JS)
│
└── README.md                           # Este arquivo

dp6-supabase/                           # Projeto local do Supabase CLI
└── supabase/
    └── functions/
        └── dp6-store/
            ├── index.ts                # Edge Function (proxy anti-CORS)
            └── deno.json

🎨 Design System
TokenValorUso--bg-deep#0a1520Background principal--bg-card#112236Cards e painéis--cyan#00c2d4Destaque, CTAs, elementos primários--muted#6a8a9fTextos secundáriosFontInterToda a tipografia

🛠️ Tecnologias
TecnologiaUsoHTML/CSS/JS vanillaFrontend (zero dependências)GitHub PagesHospedagem gratuitaSupabase PostgreSQLBanco de dadosSupabase Edge FunctionsProxy API (resolve CORS)Deno (TypeScript)Runtime da Edge Function

📄 Licença
Uso interno DP6. Todos os direitos reservados.
