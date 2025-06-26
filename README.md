## 8. Estoque: Saldo negativo por erro de movimentação

### 🛠 Problema real
No Protheus, mesmo com controle de estoque ativado, é possível gerar **saldo negativo** por diversos motivos: movimentações manuais, apontamentos incorretos de produção, falhas de integração com WMS, ou até ausência de bloqueio para saldo zerado.

Esse tipo de erro impacta diretamente o **controle de inventário, contabilidade, e o planejamento de compras**.

### 📉 Impacto
- Inventário divergente da realidade
- Compras urgentes sem base real
- Produção paralisada por itens que “não existem”
- Problemas na contabilidade e auditoria
- Dificuldade em apurar perdas reais

### 💡 Solução aplicada
Criação de uma rotina que varre a tabela `SB2` (saldo por produto x depósito) e **identifica todos os itens com saldo negativo**. A partir disso, é possível:

1. Gerar relatório de inconsistências
2. Realizar ajuste automático (opcional e controlado)
3. Alimentar um painel para acompanhamento diário

> A solução pode ser integrada com um alerta diário ou dashboard interno.

### 🧾 Código exemplo (ADVPL)
```advpl
User Function VerificaSaldoNegativo()
    Local cArquivo := "C:\relatorios\SaldoNegativo.csv"
    Local cLinha := "Produto;Descrição;Depósito;Saldo Atual" + CRLF

    dbSelectArea("SB2")
    dbSetOrder(1)
    dbGoTop()

    While !Eof()
        If SB2->B2_QATU < 0
            cLinha += SB2->B2_COD + ";" +;
                      SB1->B1_DESC + ";" +;
                      SB2->B2_LOCAL + ";" +;
                      Str(SB2->B2_QATU,10,2) + CRLF
        EndIf
        dbSkip()
    EndDo

    MemoWrite(cArquivo, cLinha)
    MsgInfo("Relatório de saldo negativo gerado com sucesso.")
Return

```
> Caso o saldo precise ser ajustado automaticamente (ex: zerar saldo para inventário), é possível estender a rotina com movimentações automáticas de entrada via SD3.

## 🧪 Testes realizados - Monitoramento de Saldo Negativo

| Situação               | Resultado Esperado             | Status  | Lógica Aplicada                  |
|------------------------|--------------------------------|---------|----------------------------------|
| Produto com saldo -5   | Incluído no relatório          | ✅      | Saldo negativo = atenção necessária |
| Produto com saldo 0    | Ignorado pelo sistema          | 🚫      | ZERO = sem necessidade de ação    |
| Produto com saldo +10  | Ignorado pelo sistema          | 🚫      | Positivo = situação normal       |

> Explicação técnica:
**Parâmetros Considerados:**
- Tipo de relatório: `EstoqueNegativo.rpt`
- Filtro aplicado: `WHERE SALDO_ATUAL < 0`
- Tabelas verificadas: `SB1` (cadastro), `SB2` (saldo)

**Próximas melhorias:**
> - Adicionar filtro por depósito (`SB2.DEPTO`)
> - Incluir coluna "Dias em negativo"
> - Opção para exportar em CSV

 Benefícios
Visão clara de erros de movimentação

Base para ações corretivas ou ajustes manuais

Suporte à auditoria e inventário rotativo

Prevenção de falhas na produção e compras

🏷️ Tags
#Protheus #Estoque #SB2 #SaldoNegativo #ControleEstoque #ADVPL #Backoffice
