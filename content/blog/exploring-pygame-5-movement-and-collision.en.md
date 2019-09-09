+++
description = "Learn how to move the objects on the screen and write you first collision detection with pygame"
draft = true
images = []
katex = true
markup = "mmark"
publishDate = "2019-08-30T00:00:00-03:00"
tags = ["python", "pygame", "gamedev"]
title = "Exploring pygame 5 - Movement and Collision"

+++
Movement is a significant characteristic of a large part of the games. When jumping between platforms, shooting against a horde of enemies, piloting a space ship and running through the streets, we are causing movement and interacting with the game environment, applying action and causing reactions.

This chapter is to describe the basic concepts of moving objects across the screen and their interaction with other elements through collision detection.

## Movement

If you are following this series of posts, you already saw an example of movement at the post about [the game loop](https://humberto.io/pt-br/blog/desbravando-o-pygame-3-game-loop/), where we implemented a ball that moves through the screen bouncing around.

This time are revisiting some parts of that code, adding more detail to it and also, adding some new concepts.

To start, let's write the most basic form of movement of an object through the screen:

{{< highlight python "linenos=table,hl_lines=21 26" >}}
import pygame

BLACK = pygame.Color(0, 0, 0)
WHITE = pygame.Color(255, 255, 255)

pygame.init()

screen = pygame.display.set_mode((640, 480))

pygame.display.set_caption('Simple Movement')

position_x = 0

while True:
    event = pygame.event.poll()

    if event.type == pygame.QUIT:
        break

    # moves the square one pixel per cycle
    position_x += 1

    screen.fill(BLACK)

    # draws the square with the incremented position
    pygame.draw.rect(screen, WHITE, [position_x, 230, 20, 20])

    pygame.display.flip()
{{< / highlight >}}

O código é bem direto ao ponto, criamos a variável `position_x` para guardar a posição da bola no eixo x.

Dentro do loop sua posição é incrementada em um pixel a cada ciclo e a bola é desenhada novamente em sua nova posição.

Esta abordagem possuí um problema. Você não consegue ter controle sobre a velocidade de movimento da bola. Em computadores mais rápidos mais loops por segundo serão processados e nos mais lentos o contrário e eventualmente terá resultados como este:

{{< videogif "/img/exploring-pygame/square-fast.webm" >}}

Para corrigir este problema precisamos voltar as aulas de física quando nos ensinaram sobre o **M**ovimento **R**etilíneo **U**niforme. Para garantirmos uma velocidade constante usaremos a seguinte fórmula:

$$S = S_{i} + v \\Delta t$$

Sua aplicação no código ficará desta forma:

{{< highlight python "linenos=table,hl_lines=16 19 24 26 28 36" >}}
import time

import pygame

BLACK = pygame.Color(0, 0, 0)
WHITE = pygame.Color(255, 255, 255)

pygame.init()

screen = pygame.display.set_mode((640, 480))

pygame.display.set_caption('Velocity')

position_x = 0
# 100 pixels per second
velocity_x = 100

# capture the initial time
ti = time.time()

while True:
    # gets the time for
    # this cycle
    tf = time.time()
    # calculate the delta
    dt = (tf - ti)
    # sets final time as the initial time
    ti = tf

    event = pygame.event.poll()

    if event.type == pygame.QUIT:
        break

    # moves the square at the velocity defined
    position_x += velocity_x * dt

    screen.fill(BLACK)

    pygame.draw.rect(screen, WHITE, [position_x, 230, 20, 20])

    pygame.display.flip()
{{< / highlight >}}

Começamos definindo a velocidade da bola no eixo x para 100 pixels por segundo na **linha 16**.

Em seguida na **linha 19** capturamos o tempo inicial para o cálculo do delta de tempo, que é quanto tempo se passou entre os ciclos do loop.

Entrando no loop capturamos o tempo final na **linha 24** e logo em seguida na **linha 26** calculamos o `dt` subtraindo o tempo final pelo inicial.

Na **linha 28** o tempo inicial passa a ser o tempo final para que possamos usá-lo no próximo ciclo.

Por fim calculamos o deslocamento que será feito na **linha 36**.

{{< videogif "/img/exploring-pygame/square-velocity.webm" >}}

Como podemos ver agora é possível controlar a velocidade da bola. Porém, isso só resolve a parte visível do problema, o loop continua sendo executado muito mais que o necessário. Nem o olho humano, nem a taxa de atualização do seu monitor vai conseguir acompanhar um volume exagerado te atualizações consecutivas além da sobrecarga desnecessária do processador.

## FPS

O controle de atualização de tela é uma prática comum dentro do mundo do áudio visual e sua unidade de medida é o **FPS** (**F**rames **P**er **S**econd).

Ter este controle implementado em seu jogo trás uma série de benefícios:

* Reduz o uso desnecessário do uso dos recursos da máquina
* Facilita a sincronização de jogos multiplayer
* Diminui a as perdas em operação de ponto flutuante. Por realizar muitas operações com frações de tempo muito pequenas em uma frequência exagerada erros de operação de ponto flutuante crescem muito mais rápido (existem técnicas mais sofisticadas para mitigar este tipo de problema mas para quem está começando isso não se preocupe com isto neste momento)
* Aumenta a previsibilidade e facilita o planejamento do seu jogo. Coisas do tipo para um computador com os requisitos mínimos, quanta coisa eu consigo processar no intervalo de um frame para outro?

No desenvolvimento de jogos o mais comum é encontrar uma taxa de atualização entre 30 e 60 fps. No nosso caso utilizaremos uma taxa de atualização de 30fps.

{{< highlight python "linenos=table,hl_lines=16 19 24" >}}
import pygame

BLACK = pygame.Color(0, 0, 0)
WHITE = pygame.Color(255, 255, 255)

pygame.init()

screen = pygame.display.set_mode((640, 480))

pygame.display.set_caption('FPS')

position_x = 0
# since pygame clock returns its value
# in milliseconds, we divide the velocity
# by 1000 to keep the 100 pixel per seconds
velocity_x = 0.1

# create pygame clock
clock = pygame.time.Clock()

while True:
    # call the clock tick for 30 fps
    # and store the delta time
    dt = clock.tick(30)

    event = pygame.event.poll()

    if event.type == pygame.QUIT:
        break

    position_x += velocity_x * dt

    screen.fill(BLACK)

    pygame.draw.rect(screen, WHITE, [position_x, 230, 20, 20])

    pygame.display.flip()
{{< / highlight >}}

Graças a implementação de relógio da classe `Clock` do pygame, não tivemos muito trabalho com a implementação, na verdade o código fica até mais curto. Primeiramente trocamos a velocidade de `100` para `0.1` na **linha 16**, pois diferentemente da biblioteca `time` do Python que trabalha com segundos, o `Clock` do pygame trabalha em milissegundos e para garantir a mesma velocidade de cem pixels por segundo precisamos dividir a velocidade por `1000`.

Na **linha 19** instanciamos o `Clock` e logo mais, na **linha 24** chamamos sua função `tick` passando como argumento a quantidade de **FPS** que queremos limitar nosso loop de jogo.

A função `tick` deve ser chamada a cada ciclo e caso o ciclo anterior tenha sido muito rápido ela para a execução do programa por um breve tempo para manter a frequência desejada. Como resultado, esta função retorna o delta de tempo entre esta e a vez anterior em que ela foi chamada.

{{% tip class="info" %}}
Dê uma olhada na [documentação da função ](https://www.pygame.org/docs/ref/time.html#pygame.time.Clock.tick)`[tick](https://www.pygame.org/docs/ref/time.html#pygame.time.Clock.tick)`, ela possuí uma questão quanto a precisão entre plataformas, mas existe uma função alternativa mais precisa (porém mais pesada) que pode realizar este trabalho caso esta precisão seja importante para o seu jogo.
{{% /tip %}}

Agora que temos nossa bola percorrendo a tela a uma velocidade constante podemos seguir para a etapa de detecção de colisão.

## Colisão

A colisão é o produto da interação dos objetos do seu jogo. Esta interação pode ocorrer entre si e com o ambiente. A detecção de colisão costuma crescer em complexidade na medida em que mais elementos de diferentes formatos são adicionados em cena.

Em nosso exemplo vamos nos ater aos conceitos básicos fazendo a bola interagir com os limites da tela mudando de direção ao colidir com suas extremidades:

{{< highlight python "linenos=table,hl_lines=12-19 33-34 36-38 42-47" >}}
import pygame

BLACK = pygame.Color(0, 0, 0)
WHITE = pygame.Color(255, 255, 255)

pygame.init()

screen = pygame.display.set_mode((640, 480))

pygame.display.set_caption('Collision')

# create the square Rect
square = pygame.Rect(300, 230, 20, 20)

# create the pads Rect
left_pad = pygame.Rect(20, 210, 20, 60)
right_pad = pygame.Rect(600, 210, 20, 60)

pads = [left_pad, right_pad]

velocity_x = 0.1

clock = pygame.time.Clock()

while True:
    dt = clock.tick(30)

    event = pygame.event.poll()

    if event.type == pygame.QUIT:
        break

    # use the move function inplace
    square.move_ip(velocity_x * dt, 0)

    # check for collision with the pads
    if square.collidelist(pads) >= 0:
        velocity_x = -velocity_x

    screen.fill(BLACK)

    # draw using the rect
    pygame.draw.rect(screen, WHITE, square)

    # draw the pads
    for pad in pads:
        pygame.draw.rect(screen, WHITE, pad)

    pygame.display.flip()
{{< / highlight >}}

A técnica de detecção de colisão mais simples é a de tratar todos os elementos como áreas retangulares e o pygame implementa esta mecânica através da classe `Rect` que foi utilizada a partir da **linha 12** onde foi criada uma área retangular para a bola seguida da criação de dois blocos com os quais a bola irá se colidir.

Com a criação do `Rect` para a bola, passamos a usar a função `move_ip` para deslocá-la na **linha 34**. Esta função altera a posição do objeto que a chama, diferentemente da função `move` que retorna uma cópia do objeto com sua posição alterada.

Na **linha 37** a função `collidelist` verifica se ocorreu alguma colisão com um dos elementos da lista, retornado seu índice em caso positivo e `-1` em caso negativo.

E por fim a bola e os pads são desenhados na tela utilizando suas instâncias de `Rect` produzindo o resultado a seguir:

{{< videogif "/img/exploring-pygame/square-collision.webm" >}}

## Conclusão

Com estes conceitos de movimentação e colisão já é possível criar jogos bem interessantes como o [Pong](https://pt.wikipedia.org/wiki/Pong). Vou encerrar esta postagem deixando como proposta que você utilize estes conceitos para implementá-lo.

Os códigos utilizados nesta postagem estão disponíveis em [exploring-pygame](https://github.com/humrochagf/exploring-pygame/tree/master/05-movement-and-collision).