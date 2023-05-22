# Bézier Curves

[https://pomax.github.io/bezierinfo/#yforx](https://pomax.github.io/bezierinfo/#yforx)

Na documentação acima é utilizado pseudo-código para os exemplos, isso para evitar a cópia sem o entendimento do negócio. Nesta documentação irei utilizar a linguagem typescript e o Framework React Nativa para representar a matemática de forma gráfica. Irei utilizar uma biblioteca chamada React Nativa Skia para desenhar de forma nativa o que estamos vendo em forma de calculo.

Levo em consideração que você ja tenha instalado e configurado o React Nativa Skia, e já tenha um projeto previamente pronto.

Antes de se aprofundar em Skia e começar a desenhar tudo, é importante entender como o computador entende os desenhos. Para um humano, basta pegar o lapis e desenhar qualquer coisa. Mas um computador não consegue desenhar cosias de forma fácil, para ele, tudo que será renderizado será uma função.

Pense em desenhar uma reta com uma caneta, extremamente fácil. Para retas, um computador desenha de forma fácil também, é só lembrar no ensino médio, estou falando de polinômio de primeiro grau. Simples e fácil.

Agora o problema começa quando queremos que o nosso querido pc desenhe uma curva 😃. Para isso existe uma função que nos permite desenhar curvas, Bézier Curves que também é um polinômio.

Você pode vincular várias curvas de Bézier para que a combinação pareça uma única curva. Se você já desenhou "caminhos" no Photoshop ou trabalhou com programas de desenho vetorial como Flash, Illustrator ou Inkscape, essas curvas que você desenhou são curvas de Bézier.

Mas e se você mesmo precisar programá-los? Quais são as armadilhas? Como você os desenha? Quais são as caixas delimitadoras, como você determina as interseções, como você pode extrudar uma curva, em resumo: como você faz tudo o que deseja fazer com essas curvas? É para isso que serve esta página. Prepare-se para ser matemático!

Existem duas maneiras de explicar o que é uma curva de Bézier, e elas são totalmente equivalentes, mas uma delas usa matemática complicada e a outra usa matemática realmente simples. Então... vamos começar com a explicação simples:

### simples

As curvas de Bézier são o resultado de [interpolações lineares](https://en.wikipedia.org/wiki/Linear_interpolation) . Isso parece complicado, mas você faz interpolação linear desde muito jovem: sempre que precisa apontar para algo entre duas outras coisas, você aplica interpolação linear. É simplesmente "escolher um ponto entre dois pontos".

![https://pomax.github.io/bezierinfo/images/chapters/whatis/c7f8cdd755d744412476b87230d0400d.svg](https://pomax.github.io/bezierinfo/images/chapters/whatis/c7f8cdd755d744412476b87230d0400d.svg)

### complicada: cálculo

As curvas de Bézier são uma forma de função "paramétrica". Matematicamente falando, as funções paramétricas são trapaças: uma "função" é, na verdade, um termo bem definido que representa um mapeamento de qualquer número de entradas para uma **única** saída. Números entram, um único número sai. Mude os números que entram e o número que sai ainda é um único número.

Fraude de funções paramétricas. Eles basicamente dizem "tudo bem, bem, queremos vários valores saindo, então usaremos apenas mais de uma função". Uma ilustração: digamos que temos uma função que mapeia algum valor, vamos chamá-lo de *x* , para algum outro valor, usando algum tipo de manipulação numérica:

![https://pomax.github.io/bezierinfo/images/chapters/explanation/0cc876c56200446c60114c1b0eeeb2cc.svg](https://pomax.github.io/bezierinfo/images/chapters/explanation/0cc876c56200446c60114c1b0eeeb2cc.svg)

A notação *f(x)* é a forma padrão de mostrar que é uma função (chamada por convenção de *f* se estivermos listando apenas uma) e sua saída muda com base em uma variável (neste caso, *x* ). Mude *x* , e a saída para *f(x)* muda.

Até agora tudo bem. Agora, vamos ver as funções paramétricas e como elas trapaceiam. Vamos pegar as duas funções a seguir:

![https://pomax.github.io/bezierinfo/images/chapters/explanation/a2891980850ddbb27d308ac112d69f74.svg](https://pomax.github.io/bezierinfo/images/chapters/explanation/a2891980850ddbb27d308ac112d69f74.svg)

Não há nada realmente notável sobre eles, eles são apenas uma função seno e cosseno, mas você notará que as entradas têm nomes diferentes. Se alterarmos o valor de *a* , não alteraremos o valor de saída de *f(b)* , pois *a* não é usado nessa função. As funções paramétricas trapaceiam ao mudar isso. Em uma função paramétrica, todas as diferentes funções compartilham uma variável, como esta:

![https://pomax.github.io/bezierinfo/images/chapters/explanation/7acc94ec70f053fd10dab69d424b02a6.svg](https://pomax.github.io/bezierinfo/images/chapters/explanation/7acc94ec70f053fd10dab69d424b02a6.svg)

Várias funções, mas apenas uma variável. Se alterarmos o valor de *t* , alteramos o resultado de *f a (t)* e *f b (t)* . Você pode se perguntar como isso é útil, e a resposta é realmente muito simples: se mudarmos os rótulos *f a (t)* e *f b (t)* pelo que normalmente significamos com eles para curvas paramétricas, as coisas podem ser muito mais óbvias:

![https://pomax.github.io/bezierinfo/images/chapters/explanation/6914ba615733c387251682db7a3db045.svg](https://pomax.github.io/bezierinfo/images/chapters/explanation/6914ba615733c387251682db7a3db045.svg)

Aqui vamos nós. *coordenadas x* / *y* , ligadas por algum valor misterioso *t* .

Portanto, as curvas paramétricas não definem uma coordenada *y* em termos de uma coordenada *x* , como fazem as funções normais, mas, em vez disso, vinculam os valores a uma variável de "controle". Se variarmos o valor de *t* , a cada alteração obtemos **dois** valores, que podemos usar como coordenadas ( *x* , *y* ) em um gráfico. O conjunto de funções acima, por exemplo, gera pontos em um círculo: Podemos variar *t* de negativo a infinito positivo, e as coordenadas resultantes ( *x* , *y* ) sempre estarão em um círculo com raio 1 ao redor da origem (0,0 ).

utilizando a biblioteca Skia no RN, em conjunto com a função acima, ficaria assim:

```jsx
import {Canvas, Circle} from '@shopify/react-native-skia';
import {useEffect, useState} from 'react';

const SCALE = 120;
const points = 20;

type TPositions = {
  x: number;
  y: number;
};

export function Board() {
  const [positions, setPositions] = useState<TPositions[]>([]);

  useEffect(() => {
    const newPositions = [];
    for (let t = 0; t <= points; t += 1) {
      const x = Math.cos(t);
      const y = Math.sin(t);

      newPositions.push({x: x * SCALE + SCALE, y: y * SCALE + SCALE});
    }
    setPositions(newPositions);
  }, []);

  return (
    <Canvas style={{flex: 1, backgroundColor: '#22c25f'}}>
      {/* <Path path={path} color="black" strokeWidth={2} /> */}
      {positions.map((item, index) => {
        return (
          <Circle color={'#fff'} key={index} r={2} cx={item.x} cy={item.y} />
        );
      })}
    </Canvas>
  );
}
```

O resultado seria esse: 

![Simulator Screen Shot - iPhone SE (3rd generation) - 2023-03-31 at 17.02.34.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fed0873aa-1e24-40dc-af8c-d5d2556ef632%2FSimulator_Screen_Shot_-_iPhone_SE_(3rd_generation)_-_2023-03-31_at_17.02.34.png?id=7e71511d-3162-4478-999d-582e14e21a98&table=block&spaceId=f9ac00f4-8056-4026-8eda-a70fb5c0c917&width=2000&userId=5c63ee33-b7c0-4ad2-8d91-e3b691ec3b52&cache=v2)

As curvas de Bézier são apenas uma das muitas classes de funções paramétricas e são caracterizadas pelo uso da mesma função base para todos os valores de saída. No exemplo que vimos acima, os valores *x* e *y* foram gerados por diferentes funções (uma usa um seno, a outra um cosseno); mas as curvas de Bézier usam o "polinômio binomial" para as saídas *x* e *y*. Então, o que são polinômios binomiais?

Você pode se lembrar de polinômios do ensino médio. São aquelas somas que se parecem com isso:

![https://pomax.github.io/bezierinfo/images/chapters/explanation/855a34c7f72733be6529c3fb33fa1a23.svg](https://pomax.github.io/bezierinfo/images/chapters/explanation/855a34c7f72733be6529c3fb33fa1a23.svg)

Se o termo de ordem mais alta que eles têm é *x3*, eles são chamados de polinômios "cúbicos"; se for *x2,* é um polinômio "quadrado"; se for apenas *x*, é uma linha (e se não houver nem mesmo termos com *x*, não é um polinômio!)

As curvas de Bézier são polinômios de *t*, em vez de *x*, com o valor de *t* sendo fixo entre 0 e 1, com coeficientes *a*, *b* etc. tomando a forma "binomial", o que parece extravagante, mas na verdade é uma descrição bastante simples para misturar valores:

![https://pomax.github.io/bezierinfo/images/chapters/explanation/7f74178029422a35267fd033b392fe4c.svg](https://pomax.github.io/bezierinfo/images/chapters/explanation/7f74178029422a35267fd033b392fe4c.svg)

Eu sei o que você está pensando: isso não parece muito simples! Mas se removermos *t* 
e adicionarmos "times one", as coisas de repente parecem muito fáceis. Confira estes termos binomiais:

![https://pomax.github.io/bezierinfo/images/chapters/explanation/af40980136c291814e8970dc2a3d8e63.svg](https://pomax.github.io/bezierinfo/images/chapters/explanation/af40980136c291814e8970dc2a3d8e63.svg)

Observe que 2 é o mesmo que 1+1, e 3 é 2+1 e 1+2, e 6 é 3+3... Como você pode ver, cada vez que sobimos uma dimensão, simplesmente começamos e terminamos com 1, e tudo no meio é apenas "os dois números acima dele, somados", nos dando uma sequência numérica simples conhecida como [triângulo de Pascal](https://en.wikipedia.org/wiki/Pascal%27s_triangle). Agora *isso é* fácil de lembrar.

![https://upload.wikimedia.org/wikipedia/commons/0/0d/PascalTriangleAnimated2.gif](https://upload.wikimedia.org/wikipedia/commons/0/0d/PascalTriangleAnimated2.gif)

Há uma maneira igualmente simples de descobrir como os termos polinomiais funcionam: se renomearmos *(1-t)* para *a* e *t* para *b*, e removermos os pesos por um momento, teremos isso:

![https://pomax.github.io/bezierinfo/images/chapters/explanation/4319b2e361960c842a4308a610a35048.svg](https://pomax.github.io/bezierinfo/images/chapters/explanation/4319b2e361960c842a4308a610a35048.svg)

É basicamente apenas uma soma de "cada combinação de *a* e *b*", substituindo progressivamente *a's* por *b's* após cada sinal +. Então, na verdade, isso também é bem simples. Então, agora você conhece polinômios binomiais, e apenas para completar, vou mostrar a função genérica para isso:

![https://pomax.github.io/bezierinfo/images/chapters/explanation/39d33ea94e7527ed221a809ca6054174.svg](https://pomax.github.io/bezierinfo/images/chapters/explanation/39d33ea94e7527ed221a809ca6054174.svg)

E essa é a descrição completa das curvas de Bézier. Σ nesta função indica que esta é uma série de adições (usando a variável listada abaixo do Σ, começando em ...=<valor> e terminando no valor listado no topo do Σ).

Para a maioria dos fins de computação gráfica, não precisamos de curvas arbitrárias (embora também forneçamos código para curvas arbitrárias neste primer); precisamos de curvas quadráticas e cúbicas, e isso significa que podemos simplificar drasticamente o código:

```jsx
function bezier(2,t){
  const t2 = t * t
  const mt = 1-t
  const mt2 = mt * mt
  return mt2 + 2*mt*t + t2
}
function bezier(3,t){
  const t2 = t * t
  const t3 = t2 * t
  const mt = 1-t
  const mt2 = mt * mt
  const mt3 = mt2 * mt
  return mt3 + 3*mt2*t + 3*mt*t2 + t3
}
```

![Captura de Tela 2023-04-01 às 15.51.29.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fbd5713ad-526f-4606-b6db-509b4b9722a4%2FCaptura_de_Tela_2023-04-01_s_15.51.29.png?id=c1626cb5-382f-4109-95cb-13c34b5532e2&table=block&spaceId=f9ac00f4-8056-4026-8eda-a70fb5c0c917&width=2000&userId=5c63ee33-b7c0-4ad2-8d91-e3b691ec3b52&cache=v2)

E agora sabemos como programar a função base. Excelente.

As curvas de Bézier são, como todas as "splines", funções de interpolação. Isso significa que eles pegam um conjunto de pontos e geram valores em algum lugar "entre" esses pontos. (Uma das consequências disso é que você nunca será capaz de gerar um ponto que esteja fora do contorno para os pontos de controle, comumente chamado de "casco" para a curva. Informações úteis!). Na verdade, podemos visualizar como cada ponto contribui para o valor gerado pela função, para que possamos ver quais pontos são importantes, onde, na curva.

Os gráficos a seguir mostram as funções de interpolação para curvas quadráticas e cúbicas, com "S" sendo a força da contribuição de um ponto para a soma total da função de Bézier. Clique e arraste para ver as porcentagens de interpolação para cada ponto definidor de curva em um valor *t* específico.

Os gráficos a seguir mostram as funções de interpolação para curvas quadráticas e cúbicas, com "S" sendo a força da contribuição de um ponto para a soma total da função de Bézier. Clique e arraste para ver as porcentagens de interpolação para cada ponto definidor de curva em um valor *t* específico.

Também é mostrada a função de interpolação para uma função Bézier de 15a ordem. Como você pode ver, o ponto inicial e final contribui consideravelmente mais para a forma da curva do que qualquer outro ponto no conjunto de pontos de controle.

Se quisermos mudar a curva, precisamos mudar os pesos de cada ponto, alterando efetivamente as interpolações. A maneira de fazer isso é o mais simples possível: basta multiplicar cada ponto por um valor que mude sua força. Esses valores são convencionalmente chamados de "pesos", e podemos adicioná-los à nossa função Bézier original:

![https://pomax.github.io/bezierinfo/images/chapters/control/c2f2fe0ef5d0089d9dd8e5e3999405cb.svg](https://pomax.github.io/bezierinfo/images/chapters/control/c2f2fe0ef5d0089d9dd8e5e3999405cb.svg)

Isso parece complicado, mas acontece que os "pesos" são na verdade apenas os valores de coordenadas que queremos que nossa curva tenha: para uma curva *de enésima* ordem, w0 é nossa coordenada inicial, wn é nossa última coordenada e tudo no meio é uma coordenada controladora. Digamos que queremos uma curva cúbica que comece em (110.150), seja controlada por (25.190) e (210.250) e termine em (210,30), usamos esta curva de Bézier:

![https://pomax.github.io/bezierinfo/images/chapters/control/be73034ac382b54863c7a18c2932cbbc.svg](https://pomax.github.io/bezierinfo/images/chapters/control/be73034ac382b54863c7a18c2932cbbc.svg)

![Captura de Tela 2023-04-01 às 15.55.41.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fcc9e1676-675b-4f02-96dc-c6bac188dcf8%2FCaptura_de_Tela_2023-04-01_s_15.55.41.png?id=217877af-4ded-4c11-8993-cb53c4cc97d6&table=block&spaceId=f9ac00f4-8056-4026-8eda-a70fb5c0c917&width=2000&userId=5c63ee33-b7c0-4ad2-8d91-e3b691ec3b52&cache=v2)

Para adicionar os pontos de controle as nossas funções de base, basta:

```jsx
export function quadraticBezier(t: number){
  const t2 = t * t;
  const mt = 1-t;
  const mt2 = mt * mt;
  return mt2 + 2*mt*t + t2;
}
export function cubicBezier(t: number, w: number[]) {
    const t2 = t * t;
    const t3 = t2 * t;
    const mt = 1 - t;
    const mt2 = mt * mt;
    const mt3 = mt2 * mt;
    return w[0] * mt3 + 3 * w[1] * mt2 * t + 3 * w[2] * mt * t2 + w[3] * t3;
}
```

E agora sabemos como programar a função de base ponderada.

O exemplo acima pode ser gerado no lado do React Nativa graças ao nosso querido Skia, eu utilizei um exemplo posicionando um circulo nas coordenadas geradas, mas por questão de performance, conseguimos melhorar isso: 

```jsx
import { useEffect, useState } from 'react';
import { cubicBezier} from './cubicBezier';

const SCALE = 20;

type TPositions = {
    x: number;
    y: number;
};

export function useExampleWeight1() {
    const [positions, setPositions] = useState<TPositions[]>([]);
    useEffect(() => {
        const newPositions = [];
        for (let t = 0; t <= 1; t += 0.01) {
            const x = cubicBezier(t, [110, 25, 210, 210]);
            const y = cubicBezier(t, [150, 190, 250, 30]);

            newPositions.push({ x: x + SCALE, y: y + SCALE });
        }
        setPositions(newPositions);
    }, []);

    return { positions };
}
```

```jsx
import {Canvas, Circle} from '@shopify/react-native-skia';
import {useExampleWeight1} from '../../math/polynomials/useExamples';

export function Board() {
  // const {positions} = useSincos();
  const {positions} = useExampleWeight1();

  return (
    <Canvas style={{flex: 1, backgroundColor: '#22c25f'}}>
      {positions.map((item, index) => {
        return (
          <Circle color={'#fff'} key={index} r={2} cx={item.x} cy={item.y} />
        );
      })}
    </Canvas>
  );
}
```

O resultado do código acima será parecido com esse:

![Simulator Screen Shot - iPhone SE (3rd generation) - 2023-04-01 at 16.03.30.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F3ac0b162-f6ac-402e-8429-06529f6d7077%2FSimulator_Screen_Shot_-_iPhone_SE_(3rd_generation)_-_2023-04-01_at_16.03.30.png?id=a8a8f519-2ad1-4c79-b40e-fd47b235bacd&table=block&spaceId=f9ac00f4-8056-4026-8eda-a70fb5c0c917&width=2000&userId=5c63ee33-b7c0-4ad2-8d91-e3b691ec3b52&cache=v2)

Claro, quanto mais pontos gerarmos com a equação mais o elemento será visível.

Podemos controlar ainda mais as curvas de Bézier "racionalizando" elas: ou seja, adicionando um valor de "razão" além do valor de peso discutido na seção anterior, ganhando assim o controle sobre "quão fortemente" cada coordenada influencia a curva.

Adicionar esses valores de proporção à função regular da curva de Bézier é bastante fácil. Onde a função regular é a seguinte:

![https://pomax.github.io/bezierinfo/images/chapters/weightcontrol/ceac4259d2aed0767c7765d2237ca1a3.svg](https://pomax.github.io/bezierinfo/images/chapters/weightcontrol/ceac4259d2aed0767c7765d2237ca1a3.svg)

A função para curvas de Bézier racionais tem mais dois termos:

![https://pomax.github.io/bezierinfo/images/chapters/weightcontrol/942e3b3cacc7f403ad95fcd4acce7d19.svg](https://pomax.github.io/bezierinfo/images/chapters/weightcontrol/942e3b3cacc7f403ad95fcd4acce7d19.svg)

No entanto, o segundo novo termo é o que faz a diferença: cada ponto na curva não é apenas um ponto "duplamente ponderado", é uma *fração* do valor "duplamente ponderado" que calculamos introduzindo essa proporção. Ao calcular pontos na curva, calculamos o valor "normal" de Bézier e depois *dividimos* isso pelo valor de Bézier para a curva que usa apenas proporções, não pesos.

Isso faz algo inesperado: transforma nosso polinômio em algo que *não é* mais um polinômio. Agora é um tipo de curva que é uma super classe dos polinômios, e pode fazer algumas coisas muito legais que as curvas de Bézier não podem fazer "por conta própria", como descrever perfeitamente os círculos (o que veremos em uma seção posterior é literalmente impossível usar as curvas padrão de Bézier).

Mas a melhor maneira de mostrar o que isso faz é fazer literalmente isso: vamos olhar para o efeito de "racionalizar" nossas curvas de Bézier usando um gráfico interativo para curvas racionadas. O gráfico a seguir mostra a curva de Bézier da seção anterior, "enriquecida" com fatores de proporção para cada coordenada. Quanto mais perto de zero definimos um ou mais termos, menos influência relativa a coordenada associada exerce na curva (e, claro, quanto maior os definimos, mais influência eles têm).

![Captura de Tela 2023-04-01 às 16.21.45.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F21833539-bb07-4857-b075-5cbf09144e1c%2FCaptura_de_Tela_2023-04-01_s_16.21.45.png?id=3b81a017-c64b-4fb7-8b33-fddc98ab943c&table=block&spaceId=f9ac00f4-8056-4026-8eda-a70fb5c0c917&width=2000&userId=5c63ee33-b7c0-4ad2-8d91-e3b691ec3b52&cache=v2)

Você pode pensar nos valores da razão como a "gravidade" de cada coordenada: quanto maior a gravidade, mais próxima dessa coordenada a curva vai querer estar. Você também notará que, se você simplesmente aumentar ou diminuir todas as proporções pela mesma quantidade, nada muda... muito parecido com a gravidade, se os pontos fortes relativos permanecerem os mesmos, nada realmente muda. Os valores definem a influência de cada coordenada *em relação a todos os outros pontos*.

Estender o código da seção anterior para incluir proporções é quase trivial:

```jsx
export function rationalQuadraticBezier(t: number, w: number[], r: number[]){
  const t2 = t * t;
  const mt = 1-t;
  const mt2 = mt * mt;
  const f = [
    r[0] * mt2,
    2 * r[1] * mt * t,
    r[2] * t2,
  ];
  const basis = f[0] + f[1] + f[2];
  return (f[0] * w[0] + f[1] * w[1] + f[2] * w[2])/basis;
}
export function rationalCubicBezier(t: number, w: number[], r: number[]) {
    const t2 = t * t;
    const t3 = t2 * t;
    const mt = 1 - t;
    const mt2 = mt * mt;
    const mt3 = mt2 * mt;
    const f = [
        r[0] * mt3,
        3 * r[1] * mt2 * t,
        3 * r[2] * mt * t2,
        r[3] * t3,
    ];
    const basis = f[0] + f[1] + f[2] + f[3];
    return (f[0] * w[0] + f[1] * w[1] + f[2] * w[2] + f[3] * w[3]) / basis;
}
```

em React Native:

```jsx
import { useEffect, useState } from 'react';
import { rationalCubicBezier } from './rationalBezier';

const SCALE = 20;

type TPositions = {
    x: number;
    y: number;
};

export function useExampleWeight1() {
    const [positions, setPositions] = useState<TPositions[]>([]);
    useEffect(() => {
        const newPositions = [];
        for (let t = 0; t <= 1; t += 0.01) {
            const x = rationalCubicBezier(t, [110, 25, 210, 210], [2, 1, 1, 1]);
            const y = rationalCubicBezier(t, [150, 190, 250, 30], [2, 1, 1, 1]);

            newPositions.push({ x: x + SCALE, y: y + SCALE });
        }
        setPositions(newPositions);
    }, []);

    return { positions };
}
```

```jsx
import {Canvas, Circle} from '@shopify/react-native-skia';
import {useExampleWeight1} from '../../math/polynomials/useExamples';

export function Board() {
  // const {positions} = useSincos();
  const {positions} = useExampleWeight1();

  return (
    <Canvas style={{flex: 1, backgroundColor: '#22c25f'}}>
      {positions.map((item, index) => {
        return (
          <Circle color={'#fff'} key={index} r={2} cx={item.x} cy={item.y} />
        );
      })}
    </Canvas>
  );
}
```
