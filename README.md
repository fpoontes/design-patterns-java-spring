# Design Patterns com Java e Spring
# Design Patterns com Java (GoF) e Spring Boot

> Desafio do Bootcamp DIO — **Dos Clássicos (GoF) ao Spring Framework**  
> Implementações com **Java Puro** (Singleton, Strategy, Facade) e uma **API REST** com **Spring Boot** aplicando Strategy + Facade.

![Java](https://img.shields.io/badge/Java-17+-red) ![Spring](https://img.shields.io/badge/Spring_Boot-3.x-brightgreen) ![Maven](https://img.shields.io/badge/Build-Maven-blue) ![DB](https://img.shields.io/badge/DB-H2-informational)

---

## 📚 Conteúdos

- Padrões de Projeto (GoF)
- **Java Puro**: Singleton, Strategy, Facade
- **Spring Boot**:
  - API REST (CRUD mínimo de `Produto`)
  - Strategy (cálculo de desconto)
  - Facade (orquestração de cadastro + notificação)
  - Observação: beans no Spring são **Singleton** por padrão

---

## 🏗 Estrutura do Repositório


├── README.md
├── pom.xml
├── src/
│   ├── main/java/br/com/fpoontes/design_patterns_spring/
│   │   ├── DesignPatternsSpringApplication.java
│   │   ├── model/Produto.java
│   │   ├── repository/ProdutoRepository.java
│   │   ├── service/
│   │   │   ├── ProdutoService.java
│   │   │   ├── NotificacaoService.java
│   │   │   └── desconto/
│   │   │       ├── DescontoStrategy.java
│   │   │       ├── SemDesconto.java
│   │   │       ├── PixDesconto.java
│   │   │       └── BlackFridayDesconto.java
│   │   ├── facade/CadastrarProdutoFacade.java
│   │   └── controller/ProdutoController.java
│   └── main/resources/application.properties
└── java-puro/                      # (opcional, demonstrações em Java puro)
    ├── singleton/SingletonDemo.java
    ├── strategy/StrategyDemo.java
    └── facade/FacadeDemo.java

> **Pacote base do projeto Spring**: `br.com.fpoontes.design_patterns_spring`.  
> Se mover arquivos, ajuste a primeira linha `package ...;` para bater com a pasta.

---

## ✅ Requisitos

- **JDK 17+**
- **Maven 3.9+** (ou `mvnw`/`mvnw.cmd`)
- IDE (IntelliJ IDEA recomendado)

---

## ⚙️ Configuração do Banco (H2 em memória)

`src/main/resources/application.properties`
```properties
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driverClassName=org.h2.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

spring.h2.console.enabled=true
spring.h2.console.path=/h2


Pacote base usado no projeto Spring: br.com.fpoontes.design_patterns_spring.
Se você mover arquivos, ajuste a primeira linha package ...; para bater com a pasta.

## 🚀 Como Rodar

# Clonar
git clone https://github.com/fpoontes/design-patterns-java-spring.git
cd design-patterns-java-spring

# Rodar a API
mvn spring-boot:run
# ou no Windows:
# .\mvnw.cmd spring-boot:run
🧪 Endpoints (API REST)
1) Criar produto (Strategy de desconto opcional)
POST /produtos?strategy={none|pix|blackfriday}
Content-Type: application/json


Body:

{ "nome": "Notebook", "preco": 3000.0 }


Exemplo (PIX 5% OFF):

curl -X POST "http://localhost:8080/produtos?strategy=pix" \
  -H "Content-Type: application/json" \
  -d '{"nome":"Notebook","preco":3000}'


Exemplo (sem desconto):

curl -X POST "http://localhost:8080/produtos" \
  -H "Content-Type: application/json" \
  -d '{"nome":"Cadeira Gamer","preco":1000}'

2) Listar produtos
GET /produtos

curl http://localhost:8080/produtos

🧠 Padrões Implementados
Java Puro (java-puro/)

Singleton: controle de única instância (SingletonDemo.java)

Strategy: troca de algoritmo em tempo de execução (StrategyDemo.java)

Facade: simplificação de subsistemas (FacadeDemo.java)

Spring Boot (API REST)

Strategy: cálculo de desconto no serviço

DescontoStrategy (contrato)

SemDesconto, PixDesconto (5%), BlackFridayDesconto (20%)

Seleção pela query ?strategy=...

Facade: CadastrarProdutoFacade

Orquestra o salvar produto e o envio de notificação

Singleton (Spring): por padrão, beans (@Service, @Component, @Repository, @Controller) são Singleton

🧩 Como Estender
Nova estratégia de desconto

Criar classe em service/desconto/:

@Component
public class CupomDesconto implements DescontoStrategy {
    public String key() { return "cupom10"; }
    public double calcular(double preco) { return preco * 0.90; }
}


Usar: POST /produtos?strategy=cupom10

Trocar H2 por Postgres/MySQL

Ajuste application.properties (URL, usuário, senha)

Inclua dependência do driver no pom.xml

🔗 Links Úteis

Slides do Bootcamp DIO: (adicione aqui o link do seu bootcamp / aula)

GoF em Java (ref.): https://refactoring.guru/pt-br/design-patterns/java

Spring Boot Docs: https://docs.spring.io/spring-boot/

📝 Checklist DIO

 Repositório público no GitHub

 README com: descrição, como rodar, endpoints, padrões

 Java Puro: Singleton / Strategy / Facade (java-puro/)

 Spring: API /produtos com Strategy + Facade

 Prints/GIF (opcional) do H2 e das requisições

 Commits claros

📄 Licença

MIT — use e adapte livremente.


---

## 💾 Como atualizar o README no seu repo

### Opção A — Pelo GitHub (mais simples)
1. Abra o seu repo → clique em **README.md** → **Edit** (lápis).  
2. Apague o conteúdo atual e **cole** o README acima.  
3. **Commit changes**.

### Opção B — Pelo terminal (Git Bash / PowerShell)
> Dentro da pasta do projeto:

```bash
# Windows (Git Bash) / Linux / macOS
cat > README.md << 'EOF'
# (cole AQUI todo o conteúdo do README acima)
EOF

git add README.md
git commit -m "docs: atualiza README com patterns GoF + Spring"
git push
