
# Trabalho realizado na Semana #3

## Identificação

- pipenv é uma ferramenta que desempenha funções semelhantes a pip e virtualenv da bash;
- pode instalar packages do Python segundo um ficheiro de requisitos ("pipenv install -r requirements.txt");
- está sujeita à utilização de URLs maliciosos através de requisitos customizados;
- Presente da versão 2018.10.9 até à 2022.1.8.

## Catalogação

- Gravidade da vulnerabilidade: 9.3/10 (crítico/severo);
- Descoberta a 08-01-2022;
- Chris Passarello decobriu e reportou o problema.

## Exploit

- O exploit é do tipo RCE(remote code execution);
- Recorre a strings bem elaboradas para disfarçar links maliciosos nos comentários do programa;
- Os links direcionam a um servidor controlado pelo atacante;
- O servidor contém packages malignas.

## Ataques

- O impacto do ataque depende da experiência do atacante;
- Pode ser usado para instalar packages maliciosas;
- As packages podem roubar credenciais, utilizar os recursos da máquina ou instalar malware.

