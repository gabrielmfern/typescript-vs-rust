**AVISO: NÃO LEIA CASO VOCÊ NÃO SUPORTE CENAS FORTES, PORQUE UM 
NOCAUTE ESTÁ ESCRITO A SEGUIR, COM MUITA, MUITA VIOLÊNCIA. O ESCRITOR NÃO SE RESPONSAIBILIZA PELOS EFEITOS COLATERAIS A SEGUIR:**
- Emoções fortes;
- tristeza imensa;
- choques de relidades;
- náuseas sem fim;
- ódio por TypeScript; (😏)
- etc.

# TypeScript vs Rust, em: O PasswordManager

Recentemente eu assisti um vídeo bem interessante mostrando uma certa capacidade do Rust em fazer um PackageManager
e em como pode ser feito de uma forma extremamente segura que evita qualquer tipo de código mal feito em tempo de compilação.

Então pensei: por que não escrever um README simpleszinho comparando o TypeScript com o Rust para mostrar
quão melhor o Rust pode ser para os meus companheiros de trabalho e meu querido pai que ainda nega
aprender Rust? 

Então vem ler que você vai ver.

## O nosso PasswordManager

Imagine que nós temos uma estrutura de dados chamada PasswordManager, (e eu não digo classe por uma razão que vou mostrar logo, logo).

Esse nosso PasswordManager deve conter, de alguma maneira, uma lista de senhas. *Mas essas senhas não podem ser acessíveis sempre*.
Para poder ter acesso a estas senhas o nosso PasswordManager terá de ter dois estados: Locked e Unlocked. No caso do Locked,
não dever ser possível ver as senhas, mas no caso do Unlocked deve sim ser possível.

Além disso, devemos ter algum tipo de método tanto para **trancar** o nosso password quanto para **destrancar**. 
E ambos estes métodos só devem poder ser chamados primeiro, apenas caso o PasswordManager esteja no estado *Locked* para destrancar, e segundo apenas caso esteja no estado *Unlocked* para trancar.

Para complicar um pouco mais as coisas, seria interessante ter um método que mostra a versão do PasswordManager.

## TypeScript

Primeiro vamos começar pela mais péssima e terrível implementação que pode ser feita: *em TypeScript*.

### Criação básica da classe (implementação ingênua)

Como qualquer outra linguagem padrão, *sem graça*, *e sem brilho* que contém Orientação a Objeto, 
precisamos criar uma classe primeiramente:

```typescript
class PasswordManager {
}
```

Dentro dessa classe vamos simplesmente precisar do campo `passwords`, `isLocked` e nosso método `version`:

```typescript
class PasswordManager {
  private passwords: string[] = [];
  private isLocked = true;
  
  version(): string {
    return "typescript é muito ruim kkkk slk tio 🤣";
  }
}
```

A gente vai então precisar dos métodos para dar `lock`, `unlock` e poder listar as passwords:

```typescript
class PasswordManager {
  private passwords: string[] = [];
  private isLocked = true;
  
  version(): string {
    return "typescript é muito ruim kkkk slk tio 🤣";
  }
  
  lock(): void {
    this.isLocked = true;
  }
  
  unlock(): void {
    this.isLocked = false;
  }
  
  listPasswords(): string[] {
    if (this.isLocked === false) {
      return this.passwords;
    }
  }
}
```

### Implementação mais segura

Agora, caso você preste um pouco de atenção no código, pode notar vários problemas nessa implementação, como:
- Pode trancar/destrancar o PasswordManager várias vezes em chamadas, o que pode ser confuso;
- no caso de o PasswordManager estar trancado o `listPasswords` não somente pode ser chamado, como também vai retornar undefined
gerando comportamento indefinido que pode acabar levando à grandes problemas em codebases grandes;

Como você pode ver, o Typescript nos dá maravilhosos problemas que podem acarretar em vários problemas de graça, sem nenhum tipo de erro (a não ser que esteja sendo usado algum tipo de eslint que pode evitar o 2° problema).
Mas aí vocês fanboys de Typescript podem estar gritando atrás de seus teclados prontos para iniciar uma guerra: MAS COM CERTEZA AINDA
DÁ PARA MELHORAR O CÓDIGO EVITANDO ESTES PROBLEMAS!

E a resposta para sua questão é que **sim**. Podemos sim melhorar o código, e nós vamos logo em seguida, mas não se acanhe pequeno fanboy, você ainda será derrotado como o golum no último filme de Senhor dos Anéis com seu querido e precioso objeto caindo em um vulcão na montanha da perdição.

Para melhorar nós podemos simplesmente tratar os casos mencionados pelos problemas lá de cima com o código abaio:

```typescript
class PasswordManager {
  private passwords: string[] = [];
  private isLocked = true;
  
  version(): string {
    return "typescript é muito ruim kkkk slk tio 🤣";
  }
  
  lock(): void {
    if (this.isLocked === true) {
      this.isLocked = true;
    } else {
      throw new Error('😭');
    }
  }
  
  unlock(): void {
    if (this.isLocked === true) {
      this.isLocked = false;
    } else {
      throw new Error('então, por mais que você possa meu caro querido usuário de PassowrdManager destrancar um que já está trancado, não é muito legal fazer isto querido usuário queridamente lhe pesso QUE NÃO FAÇA ISSO, obrigado');
    }
  }
  
  listPasswords(): string[] {
    if (this.isLocked === false) {
      return this.passwords;
    } else {
      throw new Error('ô, bobão, tá trancado o trem aqui dá pra pega as senha não');
    }
  }
}
```

Até aí tudo bem. Mas nós ainda temos uns problemas, que por mais que fanboys ainda queiram, são impossíveis de se solucionar.

### Os problemas insolulávelivelmente insolúveis

Acontece que este código acaba ainda sendo algo que pode muito facilmente prejudicar
uma empresa rodando em produção. "Por quê?" Vocẽ diz? Ah sim, claro, por causa de, principalmente, um simples fato.

Esses erros são erros que acontecem em tempo de execução! Isso é algo impossível de se evitar com TypeScript. Além disso, os erros que podem acontecer não são necessariamente notáveis por um usuário da classe a não ser que esteja claramente documentado e que ele leia a documentação. Algo que, caia entre nós, é bem difícil.

## Rust

Muito provavelmente, você deve estar se perguntando: "Okay, mas então como que diabos que o Rust resolve isso? Como que é possível evitar esse tipo de problema?". E a resposta é simples, ela consiste em apenas uma palavra: **compilação**.

O que compilação tem haver? Bom, o erro só tem duas opções de onde pode ocorrer: compilação ou execução. O melhor momento, afim de não prejudicar a empresa depois, é em tempo de compilação, e sim, isso é possível em Rust.

Sem mais delongas, depois de 152 linhas de Markdown eu lhe apresento o código final de Rust que evita todos estes problemas mencionados até agora:

```rust
pub struct Locked;
pub struct Unlocked;

pub struct PasswordManager<State = Locked> {
    passwords: Vec<String>,
    state: PhantomData<State>,
}

impl PasswordManager {
    pub fn new(passwords: Vec<&str>) -> PasswordManager<Locked> {
        return PasswordManager {
            passwords: passwords.iter()
                .map(|p| p.to_string())
                .collect(),
            state: PhantomData,
        };
    }

    pub fn version(&self) -> &'static str {
        return "";
    }
}

impl PasswordManager<Locked> {
    pub fn unlock(self) -> PasswordManager<Unlocked> {
        return PasswordManager {
            passwords: self.passwords.clone(),
            state: PhantomData,
        };
    }
}

impl PasswordManager<Unlocked> {
    pub fn lock(self) -> PasswordManager<Locked> {
        return PasswordManager {
            passwords: self.passwords.clone(),
            state: PhantomData
        };
    }

    pub fn list_passwords(&self) -> &Vec<String> {
        return &self.passwords;
    }
}
```

Eu sei que parece demais, eu sei. Mas não se preocupe que eu vou lhe explicar cada parte.

### Traits e Structs vs Classes

Primeiro eu quero lhe apresentar o que são traits e structs em Rust. O rust implementa uma forma de tratar os dados
semelhante ao C mas com algo há mais: **traits**.

No Rust, as structs são simplesmente uma definição de como um certo dado em questão vai ser definido, com tipos concretos, e outras coisas.
Então, para poder ter certo comportamento para a nossa estrutura de dados, o Rust tem o que chamamos de **implementações**. Ou **impl** como está escrito. Essa definição no Rust, diferente do TypeScript, permite que a gente possa ter um certo comportamento que é possível **apenas** para certo parâmetro de tipo.

E isso nos leva às traits. Elas são como interfaces, mas nunca confunda elas com interfaces, elas **apenas** definem comportamento de uma estrutura. Uma trait pode, por exemplo, definir se uma certa estrutura é somável com outra, como por exemplo:

```Rust
use std::ops::Add;

struct Number {
    value: f32,
}

impl Add for Number {
    type Output = Self;

    fn add(self, rhs: Self) -> Self::Output {
        Number {
            value: self.value + rhs.value
        }
    }
}
```

Esse `Add` é a trait que define exatamente isso, fazendo com que caso nós tenhamos dois Number, nós podemos simplesmente fazer
`a + b` que o Rust simplesmente vai entender o que você quer no final. Isso é semelhante como o Python permite fazer algo parecido, 
mas de uma forma muito mais rigorosa e segura do que ele.

### PhnatomData, otimização de memória e referências

Agora, uma das partes mais interessantes do Rust que por causa dela o Rust acaba sendo mais diferenciado entre todas as linguages.

#### O Borrow Checker e as referências

Uma referência no Rust, geralmente denotada com **&**, nunca pode ser confundida com o que pode ser parecido no C e em linguagens parecidas: um pointer.

Uma referência na verdade é algo que o Rust mantém internamente, com um objetivo de poder saber até quando certa variável, ou certo dado na
memória RAM do computador, vai ser utilizado para que então ele possa remover ela de lá. Isso faz com que o Rust seja extremamente rápido, e é isso que a gente geralmente chama de **Borrow Checker**.

Com o Borrow Checker, nós podemos ter toda a velocidade do C, com toda a simplicidade de linguagens como Python. Algo incrível que nunca
existiu até recentemente.

#### O gigante PhantomData

Ao contrário do header dessa seção, o PhantomData na verdade é bem simples. O que acontece é que o Rust **precisa** que nós usemos
o parâmetro de tipo definido como `State`, e então pra evitar que a gente tenha de adicionar um variável que fique ocupando memória RAM, nós precisamos de um PhantomData. 

No final, o que ele faz é simplesmente utilizar aquele tipo sem ter nenhum tipo de efeito na quantidade de memória RAM usada pelo programa.

### Mas então como tudo isso faz com que esse código em Rust seja melhor?

Okay, okay, eu sei que eu passei umas 50 linhas de markdown só explicando certas característica do Rust para que você entenda, mas agora eu vou lhe mostrar o que essa nova implementação acarreta apenas mostrando como seria utilizar aquela estrutura.

Vamos então, criar um novo PasswordManager e ver se ele passa por todas as nossas exigências. Primeiro precismaos criar o PasswordManager:

```rust
fn main() {
  let password_manager: PasswordManager<Locked> = PasswordManager::new(vec![
      "minha super senha",
      "123456",
      "agrofresh é o máximo",
  ]);
}
```

Agora vamos ver se nós podemos trancar denovo esse PasswordManager trancado:

```rust
fn main() {   
  let password_manager: PasswordManager<Locked> = PasswordManager::new(vec![
      "minha super senha",
      "123456",
      "agrofresh é o máximo",
  ]);

  // aqui acontece um erro de compilação!
  password_manager.lock(); // <-- method `lock` not found for this struct
}
```

O que evita que nós possamos fazer qualquer código confuso, resolvendo um dos problemas do TypeScript.

Agora vamos tentar destrancar o nosso password_manager, já isso, nós vamos precisar fazer um pouco diferente do TypeScript:


```rust
fn main() {   
  let mut password_manager = PasswordManager::new(vec![
    "minha super senha",
    "123456",
    "agrofresh é o máximo",
  ]);

  let password_manager = password_manager.unlock();
  password_manager.list_passwords();
}
```

Já isso aqui, compila normalmente, como é um código válido, agora nós podemos até tentar dar unlock denovo e a gente vai ter um erro igual antes:

```rust
fn main() {   
  let mut password_manager = PasswordManager::new(vec![
    "minha super senha",
    "123456",
    "agrofresh é o máximo",
  ]);

  let password_manager = password_manager.unlock();
  password_manager.list_passwords();
  let password_manager = password_manager.unlock(); // <-- no method named `unlock` found for struct `PasswordManager<Unlocked>` in the current scope
  password_manager.list_passwords();
}
```

Isso acaba fazendo com que caso haja algum tipo de confusão da parte de um desenvolvedor usando a nossa API, no final, dê erro antes de chegar em produção e até homologação. E no final, o Rust ganha. 
