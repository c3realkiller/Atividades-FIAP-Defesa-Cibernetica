![[Pasted image 20260927120741.png]]
![[Pasted image 20260927121656.png]]

Apresenta um código:
RkYgRDggRkYgRTAgMDAgMTAgNEEgNDYgNDkgNDYgMDAgMDEgMDEgMDEgMDAgNkEgMDAgNkEgMDAg

MDAgRkYgREIgMDAgNDMgMDAgMDMgMDIgMDIgMDMgMDIgMDIgMDMgMDMgMDMgMDMgMDQgMDMgMDMg

MDQgMDUgMDggMDUgMDUgMDQgMDQgMDUgMEEgMDcgMDcgMDYgMDggMEMgMEEgMEMgMEMgMEIgMEEg

MEIgMEIgMEQgMEUgMTIgMTAgMEQgMEUgMTEgMEUgMEIgMEIgMTAgMTYgMTAgMTEgMTMgMTQgMTUg

MTUgMTUgMEMgMEYgMTcgMTggMTYgMTQgMTggMTIgMTQgMTUgMTQgRkYgREIgMDAgNDMgMDEgMDMg

MDQgMDQgMDUgMDQgMDUgMDkgMDUgMDUgMDkgMTQgMEQgMEIgMEQgMTQgMTQgMTQgMTQgMTQgMTQg

MTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQg

MTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQgMTQg

MTQgMTQgMTQgMTQgMTQgMTQgRkYgQzAgMDAgMTEgMDggMDIgMTEgMDEgOTkgMDMgMDEgMjIgMDAg

MDIgMTEgMDEgMDMgMTEgMDEgRkYgQzQgMDAgMUYgMDAgMDAgMDEgMDUgMDEgMDEgMDEgMDEgMDEg

MDEgMDAgMDAgMDAgMDAgMDAgMDAgMDAgMDAgMDEgMDIgMDMgMDQgMDUgMDYgMDcgMDggMDkgMEEg

MEIgRkYgQzQgMDAgQjUgMTAgMDAgMDIgMDEgMDMgMDMgMDIgMDQgMDMgMDUgMDUgMDQgMDQgMDAg

MDAgMDEgN0QgMDEgMDIgMDMgMDAgMDQgMTEgMDUgMTIgMjEgMzEgNDEgMDYgMTMgNTEgNjEgMDcg

MjIgNzEgMTQgMzIgODEgOTEgQTEgMDggMjMK

Convertido no cyberchef, sendo um base64
![[Pasted image 20260927121941.png]]
Onde nos gerou um JPEG aparentemente.
![[Pasted image 20260927122115.png]]
Utilizado o xxd para gerar essa imagem:
![[Pasted image 20260927122143.png]]
Mas está quebrado a imagem.
Utilizando o filtro do wireshark por arquivos mais pesados.
![[Pasted image 20260927124725.png]]
Onde podemos pegar exatamente a comunicação feita:
![[Pasted image 20260927124751.png]]
Convertido:
![[Pasted image 20260927124806.png]]
Mas isso não nos levou a nada.
Voltado e filtrado por conexões SYN, ACK e colocado  no filtro expert para analisarmos, vimos uma porta estranha sendo usada:
![[Pasted image 20260927220855.png]]
![[Pasted image 20260927221024.png]]
Convertido para hex dump e jogado para um arquivo .txt
![[Pasted image 20260927224026.png]]
Onde analisado o arquivo começa por:
```
00000000 42 b1 c1 15 52 d1 f0 24 ... 
00000010 18 19 1a 25 ...
```
Isso não parece texto nem código de shell. Então a primeira coisa é procurar **assinaturas de arquivos** e **marcadores conhecidos**.
**Procurei marcadores `FF` característicos de JPEG**

No dump aparecem:
```
ff c4
ff c4
ff da
...
ff d9
```

Marcadores:

|   |   |
|---|---|
|`FF D8`|início do JPEG — SOI|

|   |   |
|---|---|
|`FF C4`|tabela Huffman — DHT|

|   |   |
|---|---|
|`FF DA`|início dos dados da imagem — SOS|

|   |   |
|---|---|
|`FF D9`|fim do JPEG — EOI|
O problema é que **não existe `FF D8` no começo**.

Isso indica que o arquivo está **truncado/corrompido no início**, mas o restante contém uma imagem JPEG.

Onde identificamos o trecho:
```
00000160  fa ff da 00 0c 03 01 00 02 11 03 11 00 3f 00 f8
```
Depois dele começa a informação comprimida da imagem.

Um JPEG normal teria algo parecido com:
```
FF D8
FF E0 ...
JFIF
...
FF DB ...
FF C0 ...
FF C4 ...
FF DA
```
No nosso dump, começamos praticamente no meio dessas estruturas.

A estrutura recuperada precisava receber um cabeçalho antes dos bytes fornecidos.

Onde feito a conversão do hexdump:
```
awk '{
    for (i=2; i<=17 && $i ~ /^[0-9A-Fa-f][0-9A-Fa-f]$/; i++)
        printf "%s", $i
    printf "\n"
}' dump.txt > payload.hex
```
E depois para binário:
```
xxd -r -p payload.hex payload.bin
```

e criado nosso header:
![[Pasted image 20260927224547.png]]
```
ffd8ffe000104a46494600010100000100010000ffdb004300080606070605080707070909080a0c140d0c0b0b0c1912130f141d1a1f1e1d1a1c1c20242e2720222c231c1c2837292c30313434341f27393d38323c2e333432ffdb0043010909090c0b0c180d0d1832211c213232323232323232323232323232323232323232323232323232323232323232323232323232323232323232323232323232323232ffc0001108011801a003012200021101031101ffc4001f0000010501010101010100000000000000000102030405060708090a0bffc400b5100002010303020403050504040000017d01020300041105122131410613516107227114328191a10823
```
E convertido para bin.
`xxd -r -p header.hex header.bin`
Onde apresenta ser o jpg.
![[Pasted image 20260927224702.png]]

O cabeçalho reconstruído ficou:
```
FF D8
FF E0 00 10 4A 46 49 46 ...
FF DB ...
FF DB ...
FF C0 ...
FF C4 ...
FF C4 ...
```
O ponto importante é que **não alteramos os dados da imagem que estavam no desafio**. Apenas reconstruímos a parte inicial que estava ausente.

No cabeçalho reconstruído aparece:
```
FF C0 00 11 08 01 18 01 A0 ...
```
Onde interpretado:
```
01 18 = 280
01 A0 = 416
```
Sendo altura e largura.

No JPEG recuperado, o deslocamento fica:
```
00000100  42 b1 c1 15 52 d1 f0 24 ...
```
O cabeçalho adicionado tinha **0x100 bytes = 256 bytes**.

Agora juntando os dois:
```
cat header.bin payload.bin > recuperado.jpg
```
Depois de juntar:
```
HEADER_JPEG + DUMP
```
o arquivo passa a possuir:
```
FF D8    → início
...
FF DA    → dados da imagem
...
FF D9    → final
```
Então ele pode ser aberto normalmente como JPEG.
![[Pasted image 20260927224817.png]]

Resposta: FIAP{WIR3SH4RK}
