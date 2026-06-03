# CapSign Schema

[English](README.md) · **Português**

Um schema legível por máquina para descrever o **layout de placas de sinalização rodoviária brasileiras** (CONTRAN / DNIT IPR-743) — o suficiente para redesenhar a face da placa, e não apenas catalogar onde ela está instalada.

O CapSign descreve a **composição** de uma placa estática e fabricada: seu tipo, sua categoria de cor e os módulos, linhas de texto, setas, pictogramas, escudos de rota e tarjas que a compõem. É um **contrato de interface** entre qualquer coisa que *reconhece ou cria* uma placa e qualquer coisa que a *desenha, valida ou armazena*.

Ele preenche uma lacuna que nenhum padrão existente cobre. Sistemas de inventário de placas armazenam placas como pontos localizados com atributos de condição (*onde* uma placa está); o DATEX II modela conteúdo dinâmico de painéis eletrônicos (VMS); os manuais do DNIT/CONTRAN definem as regras visuais — mas nenhum descreve o layout composicional de uma placa de orientação personalizada em formato legível por máquina. O CapSign descreve.

## Conteúdo

| Arquivo | O que é |
|------|------------|
| [`capsign.schema.json`](capsign.schema.json) | A definição autoritativa e verificável por máquina (JSON Schema 2020-12). |
| [`SPECIFICATION.md`](SPECIFICATION.md) | A especificação legível por humanos: intenção, regras e o julgamento que um produtor aplica. |
| [`examples/`](examples/) | Placas reais expressas como instâncias conformes. |
| `LICENSE` | MIT (schema e código). O documento de especificação também é oferecido sob CC BY 4.0. |

## Visão rápida

Uma placa de orientação simples com coluna de km:

```json
{
  "sign": {
    "type": "orientação",
    "colorScheme": "orientação",
    "confidence": 1.0,
    "modules": [
      { "lines": [
        { "text": "Curitiba", "km": 85, "fontSize": 150 },
        { "text": "Ponta Grossa", "km": 23, "fontSize": 150 }
      ] }
    ]
  }
}
```

Uma linha de destino terminando em uma tarja de rota azul embutida (array de segmentos):

```json
[
  { "text": "P. Alegre", "fontSize": 150 },
  { "text": "BR-101", "fontSize": 150, "backColor": "blue" }
]
```

Veja [`examples/`](examples/) para seis casos trabalhados, incluindo uma placa turística com múltiplos módulos, um elemento de escudo de rota, uma fileira de pictogramas e uma placa de advertência codificada.

## Conceitos centrais (um parágrafo)

Uma placa tem um **type** (`modules`, `orientação`, `pictograms`, `tree`, `special`) e um **colorScheme** (categoria de cor CONTRAN, lida a partir do *corpo* da placa). Uma placa `modules`/`orientação` é uma pilha de **módulos** (painéis); um módulo contém **linhas**; uma linha é ou um objeto de texto ou — quando a cor muda *dentro* da linha — um array de **segmentos**. Linhas e módulos podem carregar **elementos**: setas, pictogramas ou escudos de rota, cada um com uma `position`. Um **módulo** separado existe somente onde uma borda visível divide os painéis; uma tarja colorida flutuando dentro de um painel é um `backColor` de linha, não um módulo. Veja a [especificação](SPECIFICATION.md) para as regras completas.

## Validar uma instância

```bash
pip install jsonschema
```

```python
import json
from jsonschema import Draft202012Validator

schema = json.load(open("capsign.schema.json"))
instance = json.load(open("examples/01-orientacao-km-shield.json"))

errors = list(Draft202012Validator(schema).iter_errors(instance))
print("valid" if not errors else [e.message for e in errors])
```

## Vocabulário de pictogramas recomendado

O `id` de um pictograma é ou um código oficial do CONTRAN (ex.: `A-18`, `R-19`) para símbolos padrão de advertência/regulamentação, ou um termo descritivo em minúsculas para símbolos de serviço/turismo. O conjunto descritivo é aberto; um consumidor mapeia termos desconhecidos para um marcador genérico. Um vocabulário inicial recomendado:

`fuel`, `camping`, `airport`, `ferry`, `restroom`, `tyre`, `post-office`, `parking`, `heliport`, `hospital`, `information`, `port`, `restaurant`, `phone`, `mechanic`, `bus-stop`, `train-terminal`, `boat-terminal`, `toll`, `church`, `rural-tourism`, `yacht-club`, `waterfall`, `beach`, `museum`, `theater`, `monument`, `cycling`, `horse-riding`, `fishing`, `diving`, `surfing`, `viewpoint`, `cave`, `island`, `lighthouse`, `zoo`, `park`, `ruins`, `cultural-center`, `golf`, `soccer`, `handicraft`.

## Escopo

**No escopo:** a face da placa — tipo, cor, painéis, linhas, texto, setas, pictogramas, escudos, tarjas, tamanhos relativos.

**Fora do escopo:** local de instalação, condição, retrorrefletividade, material (use um sistema de inventário — o CapSign combina perfeitamente com um); geometria de renderização, fontes, detalhe de CAD (responsabilidade do consumidor); conteúdo dinâmico de painéis eletrônicos VMS (veja o DATEX II).

## Status e versionamento

Versão 1.0, estável. Os tipos de placa `tree` e `special` são intencionalmente opacos na 1.0 (apenas dimensões). Adições retrocompatíveis incrementam a versão menor; mudanças incompatíveis incrementam a versão maior.

## Licença

Schema, exemplos e qualquer código: **MIT** (veja `LICENSE`). O documento de especificação em prosa está adicionalmente disponível sob **CC BY 4.0**. Em resumo: use-o, construa sobre ele, comercialmente ou não, com atribuição.

## Contribuindo

Issues e propostas — novos termos de pictograma, tipos adicionais de placa, esclarecimentos — são bem-vindos. Como as instâncias são validadas de forma estrita, por favor inclua uma placa de exemplo (e, quando relevante, uma foto ou referência) com qualquer mudança estrutural proposta.
