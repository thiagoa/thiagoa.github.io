---
layout: post
title: Dicas de Histórico de Linha de Comando [pt-BR]
description: Evite retrabalho
tags: [shell bash zsh]
comments: true
---

You can read the english version of this post [here](http://thiagoa.github.io/command-line-history-tricks/).

Para qualquer tarefa executada no computador, podem existir atalhos para acelerar a mesma ou ser mais preciso. O problema não é lentidão; o ponto onde quero chegar é, arrastar um cursor letra por letra (segurando as setinhas ou algo assim), copiar e colar com o mouse, etc, rapidamente se torna algo enfadonho e te deixa cansado se feito muitas vezes no decorrer do dia.

Quando se fala do shell, existem várias maneiras de ser preciso. Shells têm capacidades de edição e expansão de histórico bem avançadas. Eu não quero encher você de conceitos nesse post, ou mencionar todos os recursos do histórico. O objetivo é te dar atalhos e um modelo mental para que se tornem rememoráveis, pois podem não ser tão fáceis de lembrar.

## O caractere de expansão do shell

O caractere de expansão do shell é `!` (bang). Coloque isso na cabeça agora mesmo! Iniciando com o bang, você pode expandir:

- Qualquer elemento do primeiro comando: o comando completo, um argumento específico, uma faixa de argumentos, etc.
- Qualquer elemento do último comando: o comando completo, um argumento específico, uma faixa de argumentos, etc.
- Qualquer elemento de qualquer comando anterior!

## Exemplo preliminar

Antes de tudo, gostaria de te dar uma ideia geral sobre como o histórico do shell pode ser usado de forma produtiva.

Para referenciar argumentos do comando atual, use o atalho `!#:N`. Vamos supor que você queira renomear um arquivo:

```sh
mv uma/pasta/com/nome/grande !#:1
```

Ao digitar `!#:1` e pressionar `TAB`, sua linha de comando vai expandir para:

```sh
mv uma/pasta/com/nome/grande uma/pasta/com/nome/grande
```

> TAB é usado para editar a expansão ou para acessibilidade. O shell vai expandir o conteúdo automaticamente ao executar o comando.

Agora é possível editar o último argumento como quiser, o que, provavelmente, seria mais rápido do que redigitar ou copiar e colar.

Eu escolhi expandir o argumento 1. Argumentos iniciam em 0, então `#!:0` repete o primeiro argumento, `#!:1` repete o segundo, e por aí vai.

`!#:N` é bem estranho, mas eu tenho uma dica. Lembre-se que cerquilhas ou hashtags (#) geralmente são usadas para fazer referência a números - #0, #1, #2, etc. -, e, como você já sabe, o caractere de expansão do shell é `!` (bang). Então para fazer referência ao primeiro argumento é só digitar `!#1` e depois colocar dois pontos (`:`) entre `#` e `1`.

Da mesma forma, é possível obter o comando anterior do histórico com `!!` (dois bangs). Por exemplo, se você esqueceu do `sudo` no comando anterior, faça o seguinte:

```sh
systemctl stop gdm # Ooops...
sudo !! # Isso é o que você quer!
```

Como esperado, é possível obter argumentos individuais do último comando com `!!:N`. Para o último argumento, use um cifrão (`$`), da seguinte maneira: `!!:$` -, o que é muito útil para digitar sequências de comandos onde é sempre necessário repetir o último argumento:

```sh
mkdir -p alguma/pasta
cd !!:$ # Vou explicar depois como simplificar isso...
```

## Expandindo qualquer comando: Designadores de Evento

Existe um padrão bem tranquilo de se seguir:
É possível fazer referência a qualquer comando, incluindo a linha atual:

- `!#` - Comando atual
- `!!` or `!-1` - Último comando
- `!-2` - Penúltimo comando
- `!-3` - Antepenúltimo comando
- `!-N` - Comando anterior nº N
- `!padrao` - O último comando que inicia com `padrao`

Vamos chamar os elementos acima de "designadores de evento". Designadores de Evento especificam a linha do histórico que será utilizada, e podem ser manipulados para extrair palavras ou faixas de palavra, que podem ser inclusive modificadas (não falaremos sobre modificadores nesse artigo). Inicie com os padrões acima para fatiar qualquer parte de qualquer comando, mas note que eles por si só expandem toda a linha! Se você quer repetir tudo que já digitou, use:

```sh
ls algum/caminho/de/pasta !#
```

Que vai expandir para:

```sh
ls algum/caminho/de/pasta ls algum/caminho/de/pasta
```

Isso não é muito útil, certo? Lembre-se: expandir o comando completo é mais útil para fazer referência a comandos anteriores. `sudo !!` é um ótimo exemplo.

Claro, você pode também repetir comandos anteriores completos:

```sh
!-2
```

É a mesma coisa que pressionar `ctrl+p` ou setinha para cima duas vezes para chegar no penúltimo comando.

## Expandindo palavras específicas

Vamos supor que o dois pontos (`:`) é o "operador para pegar palavras". "Palavra" (word) é a terminologia usada pelo `bash`, mas "argumento" é mais fácil de entender. Se você quer obter o argumento N, comece com um designador de palavra e termine com `:N`. Por exemplo:

- `!#:N` - Argumento N do comando atual
- `!!:N` - Argumento N do último comando
- `!-2:N` - Argumento N do penúltimo comando
- `!-3:N` - Argumento N do antepenúltimo comando
- `!-M:N` - Argumento N do comando nº M
- `!padrao:N` - Argumento N do comando que inicia com `padrao`

Onde N é um inteiro que inicia em 0. Ok? O padrão é o seguinte:

```
designador_de_evento:N
```

## Obtendo faixas de palavras

Faixas de palavra podem ser extraídas de forma semelhante. Aqui `:` vai assumir o papel de "operador de faixas de palavras".

- `!#:A-B` - Argumentos A até B do comando atual
- `!!:A-B` - Argumentos A até B do último comando
- `!-2:A-B` - Argumentos A até B do penúltimo comando
- `!-3:A-B` - Argumentos A até B do antepenúltimo comando
- `!-M:A-B` - Argumentos A até B do comando nº M
- `!padrao:A-B` - Argumentos A até B do comando que inicia com `padrao`

Onde A e B são inteiros que iniciam em 0. O padrão aqui é:

```
designador_de_evento:A-B
```

Existem designadores de palavra reservados que evitam a necessidade de contar palavras. Eles podem ser usados no lugar do A ou B:

- `^` para fazer referência ao primeiro argumento
- `$` para o último argumento

Exemplo:

```sh
co algum/arquivo outro/arquivo # Opa! Deveria ter sido um cp, e não co...
cp !!:^-$ # Agora está ok!
```

A faixa `^-$` é tão especial que existe um atalho para ela, que é `*`. Por exemplo, se você quiser expandir todos os argumentos do penúltimo comando, use `!-1*`:

```sh
co algum/arquivo outro/arquivo # Oopa! Deveria ter sido cp, e não co...
mkdir outra/pasta # Sem problemas, vou continuar fazendo outras coisas
cp !-1* # Agora está ok!
```

Ok, isso não é muito comum, pois shells modernos iriam te sugerir o comando `cp`, mas a ideia é ser preciso quando necessário!

Veja [word designators](https://www.gnu.org/software/bash/manual/html_node/Word-Designators.html) (em inglês) para mais detalhes.

## A última palavra: o caso mais comum

Como eu disse anteriormente, `!$` representa o último argumento do último comando. É um atalho para `!!$`, que por sua vez é um atalho para `!!:$`.

Então temos:

- `!#$` - Último argumento do comando atual
- `!!$`, `!!:$` ou `!$` - Último argumento do último comando
- `!-2$` - Último argumento do penúltimo comando
- `!-3$` - Último argumento do antepenúltimo comando
- `!-M$` - Último argumento do comando nº M
- `!padrao:A-B` - Último argumento do comando que começa com `padrao`

Você consegue enxergar um padrão aí?

> BONUS: `alt+.` é um atalho para expandir `!$`. O caso mais comum tinha que ter um atalho dedicado, não é mesmo?

## Conclusão

Use esses atalhos para nivelar o seu uso do terminal! Um modelo mental é importante, pois sem ele os comandos iriam parecer bem esquisitos, mas eles seguem um padrão consistente que você pode aprender e praticar. Espero que tenha achado isso útil!
