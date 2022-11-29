## Tipos de Projetos ASP.NET Core

O ASP.NET Core é o framework base para o Web API um template (ou tipo de projeto) que aplica o padrão REST (Integração e comunicação via requisições HTTP) utilizando a arquitetura MVC (Model-View-Controller).

## Estrutura do projeto

.csproj
    UserSecretId -> Guid para o secret local

**OpenAPI == Swagger**

Dependências
    Analyzers  -> Gestão e monitoramento de performance do VS
    NuGet -> Pacotes

Propriedades/launchSettings.json
    Configurar como sua aplicação vai iniciar dentro do VS

wwwroot
    Arquivos que atendam html, arquivos estáticos
    css, java scrit, libs, ícones
    Arquivos estáticos são arquios que não são compilados, são escritos em texto puro ou binários

App Settings
    Configurações


Owin (OPen Web Interface for .NET)


Middleware
    Processamento entre o recebimento do request e o envio do response

IConfiguration
    Serve, dentre outras coisas, para pegar a connection string lá no appSettings.json

Durante a inicialização da startup, ocorre a adição de configurações e posteriormente uso dessas configurações (middlewares).

Os middlewares fazem são pequenas partes do processo de inicialização e ajudam a executar parte da lógica.

A principal diferença entre uma classe normal e um middleware, é que o middleware é criado e utilizado na inicialização do projeto. Ele faz parte da pipeline de

