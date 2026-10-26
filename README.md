# Brasfoot Next: banco de dados

O banco oficial do [Brasfoot Next](https://github.com/Brasfoot-Next/brasfoot-next): times, elencos, ligas, estaduais, escudos, camisas, troféus e seleções, num formato aberto de texto e imagens que qualquer pessoa pode ler, editar e estender.

| | |
|---|---|
| Clubes | 1.236, de 132 países |
| Jogadores | 34.895 |
| Ligas nacionais configuradas | 34 |
| Estaduais configurados | 27 |
| Escudos e camisas | 1.122 escudos, 2.805 camisas |
| Seleções | 224 |
| Troféus | 292 |

## No jogo

O Brasfoot Next já vem com este banco cadastrado. Na tela de abertura, **Patches** mostra as fontes de dados: este banco embaixo e, por cima, os patches que você adicionar pelo link do GitHub. Os dados entram no **Novo jogo**; carreiras em andamento não mudam.

O jogo baixa o pacote pronto do branch [`pack`](../../tree/pack), gerado automaticamente a cada mudança: poucos arquivos, com retomada e, numa atualização, só o que mudou.

## Formato

```text
manifest.toml
clubs/<país>/<clube>/club.toml, squad.csv, youth.csv, crest.png, kit.png, kit2.png, kit3.png
leagues/<país>.toml
state-leagues/<estado>.toml
national-teams/<país>/crest.png, kit.png, kit2.png
trophies/<tipo>/<nome>.png
```

Um clube é uma pasta: `club.toml` com nome, país, estado, nível, reputação, cores, clássicos, estádio e técnico; `squad.csv` com o elenco, que abre em qualquer planilha. Países pelo trigrama da FIFA, estados pela sigla.

A especificação completa, com todos os campos e valores, está em [docs/patches.md](https://github.com/Brasfoot-Next/brasfoot-next/blob/main/docs/patches.md).

## Contribuindo

Abra um pull request. Cada PR é validado automaticamente (campos obrigatórios, códigos de país, estado, posição e característica, nomes de pasta, ligas): o resultado aparece no próprio PR, com o arquivo e a linha de cada problema.

Para validar antes de enviar:

```sh
git clone https://github.com/Brasfoot-Next/brasfoot-next
cd brasfoot-next
cargo run --release -p brasfoot-datapack --bin brasfoot-data -- validate ../database
```

## Fazendo o seu patch

Um patch é um repositório com só o que muda. Por exemplo, elencos atualizados de alguns times:

```text
manifest.toml
clubs/bra/flamengo/squad.csv
clubs/bra/palmeiras/squad.csv
```

```toml
# manifest.toml
format = 1
id = "seu-usuario/elencos-janeiro"
kind = "patch"
name = "Elencos de janeiro"
extends = "brasfoot-next/database"
```

Cada arquivo do patch substitui o mesmo arquivo do banco; o que o patch não traz continua vindo do banco. Clubes novos são pastas novas. Para tirar clubes ou ligas, use `[remove]` no manifest.

Para o jogo baixar o seu patch, adicione `.github/workflows/pack.yml`:

```yaml
name: Pacote
on:
  push:
    branches: [main]
  pull_request:
jobs:
  pack:
    uses: Brasfoot-Next/brasfoot-next/.github/workflows/datapack.yml@main
    permissions:
      contents: write
    with:
      base: brasfoot-next/database
```

Depois é só colar o link do repositório na tela Patches.

## De onde veio: o patch antigo e seus defeitos

Este banco é a conversão de um patch do Brasfoot 22-23 (`teams/*.ban`, `conf_ligas_nacionais/*.cfg`, `conf_estadual/*.ces` e as pastas de imagens), feita com `brasfoot-data convert`. O formato do Brasfoot original tem problemas que motivaram o formato novo:

1. **Binário, só legível pelo jogo.** Times e ligas são objetos Java serializados. Não dá para abrir num editor, comparar versões, revisar uma mudança ou mesclar o trabalho de duas pessoas. Só o editor do próprio jogo mexe neles.
2. **O nome do arquivo é a identidade do time.** E não quer dizer o que parece: **305 dos 330 arquivos `_bra` são de clubes de outros países**; só 40 times são brasileiros. Os sufixos de país nem são consistentes (`_it` em 21 arquivos, `_ita` em 19).
3. **Clássicos presos ao nome do arquivo.** O jogo reconhece clássicos por uma tabela fixa de nomes de arquivo (`flarj`, `vasco`, `corinthians_bra`...). Como os arquivos do patch têm outros nomes, **18 dos 35 clássicos não existem** para o jogo. Aqui os clássicos são dados (`rivals` no `club.toml`).
4. **Potencial errado em quase todos os times.** Em **1.030 dos 1.236 times** o "potencial" de cada jogador era só a posição dele na lista (1, 2, 3... até 58). O jogo usa esse número como potencial, numa escala de 1 a 10: os goleiros, listados primeiro, nasciam com potencial mínimo e **19.586 jogadores tinham potencial acima do máximo**. Na conversão, o potencial desses times foi sorteado de novo com a mesma regra que o jogo usa para as categorias de base (por nível e reputação do clube, com semente fixa, então a conversão é reproduzível). Os 206 times com potencial de verdade ficaram como estavam.
5. **Imagens soltas e duplicadas.** Os escudos existem em duas pastas (`escudos/` e `teams/escudos/`, mais de 9.100 arquivos cada) para 1.236 times: **8.028 escudos não pertencem a time nenhum** e **114 times não têm escudo**. Há ainda duas pastas de escudos pequenos (outros 9.100 arquivos cada), que o jogo pode gerar redimensionando. Das 10.382 camisas, 7.564 não pertencem a time nenhum.
6. **Troféus com nomes irregulares.** `TR_` e `tr_` misturados, `tr_nacional_tur` em minúsculas (no Linux o jogo não acha), `RUS1` em vez de `_d1`, `NZE`, que não é código de país do jogo, e o próprio jogo exige o erro de digitação `tr_recopasulamaericana`. Aqui: `trophies/league/tur.png`, `trophies/continental/recopa-sudamericana.png`.
7. **Ligas desligadas pelo nome.** 190 dos 224 arquivos de liga estavam desativados só pelo sufixo `.disabled`, misturados aos ativos.
8. **Lixo e campos sem uso.** `colors.csv`, com 8.888 linhas e acentos quebrados, não é lido por código nenhum. Os campos `corBase`, `tid`, `sid` e `aid` dos times também não.
9. **Dados inválidos.** Jogadores com idade 0, 15 e 56 (o jogo troca por 35) e nomes com espaço sobrando.
10. **Tamanho.** O patch antigo ocupa uns 235 MB; este banco, 43 MB, e o pacote baixado pelo jogo, 25 MB.

Fora o potencial e os espaços sobrando, a conversão preserva tudo que o jogo usa: lendo os arquivos antigos e este banco, o Novo jogo recebe os mesmos times, jogadores e ligas, campo por campo.

## Licença

Os dados deste repositório estão sob a [Creative Commons Atribuição-CompartilhaIgual 4.0](LICENSE): você pode copiar, modificar e redistribuir, inclusive em outros patches, desde que cite a fonte e mantenha a mesma licença.

Nomes, escudos, camisas e marcas pertencem aos respectivos clubes, federações e jogadores, e aparecem aqui só para identificá-los num jogo de futebol. O Brasfoot Next é um projeto de fã, sem vínculo com os autores do Brasfoot original, com clubes ou com entidades do futebol.
