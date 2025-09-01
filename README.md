# Design Patterns com Java e Spring
Design Patterns com Java (GoF) e Spring Boot

Projeto do Bootcamp DIO: Dos Clássicos (GoF) ao Spring Framework
Implementações com Java Puro (Singleton, Strategy, Facade) e uma API REST Spring Boot aplicando Strategy e Facade.

📚 Conteúdo

Padrões de Projeto (GoF)

Java Puro: Singleton, Strategy, Facade

Spring:

API REST (CRUD de Produto)

Strategy (cálculo de desconto)

Facade (orquestração de cadastro + notificação)

Observação: beans do Spring são Singleton por padrão

🏗 Estrutura do Repositório
.
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


Pacote base usado no projeto Spring: br.com.fpoontes.design_patterns_spring.
Se você mover arquivos, ajuste a primeira linha package ...; para bater com a pasta.

✅ Requisitos

JDK 17+

Maven 3.9+ (ou use o wrapper mvnw/mvnw.cmd)

IDE (IntelliJ IDEA recomendado)

⚙️ Configuração do Banco (H2 em memória)

src/main/resources/application.properties

spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driverClassName=org.h2.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

spring.h2.console.enabled=true
spring.h2.console.path=/h2


Console H2: http://localhost:8080/h2
JDBC URL: jdbc:h2:mem:db | user: sa | senha: (em branco)

🚀 Como Rodar
Clonar e subir a API
git clone https://github.com/fpoontes/design-patterns-java-spring.git
cd design-patterns-java-spring
mvn spring-boot:run
# ou no Windows:
# .\mvnw.cmd spring-boot:run


A aplicação sobe em: http://localhost:8080

🧪 Endpoints (API REST)
1) Criar produto (com Strategy de desconto opcional)
POST /produtos?strategy={none|pix|blackfriday}
Content-Type: application/json


Body:

{
  "nome": "Notebook",
  "preco": 3000.0
}


Exemplo (PIX 5% OFF via cURL):

curl -X POST "http://localhost:8080/produtos?strategy=pix" \
  -H "Content-Type: application/json" \
  -d '{"nome":"Notebook","preco":3000}'


Exemplo (sem desconto):

curl -X POST "http://localhost:8080/produtos" \
  -H "Content-Type: application/json" \
  -d '{"nome":"Cadeira Gamer","preco":1000}'

2) Listar produtos
GET /produtos


Exemplo:

curl http://localhost:8080/produtos

🧠 Padrões Implementados
Java Puro (pasta java-puro/)

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

🧩 Como Estender (exemplos rápidos)
Nova estratégia de desconto

Criar classe em service/desconto/:

@Component
public class CupomDesconto implements DescontoStrategy {
    public String key() { return "cupom10"; }
    public double calcular(double preco) { return preco * 0.90; }
}


Usar: POST /produtos?strategy=cupom10

Substituir a persistência

Troque H2 por Postgres/MySQL no application.properties

Ajuste a URL do datasource e dependências no pom.xml

🧾 Códigos-chave (recorte)
Produto.java
@Entity
public class Produto {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String nome;
    private double preco;
    // getters/setters/constructors
}

DescontoStrategy.java
public interface DescontoStrategy {
    String key();
    double calcular(double preco);
}

ProdutoService.java
@Service
public class ProdutoService {
    private final ProdutoRepository repository;
    private final Map<String, DescontoStrategy> strategyMap;

    public ProdutoService(ProdutoRepository repository, List<DescontoStrategy> strategies) {
        this.repository = repository;
        this.strategyMap = strategies.stream()
            .collect(Collectors.toMap(DescontoStrategy::key, s -> s));
    }

    public List<Produto> listar() { return repository.findAll(); }

    public Produto salvar(Produto p, String strategyKey) {
        DescontoStrategy s = strategyMap.getOrDefault(
            strategyKey == null ? "none" : strategyKey.toLowerCase(),
            strategyMap.get("none")
        );
        p.setPreco(s.calcular(p.getPreco()));
        return repository.save(p);
    }
}

CadastrarProdutoFacade.java
@Component
public class CadastrarProdutoFacade {
    private final ProdutoService produtoService;
    private final NotificacaoService notificacaoService;

    public CadastrarProdutoFacade(ProdutoService ps, NotificacaoService ns) {
        this.produtoService = ps;
        this.notificacaoService = ns;
    }

    public Produto executar(Produto produto, String strategyKey) {
        Produto salvo = produtoService.salvar(produto, strategyKey);
        notificacaoService.enviar("Produto cadastrado: " + salvo.getNome()
            + " | Preço final: " + salvo.getPreco());
        return salvo;
    }
}

ProdutoController.java
@RestController
@RequestMapping("/produtos")
public class ProdutoController {
    private final ProdutoService produtoService;
    private final CadastrarProdutoFacade facade;

    public ProdutoController(ProdutoService produtoService, CadastrarProdutoFacade facade) {
        this.produtoService = produtoService;
        this.facade = facade;
    }

    @PostMapping
    public Produto salvar(@RequestBody Produto produto,
                          @RequestParam(name = "strategy", defaultValue = "none") String strategy) {
        return facade.executar(produto, strategy);
    }

    @GetMapping
    public List<Produto> listar() { return produtoService.listar(); }
}

📝 Checklist de Entrega (DIO)

 Projeto público no GitHub

 README com: descrição, como rodar, endpoints, padrões utilizados

 Java Puro: Singleton / Strategy / Facade (pasta java-puro/)

 Spring: API /produtos com Strategy + Facade

 Prints/GIF (opcional) do H2 console e requisições

 Commits claros e pom.xml ok

🔗 Links Úteis

Slides / Materiais da DIO (adicione aqui o link do seu bootcamp)

GoF em Java (referência): https://refactoring.guru/pt-br/design-patterns/java

Spring Boot Docs: https://docs.spring.io/spring-boot/

📄 Licença

Este projeto é distribuído sob a licença MIT. Sinta-se livre para usar e adaptar.

Observações (Windows)

Se aparecer aviso LF will be replaced by CRLF, é normal no Windows. Para padronizar, adicione .gitattributes:

* text=auto eol=lf
*.bat text eol=crlf
*.ps1 text eol=crlf
*.sh  text eol=lf
