# Dominando ASP.NET Core

## Rotas

O jeito mais prático de definir rotas é usando o map all dentro do program.cs
    var builder = WebApplication.CreateBuilder(args);
    var app = builder.Build();

    app.MapControllers();

E em cada controller, usar notations para definir a rota

    [ApiController]
    [Route("api/{controller}/{action}/{id?:int}")]
    public abstract class BaseApiController : ControllerBase, IApiController
    {
        ...
    }

Assim, o nome da minha controller "BaseApi" é automaticamente mapeado para a rota

## Action Results

IActionResult - Síncrono -> Apropriado quando vários tipos de Action Result podem ser retornados

    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status200OK, Type = typeof(Product))]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public IActionResult GetById_IActionResult(int id)
    {
        var product = _productContext.Products.Find(id);
        return product == null ? NotFound() : Ok(product);
    }

Nesse caso, o retorno poderá ser tanto NotFound() (HTTP 404) ou um 200 com o objeto product.

Task<IActionResult> - Assíncrono -> Só faz sentido usar quando você precisa de um retorno assíncrono, como o anterior, pode retornar vários tipos de Action results

    [HttpPost()]
    [Consumes(MediaTypeNames.Application.Json)]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> CreateAsync_IActionResult(Product product)
    {
        if (product.Description.Contains("XYZ Widget"))
        {
            return BadRequest();
        }

        _productContext.Products.Add(product);
        await _productContext.SaveChangesAsync();

        return CreatedAtAction(nameof(GetById_IActionResult), new { id = product.Id }, product);
    }

ActionResult<T> x IActionResult

Um ActionResult<T> permite que você retorne tanto um action result como um tipo específico, mas limita que esse tipo seja **T**. Ou seja, você pode retornar um BadRequest(), NotFoun(), Ok(T) ou T. Mas não pode retornar Ok(A) ou A, porquê eles não são do tipo prédefinido.

Pra reforçar, a Task só é usada quando você precisa que o método seja assíncrono. Você poderia retornar apenas Task<T> (sem o ActionResult), mas aí você perderia a capacidade de retornar 404, 500 ou 200 e retornaria somente o objeto T. Mas, em alguns casos isso pode fazer sentido, quando você da um get em um conjunto de itens. Você pode receber uma lista de itens ou uma lista vazia não tem uma terceira opção.

---------------------------------------------------
---------------------------------------------------
---------------------------------------------------
---------------------------------------------------

# WEB API

- REST
É um modelo arquitetural que cria uma camada de abstração para que sistemas distribuídos (APIs) se comuniquem utilizando um protocolo web (HTTP, por exemplo).
 
- REST e Microsserviços
Os microsserviços levam a arquitetura REST pra um segundo nível. Enquanto que em monolitos uma mesma aplicação tem múltiplas responsabilidades (autenticação, regra de negócio, CRUD...) em uma arquitetura de microsserviços as responsabilidades são bastante segregadas. Cada API faz, idealmente, apenas uma única coisa.

 - MainController
 Pra poder generalizar algumas coisas na controller, você pode criar uma classe abstrata **MainController** que herda da ControllerBase e tem a notação [ApiController]. Dentro dessa classe abstrata, você cria os métodos protected (para que apenas as classes que herdarem dela tenham acesso) e implementa o que fizer sentido pra você, um CustomResponse ou um validador genérico, por exemplo.

 Classes virtuais podem ser sobrescritas (override)


Analyzers -> Me força a implementar a notation do [ProducesResponseType] para ajudar na documentação  