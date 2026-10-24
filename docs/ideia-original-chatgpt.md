# Projeto de Aplicativo — Dexter Orçamentos

A proposta é desenvolver um aplicativo de orçamento técnico para a **Dexter Soluções em Refrigeração**, com duas áreas separadas: uma para você, como administrador, e outra para os funcionários que realizam os atendimentos.

O principal objetivo é **padronizar os preços da Dexter**, para que o funcionário não precise calcular margem de lucro, impostos, custo operacional ou valor da mão de obra. Ele simplesmente informa o que será utilizado no serviço, e o aplicativo calcula o preço final automaticamente.

## 1. Como o aplicativo vai funcionar

<box border radius="xl" padding={3} gap={2}>
  <box background="surface-secondary" radius="lg" padding={3} align="center" gap={1}>
    <icon name="settings" color="secondary" size="lg"/>
    **Painel do proprietário**
    <text size="xs" color="secondary" textAlign="center">Define custos, margens, impostos e regras de precificação</text>
  </box>
  <row justify="center">
    <icon name="arrow-down" color="tertiary" size="lg"/>
  </row>
  <box background="surface-secondary" radius="lg" padding={3} align="center" gap={1}>
    <icon name="smartphone" color="secondary" size="lg"/>
    **Aplicativo do funcionário**
    <text size="xs" color="secondary" textAlign="center">Informa peças, quantidades, horas de serviço e quilômetros</text>
  </box>
  <row justify="center">
    <icon name="arrow-down" color="tertiary" size="lg"/>
  </row>
  <box background="surface-secondary" radius="lg" padding={3} align="center" gap={1}>
    <icon name="calculator" color="secondary" size="lg"/>
    **Motor de precificação**
    <text size="xs" color="secondary" textAlign="center">Aplica automaticamente os parâmetros cadastrados</text>
  </box>
  <row justify="center">
    <icon name="arrow-down" color="tertiary" size="lg"/>
  </row>
  <box border={{size:1,color:"success"}} radius="lg" padding={3} align="center" gap={1}>
    <icon name="file-check" color="success" size="lg"/>
    **Orçamento final**
    <text size="xs" color="secondary" textAlign="center">Valor de venda, registro e opção de enviar ao cliente</text>
  </box>
</box>

## 2. Como seria a tela do funcionário

A ideia é que ele consiga montar um orçamento em menos de dois minutos, usando o celular.

## 3. Protótipo funcional do orçamento

Monte um orçamento de exemplo abaixo. Você pode adicionar peças, alterar quantidades, informar horas e deslocamento. O resultado será recalculado na hora.

{@body const [itens,setItens] = DIL.useState([{id:1,nome:"Motor compressor",qtd:"1",preco:"350"},{id:2,nome:"Filtro secador",qtd:"1",preco:"25"}])}
{@body const [horas,setHoras] = DIL.useState("2")}
{@body const [km,setKm] = DIL.useState("25")}
{@body const [cliente,setCliente] = DIL.useState("")}
{@body const [equip,setEquip] = DIL.useState("Geladeira")}
{@body const [mao,setMao] = DIL.useState("80")}
{@body const [custoKm,setCustoKm] = DIL.useState("1.5")}
{@body const [markup,setMarkup] = DIL.useState("30")}
{@body const [imposto,setImposto] = DIL.useState("6")}
{@body const [fixo,setFixo] = DIL.useState("0")}
{@body const [aba,setAba] = DIL.useState("tecnico")}
{@body const [mostrar,setMostrar] = DIL.useState(false)}
{@body const n=(v)=>Math.max(0,Number(v)||0)}
{@body const material=itens.reduce((s,i)=>s+n(i.qtd)*n(i.preco),0)}
{@body const custo=material+n(horas)*n(mao)+n(km)*n(custoKm)+n(fixo)}
{@body const venda=custo*(1+n(markup)/100)/(1-Math.min(99,n(imposto))/100)}
{@body const reais=(v)=>v.toLocaleString("pt-BR",{style:"currency",currency:"BRL"})}
<box border radius="2xl" padding={3} gap={3}>
  <row align="center" gap={2}>
    <box background="#172C43" radius="lg" padding={2}>
      <icon name="wrench" color="#FFFFFF" size="lg"/>
    </box>
    <box flex="1" gap="1px">
      **DEXTER**
      <text color="secondary" size="xs">Sistema de Orçamentos Técnicos</text>
    </box>
    <badge color="success">Protótipo</badge>
  </row>
  <segmented-control block={true} size="md" value={aba} onChange={setAba} options={[{label:"Funcionário",value:"tecnico"},{label:"Administrador",value:"admin"}]}/>
  {#if aba==="tecnico"}
    <box gap={1}>
      <label>Nome do cliente</label>
      <input placeholder="Digite o nome do cliente" value={cliente} onChange={setCliente}/>
    </box>
    <box gap={1}>
      <label>Equipamento</label>
      <select block value={equip} onChange={setEquip} options={[{label:"Geladeira",value:"Geladeira"},{label:"Freezer",value:"Freezer"},{label:"Máquina de lavar",value:"Máquina de lavar"},{label:"Micro-ondas",value:"Micro-ondas"},{label:"Câmara fria",value:"Câmara fria"},{label:"Outro",value:"Outro"}]}/>
    </box>
    <divider color="subtle"/>
    <row justify="between" align="center">
      **Peças e materiais**
      <button size="sm" variant="outline" color="secondary" onClick={()=>setItens(v=>[...v,{id:Math.max(0,...v.map(i=>i.id))+1,nome:"",qtd:"1",preco:"0"}])}><icon name="plus" size="xs"/> Adicionar</button>
    </row>
    {#each itens as item,i}
      <box key={item.id} background="surface-secondary" padding={3} radius="lg" gap={2}>
        <row justify="between" align="center">
          <text weight="medium" size="sm">Item {i+1}</text>
          <button size="xs" color="secondary" variant="ghost" onClick={()=>setItens(v=>v.filter(x=>x.id!==item.id))}><icon name="trash-2" size="sm"/></button>
        </row>
        <input placeholder="Nome da peça" value={item.nome} onChange={v=>setItens(a=>a.map(x=>x.id===item.id?{...x,nome:v}:x))}/>
        <grid columns={2} gap={2}>
          <grid-item>
            <box gap={1}>
              <label>Quantidade</label>
              <input inputType="number" value={item.qtd} onChange={v=>setItens(a=>a.map(x=>x.id===item.id?{...x,qtd:v}:x))}/>
            </box>
          </grid-item>
          <grid-item>
            <box gap={1}>
              <label>Custo unitário (R$)</label>
              <input inputType="number" value={item.preco} onChange={v=>setItens(a=>a.map(x=>x.id===item.id?{...x,preco:v}:x))}/>
            </box>
          </grid-item>
        </grid>
      </box>
    {/each}
    <grid columns={2} gap={3}>
      <grid-item>
        <box gap={1}>
          <label>Horas de serviço</label>
          <input inputType="number" value={horas} onChange={setHoras}/>
        </box>
      </grid-item>
      <grid-item>
        <box gap={1}>
          <label>Deslocamento (km)</label>
          <input inputType="number" value={km} onChange={setKm}/>
        </box>
      </grid-item>
    </grid>
    <button block onClick={()=>setMostrar(true)}><icon name="calculator" size="sm"/> Calcular orçamento</button>
    {#if mostrar}
      <box background="surface-secondary" radius="xl" padding={3} gap={2}>
        <text size="xs" color="secondary" weight="medium">VALOR FINAL DO ORÇAMENTO</text>
        <title size="2xl" color="default" tabularNums>{reais(venda)}</title>
        <text color="secondary" size="xs">Valor ilustrativo calculado com os parâmetros do administrador.</text>
        <button block variant="outline" color="secondary" onClick={()=>GenUI.copy(`DEXTER - ORÇAMENTO\nCliente: ${cliente||"Não informado"}\nEquipamento: ${equip}\n${itens.map(i=>`${i.nome}: ${i.qtd} un.`).join("\n")}\nValor total: ${reais(venda)}`)}><icon name="copy" size="sm"/> Copiar orçamento</button>
      </box>
    {/if}
  {:else}
    <text color="secondary" size="sm">Área ilustrativa do proprietário. No aplicativo real, será protegida por login e permissões.</text>
    <box gap={1}>
      <label>Custo da hora técnica (R$)</label>
      <input inputType="number" value={mao} onChange={setMao}/>
    </box>
    <box gap={1}>
      <label>Custo por quilômetro (R$)</label>
      <input inputType="number" value={custoKm} onChange={setCustoKm}/>
    </box>
    <box gap={1}>
      <label>Acréscimo sobre custo (%)</label>
      <input inputType="number" value={markup} onChange={setMarkup}/>
    </box>
    <box gap={1}>
      <label>Impostos sobre venda (%)</label>
      <input inputType="number" value={imposto} onChange={setImposto}/>
    </box>
    <box gap={1}>
      <label>Custo fixo por atendimento (R$)</label>
      <input inputType="number" value={fixo} onChange={setFixo}/>
    </box>
    <box background="surface-secondary" padding={3} radius="lg" gap={2}>
      **Simulação do cálculo**
      <row justify="between">
        <text size="sm">Materiais</text>
        <text tabularNums>{reais(material)}</text>
      </row>
      <row justify="between">
        <text size="sm">Mão de obra</text>
        <text tabularNums>{reais(n(horas)*n(mao))}</text>
      </row>
      <row justify="between">
        <text size="sm">Deslocamento</text>
        <text tabularNums>{reais(n(km)*n(custoKm))}</text>
      </row>
      <row justify="between">
        <text size="sm">Custo fixo</text>
        <text tabularNums>{reais(n(fixo))}</text>
      </row>
      <divider color="subtle"/>
      <row justify="between">
        **Preço de venda**
        **{reais(venda)}**
      </row>
    </box>
  {/if}
</box>

<caption>Este protótipo demonstra o funcionamento do cálculo. Ainda não possui banco de dados, contas de usuário, armazenamento permanente ou integração com o GestãoClick.</caption>

## 4. Estrutura do aplicativo definitivo

| Área | Funcionalidades |
|---|---|
| Funcionário | Criar orçamento, cadastrar peças, horas, deslocamento e visualizar preço |
| Administrador | Configurar precificação, custos, impostos, margens e permissões |
| Banco de peças | Cadastro de peças, custo de aquisição e histórico de preços |
| Orçamentos | Histórico por cliente, técnico, equipamento e status |
| Documentos | Gerar PDF personalizado com a marca Dexter |
| Gestão | Relatórios de faturamento previsto, custos e rentabilidade |

Também recomendo três recursos importantes: aprovação do proprietário para descontos, registro da versão dos parâmetros usados em cada orçamento e funcionamento com sincronização quando o celular voltar a ter internet.

## 5. Como funcionará a fórmula

O aplicativo terá um motor de cálculo configurável, baseado na sua tabela de precificação.

No protótipo, usamos:

\[
C=M+(H\times V_h)+(K\times V_k)+F
\]

\[
P=\frac{C(1+A)}{1-T}
\]

Onde \(M\) é o custo dos materiais, \(H\) são as horas, \(V_h\) o custo por hora, \(K\) os quilômetros, \(V_k\) o custo por quilômetro, \(F\) o custo fixo, \(A\) o acréscimo sobre custos e \(T\) a alíquota de impostos sobre venda.

**Importante:** acréscimo sobre custo e margem de lucro sobre preço de venda são conceitos diferentes. O sistema definitivo deve respeitar a fórmula que você já utiliza na Dexter, em vez de substituir seus critérios pelos valores demonstrativos acima.

## 6. Como transformar isso em um aplicativo real

Minha recomendação é começar com um **aplicativo web instalável (PWA)**. Ele abre no Android como um aplicativo, mas também funciona no computador, permitindo que você gerencie tudo pelo escritório.

Podemos usar o Lovable para construir e publicar a primeira versão com banco de dados, login e permissões. Você pode conectá-lo diretamente pelo ChatGPT.

Para a primeira versão, eu priorizaria a calculadora, o cadastro de peças, o painel administrativo e o histórico de orçamentos. Depois, acrescentaria PDF, aprovação de descontos e eventual integração com o GestãoClick.

**Preciso confirmar um detalhe importante:** você quer que o funcionário digite manualmente o custo de cada peça, ou prefere que ele selecione a peça de uma lista já cadastrada por você, com o preço de custo preenchido automaticamente?

Isso vai definir como construiremos o banco de materiais e o controle de preços da Dexter.

---

If you want, I can:

- Explique detalhadamente como o administrador irá configurar a precificação
- Descreva passo a passo como o funcionário deve criar um orçamento no app
- Liste requisitos técnicos para implementar o banco de peças com preços automatizados