![[Pasted image 20260930134541.png]]
os `GET` com status **200** carregam blocos Base64. O primeiro bloco começa com `/9j/4AA...`, que é a codificação Base64 típica do início de um JPEG, e os blocos seguintes continuam a sequência.
![[Pasted image 20260930224257.png]]
Utilizado um script para juntarmos os blocos de requisição 200 para ver oq nos traz.
```
import base64
import re

ARQUIVO = "access.txt"
SAIDA = "recuperado.jpg"

padrao = re.compile(r'"GET ([^"]+) 200"$')

dados = bytearray()
total = 0

with open(ARQUIVO, "r", errors="ignore") as f:
    for linha in f:
        m = padrao.search(linha)
        if not m:
            continue

        chunk = m.group(1)

        # O campo da URL é o próprio Base64.
        # Mantemos a barra inicial do primeiro chunk (/9j/...).
        chunk += "=" * (-len(chunk) % 4)

        try:
            dados.extend(base64.b64decode(chunk, validate=True))
            total += 1
        except Exception as e:
            print(f"[!] Chunk inválido: {chunk[:30]}... -> {e}")

with open(SAIDA, "wb") as f:
    f.write(dados)

print(f"[+] Chunks reconstruídos: {total}")
print(f"[+] Tamanho: {len(dados)} bytes")
print(f"[+] Saída: {SAIDA}")
```

![[Pasted image 20260930224513.png]]
Analisando com o stegsolve, encontramos uma informação:

![[Pasted image 20260930225253.png]]
R: FIAP{ANTI_FORENSE}