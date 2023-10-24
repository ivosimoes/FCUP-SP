**Environment Variable and Set-UID Program Lab**

  2.1 Task 1: Manipulating Environment Variables
  
    A primeira tarefa pede-nos para imprimir as variáveis de ambiente recorrendo ao comando "env" ou "printenv" da shell:
    
      - Comando "env": 
      
   ![imagem](https://user-images.githubusercontent.com/126570489/225076724-cd25bc63-bf81-4861-b956-9f3d2d8efb38.png)
   
      - Comando "printenv": 
      
   ![imagem](https://user-images.githubusercontent.com/126570489/225095481-5c978fee-c17f-4a7c-81d3-c2a3244c229f.png)

    Podemos recorrer ao método anterior para, por exemplo, saber o valor de PWD (equivalente ao comando "pwd" da shell):

   ![imagem](https://user-images.githubusercontent.com/126570489/225096076-5d1e43cc-64e6-455d-a0b6-3996754ec1e4.png)

    Os métodos acima podem ser usados para passar o valor de variáveis de ambiente a programas que necessitem delas.
    A seguir é nos pedido para usar os comandos "export" e "unset" da shell para manipular os valores das variáveis de ambiente. Seguem-se dois exemplos:
    (Nota: nas imagens recorre-se ao programa "myprintenv.c" para visualizar o valor de HOME)

      - Comando "export": 
      
   ![imagem](https://user-images.githubusercontent.com/126570489/225083771-5ee569ad-deb5-49a1-967e-d328fb54ebe1.png)
   
      - Comando "unset": 
      
   ![imagem](https://user-images.githubusercontent.com/126570489/225084600-d007586e-4662-45bd-9c3c-c2f4f53743ee.png)
      
    No primeiro exemplo, conseguimos observar o comportamento do comando "export". Inicialmente, o variável HOME (que guarda o valor de "~") tem o valor /home/seed. Se executarmos o comando "cd ~" somos dirigidos para o diretório especificado pela variável. Ao alterar o diretório destino da variável através de "export" e se executarmos o comando "cd ~" outra vez, desta vez somos realocados para o novo valor de HOME.
    No segundo exemplo, conseguimos observar um comportamento quase oposto ao do comando anterior. Desta vez, através do comando "unset", removemos o valor de HOME, deixando este de existir. Se tentarmos correr "cd ~" veremos que somos dirigidos para o diretório default (valor default da variável antes desta ser apagada, que neste caso é /home/seed).
  
  
  2.2 Task 2: Passing Environment Variables from Parent Process to Child Process
  
    Começamos a segunda tarefa por compilar e executar um dos programas contidos no ficheiro "Labsetup.zip" chamado "myprintenv.c". O nome do programa já nos dá uma ideia do que o mesmo faz e após analisar o seu output, confirma-se que é uma versão do printenv inicialmente descrito na tarefa 2.1. Eis um excerto do output do programa:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/225103969-6135a4f9-f8e7-4b2a-97f4-af916a76b5f6.png)
    
    O segundo passo pede-nos para alterar o código do "myprintenv.c", mais especificamente para comentar o "printenv()" do processo filho e tirar o comentário do "printenv()" do processo parente. Após voltar a executar o programa com as devidas alterações feitas, o output não parece ter mudado. Aqui fica um excerto do novo programa:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/225102999-0e083e8c-79bc-4792-bd43-9c0dc3e6374b.png)
    
    Se recorremos a uma ferramenta da shell para comparar ficheiros ("diff"), veremos que, de facto, o output de ambos os programas é exatamente igual. Qual o motivo dos outputs serem iguais? 
    Uma resposta à pergunta é o uso da função "fork()". A função "fork()" divide o processo inicial em dois processos, sendo o segundo uma réplica do primeiro. Para fazer a réplica do processo inicial, o sistema operativo copia as variáveis de ambiente de um processo e aplica-as no outro, passando os dois a ter variáveis de ambiente iguais. 
    Segundo o manual do "fork()", o novo processo contém o mesmo conteúdo em memória do processo original, apesar dos dois ocuparem sítios diferentes da memória, explicando-se assim o facto das variáveis de ambiente terem os mesmos valores. Seguem-se umas imagens para comprovar as afirmações:

      - Diferença entre outputs: 
      
   ![imagem](https://user-images.githubusercontent.com/126570489/225107713-e1a76790-98d8-41f9-b0a8-37f1267173aa.png)
   
      - Excerto do manual do "fork()": 
      
   ![imagem](https://user-images.githubusercontent.com/126570489/225108097-e4497ab1-fe63-4158-89a5-4b099ed35264.png)
  
  
  2.3 Task 3: Environment Variables and execve()
  
    Nesta tarefa é nos pedido para compilar outro programa do "Labsetup.zip": o "myenv.c". Este programa não é tão intuitivo como o anterior, já que quando é compilado e executado, não parece produzir resultado algum. Fica aqui uma imagem da compilação e output do programa:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/225111803-bcdad635-a9af-4a13-9414-acdd6caf0c33.png)
      
    Já no segundo passo, não acontece o mesmo. É nos agora pedido para alterar um dos argumentos da função "execve()", neste caso passar o NULL a environ. Eis um excerto do output produzido:

   ![imagem](https://user-images.githubusercontent.com/126570489/225112876-370add46-37b1-4994-bf8a-a3e1c8d136ad.png)

    Este último output já nos é familiar; são as variáveis ambiente do programa. Mas porque é que só foram imprimidas agora e não antes da mudança?
    A explicação para este fenómeno reside na chamada da função "execve()". A função "execve()" realiza uma substituição do programa que está a ser executado, substituindo-o por um novo programa com uma nova stack, heap e data segment. A função tem, por isso, como argumentos o nome do ficheiro executável, a string de argumentos do programa inicial e as variáveis de ambiente a serem passadas ao novo programa.
    No nosso caso em específico, a diferença reside nas variáveis de ambiente passadas em ambos os casos. No primeiro caso, o novo programa não recebe nenhuma variável ambiente do programa anterior (devido ao argumento NULL), fazendo com que o output não contenha variáveis de ambiente (daí o output vazio). Já no segundo caso são passadas todas as variáveis ambiente do programa anterior para o novo programa, sendo estas agora visíveis no output.
    Podemos até certo ponto comparar as funções "execve()" e "fork()". Enquanto que a função "fork()" divide o programa inicial em dois (este e a sua réplica), o "execve()" abandona o programa inicial para dar lugar a um novo programa que pode ser considerado uma réplica (dependendo dos argumentos fornecidos à função). Apesar das duas funções desempenharem papeis diferentes, as variáveis ambiente criadas pelo novo programa são iguais às do programa original (dependendo dos argumentos fornecidos como vimos).
    A informação obtida para esta resposta veio do manual do "execve()", da qual deixamos aqui uma imagem:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/225117944-3fb5e5ad-4474-495b-8a27-86bc4a7945dd.png)


  2.4 Task 4: Environment Variables and system()
  
    Para esta tarefa temos de compilar o código fornecido no ficheiro PDF fornecido para a aula e verificar que a função system() executa /bin/sh/ e pede á shell para executar o comando. Essa informação aparece nas seguintes imagens (devido ao tamanho do output foram necessários várias):
    
   ![VirtualBox_SP_14_03_2023_23_19_38](https://user-images.githubusercontent.com/125911788/225179480-15aef2a1-9c60-495f-832b-fa67947418f9.png)

   ![VirtualBox_SP_14_03_2023_23_20_25](https://user-images.githubusercontent.com/125911788/225179501-e60af20c-e8b2-44ed-921e-829d1e31eb91.png)
    
   ![VirtualBox_SP_14_03_2023_23_21_18](https://user-images.githubusercontent.com/125911788/225179610-acff4b6a-fde2-4baf-9ee8-9de9221e2c49.png)

   ![VirtualBox_SP_14_03_2023_23_22_08](https://user-images.githubusercontent.com/125911788/225179644-344fb5b4-61fa-4ebf-9df1-c8e1f134df9c.png)
    
   ![VirtualBox_SP_14_03_2023_23_22_23](https://user-images.githubusercontent.com/125911788/225179677-b00d55d2-0910-4fa1-8b26-cc0183353c21.png)
    
    Tal como é dito na ficha, a função system() utiliza execl() para executar /bin/sh, para onde serão passadas as variáveis do ambiente de processo de chamada. Isso pode ser verificado no manual da função system(), que está apresentado na imagem:
    
   ![VirtualBox_SP_14_03_2023_23_33_07](https://user-images.githubusercontent.com/125911788/225180254-45f182bc-9c99-4957-8e51-2e199dac7b01.png)


  2.5 Task 5: Environment Variable and Set-UID Programs
  
    Nesta tarefa seguimos 3 passos: guardamos o código dado no ficheiro PDF fornecido no ambiente, compilamos, atribuímos o ficheiro a root, tornando-o um ficheiro Set-UID. Na shell foi usado o comando export definir as variáveis PATH, LD_LIBRARY_PATH e SPFICHA (o nome escolhido para a variável). Tudo o que foi feito no terminal é apresentado nas seguintes imagens:
    
   ![VirtualBox_SP_1](https://user-images.githubusercontent.com/125911788/226604394-3a76c5a9-6950-406b-9133-bfad9bb909ac.png)
   
   <img width="708" alt="image" src="https://user-images.githubusercontent.com/125911788/226602099-7e21c643-21c2-4af0-8cc5-6f455d7a524b.png">
    
   <img width="740" alt="image" src="https://user-images.githubusercontent.com/125911788/226602607-948e1edc-c611-442e-a247-9e3329176b21.png">

   <img width="890" alt="image" src="https://user-images.githubusercontent.com/125911788/226602833-25bdff69-9717-400d-867d-88b45935b841.png">  

   <img width="892" alt="image" src="https://user-images.githubusercontent.com/125911788/226603098-b8853fc3-5a2b-48cf-b74d-4eafa551315c.png">
    
    Das 3 variáveis que definimos com o comando export apenas a variável PATH e SPFICHA são encontrados no resultado (na penúltima imagem), algo que foi motivo de surpresa. LD_LIBRARY_PATH não apareceu no resultado. Recorrendo ao manual da função environ (usada em foo.c), concluímos que LD_LIBRARY_PATH é ignorada por motivos de segurança sendo uma variável que pode ser usada para alterar valores do dynamic loader/linker redirecionando para bibliotecas potencialmente maliciosas do utilizador. Para prevenir o problema descrito a variável é ignorada.
    Apesar de não ter sido alterado no terminal quando o programa foi testado, segundo o manual da função environ, uma mudança na variável PATH pode tornar o sistema mais lento uma vez que esta guarda diretórios a serem procurados em caso de ficheiros conhecidos possuírem caminhos incompletos.


  2.6 Task 6: The PATH Environment Variable and Set-UID Programs
 
    Nesta tarefa, tal como pedido, foi compilado o programa malicioso apresentado no ficheiro PDF que seguidamente foi atribuído a root e tornado num ficheiro Set-UID. Fazendo o indicado no terminal obtem-se o representado na imagem:
    
   ![VirtualBox_SP_task6](https://user-images.githubusercontent.com/125911788/225457102-b63f6303-23a9-449f-8322-bd44c10e3a04.png)
    
    Como podemos verificar o código malicioso corre no terminal com privilégios root uma vez que mostra todos os ficheiros presentes no diretório currente tal como o /bin/ls faria.
