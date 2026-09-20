# Coletor de dados de sites

Lê uma página, extrai os itens que você indicar e grava tudo numa planilha do
Excel. Na próxima execução, só acrescenta o que é novo.

Feito para tarefas do tipo "toda segunda eu copio os preços/vagas/anúncios
desse site para uma planilha".

*A web scraper that turns any list page into a clean Excel file. Selectors live
in a YAML config, so the same program works on a new site without touching the
code. Skips items already collected, respects robots.txt, and ships with tests.*

## O que ele faz

- Os seletores ficam num arquivo `.yaml`, então o mesmo programa serve para
  outro site sem mexer no código.
- Percorre várias páginas, seguindo o link de "próxima página".
- Não grava duas vezes o mesmo item: compara pela coluna-chave.
- Respeita o `robots.txt` do site e faz uma pausa entre as páginas.
- Transforma links relativos em endereços completos.
- Tenta de novo quando a conexão falha, com espera crescente.
- Registra tudo o que está fazendo na tela.

## Instalação

```bash
pip install -r requirements.txt
```

## Uso

```bash
# coleta de verdade
python -m coletor.principal exemplos/livros.yaml

# mostra o que encontrou, sem gravar nada
python -m coletor.principal exemplos/livros.yaml --teste

# muda o número de páginas ou o arquivo de saída
python -m coletor.principal exemplos/livros.yaml --paginas 5 --saida saida/meus_livros.xlsx
```

## O arquivo de configuração

```yaml
url: "https://books.toscrape.com/"      # por onde começar
seletor_item: "article.product_pod"     # o bloco que se repete na página
campo_chave: "titulo"                   # coluna usada para não duplicar
campos:                                 # uma entrada por coluna da planilha
  - nome: "titulo"
    seletor: "h3 a"
    atributo: "title"                   # sem 'atributo', pega o texto
  - nome: "preco"
    seletor: "p.price_color"
seletor_proxima_pagina: "li.next a"     # opcional
paginas: 2
espera_segundos: 1.0                    # pausa entre páginas
arquivo_saida: "saida/livros.xlsx"
aba: "Livros"
```

Para descobrir os seletores: abra o site, clique com o botão direito sobre o
item, escolha "Inspecionar" e veja a classe do bloco.

Em `exemplos/` há dois arquivos: `livros.yaml`, que funciona de verdade no
books.toscrape.com (um site feito para treinar coleta), e `vagas.yaml`, um
modelo em branco para adaptar.

## Testes

```bash
pytest -q
```

Os testes usam uma página HTML guardada em `tests/`, então rodam sem internet
e sempre dão o mesmo resultado.

## Agendar a coleta

**Windows (Agendador de Tarefas)**

1. Abra o Agendador de Tarefas e clique em "Criar Tarefa Básica".
2. Escolha a frequência, por exemplo toda segunda-feira às 8h.
3. Em "Ação", escolha "Iniciar um programa".
4. Programa: `python`
5. Argumentos: `-m coletor.principal exemplos/livros.yaml`
6. Iniciar em: a pasta do projeto.

**Linux ou macOS (cron)**

```
0 8 * * 1 cd /caminho/do/projeto && python3 -m coletor.principal exemplos/livros.yaml
```

## Limites

- Serve para páginas que já vêm prontas do servidor. Sites que montam a lista
  com JavaScript depois de carregar precisam de outra abordagem (Playwright).
- Se o site mudar o layout, os seletores precisam ser atualizados. É só editar
  o `.yaml`.
- Colete apenas dados públicos e respeite os termos de uso de cada site.

## Licença

MIT.
