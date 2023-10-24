# Desafio 1

  O primeiro desafio é bastante simples: temos um programa que cria uma string 'meme_file' com comprimento 8 com o conteúdo "mem.txt\n" dentro e um buffer de tamanho 20. Após a declaração das variáveis, encontramos a vulnerabilidade: um scanf que tenta ler 28 carateres para um buffer de tamanho 20. Curiosamente, o buffer e o 'meme_file' têm um tamanho combinado de 28 carateres e, como ambas as variáveis foram declaradas consecutivamente, estão juntas na stack.

  Uma ideia para explorar a vulnerabilidade referida consiste em encher o buffer com lixo (com chars 'A' no nosso caso) e preencher os 8 carateres do 'meme_file' com "flag.txt". Como a flag encontra-se no ficheiro anteriormente mencionado e este ficheiro acaba por ser aberto logo a seguir ao scanf, o método descrito é suficiente para chegar ao objetivo. Deixamos umas imagens para ilustrar o processo.
  
    - Código fonte do programa:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/232883427-c6cf5015-8355-4af9-b190-47470ac3b3f4.png)

    - Terminal:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/232882908-7537cef1-2b2b-4b1c-889a-dec4dc2b3c70.png)

# Desafio 2

  O segundo desafio é semelhante ao primeiro, havendo uma ligeira diferença nas variáveis. Anteriormente tinhamos apenas um buffer e uma variável para guardar a localização da flag. Desta vez temos uma variável extra que, no momento da declaração, guarda o valor hexadecimal correspondente a 0xdeadbeef (de tamanho 4). A quantidade de carateres que o scanf lê também aumentou, passando de 28 a 32, compensando o tamanho da nova variável. Porém, se avançarmos um pouco no programa, veremos que também foi adicionada uma nova condição (if (*(int*)val == 0xfefc2223)) que bloqueia o nosso acesso à flag. Como é que chegamos lá?
  
  Para chegar à flag vamos ter que alterar o valor de 'val' para 0xfefc2223. Tal como no último desafio, esta variável situa-se em posições consecutivas de memória, junta com 'buffer' e 'meme_file'. Assim sendo, necessitamos preencher estas posições de memória (recorrendo à vulnerabilidade no scanf) de modo a passarmos a nova condição e alcançarmos a flag. Para tal, vamos recorrer ao python2 para criar uma string que codifique 0xfefc2223 em 4 bytes e coloque este mesmo valor no meio da string que criámos no desafio anterior. Segue-se a demonstração do método:
  
    - Código fonte do programa:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/232888998-82a42630-a8e3-4ec2-8a9c-eab5ac53a7e4.png)
  
    - Criação da payload e resultado:
    
   ![imagem](https://user-images.githubusercontent.com/126570489/232888597-d3f7c9ee-c748-43ef-8fc3-54b6c13f0902.png)
   
 O método usado aqui é semelhante ao que foi usado na Semana 7 - Task 2.B ("Heap Data"), explicado nos pontos 1 e 2. Mesmo assim, vamos fazer um pequeno resumo. O python2, ao contrário do python3, codifica carateres hexadecimais em UTF-8 (codificação utilizada pelo programa). Estes carateres têm que ser escritos em grupos de 2 e em ordem invertida de modo a ocuparem os 4 bytes da variável e a respeitar a arquitetura little-endian. Juntando isto aos 20 'A's iniciais e adicionando a localização da flag ao fim da string, conseguimos modificar os valores na ordem correta, fornecendo-nos acesso à flag.
