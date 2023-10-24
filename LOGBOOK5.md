# Buffer Overflow Attack Lab (Set-UID Version)

## Task 1: Getting Familiar with Shellcode

  A primeira tarefa tem como objetivo familiarizar-nos com o Shellcode. Para isso, é nos fornecido um ficheiro "call_shellcode.c" que, de acordo com o descrito no enunciado da tarefa, tem como objetivo invocar uma shell quando compilado e executado sob condições específicas. Abaixo fica uma imagem do output do programa, segundo as instruções fornecidas:

    - Output: 
    
   ![imagem](https://user-images.githubusercontent.com/126570489/227331125-5f989ce0-86cc-451a-a3c0-2fe10712b1cd.png)

  Após a execução do programa, podemos ver o que parece ser uma shell; o programa fica à espera de input e cada linha é antecedida do símbolo '$' indicando a possibilidade de estarmos perante a prompt de um terminal shell. Vamos testar uns comandos básicos da shell do linux para testar a nossa hipótese:

    - Uso de comandos shell (32 bits): 
    
   ![imagem](https://user-images.githubusercontent.com/126570489/227333995-bf60d779-32d6-4c01-81bf-20fab3547422.png)
    
  A imagem anterior comprova a nossa hipótese: estamos de facto a lidar com um terminal shell. Vamos também testar o executável de 64 bits também gerado pelo "make":

    - Uso de comandos shell (64 bits): 
    
   ![imagem](https://user-images.githubusercontent.com/126570489/227334880-d48e2df8-6c8b-4b0c-84c8-607d82bba698.png)

  Como esperado, este executável produz um output semelhante.
  Podemos concluir que o objetivo inicial (injetar Shellcode através de um programa em C) foi atingido com sucesso. A seguir deixamos a nossa perspetiva sobre o que aconteceu.
  O programa em C que usamos tem 2 instâncias de Shellcode: uma de 32 bits e outra de 64 bits. Apesar do Shellcode presente no programa não ter sido obtido por nós, o mesmo é proveniente de 2 funções em assembly presentes no enunciado. No código C, a shell é invocada através da execução do Shellcode recorrendo ao apontador de função "func" que aponta para a variável "code" (que contém o Shellcode) como se fosse uma função, passando posteriormente a existir acesso à shell quando "func()" é chamada.

## Task 2: Understanding the Vulnerable Program

  Nesta tarefa, o objetivo é entender como um buffer overflow funciona e potenciais usos que podemos dar a este exploit. É nos então fornecido um programa "stack.c" bem como a explicação do que faz: recebe uma string de 517 carateres (517 bytes) e passa essa string a uma função bof. A função bof contém um buffer de 100 bytes que copia os primeiros 100 carateres da string de input.
  O buffer overflow do programa situa-se no buffer da função bof: o buffer copia 100 carateres, mas como a string inicial pode conter mais de 100, o buffer dá overwrite ao return da função bof para a função main, podendo retornar para sítios indesejados (caso do shellcode).
  É também importante notar que L1 e L2 foram compilados em 32 bits e L3 e L4 foram compilados em 64 bits. Abaixo fica uma imagem da compilação do programa:

    - Compilação do programa: 
    
  ![imagem](https://user-images.githubusercontent.com/126570489/228836428-95b7a746-47f8-4e17-883e-719be104ecc5.png)
    
  Após compilar "stack.c" usando "make", ficamos com 4 executáveis diferentes dentro da pasta, cada um com tamanhos de buffers diferentes. Se abrirmos o "Makefile" (ficheiro usado para compilar "stack.c" com os diferentes buffers), descobrimos que L1 = 100, L2 = 160, L3 = 200 e L4 = 10. Os mesmos valores podem ser observados durante o processo de compilação, olhando para o valor de "DBUF_SIZE". Deixamos aqui uma imagem que mostra os valores de L em "Makefile":
  
    - Tamanhos dos buffers (valores das variáveis): 
    
  ![imagem](https://user-images.githubusercontent.com/126570489/228849121-1b70f170-7c69-4d88-8a78-b93abbd19a0f.png)

  Com estas informações, vamos tentar lançar um ataque através dos novos executáveis na tarefa seguinte.

## Task 3: Launching Attack on 32-bit Program (Level 1)

  Vamos agora preparar o ataque ao sistema. Recorrendo ao gdb (debugger) vamos descobrir as localizações do buffer e do return address da função bof, assim como a sua distância. Para tal, vamos correr o programa no debugger, adicionando um breakpoint para descobrir os parâmetros descritos anteriormente. Segue-se uma série de prints do processo conretizado:
  
    - Criação do badfile e iniciação do gdb: 
    
  ![imagem](https://user-images.githubusercontent.com/126570489/228885823-222338d8-9a74-4bb7-aa24-235b471bae37.png)
  
    - Criação de um breakpoint em bof: 
    
  ![imagem](https://user-images.githubusercontent.com/126570489/228886050-cb4e6245-e7c2-4394-abb5-b10d5b193b10.png)
  
    - Correr o programa em gdb: 
    
  ![imagem](https://user-images.githubusercontent.com/126570489/228988493-e5e2d4eb-71a2-4e07-9833-b1f29cfd42b2.png)
  
    - Fim do output anterior e um bocado do output do next: 
    
  ![imagem](https://user-images.githubusercontent.com/126570489/228988361-0c2d169c-02f3-406b-945f-f570721318a1.png)
  
    - Fim do output anterior e valor do endereço do buffer: 
    
  ![imagem](https://user-images.githubusercontent.com/126570489/228988060-b98e1653-bd2c-4ef0-8ba8-4d97958281f1.png)
  
    - Valor de ebp: 
    
  ![imagem](https://user-images.githubusercontent.com/126570489/228988197-a9582392-2781-4b70-94c7-5c61100404b9.png)

  O processo anterior permite-nos obter os valores do endereço do buffer da função bof e do frame pointer. Com estes valores, podemos obter de imediato o valor do offset, que corresponde à diferença $ebp-&buffer+4. Porquê esta diferença?
  O return address de bof situa-se após o buffer e o frame pointer na stack. Quando executamos os comandos anteriores, descobrimos precisamente onde se localizavam o buffer e o frame pointer. Como o frame pointer ocupa 4 bytes (programa de 32 bits) de memória e o return address situa-se imediatamente a seguir ao frame pointer, o offset (distância do início do buffer ao return address) pode ser dado pela fórmula acima. 
    
  A seguir arranjamos o shellcode e um sítio para metê-lo. Nesta ficha já nos tinha sido fornecido um shellcode para programas de 32 bits, por isso fomos copiá-lo aí. Quanto a um sítio para metê-lo, decidimos que o fim de str (string de argumento) era um bom sítio para colocar por motivos que veremos a seguir.
  
  Por fim, precisamos de definir o return address. Como nos tinha sido avisado na nota 2, a execução de um programa em ambient gdb e em ambiente normal difere um bocado, havendo ligeiras diferenças nas variáveis guardadas e no frame pointer. Para aumentar a probabilidade de acerto no shellcode, este foi colocado no fim da string e como até lá a string está preenchida de NOPs, é só colocarmos o return address para um endereço que contenha NOP e, eventualmente, após passar por todos os NOPs, o stack pointer chegará ao shellcode, invoncando uma shell. No nosso caso, escolhemos saltar 220 endereços, de modo a não saltar o shellcode e a ser o suficiente para atender ao aviso referido.

  Ficam aqui algumas imagens a mostrar o terminal e o script python editados:
  
    - Estado final do ficheiro python: 
    
  ![imagem](https://user-images.githubusercontent.com/126570489/228987035-33bf5a3b-ac2b-4b0a-a69d-3293d3dfef82.png)
  
    - Ataque realizado com sucesso: 
    
  ![imagem](https://user-images.githubusercontent.com/126570489/228987373-00f2eb8a-abf5-4112-9a09-b28a11f70878.png)
