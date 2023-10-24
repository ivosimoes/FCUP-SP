# Format String Attack Lab

  Antes de começar, foi necessário criar containers com o docker-compose. Ficam aqui algumas imagens com o processo que seguimos:
  
    - Criar os containers:

   ![imagem](https://user-images.githubusercontent.com/126570489/231816039-c932668b-8bfd-41fd-b8ae-5b87409dd156.png)
   
    - Iniciar os containers:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/231816809-ce4751d6-99bc-4924-a965-ea509d560dfa.png)

    - Verificar os containers que estão on:

   ![imagem](https://user-images.githubusercontent.com/126570489/231817337-c16dd567-c9ee-4ba5-8526-d204db4e3cd3.png)

  Estando o setup inicial concluído, podemos começar a fazer as tarefas.

## Task 1: Crashing the Program

  Começamos com um teste inicial para ver se o servidor recebeu a nossa mensagem:

    - Teste:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/231822180-827aa5e2-5e30-434b-b083-ceca3c64303d.png)
   
   ![imagem](https://user-images.githubusercontent.com/126570489/231822279-0a2b785e-0473-4616-9252-24b5d2015bf7.png)

  O servidor recebeu a nossa mensagem e forneceu um output nos logs semelhante ao esperado. Vamos agora tentar crashar o programa que o servidor está a correr. Para tal, vamos tentar preencher o input com string formats ('%s') e ver o que acontece:
  
    - Payload:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/231832519-f8653bfe-3eeb-4560-ba0a-59743d12bf4e.png)

    - Resultado produzido pelo servidor:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/231833016-11c47b2e-4ca7-437d-8149-5bfa15657bc5.png)

   ![imagem](https://user-images.githubusercontent.com/126570489/231832707-c326b075-f80d-41ee-8823-86a852042890.png)

  Como se pode observar nas imagens, o payload foi obtido através de um simples comando em Python (python3 -c 'print (1500 * "%s")') que permite produzir 1500 formatadores string para um ficheiro 'payload.txt'. Depois de contactarmos o servidor e fornecermos a nossa payload, conseguimos ver que o output já não é o mesmo de antes; o programa que o servidor está a correr tenta imprimir o input, mas não o consegue fazer e acaba por crashar. Porquê?

  Quando usamos printf em C e queremos imprimir o valor de uma variável com formato string, o programa fica à espera de uma referência a um array de carateres que constitui a string. O problema é que, se os formatadores string não forem especificados no printf (e.g. em vez de colocar printf("%s", argv[1]), colocar printf(argv[1])) e o programa receber esses formatadores no input, o programa vai tentar buscar essa string ao buffer e, eventualmente, a sítios de memória fora do seu alcance, resultando num crash.

## Task 2: Printing Out the Server Program’s Memory

### Task 2.A: Stack Data

  Para esta parte da tarefa 2, vamos tentar descobrir quantos hex formats (%x) são necessários para imprimir os primeiros 4 bytes do nosso input. Como exemplo de teste vamos usar "AAAA" que se traduz para 0x41414141 (codificação UTF-8). Tentemos com uma quantidade inicial de 200 formatadores:

    - Payload:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/231844626-297b5443-3fa5-4e31-ad7e-fe9f005ef06e.png)

   ![imagem](https://user-images.githubusercontent.com/126570489/231844758-6e3d52c4-3af9-42bb-a9a7-161b74df3b64.png)

    - Logs do servidor:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/231845539-8b1df76e-168c-4ba4-8384-e4549f6f0f14.png)

  Através do resultado obtido, conseguimos ver que os 4 bytes de input foram imprimidos (em hexadecimal) por um dos formatadores. Poderíamos contar quantos formatadores foram usados até imprimir o valor que queríamos, mas, para ter certeza, vamos usar tentativa e erro até descobrir quantos foram usados ao certo:
  
    - Payload corrigido:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/231847639-5fe59e24-d47d-4671-ac85-3db281419428.png)
   
    - Logs do novo payload: 

   ![imagem](https://user-images.githubusercontent.com/126570489/231847885-9d0e7481-6ade-4f6e-9fbb-bb45ea970729.png)
   
  Recorrendo às imagens acima, concluimos que 64 hex formats é a quantidade necessária para imprimir a string de input.

### Task 2.B: Heap Data

  Com o conhecimento que adquirimos no ponto anterior, vamos tentar aceder à mensagem secreta. Das diversas vezes que testamos input e olhamos para os logs do server, notamos que, de cada vez que o programa é executado, a localização da mensagem secreta é escrita: 0x080b4008. Este detalhe (destacado a amarelo) pode ser observado nesta imagem:
  
    - Localização da mensagem secreta:

   ![imagem](https://user-images.githubusercontent.com/126570489/232050912-8415ae62-51dc-4237-8806-3e1f9a165f30.png)

  Para aceder ao endereço da mensagem, vamos usar uma técnica que aprendemos antes: escrever os 4 bytes do endereço no início do buffer e, usando um formatador string, ler esse endereço para ir buscar a mensagem secreta. Vamos mostrar o método que usamos, seguido de uma explicação:

    - Payload:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/232052595-3a57e336-1996-4692-bc72-0381b9e40ffb.png)

    - Logs do servidor:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/232052970-04110c50-0520-4eb2-bb57-45b606402239.png)

  Como dá para ver nas imagens acima, o nosso método para criar a payload difere um pouco do que foi indicado no guião. Eis uma explicação passo a passo: 
  
  1. O motivo de usar python2 em vez de python3 tem a ver com a codificação de carateres usada pelas 2 versões da linguagem: enquanto que o python2 usa a codificação UTF-8 para chars, o python3 utiliza uma codificação diferente. Como a linguagem C segue a codificação UTF-8, optamos por usar o python2. 
  
  2. Quanto ao endereço que foi colocado no início do buffer, este é representado pela string "\x08\x40\x0b\x08" que representa a codificação em hexadecimal do endereço 0x080b4008 em little-endian. 
  
  3. Relativamente à quantidade de formatadores usados, descobrimos no ponto anterior que o programa tenta aceder ao conteúdo do buffer ao 64º formatador. Com esta informação em mente, colocámos 63 formatadores hex (63 * " %x") a preceder um formatador string (" %s") de modo ao programa não crashar (se todos os 64 fossem de string, o programa estaria a tentar aceder a zonas de memória inválidas).
  
  Seguindo estes passos, foi possível extrair a mensagem secreta destacada a amarelo na segunda imagem.
  
  (Nota: a técnica usada com python2 é a mesma que usámos para o desafio 2 da semana 5 que aprendemos neste vídeo https://www.youtube.com/watch?v=5-ZQubBWz3c)

## Task 3: Modifying the Server Program’s Memory

### Task 3.A: Change the value to a different value

  Nesta tarefa é nos pedido para alterar o valor da varável target (0x11223344) que reside em (0x080e5068). Visto que até agora só recorremos a formatadores que permitem imprimir valores, vamos ter de recorrer a outros. Aqui entra o formatador "%n". Este formatador permite introduzir o número de carateres que o programa imprimiu até ao momento em que este formatador foi usado. Para exemplificar, deixamos aqui as imagens que mostram a aplicação do "%n" neste problema:

    - Payload:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/232078090-d958412a-75bd-4bb6-a6ab-57d0f5b9a46d.png)

    - Logs do servidor:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/232080436-01c2f955-3bc6-4506-acc9-36c582e8fd2e.png)

  O payload usado aqui é quase idêntico ao que foi usado em 2.B, a diferença sendo o uso de "%n" em vez de "%s" para modificar o valor de target em vez de imprimir. A grande diferença reside nos logs do servidor e o facto que o valor de target foi alterado para 0x0000012e. Porquê? 
  
  O método do ataque já foi definido e explicado em 2.B. Quanto ao valor de target, a explicação meio que também já foi dada. Como referimos anteriormente, o formatador "%n" modifica um valor de memória para o número de carateres que o programa imprimiu até aquele momento. Se contarmos a quantidade de carateres que o programa imprimiu nos logs do server, veremos que o seu número chega a 302, que se traduz para o hexadecimal 0x0000012e (valor do target).

### Task 3.B: Change the value to 0x5000

  Nesta parte queremos fazer o mesmo que fizemos antes, mas, em vez de definir um valor aleatório em target, queremos colocar o valor 0x0005000 (correspondente a 20480). Logicamente, para colocar o valor pedido em target, poderíamos preencher o espaço de input com 20480 carateres para conseguir fazer a mudança. O problema é que não dispomos de 20480 bytes de input; só conseguimos inserir 1500. Então, como é que chegamos ao valor pretendido?
  
  Existe uma funcionalidade que nos pode ajudar. Se colocarmos o valor que queremos no meio de um formatador hex (e.g. %<valor>x), o programa vai encher o output com a quantidade de espaços correspondentes a esse valor, sem modificar a quantidade de memória de input. Como o "%n" conta os espaços individualmente, só precisamos de calcular quantos espaços necessitamos para chegar ao valor 0x00005000, que é, aproximadamente 20178 (0x00005000 - 0x0000012e). 

  Deixamos aqui imagens do nosso método de resolução:

    - Payload:

   ![imagem](https://user-images.githubusercontent.com/126570489/232092787-df9ad4a7-34ce-4d19-9de4-2b92e41e486c.png)
  
    - Logs do servidor:
  
   ![imagem](https://user-images.githubusercontent.com/126570489/232093364-59b29b0a-66a3-475a-93bc-4f761e3f3f8b.png)
   ![imagem](https://user-images.githubusercontent.com/126570489/232093550-22b86bea-9d2e-4e15-8992-ed09b28d8a7d.png)

  Em relação à tarefa anterior, só movemos um dos 63 "%x" de sítio para poder modificar o valor especificado acima. É de notar que a quantidade de espaços gerada não coincide com a estimativa inicial, tendo existido necessidade de a modificar até o valor de target coincidir com o desejado (apesar da estimativa ter ficado muito próxima do valor real).
  
  Quanto aos logs do servidor, seria um bocado complicado estar a colocar o log completo que o programa produziu já que existem cerca de 20000 espaços pelo meio, tendo só sido colocados o início e o fim do log.
  
  (Nota: A técnica de espaços utilizada foi baseada no tutorial presente em https://axcheron.github.io/exploit-101-format-strings/)
