---
layout: post
title: Snippets Legais com o Emacs [pt-BR]
description: A ideia por trás da mistura de snippets com código
tags: [emacs ruby]
comments: true
---

You can read the english version of this post [here](http://thiagoa.github.io/powerful-emacs-snippets/).

Um dos aspectos legais do editor Emacs é a possibilidade de usar código Lisp para empoderar a expansão de snippets. O propósito desse post é mostrar a potência dessa técnica para que você possa implementá-la no seu próprio editor (Neovim é um bom candidato) se necessário.

## Expansão de snippets via código

Eu tenho dois snippets para gerar declarações de código Ruby para `class` e `module` através do pacote [`yasnippet`](https://github.com/joaotavora/yasnippet). Se eu abrir o arquivo `lib/job_tracker/job.rb` no meu projeto e digitar o seguinte gatilho seguido de `TAB`:

```
cclas
```

Isso gera o seguinte código:

```ruby
module JobTracker
  class Job
    $
  end
end
```

`$` representa a posição final do cursor após expandir o snippet. Eu também tenho um snippet similar, o `cmod`, que gera um `module` ao invés de `class`. Se você é um programador, provavelmente já está imaginando como implementar esse snippet:

1. Obtenha o caminho do arquivo, `lib/job_tracker/job.rb`;
2. Retire o prefixo `lib/`, resultando em `job_tracker/job.rb`;
3. Retire a extensão `.rb`, resultando em `job_tracker/job`;
4. Quebre a string anterior pelo delimitador `/` para obter uma lista de duas strings: `['job_tracker', 'job']`;
5. Aplique camel case nas strings usando a função `map`: `['JobTracker', 'Job']`. Pode-se usar uma biblioteca de camel case ou uma regex avançada;
6. Transforme o primeiro elemento na string `module JobTracker\n`;
7. O segundo elemento se transformaria em `module Job\n` (com dois espaços de indentação). Se estivermos lidando com a última string do array (que é o caso), seria uma classe ao invés de `module`: `class Job\n`;
8. Feche as declarações de `class` ou `module` com `end`, no nível certo de indentação.

Claro, esse algoritmo deve ser genérico.

Meu snippet para isso é o seguinte:

```
`(ruby-code-for-fully-qualified-name-top "class")`
`(ruby-code-for-fully-qualified-name-middle)`$0
`(ruby-code-for-fully-qualified-name-bottom)`
```

> $0 indica a posição do cursor depois de expandir o snippet. Note que chamamos uma função no Lisp com `(funcao)`, e não `funcao()` como é comum em linguagens derivadas do C.

Ele é composto por três chamadas de função que geram as partes do topo, meio, e rodapé do meu código Ruby (existe um motivo para usar três funções ao invés de uma, que se traduz em uma limitação do `yasnippets`). `yasnippets` resolve todo o código Lisp envolvido em backticks (acento grave) e insere o valor de retorno no resultado da expansão do snippet. Nós certamente poderíamos ter, também, strings literais dentro do snippet! Esse é o caso mais comum.

Se você estiver curioso para ver a minha implementação em Lisp, [ela inicia aqui](https://github.com/thiagoa/dotemacs/blob/e5448f3862b0e1e365152d72d8fbe016e753bd74/lib/lang-ruby.el#L221).

## O problema

Eu devo mencionar que o meu snippet para `cclas` era o seguinte:

```
# frozen_string_literal: true

`(ruby-code-for-fully-qualified-name-top "class")`
`(ruby-code-for-fully-qualified-name-middle)`$0
`(ruby-code-for-fully-qualified-name-bottom)`
```

Ele tem o comentário mágico `frozen_string_literal`, para garantir que todas as literais de string dentro do arquivo serão congeladas pelo runtime do Ruby (para evitar mutações nas strings).

Recentemente, eu entrei em um projeto onde `frozen_string_literal` é desnecessário. Como modificar a expansão do meu snippet apenas para esse projeto? O que eu quero é que `frozen_string_literal` seja expandido dependendo de uma configuração específica. Novamente, o Emacs brilha nesse quesito.

## Como tornar o meu snippet configurável?

Para isso, posso codificar uma função em Lisp que lê o valor global de uma variável booleana; se o valor for verdadeiro (`t` em Lisp) quero que retorne a saída `# frozen_string_literal: true` dentro do meu snippet. Seria isso aqui:

```lisp
(defun ruby-code-for-frozen-string-literal ()
  (when ruby-code-for-frozen-string-literal
      "# frozen_string_literal: true\n\n"))
```

E, claro, é necessário definir uma variável (configuração) com a diretiva `defvar`:

```lisp
(defvar ruby-code-for-frozen-string-literal t)
```

O valor `t` significa que `frozen_string_literal` será expandido por padrão dentro do meu snippet. Como modificar essa variável por projeto? O Emacs tem um recurso chamado [Per-Directory Local Variables](https://www.gnu.org/software/emacs/manual/html_node/emacs/Directory-Variables.html) (variáveis locais por diretório). **Isso significa que eu posso sobrescrever qualquer `defvar` dentro do escopo de qualquer projeto**. Legal né? Então eu abri o meu projeto e acionei esse comando:

```
M-x add-dir-local-variable
```

Com ele, eu especifiquei que, para o modo (major mode) `enh-ruby-mode`, eu quero que o valor de `ruby-code-for-frozen-string-literal` seja nulo (`nil`). Então o Emacs criou um arquivo `.dir-locals` na raiz do meu projeto com isso:

```lisp
((enh-ruby-mode . ((ruby-code-for-frozen-string-literal . nil))))
```

Como um arquivo `.dir-locals` já existia, o Emacs apenas incluiu a minha nova configuração nele.

O último passo foi fazer uma pequena mudança no meu snippet:

```lisp
`(ruby-code-for-frozen-string-literal)``(ruby-code-for-fully-qualified-name-top "class")`
`(ruby-code-for-fully-qualified-name-middle)`$0
`(ruby-code-for-fully-qualified-name-bottom)`
```

Eu realmente gostaria que cada chamada de função ficasse em sua própria linha, mas isso tornaria as coisas mais difíceis por motivos que não vou explicar aqui.

E essa é a história de como eu gastei menos de 5 minutos tornando o meu snippet configurável.

## Conclusão

Eu tenho outra configuração em mente para esse snippet em específico, que é gerar declarações de módulo compactas (compact) ou aninhadas (nested) - mas não preciso disso, então não vou implementar agora.

Editores com linguagens de programação embutidas são realmente potentes e customizáveis. Você tem flexibilidade quase infinita para mudar qualquer coisa que quiser! Espero que essa leitura tenha sido interessante.
