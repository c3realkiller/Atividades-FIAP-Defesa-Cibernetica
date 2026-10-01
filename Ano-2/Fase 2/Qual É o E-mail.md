![[Pasted image 20260927125508.png]]
Baixado o arquivo para analise no autopsy.
![[Pasted image 20260927231921.png]]
Verificando com os comandos realizados abaixo:
```
 head -20 "/mnt/f/Downloads/page/page.ram"
t&fh
TCPAu2
r,fh
Invalid partition table
Error loading operating system
Missing operating system
B.5r{k4
NTFS
NTFSu
TCPAu$
fSfSfU
fY[ZfYfY
A disk read error occurred
BOOTMGR is missing
BOOTMGR is compressed
Press Ctrl+Alt+Del to restart
g:H
g:J@
f`gf
fPgf

┌──(volatility-env)(root㉿PC)-[~/forensics]
└─# wc -l "/mnt/f/Downloads/page/page.ram"
11114075 /mnt/f/Downloads/page/page.ram

┌──(volatility-env)(root㉿PC)-[~/forensics]
└─# grep -a -n -m 20 -E "Cristina|Sávio|Savio|@|Subject|From:|To:" "/mnt/f/Downloads/page/page.ram"
18:g:J@
64:fVf@fPfH
86:!@U!
87:!@U!
89:!@U!
90:!@U!
151:VWj@j
152:VWj@j
153:VWj@j
162:VWj@j
163:VWj@j
164:VWj@j
229:@.data
231:@INITDATA
237:`@V'
252:@wk4
294:orporatiQ@
302:`@V'
318:@W&1
344:orporatiQ@

┌──(volatility-env)(root㉿PC)-[~/forensics]
└─# grep -a -n -m 20 "BOOTMGR" "/mnt/f/Downloads/page/page.ram"
14:BOOTMGR is missing
15:BOOTMGR is compressed
166:BOOTMGR image is corrupt.  The system cannot boot.
167:BOOTMGR image is corrupt.  The system cannot boot.
173:BOOTMGR image is corrupt.  The system cannot boot.
2582963:BOOTMGR
3217862:BOOTMGR image is corrupt.  The system cannot boot.
3217863:BOOTMGR image is corrupt.  The system cannot boot.
3217869:BOOTMGR image is corrupt.  The system cannot boot.
3816379:BOOTMGR
6171058:BOOTMGR
7557907:BOOTMGR is missing
7557908:BOOTMGR is compressed
8020637:BOOTMGR
10252874:BOOTMGR is missing
10252875:BOOTMGR is compressed
10339628:BOOTMGR image is corrupt.  The system cannot boot.
10339629:BOOTMGR image is corrupt.  The system cannot boot.
10339635:BOOTMGR image is corrupt.  The system cannot boot.
10382204:BOOTMGR
```
Não temos um raw de memória e sim um arquivo de texto contendo a saida de strings de memória.
A evidência mais forte é esta:
```
head -20 page.ram
t&fh
TCPAu2
r,fh
Invalid partition table
...
NTFS
...
BOOTMGR is missing
```
E:
```
file page.ram
assembler source, ASCII text
```
Além disso, o arquivo possui **11.114.075 linhas**:
```
wc -l page.ram
11114075
```
Então **não adianta insistir no `windows.info` do Volatility 3 nesse arquivo**. O Volatility espera conseguir construir uma camada de memória a partir dos bytes da imagem; aqui os bytes originais já foram transformados em texto e separados por `\n`

Comando para procurar os emails:
```
grep -aEio '[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}' \
"/mnt/f/Downloads/page/page.ram" | sort -u
```
![[Pasted image 20260927233458.png]]
Onde salvo os resultados em um arquivo.
```
grep -aEio '[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}' \
"/mnt/f/Downloads/page/page.ram" | sort -u > emails.txt
```
Filtrado pela cristina:
![[Pasted image 20260927233652.png]]
Utilizado um script para ver se podíamos converter em UTF-16.
```
from pathlib import Path
import re

ARQUIVO = Path("/mnt/f/Downloads/page/page.ram")

termos = {
    "UTF-8": "Cristina".encode("utf-8"),
    "UTF-16LE": "Cristina".encode("utf-16le"),
    "UTF-16BE": "Cristina".encode("utf-16be"),
    "UTF-32LE": "Cristina".encode("utf-32le"),
    "UTF-32BE": "Cristina".encode("utf-32be"),
}

EMAIL = re.compile(
    rb"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}"
)

with open(ARQUIVO, "rb") as f:
    data = f.read()

print(f"Arquivo: {ARQUIVO}")
print(f"Tamanho: {len(data):,} bytes\n")

for nome, termo in termos.items():
    print(f"=== {nome} ===")

    inicio = 0
    encontrados = 0

    while True:
        pos = data.find(termo, inicio)

        if pos == -1:
            break

        encontrados += 1

        # contexto de 2 KB antes/depois
        inicio_contexto = max(0, pos - 2048)
        fim_contexto = min(len(data), pos + len(termo) + 2048)

        contexto = data[inicio_contexto:fim_contexto]

        print(f"\nCristina encontrada no offset: 0x{pos:X} ({pos})")

        emails = sorted(set(
            x.decode("ascii", errors="ignore")
            for x in EMAIL.findall(contexto)
        ))

        if emails:
            print("E-mails próximos:")
            for email in emails:
                print(f"  -> {email}")
        else:
            print("Nenhum e-mail ASCII encontrado no contexto.")

        inicio = pos + len(termo)

        if encontrados >= 20:
            print("\nLimite de 20 ocorrências atingido.")
            break

    print(f"Total encontrado: {encontrados}\n")
```
Onde tivemos o resultado:
```
 python3 cris.py
Arquivo: /mnt/f/Downloads/page/page.ram
Tamanho: 149,352,273 bytes

=== UTF-8 ===

Cristina encontrada no offset: 0x576A0B (5728779)
Nenhum e-mail ASCII encontrado no contexto.

Cristina encontrada no offset: 0x13BC579 (20694393)
Nenhum e-mail ASCII encontrado no contexto.

Cristina encontrada no offset: 0x467D88F (73914511)
Nenhum e-mail ASCII encontrado no contexto.

Cristina encontrada no offset: 0x533A608 (87270920)
Nenhum e-mail ASCII encontrado no contexto.

Cristina encontrada no offset: 0x5A02C05 (94383109)
Nenhum e-mail ASCII encontrado no contexto.

Cristina encontrada no offset: 0x8A50777 (145033079)
Nenhum e-mail ASCII encontrado no contexto.

Cristina encontrada no offset: 0x8D2F2C5 (148042437)
Nenhum e-mail ASCII encontrado no contexto.

Cristina encontrada no offset: 0x8DB1FCC (148578252)
Nenhum e-mail ASCII encontrado no contexto.

Cristina encontrada no offset: 0x8DB3D58 (148585816)
Nenhum e-mail ASCII encontrado no contexto.

Cristina encontrada no offset: 0x8E3DE5F (149151327)
Nenhum e-mail ASCII encontrado no contexto.
Total encontrado: 10
```
Neste caso Isso muda bastante a situação: **“Cristina” existe literalmente no arquivo em UTF-8**, então não precisamos insistir em UTF-16.

O problema agora é que os 2 KB ao redor de cada ocorrência não contêm um e-mail reconhecível. Isso pode acontecer porque o endereço está mais distante ou porque o conteúdo relevante está fragmentado.

Extraímos 8kb ao redor de cada ocorrência:
```
python3 - <<'PY'
from pathlib import Path

p = Path("/mnt/f/Downloads/page/page.ram")
data = p.read_bytes()

offsets = [
    0x576A0B,
    0x13BC579,
    0x467D88F,
    0x533A608,
    0x5A02C05,
    0x8A50777,
    0x8D2F2C5,
    0x8DB1FCC,
    0x8DB3D58,
    0x8E3DE5F,
]

for i, pos in enumerate(offsets, 1):
    start = max(0, pos - 4096)
    end = min(len(data), pos + 4096)

    out = data[start:end]

    filename = f"/tmp/cristina_{i}.bin"
    Path(filename).write_bytes(out)

    print(f"{i}: offset 0x{pos:X} -> {filename}")
PY
```
E tentado extrair as strings de forma legível.
```
for f in /tmp/cristina_*.bin; do
    echo
    echo "========== $f =========="
    strings -a -n 4 "$f"
done
```
Onde esse resultado, jogamos para um arquivo .txt
![[Pasted image 20260927235358.png]]
![[Pasted image 20260927235734.png]]
Temos:
email%5Bto%5D=crispsl83%40yahoo.com.br
Onde vemos que está em URL encoding
![[Pasted image 20260927235850.png]]
R: FIAP{crispsl83@yahoo.com}



