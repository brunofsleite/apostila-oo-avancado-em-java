# Arquitetura Hexagonal

## Por uma arquitetura centrada no domínio

No capítulo sobre DIP, citamos uma das arquiteturas mais comuns em aplicações corporativas: a arquitetura em 3 camadas. Nessa arquitetura, Apresentação depende de Negócio que depende de Persistência.

A dependência da camada de Negócio, de alto nível, em direção à camada de Persistência, de baixo nível, viola o DIP.

Por meio de abstrações fornecidas pela camada de Negócio e implementadas pela camada de Persistência, invertemos as dependências.

Ao invertermos as dependências, faríamos com que a Persistência dependa do Negócio, e não o contrário. Alto nível não depende mais de baixo nível. O DIP é respeitado.

![3 Camadas x Dependências Invertidas {w=35}](assets/imagens/cap04-dependency-inversion-principle/camadas-vs-dependencias-invertidas.png)

Todas as setas apontariam para o Negócio. Teríamos uma arquitetura centrada na lógica de negócio. O modelo de domínio passaria a ser o núcleo da aplicação.

## Hexágonos (ou Conectores & Adaptadores)

Alistair Cockburn, em seu artigo [Hexagonal Architecture](http://alistair.cockburn.us/Hexagonal+architecture) (COCKBURN, 2005), define uma estrutura de alto nível separa o código em duas partes: a de dentro e a de fora. A parte interna contém a lógica de negócio. Já a externa, contém detalhes de implementação, como UI e Persistência.

A parte interna, de Negócio, fornece uma API que é usada e/ou implementada pela parte externa.

O intuito, no fim das contas, é organizar a aplicação apartando o código relacionado a domínio do código mais técnico, relacionado a mecanismos de entrega.

Dessa maneira, é possível guiar o núcleo da aplicação por diferentes clientes: além da UI, scripts, outros programas e testes automatizados.

A mesma arquitetura é chamada por Alistair Cockburn de **Ports & Adapters**, algo como Conectores e Adaptadores. A API fornecida pela aplicação é análoga aos _ports_ (ou conectores) de dispositivos eletrônicos, que permitem que dispositivos externos sejam plugados. Cada conector é ligado a outros dispositivos por um ou mais adaptadores.

Um conector de UI fornecido pelo núcleo da aplicação pode ser usado por dispositivos como:

- uma UI de linha de comando
- uma UI Web
- um teste automatizado
- um reconhecedor de voz
- um Chat Bot

Já um conector de Persistência, pode ser usado por:

- um BD Relacional
- um BD Não-Relacional
- um BD em memória, para testes automatizados
- um WebService externo, como o Firebase

Um adaptador é código que implementa um conector, fazendo a ponte com um determinado dispositivo. Nos termos de Alistair Cockburn, traduz o protocolo do conector ao do dispositivo.

O hexágono é uma metáfora visual que dá a ideia de uma parte interna e uma parte externa. Além disso, um hexágono não unidimensional como um desenho de camadas. Os seis lados do hexágono remetem aos vários conectores de entrada e saída.

> _A idéia da Arquitetura Hexagonal é colocar entradas e saídas nas bordas do nosso design. A lógica de negócios não deve depender da exposição de uma API REST ou GraphQL, nem de onde obtemos dados: um banco de dados, uma API de um Microservice exposta por gRPC ou REST, ou um simples arquivo CSV._
>
> _Esse pattern nos permite isolar o núcleo da lógica de nossa aplicação de preocupações externas. Dessa maneira, podemos alterar facilmente os detalhes da fonte de dados **sem um impacto significativo e sem reescrever grande parte do código**._
>
> _Uma das principais vantagens que vimos ao ter uma aplicação com limites claros é a nossa estratégia de teste: a maioria dos nossos testes pode verificar nossa lógica de negócios **sem depender de protocolos que podem mudar facilmente**._
>
> Damir Svrtan e Sergii Makagon, engenheiros da Netflix, no post [Ready for changes with Hexagonal Architecture](https://netflixtechblog.com/ready-for-changes-with-hexagonal-architecture-b315ec967749) (SVRTAN; MAKAGON, 2020)

## Arquitetura Hexagonal no Cotuba

O Cotuba tem 5 módulos:

- `cotuba-core`, de alto nível, contém a lógica de negócio.
- `cotuba-cli`, de baixo nível, define uma UI de linha de comando.
- `cotuba-web`, de baixo nível, define uma UI Web.
- `cotuba-pdf`, de baixo nível, gera PDFs.
- `cotuba-epub`, de baixo nível, gera EPUBs.

O módulo central, o núcleo da aplicação, é o `cotuba-core`. Esse módulo renderiza MDs para HTMLs, aplica temas, chama o gerador de ebook no formato apropriado e permite ações pós-geração. Alguns dos conectores (ou _ports_) fornecidos são:

- `ParametrosCotuba`, um conector usado para receber parâmetros como o formato de ebook a ser gerado e o arquivo de saída.
- `RepositorioDeMDs`, um conector usado como fonte dos MDs.
- `GeradorEbook`, um conector usado para a geração de ebooks em diferentes formatos.

O módulo `cotuba-cli` torna o Cotuba acessível via Terminal, fornecendo adaptadores para os conectores `ParametrosCotuba`, que obtém parâmetros das opções de linha de comando, e `RepositorioDeMDs`, que lê os MDs de um diretório.

O módulo `cotuba-web` torna o Cotuba acessível via Navegador. O adaptador de`ParametrosCotuba` desse módulo obtém o formato do ebook a ser gerado da URL e o arquivo de saída de um diretório aleatório. Já o adaptador de `RepositorioDeMDs` lê os MDs do BD.

O módulo `cotuba-pdf` fornece um adaptador para `GeradorEbook` que usa a biblioteca iText pdfHTML para gerar um PDF a partir de HTMLs.

O módulo `cotuba-epub` fornece um adaptador para `GeradorEbook` que gerar um EPUB a partir dos HTMLs por meio da biblioteca Epublib.

![Cotuba como um hexágono {w=34}](assets/imagens/cap13-arquitetura-hexagonal/ports-and-adapters-cotuba.png)

Poderíamos definir mais módulos, encaixando-os nos conectores já existentes. Por exemplo, poderíamos definir um módulo `cotuba-mobi`, que gera um arquivo `.mobi`, adequado para ser lido no _e-reader_ Kindle. Para isso, o módulo `cotuba-core` não precisaria ser modificado.

### Os conectores de plugins do Cotuba

O módulo `cotuba-core` define também conectores para plugins:

- `Tema`, um conector para temas, usado pelo módulo `tema-paradizo`.
- `AoFinalizarGeracao`, um conector que permite ações após a geração do ebook, usado pelo módulo `estatisticas-ebook`.

Os módulos `tema-paradizo` e `estatisticas-ebook`