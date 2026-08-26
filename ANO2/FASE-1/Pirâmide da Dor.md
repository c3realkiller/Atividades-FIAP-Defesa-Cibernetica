![[Pasted image 20260822014643.png]]
![[dog.jpg]]

Utilizado o XXD para analisar se o final do arquivo realmente bate um jpg.
![[Pasted image 20260825225028.png]]
![[Pasted image 20260825225045.png]]
Neste caso, o arquivo precisa ser cortado logo após o marcador `FF D9`
O marcador `FF D9` termina exatamente no offset hexadecimal `0x78DA`
Utilizado o comando:
```
head -c 30940 dog.jpg > dog_original.jpg
md5sum dog_original.jpg
```
Onde obtemos o resultado:
FIAP{d3580a86012b483ec1a738bbdfaba4ef}

O atacante usou uma técnica muito simples: ele pegou a imagem original e apenas "colou" alguns bytes nulos (`0000 0000...`) no final do arquivo. Como a imagem continua abrindo normalmente (porque o visualizador para no `FF D9`), a olho nu nada mudou, mas o hash MD5 mudou completamente, quebrando a integridade.

Para reverter isso, precisávamos descobrir o tamanho exato da imagem original. Onde analisamos a última linha do `xxd`

Como as posições (offsets) começam a contar do zero, se o último byte está na posição `78db`, o tamanho total do arquivo em hexadecimal é `78dc` (um a mais).

O comando `head -c 30940 dog.jpg > dog_original.jpg` pegou os exatos 30.940 bytes originais da imagem (terminando cirurgicamente no `FF D9`) e descartou todo o "lixo" (`0000 0000`) que o atacante havia colocado no final.