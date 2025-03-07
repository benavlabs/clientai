<style>
    .md-typeset h1,
    .md-content__button {
        display: none;
    }
</style>

<p align="center">
  <a href="https://github.com/igorbenav/clientai">
    <img src="../../assets/ClientAI.png?raw=true" alt="ClientAI logo" width="45%" height="auto">
  </a>
</p>
<p align="center" markdown=1>
  <i>Cliente unificado para provedores de IA com suporte padrão para agentes de IA</i>
</p>
<p align="center" markdown=1>
<a href="https://github.com/igorbenav/clientai/actions/workflows/tests.yml">
  <img src="https://github.com/igorbenav/clientai/actions/workflows/tests.yml/badge.svg" alt="Tests"/>
</a>
<a href="https://pypi.org/project/clientai/">
  <img src="https://img.shields.io/pypi/v/clientai?color=%2334D058&label=pypi%20package" alt="PyPi Version"/>
</a>
<a href="https://pypi.org/project/clientai/">
  <img src="https://img.shields.io/pypi/pyversions/clientai.svg?color=%2334D058" alt="Supported Python Versions"/>
</a>
</p>
<hr>
<p align="justify">
<b>ClientAI</b> é um pacote Python que fornece um framework unificado para construção de aplicativos com IA, oferecendo uma interação direta e transparente para os principais agentes de LLM, com excelente suporte para OpenAI, Replicate, Groq e Ollama.
</p>
<hr>

## Features

- **Interface Unificada**: Métodos claros e consistentes para vários provedores de IA (OpenAI, Replicate, Groq, Ollama).
- **Supporte a Streaming**: Chat com capacidade de resposta em tempo real com streamings.
- **Agentes Inteligentes**: Framework transparente para construção de workflows multi-step LLM com integrações a tools.
- **Validação de Saída**: Sistema de validação integrado para garantir saídas estruturadas e confiáveis em cada etapa.
- **Design Moddular**: Uso de componentes independentes, de simples encapsulamento para systemas completos de agentes.
- **Segurança de Tipos**: Dicas de tipos abrangentes para uma melhor experiência de desenvolvimento.

## Instalação

Para instalar o **ClientAI** com todos os provedores, execute:

```sh
pip install clientai[all]
```

Ou, se você preferir instalar provedores específicos, execute:

```sh
pip install clientai[openai]     # Suporte para OpenAI
pip install clientai[replicate]  # Suporte para Replicate
pip install clientai[ollama]     # Suporte para Ollama
pip install clientai[groq]       # Suporte para Groq
```

## Filosofia do Designer

O módulo ClientAI Agent é construído com base em três princípios fundamentais:

1. **Design Centrado em Prompts**: Prompts são a interface chave entre você e o LLM. Eles devem ser explícitos, debugáveis, e fáceis de entender. Sem prompts ocultos ou obscuros - o que você vê é o que é enviado ao modelo.

2. **Personalização em Primeiro Lugar**: Cada componente foi desenhado para ser extendido ou sobrescito. Crie etapas customizáveis com ferramentas de seleção, ou desenvolva novos padrões de Workflow. A arquiterura abraça suas modificações.

3. **Sem travas (Zero Lock-In)**: Começe utilizando os componentes de alto nível e desça para níveis mais baixos conforme necessário. Você pode:
    - Extender `Agents` para tarefas customizadas.
    - Usar componentes individuais diretamente.
    - Substituir partes gradualmente com sua própria implementação.
    - Ou migre gradualmente para outras soluções - não há travas.

## Exemplos

### 1. Uso Básico do ClientAI

```python
from clientai import ClientAI

# Inicializando com a OpenAI
client = ClientAI('openai', api_key="your-openai-key")

# Geração de texto
resposta = client.generate_text(
    "Me conte uma piada nerd sobre computadores e inteligência artificial",
    model="gpt-3.5-turbo",
)

print(resposta)

# Funcionalidade de chat
messages = [
    {"role": "user", "content": "Qual a capital do estado do Pará?"},
    {"role": "assistant", "content": "Belém"},
    {"role": "user", "content": "Qual a sua população?"}
]

resposta = client.chat(
    messages,
    model="gpt-3.5-turbo",
)

print(resposta)
```

### 2. Começando com Agentes

O jeito mais fácil de criar um agente simples:

```python
from clientai import ClientAI as client
from clientai.agent import create_agent, tool

# criando uma tool com uma descrição explícita
@tool(name="soma", description="Soma dois números.")
def soma(x: int, y: int) -> int:
    return x + y

# criando uma tool usando as descrições com docstring
@tool(name="multiplica")
def multiplica(x: int, y: int) -> int:
    """Multiplica dois números e retorna o produto"""
    return x * y

calculadora = create_agent(
    client=client("groq", api_key="your-groq-key"),
    role="calculadora",
    system_prompt="Você é um assistente calculadora.",
    model="llama-3.2-3b-preview",
    tools=[soma, multiplica]
)

resultado = calculadora.run("Qual a soma de 5 e 3 multiplicado por 2? ")
print(resultado)
```

### 3. Customize o seu agente com Workflow

Para um maior controle, crie agentes customizados com passos definidos:

```python
from clientai import ClientAI as client
from clientai.agent import Agent, think, act, tool

# Criando uma tool independente
@tool(name="calcula_media")
def calcula_media(numeros: list[float]) -> float:
    """Calcula a media aritmética de uma lista de números"""
    return sum(numeros) / len(numeros)

# Agente Customizado com Workflow
class AnalizaDados(Agent):
    # Adiciona uma etápa de análise
    @think("analiza")
    def analiza_dados(self, dados_de_entrada: str) -> str:
        """Analise os dados de vendas calculando métricas-chave."""
        return f"""
            Por favor, analise estes valores de vendas:

            {dados_de_entrada}

            Calcule a média usando a tool calcula_media e identifique a tendência.
            """

    # Adiciona também uma etapa de ação
    @act
    def resumo_da_analise(self, analise: str) -> str:
        """Crie um breve resumo da análise"""
        return """
            Crie um breve resumo que inclua:
            1. A média dos valores de vendas
            2. Se as vendas estão em tendência de alta ou baixa
            3. Uma recomendação-chave
            """

analizador = AnalizaDados(
    client=client("replicate", api_key="your-replicate-key"),
    default_model="meta/meta-llama-3-70b-instruct",
    tool_confidence=0.8,
    tools=[calcula_media]
)

resultado = analizador.run("Vendas do Mês: [1000, 1200, 950, 1100]")
print(resultado)
```

### 4. Agente personalizado com validação de tipos

Para garantir um output estruturado com tipagem segura:

```python
from clientai import ClientAI as client
from clientai.agent import Agent, think
from pydantic import BaseModel, Field
from typing import List

class Analises(BaseModel):
    resumo: str = Field(min_length=10)
    pontos_chave: List[str] = Field(min_items=1)
    sentimento: str = Field(pattern="^(positivo|negativo|neutro)$")

class AnalisaDados(Agent):
    @think(
        name="analisa",
        json_output=True,  # Habilita formtação em JSON
    )

    def analisa_dados(self, dados: str) -> Analises: # Habilita o padrão de validação do output
        """Analisa os dados com validação estruturada de output"""
        return """
            Analise os dados e retorne um JSON com as chaves:
            - resumo: pelo menos 10 caracteres
            - pontos_chave: lista não vazia
            - sentimento: positive, negative, or neutral

            Dados: {dados}
        """

# Inicialização e uso
analizador = AnalisaDados(client=client("openai", api_key="your-groq-key"), default_model="gpt-3.5-turbo")
resultado = analizador.run("Crescimento de vendas em 25% no último trimestre.")
print(f"Sentimento: {resultado.sentimento}")
print(f"Pontos Chave: {resultado.pontos_chave}")
```

## Requisitos

Antes de instalar ClientAI, certifique-se de atender os requisitos abaixo:

* **Python:** Versão 3.9 ou mais recente.
* **Dependências:** O pacote principal do ClientAI possue dependências mínimas. Pacotes específicos de provedores (e.g., `openai`, `replicate`, `ollama`, `groq`) são opcionais e podem ser instalados separadamente.

## Uso

ClientAI oferece três principais maneiras para interagir com provedores de IA:

1. Gereção de textos: Use o método `generate_text` para tarefas de geração de texto.
2. Chat: Use o método `chat` para conversas interativas.
3. Agentes: Criação de agentes inteligentes com altomação de tools, seleção e gerenciamento de workflows.

Todos os métodos suportam respostas de streaming e retornam os objetos completos.

## Próximos Passos

1. Visite aqui o [Guia de usuário](usage/overview.md) para detalhamento de funcionalidades e features avançadas.
2. Veja aqui [Referência da API](api/overview.md) para documentação completa da API.
3. Para desenvolvimento de agentes, visite: [Guia para Agentes.](usage/agent/creating_agents.md)
4. Explore os [Exemplos](examples/overview.md) para praticar solução baseadas no mundo real e uso de padrões.

Lembre-se de tratar com cuidado suas chaves de api (Api Keys), e nunca expô-las em no código fonte ou versionador de código.

## License

[`MIT`](community/LICENSE.md)