![[Pasted image 20260928084756.png]]
Executado arquivo dentro do autopsy, e fuçando um pouco, encontramos um arquivo PDF.
![[Pasted image 20260928090145.png]]
Aparentemente é a nossa hash.
Feito a comparação e confirmado:
![[Pasted image 20260928090629.png]]
Agora respondendo a pergunta do tamanho do arquivo.
Vemos dentro do arquivo PDF.
![[Pasted image 20260928091144.png]]
Na entrada que começa em `00a0`, temos:
00b0
4C 57 4C 57 00 00 4F 7F 48 57 04 00 E2 01 00 00

Em uma entrada de diretório **FAT**, os últimos 4 bytes da entrada representam o **tamanho do arquivo em bytes**.

Nesse caso:
E2 01 00 00

FAT usa **little-endian**, então devemos inverter a ordem:
00 00 01 E2

Convertendo hexadecimal para decimal:
0x01E2 = 482

R: FIAP{482bytes}

