# 🗄️ SQL Server: Auditoria de Dados e Otimização de Consultas (ERP & PEP)

Exemplos práticos de scripts em **T-SQL (SQL Server)** aplicados à auditoria de logs, rastreabilidade de Prontuário Eletrônico do Paciente (PEP), integridade de movimentações e prevenção de perdas financeiras.

---

## 🎯 O Problema de Negócio
Falhas de parametrização ou erros operacionais no dia a dia hospitalar (como prescrições checadas sem lançamento na conta do paciente) geram evasão de receita silenciosa e gargalos de performance no banco de dados.

## 💡 A Solução Técnica
Criação de rotinas de auditoria preventiva em SQL Server para identificar divergências entre a assistência (PEP) e o faturamento antes do fechamento da conta.

---

## 💻 Script Fictício: Auditoria de Prescrições Checadas vs. Conta Faturada

```sql
/* ============================================================================
   OBJETIVO: Identificar medicamentos checados pela enfermagem no PEP 
             que não foram lançados na conta de faturamento do paciente.
   AMBIENTE: SQL Server (Estrutura demonstrativa fictícia)
   ============================================================================ */

SELECT 
    A.ID_ATENDIMENTO,
    P.NOME_PACIENTE_MASCARADO AS PACIENTE,
    C.NOME_CONVENIO AS CONVENIO,
    PR.DATA_CHECAGEM,
    M.CODIGO_ITEM,
    M.DESCRICAO_MEDICAMENTO,
    PR.QUANTIDADE_APLICADA
FROM TB_PEP_PRESCRICAO_CHECAGEM PR
INNER JOIN TB_ATENDIMENTO A 
    ON PR.ID_ATENDIMENTO = A.ID_ATENDIMENTO
INNER JOIN TB_PACIENTE P 
    ON A.ID_PACIENTE = P.ID_PACIENTE
INNER JOIN TB_CONVENIO C 
    ON A.ID_CONVENIO = C.ID_CONVENIO
INNER JOIN TB_CADASTRO_MATERIAIS M 
    ON PR.ID_ITEM = M.ID_ITEM
LEFT JOIN TB_CONTA_PACIENTE_ITENS F 
    ON PR.ID_ATENDIMENTO = F.ID_ATENDIMENTO 
    AND PR.ID_ITEM = F.ID_ITEM
    AND CAST(PR.DATA_CHECAGEM AS DATE) = CAST(F.DATA_LANCAMENTO AS DATE)
WHERE 
    PR.STATUS_CHECAGEM = 'APLICADO'
    AND F.ID_LANCAMENTO_CONTA IS NULL
    AND A.DATA_ALTA IS NULL -- Contas ainda abertas para correção preventiva
ORDER BY 
    PR.DATA_CHECAGEM DESC;
