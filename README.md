# Página pública e downloads

O site em `website/` é independente do aplicativo. HTML/CSS/JavaScript estáticos são publicados no GitHub Pages do repositório `phpforge-dev/php-ide`, o único endereço público. A autoria do site é Jeidison Farias. Não há serviço próprio nem credenciais no navegador.

Publicação: [phpforge-dev.github.io/php-ide](https://phpforge-dev.github.io/php-ide/). Repositório do site: [phpforge-dev/php-ide-website](https://github.com/phpforge-dev/php-ide-website). A conta `jeidison` é proprietária da organização. Plugins são distribuídos em [phpforge-dev/plugins](https://github.com/phpforge-dev/plugins).

## Publicar

1. Envie os arquivos do site para `main`. O workflow `.github/workflows/pages.yml` valida o build e pede ao `php-ide` (evento `website-updated`) que publique.
2. O pedido usa o secret `PHP_IDE_DISPATCH_TOKEN`: um token fine-grained com acesso só a `phpforge-dev/php-ide` e permissão **Contents: Read and write**. Sem o secret, o job avisa e não publica.
3. Para publicar à mão: `gh workflow run website.yml -R phpforge-dev/php-ide` ou **php-ide → Actions → Project website → Run workflow**.
4. O GitHub Pages deste repositório fica desligado, para não existir um segundo site.

O build consulta as releases de `phpforge-dev/php-ide`, independentemente do repositório do site. Para alterar a origem dos downloads, defina `PAGES_REPOSITORY=organizacao/repositorio`. Publica somente os arquivos estáticos explicitamente copiados pelo script; código da IDE, fixtures e artefatos de build não entram no site.

## Disponibilizar instaladores

Crie uma release no repositório `phpforge-dev/php-ide` e anexe os instaladores efetivamente gerados e testados. Consulte [o guia de build](https://github.com/phpforge-dev/php-ide/blob/main/docs/development/contributing.md#build-e-instalador-macos). A página consulta as 30 releases mais recentes via API pública, escolhe a primeira estável ou, se nenhuma existir, a primeira prévia. **Todos os downloads exibidos pertencem à mesma release**; não mistura versões antigas para preencher plataformas ausentes.

| Sistema | Arquivos reconhecidos | Arquiteturas nos nomes |
|---|---|---|
| macOS | `.dmg` | `aarch64`/`arm64`, `x64`/`x86_64`, `universal` |
| Windows | `.exe`, `.msi` | `x64`/`x86_64`/`amd64`, `arm64`/`aarch64`, `x86`/`i686` |
| Linux | `.AppImage`, `.deb`, `.rpm` | `amd64`/`x86_64`, `arm64`/`aarch64`, `i686` |

Use nomes como `PHP.IDE_0.2.7_aarch64.dmg`; sem arquitetura reconhecida, o botão orienta consultar a release. Arquivos `.sha256` não viram botões de instalador: ficam acessíveis por **Todas as versões**. Nunca renomeie um pacote de uma arquitetura para outra.

Sem instalador, o cartão informa **Aguardando publicação**. Erros de rede/limite da API mostram **Tentar novamente** e o link de releases continua acessível. Novos instaladores anexados à release aparecem após recarregar a página, sem redeploy. A consulta depende de releases públicas; não coloque tokens no site para acessar repositórios privados.

## Prévia local

```sh
node scripts/build-pages.mjs
python3 -m http.server 4178 --bind 127.0.0.1 --directory .artifacts/pages
```

Abra `http://127.0.0.1:4178`. Por padrão, a página consulta as releases públicas de `phpforge-dev/php-ide`. Para consultar releases públicas existentes, gere novamente com `PAGES_REPOSITORY=organizacao/repositorio node scripts/build-pages.mjs`. O valor de configuração contém apenas o nome público do repositório.

## Evidência e limites

Verificação da página: `node scripts/check-pages.mjs` (requer Chromium do Playwright; alternativamente `PLAYWRIGHT_CHANNEL=chrome` usa Chrome instalado). Exercita respostas sintéticas de releases, arquiteturas, prévias, ausência de instaladores, erro/repetição da consulta e layout móvel. Não publica pacotes nem comprova execução da IDE em outros sistemas.

Página própria, sem inferência de compatibilidade com PhpStorm. As capturas da própria IDE têm proveniência em [website/screenshots/README.md](website/screenshots/README.md). O workflow publica a página; **não compila instaladores**. A release 0.2.8 disponibiliza DMG macOS Apple Silicon, EXE Windows x64 e DEB/AppImage Linux x64, com SHA-256. Veja [builds Windows/Linux](https://github.com/phpforge-dev/php-ide/blob/main/docs/development/windows-linux-builds.md) e [evidências e limites](https://github.com/phpforge-dev/php-ide/blob/main/docs/research/desktop-distribution.md). Outras arquiteturas dependem de builds e validações próprios.

Referência: [workflow oficial do GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages).

## Desenvolvimento

Execute `npm ci`, `npm run build` e `npm test`. O teste usa Chromium do Playwright (`npx playwright install chromium`) ou `PLAYWRIGHT_CHANNEL=chrome`.
