# Final Format

  Neste desafio, somos apresentados com um executável de C e, pelo que deduzimos do enunciado, temos que arranjar alguma maneira de invocar uma shell para aceder à flag que se encontra em "flag.txt". Devido a não termos acesso ao código fonte, temos de trabalhar com o código assembly presente no executável. Começemos então por olhar para o código através do comando "objdump -d program":
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/cbb713fd-23b9-4c6d-86a3-2fe92f7e29d9)

  Porém, este código não é muito legível e, como tal, não percebemos ao certo o que o programa está a fazer. Para entender melhor o código temos duas alternativas: traduzir o código assembly à mão para C ou recorrer a uma ferramenta externa que o faça por nós. Como a primeira alternativa não é muito viável, pesquisámos um bocado sobre software que tenha capacidade de traduzir assembly para C e, convenientemente, encontrámos um site que disponibiliza diversas técnicas de tradução:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/67e7e106-59ef-467b-be35-79bad167f880)
  (Fonte: https://dogbolt.org/?id=ac876def-bfb1-4f33-a936-730b160db15e)
  
  Decidimos explorar os diferentes meios de tradução que o site tem e chegámos à conclusão que o mais intuitivo é o Hex-Rays:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/10073d9d-42e4-4111-8144-418c1c0c2ae0)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/28faea0f-13d7-45b0-adc3-3219b92f6be8)

  Com a ajuda da "descompilação", conseguimos entender o que o programa faz, detetámos a vulnerabilidade (printf(format);) e descobrimos uma função interessante (old_backdoor) que parece invocar a shell que queremos. Com estas novas informações, vamos começar a pensar em como explorar esta vulnerabilidade. Uma das primeiras coisas a fazer é saber as limitações impostas (i.e. que medidas de segurança estão ativas no programa):
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/7f552666-8e50-427e-8939-a0c234f02553)

  Ao que parece, temos canários ativos, dificultando a hipótese de reescrever o return e NX também, impossibilitando a injeção de shellcode. Então, qual é a ideia? Um possível procedimento dadas as funções existentes e as medidas ativas seria chamar a função "old_backdoor" para invocar uma shell, mas, como esta função nunca é chamada pelo programa, temos de criar uma chamada à função. Para tal, podemos usar uma técnica que ainda não usámos antes: Return-to-libc.
  
  O plano então é o seguinte: vamos tentar controlar o flow do programa, mudando o endereço de uma função que o programa use para a função "old_backdoor". Isto pode ser feito com a função fflush() devido a ser a única que aparece depois do input, antes do return e não compromete o ataque. Vamos dividir o ataque em 3 partes: obter o endereço de fflush(), obter o endereço de "old_backdoor" e construir a payload.
  
## 1. Endereço de fflush()

  Para obter o endereço desta função, só precisamos que usar o comando inicial ("objdump -d program") com a opção -R, que nos mostra os Dynamic Relocation Records:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/8a1295ef-80bd-4c5f-8d6d-3cbbb4d8dc8d)

  Descobrimos que o endereço de fflush é 0x0804c010.
  
## 2. Endereço de "old_backdoor"

  Usando o comando inicial outra vez, podemos obter o endereço desta função:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/fc65fc43-aca7-4d8f-a3b4-58df2954ead8)

  Neste caso, o endereço de "old_backdoor" é 0x08049236.
  
## 3. Construir a payload

  Esta parte do desafio foi baseada numa implementação exemplo presente na documentação da biblioteca pwn. Podemos dividir este processo em 2 partes: descobrir o offset entre funções e construir a payload com base no offset.
  
### 3.1 Descobrir o offset

  Antes de fazer alterações na memória, temos que descobrir o offset entre as funções. Recorrendo à implementação mencionada, podemos construir um script que calcula o que queremos:

  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/2fc5cb89-4b26-4658-90f5-efe1c9728df3)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/b84e6208-649d-4af0-8a1f-511c589019bb)
  (Nota: O offset foi calculado na VM porque a host machine (W11) parece não reconhecer o executável)

  Através do script anterior, descobrimos que o valor do offset é 1.
  
### 3.2 Payload principal

  Mais uma vez, tendo como base a implementação da documentação pwn, podemos construir uma payload que nos auxilía no ataque. A única diferença é o endereço das funções que temos que modificar: enquanto que a documentação só mostra 1 endereço, nós temos que meter os 2 juntos com o offset:
  
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/74a1d7d0-5804-4237-8026-2c1f1cc6d2cd)
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/aabd456c-82de-4a5c-b398-c61c295884aa)

  É de notar que o offset é constante (=1), daí não haver necessidade de estar a calcular em todas as execuções do script. Com este novo script, devemos conseguir ter acesso à flag:
 
  ![imagem](https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/126570489/fd73f97c-fa39-4653-83c5-95ae679546ed)

  Sucesso!
