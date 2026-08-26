![[Pasted image 20260824102208.png]]
![[Pasted image 20260824102409.png]]
O arquivo tem muito base64 sendo requisitado por GET.
Irei concatenar para vermos se traz alguma resposta.
```
awk -F"'" '{print $2}' blacklog.txt | tr -d '\r\n ' | base64 -d > flag.jpg
```
E nos trouxe essa imagem:
![[flag.jpg]]
Aparenta ser um usuário do twitter.
![[Pasted image 20260824103910.png]]
![[Pasted image 20260824132820.png]]
