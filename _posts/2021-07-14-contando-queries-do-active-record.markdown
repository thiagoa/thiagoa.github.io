---
layout: post
title: Contando Queries do Active Record [pt-BR]
description:
tags: [rails,ruby,activerecord]
comments: true
---

You can read the english version of this post [here](http://thiagoa.github.io/counting-active-record-queries/).

Recentemente, colegas de trabalho ficaram impressionados com um código que escrevi para contar queries do Active Record. Mas por que alguém faria isso? Existem boas razões!

Eu gosto de usar TDD, desenvolvimento orientado a testes, quando a situação é propícia para tanto. Eu estava tentando descobrir uma forma de otimizar a travessia de uma árvore de objetos Active Record implementada com a gem [`ancestry`](https://github.com/stefankroes/ancestry). O problema é que o GraphQL estava acessando a árvore da seguinte maneira:

```ruby
# Pseudo-código para explicar o problema
#
# A coleção "children" é fornecida pela gem ancestry
publisher.root_nav.children do |child|
  # Recursivamente acesse 'child.children' até chegar na profundidade máxima da árvore...
end
```

Para quem conhece o Active Record, é bastante visível que o código acima sofre de N+1, onde cada filho da coleção `children` roda uma query adicional para buscar outra coleção de filhos e por aí vai. O primeiro passo para resolver o problema foi escrever alguns testes e interceptar o lugar do código onde o "root nav" estava sendo buscado, então eu comecei com o seguinte código:

```ruby
class NavItemsQuery
  def self.call(root)
    root
  end
end
```

O ponto de chamada foi mudado disso:


```ruby
publisher.root_nav
```

Para isso:

```ruby
NavItemsQuery.call(publisher.root_nav)
```

Claro, isso não mudou nada mas foi um ótimo começo. Pelo menos eu criei a classe onde iria escrever o meu código, e meus testes simplesmente se limitaram a verificar o conteúdo da árvore até atingir a profundidade máxima (`max_depth`).

O próximo passo foi reutilizar os mesmos testes, mas otimizar a estratégia de obtenção dos dados. Em outras palavras, o comportamento seria mantido mas o código seria otimizado. Seguindo um workflow com TDD, como fazer essa garantia? Uma maneira possível é contar as queries SQL! Com isso em mente, eu criei o seguinte teste:

```ruby
# Tradução: executa três queries ou menos
it 'execute three queries or less' do
  root = create_tree(depth: 3)
  count = 0

  begin
    ActiveSupport::Notifications.subscribe('sql.active_record') { count += 1 }
    NavItemsQuery.call(root, max_depth: 3)

    # A contagem de queries deve ser menor ou igual que 3
    expect(count).to be <= 3
  ensure
    ActiveSupport::Notifications.unsubscribe('sql.active_record')
  end
end
```

Isso me pareceu bem razoável porque meu teste estava exercitando uma árvore com profundidade 3, e depois de alguma análise eu descobri que podia rodar uma query para cada nível da árvore.

Mas como a contagem de queries funciona? O Active Record tem um mecanismo de pubsub (publish-subscribe) onde você pode se inscrever usando a string `sql.active_record` para contar toda e qualquer query SQL executada pelo Rails, e eu simplesmente usei a closure do bloco para contar as queries. Legal né?

Como eu tinha mais de um teste contando queries, eu criei um um helper:

```ruby
def counting_active_record_queries
  count = 0
  ActiveSupport::Notifications.subscribe('sql.active_record') { count += 1 }
  yield
  count
ensure
  ActiveSupport::Notifications.unsubscribe('sql.active_record')
end
```

E então eu consegui transformar o teste acima nessa maravilha:

```ruby
# Tradução: executa três queries ou menos
it 'execute three queries or less' do
  root = create_tree(depth: 3)

  count = counting_active_record_queries do
    NavItemsQuery.call(root, max_depth: 3)
  end

  # A contagem de queries deve ser menor ou igual que 3
  expect(count).to be <= 3
end
```

Excelente! Eu tenho que dizer que durante a sessão de TDD, a asserção final do teste se transformou nisso:

```ruby
# A contagem de queries deve ser 1
expect(count).to be 1
```

O motivo é que eu descobri uma forma de buscar tudo com uma única query SQL e melhorar ainda mais a performance, mas vou deixar isso para outro post :)

## Bonus: Debugando queries do Active Record

Sabe aquela query que você vê nos logs do Rails e não tem ideia de onde vem, como uma agulha no palheiro? Você pode usar as notificações do Active Record para encontrá-las também!

```ruby
ActiveSupport::Notifications.subscribe('sql.active_record') do |_, _, _, _, details|
  if details[:sql] =~ MINHA_REGEX
    puts '*' * 50
    puts details[:sql]
    puts caller.join("\n")
    puts '*' * 50
  end
end
```

Esse truque consiste em:

1. Se inscrever nas queries Active Record, como explicado anteriormente;
2. A variável `details[:sql]` contém a query SQL, que você pode usar para fazer o match com uma regex da query que você está procurando;
3. Imprimir o backtrace do código com `caller.join("\n")` para ver onde a query está sendo executada.

Você pode até mesmo usar o `binding.pry` dentro do bloco se precisar.

## Conclusão

Esse post foi rapidinho, mas espero ter sido útil. Você pode aprender mais sobre as notificações do Active Record [aqui](https://api.rubyonrails.org/classes/ActiveSupport/Notifications.html) (em inglês).
