# Princípio de Substituição de Liskov
1. **Por que o uso do nome próprio Liskov?**
- O nome "Liskov" vem de Barbara Liskov, a cientista da computação que introduziu o princípio em 1993, através de um artigo de nome "Family Values: A Behavioral Notion of Subtyping".

2. **Qual a principal imagem relacionada ao princípio e qual a explicação sobre ela?**
- A imagem mais comum associada ao Princípio de Substituição de Liskov é a de um pato real e um pato de borracha. "Se parece com um pato, grasna como um pato, mas precisa de baterias — Provavelmente você está usando a abstração de forma incorreta". De forma geral, a violação acontece quando você espera que a instância de uma classe base possa ser substituída por uma instância da classe derivada sem afetar o comportamento do programa de maneira indesejada.

3. **Cite um exemplo onde a herança pode ser usada de forma conveniente, porém deixa uma impressão de que está sendo mal aplicada.**

```java
class Ave {
    void voar() {
        // código para voar
    }
}

class Pato extends Ave {
    // Patos podem voar, então não viola o LSP.
}

class Pinguim extends Ave {
    // Pinguins não podem voar, então viola o LSP.
}
```

4. **Cite um exemplo onde a herança pode ser usada de forma conveniente, porém deixa futuras expansões comprometidas ou com problemas de design.**
```java
class Funcionario {
    public int calcularSalario() {
        return 10000;
    }

    public int calcularBonus() {
        return 1000;
    }
}

class FuncionarioPermanente extends Funcionario {
    public int calcularSalario() {
        return 20000;
    }
}

class FuncionarioContratual extends Funcionario {
    public int calcularSalario() {
        return 15000;
    }

    public int calcularBonus() {
        // Funcionário Contratual não recebe bônus, então isso viola o LSP.
    }
}
```

5. **Nos exemplos que você citou, a composição seria mais aplicável? Refaça-os.**

```java
interface IAve {
    void voar();
}

class Ave implements IAve {
    public void voar() {
        // código para voar
    }
}

class Pato {
    private IAve ave;

    public Pato(IAve ave) {
        this.ave = ave;
    }

    public void voar() {
        ave.voar();
    }
}

class Pinguim {
    // Pinguins não podem voar.
}
```

```java
interface IFuncionario {
    int calcularSalario();
    int calcularBonus();
}

class Funcionario implements IFuncionario {
    public int calcularSalario() {
        return 10000;
    }

    public int calcularBonus() {
        return 1000;
    }
}

class FuncionarioPermanente {
    private IFuncionario funcionario;

    public FuncionarioPermanente(IFuncionario funcionario) {
        this.funcionario = funcionario;
    }

    public int calcularSalario() {
        return 20000;
    }

    public int calcularBonus() {
        return funcionario.calcularBonus();
    }
}

class FuncionarioContratual {
    private IFuncionario funcionario;

    public FuncionarioContratual(IFuncionario funcionario) {
        this.funcionario = funcionario;
    }

    public int calcularSalario() {
        return 15000;
    }

    // FuncionarioContratual não recebe bônus. Dessa forma, não possui um método calcularBonus.
}
```
