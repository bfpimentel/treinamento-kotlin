# Características Kotlin
### Um resumo sobre funções de escopo, inferência de tipo, high order functions e inline functions.

---
## Funções de escopo
- Funções de escopo são funções que possuem como papel mudar o escopo de uma variável.
- O kotlin possui 5 funções de escopo:
    - .apply
    - .also
    - .let
    - .run
    - .with

- **.apply** e **.also** retornam o próprio objeto.
- **.let**, **.run** e **.with** retornam a última coisa dentro do seu bloco.

---
### **.apply**
`definição:`
```kotlin
inline fun <TipoEntrada> TipoEntrada.apply(block: TipoEntrada.() -> Unit): TipoEntrada {
    block()
    return this
}
```

`uso:`
```kotlin
fun meRetornaUmaPessoa(): Pessoa {
    return Pessoa().apply {
        nome = "Josias"
        idade = 20
    } 
    // o resultado aqui seria igual se nós construíssemos o objeto através do construtor. 
    // ex: Pessoa(nome = "Josias", idade = 20)
}
```

*No **.apply**, nós não necessitamos nomear o parâmetro dentro do corpo, tudo que for executado já estará dentro da classe que estamos chamando essa função.*

---
### **.also**
`definição:`
```kotlin
inline fun <TipoEntrada> TipoEntrada.also(block: (TipoEntrada) -> Unit): TipoEntrada {
    block(this)
    return this
}
```

`uso:`
```kotlin
fun meRetornaUmaPessoa(): Pessoa {
    return Pessoa().also { minhaPessoa ->
        minhaPessoa.nome = "Josias"
        minhaPessoa.idade = 20
    } 
    // o resultado aqui seria igual se nós construíssemos o objeto através do construtor. 
    // ex: Pessoa(nome = "Josias", idade = 20)
}
```

*No **.also**, ou podemos nomear o parâmetro dentro do corpo, ou podemos utilizar o **it** se não quisermos nomear. Essa é a única diferença da última função.*

---
### **.let**
`definição:`
```kotlin
inline fun <TipoEntrada, TipoRetorno> TipoEntrada.let(
    block: (TipoEntrada) -> TipoRetorno
): TipoRetorno {
    return block(this)
}
```

`uso:`
```kotlin
fun meRetornaUmaString(): String {
    var umNumero = 2

    return umNumero.let { meuNumero ->
        umNumero = meuNumero + 1
        return@let "umaString $umNumero" //output: umaString 3
    }
}
```

---
### **.run**
`definição:`
```kotlin
// essa pode ser chamada sem estar ligada a algum objeto
inline fun <TipoRetorno> run(block: () -> TipoRetorno): TipoRetorno {
    return block()
}

// essa necessita ser chamada através de um objeto
inline fun <TipoEntrada, TipoRetorno> TipoEntrada.run(block: TipoEntrada.() -> TipoRetorno): TipoRetorno {
    return block()
}
```

`uso:`
```kotlin
// sem ligação com objeto
fun meRetornaUmaBebida(): Bebida = run {
    Bebida(nome = "Cerveja", preco = 5.0)
}

// sendo chamada através de um objeto
fun meCriaUmaBebida(nome: String): Bebida = nome.run { 
    Bebida(nome = this, preco = rand()) // 'this' é o 'nome'.
}
```

*Para o run, não é possível nomear o objeto que estará acessível dentro do escopo, ele será sempre tratado como **this**.*

---
### **.with**
`definição:`
```kotlin
inline fun <TipoEntrada, TipoRetorno> with(receiver: TipoEntrada, block: TipoEntrada.() -> TipoRetorno): TipoRetorno {
    return receiver.block()
}
```

`uso:`
```kotlin
fun retorneMeuNome(meuUsuario: Usuario) = with(meuUsuario) {
    this.nome 
    // 'this' sendo o 'meuUsuario'.
    // não seria necessário explicitar o this aqui,
    // poderia ter apenas retornado 'nome'.
}
```

*Assim como para o **run**, o escopo do **with** sempre contará com o **this**, e o objeto não poderá ser nomeado.*

---
## Inferência de tipo
- No Kotlin, é boa prática inferir os tipos de variáveis e funções, quando possível.
- Também é possível, em funções, omitir as chaves { } ao inferir um retorno.

`exemplos sem inferência:`
```kotlin
val saldo: Double = 10.0

fun retornaMeuSaldo(): Double {
    return saldo
}

fun buscaMeusDadosNoServidor(): DadosDoUsuario {
    return database.getUsuarioLogado().let { usuarioLogado ->
        servidor.buscarDadosDoUsuario(usuarioLogado.id)
    }
}
```

`exemplos com inferência:`
```kotlin
val saldo = 10.0

fun retornaMeuSaldo() = saldo
// retorno: Double

fun buscaMeusDadosNoServidor() = database.getUsuarioLogado().let { usuarioLogado ->
    servidor.buscarDadosDoUsuario(usuarioLogado.id)
}
// retorno: DadosDoUsuario
```

---
## High Order Functions
- **High Order Functions**, ou, em português, **Funções de Alta Ordem**, são funções que recebem funções como parâmetros ou funções que retornam outras funções.
- São vulgarmente chamados, por nós, de *callbacks*.
- Podem ser atribuídas a variáveis.

`exemplo:`
```kotlin
fun <TipoEntrada> List<TipoEntrada>.filtrarNaCondicao(
    condicao: (TipoEntrada) -> Boolean
): List<TipoEntrada> {

    val resultado = mutableListOf<T>()
    for (item in this) {
        if (condicao(item)) {
            resultado.add(item)
        }
    }

    return resultado
}
```

`uso:`
```kotlin
fun filtrarMeusUsuariosPorNome(nome: String) =
    // meusUsuarios é do tipo List<Usuario>
    meusUsuarios.filtrarNaCondicao { usuario ->
        usuario.nome.contains(nome)
    }
// essa função retorna uma lista de usuários filtrados pelo nome desejado.
```


- Essas funções são análogas às interfaces funcionais do Java, isto é, interfaces que possuiam apenas um método.

`exemplo:`
```java
// na sua declaração: 
interface Condicao {
    <TipoEntrada> boolean aplicar(final TipoEntrada objParaSerFiltrado);
}

final <TipoEntrada> List<TipoEntrada> filtrarNaCondicao(
    final List<TipoEntrada> lista,
    final Condicao condicao
) {
    final List<TipoEntrada> resultado = new ArrayList<TipoEntrada>()
    for (final TipoEntrada item : lista) {
        if (condicao.aplicar(item)) {
            resultado.add(item);
        }
    }

    return resultado;
}

// no seu uso:
final List<Usuario> filtrarMeusUsuariosPorNome(final String nome) {
    // meusUsuarios é do tipo List<Usuario>
    return filtrarNaCondicao(meusUsuarios, usuario -> {
        usuario.getNome().contains(nome)
    }) 
}
```

---
## Referências
https://medium.com/androiddevelopers/kotlin-demystified-scope-functions-57ca522895b1

https://medium.com/@agrawalsuneet/higher-order-functions-in-kotlin-3d633a86f3d7