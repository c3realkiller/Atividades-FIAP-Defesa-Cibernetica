![[Pasted image 20260930225655.png]]
Identificado o profile:
![[Pasted image 20261001003116.png]]
Utilizado o pslist que é para listar os processos que estavam em execução no sistema no momento da captura.
![[Pasted image 20261001150215.png]]
E vemos o que chama atenção:
![[Pasted image 20261001003458.png]]
Depois disso, procurei arquivos associados ao administrador da maquina.
Analisando os handles, filtrando por arquivos.
![[Pasted image 20261001150518.png]]
Vemos que tem um arquivo dump de memória.
```
C:\Documents and Settings\Administrador\Desktop\memdump.mem
```
Como está relacionado ao administrador, procurado informações sobre ele em sua área de trabalho.
![[Pasted image 20261001150735.png]]
Oq chama atenção é o CMD.PNG
Vamos recuperar esse arquivo.
![[Pasted image 20261001150901.png]]
![[Pasted image 20261001151147.png]]
Agora precisamos ver o conteúdo de `cmd.png`, porque o nome sugere que ela pode mostrar justamente o comando/procedimento usado para localizar e remontar o arquivo excluído.
![[Pasted image 20261001151401.png]]
Em sistemas de arquivos FAT, quando um arquivo é excluído, o sistema operacional não apaga os dados imediatamente. Em vez disso, ele altera o primeiro caractere do nome do arquivo (no registro do diretório) para o byte hexadecimal **`E5`** (que no texto ASCII aparece como `å`). Analisando o dump hexadecimal da imagem, encontramos exatamente isso no início de uma das linhas de registro
`E5 4F 54 4F 28 35 7E 31 4A 50 47` (que corresponde a `åOTO(5~1JPG`).

Em uma entrada de diretório FAT de 32 bytes, a arquitetura reserva exatamente os **4 últimos bytes** (offsets 28 a 31) para registrar o tamanho total do arquivo. Se contarmos os blocos dessa entrada do arquivo `E5` até o final dos 32 bytes, chegamos na área imediatamente após a caixa vermelha (`0D 00`, que indica o cluster inicial). Os 4 últimos bytes desta entrada são: **`16 27 01 00`**

Agora aplicamos a regra de inversão de arquitetura que você mencionou:

- Lemos os bytes de trás para frente: `00 01 27 16`
    
- Juntamos o valor em hexadecimal: `0x012716`
    
- Convertendo o hexadecimal `12716` para decimal:

- $1 \times 16^4 = 65536$

- $2 \times 16^3 = 8192$

- $7 \times 16^2 = 1792$
 
- $1 \times 16^1 = 16$

- $6 \times 16^0 = 6$
  
- Soma total: $65536 + 8192 + 1792 + 16 + 6 = 75542$

O tamanho exato do arquivo excluído é 75.542 bytes.
R:FIAP{75542bytes}

