# Modelo de cobrança — rascunho para discussão

28/10/2026 · versão 0.1

Este documento organiza as decisões que faltam antes do código do motor de política: como cobramos do credor, o que o agente faz em cada faixa de atraso, o que conta como "recuperado por nós" e com quais credores começamos. Tudo aqui é proposta. Os números marcados como **hipótese** saem do piloto.

## Resumo das propostas

| Decisão | Proposta | Por quê |
|---|---|---|
| Preço | Taxa por faixa de atraso, com carência sem taxa nos primeiros dias | Não cobrar pelo boleto que seria pago de qualquer jeito |
| Atribuição | Só conta o pagamento de título que entrou na fila do agente depois da carência | Regra simples que o credor entende e confere |
| Medição | Grupo de controle: parte dos títulos não é trabalhada pelo agente | É o único jeito honesto de provar que o agente recupera mais |
| Política padrão | Desconto só em juros e multa, até 3 parcelas, humano acima de um valor | Credor começa com um padrão seguro e ajusta |
| Primeiros clientes | Distribuidores da base do ERP com pelo menos 30 faturas vencidas por mês | Abaixo disso a receita por cliente não passa de R$ 1.000 |

## 1. Faixas de atraso

Cada título passa por faixas. A faixa define o que o agente pode fazer e quanto cobramos.

| Faixa | Dias em relação ao vencimento | O que o agente faz | Taxa (hipótese) |
|---|---|---|---|
| Lembrete | D-3 a D0 | Lembrete de utilidade com número da nota, valor e boleto | 0% |
| Carência | D+1 a D+5 | Um lembrete de utilidade com a segunda via. Não negocia | 0% |
| Negociação | D+6 a D+30 | Conversa, entende o motivo, oferece prazo, desconto de juros ou parcelamento | 5% |
| Recuperação | D+31 a D+90 | Mesmo que a anterior, com mais margem de desconto se o credor permitir. Sugere bloquear novas vendas a prazo | 10% |
| Fora do escopo | D+91 em diante | Fora do MVP. Devolve ao credor com o histórico | A negociar (hipótese: 15%) |

**Carência de 5 dias** é um chute. O certo é cada credor ter a sua, calculada pelo histórico: o dia em que a maior parte dos atrasos já se resolveu sozinha. Se a curva de pagamento do ERP mostrar que 70% dos atrasos se pagam até D+3, a carência é 3 dias.

## 2. Preço

### Opções consideradas

| Opção | Como funciona | Problema |
|---|---|---|
| A. 10% fixo sobre todo recuperado | Do documento-base | Cobra 10% de boleto atrasado 2 dias que seria pago de qualquer jeito. O credor percebe e cancela |
| **B. Por faixa, com carência** | Tabela da seção 1 | Receita por cliente menor que na opção A (conta abaixo) |
| C. Valor fixo por fatura no atraso curto e % no longo | Ex.: R$ 5 por fatura até D+30, 10% depois | Volta a ter custo mesmo quando não recupera, que é a objeção que queríamos tirar |
| D. Mensalidade baixa mais taxa menor | Ex.: R$ 200/mês mais 5% | Bom para cliente maior depois do piloto, não para o piloto |

**Proposta: opção B no piloto e D como evolução**, como o documento-base já previa.

### O que a opção B faz com a receita

Hipóteses: 50% do valor vencido se paga sozinho até o fim da carência; o agente recupera 40% do restante; metade do recuperado cai na faixa de 5% e metade na de 10%.

| Cenário | Valor vencido/mês | Sobra após a carência | Agente recupera | Receita/mês | Mesmo cenário com 10% fixo (documento-base) |
|---|---|---|---|---|---|
| Pequeno: 20 faturas de R$ 1.500 | R$ 30.000 | R$ 15.000 | R$ 6.000 | **R$ 450** | R$ 1.200 |
| Médio: 60 faturas de R$ 3.584 | R$ 215.040 | R$ 107.520 | R$ 43.008 | **R$ 3.226** | R$ 8.602 |

Conclusão: o cliente pequeno não passa do portão de R$ 1.000/mês. **O piloto precisa de distribuidores médios.** Isso muda o critério de escolha dos primeiros clientes (seção 5).

### Detalhes a decidir

- **Base da taxa:** proposta é sobre o valor efetivamente recebido, juros e multa incluídos. Se o agente der desconto, a taxa cai junto, o que alinha o nosso incentivo ao do credor.
- **Pagamento parcial e parcelas:** taxa sobre cada parcela paga, na faixa em que o título estava quando o acordo foi fechado.
- **Teto por fatura:** avaliar um teto (ex.: R$ 500) para faturas grandes, que vão para humano de qualquer jeito.

## 3. O que conta como "recuperado por nós"

Proposta de regra, para escrever no contrato do piloto:

1. Só conta o título que **entrou na fila do agente**, ou seja, passou da carência sem pagamento.
2. Conta o pagamento que acontece até **60 dias depois da última mensagem do agente** sobre aquele título.
3. **Não conta:** nota cancelada, devolução de mercadoria, abatimento combinado pelo vendedor, título que o credor tirou da fila antes do pagamento.
4. Se um humano do credor assumiu a conversa e fechou o acordo, **conta pela metade** da taxa. A alternativa é não contar, mas aí o agente fica penalizado por escalar quando deveria.

O ponto 4 é o mais discutível. Precisa ser testado na conversa com o credor.

## 4. Grupo de controle

Comparar com o histórico dos últimos 6 meses é fraco: muda a época, muda a carteira e o credor pode ter mudado o jeito de cobrar. Proposta mais forte:

- No piloto, **20% a 30% dos títulos vencidos são sorteados para ficar fora** do agente e seguir a cobrança atual do credor.
- Comparamos recuperação e prazo médio entre os dois grupos, na mesma época e com a mesma carteira.
- **Não cobramos nada sobre o grupo de controle.**

Se o agente recuperar mais, esse é o melhor argumento de venda que vamos ter. Se o credor não aceitar deixar títulos de fora, voltamos ao histórico.

## 5. Política padrão do credor

O credor começa com este padrão e ajusta. Tudo vira configuração no motor de política, sem IA.

| Regra | Padrão proposto |
|---|---|
| Desconto | Só em juros e multa, até 100%. Principal sem desconto |
| Parcelamento | Até 3 parcelas, entrada mínima de 30% |
| Parcela mínima | R$ 200 |
| Prazo extra sem parcelar | Até 15 dias |
| Passa para humano | Valor acima de R$ 10.000, contestação da nota, devedor irritado, pedido fora da política |
| Horário de envio | Segunda a sexta das 8h às 18h, sábado das 8h às 12h |
| Frequência | No máximo 1 mensagem iniciada pelo agente por dia por devedor |
| Modo aprovação | Ligado nas 2 primeiras semanas: o agente sugere e um humano do credor aprova |
| Bloqueio de venda a prazo | O agente nunca bloqueia. Só sugere ao credor no painel |

## 6. Com quem começar

### Perfil do primeiro cliente

- Distribuidor que usa o ERP de autopeças e pneus.
- Vende a prazo por boleto para oficinas e borracharias.
- **Pelo menos 30 faturas vencidas por mês** (pela conta da seção 2).
- Hoje cobra à mão: dono, financeiro ou vendedor mandando WhatsApp.
- O dono atende o telefone e decide sozinho.

**Bom sinal:** já pagou assessoria de cobrança ou já negativou cliente. Quer dizer que a dor existe e ele aceita pagar por ela.
**Fora do perfil:** vende quase tudo à vista ou no cartão, ou tem menos de 10 faturas vencidas por mês.

### Como escolher os 10 primeiros

Se o ERP permitir uma consulta agregada (só contagem, sem dado de cliente), ordenar a base por número de títulos vencidos no último trimestre e pegar os 15 maiores. Daí sai a lista de 10 conversas.

### Segundo nicho, para testar cedo

Hipóteses com o mesmo formato (distribuidor vendendo a prazo para muitos pequenos clientes recorrentes): material de construção para lojas e empreiteiros, bebidas e alimentos para bares e restaurantes, insumos agrícolas para produtores. A escolha depende de qual tem um segundo ERP de nicho acessível.

## 7. Perguntas para as primeiras conversas

No estilo do Mom Test: perguntar sobre o que já aconteceu, não sobre o que a pessoa faria.

1. Quantos boletos venceram sem pagamento no mês passado? Quanto isso somava?
2. Quem cobrou esses boletos? Quanto tempo isso tomou na semana?
3. Dos que atrasaram, quantos pagaram sozinhos na primeira semana?
4. Qual foi a última vez que você parcelou ou deu desconto para um cliente atrasado? Como foi?
5. Já perdeu um cliente por causa de cobrança? Já deixou de cobrar alguém para não perder?
6. Já usou assessoria, negativação ou protesto? Quanto pagou?
7. O que acontece hoje com um cliente que está atrasado e quer comprar de novo?

Compromisso a pedir no fim: a lista dos títulos vencidos do último mês para rodar o piloto.

## 8. O que isso muda no código

- **Título** ganha `faixa_atual`, `entrou_na_fila_em`, `grupo` (agente ou controle) e `motivo_exclusao`.
- **Política do credor** ganha carência, tabela de faixas com taxa, regras da seção 5 e janela de atribuição.
- **Pagamento** ganha `faixa_no_acordo`, `atribuido` (sim, não, metade) e `taxa_calculada`.
- **Motor de política** calcula a faixa, valida ofertas e calcula a taxa de cada pagamento.

## Decisões abertas

- [ ] Carência fixa de 5 dias ou calculada por credor?
- [ ] Taxas de 5% e 10% por faixa: validar nas conversas.
- [ ] Pagamento fechado por humano conta pela metade, inteiro ou não conta?
- [ ] O credor aceita grupo de controle?
- [ ] Teto de taxa por fatura?
- [ ] Mínimo de 30 faturas vencidas por mês como corte do primeiro cliente.
- [ ] Qual segundo nicho testar.
