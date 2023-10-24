# Number Station 3

  Para começar, é nos fornecido um script Python no enunciado do CTF. Adicionalmente, se executarmos este script (se contactarmos o serviço), é produzida uma mensagem que parece estar encriptada. Vamos analisar o ficheiro:
  
```python
  # Python Module ciphersuite
import os
import sys
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from binascii import hexlify, unhexlify

FLAG_FILE = '/flags/flag.txt'

# Use crypto random generation to get a key with length n
def gen(): 
	rkey = bytearray(os.urandom(16))
	for i in range(16): rkey[i] = rkey[i] & 1
	return bytes(rkey)

# Bitwise XOR operation.
def enc(k, m):
	cipher = Cipher(algorithms.AES(k), modes.ECB())
	encryptor = cipher.encryptor()
	cph = b""
	for ch in m:
		cph += encryptor.update((ch*16).encode())
	cph += encryptor.finalize()
	return cph

# Reverse operation
def dec(k, c):
	assert len(c) % 16 == 0
	cipher = Cipher(algorithms.AES(k), modes.ECB())
	decryptor = cipher.decryptor()
	blocks = len(c)//16
	msg = b""
	for i in range(0,(blocks)):
		msg+=decryptor.update(c[i*16:(i+1)*16])
		msg=msg[:-15]
	msg += decryptor.finalize()
	return msg

with open(FLAG_FILE, 'r') as fd:
	un_flag = fd.read()

k = gen()
print(hexlify(enc(k, un_flag)).decode())
sys.stdout.flush()
```

  Do que nós entendemos, o programa começa por extrair a flag de "flag.txt" e cria uma variável k que contém uma chave AES. A flag é depois encriptada usando k e imprimida no stdout. Porém, existe um problema com a maneira como o k é criado:
  
  ```python
  def gen(): 
    rkey = bytearray(os.urandom(16))
    for i in range(16): rkey[i] = rkey[i] & 1
    return bytes(rkey)
  ```
  
  Esta função gera um array aleatório de 16 bytes, ou seja, um array de 128 bits. Num caso normal, seria extremamente difícil adivinhar uma chave de 128 bits aleatórios (2^128 = 340282366920938463463374607431768211456 combinações), mas, neste programa, cada byte está mapeado ao seu primeiro bit (se este for 1, 0x01, caso contrário, 0x00), reduzindo substancialmente o número de combinações para 2^16 = 65536. Eis a demonstração:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/dd5ddd9c-18de-4b92-a07e-52a2de8cb219)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/7228e2b9-1023-4ff7-9837-679458f7aaa1)

  Se a função estivesse corretamente implementada, os bytes variavam entre 0x00 e 0xff como na primeira imagem. No programa fornecido, a função cria um array de bytes cujos valores destes variam apenas entre 0x00 e 0x01. Tendo isto em conta, uma maneira de decifrar a chave seria através de brute-force, testando para todas as 65536 combinações possíveis.
  
  Um pequeno script capaz de resolver o nosso problema é o seguinte:
  
  ```python
  # Python Module ciphersuite
  import os
  import sys
  from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
  from binascii import hexlify, unhexlify

  ENC_FLAG = "ENCRYPTED FLAG"

  # Reverse operation
  def dec(k, c):
    assert len(c) % 16 == 0
    cipher = Cipher(algorithms.AES(k), modes.ECB())
    decryptor = cipher.decryptor()
    blocks = len(c)//16
    msg = b""
    for i in range(0,(blocks)):
      msg+=decryptor.update(c[i*16:(i+1)*16])
      msg=msg[:-15]
    msg += decryptor.finalize()
    return msg

  masks = [1 << i for i in range(16)]
  flag_bytes = b'flag'

  for possible_key in range(1 << 16):
      key = bytes((1 if possible_key & mask else 0) for mask in masks)
      try:
          dec_bytes = dec(key, unhexlify(ENC_FLAG))
          if flag_bytes in dec_bytes:
              print(dec_bytes.decode())
      except Exception:
          pass
  ```
  
  Com uma pequena ajuda, criámos um script que percorre todas as combinações possíveis e verifica se os tais primeiros bits são "verdadeiros" ou "falsos". Caso o valor de verdade acerte em todos, significa que encontrámos a nossa flag. Caso contrário, continuamos a procurar. Vamos ver se o script resulta:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/d5d1640a-94ea-46ee-82d5-ed509305b048)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/37871c81-214b-4db5-ac56-6eec7ddc9b5f)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/c3a9e8e1-5b57-4146-b098-4c37bef98ded)

  Atenção: o script não contacta diretamente o servidor. É necessário introduzir manualmente a flag encriptada no programa tal como mostra a segunda imagem e executar o programa para obter a flag.
