# CTF 12

## Desafio 1

### Tarefas

  - Encontrar os números primos p e q secretos.
  
  De acordo com o enunciado, sabemos que p é um primo próximo de 2^512 e q é próximo de 2^513. O website https://www.numberempire.com/primenumbers.php permite calcular o próximo número primo aos valores 2^512 e 2^513, tal como pudemos ver pelas imagens seguintes:
  
  <img width="872" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/b182a5aa-cf28-4900-b97f-63161d7f7f78">

  <img width="881" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/ac0c8403-f63a-4299-aeba-77df988a27e9">

  p = 13407807929942597099574024998205846127479365820592393377723561443721764030073546976801874298166903427690031858186486050853753882811946569946433649006084171
  
  q = 26815615859885194199148049996411692254958731641184786755447122887443528060147093953603748596333806855380063716372972101707507765623893139892867298012168351
  
  - Calcular o expoente privado d utilizando p e q.
  
  Para descobrirmos o expoente privado d iremos usar o Algoritmo de Euclides estendido (https://pt.wikipedia.org/wiki/Algoritmo_de_Euclides_estendido). Seja phi = (p-1)x(q-1).
  
    d é um número tal que d*e % ((p-1)*(q-1)) = 1 <=> (d*e) % phi = 1 && phi = (p-1)*(q-1) <=> d = pow(e, -1, phi)
  
  Tal como mostrámos acima, usaremos a função python pow para encontrar d. Como queremos calcular o inverso multiplicativo modular d, colocamos '-1' como segundo parâmetro da função. A variável d é então dada pelo seguinte código:
  
    #p, e, q definidas antes do código seguinte
    phi = (p-1)*(q-1)
    d = pow(e, -1, phi)
  
  - Recuperar a flag dada usando o d
  
  Quando corremos no terminal o comando 'nc ctf-sp.dcc.fc.up.pt 6000' aparece o seguinte:
  
  <img width="535" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/acb26e53-5923-4327-85b7-950652fbbce6">
  
  Com a flag encriptada criamos agora o seguinte código para a descodificar:
  
    from binascii import hexlify, unhexlify
    
    p = 13407807929942597099574024998205846127479365820592393377723561443721764030073546976801874298166903427690031858186486050853753882811946569946433649006084171
    q = 26815615859885194199148049996411692254958731641184786755447122887443528060147093953603748596333806855380063716372972101707507765623893139892867298012168351
    n = p*q
    e = 0x10001
    phi = (p-1)*(q-1)
    d = pow(e, -1, phi)
    
    enc_flag = "0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001ce05052f3c8190c90b4d4cf61c27375d2c23e03307d18fb8b3ab19540b401bdde9bb82f38fdfd0acae19b1849776fae4492a641d57ef1d1b42606dfda0bc016fe3e2e6eddde650481c1ab367779fed8112855fa061d8abd9bb945ddce36d049626c29329ae941af7cad9cdddbd25a54bce50f5b6dea59b2d5884228cc3d6441c"
    
    def enc(x):
    	int_x = int.from_bytes(x, "big")
    	y = pow(int_x,e,n)
    	return hexlify(y.to_bytes(256, 'big'))
    
    def dec(y):
    	int_y = int.from_bytes(unhexlify(y), "big")
    	x = pow(int_y,d,n)
    	return x.to_bytes(256, 'big')
    
    y = dec(enc_flag)
    print(y.decode())
    
  Com isto descobrimos a flag: flag{640bab8e44dfe8fda0d51d052062cce5}.
  
## Desafio 2

### Tarefas 

  - Reflete na informação que possuis e pesquisa sobre possíveis ataques ao RSA que sejam aplicáveis. 
  
  Para encontrar a flag neste desafio são fornecidos 2 valores e diferentes (e1 = 0x10001 e e2 = 0x10003) e um valor para n (que tem um tamanho de 617 carateres).
  O algoritmo RSA é composto por 2 chaves: a chave pública (chave de encriptação) e uma chave privada (chave de descriptação). Sendo a mensagem encriptada C, a mensagem original m, C é originada pela fórmula C = m^e mod n.
  Um teorema que iremos precisar será o Teorema de Bezout que afirma que, dados inteiros a e b, não ambos nulos, existem inteiros m e n tais que am + bn = mdc(a, b).
  Se mdc(e1, e2) = 1, então teremos 2 inteiros x e y tal que xe1 + ye2 = 1.
  Usando o Algoritmo de Euclides estendido conseguimos encontrar x e y. Usando esta informação na fórmula de encriptação...
  
  <img width="220" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/32804352-e21b-47bf-92ca-0327ccf8321e">
  
  Com base nestes cálculos concluímos que, para quebrar a encriptação RSA e obter a flag precisamos de calcular isto:
  
  <img width="116" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/e2c524d4-701a-4c17-80dc-7eac45038997">
  
  - Implementa o ataque correto. 
  
  Correndo no terminal a porta 6001 de ctf-sp.dcc.fc.up.pt obtemos as mensagens codificadas que iremos descodificar. A primeira mensagem é b67542b17862cf87e1addd8dae6a8e3219603de3937a0f51a6b0db9faf3cd011ae5b8b8a7b21863cf78314feb7eb9ab71e8fbd38f57ce6470b76007ce5b50d6a3a21ab4a07f20080c827a004d43f7b492a7dd5478c72fc522f48c3fd995503177d58600acae237d46be4e407aa6ae32a6f8467754d9954a34d657c5d495d5c073fe15d8046bd1152e9fbbc66a2b2f7554fe88490c5919225176288ceb5f84c1627a6e78f0ce23fa324021d745da696d76660194ad3ed18f20b3c964e78736ce49ea83b18b677a0268411674e96e27050d79117bfc16e733db2297dcbfab00b6e1425222da065f1db0e1e70ee86215e693b087a24afd6927492c2391a4a41235d e a segunda mensagem é a232c82fc17d39b02d99f36bab09ec342e3af0f03d4f2886bef26917c08ccda35e58fea69745a947241fde2a97daa4df01ba59fe909030d0ac1f737b6ac73fe0bc4239abe7d263018cef784c6721a2578f19223dde3c49067d1851a8997927aaa0b809f156f23b03746fab33d44fc0e24cfcc106a719426123f6362d882bab5b4af9b8eebb8c8a8c336fb16f0f5ee4b51b78a712713b3cff89758760bff3ad72dbac1243bf80dcd6366d5f9ac9d93979e73bd2be3b1a0d7e240e9df739ab73a6ed10e3b4e0040c85cd11b47d16fac2c5f41e75057f1bc1f04aa38c2bafc943de76424447735f8eef2bd91de6a5b4557d89d4314b838e17a59d322cdcfeaf4f2a.
  Com todas as informações que recolhemos até agora podemos fazer o código para descodificar as mensagens:
  
    from math import gcd
    from binascii import hexlify, unhexlify
    
    n = 29802384007335836114060790946940172263849688074203847205679161119246740969024691447256543750864846273960708438254311566283952628484424015493681621963467820718118574987867439608763479491101507201581057223558989005313208698460317488564288291082719699829753178633499407801126495589784600255076069467634364857018709745459288982060955372620312140134052685549203294828798700414465539743217609556452039466944664983781342587551185679334642222972770876019643835909446132146455764152958176465019970075319062952372134839912555603753959250424342115581031979523075376134933803222122260987279941806379954414425556495125737356327411
    e1 = 0x10001
    e2 = 0x10003
    c1 = int.from_bytes(unhexlify("b67542b17862cf87e1addd8dae6a8e3219603de3937a0f51a6b0db9faf3cd011ae5b8b8a7b21863cf78314feb7eb9ab71e8fbd38f57ce6470b76007ce5b50d6a3a21ab4a07f20080c827a004d43f7b492a7dd5478c72fc522f48c3fd995503177d58600acae237d46be4e407aa6ae32a6f8467754d9954a34d657c5d495d5c073fe15d8046bd1152e9fbbc66a2b2f7554fe88490c5919225176288ceb5f84c1627a6e78f0ce23fa324021d745da696d76660194ad3ed18f20b3c964e78736ce49ea83b18b677a0268411674e96e27050d79117bfc16e733db2297dcbfab00b6e1425222da065f1db0e1e70ee86215e693b087a24afd6927492c2391a4a41235d"), "big")
    c2 = int.from_bytes(unhexlify("a232c82fc17d39b02d99f36bab09ec342e3af0f03d4f2886bef26917c08ccda35e58fea69745a947241fde2a97daa4df01ba59fe909030d0ac1f737b6ac73fe0bc4239abe7d263018cef784c6721a2578f19223dde3c49067d1851a8997927aaa0b809f156f23b03746fab33d44fc0e24cfcc106a719426123f6362d882bab5b4af9b8eebb8c8a8c336fb16f0f5ee4b51b78a712713b3cff89758760bff3ad72dbac1243bf80dcd6366d5f9ac9d93979e73bd2be3b1a0d7e240e9df739ab73a6ed10e3b4e0040c85cd11b47d16fac2c5f41e75057f1bc1f04aa38c2bafc943de76424447735f8eef2bd91de6a5b4557d89d4314b838e17a59d322cdcfeaf4f2a"), "big")
    
    s1 = pow(e1, -1, e2)
    s2 = int((gcd(e1,e2) - e1 * s1) / e2)
    temp = pow(c2, -1, n)
    m1 = pow(c1,s1,n)
    m2 = pow(temp,-s2,n)
    flag = ((m1 * m2) % n).to_bytes(256, 'big')
    print(flag.decode())
    
  Quando corrermos o programa conseguiremos descobrir a flag.
  
  - Recupera a flag.
  
  Correndo o código criado é fornecida a flag: flag{522e7ee5d35c1f11876f0ca066423f1a}.
  
  <img width="535" alt="image" src="https://github.com/DCC-FCUP-SP/sp2223-t01g03/assets/125911788/46efcdc3-b679-4c72-9202-74c16ff8c098">
