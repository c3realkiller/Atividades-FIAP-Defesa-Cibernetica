![[Pasted image 20260928091516.png]]
Aparenta que o sistema seja um linux.
![[Pasted image 20260928093246.png]]
Como falou sobre o arquivo, possivelmente está se referindo ao estudo MBR.
![[Pasted image 20260929093113.png]]
![[Pasted image 20260929094454.png]]
Como a imagem não exibe a numeração dos endereços das linhas à esquerda, usamos a assinatura final `55 AA` (que obrigatoriamente fica nos dois últimos bytes, colunas `0E` e `0F`, da última linha) como âncora para mapear as posições. A tabela de partições sempre começa no byte 446 (Offset hexadecimal `0x01BE`) e contém quatro partições, ocupando 16 bytes cada.

Devido à formatação em blocos de colunas de `00` a `0F` (16 bytes por linha), as entradas das partições na imagem estão divididas entre o final de uma linha e o começo da outra.

A primeira partição começa nos dois últimos bytes (colunas `0E` e `0F`) da linha que contém a assinatura de disco `4E C7 6D 7D`

Os 16 bytes completos da 1ª partição são formados por `00 80` (final dessa linha) somados a `01 04 83 2B 02 2C 00 08 00 00 00 50 00 00` (os 14 primeiros bytes da linha de baixo).

O byte que define se a partição é bootável (ativa) é sempre o _primeiro_ byte dos 16. Nesta entrada (coluna `0E`), o valor é `00`, o que significa que a partição está tecnicamente **inativa**. O valor `80` na coluna `0F` é, na verdade, o segundo byte da estrutura, que representa a Cabeça (Head) inicial do endereçamento físico do disco

O quinto byte desta partição é o `83` (localizado na coluna `02` da linha de baixo), o que identifica uma partição Linux.

Então assim, podemos ver que apresenta o 80 demonstrando estar ativo a partição 1, o 83 representando o linux, e somando o valor do tamanho de cada partição que é definido pelos últimos 4 bytes.

Os últimos 4 bytes da primeira partição encontram-se nas colunas `0A`, `0B`, `0C` e `0D` da primeira linha completa de partições, apresentando os valores `00 50 00 00`.
Onde precisa ser convertido em little ending, lendo da direita para a esquerda, o valor hexadecimal real é `00 00 50 00` (ou apenas `0x5000`).
**Hexadecimal para Decimal:** O valor `5000` em hexadecimal equivale ao número decimal 20.480.

**Cálculo em Bytes:** 20.480 setores × 512 bytes por setor = **10.485.760 bytes**.

**Cálculo em Megabytes (MB):** 10.485.760 bytes divididos por 1024 (para KB) e depois por 1024 novamente resulta em exatos **10 MB**.


R:FIAP{sim_linux_10mb}