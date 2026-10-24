# Dexter Orçamentos — Plano

> Versão 0.5 · 2026-10-16 · status: planejamento + protótipo navegável
> Origem da ideia: `ideia-original-chatgpt.md` (texto bruto da conversa no ChatGPT)
> Protótipo: `../prototipo/index.html` (abre no navegador; dados fictícios e valores provisórios)
> Perguntas para o dono: `perguntas-para-o-dono.md`

## 1. Objetivo

Padronizar os preços da Dexter Soluções em Refrigeração. O técnico informa o que o serviço precisa e o app aplica as regras de preço do dono (mão de obra, deslocamento, margens, acréscimo de segurança e impostos). O técnico não calcula margem.

**Visão de longo prazo:** o app nasce para a Dexter, mas a intenção é virar um **SaaS totalmente configurável e modular**, que cresça para um sistema grande. Isso orienta a arquitetura desde o MVP (ver seção 12).

## 2. Restrição principal

**O preço de peça não pode ficar fixo no banco de dados.** Ele varia demais, e todo orçamento exige consultar fornecedores antes. Hoje, esse é o gargalo do processo.

Consequências:

- Não existe catálogo com preço nem estoque. O custo da peça entra **em cada orçamento**, a partir de uma cotação recente.
- O app não elimina a consulta ao fornecedor. Ele **organiza e acelera** essa etapa.
- O orçamento enviado ao cliente precisa de **validade curta e explícita**. O CDC permite estipular a validade; quando ela não é estipulada, o orçamento vale 10 dias (art. 40, §1º).
- **O cliente deve receber o preço o quanto antes.** A velocidade entre o atendimento e o envio do orçamento é a principal métrica do app.

## 3. Fluxo proposto

```
Atendimento ──► Cotação ──► Preço ──► Cliente ──► Execução
(diagnóstico)  (fornecedores) (motor)  (envio/aceite)
```

1. **Atendimento:** o técnico registra cliente, equipamento, defeito, peças necessárias (sem preço), horas estimadas e km. Status: `aguardando cotação`.
2. **Cotação:** **o técnico e o escritório/dono cotam**. O orçamento entra numa fila de cotação que os dois veem. O app gera a mensagem de pedido de cotação para o WhatsApp dos fornecedores. Quem receber a resposta lança o fornecedor, o custo e o prazo, e o app registra quem lançou. Status: `cotado`.
3. **Preço:** o motor aplica os parâmetros vigentes do dono e calcula o preço de venda de cada linha e o total.
4. **Cliente:** o orçamento discriminado (mão de obra e materiais separados, forma de pagamento, prazo e validade) vai por WhatsApp, como texto ou link (PDF na fase 2). Status: `enviado`, depois `aprovado`, `recusado` ou `expirado`.
5. **Execução:** se o cliente aprovar depois da validade, ou se o fornecedor tiver mudado o preço, é preciso cotar de novo.

## 4. Como o app ataca o gargalo da cotação

- **Fila de cotação compartilhada:** técnico e escritório veem os orçamentos `aguardando cotação`, o que evita cotar a mesma peça duas vezes. Cada cotação registra quem a lançou.
- **Mensagem de cotação pronta:** você seleciona as peças, o app gera um texto padronizado, e um botão abre o WhatsApp de cada fornecedor (links `wa.me`).
- **Cadastro de fornecedores:** nome, WhatsApp e o que cada um costuma ter.
- **Histórico de cotações como referência, não como preço:** por exemplo, "último preço cotado: R$ X com o fornecedor Y há 2 dias".
- **Reaproveitar uma cotação recente para enviar mais rápido (a decidir com o dono):** o acréscimo de segurança cresce com a idade da cotação. Por exemplo: cotação de hoje 0%, até 3 dias 5%, até 7 dias 10%, mais velha exige nova cotação. Assim o técnico manda o preço na hora quando há uma cotação recente.
- **Comparação de fornecedores** para a mesma peça (preço e prazo).
- **Lista de peças sem preço**, com nomes padronizados, só para autocompletar e agrupar o histórico.

## 5. Motor de precificação

- Fica num pacote isolado (`packages/pricing`), com funções puras e testes no Vitest. A API é a fonte da verdade; o front só mostra uma prévia.
- Valores em **centavos inteiros**.
- Calcula o **preço de venda de cada linha**, porque o CDC (art. 40) exige separar mão de obra e materiais.
- Cada orçamento guarda uma **cópia dos parâmetros** usados (a versão). Assim, mudar a margem não altera orçamentos antigos.
- **Margem e acréscimo de segurança são coisas diferentes:**
  - A **margem** é o lucro que o dono quer ter.
  - O **acréscimo de segurança** cobre o risco do preço da peça subir entre a cotação e a compra. Só se aplica a peças.
  - Na conta, os dois viram um multiplicador só. Separar serve para o dono enxergar quanto é lucro e quanto é proteção, e permite o acréscimo variar com a idade da cotação.
- A **margem pode ser diferente para peças e mão de obra** (a decidir). O cliente vê o preço das peças discriminado e pode pesquisar na internet.
- A fórmula abaixo é provisória. **A fórmula real vem do dono** (ver perguntas).

Fórmula provisória (por linha):

- Peça: `custo = custo_cotado × qtd × (1 + seguranca)` e `preco = custo × (1 + margem_pecas) / (1 − imposto)`
- Serviço: `custo = horas × valor_hora + km × valor_km + custo_fixo` e `preco = custo × (1 + margem_servico) / (1 − imposto)`

Exemplo: peça cotada a R$ 100, margem de 30%, imposto de 6%.

| | Preço ao cliente | Lucro se a peça custar R$ 100 | Lucro se subir para R$ 110 |
|---|---|---|---|
| Só margem | R$ 138,30 | R$ 30,00 | R$ 20,00 |
| Margem + segurança de 5% | R$ 145,21 | R$ 36,50 | R$ 26,50 |

## 6. Stack (proposta, a confirmar)

- **PWA:** o técnico usa no celular e o dono no computador, sem passar pela loja de apps.
- **Mobile-first:** as telas são desenhadas primeiro para o celular do técnico. No computador, a navegação vira uma barra lateral. Em telas largas (a partir de 1200px), o layout tem três colunas: menu, lista e detalhe, com navegação por teclado (`j`/`k` e `n`).
- **Visual:** estilo terminal (TUI), com fonte JetBrains Mono e paleta Nord, nos temas escuro e claro. Esse é o **tema padrão**: paleta, fonte e marca serão configuráveis (seção 12).
- Front: React, Vite, TypeScript e Tailwind.
- API: Node com Fastify, Drizzle, Postgres, Better Auth e Zod.
- Monorepo pnpm: `apps/web`, `apps/api` e `packages/pricing`.
- Hospedagem: a definir, **fora** da infra da ICL.

## 7. Acesso e segurança

- **Login obrigatório para todos**, administrador e técnico. Nenhuma tela funciona sem sessão.
- **Sem cadastro aberto.** Só o administrador cria acessos: a pessoa recebe um convite por e-mail e cria a própria senha. Quem descobrir o endereço do app não consegue criar orçamentos.
- **Papéis no MVP:**
  - o técnico cria e cota orçamentos;
  - o administrador também muda preços, equipe e fornecedores.
  - Depois, os papéis viram **cargos com permissões configuráveis** (seção 12). O MVP já é construído sobre permissões, mesmo saindo só com esses dois cargos.
- **A API valida a sessão e a permissão em toda rota.** Esconder o botão na tela não basta.
- **Proteção do login:** limite de tentativas e sessão que expira.
- **Desativar o acesso** de quem sai da equipe.
- **Link de orçamento para o cliente (se existir, fase 2):** só leitura, com um token impossível de adivinhar, que expira junto com a validade do orçamento.

## 8. Modelo de dados (rascunho)

| Tabela | Campos principais |
|---|---|
| empresas | nome, ramo, logo (só se for multiempresa; ver seção 12) |
| configuracoes | tema (paleta, fonte), módulos ligados, textos padrão |
| cargos | nome, fixo (o Administrador não pode ser apagado nem perder permissões) |
| cargo_permissoes | cargo, permissão (ex.: `orcamento.enviar`) |
| usuarios | nome, e-mail, cargo, status (convidado / ativo / desativado) |
| parametros | versão, valor_hora, valor_km, custo_fixo, margem_pecas, margem_servico, imposto, regra de acréscimo de segurança, validade_orcamento_dias, vigente_desde |
| fornecedores | nome, whatsapp, observações |
| pecas | nome padronizado (sem preço) |
| cotacoes | peça, fornecedor, custo_centavos, prazo_entrega, data, quem lançou |
| orcamentos | cliente, equipamento, defeito, status, versão dos parâmetros, validade, total |
| itens_orcamento | tipo (peça / mão de obra / deslocamento), descrição, qtd, custo, acréscimo de segurança aplicado, preço de venda, cotação |

## 9. Fases

| Fase | Entregas |
|---|---|
| 0. Conversa com o dono | Mostrar o protótipo, pegar a planilha ou a fórmula atual de preço e validar o fluxo acima |
| 1. MVP | Login com permissões por função (cargos Administrador e Técnico), parâmetros com versão, fornecedores, orçamento com status, fila de cotação compartilhada, lançamento de cotação, mensagem de cotação para o WhatsApp, cálculo com margens e acréscimo de segurança, envio ao cliente por WhatsApp, histórico |
| 2. Documento e referência | PDF com a marca, lista de peças para autocompletar, histórico de cotações como referência, reaproveitar cotação com acréscimo por idade, comparação de fornecedores, aviso (push) quando a cotação ficar pronta, desconto com aprovação, relatórios |
| 3. Extras | Funcionar offline, integração com o GestãoClick, foto ou print da cotação como comprovante |
| 4. Plataforma (SaaS) | Tela de cargos e permissões, ligar e desligar módulos, editor de aparência (paleta, fonte, marca), novos módulos (clientes, ordens de serviço, agenda, financeiro, relatórios, marketing), multiempresa se for vender para outras empresas |

## 10. Perguntas em aberto

**Para o dono:**

- A margem é igual para peça e mão de obra, ou diferente? É margem sobre o custo ou sobre o preço de venda?
- Qual o tamanho do acréscimo de segurança? Fixo ou crescendo com a idade da cotação?
- Por quantos dias uma cotação pode ser reaproveitada?
- Qual a validade do orçamento hoje?
- No orçamento, o cliente vê o preço de cada peça ou só o total das peças?
- Deslocamento: cobra por km? Quanto?
- Quantos fornecedores vocês costumam consultar por peça? Eles respondem por WhatsApp, telefone ou site?
- Como é a planilha ou a fórmula atual de preço?
- Qual o regime tributário e a alíquota?
- Cobra por hora, por tipo de serviço ou com taxa de visita/diagnóstico?
- O técnico pode dar desconto? Até quanto?
- Quantos técnicos são? Usam Android? Costuma ter sinal no local do atendimento?
- Para que vocês usam o GestãoClick?

**Para nós:**

- Modo ensino ou modo entrega?
- Onde hospedar e quem paga?
- Qual o prazo e qual o acordo com o amigo?
- **Multiempresa:** o sistema vai ser vendido para outras empresas? Se sim, isso precisa ser decidido antes de criar o banco (seção 12).
- Se virar SaaS, o código fica com a gente e a Dexter paga licença ou mensalidade? Isso muda a proposta comercial.
- O produto terá nome próprio, separado da marca Dexter?

## 11. Decisões tomadas

- **2026-10-15:** sem catálogo com preço fixo nem estoque. O custo de cada peça vem de uma cotação feita para aquele orçamento.
- **2026-10-15:** o técnico e o escritório/dono cotam. A fila de cotação é compartilhada e cada cotação registra quem a lançou.
- **2026-10-15:** o cliente deve receber o orçamento o quanto antes. A velocidade é a prioridade do fluxo.
- **2026-10-15:** haverá margem de lucro nas peças **e** um acréscimo de segurança separado, aplicado só às peças.
- **2026-10-15:** login obrigatório para administrador e técnico, sem cadastro aberto. Só o administrador cria acessos.
- **2026-10-15:** desenvolvimento mobile-first.
- **2026-10-15:** visual TUI com tema Nord. No computador largo, três colunas (menu, lista, detalhe).
- **2026-10-16:** o produto é um SaaS totalmente configurável: variáveis de preço, percentuais de lucro e até o tema da interface.
- **2026-10-16:** o sistema será modular. O administrador escolhe quais funções cada tipo de usuário pode usar, e os tipos vão além de admin e técnico (financeiro, marketing, atendimento, gerente etc.).

## 12. Configuração e módulos (visão de produto)

A ideia é crescer para um sistema grande. As telas virão aos poucos, mas a base precisa nascer preparada para isso.

**Tudo configurável pelo administrador, sem mexer no código:**

- **Variáveis de preço:** valor da hora, custo por km, taxa fixa, imposto, validade e formas de pagamento.
- **Percentuais de lucro:** as margens, e também as faixas do acréscimo de segurança (dias e %).
- **Aparência:** paleta de cores, fonte, nome e logo da empresa. O visual TUI/Nord é o tema padrão. A escolha entre claro e escuro continua sendo de cada pessoa.
- **Com o tempo:** listas (peças, equipamentos) e textos das mensagens.

**Módulos:**

- O administrador liga e desliga módulos.
- Hoje existem orçamentos e cotações.
- Candidatos para depois: clientes, ordens de serviço, agenda, financeiro, relatórios e marketing.

**Cargos e permissões:**

- Os dois papéis fixos (administrador e técnico) viram cargos criados pelo administrador: técnico, atendimento, financeiro, marketing, gerente etc.
- Cada cargo recebe uma lista de permissões, uma por função. Exemplos: criar orçamento, ver os orçamentos de toda a equipe, lançar custo, enviar ao cliente, ver margens, alterar preços e gerenciar a equipe.
- O cargo Administrador é fixo e tem todas as permissões. Assim, ninguém perde o acesso por engano.
- Uma permissão só vale se o módulo dela estiver ligado.

**O que isso muda na construção, já no MVP:**

- **A API checa permissões, não papéis.** Por exemplo, `pode(usuario, 'orcamento.enviar')` em vez de `papel === 'admin'`. Começar assim custa pouco agora e evita reescrever todas as rotas depois. O MVP pode sair só com Administrador e Técnico, montados sobre esse mecanismo.
- **As configurações ficam no banco.** As que afetam preço têm versão, como já acontece com os parâmetros.
- **O tema usa variáveis CSS** carregadas da configuração da empresa. O protótipo já funciona assim.
- **Cada módulo fica numa pasta própria**, com rotas, telas e tabelas. Um registro central lista os módulos e as permissões de cada um.
- **Multiempresa:** se o sistema for vendido para outras empresas, toda tabela leva `empresa_id` desde a primeira migração. Acrescentar isso depois é caro.
- **Cuidado com o excesso:** configuração demais vira complexidade para o dono. Cada opção nasce com um padrão bom, e só vira configurável quando alguém precisar.
