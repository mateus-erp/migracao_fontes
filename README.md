
# Migração dos fontes AdvPL/TLPP

Esta documentação contêm os erros e como corrigi-los por categoria.

### SX3 - Uso NÃO PERMITIDO de leitura do metadados
Exemplo:
```
Local aHeader := {}

dbSelectArea('SX3')
SX3->(dbSetOrder(1))
DbSeek('ZZM')
While !EOF() .AND. X3_ARQUIVO == "ZZM"
    aAdd(aHeader,{TRIM(X3_TITULO),X3_CAMPO,X3_PICTURE,X3_TAMANHO,X3_DECIMAL,X3_VALID,X3_USADO,X3_TIPO,X3_ARQUIVO,X3_CONTEXT})
    SX3->(dbSkip())
EndDo
SX3->(dbCloseArea())
```

Como corrigir a violação:
```
Local aCampos := FWSX3Util():GetAllFields("ZZM", .T.)
Local nCount  := 1
Local aHeader := {}

For nCount := 1 To Len(aCampos)
    aAdd(aHeader, { GetSx3Cache(aCampos[nCount], "X3_TITULO"),; 
                    GetSx3Cache(aCampos[nCount], "X3_CAMPO"),;
                    GetSx3Cache(aCampos[nCount], "X3_PICTURE"),; 
                    GetSx3Cache(aCampos[nCount], "X3_TAMANHO"),;
                    GetSx3Cache(aCampos[nCount], "X3_DECIMAL"),;
                    GetSx3Cache(aCampos[nCount], "X3_VALID"),;
                    GetSx3Cache(aCampos[nCount], "X3_USADO"),;
                    GetSx3Cache(aCampos[nCount], "X3_TIPO"),;
                    GetSx3Cache(aCampos[nCount], "X3_ARQUIVO"),;
                    GetSx3Cache(aCampos[nCount], "X3_CONTEXT")})
Next 
```

### Chamada descontinuada de Driver ISAM
Exemplo:

```
_stru:={}
AADD(_stru, {"N3_CBASE","C",10,0})
AADD(_stru, {"N3_ITEM","C",04,0})

cArq := Criatrab(_stru,.T.)
DBUSEAREA(.t.,,carq,"TP5")
INDEX ON N3_CBASE+N3_ITEM TO &CARQ
SET INDEX TO &CARQ
```

Como corrigir a violação:
```
Local oTmp := FWTemporaryTable():New("TP5")

_stru:={}
AADD(_stru, {"N3_CBASE","C",10,0})
AADD(_stru, {"N3_ITEM","C",04,0})

oTmp:SetFields(_stru)
oTmp:AddIndex("01",{"N3_CBASE", "N3_ITEM"})
oTmp:Create()

// usar no final da execução do programa para deletar a tabela
oTmp:delete()
```

### Uso NÃO PERMITIDO de API em LOOP
Exemplo:
```
Local lRegra := .T.
Local cTipo  := "A"

While lRegra
    If cTipo == getMv("ES_SMTSRV")
        ...
    EndIf
EndDo
```

Como corrigir a violação:
```
Local lRegra := .T.
Local cTipo  := "A"
Local cParam := getMv("ES_SMTSRV")

While lRegra
    If cTipo == cParam
        ...
    EndIf
EndDo
```

### SX5 - Uso DESCONTINUADO de leitura do metadados
Exemplo:
```
SX5->(dbSeek(xFilial("SX5")+"16"))
While SX5->X5_FILIAL+SX5->X5_TABELA == xFilial("SX5")+"16"
    ...
EndDo
```

Como corrigir a violação:
```
Local aGenericos := FWGetSX5("16")
Local nCount     := 1

For nCount := 1 To Len(aGenericos)
    aGenericos[nCount][1]--> FILIAL
    aGenericos[nCount][2]--> TABELA
    aGenericos[nCount][3]--> CHAVE
    aGenericos[nCount][4]--> DESCRICAO
    ...
Next
```

### SX1 - Uso NÃO PERMITIDO de atribuição do metadados
Não existe API/Função disponível para atribuição de metadados da SX1.

### Uso NÃO PERMITIDO de API em TRANSAÇÃO
Exemplo:
```
Local lCondicao := .T.

Begin Transaction
    If lCondicao 
        return 
    EndIf
End Transaction
```

Como corrigir a violação:
```
Local lCondicao := .T.

Begin Transaction
    If lCondicao
        // Se caso, precisar dar rollback
        DisarmTransaction()

        // Finaliza a transação e desvia para depois do próximo comando END TRANSACTION 
        Break 
    EndIf
End Transaction
```

### SX7 - Uso NÃO PERMITIDO de leitura do metadados
Não existe API/Função disponível para atribuição de metadados da SX7.

### SX1 - Uso NÃO PERMITIDO de leitura do metadados 
Exemplo:
```
Local cPerg := "ATA010"

dbSelectArea("SX1")
dbSetOrder(1)
dbSeek(cPerg)
While !Eof() .And. SX1->X1_GRUPO == cPerg
    ...
EndDo
SX1->(dbCloseArea)
```

Como corrigir a violação:
```
Local oObj      := FWSX1Util():New()
Local aPergunte := {}
Local nCount    := 1

oObj:AddGroup("ATA010")
oObj:SearchGroup()

aPergunte := oObj:GetGroup("ATA010")
For nCount := 1 To Len(aPergunte)
    ...
Next
```

### SXG - Uso NÃO PERMITIDO de leitura do metadados 
Não existe API/Função disponível para atribuição de metadados da SXG.

### SX2 - Uso NÃO PERMITIDO de leitura do metadados 
Exemplo:
```
Local cUnico := ""

dbSelectArea("SX2")
dbSeek("SB1")	
While SX2->X2_CHAVE == "SB1"
    cUnico := SX2->X2_UNICO
    SX2->(dbSkip())
EndDo
SX2->(dbCloseArea())
```

Como corrigir a violação:
```
Local aReturn := FwSX2Util():GetSX2data('SB1', {"X2_UNICO", "X2_ARQUIVO"})
Local nCount  := 1

For nCount := 1 To Len(aReturn)
    aReturn[nCount][1]--> Campo da SX2
    aReturn[nCount][2]--> Conteúdo do campo
Next 
```

### SX6 - Uso DESCONTINUADO de leitura do metadados 
Não existe API/Função disponível para leitura de toda a tabela, mas existe a função GetMv() e a SuperGetMv para buscar um parâmetro especificado na SX6.

### SX6 - Uso DESCONTINUADO de atribuição do metadados
Exemplo:
```
Local cParam := "MV_PARAM"

dbSelectArea("SX6")
SX6->(dbSetOrder(1))
If !DBSEEK(cParam)
    RecLock("SX6",.T.)
        SX6->X6_VAR	:= 1
    SX6->(MsunLock())
EndIf
SX6->(dbCloseArea())
```

Como corrigir a violação:
```
PUTMV("MV_PARAM", 1)
```

### SX5 - Uso DESCONTINUADO de atribuição do metadados
Exemplo:
```
dbSelectArea("SX5")
SX5->(dbSetOrder(1))
If dbSeek(xFilial("SX5")+"01"+"CTR011")
    RecLock("SX5",.F.)
        SX5->X5_DESCRI := "TABELAS DE OCORRENCIAS ATIVO"
    SX5->(MsunLock())
EndIf
SX5->(dbCloseArea())
```

Como corrigir a violação:
```
FwPutSX5(/*cFlavour*/, "01", "CTR011", "TABELAS DE OCORRENCIAS ATIVO", "TABLA DE EVENTOS DEL ACTIVO", "OCCURRENCES TABLE - ASSETS")
```

### SX9 - Uso NÃO PERMITIDO de leitura do metadados
Em análise

### Erro de compilação
Deve ser realizado a analise se a include não foi disponibilizada no code analysis ou se a mesma está com problema de sintaxe. Em caso de sintaxe, deve ser removida do projeto.
