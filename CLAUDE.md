# Contexto do projeto — Relatórios de Física 1 Experimental (UnB Gama)

Repositório de relatórios da disciplina **Física 1 Experimental (FIS EXP 1)** da
Universidade de Brasília – Campus Gama. Cada experimento tem sua pasta com os
materiais (dados, gráficos, imagens) e o relatório final em PDF. Uma pipeline de
CI compila/coleta os PDFs e os publica numa release do GitHub.

- **Repositório:** https://github.com/renannOgomes/fis-exp-2026-01
- **Branch principal:** `master`
- **Release dos PDFs:** https://github.com/renannOgomes/fis-exp-2026-01/releases/tag/latest

## Estrutura de pastas

```
.
├── .github/workflows/build-latex.yml   # Pipeline de CI (compila/coleta e publica)
├── .gitignore                          # Ignora auxiliares do LaTeX e o PDF gerado
├── Plano_de_ensino.pdf                 # Plano de ensino da disciplina
├── Modelo de Relatório-20260420/       # Modelo LaTeX de referência (NÃO entra na pipeline)
│   ├── Relatorio_Modelo.tex
│   └── imagens do modelo (.jpg/.png)
├── Relat_1/  Relat_2/  Relat_3/        # Experimentos com o PDF final pronto
│   └── Relat_N.pdf  (+ dados, gráficos, .sciprj)
└── Relat_4/                            # Experimento compilado a partir do LaTeX
    ├── Relatorio_Modelo.tex
    └── graf*.jpg / *.sciprj
```

### Convenções importantes

- **Só pastas no padrão `Relat_*/` entram na pipeline.** A pasta
  `Modelo de Relatório-20260420/` é apenas referência e é ignorada de propósito.
- Cada relatório pode existir de **duas formas**, e a pipeline decide o que fazer:
  1. **PDF final já pronto** — arquivo `Relat_N/Relat_N.pdf` (mesmo nome da pasta).
     Nesse caso a pipeline **só reaproveita** o PDF, sem recompilar.
  2. **Fonte LaTeX** — arquivo `Relat_N/Relatorio_Modelo.tex`. Nesse caso a pipeline
     **compila** com `latexmk` e renomeia a saída para `Relat_N.pdf`.
  - Se a pasta tiver os dois, o PDF pronto tem prioridade. Se não tiver nenhum, é pulada.
- O `.gitignore` versiona os PDFs finais (`Relat_N.pdf`) mas **ignora** os auxiliares
  do LaTeX (`*.aux`, `*.log`, etc.) e o `Relatorio_Modelo.pdf` gerado localmente
  (esse é gerado pela pipeline, não deve ser commitado).
- Arquivos `.sciprj` são projetos do **SciDAVis** usados para gerar os gráficos.

## Pipeline de CI (`.github/workflows/build-latex.yml`)

- **Dispara** a cada `push` na `master` e também manualmente (`workflow_dispatch`
  pela aba Actions).
- Roda no container oficial **`texlive/texlive:latest`** (TeX Live completo: já
  inclui `pdfpages`, `babel-portuges`, etc.).
- **Passo de coleta/compilação** (usa `shell: bash` — o container usa `sh` por
  padrão, que não tem `shopt`): para cada `Relat_*/`, aplica a regra de
  reaproveitar-ou-compilar descrita acima e junta tudo em `saida/Relat_N.pdf`.
- **Publicação:** `softprops/action-gh-release@v2` envia `saida/*.pdf` para a
  release de tag fixa `latest` (sobrescreve a cada build).
- Requer `permissions: contents: write` para criar/atualizar a release.

### Como adicionar um novo relatório

- **Entrega já pronta em PDF:** crie a pasta `Relat_N/`, coloque o PDF como
  `Relat_N/Relat_N.pdf`, commit e push.
- **A partir do LaTeX:** crie `Relat_N/Relatorio_Modelo.tex` (mais as imagens que
  ele referencia), commit e push. A pipeline compila e publica como `Relat_N.pdf`.

## Tarefas comuns / dicas

- **Ver status do último build:** `gh run list --repo renannOgomes/fis-exp-2026-01 --limit 1`
- **Ver logs de falha:** `gh run view <id> --repo renannOgomes/fis-exp-2026-01 --log-failed`
- **Listar PDFs na release:** `gh release view latest --repo renannOgomes/fis-exp-2026-01`
- **Ambiente:** Windows + PowerShell. O `gh` CLI está instalado em
  `C:\Program Files\GitHub CLI\gh.exe` e autenticado.
- **Manutenção pendente:** `actions/checkout@v4` e `softprops/action-gh-release@v2`
  ainda usam Node 20 (em descontinuação). Bumpar quando houver versão nova.
