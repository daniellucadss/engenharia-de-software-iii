# Princípio de Substituição de Liskov
**1. Em um sistema bancário temos as seguintes classes:**
<details>
<summary>🔽 Clique aqui para exibir o código 🔽</summary>

~~~Java
public class ContaBancaria {
    private double saldo;

    public ContaBancaria(double saldoInicial) {
        this.saldo = saldoInicial;
    }

    public void depositar(double valor) { saldo += valor; }
    public void sacar(double valor) { saldo -= valor; }
}

public class ContaPoupanca extends ContaBancaria {
    public ContaPoupanca(double saldoInicial) {
        super(saldoInicial);
    }

    @Override
    public void sacar(double valor) {
        if (valor > 1000) {
            throw new
                RuntimeException("Não pode sacar mais de 1000 em uma poupança");
        }
        super.sacar(valor);
    }
}
~~~
</details>

**Explique por que motivo a herança entre as classes não é justificada e proponha uma solução.**
- A herança entre as classes ContaBancaria e ContaPoupanca viola o Princípio de Substituição de Liskov porque a classe ContaPoupanca altera o comportamento do método sacar da classe ContaBancaria, existindo uma verificação adicional nesse método.

A solução seria remover a herança e usar composição em vez disso: 

~~~Java
public interface Conta {
    void depositar(double valor);
    void sacar(double valor);
}

public class ContaBancaria implements Conta {
    private double saldo;

    public ContaBancaria(double saldoInicial) {
        this.saldo = saldoInicial;
    }

    public void depositar(double valor) { saldo += valor; }
    public void sacar(double valor) { saldo -= valor; }
}

public class ContaPoupanca implements Conta {
    private double saldo;

    public ContaPoupanca(double saldoInicial) {
        this.saldo = saldoInicial;
    }

    public void depositar(double valor) { saldo += valor; }
    public void sacar(double valor) {
        if (valor > 1000) {
            throw new RuntimeException("Não pode sacar mais de 1000 em uma poupança");
        }
        saldo -= valor;
    }
}
~~~

**2. Proponha uma solução que evite o uso da herança no código abaixo:**

<details>
<summary>🔽 Clique aqui para exibir o código 🔽</summary>

~~~Java
public class Conta {
    private double saldo;
    private String numeroConta;

    public Conta(String numeroConta, double saldoInicial) {
        this.numeroConta = numeroConta;
        this.saldo = saldoInicial;
    }

    public void depositar(double valor) {
        saldo += valor;
    }

    public void sacar(double valor) {
        if (valor > saldo) {
            throw new IllegalArgumentException("Saldo insuficiente.");
        }
        saldo -= valor;
    }

    // Outros métodos relevantes...
}

public class ContaCliente extends Conta {
    private String nome;
    private String cpf;
    private String endereco;

    public ContaCliente(String numeroConta, double saldoInicial, String nome, String cpf, String endereco) {
        super(numeroConta, saldoInicial);
        this.nome = nome;
        this.cpf = cpf;
        this.endereco = endereco;
    }

    // Métodos específicos do cliente...
}
~~~
</details>

- ContaCliente não é um tipo de Conta. Dessa forma podemos usar uma composição como uma possível solução:

~~~Java
public class Conta {
    private double saldo;
    private String numeroConta;

    public Conta(String numeroConta, double saldoInicial) {
        this.numeroConta = numeroConta;
        this.saldo = saldoInicial;
    }

    public void depositar(double valor) {
        saldo += valor;
    }

    public void sacar(double valor) {
        if (valor > saldo) {
            throw new IllegalArgumentException("Saldo insuficiente.");
        }
        saldo -= valor;
    }

    // Outros métodos relevantes...
}

public class Cliente {
    private String nome;
    private String cpf;
    private String endereco;
    private Conta conta;

    public Cliente(String nome, String cpf, String endereco, String numeroConta, double saldoInicial) {
        this.nome = nome;
        this.cpf = cpf;
        this.endereco = endereco;
        this.conta = new Conta(numeroConta, saldoInicial);
    }

    public Conta getConta() {
        return conta;
    }

    // Métodos específicos do cliente...
}
~~~

**3. Aplique o princípio LSP à implementação abaixo de forma que persistência seja um atributo da segunda classe. Crie um exemplo real com o resultado da refatoração.**

<details>
<summary>🔽 Clique aqui para exibir o código 🔽</summary>

~~~Java
import java.io.*;

public class Persistencia {
    public void salvar(String dados, String arquivo) throws IOException {
        try (FileWriter writer = new FileWriter(arquivo)) {
            writer.write(dados);
        }
    }
}
    
public class PersistenciaJSON extends Persistencia {
    @Override
    public void salvar(String dados, String arquivo) throws IOException {
        if (!dados.startsWith("{")) {
            throw new
                IllegalArgumentException("Dados não estão em formato JSON.");
        }
        super.salvar(dados, arquivo);
    }
}
~~~
</details>

- Solução:
~~~Java
import java.io.*;

public interface Persistencia {
    void salvar(String dados, String arquivo) throws IOException;
}

public class PersistenciaArquivo implements Persistencia {
    public void salvar(String dados, String arquivo) throws IOException {
        try (FileWriter writer = new FileWriter(arquivo)) {
            writer.write(dados);
        }
    }
}

public class PersistenciaJSON implements Persistencia {
    private Persistencia persistencia;

    public PersistenciaJSON(Persistencia persistencia) {
        this.persistencia = persistencia;
    }

    public void salvar(String dados, String arquivo) throws IOException {
        if (!dados.startsWith("{")) {
            throw new IllegalArgumentException("Dados não estão em formato JSON.");
        }
        persistencia.salvar(dados, arquivo);
    }
}
~~~

~~~Java
public class Main {
    public static void main(String[] args) throws IOException {
        Persistencia persistenciaArquivo = new PersistenciaArquivo();
        Persistencia persistenciaJSON = new PersistenciaJSON(persistenciaArquivo);

        persistenciaJSON.salvar("{\"nome\":\"Daniel\"}", "dados.json");
    }
}
~~~

**4. Entenda o problema da herança entre patos e suas capacidades presentes no exemplo do link:**

https://www.quora.com/What-are-some-Java-examples-for-the-OOP-principle-of-favoring-object-composition-over-inheritance

**Proponha um exemplo que também viola o LSP e exiba sua solução.**

~~~Java
public class Retangulo {
    private double altura;
    private double largura;

    public void setAltura(double altura) {
        this.altura = altura;
    }

    public void setLargura(double largura) {
        this.largura = largura;
    }

    public double getArea() {
        return altura * largura;
    }
}

public class Quadrado extends Retangulo {
    @Override
    public void setAltura(double altura) {
        super.setAltura(altura);
        super.setLargura(altura);
    }

    @Override
    public void setLargura(double largura) {
        super.setAltura(largura);
        super.setLargura(largura);
    }
}
~~~

- Solução usando composição:

~~~Java
public interface Forma {
    double getArea();
}

public class Retangulo implements Forma {
    private double altura;
    private double largura;

    public Retangulo(double altura, double largura) {
        this.altura = altura;
        this.largura = largura;
    }

    public double getArea() {
        return altura * largura;
    }
}

public class Quadrado implements Forma {
    private double lado;

    public Quadrado(double lado) {
        this.lado = lado;
    }

    public double getArea() {
        return lado * lado;
    }
}
~~~
Solução com base no link a seguir: https://robsoncastilho.com.br/2013/03/21/principios-solid-principio-de-substituicao-de-liskov-lsp/

**5. As classes Postagem, Reacao e Comentario possuem uma herança apenas para aproveitar alguns atributos e reescrever o método exibir().**

<details>
<summary>🔽 Clique aqui para exibir o código 🔽</summary>

~~~Java
public class Perfil {
    private int id;
    private String nomeUsuario;

    public Perfil(int id, String nomeUsuario) {
        this.nomeUsuario = nomeUsuario;
        this.id = id;
    }

    // Outros métodos
}

public class Postagem {
    private String id;
    private Perfil autor;
    private String conteudo;

    public Postagem(String id, Perfil autor, String conteudo) {
        this.id = id;
        this.autor = autor;
        this.conteudo = conteudo;
    }

    public void exibir() {
        System.out.println("Postagem [" + id + "] de " + autor.getNomeUsuario() + ": " + conteudo);
    }

    // outros métodos
}

public class Reacao extends Postagem {
    private String tipoReacao;

    public Reacao(String id, Perfil autor, String tipoReacao) {
        super(id, autor, null);
        this.tipoReacao = tipoReacao;
    }

    @Override
    public void exibir() {
        System.out.println("Reação [" + tipoReacao + "] de " + getAutor().getNomeUsuario() + " na postagem " + getId());
    }

    // outros métodos
}

public class Comentario extends Postagem {
    private Postagem postagemOriginal;

    public Comentario(String id, Perfil autor, String conteudo, Postagem postagemOriginal) {
        super(id, autor, conteudo);
        this.postagemOriginal = postagemOriginal;
    }

    @Override
    public void exibir() {
        System.out.println("Comentário de " + getAutor().getNomeUsuario() + " em resposta a postagem [" + postagemOriginal.getId() + "]: " + conteudo);
    }

    // outros métodos
}
~~~
</details>

**Refatore as classes de forma a:**
**a. Reacao tenha uma composição com postagem;**
**b. Comentário também tenha uma composição com postagem;**
**c. Postagem tenha uma coleção de reações e comentários;**
**d. Todos implementem a interface abaixo:**

<details>
<summary>🔽 Clique aqui para exibir o código 🔽</summary>

~~~Java
public interface Publicavel {
    void exibir();
    Perfil getAutor();
}
~~~
</details>

- Solução:

~~~Java
public interface Publicavel {
    void exibir();
    Perfil getAutor();
}

public class Perfil {
    private int id;
    private String nomeUsuario;

    public Perfil(int id, String nomeUsuario) {
        this.nomeUsuario = nomeUsuario;
        this.id = id;
    }

    // Outros métodos
}

public class Postagem implements Publicavel {
    private String id;
    private Perfil autor;
    private String conteudo;
    private List<Reacao> reacoes;
    private List<Comentario> comentarios;

    public Postagem(String id, Perfil autor, String conteudo) {
        this.id = id;
        this.autor = autor;
        this.conteudo = conteudo;
        this.reacoes = new ArrayList<>();
        this.comentarios = new ArrayList<>();
    }

    public void exibir() {
        System.out.println("Postagem [" + id + "] de " + autor.getNomeUsuario() + ": " + conteudo);
        for (Reacao reacao : reacoes) {
            reacao.exibir();
        }

        for (Comentario comentario : comentarios) {
            comentario.exibir();
        }
    }

    // outros métodos
}

public class Reacao implements Publicavel {
    private String id;
    private Perfil autor;
    private String tipoReacao;
    private Postagem postagem;

    public Reacao(String id, Perfil autor, String tipoReacao, Postagem postagem) {
        this.id = id;
        this.autor = autor;
        this.tipoReacao = tipoReacao;
        this.postagem = postagem;
    }

    public void exibir() {
        System.out.println("Reação [" + tipoReacao + "] de " + autor.getNomeUsuario() + " na postagem " + postagem.getId());
    }

    // outros métodos
}

public class Comentario implements Publicavel {
    private String id;
    private Perfil autor;
    private String conteudo;
    private Postagem postagem;

    public Comentario(String id, Perfil autor, String conteudo, Postagem postagem) {
        this.id = id;
        this.autor = autor;
        this.conteudo = conteudo;
        this.postagem = postageml;
    }

    public void exibir() {
        System.out.println("Comentário de " + autor.getNomeUsuario() + " em resposta a postagem [" + postagem.getId() + "]: " + conteudo);
    }

    // outros métodos
}
~~~
