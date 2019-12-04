# Clean Coding
### Um resumo sobre boas práticas de programação.
- O que é Clean Coding?
  - Clean Coding é uma filosofia de desenvolvimento onde o principal objetivo é aplicar técnicas para que um código fique fácil de ler e de entender.

*“Qualquer um consegue escrever código que um computador entende. Bons programadores escrevem código que humanos entendem”* — Martin Fowler.

- Porque nós nos deparamos com tanto código "feio"?
  - Os requisitos mudaram de forma expressiva
  - "Estou sendo pressionado pra entregar mais rápido"
  - "Vou fazer funcionar o básico, depois melhoro o resto"

---
## Nomenclaturas expressivas
- Uma variável, uma classe ou um método precisa ter um nome que tenha um siginificado se referindo ao seu escopo e para melhor referência.
- **Não tem problema o nome ficar grande**

`ruim:`
```kotlin
val indice = 0

// isso era muito frequente em nossos projetos mais antigos, 
// só combinava com o método que estava sendo chamado anteriormente, fora isso, não dá pra saber o que tá acontecendo.
fun onTalCoisaSuccess(...) {
    // mostra um dialog de sucesso e sai da tela
}

// aqui, olhando para o nome do método, a gente não sabe o que ele retorna e nem o que ele faz.
fun checar(valor: Int): Boolean = valor > 10

// dados do que?
data class Dados(
    val nome: String,
    val idade: Int,
    val endereco: String
)

interface BuscaPraMim {
    fun busca(): Usuario
}
```

`bom:`
```kotlin
val indiceSelecionado = 0

fun mostraMeuDialogEntaoSaiaDaTela() {
    /* */
}

fun valorÉMaiorDoQueOMinimo(valor: Int): Boolean = valor > VALOR_MINIMO // 10

data class DadosDoUsuario(
    val nome: String,
    val idade: Int,
    val endereco: String
)

interface BuscaMeuUsuario {
    fun buscaMeuUsuario(): Usuario
}
```

---
## SOLID
- SOLID é o acrônimo de cinco princípios da programação orientada a objeto e design de código. Ele nos ajuda a manter o nosso código limpo.

---
### Single Responsibility Principle (Princípio da Responsabilidade Única)
- Uma classe deve ter só um motivo para mudar.

`não segue o SRP:`
```kotlin
class UsuarioUtils {

    fun fazLogin() { /* */ }

    fun fazLogout() { /* */ }

    fun buscaUsuarioNoBanco(): Usuario { /* */ }

    fun alteraDadosDoUsuarioNoBanco(usuario: Usuario) { /* */ }

    fun registraUsuarioNoServidor(usuario: Usuario) { /* */ }

    fun deletaUsuarioNoServidor(usuarioId: Long) { /* */ }
}
```

`segue o SRP:`
```kotlin
class LoginManager {

    fun fazLogin()

    fun fazLogout()
}

class DatabaseDoUsuario {

    fun buscaUsuarioNoBanco(): Usuario { /* */ }

    fun alteraDadosDoUsuarioNoBanco(usuario: Usuario) { /* */ }
}

class ServidorDoUsuario {

    fun registraUsuarioNoServidor(usuario: Usuario) { /* */ }

    fun deletaUsuarioNoServidor(usuarioId: Long) { /* */ }
}
```

---
### Open-Closed Principle (Princípio do Aberto-Fechado)
- Abertos para extensão, fechados para edição

`não segue o OCP:`
```kotlin
class Questao(
    val multiplasEscolhas: List<Escolha>? = null,
    val escolhaUnica: Escolha? = null
    val texto: String? = null
)

class AnalisadorDeQuestoes {

    fun analisaMinhasQuestoes(questoes: List<Questao>) {
        for (questao in questoes) {
            analisaQuestao(questao)
        }
    }

    fun analisaQuestao(questao: Questao) {
        when (questao) {
            multpiplasEscolhas != null -> analisaMultiplaEscolha(questao)
            escolhaUnica != null -> analisaEscolhaUnica(questao)
            texto != null -> analisaTexto(questao)
        }
    }

    /* */
}
```

- E se precisarmos adicionar mais um tipo de questão? Ou remover um?

`segue o OCP:`
```kotlin
interface Questao {
    fun analisar()
}

class QuestaoDeMultiplaEscolha(
    val multiplasEscolhas: List<Escolha>
) : Questao {

    fun analisar() {
        /* analisar a propriedade multiplasEscolhas */
    }
}

class QuestaoDeEscolhaUnica(
    val escolhaUnica: Escolha
) : Questao {

    fun analisar() {
        /* analisar a propriedade escolhaUnica */
    }
}

class QuestaoDeTexto(
    val texto: String
) : Questao {

    fun analisar() {
        /* analisar a propriedade texto */
    }
}

class AnalisadorDeQuestoes {

    fun analisaMinhasQuestoes(questoes: List<Questao>) {
        for (questao in questoes) {
            questao.analisar()
        }
    }
}
```

---
### Liskov Substitution Principle (Princípio de Substituição de Liskov)


---
### Interface Segregation Principle (Princípio da Segregação de Interface)
- Criação de interfaces mais específicas e não tão genéricas.

`não segue o ISP:`
```kotlin
interface AbstracaoDeUsuarioUtils {
    fun fazLogin()
    fun fazLogout()
    fun buscaUsuarioNoBanco(): Usuario
    fun alteraDadosDoUsuarioNoBanco(usuario: Usuario)
    fun registraUsuarioNoServidor(usuario: Usuario)
    fun deletaUsuarioNoServidor(usuarioId: Long)
}

class MinhaClasseDeChamadas(
    private val usuarioUtils: AbstracaoDeUsuarioUtils
) {

    // até aqui não parece estar ruim, mas a partir do momento que a interface vai ficando maior, quem for usar terá acesso a inúmeros métodos que não são necessários.
    fun buscaMeuUsarioEntaoPrintaEleNoLog() {
        usuarioUtils.buscaUsuarioNoBanco()
    }
}
```

`segue o ISP:`
```kotlin
interface AbstracaoDaDatabaseDoUsuario {
    fun buscaMeuUsuario(): Usuario
    fun alteraDadosDoUsuarioNoBanco(usuario: Usuario)
}
```  

---
### Dependency Inversion Principle (Princípio da Inversão de Dependência)
- Dependa da abstração e não da implementação.

`não segue o DIP:`
```kotlin
class RepositorioSQLite<Tipo> {
    fun buscarTodos(): List<Tipo> = /* busca todos "Tipos" no banco */
}

class RepositorioServidor<Tipo> {
    fun buscarTodos(): List<Tipo> = /* busca todos "Tipos" no servidor */
}

class RepositorioDoUsuario(
    private val repositorio: RepositorioSQLite<Usuario>
) {
    fun buscarTodosOsUsuarios(): List<Usuario> = repositorio.buscarTodos()
}

// na criação do RepositórioDoUsuario:
class MinhaClasse {

    lateinit var repositorioDoUsuario: RespositorioDoUsuario

    fun criarRepositorio() {
        // possível
        val repositorioSQLite = RepositorioSQLite()
        repositorioDoUsuario = RepositorioDoUsuario(repositorioSQLite)

        // não é possível
        val repositorioServidor = RepositorioServidor()
        repositorioDoUsuario = RepositorioDoUsuario(repositorioDoServidor) 
    }
}
```

`segue o DIP:`
```kotlin
interface Repositorio<Tipo> {
    fun buscarTodos(): List<Tipo>
}

class RepositorioSQLite<Tipo>: Repositorio<Tipo> {
    override fun buscarTodos(): List<Tipo> = /* busca todos "Tipos" no banco */
}

class RepositorioServidor<Tipo>: Repositorio<Tipo> {
    override fun buscarTodos(): List<Tipo> = /* busca todos "Tipos" no servidor */
}

class RepositorioDoUsuario(
    private val repositorio: Repositorio<Usuario>
) {
    fun buscarTodosOsUsuarios(): List<Usuario> = repositorio.buscarTodos()
}

// na criação do RepositórioDoUsuario:
class MinhaClasse {

    lateinit var repositorioDoUsuario: RespositorioDoUsuario

    fun criarRepositorio() {
        // possível
        val repositorioSQLite = RepositorioSQLite<Usuario>()
        repositorioDoUsuario = RepositorioDoUsuario(repositorioSQLite)

        // também é possível
        val repositorioServidor = RepositorioServidor<Usuario>()
        repositorioDoUsuario = RepositorioDoUsuario(repositorioDoServidor) 
    }
}
```

---
## Injeção de dependência
- Injeção de dependência é um padrão de projeto reponsável por manter baixo o nível de desacoplamento de um projeto.
- Facilita a realização de testes unitários

`não é injeção de dependência:` 
```kotlin
class MinhaClasse1 {

    private lateinit var meuRepositorio: Repositorio
    private lateinit var meuContexto: Contexto
    private lateinit var meuLogger: Logger

    fun inicializar() {
        meuLogger = AndroidLogger()
        meuContexto = contextoGlobal()
        meuRepositorio = RepositorioServidor(meuLogger, meuContexto)
    }

    /* */
}


class MinhaClasse2 {

    private lateinit var meuRepositorio: Repositorio
    private lateinit var meuContexto: Contexto
    private lateinit var meuLogger: Logger

    fun inicializar() {
        meuContexto = contextoGlobal()
        meuLogger = AndroidLogger(meuContexto)
        meuRepositorio = RepositorioServidor(meuLogger, meuContexto)
    }
    /* */
}
```

`injeção de dependência:`
```kotlin
object Modulo {

    private var logger: Logger? = null
    var meuContexto = contextoGlobal()

    fun pegarInstanciaDoLogger(): Logger {
        if (logger == null) {
            logger = AndroidLogger(meuContexto)
        }
        return logger
    }

    fun pegarRepositorio(): Repositorio =
        RepositoServidor(LOGGER_PADRAO, meuContexto)

    companion object {
        val LOGGER_PADRAO = pegarInstanciaDoLogger()
    }
}

class MinhaClasse1 {

    private lateinit var meuRepositorio: Repositorio

    fun inicializar() {
        meuRepositorio = Modulo.pegarRepositorio()
    }
    /* */
}


class MinhaClasse2 {

    private lateinit var meuRepositorio: Repositorio
    private lateinit var meuLogger: Logger

    fun inicializar() {
        meuLogger = Module.LOGGER_PADRAO
        meuRepositorio = Modulo.pegarRepositorio()
    }
    /* */
}
```