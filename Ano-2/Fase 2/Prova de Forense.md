![[Pasted image 20260929094612.png]]

A segunda partição começa exatos 16 bytes após o início da primeira, localizando-se no final da linha seguinte.

Seus 16 bytes iniciam nas colunas `0E` e `0F` com os valores `00 2C`

Os 14 bytes restantes continuam na linha imediatamente abaixo: `01 2C 07 53 02 54 00 58 00 00 00 50 00 00`.

Nesta partição, o primeiro byte também é `00` (inativa) e o quinto byte é `07` (indicando um sistema de arquivos NTFS ou exFAT).

Onde verificando, como começa com 00 a resposta é não, 07 sendo NTFS.
E o valor do tamanho de cada partição é definido pelos últimos 4 bytes,  onde vemos que é o mesmo valor da primeira partição.
Sendo a resposta.
R: FIAP{nao_NTFS_10mb}

