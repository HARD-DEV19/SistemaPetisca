# 🧠 Central de Inteligência Modular (PetiscaPeixe)

> **DIRETRIZES FUNDAMENTAIS DO CÉREBRO (LEITURA OBRIGATÓRIA ANTES DE QUALQUER AÇÃO):**
> 1. **Registro Contínuo:** Qualquer informação, compreensão ou método descoberto DEVE ser registrado aqui *durante* as interações, não apenas no final.
> 2. **Abordagem Modular:** O conhecimento é dividido em módulos isolados. Alterações no sistema devem refletir atualizações no módulo correspondente.
> 3. **Rastreabilidade Absoluta:** É obrigatório registrar elementos, relações de dependência entre arquivos, fragilidades estruturais e histórico de bugs críticos.
> 4. **Proibido Suposições:** Se a resposta não estiver neste cérebro, o agente deve investigar o código antes de agir e, em seguida, documentar o achado aqui.

---

## 📦 MÓDULO 1: Identidade e Interface (Frontend)
- **Nome Oficial:** PetiscaPeixe (Anteriormente RestoSys).
- **Logo:** `public/logo.jpg` (Original em `media/`). Usada como Favicon e em todos os cabeçalhos do sistema.
- **Tech Stack:** React 19 (Vite), Tailwind CSS v4, Lucide React, React Router.
- **Identidade Visual:** Paleta estritamente em tons de Azul e Branco (`blue-50` para fundos, `blue-600` a `blue-900` para destaques/botões).
- **Abordagem UI:** Single Page Application (SPA) com proteção de rotas baseada em `roles`.

## 🔐 MÓDULO 2: Arquitetura de Autenticação (Sessão Híbrida)
- **Problema Histórico:** Múltiplos logins no mesmo domínio derrubavam a conta principal, e regras complexas no Firestore causavam `Permission Denied` (Deadlock).
- **Solução Adotada (A Relação Híbrida):**
  - **Autoridade (Firebase Auth):** Apenas um usuário "Master" (Admin, Manager ou Cashier) faz o login real no Firebase.
  - **Operação (Waiter UI):** O Garçom entra via PIN na rota `/waiter/login`. O sistema **NÃO** troca o usuário no Firebase. Ele apenas valida o PIN consultando a coleção `users` usando a autoridade do Master, salva os dados no `localStorage` (`waiter_session`) e libera a UI restrita do garçom.
  - **Bypass de Emergência:** Em `src/contexts/AuthContext.tsx`, emails hardcoded (ex: `harddisk1911@gmail.com`) ganham cargo `admin` direto na memória do React. Isso previne perda de acesso se a rede do Firestore falhar durante o handshake.

## 🗄️ MÓDULO 3: Ontologia de Dados e Segurança (Firestore)
- **Database ID (UNIFICAÇÃO):** O projeto Firebase `gen-lang-client-0575100100` possui múltiplos bancos. Para garantir a universalidade do login (qualquer dispositivo ver os garçons criados), o sistema foi forçado a usar **EXCLUSIVAMENTE o banco `(default)`**.
  - *Causa de Falhas Anteriores:* O uso de IDs de banco específicos (ai-studio-...) causava fragmentação de dados (garçons criados em um banco não eram vistos por dispositivos conectados a outro).
- **Persistência (B815 Fix):** Em `src/lib/firebase.ts`, a inicialização usa `localCache: undefined` (memoryCache). O uso de IndexedDB estava causando crashes no React 19.
- **Regras de Segurança (`firestore.rules`):**
  - Publicadas simultaneamente em todos os bancos ativos via `firebase.json` para evitar erros de permissão em dispositivos legados.
  - Estratégia de "Ponte Aberta": Quase todas as coleções operacionais estão sob a regra `allow read, write: if isAuth();`. 


## ⚙️ MÓDULO 4: Lógica de Negócios e Fluxos de Dados
- **Gestão de Categorias e Estoque:**
  - **Insumos:** Engloba `ingredient` (matéria-prima) e `supply` (suprimentos/embalagens). Aparecem na aba "Insumos" do estoque.
  - **Cardápio:** Engloba `food` e `drink`. Itens industrializados (bebidas) podem ter estoque gerido diretamente na nova aba "Cardápio" do estoque.
  - **Ficha Técnica (Receitas):** Permite vincular `ingredient`, `drink` e `supply` a um prato para baixa automática múltipla.
- **Controle de Operação (Setores):**
  - **Bloqueio por Horário (CORRIGIDO - 14/04/2026):** O sistema agora valida em tempo real se o setor (Cozinha/Bar) está aberto antes de permitir o envio do pedido no `WaiterApp.tsx`. Displays de Cozinha e Bar exibem banner de alerta se estiverem fora do horário.
- **Mesas Especiais:**
  - Mesas nomeadas exatamente como "Balcão" ou "Bar" possuem prioridade de exibição (sempre no topo da lista).

## 🖨️ MÓDULO 5: O Motor de Impressão (GDI + 80mm)
- **A Blindagem de Quatro Lados (Safe Zones):**
  - **Direita:** Largura lógica base cravada em **32 colunas** (10pt). Este recuo é o "Safe Zone" universal para 80mm.
  - **Esquerda:** O Agente usa a margem física do driver (`MarginBounds.Left`) + 10 unidades de respiro.
  - **Topo:** Inicia a coordenada Y com um deslocamento de 10 unidades após a margem superior do driver.
  - **Baixo (Feed):** O layout Web injeta **8 linhas vazias** no final do documento para garantir que o texto passe pela guilhotina/serra.
- **A Separação de Mentes (Sistema vs Hardware):**
  - **O Cérebro (Frontend - `src/lib/escpos.ts`):** O sistema web dita o layout usando a largura dinâmica de 32. Ele formata strings exatas baseadas nas colunas estabelecidas e possui um loop que quebra linhas ativamente se o texto for maior que o espaço, empurrando a sobra para a linha de baixo sem desalinhar o preço.
  - **O Distribuidor (`src/components/PrintJobListener.tsx`):** Escuta `printJobs`. Separa o pedido em partes independentes (Cozinha/Bar) e gerencia o status `printing`. **CRÍTICO:** Possui um `AbortController` de 10s e um `recheck` (freshDoc) no Firebase antes do envio para evitar duplicidade.
  - **O Entregador (Agente Local - `scripts/print-agent.js`):** Um servidor Node.js compilado via `pkg` para `PrintAgent.exe`. Roda na porta `17321`.
    - **Tecnologia Adotada:** GDI (System.Drawing) Multi-Font.
    - **Offset Dinâmico:** Abandonada a margem fixa (15). Agora utiliza as propriedades de hardware do driver do Windows para posicionamento seguro.
    - **Tags de Escala:** O agente reconhece o prefixo `[BIG]`. Ao detectá-lo, aplica **16pt Bold** à linha e remove a tag.
    - **Método:** Recebe o texto e utiliza PowerShell para compilar dinamicamente o código C# de desenho. 
## ⚠️ MÓDULO 6: Fragilidades Conhecidas e Alertas (NÃO MEXER SEM LER)
- **[SUCESSO] Bloqueio por Horário (14/04/2026):** Sincronizado `WaiterApp`, `KitchenDisplay` e `BarDisplay` com o documento `settings/printAgent`. O bloqueio agora é impeditivo na origem (Garçom).
- **[FRAGILIDADE 1] Logout Automático:** Não insira `setTimeout` com `logout()` após o envio de pedidos do garçom.
- **[FRAGILIDADE 2] Largura de Linha [BIG]:** Linhas marcadas com `[BIG]` em `escpos.ts` suportam aprox. 17 caracteres.
- **[FRAGILIDADE 3] Múltiplos Bancos Firebase:** Sempre garanta que o deploy aponte para o banco `(default)`.
- **[DEPENDÊNCIA CRÍTICA] Executável do Agente:** Qualquer alteração em `scripts/print-agent.js` exige a compilação do `PrintAgent.exe`.
- **[ALERTA] Tags [BIG]:** Nunca remova essa tag sem atualizar a lógica correspondente no `PrintAgent.exe`.

---
*Cérebro inicializado e formatado para expansão contínua em 06 de Abril de 2026.*